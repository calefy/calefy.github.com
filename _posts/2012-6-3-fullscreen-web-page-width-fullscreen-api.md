---
layout: post
title: Fullscreen API 全屏显示网页
summary: Fullscreen API 可以实现网页上某个部分的全屏显示。如果页面上有想让读者看更大的内容，这个API将是你的不二选择，本站的文章都可以全屏显示。它只有有限的几个方法和属性，但浏览器的实现和表现上还是有差别的，本文就将这个API和实际中遇到的问题一一列出。
category: javascript
tags: [ javascript, fullscreen ]
---

{{ page.title }}
================

第一次看到应用 Fullscreen API 全屏显示网页，是 FaceBook 中的照片放大。作为一个比较新的 API，目前只有 Safari、Chrome 和 FireFox 三种浏览器支持该特性。因为尚未发布正式版的标准，所以必须使用浏览器特定的方法，也就是应用添加前缀（webit/moz）的方法。

这个 API 不仅能够使整个页面全屏显示，也可以使页面中的某个元素全屏显示。它的设计初衷是为了全屏显示 HTML5 视频和游戏，以便更全面的替代 flash 功能。尽管还有很多有待完善的地方，但是作为一个新的浏览器特性，在某些地方还是能够极大地增强用户体验。


1\. 标准调用方式
----------------

要对某个元素使用全屏特效，标准的流程是：

1. 调用这个元素对象的 `requestFullscreen()` 方法；
2. 浏览器将元素全屏显示，改变相关的属性值，然后触发 document 的 `fullscreenchange` 事件；
3. 退出全屏时有两种方式，一种是默认的按 `ESC` 键退出，一种是调用 document 的 `exitFullscreen()` 方法；
4. 浏览器将元素退出全屏显示，改变相关属性值，再次触发 `fullscreenchange` 方法。

浏览器在改变全屏状态时修改的相关属性，是指修改当前全屏状态有否、全屏显示的元素对象，这些属性都是只读的。

浏览器触发 `fullscreenchange` 事件，默认不做任何处理，内部的处理函数需要编程人员自行判断当前全屏状态后，进行相应处理。

对应的，规范中还添加了一个 `:fullscreen` 伪类，对当前全屏的元素进行样式定义。


2\. 封装API
------------

Fullscreen 目前只有两个方法：进入全屏、退出全屏，三个属性（全部是只读的）：是否支持全屏、当前全屏状态、当前全屏元素，以及一个在全屏状态改变时触发的事件（ [Using full-screen mode][2] 中提到还有一个 `fullscreenerror`，但是我没有测试出如何才能触发这个事件 ）。与 [W3 草案][1] 相比，FireFox 的实现更符合标准，而 webkit 内核浏览器中的方法则要自我很多。

所有的方法和属性中，只有 `requestFullscreen()` 是 *element* 对象的方法，其他全部是 *document* 对象所有的方法和属性。

###2\.1 进入全屏：`element.requestFullscreen()`###

将 *element* 全屏显示。webkit内核浏览器和Firefox表现不同，前者只要求element是DOM元素即可，后者则要求DOM必须是文档流中的元素，比较严格，否则不能全屏显示。

出于安全考虑，全屏状态下默认是不允许用户输入的。webkit 内核浏览器会阻止除方向键、控制键之外的键盘输入，FireFox 会在输入时发出要求用户退出全屏状态的提示。前者可以通过在方法 `webkitRequestFullScreen()` 中传入参数 `Element.LLOW_KEYBOARD_INPUT` 允许用户输入，但 Safari 一旦传入该参数，整个 Fullscreen 功能都会坏掉（这应该是 Safari 的一个bug）；后者直接就可以输入，除了有个烦人的提示。

webkit 浏览器中可以通过只读属性 `document.webkitFullScreenKeyboardInputAllowed` 查看当前是否允许全屏状态下的输入。

{% highlight javascript %}
/**
 * 标准化 requestFullscreen 方法
 * @param {DOM} elem 要全屏显示的元素(webkit下只要是DOM即可，Firefox下必须是文档中的DOM元素)
 */
