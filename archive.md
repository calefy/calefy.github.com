---
layout: default
title : 文章列表
---


文章列表
--------

{% for post in site.posts %}

- {{ post.date | date_to_string }} &raquo; [{{ post.title }}]({{ post.url }})

{% endfor %}


----

全站分类
--------

{% for category in site.categories %}

- {{ category[0] }}

{% endfor %}


全站标签
--------

{% for tag in site.tags %}

- {{ tag.[0] }}

{% endfor %}

