---
layout : post
title : 【实践】关于thymeleaf的select下拉选择框的默认选中项
date : 2020-05-29
author : xiepl1997
tags : thymeleaf springboot 小记 实践
---

最近在敲敲敲的过程遇到一个问题，这个问题是这样的：我需要在页面上提供用户更新信息的功能，在进入该页面前，首先获取信息，使用thymeleaf模板填充信息。前端有一个select标签，需求是显示用户之前所选中的内容。  

但是尝试了通过th:attr和th:if判断，都不好用，使用jquery来进行attr的设置也不好使。查阅一番才知道了解决办法：**使用th:selected**  

```java
<select name="select" id="projecttype" class="form-control" >
	<option value="横向项目" th:selected="${project.type} eq '横向项目'">横向项目</option>
	<option value="纵向项目" th:selected="${project.type} eq '纵向项目'">纵向项目</option>
</select>
```

thymeleaf模板自带属性：<font color=red>th:selected</font>，想让哪个option成为默认值只需要设置th:selected，后面加上判断，如果条件为true，就会显示selected属性。  

特此记录。