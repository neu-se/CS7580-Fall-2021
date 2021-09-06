---
layout: home
title: Home
nav_exclude: true
seo:
  type: Course
  name: CS 7580, Special Topics in Software Engineering
---

# {{ site.tagline }}
{: .mb-2 }
CS 7580, Fall 2021, T/F 9:50-11:30am, International Village 22
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

The primary objective for this course is for students to learn different methods for analyzing software, with several applications in software engineering, particularly testing. We will study different analysis techniques, learn how they work by studying specific algorithms and tools, and discuss applications of the techniques. Our goal will be to explore the current research issues in this cutting edge area, to learn how to build software analysis tools, and to understand how these techniques can be applied to software development activities. We will primarily focus on applications for testing software, including automatic test data generation.


## Pre-requisites
Students are required to have previously taken a compilers course, operating systems course, or software engineering course. This class also assumes working knowledge of programming in Java, C/C++ and/or JavaScript. Students must also have working understanding of UNIX shell environments.

## Intended Audience
This graduate-level course is intended for students who would like to prepare for research in software engineering and other adjacent fields of computer science (for example: security and systems). This course is also designed to be highly applicable to students who are *not* interested in pursuing a career in research, but who would like to become more productive software engineers in industry. If you are an *undergraduate* and are interested in taking this course, please [contact Professor Bell](mailto:j.bell@northeastern.edu) to discuss registration options.

