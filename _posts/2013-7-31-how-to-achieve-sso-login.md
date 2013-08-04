---
layout: post
title: SSO (单点登录)实现方式
category: web
summary: SSO (Single-Sign-On) 即单点登录，在互联网应用中是多个站点通过一次登录即可访问所有产品，如Google所有产品通过 http://accounts.google.com/，百度所有产品统一登录地点是 http://passport.baidu.com/ 等，也有些产品是提供自己的登录界面，然后到统一入口验证。总之，就是要实现一次登录，处处登录。
tags: [SSO, 单点登录, 跨域]
---

{{ page.title }}
================

{{page.summary}}

如果所有产品都是在同一主域下，那么只要把登录的标识信息存放到主域的 cookie 中，即可实现访问任一产品的页面时把登录信息传递到服务器，然后服务器根据该信息判断是否需要用户再次登录。虽然大多数公司的产品都是在同一个域名下，但有些还是独立域名的，这时就涉及到跨域问题，也是 SSO 的难点所在。

基本思路是在一个固定的入口登录，成功后返回一个 token，将这个 token 附带在跨域访问的某个文件上，该文件拿到这个 token 和服务器上存放的值比较，并获取对应的登录用户信息，然后设置登录标识 cookie，以此完成 SSO 登录。服务器上的 token 可以存在 memcache 等缓存中，或者通过 RPC 访问。

通过查看现有网站，主要是百度和新浪的 SSO 实现方案（截至2013-7-31），来对 SSO 实现方式有一个详细的了解。

### 百度的 SSO

百度的很多产品都是在 baidu.com 这个主域名下，纳入到 SSO 的其他独立域名目前只有 hao123.com（很讨厌这个网站，一不小心就会在装软件时将它设为默认首页，还好现在不怎么用 IE）。

点击百度产品的登录，一般会跳转到 passport.baidu.com，在这个页面完成登录，也有些是浮层，但登录实现方式一样：

1. 填写完用户名密码后，点击“登录”按钮。
2. js 创建一个 div 容器，在这个容器中主要创建两个元素：form 表单和 iframe。
  >- from 中包含很多 `type="hidden"` 的 `<input/>` 标记，主要包含填写内容和其他相关内容，如要登录的产品标识和成功后的跳转地址等。form 的提交目标窗口指向之后的 iframe。
  >- iframe 初始地址所一个 _blank.html 文件，会造成一次 http 请求。
  >- 这些内容都是拼成一个字符串，一次性写入的。
3. 动态创建节点完成后，马上提交数据，指向 iframe 中 passport.baidu.com/v2/api/?login。
4. 提交返回的结果内容在 iframe 中，通过 js 中的 'location.replace()' 跳转 iframe 到一个新的页面，这个页面是第2步中的表单项 'staticpage' 指定的，一般都叫 v3Jump.hmtl，不同产品只是路径不同。
5. v3Jump.html 中的 js 的作用就是以形如 `parent.someMethod({})` 的方式调用 iframe 父级窗口的 js 方法。涉及到不同域名调用主窗口 js 方法，因此一般 v3Jump.html 都是放在和主窗口同一域名下。
6. 主窗口的 js 处理页面表现，同时为了实现跨域，会调用 https://user.hao123.com/static/crossdomain.php?bdu=...&t=3434312，这个 php 会根据 bdu 值验证登录状态，并设置 hao123.com 这个域名的 cookie 登录标识。比较巧妙的是，百度把这个地址作为一个 Image 对象的 src 属性，这样能完成请求，还不会渲染到页面上。
7. 最后，主窗口的 js 跳转页面到指定的地址。


### 新浪的 SSO

新浪的独立域名好像只有一个 weibo.com，其他产品都是在 sina.com.cn 下。它的登录方式与百度基本相同，这里挑选不同的几点说明。

