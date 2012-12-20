---
layout: post
title: 再谈手机浏览器中输入框焦点莫名丢失问题
category: mobile
summary: 输入框焦点莫名丢失，这次不止是iphone手机上出现，在基于android系统的浏览器上也发现了这个问题。尽管表现不尽相同，但却是同一个bug。究其原因，是输入框先触发touchend事件，然后再根据点击位置触发focus事件，如果点击位置下没有了输入框，就无法触发focus事件了。
tags: [mobile browser input bug, 输入框焦点丢失]
---

{{page.title}}
==============

上次发现 [iOS 5 系统的浏览器输入框丢失](/2012/11/17/input-bug-in-ios5-phone-browser.html) ，只是简单地修改了监听事件，本以为只是 iOS 5 的个例，忽然发现基于 android 的手机浏览器中，如果用 focus 事件，也会引发类似的问题：输入框看得到，但是焦点却不知跑哪儿去了。

通过输出日志、注释掉一部分代码等，最终定位到输入框位置改变的部分。试着将位置改变方式修改为通过 `position:relative;top:Npx` 定位的方式，也是同样的效果，但是将这个移动的值设地比较小，就不会触发这个bug了。

对比输入的日志，原来手机上的事件触发顺序是 touchend -> focus（PC上是 focus -> click），仔细观察手机上的触摸反应，点击后，当手抬起时，手指下面会有个阴影区，覆盖的是原来的输入框位置，就像是点击链接时触发的背景色块一样，但是输入框已经挪了位置。如果设置为 `top:-30px;`，手点击输入框的上半部分，可以获得焦点，但是点击下半部分，就不会有焦点。

根据现象推理，touchend触发后，会寻找点击的位置，如果有对应的input，就继续触发下面input的focus事件，否则就没有其他事件触发了。就好像事件流是捕获式的，而不是冒泡的方式，同时还严格依赖元素的位置。或许苹果 iOS 6 的改进注意到了这一点。

解决的办法还是一样的，只监视触摸事件，在事件最后触发 focus 事件。针对有些特别的浏览器无法响应触摸事件的，就监视 focus 事件。在代码中通过一个变量控制，仅执行一种事件处理，保证响应事件的触发仅是其中一个。

{%highlight javascript%}
// 标记执行过的事件类型
// 执行一个事件句柄后，其他类型的事件就不再执行
if (this._runnedThisHandler && this._runnedThisHandler !== evt.type) {
	return;
}
this._runnedThisHandler = evt.type;
{%endhighlight%}


*后记*

*看到有些参考资料上讲到，手机上的webkit，如果触发mousemove事件时页面变化了，就不再触发其他事件，这个bug应该就是类似的问题。*
