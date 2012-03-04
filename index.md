---
layout: default
title: Calefy
---


文章列表
--------

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

待办
----

- 主题样式尚未完成
- js解析Markdown

