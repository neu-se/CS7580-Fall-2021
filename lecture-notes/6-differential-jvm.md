---
layout: page
title: Lecture 6 - Chen et al ICSE 2019 Deep Differential JVM Testing
parent: Lecture Notes
nav_order: 6
---
# Lesson 6 - Deep differential Testing of JVM Implementations

#### Hot takes?
Reminder: Differential testing does not guarantee to find all bugs - need to have at least one system provide the correct answer (or both systems provide different buggy answers)
Discussion: What makes this "deep"? Keep in mind: "Deep" is relative

#### What is the motivation for this paper?
Want some deeper testing of the JVM - more than syntactic checking?
Lots of fuzzing gets stopped at testing the entry-level parts of the JVM - like the parsing of class files, rather than the execution, JIT'ing, or verifying

Q: What kinds of bugs can we find?
Look for bugs that are triggered by Java programs with irregular control flow
May or may not find security bugs - unclear if the IBM J9 CVE was discovered by this process

#### Why is this hard?

1. Space of inputs that are syntactically valid is heavily constrained - quite likely to create invalid class files (can't execute the code you want to test)
2. Getting code coverage of the JVM itself is non-deterministic for the same input, because you might trigger garbage collection or JIT'ing
	1. Q: Are both challenges on the same scale? Consider...
		1. Option 1 - Maybe we want to try to collect coverage of those non-deterministic elements, can maybe try to build something that tracks that
		2. Option 2 - java -help -> -Xint (turn on interpreted mode, no JIT'ing)
		3. Option 3 - Run multiple trials, figure out which part of the code is the non-deterministic part

Existing approach, ClassFuzz tends to create lots of bytecode files that are invalid

Example java bytecode:
```java
static int difference(int a1, int a2){
	return 5 + (a1 > a2 ? A1 - a2 : a2 - a1);
}

Static void main(String[] args){
	System.out.println(difference(10, -5));
}
```

```java
static int difference(int a1, int a2)
BIPUSH 5
ILOAD 0 //a1
ILOAD 1 //a2
IF_ICMPGT L2
ILOAD 1 //a2
ILOAD 0 //a1
ISUB
GOTO L3
L2: //FRAME Stack [I], Local: [I, I]
ILOAD 0 //a1
ILOAD 1 //a2
ISUB
L3: //FRAME Stack [I,I], Locals: [I, I]
IADD
IRETURN
```

How do we accidentally create an invalid bytecode file?
1. Break these frames - push too much or too little onto stack
2. Insert instructions that are incompatible with the current stack, or make it become invalid
3. More broadly: break typechecking (to the extent that we want to test execution or JIT'ing of code)

Purely random mutation (coverage guided?) approach *still* found bugs though! `public abstract {};`  example was found by classfuzz

#### How do we determine that we found a bug?
Differential testing, followed by manual verification:
* Check specification
* Check with developers

### What is the proposed solution?

"LBC" - Live ByteCode mutation. Analyze the bytecode that was actually run, and then mutate that code based on what you saw going on.
Q: what's a better term for this?
A: Covered bytecode mutation?

"Live mutants" and "live methods" - mutants and methods that were covered by the workload


What are the mutations that they apply*
* Add some instructions to the code. Limited set of instructions - only control flow instructions (GOTO, TABLESWITCH, LOOKUPSWITCH, RETURN, THROW)
	* Note: no "if" statements are added
	* "HI"s "Hooking Instructions"
* Determine where to add the mutation
	* Rank each of the methods by their "potential" ratio of instructions/times selected already.
	* "Some methods need to be more frequently mutated than others because they are more complex"
		* Why???? - How is this justified? Is this just "put mutants everywhere?"
		* Would using code coverage instead help to guide where to put mutations?
* Insert the "hook" (mutant, aka one of those five instructions)
	* Not purely random decision of where to put that instruction
		* Perform a dataflow analysis of each method, find the def/use pairs, target that 
* Find a "target point" too

Implementation discussion:
1. How to choose the location to insert the mutant?
2. How to choose the target location for the mutant?
3.  Sort methods vs priority queue?
4. If you just randomly insert GOTOs, won't you break the type checker? Need a stack map frame!?!

How do they select an input to mutate?

"Seed coverage" - Metric that measure the number of instructions in the initial seed that are then covered by the mutants
Examples:
* Initial method: 100 instructions
* Mutant: 101 instructions; only covers first 50 -> seed coverage of 50%
* Mutant: 101 instructions, covers all 100 -> seed coverage of 100%

Q: Why does "seed coverage" make any sense - what is it measuring?
Guess: An input that has a typecheck error at the very start of the method will have lower seed coverage than one that has it at the end

Weird side effect: Jump targets are always BEFORE the inserted jump (mutation) - so probably better to insert the mutations at the end

Q: Why Markov Chain Monte Carlo sampling, and what does this result in?
A: ?

Q: How does this solve the problem of invalid class files?
* Still make tons of invalid class files. Maybe do a little better because of limited set of instructions we insert (might be easier to be valid???)
* Seed coverage might mean that there is more code in a method before hitting the invalid part

Q: How does this solve the problem of not being able to collect coverage of JVM
* Sidesteps entirely, not bother collecting coverage or trying to use it


## How is this evaluated?
3 baseline systems:
1. Classfuzz' - classfuzz, but with some difference that was not explained
2. CLRandom - which randomly changes control and data flow
3. CLGreedy - greedy search instead of Monte Carlo

Q: What does the evaluation actually run? Does it run programs, or just try to load their class files?

Q: How did they determine the number of iterations to run?

RQ1 - Invalid input rate
Note greedy selection has more valid inputs than Monte Carlo

Q: Since Classming found more bugs than CLGreedy, but CLGreedy had a higher valid input rate, what happened?
* A - Maybe greedy is finding slower inputs?
* A - Maybe greedy is overturned to a bad criteria
* A - Maybe classming got lucky? Ran 5 times, then took the suite with the highest number of distinctions. No reporting of variation.

Constrained mutant execution to 20 seconds - what happened in those 20 seconds?

Table 2 - difficult to read the sums? (The "Total" row adds all kinds of issues reported, and the parentheses count all crashes and verification diffs)

Q: How much closer does this really get us to finding those "deep" semantic bugs?
* Not that much closer. What could we do instead?
	* Have more seeds, with more variety
Q: If low invalid input rate is overall goal, could we do even better?
A: You could certainly define better mutators that are less likely to break the type checker - Insert multiple instructions as necessary, and use some randomness to still make invalid things
A: Definitely could do better with a wider variety of input files, too

How do we feel about the overall contribution?
* Still finds lots of bugs, that might potentially have a large impact, and did create a new tool/approach to find them
* What about more discussion of the differential element?
	* Nice paper to follow up with: [JUSTGen: Effective Test Generation for Unspecified JNI Behaviors on JVMs](https://ieeexplore.ieee.org/abstract/document/9402099)
