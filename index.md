---
layout: default
title: Home
---

Welcome to my notes.

# Recent CTFs

{% assign ctf_pages = site.pages | where: "parent", "pwn.college writeups" | sort: "date" | reverse %}
{% for node in ctf_pages %}
- [{{ node.title }}]({{ node.url }}) - *{{ node.date | date: "%B %d, %Y" }}*
{% endfor %}
