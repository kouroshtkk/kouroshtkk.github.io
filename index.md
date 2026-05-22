---
layout: default
title: Home
---

Welcome to my blog. [Learn more about me here.](/about/)

# Recent ctf writeups

{% assign ctf_pages = site.pages | where: "parent", "pwn.college writeups" | sort: "date" | reverse %}
{% for node in ctf_pages %}
- [{{ node.title }}]({{ node.url }}) - *{{ node.date | date: "%B %d, %Y" }}*
{% endfor %}
