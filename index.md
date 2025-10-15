---
title: "My Blog"
layout: default
---

# My Blog

Latest posts:

<ul>
{% for post in site.posts %}
  <li>
    <span>{{ post.date | date: "%Y-%m-%d" }}</span> —
    <a href="{{ post.url | relative_url }}">{{ post.title }}</a><br/>
    <small>{{ post.description }}</small>
  </li>
{% endfor %}
</ul>
