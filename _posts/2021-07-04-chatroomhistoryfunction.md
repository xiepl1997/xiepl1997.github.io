---
layout: post
title: 【实践】Web聊天室历史记录解决方案（轻喷。。）
date: 2021-06-25
author: xiepl1997
tags: java springboot 实践
---

[聊天室快速访问](http://www.xpllyn.com/chatroom)  
之前写的Web聊天室一直没有更新了，其实还有一些功能没有完善，比如历史记录、视频对话等。这几天心血来潮，捡起之前的代码，从看起来最简单的聊天记录开始整。  

## 开始之前
当时写这个聊天室的时候，没有考虑保存聊天记录的功能，因为当时把写的东西先跑起来实现消息发送再说，，，汗，，。 跑起来之后想了想历史记录保存的问题，第一反应是保存到数据库啊！！！就是发一条，就保存一条，执行一条SQL语句。嗯，，很直接暴力。另外查了查资料，网上的文章和帖子都说即时通讯应用的聊天记录都是不会对数据库进行频繁读写的，因为即时通讯，强调**即时**，**聊天的时候频繁地对数据库进行操作会严重影响用户体验的**，尤其是用户多了之后（虽然我这个可能没人用，但我也得这么想啊）。  

不存下来，那么我该怎么办？

## 方案的抉择
虽然我做的是一个小应用，可能也不会有人真的来用，但是吧，我的态度需要端正，我需要认真对待严谨论证功能设计。哈哈哈，，嗝。。  

首先我想到qq和微信，这两货用户量那么大，肯定不会把所有人的聊天记录都保存下来的，那也太海量了，不现实。结合自己的使用经验，推知qq和微信有一部分聊天记录是保存在用户本地的，因为我家那个老电脑登录上N年前安装的qq，就能看到N年前的聊天记录，在别的电脑上就看不到了，无疑，TX在蹭我们的资源，哼😕。但是另一方面，qq消息记录可以漫游，可以知道这个近期消息也是会保存在服务器里的。（我对qq和微信的技术内幕也不清楚，只是根据自己的使用经验做出的一点点判断）  

emmm，所以我也开始想蹭用户资源，直接聊天记录保存到本地，不就VANS了？？是啊，可是，PC版和手机版qq是应用啊，是app，而，我这个只靠个浏览器，怎么蹭用户资源来存聊天记录？？首先我想到的是cookie、session、localstorage，前面这两货首先排除掉了，cookie用于零时存储一点小数据，session用于存储用户会话，并且保存在服务器，完全不符合场景！而localstorage还凑合，用于在浏览器永久保存网站的数据，没有过期时间，适合我用来蹭用户资源。  

可是，我又想到了，聊天记录保存在localstorage，那我换个浏览器，不就没了吗？？是啊，这种体验挺糟糕。  

看来还是得回归到服务器存储，那么我怎么做到在聊天的时候不对数据库进行操作又要保存聊天记录的呢？？不对磁盘进行操作，那就对内存进行操作啊，对内存的操作是快速高效的，可以利用内存缓存聊天记录，然后在深更半夜把内存中的聊天记录保存到数据库，是一个方法嗷！！

## 实施
首先选用Redis用于缓存聊天记录，MySQL用于消息记录的持久保存。Redis为每对聊天设立**两个缓存**，数据结构都是List，一个是history缓存，只会保存最近15条缓存，并且过期时间为48小时；一个是chat缓存，用于保存这对聊天未持久化的聊天记录，过期时间为25小时。聊天时一条记录会插入到这两个List中。那么缓存的key如何设置呢？  

我是这么做的，例如有id为7的用户和id为13的用户在聊天，那么Redis就会建立两个缓存，key分别为：7-13-history、7-13-chat，id小的在前。这样每对聊天缓存就有唯一的key了，每对聊天中的两个用户共用这个缓存。  

**那么为什么要两个缓存队列呢？**  
这是为了用户刚登录时消息记录获取逻辑的简单化、和定时任务时逻辑的简单化。history缓存最多只保存15条记录，在用户刚登录时，只需要获取history缓存的数据去渲染前端即可，不需要读取数据库，只有在history失效之后才从数据库中读取并放入history缓存中。另外在定时任务的时候，只需要把chat缓存中的数据全部持久化到MySQL中即可（定时任务设置在凌晨三点），持久化完毕后清空chat缓存，这样逻辑简单明了。  

另外，在每个聊天窗口最上面，可以点击“更多聊天记录”，这样的话会去数据库分页查找最近20条记录，构成一个聊天记录的列别，通过Ajax渲染完html后异步更新插入到当前聊天的div之前。**用户点击“更多聊天记录”时分为两种情况**，分为**第一次点击**和**不是第一次点击**，如果用户是第一次点击，只需要在数据库里获取最近20条记录，合上chat缓存中没有持久化到MySQL中的聊天记录，一起传给Ajax让Ajax构建新的html覆盖与对应好友的聊天div的html内容；如果不是第一次点击，则去数据中分页查找最近的20条记录（上一次查询之后的最近20条记录），传给Ajax构建html追加在对应好友的聊天div的html前面。

## Redis宕机了怎么办
如果Redis宕机了，对于我这个可怜的单服务器系统（阿里云最便宜的学生服务器），宕机了，那就只能聊一句，就插入一下数据库了:(  

另外，Redis的持久化策略使用的默认的RDB，900秒内有1次操作、300秒内有10次操作、60秒内有10000次操作的话，都会触发快照。当然我的系统只可能900秒内1次操作或者300秒内10次操作。没人用啊。。只需要选择对性能影响更小的RDB策略而不是AOF。  

这就是我的历史聊天记录功能的实现，想法挺简单，当然肯定还有更好的办法，可以一起交流。
最后附上定时任务的代码，每天3:00夜深人静准时启动！！
```java
@Configuration
@EnableScheduling
@Slf4j
public class ChatMessageSaveTask {

    @Autowired
    private IMRedisService imRedisService;

    @Autowired
    private ChatService chatService;

    /**
     * 定时任务，每天凌晨3点进行redis缓存的持久化，将聊天记录保存到数据库
     */
    @Scheduled(cron = "0 0 3 * * ?")
    public void chatMessageSave() {
        log.info("【定时任务】 开始！");

        // 获取缓存中的群聊天缓存
        List<GroupMessage> gm = imRedisService.getGroupMessage();
        // 聊天记录持久化
        if (gm != null) {
            try {
                chatService.insertGroupMessages(gm);
                // 清空缓存
                imRedisService.remove("group");
                log.info("【定时任务】 " + gm.size() + "条世界频道新聊天记录持久化到MySQL。");
                log.info("【定时任务】 " + gm.size() + "条世界频道新聊天记录缓存已清空。");
            } catch (Exception e) {
                log.error("【定时任务】 Redis出错！");
            }
        } else {
            log.info("【定时任务】 无世界频道新聊天记录持久化到MySQL。");
        }

        // 获取单聊聊天缓存
        String pattern = "*-chat";
        Set<String> keys = imRedisService.getRedisTemplate().keys(pattern);
        int cnt = 0;
        if (keys != null) {
            for (String key : keys) {
                List<ChatMessage> cm = imRedisService.getChatMessage(key);
                // 聊天记录持久化
                if (cm != null) {
                    try {
                        chatService.insertChatMessages(cm);
                        // 清空缓存
                        imRedisService.remove(key);
                        cnt += cm.size();
                    } catch (Exception e) {
                        log.error("【定时任务】 Redis出错！");
                    }
                }
            }
        }
        if (cnt != 0) {
            log.info("【定时任务】 " + cnt + "条单聊新聊天记录从Redis新消息缓存持久化到MySQL。");
            log.info("【Redis】 " + cnt + "新消息缓存已清除。");
        } else {
            log.info("【定时任务】 无单聊新聊天记录持久化到MySQL。");
        }
    }
}
```