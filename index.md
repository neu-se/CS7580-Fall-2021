---
layout: home
title: Just the Class
nav_exclude: true
seo:
  type: Course
  name: Just the Class
---

# {{ site.tagline }}
{: .mb-2 }
CS 7580, Fall 2021, T/F 9:50-11:30am, Hastings Room 114
{: .fs-6 .fw-300 }

{% if site.announcements %}
{{ site.announcements.last }}
[Announcements](announcements.md){: .btn .btn-outline .fs-3 }
{% endif %}

## Instructor
{% assign instructors = site.staffers | where: 'role', 'Instructor' %}
{% for staffer in instructors %}
{{ staffer }}
{% endfor %}

{% assign teaching_assistants = site.staffers | where: 'role', 'Teaching Assistant' %}
{% assign num_teaching_assistants = teaching_assistants | size %}
{% if num_teaching_assistants != 0 %}
## Teaching Assistants

{% for staffer in teaching_assistants %}
{{ staffer }}
{% endfor %}
{% endif %}

## Overview
Building, delivering and maintaining successful software products requires more than being good at programming. Software engineering encompasses the tools and processes that we use to design, construct and maintain programs over time. Software engineering has been said to consider the "multi person development of multi version programs." Development processes that work well for a single developer do not scale to large or even medium-sized teams. Similarly, development processes that work well for quickly delivering a one-off program to a client cause chaos when applied to a codebase that needs to be maintained and updated over months and years. 
This class will explore recent research in software engineering, focusing particularly on automated approaches that help developers create higher quality software, faster.

The primary objective for this course is for students to learn different methods for analyzing software, with several applications in software engineering, particularly testing. We will study different analysis techniques, learn how they work by studying specific algorithms and tools, and discuss applications of the techniques. Our goal will be to explore the current research issues in this cutting edge area, to learn how to build software analysis tools, and to understand how these techniques can be applied to software development activities.

We will primarily focus on applications for testing software, including automatic test data generation. We will also consider using analysis techniques for other software related activities such as maintenance, reuse, metrics, and optimization. Some of the specific analysis techniques to be studied are parsing, software representation methods (control flow graphs, data flow graphs, program dependency graphs), symbolic evaluation, constraints, program slicing, software coupling, and testability.

While we will consider analyses in various languages (e.g. applying to native x86 binaries, Java and JavaScript), there will be a particular emphasis towards Java analysis. Students will complete a research project involving dynamic analysis of Java or JavaScript applications.


## Pre-requisites
Students are required to have previously taken a compilers course, operating systems course, or software engineering course. This class also assumes working knowledge of programming in Java, C/C++ and/or JavaScript (although most assignments will use Java). Students must also have working understanding of UNIX shell environments.

## Intended Audience
This graduate-level course is intended for students who would like to prepare for research in software engineering and other adjacent fields of computer science (for example: security and systems). This course is also designed to be highly applicable to students who are *not* interested in pursuing a career in research, but who would like to become more productive software engineers in industry. If you are an *undergraduate* and are interested in taking this course, please [contact Professor Bell](mailto:j.bell@northeastern.edu) to discuss registration options.

## Term Project 
This seminar-style class will include a term project, worth 50% of your grade. The project will involve a hands-on application of the techniques and tools that we discuss in class, and can be completed either individually or in a group of at most two. Topics for the project will be discussed in the first three weeks of class, and your specific project topic will be finalized soon after. At a high level, project topics will include two tracks:

1. Research track: These projects are intended for students interested in pushing the state-of-the-art in program analysis and software testing tools. These projects will involve creating a prototype tool that implements some new concept. Example research projects might include: extending [Phosphor](https://github.com/gmu-swe/phosphor) or optimizing or enhancing [EvoSuite](http://www.evosuite.org/). If you would like to create a formal writeup of your research project, most of these projects will be suitable for publication in an academic workshop.
2. Industry track: These projects are intended primarily for students who are interested in honing their programming and software engineering skills, by taking a state-of-the-art idea, and transitioning it to the state-of-practice. These projects will involve taking some idea that we talk about in class and implementing it in a popular tool, for instance, improving the [flaky test detector built into Maven](https://maven.apache.org/surefire/maven-surefire-plugin/examples/rerun-failing-tests.html), optimizing the [JaCoCo coverage tool](https://www.jacoco.org), or hacking on the regression test selection system in [Clover](http://openclover.org). These projects are intended to be of sufficient quality that you can submit a pull request to have your changes included in the open source tool. Think about next time you are on an interview and you are asked if you are familiar with (insert name of tool here), and you can say "Oh yes, and I have contributed code that made its way in to X."

