---
layout: post
title: 【踩错】解决Shiro第一次重定向url带有jsessionid导致400错误
date: 2021-02-18
author: xiepl1997
tags: java springboot 踩错
---

在Shiro进行第一次重定向时，会在url后携带jsessionid，这会导致400错误（无法找到该网页）。  
原因在于ShiroHttpServletResponse配置类的doIsEncodeable当中，会将url自动拼接jsessionid。  

解决办法：  
1. 在Shiro的配置类中的sessionManager()方法中，将sessionIdUrlRewritingEnabled属性设置为false。该方法返回一个DefaultWebSessionManager实例。
2. 将上面方法返回的实例设置为DefaultWebSecurityManager实例的sessionManager。  

代码如下：
```java
@Bean
public SecurityManager securityManager() {
	DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
	securityManager.setSessionManager(sessionManager());
	securityManager.setRealm(myShiroRealm());
	//将cookie管理器交给SecurityManager进行管理
	securityManager.setRememberMeManager(rememberMeManager());
	return securityManager;
}

@Bean
public DefaultWebSessionManager sessionManager() {
	DefaultWebSessionManager sessionManager = new DefaultWebSessionManager();
	sessionManager.setSessionIdUrlRewritingEnabled(false);
	return sessionManager;
}
```