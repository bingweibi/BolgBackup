---
title: RxJava2(1)
date: 2018-06-24 19:32:57
tags: RxJava2
---

# 简介

RxJava2 提供一套基于[观察者模式](http://design-patterns.readthedocs.io/zh_CN/latest/behavioral_patterns/observer.html)的[异步编程](https://www.jianshu.com/p/c4dc7866eb81)的API，且链式调用。

## 如何开始

- 基本元素
    1. 被观察者(Observable)
    2. 观察者(Observer)
    3. 订阅(subscribe)
相信理解了观察者模式的很好理解这些概念。
    
- 在Gradle文件中添加依赖
```
implementation "io.reactivex.rxjava2:rxjava:2.1.15"
implementation 'io.reactivex.rxjava2:rxandroid:2.0.2'
```

<!--more-->

## 示例应用

```
        //创建被观察者
        Observable observable = Observable.create(new ObservableOnSubscribe() {
            @Override
            public void subscribe(ObservableEmitter emitter) throws Exception {
                Log.i(TAG, "subscribe thread: " + Thread.currentThread().getName());
                emitter.onNext(1);
                emitter.onNext(2);
                emitter.onNext(3);
                emitter.onComplete();
            }
        });

        //创建观察者
        Observer observer = new Observer<Integer>() {
            @Override
            public void onSubscribe(Disposable d) {
                Log.i(TAG, "onSubscribe: ");
            }

            @Override
            public void onNext(Integer i) {
                Log.i(TAG, "onNext: " + i);
            }

            @Override
            public void onError(Throwable e) {
                Log.i(TAG, "onError: " + e.toString());
            }

            @Override
            public void onComplete() {
                Log.i(TAG, "onComplete: ");
            }
        };

        //建立订阅关系
        observable.subscribe(observer);
```

## 输出结果

```
    onSubscribe: 
    subscribe thread: main
    onNext: 1
    onNext: 2
    onNext: 3
    onComplete: 
```

## 解释

被观察者可以发送如下几种事件
|事件种类|作用|
|:--:|:--:|
|onNext()|被观察者发送该事件，观察者onNext()接收|
|onError()|被观察者发送该事件，观察者onError()接收，之后不再接收其他事件|
|onComplete()|被观察者发送该事件,观察者onComplete()接收，之后不再接收其他事件|

## RxJava2流程

<center>![](http://p2g00vblr.bkt.clouddn.com/RxJava2%E6%B5%81%E7%A8%8B%E5%9B%BE.png)</center>