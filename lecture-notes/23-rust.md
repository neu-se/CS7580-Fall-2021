---
layout: page
title: "Lecture 23 - Fuzzing the Rust Typechecker using CLP"
parent: Lecture Notes
nav_order: 23

---
# Fuzzing the Rust Typechecker using CLP

### Rust
* Linear types
    * What is this?
    * Each variable is used exactly once
* Borrowing
    * At each given point, there can be exactly one owner for each variable, and borrowing is the notion of one scope taking ownership of a variable
* Mozilla...

### PL things that we might need to discuss
* Lambda calculus
    * Formal notation for defining functions that accept a single argument, lambda x:type . expression
    * Used to define type systems
    * Has only functions, and variables that are used in those functions
    * Define an abstraction, which describes how to write functions, and an application which describes how to use functions
    * Start from this small kernel and then create booleans, numbers, and anything else
* System F
    * Extends the lambda calculus so that you can define functions that accept multiple kinds of variables (like Java generics), so that you can define type variables for your types
    * System F also has a notion of type abstractions, which parameterizes an expression by a type - you can pass the type itself as a variable
    * If you want to define a type system that has generics, then you probably want to start with System F
    * Figure 2 shows example typing rules. Typing rules for rust will be *much* more complicated
* Curry-Howard
    * Type system is the logic system
    * Program is a proof of that type system
* CLP
    * Relevant for this paper: you can have a complex type system expressed as logic propositions, then use a logic programming language like prolog to encode the typing rules

### Problem space and alternative approaches
* CSmith: The problem is that it's really specific to C
    * How does CLP solve that problem?
* "Almost well-typed" generators
    * This is somewhat more novel compared to other structured fuzzing stuff that we looked at: they are purposely generating things that are slightly off from valid, rather than only fully valid, which means they will maybe be more likely to test the boundary conditions of what is "nearly" invalid or "nearly" valid


Q: How do you go about creating typing rules for Rust?
* Spend a lot of time in your PhD writing them?
* Do they formalize the ENTIRE type system?
    * Not entirely - each generator that they create uses a different (but perhaps overlapping) set of typing rules
* How do you test this?
    * Differential testing?
* The typical approach for language design starts with a grammar and a compiler, and then the type system is "figured out as you go" - so no formal spec

Q: Does this require more human resources than CSmith?
* What was the part of CSmith that took so many person-hours?
    * Iterating on developing the generator
    * Examining the test cases generated and their results
* No: creating the typing rules might be easier than writing a generator directly, especially if the typing rules are closely related to some other set of typing rules
    * For TypeScript, it's probably easier to write the generator directly :)
* What about compared to the Java differential testing paper?
    * Creating entirely invalid input files to JVM is certainly easier, even with "hooking instructions"
* "We generated 900m programs" is a lot (is it?), but what fraction of them required examining/manual investigation to get down to the 18 reports?
* What about a baseline of just writing test cases for Rust's type checker?

Q: Is this a paper about fuzzing, or about manually constructing an oracle for testing a type checker?
* AKA: This paper is differential testing, and the authors implemented the alternative system to differentially test to - which is the typing rules
    * Is there a contribution from encoding the Rust type system in CLP?
        * Similar approach has been used for other type systems, these authors did it for JS previously, too
* "How do I generate programs that might expose differences between the encoding of the typing rules and the implementation of the type system?"
    * How do you get from the typing rules to something that creates programs?
        * It's a DFS or the typing rules
    * How do they solve the pitfalls of CLP described in section III.B?
        * Are there other ways that we could generate programs that might be even more likely to reveal bugs in the typechecker that doesn't use CLP?
            * Create a random generator that creates rust programs, which may or may not be valid grammatically or type-safe
                * Use the typechecking rules in our fuzzer to determine whether in an input is "valid"
                * Use reinforcement learning to guide the fuzzer to create more, random, "valid" programs
            * Here's a better idea: Define *mutators* that will not change the type-correctness of a program. Get a corpus of well-typed programs and invalid-typed programs, mutate them, see what happens.
                * "Type-safe mutation"
                * Does this work for all of the kinds of bugs that were discussed in this paper?
                  Q: What are the types of type checking bugs that they look for?
* Precision (Typechecker conservatively rejects that it should accept)
* Soundness (Typechecker optimistically accepts what it should reject)
* Consistency (Multiple programs that are equivalent in some way with regards to typing are not treated the same)
    * How are "consistency" bugs any different from precision or soundness bugs?
    * Page 6 describes their type-safe mutation that creates this
    * Could we use some kind of feedback to better guide the search for consistency bugs?
        * Greybox fuzzing ideas here? Use branch coverage of the typechecker as some fitness metric, evolve programs to try to cover the different branches?
        * Or: "production coverage" where you try to cover the different productions of the language's grammar
    * Had they gone further with the mutator, it might be possible to also examine cases where the generators are not able to create programs that explore all branches (because of limitations of the generators), since the mutator could then transform that code into something that wouldn't have been generated otherwise, and then covered that other code

Q: What would have been an empirical baseline to compare against?
* Stochastic grammars - only create valid programs, but most of the bugs that they found were revealed by valid inputs, so you could show overlap there at least
* Something in the style of CSmith?
* Something that is a super naive fuzzer?

Q: Are there other applications of these Rust (or other language) typing rules in SE?
* Testing?
* Could you use the typing rules to auto-generate "more useful" compiler error messages for ill-typed programs?
    * Rust might not be a very interesting target for this, because its type system might limit the complexity of the error?
