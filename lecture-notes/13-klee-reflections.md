---
layout: page
title: Lecture 13 - On the Techniques We Create, the Tools We Build, and Their Misalignments, a Study of KLEE
parent: Lecture Notes
nav_order: 13

---
# Lecture 13 - On the Techniques We Create, the Tools We Build, and Their Misalignments: a Study of KLEE

### General discussion
* The full details of their evaluation of each paper is very revealing
* Question: Software engineering papers in general, is this the normal format? (10 pages, no appendix?) Other fields might say... 9 pages, plus appendices
    * Unfortunately, it's what we do... but maybe getting better wrt artifact evaluation
* 2 key issues in this paper: 1) study of KLEE's evolution, 2) discussion of incentives in peer review systems
* If KLEE were an industry-built tool, maybe it would be engineered and maintained differently - and with better guidance for parameter tuning
* Compare to development and maintenance of Daikon - but: where does the money come from to allow that?

## Scientific discovery vs engineering
Q: How and where do we draw the line between something that we can sell as "novel" and "newly discovered" vs engineering? Is it possible to have novel engineering insights?

### What were the engineering bugs/fixes that they addressed in this work?

* Re-computation optimization
    * KLEE invokes solver up to twice for each constraint set: once to see if the path is SAT, then if it is SAT + will cause a crash, it invokes the solver again to solve the constraints and generate a concrete input
    * Only in the first call would KLEE do factoring + minimization of constraints before looking in the cache
    * Is this just a bug? Can anyone argue that this is something novel?
        * Probably hard to make a case for something besides engineering
        * Sidebar: Maven performance profiling + ISSTA experience report
* Quick cache bug
    * You find a branch, you check the first edge to see if it's SAT, if it's not a cache hit on the first edge, then the second edge is NEVER checked in the cache, and goes straight to solver
    * This bug is still not "fixed"
    * Q: Is this one such an obvious bug compared to the other ones?
        * Maybe this is less of a bug, and more of an optimization?
        * "On the design and optimization of novel caching structures for SMT solvers in dynamic symbolic execution"
        * Maybe more subtle than "add a cache" - maybe there is something here that we could call "novel" - a novel experimental design to find how to place caches in a DSE tool. Is this just "good engineering practices" or is it something novel?