#### 在 weibo.com 登录
1. 同百度
2. 同百度
3. 提交数据到 iframe, 指向 login.sina.com.cn/sso/login.php，提交表单后，会立马删除 from 节点（百度只在登录失败再次提交时才替换这个 form 节点)
4. 如果登录失败，iframe 中的内容为 `location.replace()` 跳转到 weibo.com/ajaxlogin.php；如果登录成功，则跳转地址为 weibo.com/sso/login.php，该文件返回 302，跳转到 ajaxlogin.php。
5. ajaxlogin.php 负责用形如 `parent.someMethod({})` 方式调用父窗口 js，处理页面。
6. 父窗口 js 删除 iframe 节点
7. 同百度

#### 新浪其他 sina.com.cn 子域登录

1. 同 weibo
2. 同 weibo
3. 同 weibo，但不会删除 form 节点
4. 这里有些不同，login.sina.com.cn 下，login.php 的内容用 `location.replace()` 指向 login.sina.com.cn/crossdomain2.php，由这个页面调用 `parent.someMethod()`；而在新浪首页，login.php 中先设置 `document.domain='sina.com.cn'`(login.php 与页面域名不完全相同)，然后再调用 `parent.someMethod()`。
5. 父窗口的 js 处理页面表现，并用jsonp方式（放到script标记的src中）调用 weibo.com/sso/login.php 来完成跨域登录。
6. 结束

新浪的登录 js 并不统一，明显 weibo 的 js 处理的更精细。

weibo 登录中的第 4 步，iframe 中跳转方式实现对跨域的访问，显然有些局限。如果还有其他独立域名需要访问，这种跳转将无法兼顾。


### 总结

上面两个站点的实现方式基本相同，都是用 iframe 登录，然后在 iframe 中跳转来达到通过同一入口进行登录。难点在于跨域，特别需要注意的是：

- iframe 中的 js 如果要访问父窗口的方法，需要保证域名相同，或者主域相同并都设置domain属性为主域地址，否则浏览器会报安全警告。
- 跨域只要发起跨域的http请求即可，可以放到 script 标记的src中，也可已放到图片对象的 src 中，推荐后者，因为后者不用放到dom节点中。需要注意的是，这两种方式都要带一个每次都不同的参数，可以是时间戳，以防止浏览器的自动缓存。

上面着重关注了访问的跳转顺序，还有一个关键点是如何标识登录状态。上面的方案中，都会在请求跨域文件时，带上登录返回的一个唯一字符串，将它作为一个 token，在后端验证登录，识别登录的用户。可以把这个 token 设置为很短的时间有效，如1分钟，并在访问一次后删除，可以保证一定的安全性，但是如果在使用前截取，那么可以拿到任何一个地方登录。因为上面两个网站对跨域文件的访问都是写在主窗口的js里，因此可以很容易设置断点获取之。

据说 Google 在这点上是安全的，可惜身边没法翻墙，留待以后吧。


************

### 补记

*2013-8-4

周末有空折腾了一下翻墙，找到个超级好东西：[SmartHosts](https://smarthosts.googlecode.com/svn/trunk/hosts)，Youtube/Facebook/twitter... 都可以上了。

Google 所有产品的登录都是通过 https://accounts.google.com/ 进行处理，基本流程如下：

1. 在 Google 产品站点击“登录”，会跳转到 https://accounts.google.com/ServiceLogin 登录；
2. 表单 post 提交到 https://accounts.google.com/ServiceLoginAuth 验证；
3. 登录成功则直接通过 302 转到 https://accounts.google.com/CheckCookie，访问跨域产品的页面，一般是 accounts.hostname/accounts/SetSID，设置登录cookie；
4. 页面跳转回登录来源页。

针对不同的来源，具体处理上在第3步有些差别：

  > - 如果登录来源是 *.google.com 或 youtube.com，则直接通过302指向 accounts.youtube.com/accounts/SetSID，之后该地址再转向到来源地址。
  > - 如果登录来源是 *.google.com.hk，则返回200，通过 jsonp 请求 accounts.google.com.hk/accounts/SetSID 和 accounts.youtube.com/accounts/SetSID

可以看到，基本原理是一样的，都是登录后，访问一下跨域的页面，完成登录信息的cookie设置。

过程中通过修改 accounts.youtube.com 的 ip 指向，截取到了登录后的 /accounts/SetSID 链接，直接在一个新的浏览器中访问，是可以完成登录的。因此，Google 的 token 也不能保证这种截取。
