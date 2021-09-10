# Lecture 1: Introduction and overview

1. Introduce self, describe why I am here
2. Introductions around the room: Name, degree program, past experiences with software engineering, why are you here?
3. Review course policies and schedule: https://neu-se.github.io/CS7580-Fall-2021/
    1. Intended structure of each class period
    2. Remote and in-person participation expectations
    3. Grading philosophy
        1. Why use coarse buckets for grading? No credit, Check-, Check, Check+ 
        2. Good Faith Effort standard
        3. Due dates
        4. How are grades combined into final grade?
    4. What are assessments?
        1. Participation
            1. Have something to say about the paper
            2. Alternative: email 3-5 sentences to instructor
        2. Paper
        3. Project
4. OK, so what is the space that we are working in: Testing and analysis, but mostly testing. What is testing?
    1. What makes a good test?
        * Hypothetical: building a system that checks a zip code validator service
        [Diagram] Input is characters from user, output is "error" or 1..n "place names" that match that zip code (example: 02119 -> Roxbury, MA and Boston, MA)
    2. Observation: two big problems in testing - inputs and oracles
    3. What are input generation ideas?
        1. Blackbox:
            1. Boundary values (example: zip code, boundaries are length 4, 6, very long, no digits, non character)
            2. Random values (note: testing all is hard, consider all zip+4 - 100,000 inputs to test, but maybe we can do it if its fast and automated)
            3. Sampled values from production system
        4. Whitebox:
            1. Symbolic execution
            2. Manual analysis of code
    4. What are oracle ideas?
        1. Human knows the right answer
        2. No-crash
        3. Formal specification
        4. Pseudo-oracles:
            1. Regression
            2. Differential
            3. Metamorphic
                1. Example: self-driving cars, transform input images in a way that you can predict the output, e.g. adding fog. Question: how do you know the fog is realistic? :(
5. [Save at least 20 minutes - start this by 11:10am]: What to do for next class - how to read paper?
    1. Note, if haven't read technical papers before: might be challenging, often written in very condensed style, assumes certain knowledge
    2. Note, might want to read a paper multiple times, first skimming and skipping over parts that you don't fully understand, then go back over again more carefully. Might help to do in different sittings
    3. Considering highlighting, note taking
    4. Might want to think about what you want to get out of each paper, thinking about questions like:
        1. What is the motivation for this work? 
        2. What is the problem that is being solved?
        3. What is hard about that problem?
        4. What is the proposed solution?
        5. How is that solution achieved?
        6. How is that solution evaluated?
    5. After reading, good to reflect on the paper - the problem, the solution, and the evaluation:
        1. Is this a problem worth solving?
        2. Is the solution a good idea?
        3. Do you see limitations to the problem, or the solution?
        4. Is future work needed to fit this research prototype into the real world problem domain?
        5. What questions does this paper leave you with?
    6. If you think that you can answer each of these questions, then you have done an excellent job reading the paper. If you don't think you can answer them each (despite re-reading the paper to try to find those answers), that is OK! Not graded weekly on answering these questions - reflection paper will do that, and can pick which papers.
    7. Suggested further reading on how to read: https://cseweb.ucsd.edu/~wgg/CSE210/howtoread.html , note-taking template: https://cseweb.ucsd.edu/~wgg/CSE210/paperform.pdf
