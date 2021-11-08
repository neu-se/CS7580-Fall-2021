---
layout: page
title: Lecture 17 - UniFuzz A Holistic and Pragmatic Metrics-Driven Platform for Evaluating Fuzzers
parent: Lecture Notes
nav_order: 17

---
# Lecture 17 - UniFuzz: A Holistic and Pragmatic Metrics-Driven Platform for Evaluating Fuzzers

![XKCD's hot take on futility of standards](https://imgs.xkcd.com/comics/standards.png)

* Compare to FuzzBench: maybe what we are most interested in learning is not "which fuzzer is better" but "where is one fuzzer better and why?"
* Per-project data is useful to see.. but wow this is a lot of data in the tables + graphs
    * 1-plot-per-target vs 1-plot-per-fuzzer
    * Maybe an interactive online companion in Tableau or Flourish would be a nice way to examine closer
    * Maybe this is a defense of FuzzBench :)
    * Relative ranks might be a more useful compression of the various factors into saying which is "best"
        * Most of the statistical tests are using rank anyway...
* Where is the artifact with the results?
* In general, how much of our page-limit goes to the evaluation vs presenting what we actually did and why
* It is, for sure, hard to draw conclusions about fuzzers in general from this

Q: "Usability" evaluation?
* What is it that we typically evaluate in a paper? "Does this power schedule result in more bugs detected than the default schedule"? - What is the standard-of-quality for the performance and ease-of-use and bug-proneness of research tools?
    * As we saw with the KLEE reflection paper - sometimes these performance/implementation issues can dramatically impact the overall conclusions/results from a paper
* What are the aspects of usability that might be more relevant for fuzzing applications?
    * AFL - works out of box on an application that takes a file as an input
    * JQF/Zest - need to write a generator and driver
    * Many other fuzzers - need to write some model of the system, grammar for input, driver for program, etc.
    * Is there additional cost during program evolution?

Q: What are the problems that these authors see with existing fuzzing evaluation methodologies, and they are trying to address with this work?
* Different fuzzers are evaluated on different targets, and might be biased in the selection of those targets. Those targets might also be too small
* Metrics are not suitable (e.g. not just "bugs found" or worse, "unique crashes")
* Re-use of results directly without rerunning them (different hardware for different fuzzers)
* No typical reporting of resource consumption


Q: Are these problems with fuzzing evaluation methodologies better solved with UniFuzz than by FuzzBench? Are they solved now? Are they still open problems?
* Can only solve the problem of "different benchmarks" for different tools when everyone actually agrees on it, not just another paper is published saying it's the best
* Maintenance burden of updating the targets, and the fuzzers
    * FuzzBench pushes this to the maintainers of the projects who benefit from OSSFuzz
* Ground truth on bug triggering is still very weak - stack hashing + ASan
    * FuzzBench pushes this to maintainers of the projects who respond to bug reports
    * Compare to: CSmith ('we reported X bugs and Y were patched, including Z patched quite urgently') or GLFuzz ('we found X reports, of which we found Y false positives, reported Z bugs...')

Q: FuzzBench or UniFuzz? Can we bring something from UniFuzz to FuzzBench? Or from FuzzBench to UniFuzz?
* UniFuzz's metrics are nice and thorough (but also overwhelming)
* FuzzBench is, maybe "more usable"
* Both works have a lot of strong opinions about what is right in an evaluation, without necessarily having a clear, reasoned argument to support those conclusions
    * FuzzBench showed some more experimental results to show why their number of trials made sense, or why their campaign length made sense...
    * Why is it possible to say "30 trials, 24 hours for all targets", but not "AFL++ is best on every target"?
* Seed selection:
    * UNIFuzz: "We found random seeds on the internet"
    * FuzzBench: "We tried with no seed, and then we tried with the seeds that were provided by the developers"


Q: What do we think about all of these metrics
* "Rare" bugs and "dangerous" bugs
    * Useful to know, but unsure what conclusion we can really draw
    * We define dangerous as "GDB says it's dangerous" ;)
    * Some bugs are certainly likely to be more impactful than others
        * Given multiple fuzzers with the same oracle, is even possible to design a fuzzer that will find more "important" bugs than the other?
        * Same argument as "are defects evenly distributed in the codebase"?
* Speed of finding bugs
    * Point: "We ran our enhanced AFL and AFL for 1 hour each, and we found a lot more bugs!"
        * More realistically: Fuzzing-as-a-service products like OSSFuzz run in a time-bound, so what you can find in that time-bound is probably useful
    * Counterpoint: Maybe the best thing to do is "run as many fuzzers as you can for as long as you can, and we don't understand why what happens happens"
    * Speed of finding bugs might vary significantly across programs
    * Alternative: "It doesn't matter how fast you find the bugs, as long as you find them all within the specified time bound"
    * Hard to report metrics for this, particularly with repeated trials
    * Also hard to measure this when you have a relatively small number of bugs, because there is limited information (bugs are sparse - same issues as reporting bug). Speed of FINDING BRANCH COVERAGE might be more interesting to look at
* Coverage
    * UniFuzz does not look at the "differential coverage" aspect, but this would be a nice thing to examine to understand if there is much diversity between the different fuzzers
    * Number of lines covered vs percentage of lines covered?
        * Figure 6 shows total lines
        * Figure 6 is mega misleading because every graph has a different axis - but using percentages would be impossible to read also
        * How else would we present this information in a way that is readable? (Table?)

Q: Why start with 35 fuzzers, then evaluate with 8?
* AND no AFL++!?!?
    * Maybe because they started in 2018, found CVE's in 2019, then the paper got bounced around?
    * For the scale of this research, fair that it takes 2-3 years
    * But because no AFL++: No LAF-INTEL, red queen, etc.
* What was the goal of the "usability" evaluation and what is its contribution?
    * Could maybe have done with just the 8?
    * Maybe they had a lot of problems using the tools and were frustrated, and here we are :)
        * "Fuzzing the fuzzers" paper?
        * Interesting to look through the issues, authors of fuzzing tools [often respond](https://github.com/unifuzz/supplementary_results/blob/master/issues_of_fuzzers.md) 
    * Now that they have the 35 fuzzers scripted in Docker, could you make a meta-fuzzer?
        * How many fuzzers are in FuzzBench?
            * ONLY 22 ;)
    * Some of the fuzzers require models (peach) - what to do about that for the targets?

Q: Is there some large number of projects at which we can say that we have a generalizable conclusion? Or: how do we control for bias in the selection of these projects, where my incentive as the benchmark developer is: "There should be a clear rank-order of fuzzer performance across all benchmarks?"
* Is there a Friedman test here that says "There is no clear winner?"
* Maybe they could have done this for any/all of the metrics that were included
* We still like to see the different performance across different targets, and to highlight this instead of ignore it
* Could maybe come up with something like "We have N benchmarks and we were in the top-M ranks for all of the benchmarks"

Comparing FuzzBench + UniFuzz:
* So much compression looking at the aggregated results

Figure 5: Correlation coefficient between number of unique bugs and coverage
* Why are there so many omissions? (No bugs found?)
* Overall conclusion? 
* Bug sparsity is a significant threat to validity of any conclusion/correlation here


What do we do to evaluate fuzzers now?
* How do we adequately compress this information into a paper?
* OSSFuzz has "an ideal integration award" for fuzz targets - they reward developers for the first integration of their tools into their fuzz harness
* If only we had a technique for adding faults to a program that are a valid substitute for real faults :)
