---
title: Cookie
date: 2017-06-02 16:19:44
tags:
    - javaScript
    - 前端
categories:
    - 总结
---

Cookie一直以来在我心中，是一个熟悉但是又陌生的东西。在书本知识上，各种面试题等，多多少少都有涉及Cookie相关话题。觉得，好像懂了Cookie，但是也都是纸上谈兵。这次工作中遇到一个涉及登录同步的问题，算是真正走入了cookie的机制。

---
# 什么是Cookie
> HTTP Cookie（也叫Web cookie或者浏览器Cookie）是服务器发送到用户浏览器并保存在浏览器上的一块数据，它会在浏览器下一次发起请求时被携带并发送到服务器上。比较经典的，可以它用来确定两次请求是否来自于同一个浏览器，从而能够确认和保持用户的登录状态。Cookie的使用使得基于无状态的HTTP协议上记录稳定的状态信息成为了可能。

<!-- more -->
## 设置
服务器对任意HTTP请求发送Set-Cookie HTTP头部作为响应的一部分。
```
HTTP/1.0 200 OK
Content-type: text/html
Set-Cookie: name=value
Set-Cookie: name2=value2
```
此请求后，对该服务器发起的每一次新的请求，浏览器都会将之前保存的Cookie信息通过Cookie请求头发送给服务器。
```
GET /index.html HTTP/1.1
Host: www.aaa.com
Cookie: name=value; name2=value2
```
咱们最常见的，登录态的保持。登录成功之后，服务器设置一个对应的cookie值，同时可以设置cookie的过期时间。后面的每一次请求都带着标识登录状态的cookie，服务器通过cookie来判断用户端是否登录。

## js对于cookie的api
获取cookie:
```
document.cookie返回当前域的cookie

BAIDUID=A61E6ADEFBEB1EC3BF31C08C9DBD2A24:FG=1; BIDUPSID=A61E6ADEFBEB1EC3BF31C08C9DBD2A24; PSTM=1491035513; MCITY=-179%3A; pgv_pvi=131430400; locale=zh; BDRCVFR[feWj1Vr5u3D]=I67x6TjHwwYf0; cflag=15%3A3; BD_CK_SAM=1; PSINO=7; BDORZ=B490B5EBF6F3CD402E515D22BCDA1598; BD_HOME=0; H_PS_PSSID=1432_21120_18560_22157; BD_UPN=123253

以上是我在百度主页打印出来的cookie信息。cookie以key=value的形式存在，等号两边不能有空格。两个cookie之间以"; "间隔。
```

设置cookie:
```
document.cookie = "name=aaa";
这时候的cookie值为：
"BAIDUID=A61E6ADEFBEB1EC3BF31C08C9DBD2A24:FG=1; BIDUPSID=A61E6ADEFBEB1EC3BF31C08C9DBD2A24; PSTM=1491035513; MCITY=-179%3A; pgv_pvi=131430400; locale=zh; BDRCVFR[feWj1Vr5u3D]=I67x6TjHwwYf0; cflag=15%3A3; BD_CK_SAM=1; PSINO=7; BDORZ=B490B5EBF6F3CD402E515D22BCDA1598; BD_HOME=0; H_PS_PSSID=1432_21120_18560_22157; BD_UPN=123253; name=aaa"
```
这样的设置方法并不会覆盖原有的cookie，除非设置的cookie名称已经存在；同时，不能用这种方法同时设置两个cookie

---
# Cookie的那些属性
## name
一个唯一确定cookie的名称。不区分大小写。cookie名称必须是经过URL编码的。
## value
存储在cookie中的`字符串值`，值必须经过URL编码的。
## domain
域名可以是，www.example.com(包含本身及子域)，.example.com(仅包含子作用域)。如果没有明确设定，则默认为当前域。
所指定的域名必须是当前发送Cookie的域名的一部分，比如当前访问的域名是example.com，就不能将其设为google.com。
cookie在哪个域中是有效的，则在当前域发送的请求中都会包含这个cookie信息。
## path
对于指定域中的那个路径。只有path属性匹配向服务器发送的路径，Cookie才会发送。默认根路径。
这里的匹配并不是绝对匹配。比如路径为/aaa, 则/aaa/bbb/xxx.html也会算匹配。
## expires
失效时间。表示cookie什么时候应该被删除的时间戳，采用Date.toUTCString()的时间格式。
如果没有设置，或设置为null,默认为当前会话有效。一点浏览器关闭，cookie失效。
如果要手动删除某个cookie,可以将cookie的失效时间设置为一个过去时间。
## secure
该选项只是一个标记而没有值。只有当一个请求通过 SSL 或 HTTPS 创建时，包含 secure 选项的 cookie 才能被发送至服务器。
```    
Set-Cookie: name=aaa; secure
```
## Http-Only
设置Cookie的时候，如果服务器加上了HttpOnly属性，则这个Cookie无法被JavaScript读取（即document.cookie不会返回这个Cookie的值），只用于向服务器发送。

---
# Cookie的限制
*   绑定在特定域名下
*   总数有限制（不同浏览器不通同），超过个数再设置cookie的话，浏览器会清除以前设置的cookie(清除机制因浏览器而异)。
*   尺寸有限制，大多数浏览器限制在4096B以内。尺寸影响的是一个域下所有cookie，而非每个cookie单独限制。尺寸过大，cookie会被悄无声息地丢掉。

---
# Cookie的设置与获取
## 子Cookie

---
# 跨域