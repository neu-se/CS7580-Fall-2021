---
layout: page
title: Lecture 16 -  FuzzBench An Open Fuzzer Benchmarking Platform and Service
parent: Lecture Notes
nav_order: 16

---
# Lecture 16 - FuzzBench: An Open Fuzzer Benchmarking Platform and Service

* Continuous integration + regression testing for fuzzer development?
* Is this a more sustainable approach to fuzzer evaluation?
* Provides a better baseline than some of the papers that we've read - this lets us conclude "fuzzer X works better overall" rather than optimized for a small set of programs 

That said: is it possible/desirable to conclude: "fuzzer X works better overall"
* Counter-point: Requires that we trust that the choice of 20 programs is representative of all programs, and unclear if that is the case
    * Argument for generalizability: these programs have different input formats
* Point: In other cases, the selection of programs is made by the developers of the fuzzer, which is certainly going to be more biased than an independent set of researchers selecting the programs that you run your evaluation on
* Ideally - we want to understand which projects are more or less suitable to different fuzzers

What are different features of programs/their code that might influence the superior (or inferior) performance of a fuzzer on that project?
* How strict are the structural requirements of the input? Very hard to create valid XML inputs, but maybe easier to create an input if the valid space is "all ints"
    * Other examples - use of checksums embedded in an input
* Coding style
    * Recursive vs iterative calls
    * Switch statements vs if statements
* External dependencies? E.g. network/file IO
* Domain - is it a compiler vs an image/font parser?

What are the assumptions that are baked into FuzzBench?
* Each execution of the fuzzer must be deterministic wrt the input specified

OSSFuzz?
* Continuous fuzzing for open source projects
* Very positive element towards increasing the validity of results - the seeds + drivers for the fuzzers are developed by the developers of the TARGET PROJECTS, not by the developers of the fuzzers

### Metrics

Return to a classic: measure code coverage, or measure bugs found.
Q: why should we measure and report and rank on code coverage?
* Point: If we reported only bugs found, the results might not generalize, because bugs are probably not evenly distributed in the code.
* "If you cover more code, you are more likely to find more bugs" - if you do NOT cover some code, you can never find a bug in it. We do not have a ground truth of where all of the bugs are
* These fuzzers mostly use "no-crash" oracles, so it is certainly possible that an input that a fuzzer generates does exercise a bug, but does not detect that there was a bug, in the future we might develop an oracle/checker to determine that we did in fact execute a bug

Sidebar: What are sanitizers in this context?
* Examples: ThreadSanitizer (TSan), AddressSanitizer (ASan)
    * Dynamic analyses for detecting memory/thread errors, no guarantees of completeness, incur some performance penalty
    * Checked Coverage paper discussed the "implicit checks" that the JVM provides - these sanitizers are effectively implementations of some of those same implicit checks, but for C

Q: Is "repeatability" a metric that should be measured here - coverage repeatability and branch repeatability?
* Counterpoint: That's an enormous amount of information, unclear what we do with it. More fuzzers, more benchmarks, more bugs, how do we create a metric to help understand these results? What do we do with this result?


Q: Is this a research paper? How would you categorize and evaluate it? Novelty? Significance?
* "Already a standard approach"
* This is an empirical study - from our reading list, maybe most comparable to Mutants + Real faults paper by Just et al.
    * Just et al had a more in depth discussion of why different decisions were made in the study design - this paper doesn't have so much of that
* 10% of the paper is an ad for google compute engine :)
    * Original FuzzBench (before this paper) was only GCP, no support for local docker, so that's good at least
    * But, it's free.
        * For now
        * Maybe there is a value proposition - the internal cost of running this stuff is probably cheap; if there is strong support for fuzzing OSS at Google, then this surely contributes to that goal
        * There is gatekeeping - right now this might make large scale fuzzing evaluations more widely available (because it's free), but there is a gatekeeper to decide if they will run it for you or not
        * Alternative approach: gatekeeper is some conference organizers - see http://fuzzbench.com/blog/2021/04/22/special-issue/
        * Conference organizers might over-select novel handwaving for pragmatic engineering improvements that improve fuzzer performance more dramatically
        * Why do we even need the gatekeeper?
            * $$$


### Evaluation discussion

Q: Should we encourage authors of fuzzing papers to present experiment-level results (e.g. fig 3) or something that shows per-target performance? "To the best of our knowledge we are the first to show per-experiment results in aggregate"
* Critical difference diagrams might be hard to interpret. Fig 3 says: AFLPusPlus is NOT statistically significantly better than mopt, but is significantly better than libfuzzer
* WHAT you are fuzzing is an extremely important question to answer when interpreting fuzzing results - are we trying to write "general" fuzzers that work best overall, or care more about the distinctions between different targets?
* If you are trying to make a general purpose fuzzer, then this is good
* Why not categorize the benchmark targets by the kind of input, and then report results by those?
    * E.g. report experiment-level metrics on "all compilers" and "all parsers"
    * Could you cluster the benchmark targets by relative performance of each fuzzer and then extract what the features are that explain why those targets are clustered in performance?
* Argument for default generic configuration:
    * If you don't know what your programs do, maybe you want some "general best" configuration
    * Especially if you assume extremely naive + resource-strapped analysts who are applying these tools

Reproducible experiment
* Why is it important to show that FuzzBench results are reproducible?
    * Need to be able to have stable/robust results to the stochastic nature of fuzzing
* Is table 5 convincing that the results are reproducible?
    * Comparing experiment 1 & 2 - Honggfuzz's ranking changes, but not in a way that is statistically significant, so it's OK (InfoVis challenge - wouldn't it be nice if the table represented that?)
    * The rankings are shown at the per-experiment level
    * Do we care about whether the rankings vary (at a statistically significant level) on each target, perhaps more so than whether the rankings vary on the experiment-level results?
    * How do they calculate the scores?
        * Non-parametric statistical tests on rank orders: take all of the fuzzers on each target, and assign a rank, then you can average rank across those targets
* Many important research questions, but we've seen them in other papers - hving so many RQs might dilute the ability to provide depth of exploration for each target

Insights from the case studies?
* Nice to see some empirically-grounded exploration of configuration settings (e.g. AFL deterministic configuration)
* What about a fuzzing/hyper parameter tuning experiment on the fuzzers?
    * This has been done in other applications of genetic algorithms
* Interesting to see this as "CI for fuzzing development"
    * Maybe this fits in the general trend of "how do I test my AI project" - where traditional testing mechanisms fall short
    * Is software engineering adequately supporting the development of AI applications broadly, where fuzzing is an example of this?
