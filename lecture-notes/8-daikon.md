---
layout: page
title: Lecture 8 - Ernst et al, ICSE '99 Dynamically Discovering Likely Program Invariants to Support Program Evolution
parent: Lecture Notes
nav_order: 8
---
# Lesson 8 - Dynamically Discovering Likely Program Invariants to Support Program Evolution

### What is the motivation?
We want to find to find likely program invariants to support program evolution

Q: why this problem?
* Targeted at developers - how to gain insights that might not be apparent by reading the source code
	* Example insight: loop numbers are always within a certain range
* "Likely" invariants are maybe useful to know to get started with writing out what the actual invariants are
* Maybe helpful for code that is not commented - the invariants might be useful for program understanding

Problem statement: Picking up some undocumented code (maybe your own) - not sure how to make changes, not sure of their impact

Static vs dynamic? (see a great discussion of this in [Mike Ernst's SIGSOFT Outstanding Researcher Award Talk](https://youtu.be/UIBeeWBKE3U?t=6379))
1. How do we validate specifications? (Static: Theorem proving, Dynamic: assertion statements)
2. How do we generate specifications? (Static: Abstract interpretation, Dynamic: This paper, Daikon)

Q: How do likely invariants support program evolution?

A: Documentation, While writing code - see what it does to understand better how different values relate to each other, pseudo-oracle: as you develop and change your program, see if the invariants change

A: What about a use-case where a developer says "just observe invariants around these particular variables" - maybe interesting/useful to compress lots of executions of a statement of code and present the invariants observed on those values, rather than printing out every value.

A: Can help find edge cases 

A: Interesting use-cases: refactoring

#### Legacy
People are using it - still - in practice, and in research

Lots of different frontends now (Java, C#, etc)

"Real" software - really works, supported

Used in research for a lot of different topics (see [Daikon publications index](https://plse.cs.washington.edu/daikon/pubs/) for a comprehensive list):
* Specification mining
* Program repair
* Detecting equivalent mutants
* Guiding fuzzing [The Use of Likely Invariants as Feedback for Fuzzers | USENIX](https://www.usenix.org/conference/usenixsecurity21/presentation/fioraldi) 
* Runtime property checking

#### What is an invariant?
Some property that a program will always hold throughout its execution
Example: a loop invariant is a property that is always true - before, during and after a loop execution 

Goal: Prove that our program is correct
Intermediate solution: Write down these invariants/properties, then get somethign to try to prove them
That's for invariants overall - but these are likely invariants


```c
I = 0;
S = 0;
N = size(b);
While (I != n){
	s = s + b[I];
	I = I + 1;
}
```
Pre-condition: `n>=0`
Post-condition: `s = Summation over all j from 0..n of b[j]`
Loop invariant: `0<=i<=n`, `s=summation over all j from 0 to I of b[j]`

### What are the challenges in detecting these invariants?
1. Can't guarantee we find true invariants, only likely (prone to false positives)
2. Can't guarantee we find all invariants (test suite is not comprehensive enough, or don't have enough templates to match everything)
3. Challenging to scale this - need to store lots of execution data

### How do we infer likely invariants?
* Generalize likely invariants from examples that we see

Example. Generate 100 random inputs with lengths distributed between 7-13, elements from -100 to 100.
```c
I = 0;
S = 0;
N = size(b);
While (I != n){
	s = s + b[I];
	I = I + 1;
}
```
Pre-condition: `n>=0`
Post-condition: `s = Summation over all j from 0..n of b[j]`
Loop invariant: `0<=i<=n`, `s=summation over all j from 0 to I of b[j]`

#### Implementation details
1. Create some dummy "derived variables" that are traced too - like original at method invocation, or sum of an array, length of an array
2. Match invariants based on template

Q: What kinds of invariants can we infer using the Daikon approach?

A: Infer only those that we have template for, and infer all of them based on that template

Only detect invariants that involve 1, 2 or 3 variables. Q: Why this restriction?

A: Total number of combinations of variables grows really fast. Picking up to 3 means that they need to consider all three-set combinations of in-scope variables.
	* Scaling problem - computationally
	* Scaling problem - wrt "interesting" invariants 
	* A somewhat arbitrary choice - based on intuition. Might also be able to derive larger invariants also by combining multiple 3-wise or pair invariants that are related. Implementation can report "compound relations"

Q: Why `x = a (mod b)` and not just `x = c`

A: If you have some starting value, and then it is incremented by a constant? (still unsure what these invaraints represent)

Q: What do we think of the likely invariants in figure 2 that are not part of the invariants provided by the source textbook, like bounds on `N` or `B`?

A: "These extra invariants are not merely artifacts of our technique, they provide valuable information about the data set, such as variable ranges" - maybe not useful when it's obvious that we are providing this particular input, but maybe useful elsewhere. A very nice observation, very well said. This is information that you CAN NOT get from a static tool, it's based on the dynamic execution, so if you find that information useful, this is how you need to get it.

### What do you think about this solution?

* It would be nice if there was null tracking - and e.g. invariants that something is or is not null. (Note - this has been done since the original paper that we read)
* Can you use SMT solvers to determine some of these relationships? 
* How to support developers who are reasoning about software evolution is an interesting problem, and not well-supported by tools.
* Running time: Cubic with in-scope variables, linear wrt number of tests, linear wrt size of program

##### What evaluation would you like to see today?

1. Ground truth evaluation on invariants: how many invariants do you detect compared to a ground truth set (like from a textbook) - With restriction that maybe we want a broader set of examples
2. User study: When given a development task that is representative of real tasks that real developers have, give real developers Daikon, give some of them no Daikon, then evaluate various quality attributes of the code that the developers produce

#### Overall discussion
* Interesting vs uninteresting invariants? What is ground truth for interesting, and can we tune something to get more that are interesting?
* How else can we compress execution traces from tests into something human-readable?
	* Alternative: add print statements?
* Some kinds of invariants might be broadly useful (like: what is mutable?). But, different programmers will have different ways of thinking, and hence, different information needs

