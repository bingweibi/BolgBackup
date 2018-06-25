---
title: Android消息机制
date: 2018-04-10 16:58:41
tags: 
	- Android
---

# 概述

Android的消息机制主要是指`Handler`的运行机制以及Handler所附带的`MessageQueue`和`Looper`的工作过程。

`Hnadler`：主要作用是讲一个任务切换到某个指定的线程中去执行
`MessageQueue`: 消息队列，内部实现以单链表的形式来存储消息列表，并对外提供插入和删除操作。
`Looper`:以无限循环的形式去查找是否有新的消息，如果有的话就处理消息，否则就一直等待着。
`ThreadLocal`:ThreadLocal可以再不同的线程中互不干扰地存储并提供数据，通过ThreadLocal可以轻松获取每个线程的Looper.

Android中UI线程不安全，为什么不通过加锁解决？
1. 加锁使得逻辑复杂
2. 加锁使得效率降低

# 解决方案

1. 创建一个Message对象，借助Handler发送出去
2. 之后再handlerMessage()中获取到刚刚发送出去的Message对象

子线程创建Handler,加上Looper.prepare，主线程自动调用Looper.prepare,Looper.Loop开启循环

# 流程分析

1. Handler利用send/post发送消息
2. 调用MessageQueue的enqueueMessage将消息入列
3. Looper发现有消息(MesssgaeQueue的next方法)到来，就会处理这个消息
4. Looper交由Hnadler处理Handler的dispatchMeaasge方法调用 
5. Handler的handleMessage方法调用

<center>![](http://p2g00vblr.bkt.clouddn.com/%E5%BC%82%E6%AD%A5%E6%B6%88%E6%81%AF%E5%A4%84%E7%90%86%E6%B5%81%E7%A8%8B.png)</center>