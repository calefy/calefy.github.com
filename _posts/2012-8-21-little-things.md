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

需要特别注意的是 `NaN` 这个特殊的变种。用新的方式 `+NaN` 后仍是原来的值，而它**自己不等于自己**，因此当参数为 `NaN` 时，新的方式判断是错误的。这点瑕疵相信在大多数程序里面都不会用到而引发问题。

新的方式不仅代码更简短，效率上因为不再是字符串的逐字符比较而有所提升。



###2. 自调用函数中的this###

自调用函数的this会指向全局，因此自调用函数中的this要小心应用。

**“在Javascript中,This关键字永远都指向函数(方法)的所有者。”**

关于this，只要看包含this的函数到底是哪个对象的函数，就能确定this指向了这个对象，如果没有明确的对象，则默认都是window。这里有两个例外，一是call、apply方式的调用，二是new操作符的调用。如直接声明的函数、自调用函数、定时操作的函数。

> 关于变量作用域：
>
> - 函数内定义的变量和函数，因为预处理，所以无论在函数内何处定义，都相当于在函数开始就进行了声明
> - 通过 `new` 创建的对象，访问其属性时，首先查找其本身的属性，如果没有再去查找其原型链上的该属性，如果还没找到，才定义为 `undefined`
> - 尽管函数的作用域在定义时就决定了，但是如果内部有this的引用，执行的时候就要根据运行时的上下文来确定了。时常见到的比如返回一个函数的函数，还有自调用函数也是一个最需要注意的地方。

{% highlight javascript %}
var number = 5,
	rate = 4;

function C (number) {
	this.number = number || 2;
};
C.prototype.number = 9;
C.prototype.rate = 3;

var obj = {
	'number': 7,
	'rate': 1,
	'compute': (function () {
		c = new C();
		var c;

		this.number = Math.max(c.number, this.number);
		this.rate =   Math.max(c.rate, this.rate);

		return cal;
		
		function cal () {
			this.number *= this.rate;
		};
	})()
};

var compute = obj.compute;

compute();
obj.compute();

alert(window.number + '--' + obj.number);
{% endhighlight %}



###3. 哪些元素可以获得键盘事件###

`keydown` 或着其他键盘事件只能绑定在可聚焦的元素上，包括输入框等表单元素、链接，必须保证元素获得焦点，这样绑定的事件才能起作用。

要为`div`，`span`等不可聚焦元素绑定事件，需要添加属性`tabindex`，其值为任意整数即可。



###4. setTimeout 在线程结束处执行###

`setTimeout(function, 0)` 并不是简单地 0ms 后执行 function ，而是在整个js线程最后执行。

详细的介绍js定时机制的文章：[深入理解JavaScript定时机制](http://www.laruence.com/2009/09/23/1089.html)

总之，js是单线程的，setTimeout方式定时，直接放到线程的最后排队；setInterval还有另一个计时线程，固定的间隔往js线程最后加上处理函数。

如果setTimeout 直接在页面中嵌入，则等页面加载完成，对应的完成事件函数处理完成才开始执行。

测试：<http://jsfiddle.net/calefy/93J5u/>


