---
layout: post
title: Javascript键盘事件及输入控制
summary: 进行web开发时，我们会对表单做提交验证，为了进一步增强用户体验，要在用户输入时就进行限制，比如针对电话只能输入数字、长度不允许超过固定值等。随着微博的流行，从Twitter开始，很多记录的文字输入在设计时就都添加上了字数限制与提示的功能，这已成了输入框的标配。本文从这一需求出发，说明Javascript中键盘事件和输入控制的技术细节。
category: javascript
tags: [键盘事件, 字数控制, oninput, onpropertychange]
---

{{page.title}}
=============

{{page.summary}}

## 功能需求

限制输入功能一般满足下面的需求：

- 统计还可以输入多少文字并实时显示
- 限制的字数可能是以汉字为单位计数，这时要判断单字节、双字节的输入
- 可能还要求达到限制数量后，输入框不可输入，或仅允许输入固定类型（如果用keyup事件，输入框内容会跳动）

这里讨论实现这些需求的方案，基于我们日常使用的标准键盘和最常见的四种内核的浏览器，对于一个web应用，这些兼容就足够了。

*下面讨论中仅以事件名称代替，不加前缀‘on’*

## 解决方案

为了实时监控文字的多少，需要对每个输入进行监控，每次改变都应该被纳入到监控中。Javascript中的键盘事件只是监控每次按键，但是对于右键粘贴、拖拽方式改变输入框的内容就无能为力了，我们只能去探寻其他事件。

对字数和类型的限制，原理上都是检查输入的是否符合要求，不符合就把不允许的部分删除，重新给输入框赋值。但是只要用过键盘事件就会发现，keydown、keypress 事件无法获取刚输入的内容，而 keyup 又貌似控制晚了：总是看到字符被输入又再被删除。

什么事件能够满足这种需求呢？那就是 input 和 propertychange 事件。如果你知道这两个事件，一切都不是问题，只要监控它们，既能得到新输入的内容，又能随时控制内容，而不担心文字显示又删除的过程展示在用户面前。

{%highlight javascript%}
$('textarea').bind('input', function () {
	var value = $(this).val();
	// todo: 处理截断 or 对比是否超出限制
});
{%endhighlight%}

但我还是想看看用键盘事件能够做到哪一步。

{%highlight javascript%}
/* 因为中文不同浏览器返回不同，这里只考虑键盘上字符的输入 */
var input = $('textarea'),
	limitLength = 20, // 限制字符长度
	markForCountWhenKeyup = false; // 标记需要keyup后才能计数(限制输入时控制删除键)

// 绑定事件
input.bind('keydown', keydown)
	.bind('keyup', keyup);
	
// keydown事件
function keydown (evt) {
	// 键盘上的全部可打印键
	var isPrintableKey = function (k) {
			return !(evt.altKey || evt.ctrlKey || evt.metaKey) && (
				   k === 32 ||              // 空格
				   (k >= 48 && k <= 57) ||  // 数字
				   (k >= 65 && k <= 90) ||  // 字母
				   (k >= 96 && k <= 105) || // 小键盘数字
				   // 其他符号键
				   (k === 106 || (k >= 186 && k <= 192) || (k >= 219 || k <= 222))
				   ); 
		};

	switch (evt.keyCode) {
	// 1. 回车键，除了shift+Enter执行默认换行，其他Enter都是提交
	case 13: 
		if (!evt.shiftKey) {
			// todo: 提交处理
			evt.preventDefault(); // 防止换行
		}
		break;
	// 2. 删除键/中文，在keyup后再计算字数
	case 8:   // backspace
	case 46:  // delete
		markForCountWhenKeyup = true;
		break;
	// 3. 输入键，在原内容基础上加1 (这时文本框的值还未改变)
	default:
		if (isPrintableKey(evt.keyCode)) {
			// 超出范围，不再允许将该次键值输入
			if (input.val().length + 1 - limitLength > 0) {
				evt.preventDefault();
			}
		}
	}
}
// 配合keydown，只有退格、删除键才响应，检测字数。(这时文本框的值已经改变)
function keyup (evt) {
	var k = evt.keyCode;
	if (markForCountWhenKeyup && (k === 8 || k === 46 )) {
		// 允许删除
		markForCountWhenKeyup = false;
	}
}
{%endhighlight%}

上面的代码满足键盘输入控制，但是仅能处理键盘上看得到的字符的输入，一旦遇到中文、粘贴、拖拽改变输入框内容时，就无能为力了。


## 技术探究

### 输入元素

HTML中一般用到的输入文字的元素是文本框，包括input、textarea。富文本编辑不在讨论之列。

### js中的键盘事件

