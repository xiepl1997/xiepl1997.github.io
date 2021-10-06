---
layout: post
title: Spring源码环境搭建踩坑记录
date: 2021-10-06
author: xiepl1997
tags: java spring 源码阅读
---

之前调试Spring源码为了图省事，都是直接利用idea搭建一个Spring项目然后断点调试。这样的好处是快，坏处是对于Spring的整个代码架构没办法像自己的项目代码那样每个包、每个模块一目了然，并且是只读的，没有办法做一些修改与注释。搭建Spring源码阅读环境势在必行。  

废话不多说，记录一下坑。  

我使用的是Spring-5.0.4、gradle-4.4.1  

1.下载Spring源码，下载并配置gradle  

2.配置Spring源码依赖的jar包的下载地址，打开根目录下的build.gradle  
第一处：在文件的首行，修改后的配置如下：  
repositories {  
	maven { url “https://maven.aliyun.com/repository/spring-plugin” }  
	maven{ url “https://maven.aliyun.com/nexus/content/repositories/spring-plugin”}  
}  
第二处：大概在150行左右，修改后的配置如下：
repositories {  
	maven { url “https://maven.aliyun.com/repository/public” }  
	maven { url “https://maven.aliyun.com/repository/central” }  
	maven { url “https://maven.aliyun.com/repository/spring” }  
}  

3.把gradle-4.4.1-bin.zip拷贝到spring-framework-5.0.4.RELEASE\gradle\wrapper下，修改wrapper文件夹中的gradle-wrapper.properties文件，把distributionUrl=https\://services.gradle.org/distributions/gradle-4.4.1-bin.zip改为distributionUrl=file:/C:/Intellij-Idea-Project/spring-framework-5.0.4.RELEASE/gradle/wrapper/gradle-4.4.1-bin.zip  


4.在Spring根目录执行gradlew.bat  
把Spring根目录中的build.gradle中的id "org.jetbrains.dokka" version "0.9.15"修改为id "org.jetbrains.dokka" version "0.9.17"  

5.在Spring根目录下执行  
gradle objenesisRepackJar  
gradle cglibRepackJar  

6.再使用idea导入Spring源码