---
layout: page
title: 侯振永的技术博客
tagline: 山高月小，水落石出
---
{% include JB/setup %}


## 博客列表

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

