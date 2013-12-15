---
layout: post
title: Ubuntu 13.10 下 Jekyll 安装问题
category: jekyll
summary: 公司的项目终于告一段落，有空来整一下许久未动的网站。换用了 ubuntu 13.10，开始折腾 jekyll 在本地运行，结果又有因为依赖的问题，罢工了。试着搜索都没找到专门的解决办法，无意中看到别人的文章，才知道原来所缺少 ruby1.9.1-dev。
tags: [ubuntu 13.10, jekyll, rdiscount]
---

{{ page.title }}
================

{{page.summary}}

机子上的基本环境：

- ubuntu 13.10
- ruby 1.9.3
- gem 1.8.23

首先安装 jekyll：`gem install jekyll`。

运行 jekyll：`jekyll --server`，提示：

{% highlight text %}
You are missing a library required for Markdown. Please run:
  $ [sudo] gem install rdiscount
  [2013-12-15 18:20:37] INFO  WEBrick 1.3.1
  [2013-12-15 18:20:37] INFO  ruby 1.9.3 (2012-04-20) [x86_64-linux]
  [2013-12-15 18:20:37] WARN  TCPServer Error: Address already in use - bind(2)
  [2013-12-15 18:20:37] INFO  WEBrick::HTTPServer#start: pid=10345 port=4000
{% endhighlight %}

直接访问网站 localhost:4000，提示： *no access permission to `/'* 。

这个站点使用的是 `rdiscount` 来解析 `markdown`，因此这个 gem 必不可少。按照提示安装：

{% highlight text %}
$ sudo gem install rdiscount
Building native extensions.  This could take a while...
ERROR:  Error installing rdiscount:
  ERROR: Failed to build gem native extension.

          /usr/bin/ruby1.9.1 extconf.rb
          /usr/lib/ruby/1.9.1/rubygems/custom_require.rb:36:in `require': cannot load such file -- mkmf (LoadError)
            from /usr/lib/ruby/1.9.1/rubygems/custom_require.rb:36:in `require'
              from extconf.rb:1:in `<main>'
{% endhighlight %}

安装不上，这就是问题所在了。

搜索半天无果，幸亏无意中看到了 [Jekyll 的安装](http://havee.me/internet/2013-07/jekyll-install.html)。里面提到：

> 如果遇到 gem 安装错误，可能需要 **安装 ruby1.9.1 的编译扩展组件的头文件** :
>
> `sudo apt-get install ruby1.9.1-dev`

按照这个说明，安装完这个扩展，再执行 `sudo gem install rdiscount` ，很顺利就安装成功了。

