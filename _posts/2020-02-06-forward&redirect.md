---
layout: post
title: 转发与重定向的区别
date: 2020-02-06
author: xiepl1997
tags: java
---

一直没有搞懂转发和重定向的概念，更别提区别了。今天在上面栽跟头，特查资料总结如下。

##转发（服务端行为）

形式：`request.getRequestDispatcher().forward(request,response)`  
在springboot中的controller进行试图解析时默认转发。  
转发在服务器端发挥作用，通过forward()方法提交信息在多个页面之间进行传递。
* 地址栏不会变
* 转发只能转发到当前web应用内的资源
* 在转发过程中，可以将数据保存到request域对象中去
* 转发只有一次请求
转发是服务器端行为

####转发过程

1.客户端浏览器发送http
2.web浏览器接受请求
3.调用内部的一个方法在容器内部完成请求处理和转发动作  

需要注意的是：转发路径必须是同一个web容器下的url。在客户端浏览器路径显示的仍然是第一次访问的路径。转发行为是浏览器只做了一次访问请求。

##重定向（客户端行为）

形式：`response.sendRedirect("");`  
springboot中进行重定向时，controller中写
```java
	mv.setViewName("redirect:/main.html");
	return mv;
```
随后使用WebMvcConfigurer扩展SpringMVC
```java
	@Configuration
	public class ViewControllerimpl implements WebMvcConfigurer {
	    @Override
	    public void addViewControllers(ViewControllerRegistry registry) {
	    	//将templates中的StudentPage.html映射到路径urlpath为“/main.html”上
	        registry.addViewController("/main.html").setViewName("StudentPage");
	    }
	}
```
如果controller中直接写`mv.setViewName("redirect:StudentPage")`的话，将会出现404（若StudentPage.html存放在templates文件夹中）。这是因为templates不是springboot项目的静态资源地址，重定向是二次请求，所以无法访问templates中资源。  
* 重定向地址栏会改变
* 重定向可以跳转到当前web应用，甚至是外部域名网站
* 不能在重定向过程中，将数据保存到request域对象中

####重定向过程
1.客户端发送http请求
2.web服务器接收后，发送302状态码，以及新的location给客户端浏览器
3.客户端浏览器发现是302响应，则自动发送一个http请求，请求url为重定向的地址，相应的服务器根据此请求寻找资源并发送给客户。