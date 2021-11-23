---
layout: page
title: "Lecture 22 - Debugging Reinvented: Asking and Answering Why and Why Not Questions about Program Behavior"
parent: Lecture Notes
nav_order: 22

---
# Lecture 22 - Debugging Reinvented: Asking and Answering Why and Why Not Questions about Program Behavior
### Hot takes
Has this seen any more recent development or use? Seems useful in certain pedagogical settings, perhaps in conjunction with other techniques, but hard to scale to industrial software

Q: What is the state-of-the-art and state-of-the-practice tools for debugging?
* Examining coredumps
    * Reconstruct the program state before it crashed to examine it
* "Just put `printf` statements in your code"
    * Good: Very easy to do
    * Bad: "Kinda hard" - where do you put the print statements, then also remember to remove them; can't capture everything; multi-threading is not a good story...
    * (Multi-threading is going to be a general problem)
* Breakpoint debuggers
    * Still need to know where to put the breakpoint
    * Don't need to worry about leaving breakpoints in
        * UNLESS you write `debugger` statements in the code, and then distribute that, we don't understand why some languages offer this feature - "maybe" convenient
* Time travel debuggers
    * Allow the developer to go both forwards in execution (normal) and backwards in execution ("time travel")
    * Might make it easier to figure out where to put those breakpoints, because can wind back execution and put them in after-the-fact
    * Problems:
        * Hard to make work perfectly from technical perspective
        * Still need to figure out where to put breakpoints
* Slicing tools
    * Use a slicer on a statement that the developer thinks is buggy - and find out which code is related to that bug
        * "Garbage-in garbage-out"
        * What statement do I slice on?
* Taint-tracking-based tools
    * Same problems as slicing, but maybe different solutions.
* Test/input minimization/reduction
    * Goal: Produce a smaller crashing input that (theoretically) is easier to debug
* Fault localization
    * Goal: tell the developer the statements that are most likely to be causing the bug
        * Statistical analysis of test cases that pass and fail
        * Produce a ranked list on top-N performance - the true buggy line is within the top-1, top-3, top-5
* Other program understanding-adjacent tools that might help developers to form or test debugging hypotheses
    * Example: daikon

Q: How do you debug? Hypothesis generation + testing, or other?
* Follow the stack trace - if there is an error thrown, can work backwards to trace what caused that error
* If the output is wrong, follow output backwards through program, keep adding print statements as you go back
* Track dataflow - see how variables change through the program execution, comparing to your mental model of how those variables should change
* How does JB debug?
    * Start by brainstorming hypotheses
    * Test those hypotheses using debugging strategies
    * Keep a log
* Other experiences with multi-hour debug sessions?
    * YT: Debugging compilers is an enormous PITA because there aren't really any tools to help. Logs + luck are the keys
    * The easiest answer is to try to avoid having to have such a big debugging session by testing code in smaller units
* How do we teach debugging, and how do we conduct research on debugging to advance the state-of-the-art?
    * Explicit instruction in some programs on how to use debuggers - e.g. Java debugger (including [DrJava](http://www.drjava.org)), GDB, etc.
        * Mixed reception to this kind of instruction: debuggers (especially GDB) can be cumbersome, and you need a sufficiently complex bug to make it worthwhile
        * Teaching students how to set up a debugger is different from teaching student show to debug
        * Specialized tools should be brought in, too - like Valgrind
            * Jon once wrote a [hilariously simple yet challenging assignment at GMU](https://www.jonbell.net/gmu-cs-475-spring-2018/homework-1/) that auto-graded students using valgrind
        * [Effective Debugging: 66 Specific Ways to Debug Software and Systems Book](https://learning.oreilly.com/library/view/effective-debugging-66/9780134394909/)
    * Encourage students to copy/paste error into Google...
    * Implicit assumption that students will use print statements
    * Implicit instruction in some programs - when the Prof is demoing something and it goes wrong, and then they have to debug it in front of the class
    * "Nobody really taught us how to X, so why should we explicitly teach these students how to do X?"
        * For X in [Reading error messages, debugging, code navigation, emacs, bash, IDEs]
    * Does teaching debugging explicitly make students have a narrow view of how to debug?
        * If it's taught poorly, then yes :)
        * Having debugging come up in multiple courses, in multiple contexts, in multiple languages, with multiple instructors can provide a broader viewpoint - but you need to agree to teach it


### Whyline
Problems that WhyLine addresses:
* How to generate hypotheses for what is wrong (the tool suggests questions to ask)
* How to test those hypotheses (the tool provides answers to those questions)
* Specifically generates and answers questions of the format "Why did this" and "Why didn't this"
  The example in Section 2:
* Definitely a clear example for how a bug like the bug in the sample application could be debugged
* What would the output of the tool look like for more complex control flow?
* Q: Do you think that this technique would be feasible to extend as-is to other domains, not just GUI apps?
    * Hard to reason about what data to keep track of, might explode on other applications
    * In some other (potentially narrow) applications, this could also be quite useful - in particular, in JIT compilers
    * Need standard, templated patterns for the questions that will be answerable in the "why" and "why not" formats

### Evaluation
* What would you want this evaluation to show (independent of what it does or doesn't show)?
    * Show that this approach helps developers find bugs faster and to be more satisfied
        * Faster than what? Breakpoint debugging? Subject's choice? Delta debugging?
        * Expert developers or novice developers?
            * Expertise comes in multiple dimensions:
                * Familiar with debugging in general
                * Familiar with debugging in this language
                * Familiar with debugging in this codebase
                * Familiar with codebase but not with debugging
    * Does it need to apply to all bugs?
        * Maybe show that it is applicable to at least some subset of bugs that are "common"
* Is 9 people too small?
    * This is a reasonable size for this kind of study - if there's a methodological critique, it's probably on the backgrounds of the participants - expertise is probably a strong confounding factor here, so grouping the subjects by their expertise and having ideally paired samples (compare expert-expert performance or novice-novice performance) would be nice
    * Providing a user study for debugging was, at the time, somewhat groundbreaking
        * Prior work focused on numeric evaluations (e.g. slicing evaluations that say "N% of the application is included in the slice, and the buggy statement was one of them!" Which doesn't really get to "is it useful")
        * Usability is an enormously important aspect of debugging - and evaluating it (like this paper does) is key
        * Hard to establish a baseline - what were the users instructed to use, and what did they actually use?
* Are there other user studies of debugging?
* See also: [work from Brian Burg](http://brrian.org)

Q: What can we take away from Why Line in terms of: our own debugging techniques, teaching debugging, or what features new debugging tools should have?
* Lots of work focuses on giving developers *answers* but not helping them formulate *questions*
    * Hard to generalize question generation though - "It is clear that while the analyses used to answer questions may generalize, the analyses used to generate questions may not"
    * How would you generalize the question generation?
        * Or: how would you formalize a language that developers to ask questions? (And to reason about the answers)
        * Another way to cast this problem is: How do you help developers to add test cases to reveal buggy behavior? That is - in a scenario where your program fails under some non-automated case, and you want to create a test case that can automatically reveal the fault
* [Using Hypotheses as a Debugging Aid](https://arxiv.org/pdf/2005.13652.pdf)