js中的键盘事件只有三种：keydown、keyup、keypress。它们触发的顺序是：keydown -> keypress -> keyup。当按下一个键不放开，一般会重复地触发 keydown+keypress，直到放开后触发一个 keyup 事件。

#### 三种键盘事件区别

keydown 和 keyup 是比较底层的事件，分别对应一个键的按下与放开，键盘上任何一个键被点按，都会触发这两个事件。keypress 更接近用户，只有可打印的字符和控制字符（如换行）被键入才触发。

keydown 和 keyup 被触发后，都会产生一个虚拟键盘码，用来表示按下的是哪个键。键盘上的每个键只对应一个虚拟键盘码。keypress 被触发后产生的是实际键入的字符的 Ascll 编码，而没有对应的虚拟键盘码。这两种编码有时在数值上是相等的。

如何访问到事件产生的编码信息，就又要涉及到浏览器兼容问题，几大浏览器处理是不同的。

#### 获取按键信息

在 IE 中，事件对象在 `window.event` 中，编码信息放到了 `keyCode` 属性内。`keyCode` 属性的含义根据事件变化，在 keydown 和 keyup 事件中表示的是虚拟键盘码，在 keypress 时表示的就是实际字符的编码。

非IE浏览器内，事件对象在回调函数中返回，存储在事件对象的 `keyCode`、`which`、`charCode` 中，看到了吧，相比 IE 中增加了两个属性。

`keyCode` 只表示虚拟字符编码，因为 keypress 事件没有该编码，在这个事件里，`keyCode` 的值是 0。`charCode` 只在 keypress 事件中有值，为实际键入的字符编码，其他事件中都是 0。

`which` 更像是 IE 中的 `keyCode`，当 keydown 和 keyup 事件中有虚拟键盘码时，`which` 表示虚拟键盘码；当 keypress 事件中只有字符编码时，`which` 又表示字符编码。

上面是 firefox 浏览器的标准表现形式。webkit 内核浏览器中（safari、chrome），相比 firefox，在 keypress 时，`keyCode` 不再是 0，而是和 `which`、`charCode` 两个属性的值相等。opera 浏览器相比 webkit 内核的浏览器，三个 key 事件都去掉了 `charCode` 属性。

当按下小写字母a键时，对应关系如下：

{%highlight javascript%}
            keyCode   which     charCode
--ie6/7/8:------------------------------ 
keydown       65    undefined  undefined
keypress      97    undefined  undefined
keyup         65    undefined  undefined
--ie9/10:------------------------------
keydown       65       65         0
keypress      97       97         97
keyup         65       65         0
--firefox:------------------------------
keydown       65       65         0
keypress      0        97         97
keyup         65       65         0
--webkit:-------------------------------
keydown       65       65         0
keypress      97       97         97
keyup         65       65         0
--opera:--------------------------------
keydown       65       65         0
keypress      97       97         97
keyup         65       65         0
{%endhighlight%}

*除了老版的 IE、firefox，其他浏览器的表现是一致的* <br/>
*jQuery 为 IE 事件增加了 which 属性值*

因此，如果只是 keydown 和 keyup 事件，都访问事件对象的 `keyCode` 即可；如果是 keypress 事件，就要区别对待了，IE 中还是使用 `keyCode`，非 IE 要使用 `which` ，这时他们表示的是实际输入的字符码，通过 `String.fromCharCode()` 获得实际的可打印字符。

#### 组合按键

除了单个按键，还有组合按键，如很多快捷键是由功能键和其他键组合而来。这些按键除了 opera，都比较一致：

- `ctrlKey` --- Ctrl 键
- `shiftKey`--- Shift 键
- `altKey` ---- Alt 键
- `metaKey` --- command 键（Mac下）

#### 中文支持情况

各个浏览器对中文的支持不同，中文输入法下，不同浏览器下按键触发的键盘码是不同的。但它们都不产生 keypress 事件。

- webkit 内核浏览器：每按一个键都产生 keydown、keyup；keydown 时，可打印字符的键盘码都是 229；keyup 时返回正常的键盘码值。
- firefox 浏览器：输入中文的中间过程中不产生任何键盘事件；开始输入的第一个按键产生正常的 keydown，结束的 空格或回车 产生 keyup，键盘码是正常的键盘码。
- opera 浏览器：Window 系统下与 webkit 表现一致；Mac下每次按键都只产生 keydown 事件，键盘码是正常键盘码。
- ie 浏览器：6~10几个版本触发方式与键盘码值与 webkit 内核浏览器一致。

总之，多数浏览器中文输入法(搜狗输入法)一般开始都是299，但是有些会因系统有差异。用键盘事件控制中文输入不太靠谱。

