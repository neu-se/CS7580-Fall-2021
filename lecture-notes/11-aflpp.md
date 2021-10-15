---
layout: page
title: Lecture 11 - AFL++ Combining Incremental Steps of Fuzzing Research
parent: Lecture Notes
nav_order: 11

---
# Lesson 11 - AFL++: Combining Incremental Steps of Fuzzing Research

## What is the problem that is being solved with this work?
* Large number of developed techniques to improve fuzzing keeps growing, sometimes without fully functioning code
* Those papers are often compared to an old, not necessarily reasonable baseline (AFL)
* From an engineering perspective: AFL is not very modular, hard to develop new features to replace say, seed scheduling, coverage/ feedback etc

Q: Why AFL++ and not LibFuzzer++?
* AFL:LibFuzzer emac:vim ?
* Availability of patches for AFL, not LibFuzzer
* What about an empirical comparison of AFL/AFL++/LibFuzzer?

### Forkserver
To create a new process:
1. `fork` - creates a copy of the current process, makes memory copy on write
2. `execve` - Load a program binary into that memory, start running it

"Forkserver" - Create one process to fuzz, then copy it 

*Mopt*: An alternative to AFLFast that we didn't discuss; tunes selection of mutators

### LAF-Intel

Example:
`if(strcmp(input,"abcdefg")` - hard to pass this input. Rewrite to
`if(input[0] == 'a' && input[1] == 'b'..)`
Helps to break up these complex conditions so that the fuzzer can measure progress

Other work (not discussed in this seminar) also comes up with other approaches for incremental feedback to fuzzer on complex conditions

Q: Could LAF-Intel make fuzzer performance WORSE?
* Performance of the fuzzed application might be worse due to branch prediction misses (could be some constant overhead to each fuzzed input execution)
* Can cause an increase in the number of collisions in the coverage map, which could be a very bad thing (since it would prevent the fuzzer from distinguishing between coverage of different branches that it could have before)
* Might have different kinds of interactions with different kinds of coverage/feedback/scheduling approaches because we are changing that structure

### RedQueen

"I2S" - Input-to-state, when an input is directly used in a hard to cover branch edge

`if(input[15965] == 35) crash();`

1. Mutate the first half of the input, see if the branch condition still has same value
2. Repeat a search process, changing bytes in the input until you see the branch condition change

Q: Could RedQueen have worse performance on some programs compared to AFL without RedQueen?
* If you don't have any conditions like this that are hard to find/satisfy, and are directly controlled by the input, you'll just waste time trying to find them

*AFLSmart* - Use specifications for input protocols to create structured inputs that are more likely to pass validation. +5 points good name for Peach

#### Seed Scheduling
Everything that AFLFast has, plus additional schedules that consider:
1. Times seed is chosen from queue
2. Number of inputs with the same coverage
3. Average number of generated tests with same coverage in general

Q: Are these reasonable variables to include in a seed scheduler? Is there some intuition for why they might help avoid re-generating the same input many times?
* What is the difference between 2, 3? (Unclear)
* I guess we'll see in the evaluation!

#### Coverage-related implementation issues

2 key coverage problems for AFL:
1. Overflow of counters
2. Collision of counters

*NeverZero* - Add a "carry flag" to ensure it's never zero, works well for preventing wrap-around to zero
*SaturatedCounters* - Stop incrementing once you reach the maximum value

Q: Why might one of these work better than the other, and we don't just say we have a default?
* NeverZero could be better than SaturatedCounters in cases where everything becomes saturated (and all show 255), but NeverZero can possibly let you discriminate between different inputs that have different counts, at very low cost
* SaturatedCounters could be better in [unknown? Maybe some implementation-level differences, probably is slightly more expensive than regular coverage]

Other coverage approaches:
* Context-sensitive edge coverage (XOR's block ID with the block ID of the calling method)
* Ngram (XOR's with the N-1 previous blocks)

Usefulness of these approaches probably correlates with:
* Application-specific issues (is it necessary to have the calling context?)
* Collisions in the coverage map (alternatively: performance from having a much bigger coverage map w/o collisions)

*Persistent mode*
* Much faster than forking because there is no system call at all!
* Risk: If program is not stateless, we could have difficulty exploring all program states or reproducing crashes
* Interesting question: How to determine if it's OK to use persistent mode for a given app?

Note: current version of AFL++ has collision-free coverage option, too.

### Evaluation

*FuzzBench* - 21 Open source apps
What metrics do they use to compare the fuzzers? Coverage

Different configurations are better for different targets, maybe it's OK to judge this on coverage alone

What can we conclude from the coverage results?
* What was "the default AFL setup", were there configurations not presented in the paper that could/should have been default? 

Is the goal of "providing better defaults" attainable? Or is it dangerous?
* "AFL++Optimal" is optimized per-program, but not generalizable (yet, at least)
* Fuzzing configurations may be target-specific, but is providing multiple suggested configurations dangerous, to suggest that there are no other useful configurations?

FuzzBench paper looks a bit more in depth at the differences in configuration

Can we create manual or automated guidance to tune fuzzers as the yrun?

Scaling discussion
* "New code" is thread safe...
* Expensive to run a single evaluation, but more expensive to communicate

Impact?
* Most research has shifted to AFL++ from AFL
* Repo has similar stars/forks as AFL

