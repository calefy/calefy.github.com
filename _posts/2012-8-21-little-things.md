---
layout: post
title: 小技巧积累
category: other
summary: 总结工作中遇到的问题，一些不常见优化方法
tags: [js技巧]
---

{{ page.title }}
================

###1. 数字类型判断方式###

{% highlight javascript %}
// 传统方式
function isNumber(n) {
  return typeof n === 'number'; 
}

// 新的方式
function isNumber(n) {
   return n === +n;
}
{% endhighlight %}

`+n` 通过js的自动类型转换，会将 `n` 变成数字类型，然后和原值进行全等比较，只有原值也是数字类型才有可能相等。

需要特别注意的是 `NaN` 这个特殊的变种。用新的方式 `+NaN` 后仍是原来的值，而它自己不等于自己，因此当参数为 `NaN` 时，新的方式判断是错误的。这点瑕疵相信在大多数程序里面都不会用到而引发问题。

新的方式不仅代码更简短，效率上因为不再是字符串的逐字符比较而有所提升。


###2. 为元素绑定 `keydown` 事件###

`keydown` 或着其他键盘事件只能绑定在可聚焦的元素上，包括输入框等表单元素、链接，必须保证元素获得焦点，这样绑定的事件才能起作用。

要为`div`，`span`等不可聚焦元素绑定事件，需要添加属性`tabindex`，其值为任意整数即可。

