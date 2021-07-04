---
layout: post
title: thymeleaf多重${}
date: 2021-07-03
author: xiepl1997
tags: java springboot
---

今天遇到一个问题，thymeleaf接收到controller传递过来的model数据中，含有list和map结构的数据，在thymeleaf渲染数据的过程中，首先用th:each遍历list，取list中的元素作为map的键，然后从map中取出对应的value来。  

想要在thymeleaf上渲染controller层传递过来的数据，需要使用${}来获取后台数据，例如contoller传递一个“user”到thymeleaf，需要在前端使用user.id，如下所示，可以获取到传递过来的参数。
```xml
<div th:text="${user.id}"></div>
```

有时候controller传递过来的是一个**map、list**结构的数据，并且需要通过遍历**list**来渲染界面，其中需要使用**list**中的元素来作为**map**的键，从而得到对应的值来渲染界面，那怎么办的呢？肯定是嵌套，例如下面这样
```xml
<span th:each="friend:${friends}">
	<span th:each="msg:${msgMap[__${friend.id}__]}">
		………………
	</span>
</span>
```
但是三重嵌套呢？经过我实际测试，在渲染的时候是会报错的，不能三个${}嵌套。这种情况就得另外想办法了。比如使用**th:with**来定义一个thymeleaf临时变量，替换掉一个${}的使用，这样就行了。