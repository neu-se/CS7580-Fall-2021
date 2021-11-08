---
layout: page
title: Lecture 15 - Semantic Fuzzing with Zest
parent: Lecture Notes
nav_order: 15

---
# Lecture 15 - Semantic Fuzzing with Zest
### Background material

Syntactic vs semantic fuzzing?
* Want a "deeper" fuzz - AFL and other CGF produce "a bunch" of inputs that "barely" get past any validation/parsing
* This is a problem that might be particular to applications that require well-formed inputs, perhaps like XML
    * What about "pulling JPEGs out of thin air"?
    * Maybe XMLs are much more complex; there are some constraints on JPEG, but at some point it's just a bunch of bits
* In order to make AFL work in a "highly constrained" input space, you would need to have a very good seed pool
    * What seed(s) might I want to use if I wanted to create valid JS?
        * This paper: the entirety of the React library
        * Maybe better to use lots of small inputs (?)
* Is there a reason we would ever want to fuzz a syntactic parser?
    * If only fuzz semantic side, we won't be checking the syntactic side...
    * Most of the dangerous buffer overflows etc might happen early on in the parsing, so perhaps this is good for x86 - but we are in Java
* What is an example of a syntactic vs semantic bug?

```
<projector>
<build>

</build>
</project>
```

^^ If this input crashes Maven, maybe we call it a "syntactic" bug

```
<project>
<build>
<plugins>
<plugin>
<groupID>someBadGroupID</groupID>
<artifact></artifact>
</plugin>
</plugins>
</build>
</project>
```	

^^ If this input crashes Maven after the file is parsed, but later, when Maven tries to actually load this plugin, it is maybe a semantic bug

* How would you evaluate a given bug and say that it is a "semantic" bug vs "syntactic"
	* Syntactic -> error based on grammar?
	* It is an arbitrary decision that is made, in this case it's defined based on which are the "semantic analysis classes" vs the "syntactic analysis classes"
	* In other forms of structured fuzzing, you will find that if you are sampling only or primarily from structurally valid inputs, you will miss any bugs that come from processing structurally invalid inputs. Might also be interested to look at "almost structurally valid" inputs 
* Maybe try to run BOTH AFL+Zest as complementary approaches, where you consider how they work cumulatively/in aggregate

"Often, it is necessary to run CGF tools for hours or days on end in order to find non-trivial bugs, making them impractical for use in a continuous integration testing"
* What do the authors mean here by "in CI" - "in CI for every build"? Or "in CI for every nightly build" or "in CI for every weekly build?"
* Counterpoint: Only find bug 1/20 times?
* Point: People DO use QuickCheck itself in CI (as far as we can tell, maybe not JUnit-Quickcheck, but yes for Scala, probably for Haskell and others)
* What would you want to see to convince you to devote CI build time to a fuzzer?
	* Wasted human time sorting through false positives - how many potential bugs do I have to sift through
	* Some metric of overall efficiency: how do you quantify bugs found per CPU-hours spent (and the latency on shipping a new release)
	* Want to make sure that it is deterministic + want to do regression fuzzing (don't waste time fuzzing things that didn't change)
	* Maybe need to run some kind of trial with your target application to understand the actual efficacy
	* What about "Infer" - static analysis tool from Facebook that is run in CI, compare to this?
	* Maybe an ideal experimental design if want to claim relevance for CI - do a retrospective analysis, running fuzzer on old revisions of code

### Overall approach

What is the novelty proposed?
* Track coverage of valid inputs, bias generation to create valid inputs to get "deeper" in to the semantic parser

Overall approach: consume random bytes, and use those bytes to make decisions 

Syntactically valid, semantically invalid:

```
<xml>
<foo>
</foo>
</xml>
```

Q: Under what scenario do you increase the valid coverage, but not increase the total coverage?
* Some input is not valid, put exercises some parsing code that has never been executed by a valid input. Some future input is valid, but exercises the same parsing code
* So, does what the original input did, but now is valid

Q: Are there other strategies to bias generation to valid inputs?
* Upweight those inputs - give these inputs "more power" in a power schedule scheme like AFLFast's
	* (Zest does this in practice, not sure if it's described in the paper)
* Use some other coverage metric - if you put more detailed coverage probes in the parts of the code that you are more interested in, you will be able to track more progress in those code areas, and then will find more "interesting" inputs in that space

Q: Could biasing towards semantically valid inputs hurt us in a fuzzing campaign, and make it harder to find the input that we need to find some hidden bug?
* Give up on finding syntactic bugs
* Might be less sensitive to behavioral differences in syntactic part of code, which could be correlated to behavioral differences in semantic part of code, and we would miss saving that "new" syntactically interesting input

Q: how do I know that I did a good job writing my generator?
* It found bugs! (But sometimes it didn't) I didn't find all of the bugs, but I found some, and some people cared about them.
* Hypothesis: If your generators are not great (in terms of expressivity) maybe the performance in terms of coverage + bugs will plateau very quickly

### Evaluation

How do they report coverage?
* Report primarily on relative performance (and not absolute), but having different axes on each graph is weird...
	* Counterpoint - how do we distinguish the difference :( 
* Percentage of branches covered vs absolute branches covered?
	* Percentage tells a bit more, because you know how much was missed
		* Another metric: numerator is branches covered, denominator is all branches in files that have at least one branch covered
		* KLEE's coverage denominator removed "dead code" and "uncalled functions" - this is useful for relative comparison, but then at the end if you claim "100% coverage" it is somewhat concerning...

Q: On XML (maven _ ant) AFL dominates QuickCheck, on JS (Closure + Rhino) QuickCheck dominates AFL, on BCEL, AFL dominates QuickCheck
* The authors' interpretation:
	* POM file validity is hard to satisfy (?)
* What about the quality of the generators?
	* XML generator creates any valid XML, but is not aware to the schema. It knows what valid tags are, but doesn't know the nesting
	* JS generator knows a basic JS grammar
	* BCEL generator knows Java bytecode instructions and their format but no grammar 
* What about the quality of the seeds?
	* Probably an important confounding factor... Klees et al [47] have some words on this...

Q: What would we like to see as a baseline for Zest? Are we satisfied with AFL + JUnit-QuickCheck?
* JUnit-QuickCheck might be a bit vacuous because it is something that they are building on - if your greybox fuzzer does not do better than the black box fuzzer, there is a big problem :) 
* For AFL - could we see a range of seed inputs (plus longer runs)
* Is the QuickCheck implementation the exact same that developers would use, or did they add instrumentation for coverage?
* Can we see total coverage over time? 

Q: What evaluation would support: Track coverage of valid inputs, bias generation to create valid inputs to get "deeper" in to the semantic parser
* Create a variant of Zest that does not use validity feedback, then compare Zest to that variant
* Look at real bugs, find out if they are "deeper"
* How could we measure "deeper"?
    * Look at length of path - easy for small programs, hard for big programs
    * Look at depth on CFG (folding in recursion)

If you ran AFL for 24 hours or longer, would it find more bugs?

Bug reliability:
* Is reliability a good thing?
    * Yes - if you run the tool on a modified version of the code, you might be more likely to find similar issues to what you found before
    * Yes - more confidence that a tool consistently finds bugs instead of "we got lucky"
    * No - Are we overfitting to the bugs that we found once? Is this a sign that you are searching only a relatively small part of the entire search space, which happens to have some (but not other) bugs
    * Can we draw some conclusion about the bugs themselves if they have high reliability and low MTTF?
* * * * * * * * Not all bugs are equal
