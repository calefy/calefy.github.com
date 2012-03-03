---
layout: post
title: Jekyll建站之旅
---

{{ page.title }}
================

这是一个坎坷艰难的过程⋯⋯

一周前，我还不知道Jekyll，也没有听说过markdown，尽管它们早就在那里已经很多年了。甚至查询文档，大多都是几年前的东西，自己的无知暴露无疑。只关注与日常工作相关的内容，渐渐成了井底之蛙。

看看我的曲折过程，或许会让你少走些弯路。


意外邂逅
--------

忘记是搜索什么问题了，偶然的撞进了一个人的博客，看到他的文章列表里很多都是在讲 Octopress 的东西，很好奇的进入，于是看到了 Jekyll ⋯⋯

鉴于对庞大 Wordpress 那么丁点的了解，很自然的认为 Octopress 也一样，庞大的要付出很多学习成本，即使会用了，对底层不够了解，心里仍然会觉得空落落的不放心。于是果断的准备要用它的本源 Jekyll 去构建自己的博客。

恶补各种知识⋯⋯


基础知识
------------

最全面的资料莫过于官网的 [Jekyll Wiki][jekyllwiki] 了。强烈建议把它的 `Home` 页上所有的链接地址都点一遍，涵盖了 Jekyll 博客的所有知识。如果本文中有哪些Jekyll用到的知识没有介绍到链接地址，那么在这个Wiki中都可以找到。

开始自己对 Jekyll 相关知识的了解匮乏而混乱，发现哪儿不知道了，就去搜素、查找，对 Jekyll 没有一个全面的认识。等所有的都看过了，回过头来才发现 [Jekyll Wiki][jekyllwiki] 才是最全面的。⋯⋯从零开始，理解这些内容将是一个庞大的工程。

- Git & GitHub & GitHub Pages

	这些内容在之前使用 GitHub 的时候就有了解了，如果不明白可以查看 [GitHub的帮助文档](http://help.github.com/)。

	关于 Git，可以查看专门的Git书： [Pro Git](http://progit.org/book/)。中文版有 [《Git权威指南》][gitprozh] ，但是中文版没有免费的线上文档。

	关于 GitHub， [《Git权威指南》][gitprozh] 的作者蒋鑫有关于它的另一本书 [GotGitHub](http://www.worldhello.net/gotgithub/) （中文 & **免费** ）。感谢作者的分享。

	GitHub Pages 是 GitHub 提供的托管服务，可以建立自己的主页，如果有域名还可以指向这里。在上面提到的 GotGitHub 的 `3.5` 章有详细的介绍。

- Markdown

	这一语法比HTML更加简单明了，而且以纯文本的方式书写，更让人只关注写作的内容，激发极大的写作欲望。阅读 [markdown简体中文语法讲解][markdown] ，15分钟就足够搞定了。

	GitHub 上大多数的内容都是以Markdown写就，它对原生语言做了改进，叫做 *[GitHub Flavored Markdown (GFM)](http://github.github.com/github-flavored-markdown/)*。其中我认为最有用的改进是将语法高亮以 ```` ``` ```` 括起来，不再使用 `{% highlight language %}` 这样繁琐的方式表示，而且对排版支持也更好。

- Liquid Template

	Jekyll 网站中采用的是 Liquid 模板语言，找一个对比的话就像 PHP 使用 Smarty，而这两者也有很多相似的地方。如果熟悉模板语言，入手将很快。貌似这个模板语言不是很流行，能搜索到的文档也不是很多，更没有看到一篇中文的内容。

	Jekyll 对 [原生的Liquid标记](https://github.com/shopify/liquid/wiki/liquid-for-designers) 作了一些扩展，增加了一些 [自己的标记和过滤规则](https://github.com/mojombo/jekyll/wiki/Liquid-Extensions)。

- hello 



基于Jekyll Bootsharp初建
------------------------



迟到的文档 [一步步构建Jekyll网站][buildJekyllStepbyStep]


没有GFM的纠结
-------------

[Jekyll plugins][jekyllplugin]


结论
----


相关资源
--------



[githubhelp]: http://help.github.com/ 'GitHub帮助文档'
[jekyllwiki]: https://github.com/mojombo/jekyll/wiki  'Jekyll Wiki'
[gitprozh]: http://www.worldhello.net/gotgit/ '《Git权威指南》官网'
[buildJekyllStepbyStep]: http://chen.yanping.me/cn/blog/2011/12/15/building-static-sites-with-jekyll/
                         '详细讲解一步步建立Jekyll站点过程'
[jekyllplugin]: http://dev.af83.com/2012/02/27/howto-extend-the-redcarpet2-markdown-lib.html
                '增加GFM功能的插件'
[markdown]: http://wowubuntu.com/markdown/
            'Markdown语法简体中文版'


