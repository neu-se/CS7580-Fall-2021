---
layout: page
title: Lecture 12 - KLEE Unassisted and Automatic Generation of High-Coverage Tests for Complex Systems Programs
parent: Lecture Notes
nav_order: 12

---
# Lesson 12 - KLEE: Unassisted and Automatic Generation of High-Coverage Tests for Complex Systems Programs

## What is the problem that is being solved here?
* As the authors see it: It's hard to write high-coverage test suites, and it's hard to find bugs in complex systems code - can we automate this?
* Performing symbolic execution of system programs is particularly hard because:
    * What to do about environmental interactions - OS, network, or user?
    * Exponential number of paths ("path explosion")

Q: why does symbolic execution have this "environment problem" but fuzzing does not?
* Fuzzing -> generate random inputs
    * Actually execute things
* Symbolic execution -> need constraints on the program execution
    * If we are not executing code, don't know what the return value of these things will be
    * Default is to under-constrain the return values, which might lead to infeasible executions

Fuzzing vs symbolic execution:
* Fuzzing:
    * Generate inputs fast, run them fast
    * Generate inputs with limited guidance
    * "If your tool makes it 20% more likely to find a bug but runs 100x slower, then is it a good tool?"
* Symbolic execution:
    * Generate inputs slow, run them slow
    * If you can perfectly model the program, you can always generate inputs to reach "hard branches"
    * Can provide guarantees based on the paths that it has visited as to whether some specific path is feasible


* Developers who write a program might be biased in writing the tests for that program, because they have assumptions about things that "they definitely got right," but those assumptions might cause them to miss important test cases
* Developer time is finite and expensive - can we make developers more productive by automating testing?
* Can also perform fault injection with this approach, which might help to find more bugs (although you could also do this with developer tests)
* Comes down to culture at some point - different organizations will have different cultures

Q: are Coreutils really "well tested?"
* Other research at the point was focused on Coreutils?
* Random testing was evaluated on Coreutils

Q: If these vulnerabilities existed for 15 years in GNU CoreUtils, are we solving the problem of buffer overflows, or are we helping attackers?
* These are "neutral tools" - with great power comes great responsibility, do good not evil with fuzzing
* Maybe people WERE impacted by these bugs but never reported them (like is described in the original Fuzz paper)
* If this research weren't being done publicly, then a well-funded adversary (or a "nation state") would probably still be doing this on their own, so it's good that we are doing something out in the open
* Maybe we should be building safer tools instead of more fuzzers :)

Q: What is a fair baseline for this evaluation - is developer tests fair?
* No existing symbolic execution system would scale to these programs, so can't compare
* AFL didn't exist yet

### Timeline of some relevant papers:
(The order we read != the order of discovery/publication)
* DART 2004 ("Directed Automated Random Testing")
* CUTE 2005
* EXE 2006
* KLEE 2008
* Checked Coverage 2011
* AFL 2013

### What is their solution?
Performing symbolic execution of system programs is particularly hard because:
* What to do about environmental interactions - OS, network, or user?
* Exponential number of paths ("path explosion")

What does symbolic execution do?
1. Symbolicate arguments and any file
2. Run program symbolically without any constraints
3. When you hit a branch, see what directions are feasible
4. For all feasible directions: pick a feasible direction, execute, add constraints to this path
5. At each dangerous operation, add branch to see if any allowable value would cause an error
6. If error, generate concrete values for those inputs

KLEE is an interpreter for LLVM assembly

#### How does KLEE avoid path explosion/fork bombing?

##### Minimize overhead of representing different states (3.2)
If we have to explore N different paths, can we avoid having N*(size of program in memory) memory usage?

Solution: Object-level copy-on-write

Contrast: EXE which used OS processes for each state, CoW happens at page level, have to pay some cost to call fork, but they don't pay that with KLEE because never call fork
(This is not evaluated)

##### "Optimize" the state scheduling (3.4)
Two approaches to choosing which state to look at next:
1. Random path selection
    1. This supposedly solves starvation/fork bombing
    2. BFS vs DFS:
        1. Breadth-first: The states that I examine will have fewer constraints
        2. Depth-first: The states that I examine will have increasingly more constraints
    3. Start at the root of the tree, and each time that you reach a node in the tree with a branch that is not yet explored, with a 50% probability, explore that branch. Do not visit states sequentially - visit them randomly
2. Coverage-optimized path selection
    1. Target branches that are not yet covered, because coverage is the metric that they report
    2.
Each time that KLEE picks a new state to explore, it will choose one of these strategies in a round-robin fashion

Q: Does this "solve" path explosion?
* Prioritizes shallow paths instead of deep paths (is this a good thing?)
* Could you still get stuck in, for example, a shallow loop with a symbolic condition?

Consider the following example:
```java
boolean checkPassword(String s1){
	for(int i = 0; i < s1.length; I++){ //Assume that S1's length is concrete and not symbolic
		if(s1.charAt(i) != expectedChar[i]){
			return false;
		}
	}
	return true;
}
```
The longest path is the most interesting one, and the one that a random fuzzer will have the hardest time to find - but for this code example, the random selection heuristic will be biased against it, hopefully the coverage-optimized search will get us there.

Does it impact the heuristics if loop executions are dependent on prior executions?
```java
Boolean isSorted(int[] in){
	for(int i = 1; I < in.length; I++){
		if(in[i] > in[I-1])
			return false;
	}
	return true;
}
```

Perhaps biggest impact on avoiding path explosion:
How does the interpreter work (3.1).

Q: How do they represent memory as a whole? Is it one big address space?
* KLEE maps every memory object to a finite-sized array
* This makes it faster, but also prevents symbolic-sized arrays
* Evaluation found that each pointer typically only refers to a single object (of concrete length)

##### Query optimizations (3.3)
Simplifies queries, uses a cache

Example: in cache, I have constraints `i < j, j < 20, k > 0`
I have a new execution that I want to solve, with constraints: `i + 10 < j + 10; j * 1 < 20; k > 0 + 0` -> simplify to exactly what I have in the cache

Q: How does this contribution relate to the rest of KLEE?
* Seems cool, sounds hard
* Includes some empirical evaluation demonstrating the importance of those optimizations

### Other big problem: Environment (4)
Q: how do they solve the problem of these environmental inputs?
* Model the system calls - 40 system calls, 2500 LoC
* Effectively write mocks for those system calls
* Creates a "Hybrid between an OS for a symbolic process and an interpreter"

Q: How else could you solve this?
* Concretely execute these environmental functions - when you want to read from a file or network, actually do that - Approach taken by "concolic" execution
* Could create these mocks/models at a different level, for instance instead of system calls, model libc

#### What are the inherent limitations to this approach?
1. Can only find functional bugs if there are assertions that would detect those (or you do differential testing)
2. Memory objects (e.g. arrays) must have a concrete size
3. No symbolic floating point
4. No support for "long jump"
5. No support for threads
6. No support assembly code
7. No support for the rest of the system calls that are not modeled
8. Inherit limitations of/from the constraint solver - solver will struggle with some things, so will Klee


Q: On paths, how do we think it would do compared to AFL?
* Cut to [AFLFast Journal Version](https://mboehme.github.io/paper/TSE18.pdf) that includes comparison with KLEE (section 6.3). Summary is that AFLFast finds more bugs, although sometimes Klee got them faster (although still found in less than an hour). In branch coverage, AFLFast got more than Klee when run for an hour
* Hmm, start talking more about evaluation methodologies for fuzzers...
