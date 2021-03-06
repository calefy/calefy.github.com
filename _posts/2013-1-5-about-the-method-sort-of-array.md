---
layout: post
title: 关于Array中的sort()
summary: sort() 实现的功能很简单，就是对数组进行简单的排序。但在自定义的情况下，它根据传入的函数参数返回结果判断，这个结果要求是一个数字，但在不同浏览器下，自定义会产生一定的偏差，甚至会导致IE下代码终止执行。基于此，对不同浏览器进行测试探讨。
category: javascript
tags: [sort]
---

{{page.title}}
==============

{{page.summary}}

## 概述

在原数组中对数组元素进行排序，没有创建新的数组。

### 不带参数调用

不带参数调用，将按照字符编码顺序对数组中的元素进行排序。即先把数组元素转换为字符串，再进行比较排序。这个排序默认是升序的，字符编码越大的越排到后面。但这里面有一个特例：`undefined`，它会一直被排到最后，不参与字符串字符编码值的比较（`null`/`true`/`false` 会被转换成对应的字符串，然后参与比较）。

{%highlight javascript%}
var arr = [3, 14, 1, null, false, undefined, 'uzdefined',
		   {x: 12},
		   [2, 3],
		   function(){}];
arr.sort();
// arr: 
// [1, 14, [2,3], 3, {x:12}, false,
//  function(){}, null, "uzdefined", undefined]
{%endhighlight%}

这个例子中包含了我们日常用到的所有类型，除了 `undefined` ，其他都遵循了比较转化成字符串后的编码值。其中 `[2, 3]` 字符串化后是 `"2,3"` ，`{x: 12}` 字符串化后是 `"[object Object]"`，函数字符串化后就是函数定义本身。

### 传入比较函数

`sort()` 接受一个函数作为比较函数，这个函数接受两个值作为参数，比较它们，返回一个数字表明它们的相对顺序。

接受的两个参数 a、b，比较结果如下：

- 小于0，即 `a < b`，a 应该排在 b 的前面
- 等于0，即 `a == b`
- 大于0，即 `a > b`，a 应该排在 b 的后面

`undefined` 即使是在使用了自定的排序中，也会一直排到最后，因为** `undefined` 不会传递给排序函数**。

{%highlight javascript%}
var arr = [5,null,undefined,13];
arr.sort(function(a,b){
	console.log(a, b, '=', a - b);
	return a - b;
});
console.log(arr);
// 输出：
// 5 null "=" 5
// 5 13 "=" -8
// [null, 5, 13, undefined]
{%endhighlight%}

从这个例子中，可以看到，null 在运算中会被自动转换成数字 0 ，参与排序运算，但 undefined 根本就没有被传入比较函数中。


## 特别注意

### 排序不稳定性

JavaScript中的这个排序方法是不稳定的，即对相等的元素，两个元素的顺序不一定是一致的。

当比较函数返回 0 时，ECMAScript标准并没有指定是否两个元素不交换顺序，因此，并不是所有浏览器实现都保证保持原来的顺序（比如2003版本的Mozilla）。

这个特点对针对对象的某一项属性排序时，可能有影响。比如：

{%highlight javascript%}
var arr = [
	{name: 2, content: 34},
	{name: 2, content: 35},
	{name: 2, content: 36},
	{name: 2, content: 37}
];
{%endhighlight%}

这时只针对每个数组元素的 `name` 属性排序，因为值都是 `2`，因此排序后的新数组，可能会受不稳定性的影响，导致新的顺序中的 `content` 不能再按顺序排列。

这个问题应该只针对老版本的实现，起码在 IE6 下还没有测试出这个问题。

关于更老版本可能带来的问题，参见 [Mozilla关于sort的技术文档](https://developer.mozilla.org/zh-CN/docs/Core_JavaScript_1.5_Reference/Global_Objects/Array/sort)

### 比较函数返回非数字值

自定义比较函数要求返回一个数字，但是如果返回的结果是一个非数字时，多数浏览器都能继续工作，返回的排序结果不再能保证正确顺序。但是在IE系列中，还是有些特别。IE6 中，只要返回的内容不能转换成数字，就会终止代码的执行。（测试中，最高只有IE8版本的机子，IE7/8中，只要非NaN的返回值如果不能转换成数字，代码也会终止，但是升级IE8到IE9后，通过开发者工具模拟 IE7/8，这个现象消失了，不过IETester中的模拟仍然能够重现。IE9无论返回什么都能继续执行程序）

因此，针对比较函数，请确保返回值一定是一个数字，否则可能不能按预期排序，甚至在 IE 尤其是 IE6 代码可能停止而不报错。

### 性能

暂不讨论这个方法的性能问题，猜想或许有原生的效率优势。

## 总结

针对 `sort()` 有以下几点要注意：

- 针对原数组排序，不产生新的数组
- 默认排序按字符编码值升序排列
- 排序对于相同的值，在老版本的实现中缺乏稳定性（规范未指明）
- 自定义比较函数，要确保返回值是数字，以避免可能导致的排序混乱和IE下代码终止问题

排序测试部分代码示例：<http://jsfiddle.net/calefy/m9ztx/>

