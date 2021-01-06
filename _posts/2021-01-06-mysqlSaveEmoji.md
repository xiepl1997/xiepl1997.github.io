---
layout: post
title: MySql存储emoji表情报错的处理方法
date: 2020-12-19
author: xiepl1997
tags: 随笔 java springboot
---

mysql存储emoji表情报错的处理方法：更改编码为utf8mb4  

uft-8编码可能2个字节、3个字节、4个字节，而MySql的uft-8只支持3字节的数据，而移动端的表情数据是4字节的字符。如果直接采用utf-8编码的数据库中插入表情数据，Java程序将报错：  
```java
java.sql.SQLException: Incorrect string value: '\xF0\x9F\x92\x94' for column 'name' at row 1
at com.mysql.jdbc.SQLError.createSQLException(SQLError.java:1073)
at com.mysql.jdbc.MysqlIO.checkErrorPacket(MysqlIO.java:3593)
at com.mysql.jdbc.MysqlIO.checkErrorPacket(MysqlIO.java:3525)
at com.mysql.jdbc.MysqlIO.sendCommand(MysqlIO.java:1986)
at com.mysql.jdbc.MysqlIO.sqlQueryDirect(MysqlIO.java:2140)
at com.mysql.jdbc.ConnectionImpl.execSQL(ConnectionImpl.java:2620)
at com.mysql.jdbc.StatementImpl.executeUpdate(StatementImpl.java:1662)
at com.mysql.jdbc.StatementImpl.executeUpdate(StatementImpl.java:1581)
```
解决方法之一是对4字节的字符进行编码存储，然后取出来的时候，再进行解码。这样做的话就会使得任何使用该字符的地方都要进行解码和编码。  

utf8mb4编码是utf8编码的超集，兼容utf8，并且能存储4字节的表情字符。
采用utf8mb4的好处是：<font color="#ff0000">存储与获取数据的时候，不用考虑编码和解码的问题</font>  

## 解决办法
**更改数据库的编码为uft8mb4**

### 1.MySql的版本
utf8mb4的最低版本支持版本为5.5.3+

### 2.MySql驱动
5.1.34可用，最低不能低于5.1.13

### 3.修改MySql配置文件
修改mysql的配置文件my.cnf，linux环境下一般在/etc/mysql/my.cnf位置。在文件中添加如下内容：
```
[client]
default-character-set = utf8mb4
[mysql]
default-character-set = utf8mb4
[mysqld]
character-set-client-handshake = FALSE
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci
init_connect='SET NAMES utf8mb4'
```

### 4.重启数据库，检查变量
登录mysql后输入：```SHOW VARIABLES WHERE Variable_name LIKE 'character_set_%' OR Variable_name LIKE 'collation%';```  
确保一下几个变量：  
| 系统变量 | 描述 |
| :------ | :---- |
| character_set_client | 客户端来源数据使用的字符集 |
| character_set_connection | 连接层字符集 |
| character_set_database | 当前选中数据库的默认字符集 |
| character_set_results | 查询结果字符集 |
| character_set_server | 默认的内部操作字符集 |  
这几个变量必须是utf8mb4.  

同时，数据库和建好的表也转化为utf8mb4.

### 5.数据库连接的配置
数据库连接参数中:  
**characterEncoding=utf8**会被自动识别为utf8mb4，也可以不加这个参数，会自动检测。  
而autoReconnect=true是必须加上的。  

经过上面的步骤，就可以实现在mysql数据库中存储emoji表情了。