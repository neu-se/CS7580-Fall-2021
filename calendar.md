---
layout: page
title: Calendar
description: Listing of course modules and topics.
---

# Calendar

Generally, we will discuss one research paper each class meeting, with coding activities mixed in where appropriate. A listing of papers and topics will be posted closer to the start of the term.

 {% for module in site.modules %}
 {{ module }}
 {% endfor %}
