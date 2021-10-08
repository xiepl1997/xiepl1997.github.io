---
layout: post
title: 【小记】有关cookie和session是什么
date: 2020-05-26
author: xiepl1997
tags: springboot 小记
---

cookie？ 饼干？ 是的呀，这玩意儿能让人舒服~  

cookie大家都熟悉，例如登陆一些网站，一段时间后，就要求你重新登陆。

## 1. cookie 和 session 简介
HTTP是一种无状态的一种协议，换句话说，就是服务器记不住你，可能你每刷新一次页面，就要重新输入一次账号和密码进行登陆，这显然是让人无法接受的。cookie的作用就好比服务器给你贴个标，然后你每次向服务器再发起请求的时候，服务器能够认出是你。  

抽象概括一下，**一个cookie可以认为是一个【变量】，形如name=value，存储在浏览器；一个session可以理解为一种数据结构，多数情况下是【映射】（键值对），存储在服务器。**  

一个cookie可以认为是一个变量，但是服务器可以设置一组cookie，所以有时候说cookie是一组键值对。  

cookie的作用其实很简单，无非就是服务器给每个客户端（浏览器）打的标签，方便服务器辨认而已。**但是**问题是，现在很多的网站功能很复杂，而且涉及很多的数据交互，比如说电商的购物车功能，信息量大，而且结构也比较复杂，无法通多简单的cookie机制传递那么多信息，**而且cookie时存储在HTTP header中的**，就算能够承载这些信息，也会消耗很多带宽，比较消耗网络资源。  

session就可以配合cookie解决这一问题，比如说一个cookie存储这样一个变量sessionID=xxxx，仅仅把这一个cookie传递给服务器，然后服务器通过这个ID找到对应的session，这个session是一个数据结构，里面存储着该用户的购物车等详细信息，服务器可以通过这些信息返回该用户的定制化网页，有效解决了追踪用户的问题。  

**session是一个数据结构，由网站的开发者设计，所以可以承载各种数据**，只要客户端的cookie传来一个唯一的sessionID，服务器就可以找到对应的session，认出这个用户。  

当然，由于session是存储在服务器上的，肯定会消耗服务器的资源，所以session一般都会有一个过期时间，服务器一般会定期检查并删除过期的session，如果后来用户再次访问服务器，可能会面临重新登陆等措施，然后服务器重新建立一个session，将sessionID通过cookie的形式传送给客户端。  

## 2.springboot中cookie和session的实现
下面展示登陆时分别使用session和cookie来保存用户信息的代码。
```java
//登陆时分别用session和cookie保存用户信息
@RequestMapping("/login")
public String login(@RequestParam("email") String email,
					@RequestParam("password") String password,
					Model model,
					RedirectAttributes redirectAttributes,
					HttpServletRequest request) {
	password = MD5.getMD5(password);
	//获取用户
	User user = userService.SelectUserByEmailPassword(email, password);
	if (user == null) {
		model.addAttribute("msg", "账号或密码不正确！");
		return "login";
	}

	//创建session用来保存用户信息
	HttpSession session = request.getSession();
	session.setAttribute("user", user);

	//创建cookie用来保存用户信息
	Cookie cookie = new Cookie("username",user.getName());
	cookie.setMaxAge(7 * 24 * 60 * 60);
	response.addCookie(cookie);

	return "redirect:/index";
}

//通过session或者cookie获取username，当然session中也可以只存储username而不是user
public String getUserName(HttpServletRequest request){

	String username = ""；

	//从session中获取username
	HttpSession session = request.getSession();
	Object user = session.getAttribute("user");
	username = ((User)user).getName();

	//从cookie中获取username
	Cookie[] cookies = request.getCookies();
	for(Cookie cookie : cookies){
		if(cookie.getName().equals("username")){
			username = cookie.getValue();
			break;
		}
	}
	
	return username;

}
```