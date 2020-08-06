---
layout: post
title: idea导入springboot项目使用maven引入依赖失败，包名现红色波浪线
date: 2019-10-06
author: xiepl1997
tags: springboot
---

换个了电脑之后，安装好配置好各种环境。  
从github上把之前的springboot项目clone下来，使用intellijidea对项目进行导入。  
等待了一个多钟，等来的确是maven中的包名都出现红色波浪线，是引入依赖失败了。
试了很多方法都不行，最后想到可能是因为网络的缘故，jar包没有下载完整。
最后找到本地仓库，将后缀为.lastupdate的文件都删除，再对maven进行reimport操作，令人抓狂的红色波浪线终于消失。  
（idea的默认本地仓库：C:\Ususername\.m2\repository）