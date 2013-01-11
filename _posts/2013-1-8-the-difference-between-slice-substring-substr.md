---
layout: post
title: slice/substring/substr 方法的差异
summary: 字符串的 slice()/substring()/substr() 三个方法都是从字符串取一个子集，JavaScript 手册和很多文章都有说明，但如此相近的功能，还是很容易混淆，希望能找到一种更简便的方法记住它们的用法。
category: javascript
tags: [slice/substring/substr区别]
---

{{page.title}}
==============

{{page.summary}}

> `slice([start[, end]])`
>
> `substring([start[, end]])`
> 
> `substr([start[, length]])`

JavaScript 中，索引一般都是从 0 开始计数，而范围都是包含开头不包含结尾。如 `Math.random()` 的范围是 `0 <= value < 1`，数组中的方法和这里的三个字符串方法返回的都是包含 start 但不包含 end 的一段。（如果正则表达式也算进来的话，那还是有个例外的，正则表达式的量词表达的意思两头都包含，如`{1,3}`表示其前面的正则表达式因子可以重复 1/2/3 次。）

### 测试

为了区分三者的差异和对各种情况的适用情况，测试条件主要有三个：

- 不同的参数个数
- 不同类型的参数
- 不同范围的参数

比较特别的是 `substring()` 会把两个参数中小的作为开始索引，大的作为结束索引，而不管传入的顺序，并且会把负数作为 0 处理；`substr()` 的第二个参数代表的是长度而不是索引值，因此第二个参数如果是负值，会被作为 0 处理。

这些测试条件不是互相孤立的，如给出两个数字参数，就包含了个数、类型、范围。这里仅给出一些具有代表性的典型。

{%highlight javascript%}
var s = 'Hello,world!';
// 参数个数
s.slice();              // "Hello,world!"
s.slice(5);             // ",world!"
s.slice(5, 8);          // ",wo"
// 参数类型
s.slice('5');           // ",world!"
s.slice('5abc');        // "Hello,world!"
s.slice(true);          // "ello,world!"
s.slice(false);         // "Hello,world!"
s.slice(new Array());   // "Hello,world!"
s.slice(Function);      // "Hello,world!"
// 参数范围
s.slice(-5);            // "orld!"
s.slice(-5,5);          // ""
s.slice(5,-5);          // ",w"
s.slice(5,60);          // ",world!"
s.slice(-60,5);         // "Hello"
s.slice(-Infinity,Infinity); // "Hello,world!"
// 混杂
s.slice(true,5);        // "ello"
s.slice(null,60);       // "Hello,world!"
s.slice(NaN,NaN);       // ""
// 特别
s.substring(5,-5);      // "Hello"
s.substring(-5,'34abc');// ""
s.substr(-5,'3ab');     // ""
s.substr(-5,6);         // "orld!"
{%endhighlight%}

上面的这种调用同样适用于另两个方法。

经过这些测试，不难总结出：

- 不接收参数时，都返回整个字符串
- 接收的参数不是数字时，会被自动转换成数字类型；转换后是 NaN 的将作为 0 处理
- 接收的参数或转换成数值后为负：
  > `slice()`的任一参数、`substr()`的第一个参数，会被认为是从后面倒数的位置(如-2表倒数第2个字符的位置)
  >
  > `substr()` 的第二个参数为负，会被转换为 0
  >
  > `substring()` 会把它们转换为 0
- 接收一个参数时，都返回从 start 开始及其之后的部分
- 接收两个参数时，有了范围，则返回指定的范围，超出部分不予理会；若指定的范围不包含任意字符，则返回空字符串

更多的时候，对于这三个方法的差异性讨论集中在参数为负值的情况下。只要参数表示的意思是索引值，那么除了 `substring()` 会把负值作为 0 来使用，其他的都是将负值作为倒数的标识，这时倒数起始是从 1 开始的。不管按索引值还是倒数，超出的范围会被忽略。这三个方法最不济也会返回一个空字符串。

### 另一种理解

对于负值，有一种解释说是负值加上整个串的长度，就是实际的索引位置。这个解释在参数的绝对值不大于整个字符串的长度时有效。基于这个想法，可以把这些方法的类似行为理解为先进行了一次大小转换：

{%highlight javascript%}
/**
 * 获取字符串子串函数，在内部对参数转换方式理解方式
 */

// 假设传入的两个参数是 a、b

var len = this.length, // this指字符串，记录总长度
	convertNumber, t;

// 1. 转换成数值类型
a = a >> 0;
b = b >> 0;

// 2. 转换到有效范围内（不小于 0 ，不大于总长度）
// - 第一个参数默认为 0，第二个参数默认为 length
// - 参数如果是负数，表示从后向前数（负值 + 总长度），超出下标则认为是 0
// - 参数如果是正数，超出下标则认为是 length
convertNumber = function (n, length) {
	if (n < 0) {
		n += length;
		if (n < 0) {
			n = 0;
		}
	}
	else if (n > 0 && n > length) {
		n = length;
	}
	return n;
};
// slice()
{
	a = convertNumber(a, len);
	b = convertNumber(b, len);
}
// substring() 负数都变为 0，再转换到len范围内，并排序
{
	a = convertNumber(Math.max(a, 0), len);
	b = convertNumber(Math.max(b, 0), len);
	if (a > b) { // 排序 
		t = a;
		a = b;
		b = t;
	}
}
// substr() 第二个参数表示长度，只能为正
{
	a = convertNumber(a, len);
	b = convertNumber(Math.max(b, 0), len - a);
}

// 3. 根据取值在0~len之间的 a 和 b，截取字符串
// ...
{%endhighlight%}

字符串中的 `slice()` 与数组中的同名方法类似。`substring()` 就像它的名字一样，只是截取一段字符串，它的行为感觉更符合我对截取函数的理解，但是参杂了其他方法类似但不同的使用方式，反而显得格格不入了。无论哪种实现，更希望对于同类的问题提供一种思路，避免混淆。


**`substr()` 已经被废弃，不属于 ECMAScript 规范，不建议再使用。**

