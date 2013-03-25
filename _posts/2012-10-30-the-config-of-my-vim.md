---
layout: post
title: vim配置+插件
category: vim
summary: 一直用vim作为日常工作的编辑器，越用越是感觉离不开它。虽然只是用一些基本的命令，高级用法尝试一下不用也就忘记了，但足以满足日常的需要了。使用中插件也会不断地增加，这里记录下vim配置和使用到的插件，以备后查。
tags: [vim配置]
---

{{ page.title }}
================

{{page.summary}}

### 配置文件

`.vimrc` 很简单的配置，着重在配色和字体上。

{%highlight vim%}
set encoding=utf-8
set fileencoding=utf-8
set fileencodings=ucs-bom,utf-8,chinese,cp936
set guifont=Consolas:h15
language messages zh_CN.utf-8
set lines=45 columns=100
set number
set autoindent
set smartindent
set tabstop=4
set autochdir

set shiftwidth=4
set foldmethod=manual

colorscheme Monokai
set nocompatible
set nobackup
{%endhighlight%}

#### 配色

之前使用自带的 desert 颜色主题，这是所有自带颜色主题中觉得最舒服的颜色。[sublime][1]编辑器出现后，面对漂亮的配色主题，很是羡慕。它可以支持vim模式，但是因为只能支持utf-8文件不得不放弃。现在好了，有了类似[sublime][1]配色的vim颜色文件：[vim-monokai][2]。

> 1. 把`Monokai.vim`放到vim安装目录的`colors`文件夹内（mac下可以放到`~/.vim/colors/`中）
> 2. 在配置文件中配置 `colorscheme Monokai`
> 3. 配置 `set guifont=Consolas:h15`，字号根据自己的习惯调整

### 插件

#### NERDTree

在左侧展示树形目录结构的插件。

在vim中输入命令：`:NERDTree`，就会在左侧展示当前位置的目录结构。它还有很多定位、打开方式等快捷键，可以在NerdTree窗口按`?`查看具体的命令。一般我都是将光标停到要打开的文件，然后回车，就在右侧窗口打开文件了。 

#### bufexplorer

切换文件的小插件，非常好用。

通常我们打开很多文件，如果想要切换刚刚编辑的文件，用`:bn`(下一个)、`:bN`(上一个)，或者在后面加上数字来标记第几个文件，但是修改地多了，怎么可能记得住。使用这个插件就很多简单了，只要按下`\be`，就会看到所有缓存的文件，找到要查看的文件，回车即可查看。`d`可以删除光标所在文件的缓存。

#### zencoding

zencoding方式快速书写html+css，一般用来写html，css用的很少。安装了这个插件，只要按zencoding的语法写代码，写完后按`<C+y>,`，即可生成对应的代码串。

常用代码缩写方式：
> - 自动生成html5结构：`html:5`
> - `.className`, `#idName` 设定类名和id
> - `[attribute=value]`表示元素的属性值
> - `*N`表示重复N次，`$`表示如果多个自动增加
> - `>`表示子级，`()`分组一个层级的元素
> - 选中一部分内容然后`<C-y>,`，输入要包围的tag
>
> 如：div#box>(p>a+span>em)+ul>(li.item_$*3>(p.title>a+span*2)+p.info>input[value=hello])

#### markdown

vim中markdown语法的高亮文件。高亮的配色效果和折叠一般，有时还不是很准。毕竟markdown本身就是为了纯文本诞生的东西，有一些配色、折叠算是聊胜于无了。具体的安装可以看 [vim中设置markdown语法高亮][3]。


### 结语

只是简单地记录下自己的配置，插件也是简单描述，具体的可以去Google。以后有新的好插件会随时加上。


[1]: http://www.sublimetext.com/ 'sublime'
[2]: https://github.com/sickill/vim-monokai 'vim-monokai'
[3]: /2012/03/01/set-vim-markdown-syntax-highlight.html 'vim-markdown语法高亮'
