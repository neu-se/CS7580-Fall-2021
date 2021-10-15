---
layout: page
title: Lecture 9 - Legunsen et al, ASE '16 How Good Are the Specs? 
parent: Lecture Notes
nav_order: 9
---
# Lesson 9 - How Good Are the Specs? A Study of the Bug-Finding Effectiveness of Existing Java API Specifications

### Specs
Q: What are "specs" in this case?
A: "Behavioral specifications for APIs"
* They are pretty much all specs over the Java API
* Specs can be automatically generated (by parsing documentation and using some ML) or manually generated
* Example: `hasNext` 
```java
Iterator iter = myCollection.iterator();
while(iter.hasNext()){
	Object obj = iter.next();
}
```
"Starting state" -> "HasNext was called" -> "Next was called" 
* Collections_Synchronized Collection (CSC)

Q: What do we think of the idea of RV?
* Discuss: Is this overly restrictive? How do we ensure that the specs that we encode represent all valid executions?
* Why is this better than assertions?
	* More expressivity, but slower
* There is a wide spread in the specs in terms of criticality - not all specs may be useful to all developers in all situations
	* Example: "Creating a new socket, you must check to make sure that the port that you specify is within a valid range"
	* Consequence of not doing this: You'll get an exception - what else would you want to do though?
* Discussion: What's the problem with relying on un-specified or under specified behavior? [Hyrum's Law](https://www.hyrumslaw.com) (And a defense: "Chaos mocks")
	* See also, [NonDex](https://www.cs.cornell.edu/~legunsen/pubs/GyoriETAL16NonDexToolDemo.pdf)
* Discuss: could we use the false positives to improve the specs?

Q: What makes a spec effective for bug finding?
* If it correctly models the behavior that we want the API to take on. Trade off with precision/soundness
* A spec is effective for bug finding if it finds bugs that developers care about (hopefully without many false positives) - but this is expensive! ~30 person-weeks inspecting violations and making the PRs, plus the time to find the tests, find the projects, find the specs, running Randoop, etc.

### Gigantic empirical study
200 open source projects, 182 manually written specs, 17 mined specs, 18k developers, 2m automatically generated tests

Q: What happened with the "Mined specs" - why only 17?
* Many were unusable
* 206 were too complicated to check or understand, or were specific to Java Swing and Java AWT
* Some were trivially statically checkable ("Every class that implements Serializable must have a static final long serialVersionUID field")
* Not specs that were representative of real bugs - just performance issues at best

Q: Literature survey of 100 papers -> 26 papers involved mining java specs -> only 17 had formal specifications defined -> email all 17 sets of authors -> 7 respond -> 5 provided specs
* Low reproducibility! 
* What is the authors' responsibility post-publication?

Q: What is the pipeline to go from JavaMOP violation output to bug?
A: Each violation -> 2 developers review -> determine "True positive" "false positive" or "Not sure"
3rd person reviews the final result -> If true positive, go through source code to find what is causing the violation -> create a pull request -> see if developers accept it

DV -> All violations
SV -> Unique violations

Q: why pull request?
A: In past research, the authors self-determined business. Make a PR and not just open an issue to try to make it more likely that the issues will be accepted

### Evaluation
*Performance*
* Negative overhead in some case -> why wouldn't we do repeated trails and get statistical trials?
* "4.3x overhead overall" -> "Overhead ranged from statistically insignificant to up to Nx slowdown"
* Actual time ranged from 4-12sec usually

*True bugs vs false alarms (Table 4)*
* Overall false alarm rate -> 82.81%
* "In several cases, there were no false alarms, only true bugs reported by the specs, in other cases, the false alarm rate could vary, in the worst case, up to 100%"
* Why not show Table 4, but without the 31 specs with 100% FAR and without iterator.hasNext() (Maybe was done in follow-on work)

What about the severity of the consequences of the violation?
* "We used a synchronized collection, but we don't make any claims of thread safety, so it doesn't matter that we didn't synchronize on the iterator"
* Also found some buggy specs
* Some developers liked the fixes, others thought that they were pointless

### Editorial discussion

Q: If we were unhappy with evaluations before, are you happy with this one?
* Yes! But a LOT of data to fit in 10 pages!

Q: Are there kinds of APIs where this might work better?
* Comment in the end of the paper about how you write a good spec - but code can change, and then the spec breaks
* What is the oracle here? How do you help developers write their specs? It is very subjective to determine what here is a "bug"

How about a situation where:
* An API is under-specified
* That API is under frequent revision
* Task: Infer model of API behavior TODAY, detect breaking changes TOMORROW
* See [Akita Software](https://www.akitasoftware.com)
