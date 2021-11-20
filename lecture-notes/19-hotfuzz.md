---
layout: page
title: "Lecture 19 - HotFuzz: Discovering Algorithmic Denial-of-Service Vulnerabilities Through Guided Micro-Fuzzing"
parent: Lecture Notes
nav_order: 19

---
# Lecture 19 - HotFuzz: Discovering Algorithmic Denial-of-Service Vulnerabilities Through Guided Micro-Fuzzing

### Background

*Algorithmic complexity vulnerabilities*
DoS on the cheap
Example:
* My server can process 10k requests/sec before crashing
* Hence, attacker cost is: 10k requests/sec
* What if I could create a request that takes 1,000x longer to process than any other request?
    * You can do a DoS with fewer requests/second

Why is it hard to defend against these kinds of attacks?
* How do you differentiate between legitimate requests and those that are targeted attacks
    * But: How do you compare the requests to identify if you are being potentially attacked at any given moment?
        * Traffic looks like:
            * Steady state: 5 req/sec
            * All of sudden: 15k req/sec
        * But with algorithmic complexity:
            * Steady state: 5 req/sec
            * Attack: 5 req/sec
    * What are the characteristics of inputs that are exploiting algorithmic complexity vulnerabilities?
        * (These might also be valid inputs that find performance bugs)
        * Timing?
* "Classic" examples for algorithmic DoS: Execution time that is super-linear to the size of the input
 
```xml
<?xml version="1.0"?>
<!DOCTYPE lolz [
 <!ENTITY lol "lol">
 <!ELEMENT lolz (#PCDATA)>
 <!ENTITY lol1 "&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;">
 <!ENTITY lol2 "&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;">
 <!ENTITY lol3 "&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;">
 <!ENTITY lol4 "&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;">
 <!ENTITY lol5 "&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;">
 <!ENTITY lol6 "&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;">
 <!ENTITY lol7 "&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;">
 <!ENTITY lol8 "&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;">
 <!ENTITY lol9 "&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;">
]>
<lolz>&lol9;</lolz>
```

*DARPA STAC*
* Space Time And Complexity Vulnerability Project
* Other fuzzers that find AC vulnerabilities:
    * Number of executed bytecode instructions -> proxy for runtime
    * Dynamic approaches for approximating complexity - O(n...)
* "Ground truth dataset"

### What is the big problem that HotFuzz is solving?
* Oracle problem: Existing fuzzers use an oracle of "program must crash" or a sanitizer fake-crashes the program
* "Hard to get broad code coverage"
    * Micro-fuzzing vs API fuzzing?
        * Problem with existing API fuzzing work: "Developers need to write harnesses"
        * Q: Why do those existing techniques require harnesses?
            * You might randomly violate preconditions, which will result in generate some call that could never actually happen in practice
            * Q: Does micro fuzzing solve that problem? No - can get false positives
            * "Determining whether an adversary can transform these test cases into working AC exploits on victim programs is outside the scope of this work"

### What is the solution?

Q: What does HotFuzz use as an oracle to determine that there is an AC vulnerability?
* Execution time (set a timeout)
* Q: Ideas for an alternative oracle?
    * Throw a range of inputs at the system to show input size vs runtime - try to maximize smaller inputs that get you the same runtime as bigger inputs or worse
    * What about sequences of inputs?
        * Classic example: Hash collisions in a table - you exploit this vulnerability by creating a sequence of inputs that will collide in the hash table, not obvious as a vulnerability if you look at just one input
        * Can you draw a baseline for performance under normal operation, and then find inputs that are outliers?
    * Where is the line drawn between a performance bug and an algorithmic complexity vulnerability?
        * What is a performance bug but not a complexity vulnerability?
            * Maybe it's primarily tied to: "Can an attacker deliberately trigger it"
            * Or: "An AC vulnerability is a performance bug that is found by an attacker, and not by you"
    * Why not use a static approach, or combine a static approach with a dynamic one?

#### Micro-fuzzing
Fuzzing java methods... Java methods take inputs that are typed... String, List, int, etc.
Q: How do they generate these typed inputs?
* IVI - Identity Value Initialization
    * Fixed set of default values
* SRI - Small Recursive Insnantiation
    * Random values selected from some set of possible values
* Do we care about "Validity" in the sense that Zest did?
    * No - try to directly fuzz some code while bypassing any complex checks or type invariants
* Evolve the input corpus using mutation + crossover
    * Consider AFL/Zest - performs these changes on some bytestream input. HotFuzz will manipulate types directly like java.util.List
    * How do they mutate a list?
        * Mutate a random attribute - only mutate strings, primitives, and arrays of primitives
    * How do they do crossover on a list?
        * Roughly speaking: take two objects, pick some field at random as the crossover point, exchange field values between two objects at that point
        * Example of crossover (generally):
            * Input 1: XXXXXXX
            * Input 2: YYYYYYY
            * Mutate Input 1:
                * XXXXXXXZ
            * Crossover Inputs 1, 2:
                * XXXYYYYY
* "Witness synthesis"
    * These AC-inducing inputs are found in an unrealistic environment - not a normal JVM. Create a script that can trigger this input in a normal JVM

#### Implementation
* Patch on OpenJDK
    * 1,007 lines of C+++ code, 5,497 lines of Java code, 288 lines of Python code - is this a lot or a little? :)
    * Complexity of the program, general correctness?

Q: what are the features that they need to get out of jvm, and why change it?
* Timing measurements for just a single method at a time - presumably ms or ns?
* "Didn't want to alter the program bytecode"
* Focus on performance, but still run evaluation in interpreted mode?
* Interesting discussion of JIT compilation + interpretation
* Why not generate harnesses, which collect timing info?
    * In particular - "JMH" is the internally-developed performance microbenchmarking toolkit created by the JVM developers - could build a fuzzer that uses these harnesses to precisely collect timing information, and...
        * Also offers a solution for controlling GC
* If you fuzz functions that are normally JIT'ed, and you do it in interpreted mode, will it change the vulnerabilities that you find?
    * Yes! They use "witness" to validate in non-interpreted mode JVM, but still subject to "did this get hot enough to get JIT'ed?"

* Fuzz things besides C! Interesting problems here, not fully worked out:
    * How to fuzz typed methods (ideally creating "valid" inputs")? [Input validity]
    * How to detect a bug/vulnerability? [Oracle problem]
