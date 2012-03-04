---
layout: default
title: Calefy
---


{% for post in site.posts %}

## [{{ post.title }}]({{ post.url }}) <time>{{ post.date | date_to_long_string }}</time>

标签： {{ post.tags | join: ', ' }}

{{ post.content | strip_html | truncatewords: 15 }}

[全文阅读 &raquo;]({{ post.url }})

----

{% endfor %}


