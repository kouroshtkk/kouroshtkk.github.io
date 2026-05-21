---
layout: default
title: Home
---

[Home](/) | [About](/about/)

# Recent Posts

{% for post in site.posts %}
- [{{ post.title }}]({{ post.url }}) — *{{ post.date | date: "%B %d, %Y" }}*
{% endfor %}
