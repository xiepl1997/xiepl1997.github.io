---
layout: post
title: 个人网站新功能：聊天室
date: 2021-02-20
author: xiepl1997
tags: 敲敲敲 springboot java
---

之前在个人网站上预留了一个网页聊天室的功能，在这次寒假抽出了时间来完成。[快速访问](http://www.xpllyn.com/chatroom)  

简要记录一下网页聊天室的设计过程。

## AJAX轮询
在设计之前一直纠结该使用何种方式来实现网页聊天室这个模块，最基本的想法是使用ajax来实现轮询，从而达到消息推送的目的，目前的很多网站也是通过这样的手段来实现推送技术。轮询是在特定的时间间隔（如每一秒），由浏览器对服务器发出HTTP请求，然后由服务器返回最新的数据给客户端的浏览器。这样传统的模式需要不断的向服务器发出请求，然而HTTP请求可能包含较长的头部，其中真正有效的数据可能只是很小的一部分，显然这样会浪费很多的带宽等资源。  

## WebSocket
HTML5定义的WebSocket协议，能够更好地节省服务器资源和带宽，并且能够更实时地进行通讯。  

WebSocket是为了解决实时传送消息的问题，当然也可以传送数据，但是不保证传送的效率和质量，而WebRTC可用于可靠地传输音视频数据、文件等。并且可以建立P2P连接，不需要服务进行转发数据。虚拟电话、在线面试等现在很多都采用WebRTC来实现。  

WebSocket并不是我们寻常意义上的基于TCP或者UDP协议的套接字，而是一个基于TCP的应用层协议，和HTTP协议工作在计算机网络的同一层；WebSocket其实就是，在TCP的基础上增加了一个协议，这个协议和TCP协议很想，也需要握手，挥手，发送大数据也要分包，也是按照位进行标识，但它并不是套接字，不是socket，socket工作在传输层，它工作在应用层。  

浏览器通过JavaScript想服务器发出建立WebSocket连接的请求，连接建立后，客户端和服务器端就可以通过TCP连接直接交换数据。当获取到Web Socket连接后，可以通过send()方法来向服务器发送数据，并通过onmessage时间来接受服务器返回的数据。  

JS代码如下：
```java
<script type="text/javascript">
    var socket;
    if (!window.WebSocket) {
        window.WebSocket = window.MozWebSocket;
    }
    if (window.WebSocket) {
        socket = new WebSocket("ws://www.xpllyn.com:3333");
        socket.onopen = function () {
            var param = {
                id: $('.user').attr('name'),
                type: 'REGISTER'
            };
            socket.send(JSON.stringify(param));
        };
        socket.onmessage = function (event) {
            var json = JSON.parse(event.data);
            if (json.status == 200) {
                var type = json.data.type;
                switch (type) {
                    case "SINGLE_SENDING":
                        ws.singleReceive(json.data);
                        break;
                    case "GROUP_SENDING":
                        ws.groupReceive(json.data);
                        break;
                    case "GROUP_SENDING_All":
                        ws.groupReceiveAll(json.data);
                        break;
                    case "AGREE_FRIEND_REQUEST":
                        addPerson(json.data);
                        break;
                    default:
                        break;
                }
            }
        };
        socket.onclose = function (event) {
            socket.close();
            alert('连接已断开！');
            window.location.replace('http://www.xpllyn.com/loginpage');
        };
        socket.onerror = function (event) {
            socket.close();
            alert('发生错误！');
            window.location.replace('http://www.xpllyn.com/loginpage');
        };
    } else {
        alert("您的浏览器不支持WebSocket！");
    }
</script>
```
浏览器端可以通过JavaScript来使用WebSocket实现消息的发送和接收，那么服务器端该如何处理消息发送和接收呢？😣😣

## Netty
Netty是由JBOSS提供的一个java开源框架。Netty提供异步的、事件驱动的网络应用程序框架和工具，可以快速开发高性能、高可靠性的网络服务器和客户端程序。也就是说，Netty是一个基于NIO的客户、服务器端的编程框架，使用Netty可以确保快速和简单的开发出一个网络应用，例如实现某种特定协议的客户、服务器应用。Netty相当于简化和流线化了网络应用的编程开发过程，例如基于TCP和UDP的socket服务开发。  

这次的网页聊天室的开发，选用了WebSocket协议，加上Netty对WebSocket的支持，在服务端上我选择了Netty来实现一个WebSocket服务器。HTTP服务器之所以称为**HTTP服务器**，是因为编码解码协议是HTTP协议，如果协议是Redis协议，那它就是Redis服务器，如果协议是WebSocket，那它就是WebSocket服务器，等等。  

### Netty的应用
在互联网行业，网站规模不断扩大，系统并发访问量越来越高，传统基于Tomcat等Web容器的垂直架构已经无法满足要求，需要拆分应用进行服务化，以提高开发和维护的效率，垂直架构拆分后，系统采用分布式部署，各个节点之间需要远程服务调用（RPC），高性能的RPC框架必不可少，Netty作为异步高性能的通信架构，往往作为基础通信组件被这些RPC框架使用，例如阿里的Dubbo、RocketMQ。  

游戏行业。无论是手游服务器还是大型的网络游戏，java语言得到越来越广泛的应用。Netty作为高性能的基础通信组件，本身提供了TCP/UDP和HTTP协议栈，非常方便定制和开发私有协议栈。

大数据领域、通信行业等等。

### Reactor主从多线程模型
Netty的线程模型并不简单，使用的是Reactor主从多线程模型，特点是：服务端用于接收客户端连接的不再是一个单独的NIO线程，而是一个独立的NIO线程池。  

由于内容较多，将来会单独总结为一篇博客。

## 总体思路
整个系统以Tomcat作为核心的服务器运行，单独开启一个线程启动Netty服务器（使用@PostConstruct注解，能够在Contructor、Autowired之后运行）。Tomcat端口为8080，处理除了聊天室之外的一些请求，Netty端口为3333，主要负责处理用户的WebSocket类型的请求。用户如若要使用聊天室的话，首先要先通过登陆，登陆是采用Shiro安全框架进行验证的。登陆后浏览器会发起WebSocket连接，服务器接收到连接请求后会为用户建立一条客户端连接，并通过用户的id组成键值对保存在Map中，当一个用户向另一个用户发起通信时，服务器会根据消息内容中的接收方的的id，找到保存的WebSocket连接，通过该连接向对应的channel发送消息，对方通过前端的JS能够接收到该条消息。Netty的工作主要就是转发消息。当用户退出时，释放WebSocket连接。  

在聊天室中可以进行【世界频道】，也就是所有用户的公共聊天室，另外，可以搜索系统中的用户，进行加好友的操作，添加好友之后可以进行私聊。  

目前聊天室可以正常工作[快速访问](http://www.xpllyn.com/chatroom)，但是历史聊天数据模块还没有设计完成。主要是为了保证消息的实时性，当前发送消息时没有对数据库进行操作，所以每次刷新后，界面上的聊天记录都会消失，因为根本就没有保存在任何地方。对于如何保存聊天记录，目前的想法是，使用Redis将聊天记录缓存下来，当用户的WebSocket连接断开时，将用户的聊天记录保存在他本地，也就是他自己的电脑文件中，当在聊天界面手动刷新或者登陆时，读取本地的聊天记录加载到界面上。嗯，就是这样。  

聊天界面截图：
![image](https://img-blog.csdnimg.cn/img_convert/380c3fc7dfc002905b0d371767b5edb4.png)