* Extremely interesting timing: look at [the issue in 3.1.6](https://github.com/klee/klee/pull/229) that seems like a straightforward optimization (with no downside). There were no comments on this from anyone since 2015, but a ping by someone on the issue just before our class meeting today. Coincidences? :)
 
* Q: could I have a paper where I implement some "straightforward" engineering techniques, but then have novelty in the evaluation of those techniques?
    * Yes
    * Would need to have some thorough evaluation strategy that also included a comparison to some other complex techniques
    * An important aspect is also: How do we understand *why* the performance is what it is?
  
* Q: how do we encourage adoption of better benchmarks for evaluation?
    * Start off with some large scale study that shows how existing research tools performed on old benchmarks, then this new one in comparison
        * How do you argue/prove that the new one is better, if not this?
            * Here are the goals of my benchmark evaluation: it should be a "fair" benchmark with "detectable" bugs
            * My benchmark satisfies these goals for these reasons, QED
            * How do you quantify these goals? Alternatively: how do you quantify that these goals are NOT met
            * What about using the kind of methodology from the Mutants/coverage paper to look at correlation, etc.?
    * What is our ideal baseline for evaluating fuzzing/DSE papers?
        * Gold standard: "We performed a large-scale industrial case study, where some engineers were provided the tool, others will not, and we measured developer productivity and defect detection"
            * "Oh, but what work worked for AnonCo surely won't work at anywhere else"
        * Alternative: large-scale case study on open-source code
            * But even OSS is different than industrial code - and OSS processes are different, too!
            * How do you pick the evaluation subjects, though?
                * "Development efforts that are representative of the space of current, typical practices"
            * What metrics should I measure and compare on these OSS projects to determine that one fuzzer or fuzzer enhancement is better than another?
                * It's tricky - different projects will be different. Some will have lots of defects, others fewer. Not all defects are the same. How do we quantify productivity?
                * Could look at defects
                    * (Entire discussion of "what is a defect" - a bug, a crash, a unique failure)
                * UniFuzz has a discussion of other metrics that might be useful:
                    * What is the overhead imposed by a fuzzer? (Consider - higher fidelity guidance w/ higher overhead)
                    * What is the throughput of a fuzzer?
                    * Bug finding stability (do you find the same bug each time you run fuzzer)
                    * Rareness of bugs that you find, and severity of bugs that you find
                * Coverage of the code by the fuzzer
                    * What do you measure coverage of: the actual code that the developer wrote, or the code that the developer wrote PLUS the library code that is executed by the developer's code?
                        * Argument against library code: If you use just one line of a very big library, it will expand the denominator and then you won't see a difference in the overall coverage metric.
                        * Argument for using library code: I can just report the numerator (# lines covered) - NOT the # lines that I think could be covered. Can still conclude that one fuzzer covers M lines of code, another covers N.
                        * Realistically: if you run the evaluation and find that one fuzzer is superior on "all lines covered" and the other is superior on "lines of the application code covered", which do we pick to report?
                        * Or... abstract away that library code into something like a model/state machine, consider coverage of that model
        * Even if you create such an ideal benchmark, wouldn't you need to update it?
            * Maybe we'll come back for FuzzBench (and UniFuzz?) soon
        * What about creating some sort of self-sustaining, additive benchmark suite. Or: take CVE database (or exploit DB whatever) and create a replication suite
            * Argument against a suite of known bugs: This will bias your new tool development to tools that are like existing tools because they can find the bugs that you can already find
                * "In our evaluation, including on the extremely well tested GNU Coreutils suite, MyFuzz found all 10 known existing bugs, PLUS give additional, never before detected bugs that shockingly have been in the codebase for over 25 years"
            * Maybe a problem with this kind of benchmark is that eventually your benchmark contains a different number of targets with different qualities than others, and everyone always wants to see a single "average performance" number (both reviewers, and readers, and maybe authors)
            * Let's not get frozen in this uncertainty, though...
    
#### Back to this paper
* Hypothesis: There is a research paper here that contributes all of the same engineering improvements to KLEE
    * Research questions like:
        * What is the impact of cache placement on overall DSE performance?
        * How do we determine when a cache in DSE is productive or not?
        * How do we automatically tune our DSE systems on different targets?
        * With "general" insights into development and performance of these systems. The techniques themselves aren't new, but provides the kind of principled scientific investigation into how and why to improve these systems
    * Would you read that paper?
        * Not the most interesting for everyone, but probably there is an audience!

### But back to the actual study performed
* Rubric for evaluating the papers:
    * How does a paper rely on timing?
        * Time reported ("We ran our evaluation, took 3 days")
        * Timeout used ('we ran our evaluation for up to 1 hour")
        * Time compared between works ("Our tool finished in 30 minutes and found the same bugs that another found in 3 hours")
    * How do they measure the effect of improvement?
        * Unaffected
            * "Time reported" kinds of papers <-
        * Positively affected
            * "Timeout used" kinds of papers
        * Negatively affected
            * "Time compared" kinds of papers
* Q: Is it really sound to determine the effect of improvement without actually running it?
    * Modeling physical architecture (esp caching, etc) is really tricky - differing machine speed and architectural improvements might be a confound, but unsure how big of an impact
    * Seems like the best that could be done given the circumstances
* Q: Could there have been a better methodology for selecting the papers to include in the study?
    * Detecting which papers actually build on KLEE is non-trivial and requires some actual reading
    * Maybe this is part of the point of the authors: we should have an infrastructure to keep track of who built what on top of which tool and what the result was, etc.
* Q: What was "Robust" in figure 3 and the 15 papers that went into that category?
    * Papers whose evaluation might be "robust" wrt changes to Klee?
