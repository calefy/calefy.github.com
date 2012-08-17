---
layout: post
title: Jekyll建站之旅
summary: 本来想给别人一个jekyll建站的0基础教程，不过在查找相关资料以便描述得更准确的时候发现了很多人已经写过了，而且很好，于是就偷懒了。简单的写了下相关的知识点，以及一些自己想要扩展功能的心得，文章中链接了我认为不错的教程，希望能帮你少走弯路。
category: jekyll
tags: [jekyll, tutorial]
---

{{ page.title }}
================

这是一个相当困难的过程⋯⋯

一周前，我还不知道Jekyll，也没有听说过markdown，尽管它们早就在那里已经很多年了。甚至查询文档，大多都是几年前的东西，自己的无知暴露无疑。只关注与日常工作相关的内容，渐渐成了井底之蛙。

看看我遇到的纠结问题，或许会让你少走些弯路。


意外邂逅
--------

忘记是搜索什么问题了，偶然的撞进了一个人的博客，看到他的文章列表里很多都是在讲 Octopress 的东西，很好奇的进入，于是看到了 Jekyll ⋯⋯

鉴于对庞大 Wordpress 那么丁点的了解，很自然的认为 Octopress 也一样，庞大的要付出很多学习成本，即使会用了，对底层不够了解，心里仍然会觉得空落落的不放心。于是果断的准备要用它的本源 Jekyll 去构建自己的博客。

恶补各种知识⋯⋯


开始：补充基础知识点
--------------------

最全面的资料莫过于官网的 [Jekyll Wiki][jekyllwiki] 了。强烈建议把它的 Home 页上所有的链接地址都点一遍，涵盖了 Jekyll 博客的所有知识。如果本文中有哪些Jekyll用到的知识没有介绍到链接地址，那么在这个Wiki中都可以找到。

开始自己对 Jekyll 相关知识的了解匮乏而混乱，发现哪儿不知道了，就去搜素、查找，对 Jekyll 没有一个全面的认识。等所有的都看过了，回过头来才发现 [Jekyll Wiki][jekyllwiki] 才是最全面的。⋯⋯从零开始，理解这些内容将是一个庞大的工程。

