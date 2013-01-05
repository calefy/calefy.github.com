---
layout: post
title: undefined与null的类型转换
summary: undefined和null这两个值的类型转换，与其他值的转换有些不同。
category: javascript
tags: [类型转换, undefined, null]
---

{{page.title}}
==============

- 强制转换为布尔类型，两者都是 false
- 强制转换为数值，undefined 返回的是 NaN，null 返回的是 0。用 `isNaN()` 验证，undefined 不是数字，null 是数字。
- 特别地：`undefined == null` 结果为 true
- 比较操作符 `==` 不会自动转换 undefined 和 null

{%highlight javascript%}
// 操作符 `==` 不会自动转换 undefined 和 null
undefined == false // false
undefined == true  // false
null == false      // false
null == true       // false

undefined == 0     // false
null == 0          // false

// 操作符会自动转换其他值为数值
true == 1          // true
true == 2          // false
{%endhighlight%}

很少直接用 undefined/null 与布尔值或数字比较，更多的是直接把 undefined/null 本身放到判断条件中。但这种自动转换问题仍然值得我们关注，以免犯下类似的错误。

类似地还要小心，如空串转换成数值时，是 0。其他值转换成数值时，很多时候要经过转换成字符串，但这里还是能找到一些需要小心的：

{%highlight javascript%}
// 下面值转换成数值
[]          // 0
[5]         // 5
[5,6]       // NaN
['']        // 0
['5']       // 5
['abc']     // NaN
[undefined] // 0
[null]      // 0
[undefined,null] // NaN

// 转换成字符串
[]           // ''
[undefined]  // ''
[null]       // ''
[undefined,null] // ','
{%endhighlight%}

基于数组，`==` 这时可以自动转换：

{%highlight javascript%}
[undefined] == 0  // true
[undefined] == '' // true
[null] == 0       // true
[null] == ''      // true
{%endhighlight%}

总之，null 参与数值计算时可以自动转换为 0，undefined 只能转换为 NaN。`==` 不能直接自动转换 undefined 和 null，但是 `undefined == null` 是真值。数组转换成字符串时，作为数组元素的 undefined 和 null 都将被忽略，而数组转换成数字可以看成是基于转换成的字符串再转换为数值。
