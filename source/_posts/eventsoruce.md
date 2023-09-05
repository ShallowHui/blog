---
title: EventSoruce简介
date: 2023-09-05 14:38:02
tags:
  - javascript
  - EventSource
categories: 技术
cover: https://cdn.jsdelivr.net/gh/shallowhui/cdn/top_img/javascript.png
top_img: https://cdn.jsdelivr.net/gh/shallowhui/cdn/top_img/javascript.png
description: 简单介绍了一种服务器推送消息的技术，EventSource。
---
## 简介

由于HTTP协议只是一个请求和响应的协议，所以实现服务器与客户端之间长时间的实时通信需要其他技术。`WebSocket`应该是较广为人知的一种技术，它定义了一个可以全双工通信的协议，但这篇文章主要介绍的是另外一种技术，`EventSource`。

**EventSource是Web内容与`服务器发送事件`（Server Sent Event，SSE）的一个接口**。一个EventSource的实例对象会与HTTP服务器开启一个持久化（keep-alive）的连接，以`text/event-stream`的格式发送事件消息，此连接会一直保持开启，并且在发生错误时会自动尝试重连，直到通过调用`EventSource.close()`来关闭这个长连接。

具体的API可以查看[MDN文档](https://developer.mozilla.org/zh-CN/docs/Web/API/EventSource)。

### 与WebSocket的比较

简单与WebSocket协议比较一下：

+ 基于的协议：WebSocket在初始化连接时需要先通过HTTP协议来进行协议升级，之后就是基于TCP协议来进行通信的。EventSource就是基于一个HTTP长连接来进行通信。
+ 通信方式：WebSocket是全双工通信，也就是服务器和客户端之间可以互相发送消息。而EventSource只能由服务器发送消息给客户端来进行单向通信，所以叫`SSE`。
+ 复杂性：EventSource是纯文本的简单协议，API比较简单，容易实现。WebSocket除了能发送文本消息，还有自定义数据帧和传输二进制数据等其它功能，API相对比较复杂。
+ 服务器支持：WebSocket需要服务器实现WebSocket协议。而EventSource只需要服务器按`text/event-stream`的格式往HTTP连接中写数据并发送就可以了。
+ 兼容性：EventSource和WebSocket在大多数现代浏览器中都得到了广泛的支持，但WebSocket可能在一些比较旧的浏览器中支持得不太好或者直接不支持。

总结一下，使用EventSource来实现通信比较简单，如果网站只是需要服务器往客户端发送数据，不需要客户端往服务器发送数据，比如天气消息推送服务、股票数据实时更新，那么EventSource就非常适用了。如果需要实现服务器和客户端的交互，进行实时双向通信，或者自定义协议等更高级的功能场景，那就要使用WebSocket。

## 实现SSE

下面简单实现一个SSE的Demo。

### 服务器实现

使用SpringBoot框架搭建一个HTTP服务，在Controller中定义一个专门处理SSE的方法：

```java
@Controller
public class SSEController {

    @GetMapping("/sse")
    public void sse(HttpServletResponse response) {

        System.out.println("客户端连接...");

        response.setContentType("text/event-stream");
        response.setCharacterEncoding("UTF-8");

        try (PrintWriter writer = response.getWriter()) {

            while (!writer.checkError()) {
                double random = Math.random();
                System.out.println("生成随机数字：" + random);
                writer.write("event:test\n");
                writer.write("data:随机数字：" + random + "\n\n");
                writer.flush();
                TimeUnit.SECONDS.sleep(3L);
            }

            System.out.println("客户端断开连接!");

        } catch (IOException | InterruptedException e) {
            e.printStackTrace();
        }
    }

}
```

由于EventSource要建立的是一个HTTP长连接，不能直接return返回数据，这样Spring MVC是会关闭这个连接的，客户端那边的EventSource对象又会自动尝试重连，就会不断地请求这个接口，创建多个连接，所以要用循环来维持这一个长连接，并用`HttpServletResponse`实例对象发送数据。

首先设置`text/event-stream`和`UTF-8`编码的响应头，然后获取`response`对象的字符输出流，在循环中不断地通过`writer.checkError()`方法来检测客户端是否关闭了这个连接，如果连接没有关闭，就随机生成一些数据写入到输出流中，发送给客户端。

注意`write()`方法中写入数据的格式，EventSource定义一个事件由4个字段组成：`event`、`data`、`id`、`空行`，也就是说在EventSource的连接中，一个事件消息的数据格式应该是这样的：

```text
event:{事件类型的名字}
data:{消息内容}
id:{事件id}

event:{事件类型的名字}
data:{消息内容}
id:{事件id}

event:{事件类型的名字}
data:{消息内容}
id:{事件id}
```

**每个事件消息中间都要有个空行作为分隔标识**。`event`字段是事件类型的名字，与EventSource对象上的事件监听器相对应，可以没有这个字段，没有就是无名事件，默认由EventSource的`message`事件监听器来处理。`id`字段是事件的id，主要用于确认该类型事件的最后一个消息的id，也可以没有。

**另外，EventSource建立连接是通过`GET`请求的。**

### 客户端实现

EventSource是浏览器原生支持的，可以在js中直接使用，这里简单在SpringBoot中写了个`index.html`静态页面：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>SSE</title>
    <script>
        var source

        function start() {
            source = new EventSource("/sse");
            source.onmessage = function(event) {
                console.log('接收message事件:' + event.data)
                document.getElementById('result').innerText = event.data
            }
            source.onerror = function() {
                console.log('连接失败')
                source.close()
            }
            source.addEventListener('test', (event) => {
                console.log('接收test事件:' + event.data)
                document.getElementById('result').innerText = event.data
            })
        }

        function stop() {
            source.close()
            console.log('关闭连接')
        }
    </script>
</head>
<body>
    <div id="result"></div>
    <hr>
    <button onclick="start()">开启连接</button>
    <button onclick="stop()">关闭连接</button>
</body>
</html>
```

在js代码中，通过`new EventSource()`实例化一个EventSource对象，参数是HTTP服务的URL。`source.onmessage`和`source.onerror`是在设置EventSource默认监听器的回调函数，EventSource有三个默认的监听器，分别是：

+ `error`：与服务器建立连接失败时触发。
+ `message`：处理无名事件。
+ `open`：与服务器建立连接时触发。

如果想处理自定义事件的话，可以通过`addEventListener()`函数自己添加一个监听器。

## 过程解析

启动SpringBoot，在浏览器中打开页面并点击开启连接按钮，F12查看EventSource建立连接的请求：

![Header](https://cdn.jsdelivr.net/gh/shallowhui/cdn/picgo/eventsourceheader.png)

因为响应的是`text/event-stream`格式的数据，所以这里看不到普通HTTP响应的响应体之类的数据，要在`EventStream`选项卡中才能看到响应的数据：

![EventStream](https://cdn.jsdelivr.net/gh/shallowhui/cdn/picgo/eventstream.png)

可以看到一行行的事件消息形成的事件流，最终页面效果如下：

![HTML](https://cdn.jsdelivr.net/gh/shallowhui/cdn/picgo/eventsourcehtml.png)