function requestFullscreen( elem ) {
	if (elem.requestFullscreen) {
		elem.requestFullscreen();
	}
	else if (elem.webkitRequestFullScreen) {
		// 对 Chrome 特殊处理，
		// 参数 Element.ALLOW_KEYBOARD_INPUT 使全屏状态中可以键盘输入。
		if ( window.navigator.userAgent.toUpperCase().indexOf( 'CHROME' ) >= 0 ) {
			elem.webkitRequestFullScreen( Element.ALLOW_KEYBOARD_INPUT );
		}
		// Safari 浏览器中，如果方法内有参数，则 Fullscreen 功能不可用。
		else {
			elem.webkitRequestFullScreen();
		}
	}
	else if (elem.mozRequestFullScreen) {
		elem.mozRequestFullScreen();
	}
}
{% endhighlight %}


###2\.2 退出全屏：`document.exitFullscreen()`###

从全屏状态中退出。目前实现的方法都是 `cancelFullScreen()` ,而不是标准的 `exitFullscreen()`。

{% highlight javascript %}
/**
 * 标准化 exitFullscreen 方法
 */
function exitFullscreen() {
	if (document.exitFullscreen) {
		document.exitFullscreen();
	}
	else if (document.webkitCancelFullScreen) {
		document.webkitCancelFullScreen();
	}
	else if (document.mozCancelFullScreen) {
		document.mozCancelFullScreen();
	}
}
{% endhighlight %}

###2\.3 浏览器是否支持全屏：`document.fullscreenEnabled`###

通过该属性的boolean值判断浏览器是支持 Fullscreen 功能。

webkit 内核的浏览器目前还没有该属性，因此只能通过能力判定来判断是否支持全屏显示功能。Firefox 已经有了对应的属性定义。

{% highlight javascript %}
/* 标准化 fullscreenEnabled 属性 （只读） */
document.fullscreenEnabled = ( function() {
        var doc = document.documentElement;
        return ( 'requestFullscreen' in doc ) ||
               ( 'webkitRequestFullScreen' in doc ) ||
			   // 对Firefox除了能力判断，还加上了属性判断
               ( 'mozRequestFullScreen' in doc && document.mozFullScreenEnabled ) ||
			   false;
    } )();
{% endhighlight %}


###2\.4 ：`document.fullscreenElement`###

当前全屏显示的DOM元素。

{% highlight javascript %}
/**
 * 标准化 fullscreenElement 属性 （只读）
 * 以同名方法替代
 */
function fullscreenElement() {
	return document.fullscreenElement ||
           document.webkitCurrentFullScreenElement ||
           document.mozFullScreenElement ||
           null;
}
{% endhighlight %}


###2\.5 当前全屏状态：`document.fullscreen`###

该属性并未在2012/6/3的 [W3草案][1] 中出现，但在 [Using full-screen mode][2] 一文中介绍了该属性。其值为 *boolean* 类型，判断当前文档的全屏状态。

如果最终去掉这个判断全屏状态的属性，我们仍然可以通过 `document.fullscreenElement` 的值是否为 *null* 来判断全屏与否

{% highlight javascript %}
/**
 * 标准化 fullscreen 属性 （只读）
 * 以同名方法替代
 */
function fullscreen() {
	return document.fullscreen ||
           document.webkitIsFullScreen ||
           document.mozFullScreen ||
		   false;
}
{% endhighlight %}

###2\.6 全屏状态改变事件：`fullscreenchange`###

该事件要绑定在 *document* 上，该事件仅在全屏状态改变时触发，默认没有任何动作。

{% highlight javascript %}
/* 绑定 document 的 fullscreenchange 事件 */
document.addEventListener(
	'fullscreenchange', // webkitfullscreenchange/mozfullscreenchange
	function( evt ){
		//todo 全屏状态改变时的时间处理。
		//默认不会有任何处理，需要自己判断当前屏幕全屏与否，做出相应处理。
	},
	false
);

/* 如果使用 jQuery ： */
$( document ).bind(
	'fullscreenchange webkitfullscreenchange mozfullscreenchange',
	function(){
		//todo code
	}
);
{% endhighlight %}


