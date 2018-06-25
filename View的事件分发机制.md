---
title: View的事件分发机制
date: 2018-04-10 16:05:23
tags: 
	- Android 
	- 事件分发
---

点击事件的分发过程有三个重要的方法来共同完成

1. dispatchTouchEvent(MotionEvent ev):负责进行事件的分发
2. onInterceptTouchEvent(MotionEvent event):负责是否拦截某个事件
3. onTouchEvent(MotionEvent event):用于处理点击事件

## 现在我们用类比的方式来讲解事件分发机制

假设你是一个公司的小职员，你的上司是部长，部长的上司是总经理。

1. 公司接到一个业务(点击事件开始)
2. 首先肯定是总经理先接单(根视图:ViewGroup),判断是否将这个业务派发给部长完成(触发了dispatchTouchEvent(MotionEvent ev)方法)
3. 如果总经理觉得这单业务过于简单只需要自己就可以完成，于是便拦截下来，不往下分发了(触发了onInterceptTouchEvent(MotionEvent event))方法返回true
4. 总经理就自己处理了这个事件(出发了onTouchEvent(MotionEvent event))
5. 回到3，如果总经理觉得这个事情自己不愿意做，便将这个事情派发给了部长(触发了onInterceptTouchEvent(MotionEvent event))返回false
6. 部长同时觉得很适合你来做(触发了onInterceptTouchEvent(MotionEvent event))返回false
7. 现在任务就应该由你来完成了，你是不可以拦截的(没有onInterceptTouchEvent(MotionEvent event))

## 处理事件机制
1. 设置OnTouchListener，那么OnTouchListener中的onTouch方法会被会调
2. 如果OnTouch返回false，就调用当前View的onTouchEvent
3. 如果返回true,那么onTouchEvent就不会被调用
4. 在onTouchEvent方法中，如果设置了OnClickListener，那么它的onClick会被调用，所以OnClickListener优先级最低