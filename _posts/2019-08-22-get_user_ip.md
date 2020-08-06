---
layout: post
title: 获取客户端用户真实ip方法整理
date: 2019-08-22
author: xiepl1997
tags: springboot
---

由请求获取客户端ip地址的方法是request.getRemoteAddr()，在大部分的情况下该方法是有效的，但是在通过了apache、squid等反向代理软件就不能获取到客户端的真实ip了。  

经过代理后，由于在客户端和服务之间增加了中间层，因此服务器无法直接拿取到用户的ip地址，服务器端应用也无法直接通过转发请求的地址返回给客户端。但在转发请求的HTTP头信息中，增加了X-FORWARDED-FOR信息，用以跟踪原有客户端ip地址和原来客户端的请求的服务器地址。  

获取客户端真实ip地址的方法如下
```java
/**
 * @param request
 * @return 返回用户的IP地址
 */
public String getUserIp(HttpServletRequest request){
    String ip = request.getHeader("X-Forwarded-For");

    if(ip == null || ip.length() == 0 || "unknow".equalsIgnoreCase(ip)){
        ip = request.getHeader("Proxy-Client-IP");
    }
    if(ip == null || ip.length() == 0 || "unknow".equalsIgnoreCase(ip)){
        ip = request.getHeader("WL-Proxy-Client-IP");
    }
    if(ip == null || ip.length() == 0 || "unknow".equalsIgnoreCase(ip)){
        ip = request.getHeader("HTTP_CLIENT_IP");
    }
    if(ip == null || ip.length() == 0 || "unknow".equalsIgnoreCase(ip)){
        ip = request.getHeader("HTTP_X_FORWARDED_FOR");
    }
    if(ip == null || ip.length() == 0 || "unknow".equalsIgnoreCase(ip)){
        ip = request.getRemoteAddr();
    }

    return ip;
}
```
这些请求头的意思：
* X-Forwarded-For
这是一个 Squid 开发的字段，只有在通过了HTTP代理或者负载均衡服务器时才会添加该项。
* Proxy-Client-IP/WL- Proxy-Client-IP
这个一般是经过apache http服务器的请求才会有，用apache http做代理时一般会加上Proxy-Client-IP请求头，而WL-Proxy-Client-IP是他的weblogic插件加上的头。
* HTTP_CLIENT_IP
有些代理服务器会加上此请求头。

***
（2019-08-24 补）
测试的时候因为用到自己电脑测试，于是用上诉方法会得到127.0.0.1的本地环回地址，为了获取到本机的ip地址，加上以下代码
```java
if(ip.equals("127.0.0.1")){
    //根据网卡获取ip
    InetAddress inet = null;
    try {
        inet = InetAddress.getLocalHost();
    }catch (UnknownHostException e){
        e.printStackTrace();
    }
    ip = inet.getHostAddress();
}
```
