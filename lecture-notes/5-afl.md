---
layout: page
title: Lecture 5 - AFL
parent: Lecture Notes
nav_order: 5
---

# Lecture 5- AFL
Agenda:
1. Friday check-in
2. Random vs not entirely random fuzzing
3. AFL Discussion

## From random to not entirely random fuzzing
Consider this:
```c
void magic(byte input1){
	if(input1 == 45){
		crash();
	}
}
```
What is the probability of the random fuzzer to find this crash? 1/(256) = 1/(2^8)

```c
void magic(byte input1, byte input2){
	if(input1 == 45){ //1/2^8 for this
		if(input2 == 36){ //1/2^8 for this
			crash();
		}
	}
}
```
What is the probability of the random fuzzer to find THIS crash? 1/(256*256) = 1/65,536 = 1/(2^16)

Q: What kinds of bugs are more likely to be found by this purely random fuzzer? What kinds of triggering conditions?
* The error conditions in parsing - inputs where there can be a single (or few) bytes that are invalid,
Q: What can we do to change our fuzzer to make it more likely that it will detect this crash?
* Symbolic execution - For each input, represent it as a symbolic value (instead of some actual concrete number), then detect the constraints that are applied on those symbolic variables, use SAT solver to generate new inputs that will explore different paths: Generate inputs (0,0), (45, 0), (45, 36)
* Keep fuzzing black box, but use some model of the input format. For example: if fuzzing something that accepts some particular file format (like JSON), then could write something that generates valid JSON
* Guess inputs until you find one that takes the true branch of the first if, then use that input as a starting point and guess inputs that get you to the second if.

Examples of inputs for the above code:
```
(0, 0)
(45, 0)
(0, 36)
```

Key idea of mutational fuzzing: Use some feedback to find which inputs are interesting (“greybox fuzzing”)
Loosely speaking here’s what would happen:
```c
void magic(byte input1, byte input2){
	if(input1 == 45){ //1/2^8 for this
		if(input2 == 36){ //1/2^8 for this
			crash();
		}
	}
}
```
```
(0,0) // F
(10, 68) // F
(45, 0) // T, F - this is “interesting”
(14, 0) //F
(45, 100) //T, F
(45, 36) //T, T
```

### Evolving an Input Corpus
Diagram:
Seed inputs ->fuzzer -> executes -> saves to corpus
Then, loop: fuzzer selects input from corpus -> mutates -> executes -> if interesting, saves to corpus

Discuss example and how we might generate inputs:

```c
void magic(byte input1, byte input2){
1.	if(input1 == 45){ //1/2^8 for this
2.		if(input2 == 36){ //1/2^8 for this
3. 			crash();
4.		}
5.	}
}
```
Start with seed input: 
```
(10, 50) //F
(45, 50) //T, F (“Favored”)
(45, 150)//T, F (do not save)
(45, 36) //T, T (Success!)
```

1. What makes an input “interesting”?
2. How do we determine which input to select from the corpus?
3. How do we determine how many bytes to mutate, and where?

### AFL - Design Goals and Fuzzing in 2013