### 输入框内容改变事件

form 表单中很多元素都有 change 事件，当表单元素的值发生变化时，就会触发该事件。只能是由用户触发的改变才会发出 change 事件，用脚本修改值是不会产生的，form的很多事件都是如此，包含本文提到的。但是在输入框中，需要焦点离开，改变的状态才会触发这一事件，也就是输入时每次的改变我们是无法知道的，只有在输入框外点击一下，如果内容变化了，才会发出 change 事件。

幸运的是，还有一个 input 事件，会在每次输入框的内容改变时触发，无论是敲键盘输入，还是粘贴文本，甚至是拖拽，只要内容值变化了，就会触发该事件。这个事件是 HTML5 的标准事件，除了老版 IE 基本都支持，它一般藏的比较深，很少会看到关于它的介绍。在 Chrome console 中的自动提示中可以看到一个长长的事件列表，说不准哪些将来我们就会用到了。

![Chrome console中提示的textarea支持的事件](/i/2012-11-12-01.png) <br/>*Chrome console 中提示的 textarea 支持的事件*

针对不支持的老版 IE6/7/8，还有一个替代事件：propertychange。看它的名字就知道它监视的是属性值的改变，而输入值 `value` 只是一个元素众多属性中的一个，因此这个事件返回对象需要过滤出我们需要的属性 `value`。

有一点要特别注意的是，IE9/10 同时支持 input 和 propertychange，如果两种事件都注册了，就都会触发，可能会被重复处理。

### 事件触发顺序

当我们把键盘事件和内容改变事件放到一起，他们触发的顺序是： keydown -> keypress -> input/propertychange -> keyup。

这个顺序特别的地方仍然发生在 IE 中。IE9/10 对两个内容改变事件都支持，但它们的触发顺序却又不同。IE9 是先触发 input，然后才是 propertychange，而 IE10 这个过程正好相反。

当我们敲击可打印的键，它表示的字符是什么时候输入到文本框中的呢？

根据测试观察，文本框中的值在 keypress 事件中还没有改变，但在 input 中已经改变了。在 input 中改变输入框的值，看上去没有任何不妥；如果放到 keyup 事件中再去改变，能看到文字敲进去了然后又被改变了，跳动很明显。在 input 的时候，文本框的值已经改变，但是尚未渲染显示出来，这时用脚本改变文本框中的值，不会出现内容跳动情况。如果在 input 的事件处理函数中设置断点，就能看到内容已经改变，所以我们在这里面获取到的值是改变后的值，只是没有呈现出来。没有看源码与文档，仅从现象推理，希望看到相关资料的能不吝分享。

### IE9 的特例

IE9，而且是当前系统中 IE 最高版本为 9 时，敲击退格和删除键，都只能触发 keydown -> keyup 事件。因为这两个键是不可打印的，因此正常情况应该只是不触发 keypress 事件。这个 bug 导致我们的处理程序还要加上对 IE9 这个问题的特别处理。

这个 bug 只存在当前系统最高 IE 版本是 9 的情况下，如果在 IE10 的开发者工具模拟，则不会出现该问题。

## 结语

要实时监控输入框内容变化，用 input(propertychange) 会简单有效。但不要忘记对 IE9 的特别处理。

keydown、keyup 这两个键盘事件可以直接用 `event.keyCode` 取得键盘码值，keypress 要得到实际的字符码，需要 `event.keyCode || event.which`，然后可以用 `String.fromCharCode()` 获取实际的字符表示。最后，各个类型的浏览器及其不同版本可能会有些差别 :( 。

中文的输入，测试中发现不同系统、不同版本都会有影响，无法用键盘事件来控制。


## 补充

2012-12-20:

对于限制文本框中输入特定的字符，如某些字段只允许数字，还有一种方式，即用 keypress 的 charCode 获取键入的值，判断是否是要求输入的，而不再去获取输入框的值（keypress时，输入框内尚未赋值）。

{%highlight javascript%}
// 限制 input 输入框中只能输入数字
input.bind('keypress', function (evt) {
	// 取得键入的实际字符。
	// 这里用 keyCode 替换 charCode，同样可以获得想要的值(参见前文)
	var char = String.fromCharCode(evt.keyCode);

	// 如果不是数字，就不允许输入
	if (!/\d/.test(char)) {
		evt.preventDefault();
	}
});
{%endhighlight%}

如果是键入，这个简单的处理函数可以很好地工作，但是一旦用输入法、粘贴、拖拽等，就不再能工作了。

还是要回归到 input 事件。


测试代码：<http://jsfiddle.net/calefy/uBYWH/>