- Git & GitHub & GitHub Pages

	这些内容在之前使用 GitHub 的时候就有了解了，如果不明白可以查看 [GitHub的帮助文档](http://help.github.com/)。

	关于 Git，可以查看专门的Git书： [Pro Git](http://progit.org/book/)。中文版有 [《Git权威指南》][gitprozh] ，但是中文版没有免费的线上文档。

	关于 GitHub， [《Git权威指南》][gitprozh] 的作者蒋鑫有关于它的另一本书 [GotGitHub](http://www.worldhello.net/gotgithub/) （中文 & **免费** ）。感谢作者的分享。

	GitHub Pages 是 GitHub 提供的托管服务，可以建立自己的主页，如果有域名还可以指向这里。在上面提到的 GotGitHub 的 *3.5* 章有详细的介绍。

- Markdown

	这一语法比HTML更加简单明了，而且以纯文本的方式书写，更让人只关注写作的内容，激发极大的写作欲望。阅读 [markdown简体中文语法讲解][markdown] ，15分钟就足够搞定了。

	GitHub 上大多数的内容都是以Markdown写就，它对原生语言做了改进，叫做 *[GitHub Flavored Markdown (GFM)](http://github.github.com/github-flavored-markdown/)*。其中我认为最有用的改进是将语法高亮以 ```` ``` ```` 括起来，不再使用 `highlight language` 这样繁琐的方式表示，而且对排版支持也更好。也有人在讨论 [怎么在自己的网站上使用](http://ruby-china.org/topics/966)，可以去看一下。

- Liquid Template

	Jekyll 网站中采用的是 Liquid 模板语言，找一个对比的话就像 PHP 使用 Smarty，而这两者也有很多相似的地方。如果熟悉模板语言，入手将很快。貌似这个模板语言不是很流行，能搜索到的文档也不是很多，更没有看到一篇中文的内容。

	Jekyll 对 [原生的Liquid标记](https://github.com/shopify/liquid/wiki/liquid-for-designers) 作了一些扩展，增加了一些 [自己的标记和过滤规则](https://github.com/mojombo/jekyll/wiki/Liquid-Extensions)。

- Ruby

	Jekyll 是用 Ruby 语言写的转换文件成网页的软件，关于 Ruby 的写法和 Jekyll 的细节我没有详细去了解，只是把它当一般的软件使用足矣。



搭建：基于Jekyll Bootsharp建站
------------------------------

开始这个站点是通过拷贝 <http://jekyllbootstrap.com> 这个网站提供的github模板，但是并不是这个网站的代码。这个网站也是官方Wiki上推荐的教程，有详细的解释、示例和帮助文档。

**如果是国人建的网站肯定不会把简称定义为JB的:-<*

对我来说，开始的过程还是喜欢用实例一步步构建，发现了一份迟到的文档 [一步步构建Jekyll网站][buildJekyllStepbyStep] ，这是一篇译文，但是对初学者理清概念相当不错，值得多读几遍。

仔细研究 Jekyll Bootsharp 的代码以对整个Jekyll网站有全面的了解，发现 *_layout* 中的文件绕了一个大圈去引用 *_include* 中定义的 [Twitter Bootstrap](http://twitter.github.com/bootstrap/) 中的内容，感觉很不爽。于是根据教程放弃了这个框架，开始从最基本的文件构建，一切简简单单，去掉一切我不需要的东西，把一切都置于掌控之中，感觉很棒。


写作：没有GFM的纠结
-------------------

网站建好了就开始写些东西。Markdown真的很让人有写作的欲望，一方面简单得几乎完全是我要写的东西，去掉一切样式，另一方面则是语法的优美，虽然不是完全的所见即所得，但能够看到的仍然是那么优美。

在GitHub上写评论都用它的GFM语法，尤其是 ```` ``` ```` 语法高亮让我痴迷，也想让自己的网站加上这个东西，但是几番折腾后，我还是放弃了。GitHub Pages 不允许用户运行自己的后台代码，因此基于 *.rb* 的插件都不能使用了。

如果不用GitHub Pages托管自己的网站，可以用这个插件：[Jekyll plugins][jekyllplugin] ，它可以支持Markdown应用 ```` ``` ```` 来定义语法高亮。另外作者还专门针对写插件的过程做了介绍：[How to extend the Redcarpet 2 Markdown library?](http://dev.af83.com/2012/02/27/howto-extend-the-redcarpet2-markdown-lib.html) （GitHub上很多内容都是基于Redcarpet2来转换），有兴趣可以研究一二。我甚至跑去问这个插件的作者怎么办，他说目前只能先生成html再上传了，郁闷！

这个过程中还发现这里有关于[怎么在Marked中使用GFM语法的介绍](http://support.markedapp.com/kb/how-to-tips-and-tricks/using-marked-with-github-flavored-markdown-and-syntax-highlighting)，我甚至还比较这个插件和上面插件的源码，他们都基于对Redcarpet的自定义，看得懂，试了几次却不知道怎么改==!!

**另外过程中还发现，RDiscount比Redcarpet似乎更好使，对自动转换格式更符合标准的Markdown语法*

鉴于以上种种，最后不得不退回到最原始的 `highlight language` 的方式，这种方式目前我发现的最大缺点就是不能把整个代码块缩进排版，其实是对上下的关系没能解析对。先忍了，大不了注意点儿


总结
----

无论如何，纠结之后，能改的都改了，不能改变的只能默默忍受，期待大牛们的开发了。

对整个过程再做下总结：

1. 阅读[一步步构建Jekyll网站][buildJekyllStepbyStep] ，对整个要做的事有基础的了解。
2. 根据上一步不懂的去查阅资料，只要两个地方就很齐全了：
    - 官方Wiki： <https://github.com/mojombo/jekyll/wiki>
	- 官方指定的教程：<http://jekyllbootstrap.com> ，更深入的API信息可以在这里找到。
	- 如果哪个地方还不清楚可以查看本文中出现的相关资料

*补充：在我搜索如何设置A记录和CNAME时，发现这里还有一篇从0开始建站的博客文章 [使用Github Pages建独立博客](http://beiyuu.com/github-pages/)*




[githubhelp]: http://help.github.com/ 'GitHub帮助文档'
[jekyllwiki]: https://github.com/mojombo/jekyll/wiki  'Jekyll Wiki'
[gitprozh]: http://www.worldhello.net/gotgit/ '《Git权威指南》官网'
[buildJekyllStepbyStep]: http://chen.yanping.me/cn/blog/2011/12/15/building-static-sites-with-jekyll/
                         '详细讲解一步步建立Jekyll站点过程'
[jekyllplugin]: http://dev.af83.com/2012/02/27/howto-extend-the-redcarpet2-markdown-lib.html
                '增加GFM功能的插件'
[markdown]: http://wowubuntu.com/markdown/
            'Markdown语法简体中文版'


