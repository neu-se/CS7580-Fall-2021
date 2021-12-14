---
layout: page
title: "Lecture 26 - An Analysis of Patch Plausibility and Correctness for Generate-and-Validate Patch Generation Systems"
parent: Lecture Notes
nav_order: 26

---
# Lecture 26 - An Analysis of Patch Plausibility and Correctness for Generate-and-Validate Patch Generation Systems
"Write no code, get no bugs"

### Key parts of the critique
* How do we define that a patch is "correct"?
    * Ask your magical oracle
    * The oracle will say one of two things: "correct" or "incorrect"
    * Is this part of the responsibility of the APR technique, or OK to leave as a separate black box?
    * GenProg oracle: we ran 3-5 passing tests
        * Note: the GenProg paper that is critiqued by this paper is not the TSE paper that we read, but a different one, which has much more of a description of what was done to select the bugs
    * What is the research contribution of RQ1?
        * "It is important that you have a strong test suite"
        * How do you define an adequate test suite, anyway?
        * Not a problem of GenProg as an approach, but a problem of the evaluation design
* Measuring scalability:
    * An important metric to measure, but there is not necessarily any kind of solution to this here

### Alternative solutions
* "Automatic Code Transfer"
    * What problem does this solve? "What if the fix doesn't exist in this program that we are repairing?"
    * Does the data presented in this paper argue that this is a problem?
        * Some other work examined this, but not in this paper
    * Makes the "search space" problem worse
* "Learning from successful patches"
    * What is the training data?
        * Need labeled dataset, a big one, manually labeled to say what each patch does. Like defects4j. But, hard to see how this would transfer across projects, so you have some cost to do this for every project that you would like to repair
        * Not quite "the bugs fix themselves"
* "Learned Invariants"
    * AKA "have a better test suite"
    * Probably an interaction between the search strategy and the way that fitness is calculated
        * Need to be able to slowly evolve things and make progress. Do invariants still give you that?
        * It is hard to have a search process where (some) small changes might create enormous progress, and doing a large change that includes that small change might hide that progress
* "Targeted patch generation"
    * These are useful for solving the specific problems that they solve - if you can recognize the bug as being an instance of one of these, then just use that, duh
    * "Pattern-based mutators" might be a useful alternative approach here, where some mutators are designed to be particularly effective to fix some particular defect patterns
* "Realistic expectations"
    * On whose part? :)

### Kali
Q: Does Kali's existence contradict the position that "patches that delete functionality aren't valuable patches?"
* "Time and effort"
* "The Kali patches often precisely pinpoint the exact line or lines of code to change. And they almost always provide insight into the defective functionality, the cause of the defect, and how to correct the defect."
    * Developer study?
* Analogue to the first fuzzing paper that we read: "Look at all of these really complex symbolic execution things, here's a random test generation that finds low hanging fruit!"
* Still has big scaling problems with test suite size: Evaluation showed "typically several hours" and "always less than seven hours:" running the entire test suite to evaluate the fitness of a variant is going to be a core scaling limitation, even if you are able to measure incremental progress (or explore exhaustively) you still can't really scale up
* Is there timing data that compares Kali to GenProg? Or does Kali take hours longer than GenProg?
    * Comparing GenProg paper (the ICSE paper) to Kali, Genprog on average 1.6 hours for successful patches, 11.22 hours for not successful patches - this is some high-level data that could be compared, but note:
        * No repeated trials for a stochastic process?
        * Varying hardware for evaluation?

### Where to go from here?
* AFL++ and FuzzBench for GenProg: some shared infrastructure for evaluation, plus some shared infrastructure for implementing your tools - fault localization, test suite evaluation, etc.
    * Could look to other benchmarks like... Defects4J - but if the approach is mostly succeeding on memory corruption bugs, then Java results might be pretty different
* Automated program repair scaled to extremely large codebases? "Extremely smart" ("machine" "learning"?) variant generation - mutators?
* Are there lessons about how we should structure our development of APR tools?
    * Your evaluation should answer the questions that you are interested in
    * How should we integrate that evaluation with our design process? If we evaluate often, can we get useful feedback on aspects of our design that do or do not work? Or, do we just overfit our design to the evaluation?
        * In this case, may be overfitting
        * Is it less problematic if you take all of those results, and make them available, too?
            * For all of the critique and comments, this is something that Angora's evaluation included
        * "Just make sure that your benchmark is generalizable" - we'll solve that one at the same time as "just make sure that your test suite is adequate" :)
    * Transparency in research software development (artifacts) can be useful here - "Sunlight as the best disinfectant" 
