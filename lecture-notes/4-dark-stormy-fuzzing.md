---
layout: page
title: Lecture 4 - Miller et al CACM 1990, "An empirical study of the reliability of UNIX utilities"
parent: Lecture Notes
nav_order: 4
---

# Lecture 4 - "An Empirical Study of the Reliability of UNIX Utilities"

#### What was going on in software reliability/software engineering at the time this was written?
"The Software Crisis"

"The major cause of the software crisis is that the machines have become several orders of magnitude more powerful! To put it quite bluntly: as long as there were no machines, programming was no problem at all; when we had a few weak computers, programming became a mild problem, and now we have gigantic computers, programming has become an equally gigantic problem." - Edsger Dijkstra, 1972 [Turing award acceptance speech](\0x03)

##### Motivating software disasters/bugs that we use to create a narrative for WHY software reliability is important
* Healthcare.gov 
	* Testing & development & management
* Heart bleed
	* Buffer overflow in OpenSSL
* Equifax - Remote code injection in Apache struts (Java)
* What was the motivating example in this paper?
	* Morris worm
	* There was a bug in `fingerd`
	
```c
char inputPassword[BUFSIZE];
char realPassword[17];
strncpy(realPassword, "mySecretPassword", 17);
gets(inputPassword); 
```
* This is a buffer overflow 
	* Interesting legacy from this attack:
		* Movie; professorship...
	* Why is this kind of attack/vulnerability/bug interesting for the paper, or for us today?
		* This bug *still* happens
		* Problem: need to consider *unexpected* inputs when testing
##### Unix history and development
* 1949 - US DOJ starts antitrust investigation
* 1956 - US enters Korean War, "consent decree"
* UNIX is born; because of consent decree can't commercialize
* 1983 - DOJ settles a new antitrust case against ATT, ATT starts to commercialize UNIX - GNU is born

### So, what is the motivation of this paper?
* How do we find inputs that humans are unlikely to test?
* "It was a dark and stormy night..."
	* Situation is: analog, dial-up internet. System might be unreliable, bit flips
* Complement, not replace testing
* Focus on making software "robust" (word not used directly in paper)

### What is the proposed solution?
* Generate random inputs to the program, and test them?
	* Alternative, that is not proposed or evaluate: start with the valid input, then randomly change it so that it's an unexpected input [we come back to this on Friday]
* Existing work was generating random network inputs, this takes inspiration from that
* How does it work?
	* Generate a continuous string of characters, which you can then use as an input to a program
		* Printable characters only
		* Printable + control characters
		* The above two options either with or without null byte characters
	* Options for tool include:
		* Length of input
		* Random seed
		* Save input to file for debugging
	* Depends on piping:
		* `fuzz 100 -o fuzzInput | myUtil`
	* Ptyjig - for interactive applications
		* "Pseudo-Tty"
		* "Tty" - TeleTYpewriter 
	* How does it detect a failure? [What is the oracle?]
		* Program crash (as detected by creation of a core dump)
		* Infinite loop (wait for 5 minutes after input was stopped)
	* Seems mostly reasonable
	
### How is the proposed solution evaluated?
* Run 88 unix utility programs, 6 OS's, found "a surprising number of bugs"
* What did we think of the experimental setup?
	* How long did they run the fuzzer for? Fixed inputs, or fixed amount of time?
	* Which configurations find more bugs?
	* How repeatable were each of the bugs, and what were the seeds used to find the bugs that they found?
	* Reports are classified by cause - were there multiple bugs in each program, multiple ways to get the same crash, or even multiple ways to get different crashes?

### Discussion
* What is a valid precondition for an application, and what is an input/crash that is a bug versus an input/crash that is simply in violation of the input spec, and it doesn't matter if it crashes or not.
	* At best, a bad user experience
	* Valuable to fix bugs that can be used as part of an exploit chain, and to recognize that we might not be able to consider the entire exploit chain right now
* Interesting to look at 5.1, see issues that are still problems, and issues that have been solved in PL design
* How can we use these error descriptions to help target our testing?
	* Infer array bounds, try to get inputs that will exceed bounds?
	* "Even (especially!) pointer-based array references in C should be checked for elegant style often used by experienced C programmers..."
	* Maybe we should pay more attention to writing robust code, rather than elegant code - security should not be an afterthought, and if elegant means more vulnerability-prone, don't be elegant?
* Last week there was an iOS buffer overflow 0-day on the image parser. Why is this still a thing?
	* Tradeoffs in terms of performance, readability ("writability")
	* Why are these lessons that we keep learning?
		* Is it about new developers?
			* There are a limited number of resources, and businesses are incentivized to get the product out sooner than later. Security is a cost to provide, and only "pays off" if it prevents a vulnerability!
		* Can't we just design a way to write programs so that this doesn't happen?
			* Most of these things require runtime checks -> performance penalty
* What about GOTO statements?
	* someLabel: <someStatement>
	* GOTO someLabel
	* See: Entire correspondence on [GOTO considered harmful](https://homepages.cwi.nl/~storm/teaching/reader/Dijkstra68.pdf): Several more letters exchanged ([complete set in this PDF](https://www2.cs.arizona.edu/classes/cs372/spring17/gotoletters.pdf)) including: "'GOTO Considered Harmful' considered Harmful" Considered Harmful?, "GOTO, one more time", concluding with Dijkstra's ["On a somewhat disappointing correspondence"](https://www.cs.utexas.edu/users/EWD/transcriptions/EWD10xx/EWD1009.html) (also in PDF)
* Error handling :'( 
