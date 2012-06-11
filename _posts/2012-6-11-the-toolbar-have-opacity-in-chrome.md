---
layout: post
title: Chrome中滚动条透明原因及解决
category: css
tags: [ chrome bug, toolbar opacity ]
---

{{ page.title }}
================

浮层中的滚动条在 Chrome （版本号是19）上会变透明，而且整个滚动条的颜色都跟着变化了。这应该是该版本的 Chrome 的 bug 吧，同样的东西已经在线上很长时间了，这个bug忽然间就出现了。只有 windows 系统上的 Chrome 会出现这个问题，而 Mac 下则完全没有问题。

导致问题的原因很简单，只是因为产生滚动条的元素的上层元素中设置了半透明。

正常情况，在未设置半透明时，如图1所示。

![图1 未设置半透明时状态](/i/2012-6-11-01.png) <br/> *图1 未设置半透明时状态*

上图浮层所应用的代码结构：

{% highlight html %}
<style type="text/css">
#box{
	position:absolute; top:100px; left:100px;
	width:600px; height:400px;
	overflow:auto;
	border:10px solid #fff;
}
#article{
	background:#333; color:#ddd; line-height:1.4;
}
</style>

<div id="box">
	<div id="article">
		<!--足够多的正文内容-->
	</div>
</div>
{% endhighlight %}

一旦给 `#box` 添加css样式 `opacity:.95;` （IE下用 `filter:alpha(opacity=95)`），我们期望的是整个浮层透明度为 95%，如图2所示。但在 Chrome 19 下表现却如同图3，右侧的滚动条透明度却很特别，不是我们设定的那个程度的透明度。

![图2 设置半透明95%时期望正常状态](/i/2012-6-11-03.png) <br/> *图2 设置半透明95%时期望正常状态*

![图3 设置半透明95%时Chrome19的怪异状态](/i/2012-6-11-02.png) <br/> *图3 设置半透明95%时Chrome19的怪异状态*

