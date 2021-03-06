---
layout: page
title: Lecture 21 - Evolutionary Generation of Whole Test Suites
parent: Lecture Notes
nav_order: 21

---
# Lecture 21 - Evolutionary Generation of Whole Test Suites

### What is the problem space that EvoSuite is in?
* Generating tests with no automated oracle
* With a limited time budget
* "Our goal is to target difficult faults for which automated oracles are not available (which is a common situation in practice)"
    * What is "difficult faults"?
    * Why do we need automated testing?
        * Fuzzing: "We need automated input generation that can create inputs that a developer wouldn't have thought to create, and reveal subtle memory corruption bugs"
            * Do you make any assumptions about what the developer does or does not do correctly? No
        * Randoop: "We need automated testing to find contract violations"
            * Do you make any assumptions about what the developer does or does not do correctly? Yes - Assume that the developer can specify contracts correctly
        * EvoSuite: Prioritize coverage - "We need automated testing to create test suites that have high branch coverage, which saves developers time, because then developers just have to write assertions"
            * Do you make assumptions about the developer? Yes - Assume that the developer is going to add oracles correctly

### Prior approaches
Prior approaches look at coverage at a more localized level - generating tests per-coverage goal

Why not just generate high-coverage test cases for each method individually? Why need "full program test suites?"
* Can end up with very large test suites
* Likely to have wasted resources - some branches are infeasible or hard to reach - you might end up wasting time trying to cover those branches
    * 10 minute timeout per branch 2 hour timeout for the whole thing:
        * First 12 branches that I try, I can't solve, time out -> Have "wasted" the whole time
        * Alternatively: Might have picked 100 branches that could cover trivially (and generated that suite)

Q: What is "hard" about doing this at the test-suite level, and why it was done at the per-branch level before?
* Per-branch might be more systematic, and matches with use-cases for symbolic execution
* How do we make sure that we cover not only the most probable paths, but all of them?

Q: What do you think these tests should look like?
* Depends on how we plan to add the assertions
    * We know what code is covered by each test, so maybe it doesn't matter what the test looks like, as long as we can tell what part the code is being executed and tested, and then infer some property for that code
    * Does readability matter, though?
        * Programmers will need to create oracles, still
        * Programmers will also need to be able to maintain the test suite - as code evolves, evolve the test suite too
        * When writing tests by hand, developers might start with the idea of a property or oracle in mind, and then write the test that sets up the system for that property. What do we need to do to support developers adding oracles to existing tests?
        * In general: writing oracles for tests that you didn't write is probably harder than writing oracles for tests that you wrote (regardless of whether they are automatically created tests or not)
        * What about if/when a test actually fails!? How do you debug one of these tests?
    * Hot take: https://twitter.com/_pitest/status/1452672665966333955
    * "Are we going to find a bug" requires a good answer to both input generation + oracle problem: Seems like it is painful to deal with both problems at once, but techniques that are most successful probably combine both sides of this equation (example: GLFuzz)
* Are there any metrics that we could measure and optimize over that might have something to do with readability?
    * Lines of code?
    * Cyclomatic complexity - proxy for number of independent paths in program: nesting branches/loops
        * We could argue about whether cyclomatic complexity is a useful metric or not, but the tests generated by EvoSuite don't have control structures, so this is a non-issue
        * Also: Halstead's complexity metric

### How does EvoSuite generate tests? Before getting into evolutionary part
* What is the representation of a test for EvoSuite?
    * A sequence of statements
    * Length of a test -> Number of statements
    * Each statement represents a single value
    * Kinds of statements:
        * Primitive statements (primitive variable definitions) `int x = 4`
        * Constructor statements + reference assignments `HashMap hm = new HashMap()`
        * Field statements (reading public member fields - maybe writing also?) `int size = hm.size`
        * Method statements (call methods - virtual or static, but not constructor) `Object o = hm.get(key)`
        * Where do the parameters come from to pass to methods + constructors?
            * Prior statements in that test - since each statement represents a single value

### Fitness function
Branch coverage

Optimal solution - covers all feasible branches and is minimal in number of statements

Reward test suites with better coverage, if two test suites have the same coverage, reward the smaller one

Q: Do they just track whether branches are covered or not, or more complicated?
* More complicated:
    * Track "minimal branch distance" example:

```java
void test1(){
	int x = 100;
	magic(x);
}
void test2(){
	int x = 0;
	magic(x);
}
void magic(int x){
	if(x == 95) //record x, record 95, compute distance
		fail();
}
```
* In this example, test1 is closer to reaching the branch
* Would this work for things more complicated?
    * Not sure
* Apply a normalizing function to put each of the distances into a value in [0,1]
    * In an ideal world, it is likely better to avoid normalizing, because different branches are different, and preserving that distinction might be important/useful...

### Bloat control
Negligible improvements in coverage could result in ever-increasing test suite sizes. This is controlled by:
* Limit the number of statements in a test, and the number of tests in each suite
* If children are closer in distance on some branches, but don't actually cover any more branches AND are bigger than parent, not admitted to the next generation
* "Dynamic limit control" - If a child test suite's coverage is no better than the best existing test suite's coverage, and the length is more than twice that, don't accept it
* When selecting test suites in the GA, ranks are determined first on fitness function, then on test suite size

### Search operators
* Crossover
    * Take two parent inputs, create a child input by picking some point in one parent input, then taking the first part of that input (up until the crossover point), and then the remainder of the other parent
    * Done at the test-suite level: so crossover on the test cases
    * No offspring will have more test cases than the largest of its parents
    * The total sum of length of test cases could increase
* Mutation
    * At test case + test suite level
    * To mutate a test case:
        * Each with 1/3 probability select one of these operators:
        * Remove
        * Change
        * Insert
* On average, mutate just one test case per suite at a time
    * Mutating on average a single element in a vector is typical in GP?
    * Other approaches involve annealing - start off making more drastic changes, then reduce changes as you go
* Entire search process is seeded using random tests. Q: Why is it important that the seed tests are selected with uniform random probability from the space of possible tests? What is the issue here, other than "We don't want to start with a very large seed test"
    * 1: Pick some random value r 1<=r<=L
    * 2: Generate a test sequence using approach above with length >=r
* Q: Would it make sense to have different kinds of mutators, maybe thinking more about what happens in fuzzers?
    * Grouping mutators together might be useful
        * Randoop: "let's just insert 100 of the same call" - EvoSuite might insert multiple statements in a sequence, but unlikely to be the same one multiple times
        * "unit-level" insertions/crossover... or something involving def-use chains
    * Crossover between two test cases?

### Evaluation
Open source projects + some industrial project
* Compare to non-full-suite generation approaches on coverage + test suite size
  Results are "staggering"
  What is the denominator on coverage?

Many of these RQs were studied and improved in later work, some are still open:
* Is the single-branch-at-a-time baseline realistic? What other methods could use use? Why not compare to Randoop? How do EvoSuite tests compare to developer tests on same projects, in terms of size (number of tests, LoC), and coverage?
* Readability?
* How hard to add oracles?
* How hard to main test suite?
* Ask developers to add oracles to EvoSuite tests, then see if you find any bugs?