3\. 全屏样式设置
----------------

标准中，通过 `:fullscreen` 伪类对全屏的元素进行样式定义。

默认情况下，浏览器只会简单地将元素设置为全屏显示。如果该元素全屏后，高度比屏幕还高，超出的部分将会被隐藏。为了将超出部分可以滚动显示，最顶层全屏显示的元素要特别设置：

{% highlight css %}
position : fixed;
top      : 0;
left     : 0;
width    : 100%;
height   : 100%;
overflow : auto;
{% endhighlight %}

一般情况下，要全屏显示的元素是不能像上面这样设置的。那么我们可以变通下，设置一个 `<div/>`，包围要全屏的元素，然后将这个 `<div/>` 设置为全屏，上面的样式定义就可以定义在这个 `<div/>` 上，相应的，`:fullscreen` 将会作用在这个 `<div/>` 上。这样，过长的元素就可以在这个包围层内滚动显示。


4\. 特别注意
------------

- 目前 FireFox 10、Safari 5.1+、Chrome 15+ 支持全屏
- 可以使任意元素全屏显示，不只是整个页面
- 全屏只能从事件触发（用户操作），而不能用代码直接触发
- 全屏状态下，webkit 内核浏览器默认会阻止除方向键、控制键之外的键盘输入，FireFox 会在输入时发出退出全屏状态提示。处理方法在 *封装API* 部分有说明。

下面是实际中遇到的需要注意的地方：

- 全屏状态切换需要时间。执行 `requestFullscreen()` 后，并不会立即进入全屏状态，对应的全屏属性不会立即更改，而是有一个执行时间。因此，只能在 `fullscreenchange` 事件触发后才代表进入了全屏状态。但是在 `fullscreenchange` 事件中调用 `$(window).width()` 并不总能得到全屏的尺寸，这个现象很奇怪。如果需要屏幕尺寸，可以通过 `window.screen.width` 来获得。
- 涉及修改DOM文档须注意代码位置。当用 `<div/>` 包围要全屏显示的元素时，这段 javascript 代码不应该在要全屏显示的元素内部，否则这段代码会被执行两遍，而且第二遍不会在断点中被监视到，原因将在后文详细描述。
- `ESC` 键不同系统功能不同。目前发现点击 `ESC` 退出全屏时，mac系统不会再额外触发键盘事件，但是win7系统下出发 `fullscreenchange` 事件后还会立马触发键盘事件，因此如果还有不希望被触发的键盘事件，可以设置一个监视变量，在很短时间后再修改监视变量，以错过这个立马执行的时间。


5\. 未涉及功能
--------------

- iframe 元素的 `allowfullscreen` 属性
- ::backdrop 伪类
- 具体其他细节可以参考 [W3 草案][1]


6\. 结语
--------

Fullscreen API 毕竟目前只是草案，尚未形成正式的标准，况且各个浏览器的实现情况也不完全相同，甚至细节上的实现差别更可能引发预想不到的问题。但作为渐进增强方式使用的新功能，能够极大的增强用户体验。仍要根据规范的完善，不断改进我们的代码。

详细代码可以参考：<https://github.com/calefy/calefy.github.com/blob/master/js/Fullscreen.js>


参考
----

1. [W3 草案 2012/6/3版][1]
2. [Using full-screen mode][2]
3. [Enhance Your Website with the FullScreen API][3]
4. [Using the Fullscreen API in web browsers][4]
5. 代码参考 [jQuery Fullscreen 插件](https://github.com/martinaglv/jQuery-FullScreen/blob/master/fullscreen/jquery.fullscreen.js "jQuery Fullscreen Plugin")



[1]: http://dvcs.w3.org/hg/fullscreen/raw-file/tip/Overview.html "W3 draft"
[2]: https://developer.mozilla.org/en/DOM/Using_full-screen_mode "mozilla fullscreen"
[3]: http://tutorialzine.com/2012/02/enhance-your-website-fullscreen-api/ "Enhance Your Website With the FullScreen API"
[4]: http://hacks.mozilla.org/2012/01/using-the-fullscreen-api-in-web-browsers/ "Using the Fullscreen API in web browsers"

