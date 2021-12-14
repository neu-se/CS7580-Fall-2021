---
layout: page
title: "Lecture 24 - JUSTGen: Effective Test Generation for Unspecified JNI Behaviors on JVMs"
parent: Lecture Notes
nav_order: 24

---
# Lecture 24 - JUSTGen: Effective Test Generation for Unspecified JNI Behaviors on JVMs

# Lecture 25 - GenProg: A Generic Method for Automatic Software Repair
### Hot Takes
* Not clear from the paper *why* this is the right approach to generate patches
    * "It's a large search space"
    * By the construction of the search space, need to use a meta heuristic technique like GP - but still may have some important design decisions that are not evaluated
* Would this approach do WORSE on smaller programs, where it's less likely that you can "find" the fix in the code?
* What about a baseline comparison to static bounds check analysis?

### Construction of the problem space
* Q: what is the problem that GenProg aims to address?
    * "How to automatically* fix+ bugs#"
    * What is automated about the problem?
        * Developer need not create the patch, and need not minimize those patches
        * Developer must provide an oracle, in the form of test cases. Functionality that doesn't have a test case is likely to be removed. ("Beyonce Rule")
    * What kinds of fixes do they consider valid?
        * Fix is defined as something that passes all of the test cases
        * Do not consider fixes that require creating *entirely new* code - only consider fixes that remove statements or copy statements. Do not consider changes to expressions. Do not consider creating new code.
        * Fix must be in code that is executed by the failing test case
    * What kinds of bugs do they target?
        * Evaluation targets "eight types" of bugs: infinite loop, seg fault, remote heap buffer overflow, DoS, stack overflow, integer overflow, format string vulnerabilities
        * "Generic" approach though
        * Bug might need to be localizable to a single location, be a deterministic failure

### Why is this hard to do?
* "It's a large search space"

Q: What was fault localization at the time?
* Seminal work: Jim Jones "Tarantula" - "spectrum-based fault localization" (SBFL)
* SBFL tries to solve this problem: "Given a set of passing tests, and a set of failing tests (which reveal a bug), can we automatically determine which line of code has the bug on it?"
    * Approach: Find all of the statements covered by failings tests, find all of the statements covered by non-failing tests; highlight statements to indicate which are covered more by failing or more by passing test cases
    * Intuition: If there are statements that are executed only by failing test, maybe likely that the bug is there

### Approach
Q: what is the solution to "the search space is really big"?
1. Operate at the statement level instead of expression level
2. Transplant code rather than generate code
3. Localizes the likely fix location to be within code that is executed more by the failing test case

Q: What do we think about these heuristics?
* "Statement level" - is this necessary to avoid exploring unimportant changes, or: does this serve more to simplify implementation?
* Given AST representation, why no AST-level mutations/crossover?
* Other approach - 2009: "ASSURE: Automatic Software Self-healing Using REscue points" - Find code that you think is likely to be handling similar errors elsewhere, transplant that code
* Heuristics are hard here because it is difficult to measure incremental progress towards a repair
    * Fitness function in GenProg is a weighted sum of passing test cases and failing test cases

Representation of program as AST + Weighted Path
* Weighted path is a sequence of (Statement, weight)
* Each statement in program appears exactly once, even if occurs many times on path
* Order of statements in weighted path is order that they were first encountered when executing the original program
* Q: Why does the order matter?
    * For crossover? But need to re-generate weights anyway for each variant

Selection & Genetic operators
* "Program variant" - individuals in this population
* Delete statements, add statement from elsewhere in program
* Crossover: Every surviving variant in population undergoes crossover (?)
    * Take population, pick half of it, do cross-over with that half and the other half (?)
* Selection: stochastic universal sampling is parameterized, compare to EvoSuite' selection strategy. On larger test suites (more test cases) a different selection strategy might do better to discriminate between variants that are better - look back to EvoSuite's approach

Fitness function
* Some of the tests might be harder to pass than others, so weighting all of the tests equally may not do a great job of representing the fitness of a patch. "Fitness sharing" is a common GP approach to identify and boost fitness for cases like that
* As-is, the fitness function might be most useful for preventing "nearly right" variants from becoming "very wrong" than for helping the "nearly right" get all of the way

Repair minimization
* As we talked about with EvoSuite, bloat becomes a problem
    * "Delta debugging as an evolutionary algorithm"
    * Delta debugging guarantees one-minimal patch, but might still have lots of redundancies - the EA approach could do better (or, could not)


### Repair Descriptions

Are these "diverse" bugs?
* Most overflow and overflow related bugs boil down to a missing bounds check

Where did this set of bugs come from? Is this a fair evaluation set?
* Nice to see selection of some programs that are also used as fuzzer targets: since it would be nice to hook up a fuzzer to a repair tool
* Toy examples?
* Problematic to create the tests yourself?
    * Perhaps more problematic: such small test suites? What is the manual effort in selecting down this test suite compared to adding a bounds check by hand?
        * Why only use the subset of tests? What bad things could happen if we included more passing tests?
            * If you knew in advance where the fault was localized, you could take advantage of that (but presumably trust that this didn't happen)
            * 5.3 includes a discussion about the impact of test suite size
            * Some details about how the test suite was selected to enable replication/reproduction would be cool
* Able to repair all faults (all were pre-known bugs)
    * Would it be nice to compare the auto-generated patches to the developer-created patches?
    * Interesting experimental design: Take Defects4J, use GenProg to create a patch, then compare
