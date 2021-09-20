---
layout: page
title: Lecture 3 - Schuler & Zeller ICST 2011, Checked Coverage 
parent: Lecture Notes
---

# Lecture 3: "Assessing Oracle Quality with Checked Coverage"
Received the N-10 most influential paper award (in 2021)

Agenda:
1. Friday check-in
2. Continued discussion: test adequacy
3. Discuss "Assessing Oracle Quality with Checked Coverage"

### Some examples for discussion
```java
int sum(int[] array){
	if(array == null)
		throw new IllegalArgumentException("Input must not be null");
	int ret = 0;
	for(int i = 0; i < array.length; i++){
		ret += array[i];
	}
	return ret;
}
```

```java
void testSum(){
	assertEquals(10, sum(new int[]{0, 1, 2, 3, 4}));
	// also should add a test for array is null, in order to get coverage
}
```

Do mutation testing:

```java
int sum(int[] array){
	int ret = 0;
	for(int i = 1; i < array.length; i++){ //Mutation: I = 1 instead of I=0
		ret += array[i];
	}
	return ret;
}
```

```java
void testSum(){
	assertEquals(10, sum(new int[]{0, 1, 2, 3, 4}));
	assertEquals(10, sum(new int[]{1, 0, 2, 3, 4}));
}
```

```java
void testSum(){
	assert(sum(new int[]{1, 0, 2, 3, 4}) > 0);
}
```

What were the problems in using mutation testing to evaluate test suite quality?
1. Time-consuming (machine time): Need to run lots of mutants and lots of tests
2. Time-consuming (human time): Need to detect equivalent mutants

## What is the motivation for this paper?
* Something better than branch coverage to judge how well tested our program is
* What is the design space of solutions that are considered?
	* Want a scale from 0-100, where 100% means "you are done" and 90% means "keep going"
	* Stronger criteria than just statement/branch coverage 
	* Want something FASTER than mutation testing

## What is hard about this problem?
* We need something that measures the quality of the oracle, not just the code that is executed
* Same problems that we always see evaluating test suites: the test has the oracle that says what the correct behavior is, but how do you know what the REAL correct behavior that you should be checking is
	* What does it mean to "have the results checked"

## What is the proposed solution?
* "Checked coverage" -> what statement *AND* are included on a backwards slice from any of the assertions?
* Sidebar: What is backwards slicing?
```java
static int sum(int N){
	int I = 0;
	int sum = 0;
	while(I < N){
		sum = sum + I;
		I = I + 1;
	}
	System.out.println(I);
	System.out.println(sum);
}
```
* Step 1: Take a trace of the program execution that you want to slice on, for instance `sum(2)`
```java
Int I = 0;
Int sum = 0;
I < n (true)
Sum = sum + I; //sum = 0;
I = I +1; // I=1;
I < n (true)
Sum = sum+I; //sum = 1;
I = I + 1; // I = 2;
I < n (false)
System.out.println(I); //2
System.out.println(sum); // 1
```

Example: Trace on `println(I)`
```java
Int I = 0;
I < n (true)
I = I +1; // I=1;
I < n (true)
I = I + 1; // I = 2;
I < n (false)
System.out.println(I); //2
```
Note: this is not necessarily following formal definitions of control dependent, definition of control dependent is not in the paper.
* What is the normal definition of control dependence that we should use?
    * "You are control dependent on a branch if you exist between the position where a branch splits control flow, and it comes back together" - There is a post-dominance relationship between this node and the control flow node/branch

```java
Int sum(int n){
	int I = 0;
	while(I<N){
		I = I + 1;
	}
	System.out.println(I);
}
```
* Why use slicing for checked coverage?
	* Intuition: If some code is covered, but not included in a backwards slice from the assertions, there are no data or control dependencies between the assertion and that code - "it is not checked"
	* Question: If some code IS included in that slice (it is "checked") do we have more positive sentiment towards the test?
		* We can "loosely" say that this statement influences the outcome
		* This is a totally Boolean "it is checked" or "it is not" 
* What are going to be the problems of using slicing for this?
	* Example: figure 1...?
	* This is a particularly opinionated definition of what a "good" test is, and that definition is heavily rooted in how the system was implemented (with slicing)
```java
Public void testValidExecution(){
	try{
		Object ret = doSomethingVeryRiskyThatMightThroughException(someValidInput);
		assert(ret != null); //Just because I added this, perfect checked coverage :/
	} catch(Exception ex){
		//uh oh!
		fail("Expected no exception!");
	}
}
```

```java
Public void testInvalidExecution(){
	try{
		doSomethingVeryRiskyThatMightThroughException(invalidInput);
		fail("Expected exception!");
	} catch(Exception ex){
		//Correct behavior
	}
}
```

* Problem with branch not taken:
```java
Static HashMap myHashMap = new HashMap();
Static{
	myHashMap = null;
	myHashMap.add(...);
}
Public void testValidExecution(){
	Boolean OK = determineIfSystemIsOK();
	
	if(OK){
		//don't fail test
	}
	else{
		//do more complex checks to understand the system state
		callSomeOtherMethodThatWillProbablyFailTheTestButTheyDontKnowThatItWill()
	}
}
```
* "All test classes traced separately"
	* Class initialization code will get run each time that a new test runs, that initialization code will never be checked

## Evaluation
* "Qualitative analysis"
* Discarding assertions - lots of noise
* What additional evaluation do you think that could have been done, or could be done now?
	* More statistical analysis - what is the significance of the results? (Especially for performance...)
	* Is checked coverage correlated with fault detection?
