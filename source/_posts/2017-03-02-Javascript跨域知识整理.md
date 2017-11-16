---
title: Javascript跨域知识整理
date: 2017-03-02 00:05:51
tags:
- Javascript
- 前端
---

> 引用出处：
> [浅谈跨域以及WebService对跨域的支持](http://www.cnblogs.com/yangecnu/p/introduce-cross-domain.html)
> [浅谈WEB跨域的实现（前端向）](http://www.cnblogs.com/vajoy/p/4295825.html)

<!-- more -->

## 写在前面

跨域问题是 javascript 在同源策略下产生的，javascript 只能访问和操作自己所在域的资源（相同协议、主机名、端口号）。

在以前，前端代码和后端代码混杂在一起，javascript 直接调用系统里面统一的 Httphandler 这样是不存在跨域问题的。但随着现在技术的发展，多客户端、前后端分离等开发环境使得 javascript 使用到跨域获取数据的情况越来越多，所以，下面来说说 javascript 跨域要怎么实现。

需要说明的是，同源策略是 javascript 里面的限制，其他语言如 c#、java 是可以调用其他 webserver 的。

传统的跨域解决方案是 JSONP(JSON with padding) 和 CORS(Cross-origin resource sharing)，其中，JSONP 需要客户端和服务端一起配合，CORS 则只需要在服务端进行处理就可以了。

---

## JSONP

在同源策略下，某个服务器是没有办法获取到服务器以外的数据的，iframe，img，script等标签是例外，这些标签可以通过 src 属性去请求其他服务器上的资源，JSONP 就是利用到了 script 标签的 src 来调用跨域的请求。

举例如下：服务端和客户端约定传一个名为 callback 的参数来使用 JSONP 功能，比如常规的请求地址如下：

```
http://www.example.net/sample.aspx
```

该地址的返回结果可能是一个单纯的 json 字符串，比如：

```json
{ foo: 'bar' }
```

当加上 callback 的参数之后，即开启 JSONP 功能的时候。

```
http://www.example.net/sample.aspx?callback=mycallback
```

服务端会使用传入的 callback 参数对返回的数据进行处理：

```javascript
mycallback({ foo: 'bar' })
```

这样子就能够获取到跨域的数据同时使用本域的回调函数进行处理。

JSONP 跨域方式比较方便，也支持各种比较老的浏览器，但是缺点很明显，只支持 GET 的方式提交，GET 方式对请求的参数长度有限制，有些情况下可能不满足需求。

---

## CORS(Cross-origin resource sharing)同域安全策略

CORS 同域安全策略是 W3C 在05年提出的跨域资源请求机制，它要求在当前域的相应报头添加 Access-Control-Allow-Origin 标签，从而允许该域的站点访问当前域上的资源。

服务器端在返回的相应报头中添加 Access-Control-Allow-Origin 的标签，该标签的值可为 URL 或 * ，星号表示允许所有域访问当前域。

```javascript
// 服务器端配置
require("http").createServer(function(req,res){
  //报头添加Access-Control-Allow-Origin标签，值为特定的URL或“*”
  //“*”表示允许所有域访问当前域
  res.setHeader("Access-Control-Allow-Origin","*");  
  res.end("OK");
}).listen(1234);
```

对应域的客户端只需要正常的发出 GET 或 POST 请求即可获取到返回的 OK 信息。

```javascript
// 浏览器端使用
$.ajax({
    url:"http://127.0.0.1:1234/",
    success:function(data){
        $("div").text(data)
    }
})
```

要注意的是，CORS 默认只支持 GET/POST 这两种 http 请求类型，要开启 PUT/DELETE 这两种方式的话需要在响应报头添加 Access-Control-Allow-Methods 标签。

```javascript
// 服务器端配置
require("http").createServer(function(req,res){
  res.setHeader("Access-Control-Allow-Origin","http://127.0.0.1");
  res.setHeader(
    "Access-Control-Allow-Methods",
    "PUT, GET, POST, DELETE, HEAD, PATCH"
  );
  res.end(req.method+" "+req.url);
}).listen(1234);
```

不过 CORS 的缺点是对浏览器的版本有要求，不是所有版本的浏览器都支持，

![CORS兼容情况](/311221165148630.jpg "CORS兼容情况")

---

## XDR-IE8

IE8 不支持 OCRS，不过它引入了 XDR特性来实现 CORS 的部分规范（IE11 不再支持 XDR 特性），由于 XDR 使用得很少，具体介绍可查看[原文](http://www.cnblogs.com/vajoy/p/4295825.html#it2)

---

## HTML5解决方案

### WebSocket

WebSocket protocol 是 HTML5 一种新的协议，实现了浏览器和服务器的全双工通信，也允许跨域通讯，是 server push 技术的一种很好体现。

```javascript
// 客户端-原生 webSocket 实现
var ws = new WebSocket('ws://127.0.0.1:8080/url'); //新建一个WebSocket对象，注意服务器端的协议必须为“ws://”或“wss://”，其中ws开头是普通的websocket连接，wss是安全的websocket连接，类似于https。
ws.onopen = function() {
    // 连接被打开时调用
};
ws.onerror = function(e) {
    // 在出现错误时调用，例如在连接断掉时
};
ws.onclose = function() {
    // 在连接被关闭时调用
};
ws.onmessage = function(msg) {
    // 在服务器端向客户端发送消息时调用
    // msg.data包含了消息
};
// 这里是如何给服务器端发送一些数据
ws.send('some data');
// 关闭套接口
ws.close();
```

服务端继续使用 Node.js 来编写，并使用 socket.io 模块辅助，它封装了 webSocket 接口，并对不支持 webSocket 的浏览器提供了向下兼容（如替代为 Flash Socket/Comet）。

服务端要先安装 socket.io 模块

```javascript
// 服务端脚本
var io = require('socket.io');
var server = require("http").createServer(function(req,res){
    res.writeHead(200, { 'Content-type': 'text/html'});
}).listen(1234);
io.listen(server).on('connection', function (client) {
    client.on('message', function (msg) { //监听到信息处理
        console.log('Message Received: ', msg);
        client.send('服务器收到了信息：'+ msg);
    });
    client.on("disconnect", function() { //断开处理
        console.log("Server has disconnected");
    })
});
```

但 socket.io 提供的接口与原生的有所区别：

```html
<!-- 客户端-socket.io 实现 -->
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title>socket.io</title>
    <script src="jq.js"></script>
    <script src="https://cdn.socket.io/socket.io-1.3.4.js"></script>
</head>
<body>
Incoming Chat:
<ul></ul>
<br/>
<input type="text" />
<script>
    $(function () {
        var iosocket = io.connect('http://127.0.0.1:1234/'),
                $ul = $("ul"),
                $input = $("input");
        iosocket.on('connect', function() {  //接通处理
            $ul.append($('<li>连上啦</li>'));
            iosocket.on('message', function(message) {  //收到信息处理
                $ul.append($('<li></li>').text(message));
            });
            iosocket.on('disconnect', function() { //断开处理
                $ul.append('<li>Disconnected</li>');
            });
        });
        $input.keypress(function (event) {
            if (event.which == 13) { //回车
                event.preventDefault();
                iosocket.send($input.val());
                $input.val('');
            }
        });
    });
</script>
</body>
</html>
```

webSocket 在使用 socket.io 来解决兼容之后，能够很好地摆脱无状态的 http 连接，更好的处理连接断开、数据错误等情况。
