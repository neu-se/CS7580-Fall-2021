---
layout: page
title: Lecture 7 - Donaldson et al OOPSLA 2017 Automated Testing of Graphics Shader Compilers
parent: Lecture Notes
nav_order: 7
---
# Lesson 7 - Automated Testing of Graphics Shader Compilers

#### Context and background

Graphics shader compilers are complex, tricky beasts, often full of "optimizations" that may or may not be sound

#### Metamorphic Testing

Example is: How do we test `sin(x)` without knowing what the outputs should be?

"Metamorphic relation": Can we specify some property that describes a transformation on the input, and a corresponding transformation on the output?

`sin(-x) = -sin(x)` `sin(x+2pi) = sin(x)`

X = 1
So, check: `sin(-1) = -sin(1)`

Question: What does this buy us in terms of ability to actually test things?
* Just testing some properties - might be a wide range of buggy implementations for which this property holds
* Need to make sure property is right

Question: What other domains might we be able to specify these properties for?
* Linear algebra, or anything that deals with mathematical transformations
* Physics modeling software - e.g. finite element analyses 
	* If the ship gets bigger, certain things should increase
	* Might be effective testing approach to catch integration issues between modules that might have incorrect unit conversions, etc.
* SQL - multiple ways to write the same query should result in the same return value
* If you have a formally verified compiler, could you automatically test it by using the properties that you verified to generate inputs

Discussion: What do the metamorphic relations look like and where do they come from?
* Physics simulations - define coarse properties
* Compilers + related - use "No-op" transformations, or use formally defined properties of compiler
* Is performance testing metamorphic testing?

### What is the motivation?
Shaders for graphics are implemented in languages that provide intermediate representations - GLSL, SPIR-V - so each graphics card needs to have its own graphics driver.

What's the problem with GLSL + SPIR-V vs LLVM?
* "Without any [precision] qualifiers, implementations are permitted to perform such optimizations that effectively modify the order of operations used to evaluate the expression, even if that produces different results"

Why can't we do differential testing? What makes us need metamorphic?
* Results just have to be "good enough" - how do you precisely compare the output from two machines to say that the graphics are "fine"
* Maybe there are bugs that can't be exposed by differential testing with current shaders - need to generate new shaders to reveal those differences?
* By modifying an existing fuzzer in a way that is semantics-preserving but reveals a bug, we might be able to provide more helpful feedback to developers that just says "Adding this small change causes a bug!" Rather than "Here's a 1KB file that crashes"

### What is the solution?

Apply semantics-preserving transformation, run inputs, see if output is the same
`compileAndRun(P,I)=compileAndRun(f(P), I)`

1. Generate variants
	1. Add dead-code of various kinds: conditional branches that are never taken, or "live code" that is a no-op like surrounding statements with if(true){}
		1. Inject dead code code, copy/pasted from another shader
		2. Inject dead code that includes a control-flow statement (break, continue, etc)
		3. Inject code that is not dead, but accesses disjoint data from what is already accessed in this shader
		4. Mutate expressions using mathematical identities (x = 1*x, x = 0 + x, x = x && true)
		5. Pack variables into a vector
		6. Wrap code fragments with semantics-preserving control-flow like if(true)
	2. Q: can they guarantee semantic equivalence?. A: No, it's just "essentially semantics-preserving" 
	3. Q: How do you generate this list of transformations? What other transformations might you try?
		1. Extract code into a method
		2. Could add matrix-level identity operators too (not just scalar identities)
		3. Maybe is a long, iterative process - try to make it work with just some of these properties, iterate based on results?
		4. This paper represents a *tremendous* amount of engineering + research/development effort
	4. Q: What is "swarm testing"?
2. Detect "deviant" variants
	1. Compare images using fuzzy matching, set a threshold for how different is "too different"
3. Reduce deviant variants
	1. Reverse a random set of transformations, if not equivalent, do it again. Repeat until it converges to local minima
	2. Q: This will not guarantee that we reach a global minima in program size - just a local minima. "Obviously we could use delta debugging if a global minima were necessary"
	3. Q: What do we do about variants that are just caused by floating point rounding, and not because we found a bug in the shader compiler? Do we separate those out, or handle them differently? Are they false positives? A: If the original shaders are "high value" then it is probably still quite useful to find a floating point bug

### Results

Q: looking at the bugs found in table 2… what is a reasonable effort that we think should be expended to find these?
A: If we are NVidia, maybe not that much ($1m too much? Maybe, maybe not?). Interesting problem where developers can't attribute bugs to their shader code vs to a bug in the compiler
A: If we are a company like Google or Apple, we probably will pay infinity!

What is the experimental design?
* Take 17 devices using 3 evaluation sets
* Use 1,000 shaders from GLSLSandbox.com, took 100 largest shaders that could be compiled
* How do they apply the transformations?
	* "Swarm testing" - uniformly random choice to enable or disable each transformation for each variant, then random chance to apply it at each point
* Identify false positives - do a study with 3 people, at least 2 of 3 participants had to say that images were identical to consider them as such
* 568/975 true positives -> 407 false positives -> 42% false positive rate
	* Discussion: do we think it would be possible to get this rate lower?
* 568 true positives -> 71 issues -> 64 non-false-positive issues
* (No false negative evaluation)
* Could they have used a baseline? Differential testing? Similarity metrics?

### Overall discussion

* Would it have been possible to target security vulnerabilities instead of looking for all kinds of bugs
	* If you know what kind of bug you are looking for, could you build transformers related to that? Or, a location to insert that transformation?
	* Try a transformer that changes scheduling/event delivery ordering
	* Is there something that we can learn from the security vulnerability that were found in this paper?
	* WebGL was supposed to include static checks of memory accesses, but clearly didn't actually do that
* Is there a faster (less engineering time) way to get these results?
	* Less precise/faster algorithm?
	* Is there a better software architecture for designing this thing to make it easier to scale to more/different versions of devices?
