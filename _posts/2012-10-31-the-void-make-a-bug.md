---
layout: post
title: 由void引发的错误
category: javascript
summary: 匹配语法错误在js中并不陌生，从它提示的内容中也很容易明白报错的原因。但是浏览器给出的提示的位置并不很准确，实际上发生错误的地方一般是提示的左近。更有甚者根本就无法找到出错的地方，只能一点点排查。void使用错误引发的就是这样的一个错误。
tags: [void, js错误, Unexpected token]
---

{{ page.title }}
================

`SyntaxError: Unexpected token + SOMETHING` {{page.summary}}

### bug始末

今天解决报上来的一个bug：一个`<a/>`标记作为提交按钮，其他浏览器都正常，只有ie6报了个语法错误就什么都不做了。源代码形如下：

{%highlight html%}
<form id="formId" action="/somepage.php" target="_self">
	...
	<a href="javascript:void()"
	   onclick="$('#formId').submit();">提交</a>
</form>
{%endhighlight%}

在测试时发现Chrome的console中也会报错（`SyntaxError: Unexpected token )`），然后页面就跳转了，如果不看console表现倒是正常。

![Chrome中bug定位](/i/2012-10-31-01.png)*Chrome中bug的错乱定位*

开始还以为`<a/>`或者`<form/>`还有其他注册的事件，于是试着注销事件、拦截冒泡，逐层下来才发现只是`<a/>`本身的原因。与平常不同也就是`href="javascript:void()"`了，平常写的时候是`void(0)`的，修改下再运行，还真就好了。原来只是void这个运算符用错了。

在console中试验下:

- `void` 触发错误: `SyntaxError: Unexpected token }`
- `void()` 触发错误: `SyntaxError: Unexpected token )`

### void 是什么

void在javascript中是一个一元运算符。它总是舍弃运算数的值，返回undefined。

日常很少用到void，通常有两种情景：

- **阻止链接的默认事件**：`href="javascript:void(0)"`（这种javascript伪协议阻止默认事件不推荐，更好的方式通过`event.preventDefault()`或IE下`returnValue = false`）
- **获得一个undefined值**：`void 0`或`void(0)`（undefined并不是保留字，有些浏览器下是可写的）

`void 0` 和 `void(0)` 两种写法都是可以的，后者更像一个函数。javascript中一元运算符都可以用这两种方式书写，但是像上面实验触发错误那样只写一个运算符，或者括弧中什么都没有，他们都会触发同样的错误。这样的一元运算符有：

- typeof: 类型判断
- new: 对象创建符（*你没用过 `new(Object)` 吧？:P*）
- delete: 删除对象的属性、数组元素或变量。浏览器实现和规范有出入，一般只用来删除对象的属性，另外它的返回值也挺有意思，很多时候都是true。
- void: 返回一个真正的undefined

一元运算符后面都可以用括号把表达式括起来，虽然看起来是函数的调用方式，但括号仅仅是括起表达式。

### 话外

关于undefined，一般比较容易看到的写法如下：

{%highlight javascript%}
(function (global, undefined) {
	// ...
})(this)
{%endhighlight%}

这样也能保证在这个自调用函数的作用域空间内，undefined确实是真正的undefined，而不是被定义成了别的值。