What are the design goals of AFL?
1. Speed
	1. Rationale: Worst-case: don’t be worse than brute force! “If it’s 10x more likely to find a bug but 100x slower, then :(“
	2. Goal: Fuzz at native speed: if it takes 1ms to run the input without AFL, take same 1ms with AFL
2. Reliability
	1. Rationale: Other systems are brittle
	2. Goal: Avoid complex instrumentation, make minimal changes to binary
3. Simplicity
	1. Goal: Limit the number of knobs provided to users; try to avoid over-specializing to particular fuzzing targets
4. Chainability
	1. “Make it easy to interact with fuzzed applications” - make it easier to integrate with other applications - theoretically could use it to do API fuzzing (the fork vs non fork aspect?)

### AFL - Coverage Measurements + “Interesting” input detection
##### Tracking Coverage

*AFL Tracks Edge Coverage*
Basic blocks: A, B, C, D, E

Execution 1: A -> B -> C -> D -> E
Execution 2: A -> B -> D -> C -> E

```c
cur_location = <COMPILE_TIME_RANDOM>; //Set current location
shared_mem[cur_location ^ prev_location]++; //Increment counter for this edge
prev_location = cur_location >> 1; //Update previous location
```

Foreshadowing: There can be collisions in this map (can result in multiple branch transitions being represented by the same counter, which could cause you to miss the fact that you found something new!)

##### Detecting “interesting” behaviors
After running our input, we get some coverage information, like:
How do we determine to save an input for fuzzing?
1. New edges on coverage
Execution 1: A -> B -> C -> D -> E
Execution 2: A -> B -> D -> C -> E
(CE, BD, DC) are new edges!
Execution 3: A -> B -> D -> E (Not saved, no new edges covered)
2. New “coarse hit counts”
	1. Observation: If executed D->E 5x in Exec 1, and D->E 1000x in Exec 3
	2. For each edge, keep track of “hit buckets”: powers of 2
	3. 1, 2, 3, 4-7, 8-15, 16-31, 32-127, 128+
	4. Why is it necessary to have these coarse hit counts, and not save everything that has a higher (or different) count?
		1. More efficient (Can use bit-wise operators instead of integer arithmetic to determine if something is new)
		2. Avoid path explosion (Would save every input that iterates over a loop with a different number of executions)
Q: Do I save EVERY input that meets these criteria?
A: Do not save EVERY input that meets these criteria, apply heuristics to avoid saving slow inputs
Q: How do they avoid saving everything?
A: Inputs can timeout, timeout inputs are not saved. Timeout -> Took 5x longer than the first seed input took to run, rounded up to 20ms

### Evolving the input queue
Every interesting input is saved here, inputs are selected from start of queue, and then either fuzzed or skipped

Observation: some inputs might cover a superset of what others cover
A -> B -> C
A -> B -> C -> D

Observation: some inputs might be longer to run, or are just otherwise larger
A -> B -> C (1 ms)
A -> B -> C (5 ms)

What AFL does (we think):
1. Rank on speed
2. Favor faster inputs that cover same or superset

Q: Why do we want faster inputs?
A: because they run faster! Can get more trials
Q: Why do we want SMALLER inputs?
A: Just based on coverage, the algorithm might end up creating bigger and bigger inputs. Bigger inputs can take longer, are harder to debug, /and may have many bytes that don’t actually impact the outcome!/

AFL trims inputs at times by randomly zeroing out blocks, if the hit counts are the same, you succeed

AFL prioritizes “favored” inputs (based on ranking + coverage) when selecting 

### Mutation strategies
1. Deterministic bit flips 
2. Addition and subtraction of small integers (-35 to 35)
3. Swap integers for known interesting values (-1, 256, 1024, MAX_INT-1, MAX_INT)
4. Stacked random tweaks (do some random number of each of these in random places; # of operations is power-of-two between 1-64, block size is random, max 1kb):
	1. Single-bit flips
	2. Set interesting bytes
	3. Addition or subtraction of small integers
	4. Random single-byte sets
	5. Block deletion
	6. Block duplication
	7. Block overwrite
5.  Splice multiple files together

#### Dictionaries
Can get much better performance if you have a dictionary of special values for your application, like tags in an XML file. These are then used in mutation

### De-duping crashes
General problem - how do you report the # of crashes?

AFL’s implementation:
Crash trace includes a tuple not previously seen, or:
Crash trace is missing a tuple that was awlays present in earlier approaches

Probably over-counts crashes

### Running inputs (Fork server)
Classic problem with testing: setup cost for your app, including the execute system call, linking, libc initialization, etc. Do it once, then fork, and try to reuse that process for running new inputs unless/until it crashes


### General discussion/feedback
Cool to see how the JPEG example worked (come back to discuss when discuss SAGE)
(Ran out of time; didn't get through dictionaries, de-duping or fork server; didn't have much time to have discussion at end)
