---
layout: page
title: Lecture 14 - Finding and Understanding Bugs in C Compilers
parent: Lecture Notes
nav_order: 14

---
# Lecture 14 - Finding and Understanding Bugs in C Compilers


### Important background

* Deterministic behavior
* Implementation-dependent behavior: Might vary between compilers/architectures
    * Example: `sizeof(int)`
* Unspecified behavior: Might define a set of allowable behaviors, usually within some set of reason
    * Example: order of evaluation of arguments to a function call
* Undefined behavior: Could do anything - core dump, delete all of the files on your disk, produce empty output
    * Example: Invalid pointers

C: optimizations might cause differences from "normal" execution...
* The undefined behavior is a different behavior
* More complex optimizations -> might be more likely to have a bug in how we implement the optimization

## What is the problem being solved by CSmith?

Want to find bugs in C compilers

Two arguments for why we want to do this:
1. Why do the bugs matter?
2. Why are formal methods techniques not currently sufficient?

What makes it hard to find bugs in C compilers?
* Undefined behavior (Can't just do naive differential testing)
* Atypical code might be under-represented in manually-developed compiler test suites ("fixed test suites")

Why are these bugs important?
* You could verify a kernel, but with an unverified compiler, what's the point? The entire system needs to be carefully tested and verified
* These bugs might impact safety-critical systems
* We found 25 "release-blocking bugs"
* C is also a target language - so even if a developer wouldn't generate this code, a code generation tool might generate code that would reveal one of these bugs, and therefore, it's particularly important to check for bugs like this
    * This is, for sure, a real issue
* These bugs might come not only from optimizations, but just from regular code

Does the severity of the issue vary based on whether the bug manifests in a compiler crash, vs silently incorrect output? What about Figure 1?
* Maybe this code represents closely something that generated code would look like, even if no developer would write this code
* Signed vs unsigned comparisons are a general issue in C, maybe showing up in a more complex example than this minified one

What would constitute a reasonable effort to find these bugs? How much money is too much?
* 3 years? :)
* $1,000/bug - how do you get there?
    * $325,000 total, over 3 years
    * First paper that we read that gives a direct cost per-bug
    * How many hours of whose time might be a more interesting question to answer rather than "how little do you value your and your students' time"

What is the human effort to RUN CSmith?
* The 325 bugs are de-duplicated manually (resulting in this 325)
    * Found that the largest number of bugs were in largest programs, which would be hardest to debug
    * Compare the minimization strategy in CSmith (probably manual) to the GLFuzz automated minimization strategy
        * GLFuzz - had discrete mutation operators that were applied to some original code; work on different subsets to build a more minimal example
* Over these 3 years, how many times did you run this and for how long?
* Would the final version of CSmith find the same bugs?

What is the oracle for a bug?
* Use consensus between different compilers - if any one compiler produces different results, say that you have found a failure
* Did not find any examples where there was not a clear consensus in terms of what was "correct"

### What is the overall approach?

Differential testing

What are the kinds of bugs that they look for?
* Looking for "middle end" bugs - after lexing, parsing, optimization, but before code generation
* Non-goals:
    * Standards conformance
* Try to generate programs that "look" like real programs
    * Maybe easier to analyze
    * Maybe more representative
    * Maybe more likely to get adoption/interest from developers

What do they compare between program executions?
* Generate a checksum of the non-pointer global variables that are defined at the end of the execution
* Compare crashing vs not, termination vs not
* Does this capture local variables? Is all code being checked?

Q: Why not compare local variables, e.g. at the end of each function invocation, save all local variables somewhere?
* (Maybe something related to undefined behavior? Maybe just a goal to reduce amount of output that is compared)

How do they generate input programs?

Start with a grammar for a subset of C (e.g. dropping the language features not-yet supported, like strings, dynamic memory allocation, floating point, unions, recursion, or function pointers)
1. Pick an allowable production from the grammar
2. Generate that production, including any targets (likes variables) and types
3. If that's a non-terminal production then recurse
4. Handle dataflow transfer through each new production, keeping track of in-scope locals, globals, etc.
5. Perform safety checks, revert this entire production if unsafe

Again: why avoid undfeined behavior in the code that we generate?
* Avoid false positives
    * Hey: how many false positives did they even get?
* (Would you get false positives? - not evaluated)
    * Even if there are false positives - is it interesting to quantify the impact of undefined behavior on programs?
    * The input generation is only generating a subset of the input space already, which results in possibly missing some real bugs (while eliminating false positives), but it would certainly be nice to validate the number of false positives that you avoid, and the number of true positives that you lose
    * "This was good enough"

Q: What is novel about this approach?
* Effectively testing the "mid-end" of the compiler (and the way to do that)
* Generate programs that are:
    * Syntactically valid (pass type checking + parsed)
    * Avoid generating programs that have undefined behavior

Q: Why didn't anyone just use exactly this kind of approach for testing java?

Q: Do we have insights into how to create an approach that could generate the same kinds of bug-revealing programs, but programs that could compile faster, and run faster?
* End up generating very large programs (81KB, 8k-16k tokens)
* More statements -> more possibilities for interaction between statements -> more likely to trigger optimization bugs
* More statements -> more difficulty to minimize code, explain it, de-duplicate bugs, etc
* The probability table that specifies the probability of each grammar production being selected will have a significant influence on this - how were those probabilities defined? Was there tuning, or could there be tuning?

How do I determine when building a tool like this: should I mutate existing code, or generate new code?
* Is it hard to find the code? And: what to do about undefined behavior in that code? Also have to handle file IO, other things that might be distractions.
    * SPEC benchmarks are a common target here
    * Could also mutate existing developer tests - especially given the results in section 3.5 that show high correspondence between coverage of developer tests and coverage of generated inputs
        * How correlated is coverage with bug finding? :)
