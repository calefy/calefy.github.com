---
layout: page
title: Calefy
---

阅读 [Jekyll 快速开始](http://jekyllbootstrap.com/usage/jekyll-quick-start.html)

示例和文档说明：[Jekyll Bootstrap](http://jekyllbootstrap.com)

## 更新作者属性

在 `_config.yml` 中更改自己的信息：
    
    title : My Blog =)
    
    author :
      name : Name Lastname
      email : blah@email.test
      github : username
      twitter : username

主题模板会在需要的时候引用这些信息。
    
## 示例文章

该博客包含示例文章，该示例帮助演示网页和博客数据。
当你不再需要示例文件时，删除 `_posts/core-samples` 文件夹。

    $ rm -rf _posts/core-samples

这是示例的文章列表。

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

## 待办

这套主题模板尚未完成。如果你愿意帮助完成，欢迎[加入(fork)](http://github.com/plusjade/jekyll-bootstrap)！我们需要用更好的示例模板展示用户指南。


