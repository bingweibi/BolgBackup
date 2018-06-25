---
title: Android开发艺术探索---Activity的生命周期和启动模式
date: 2018-05-02 22:17:05
tags: Android
---

# Activity的生命周期和启动模式

## Activity的生命周期全面分析

### 典型情况下的生命周期分析

<center>![](http://p2g00vblr.bkt.clouddn.com/Activity%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F.png)</center>

<!--more-->

注意情况：

1. 针对一个特定的Activity,第一次启动的时候，回调顺序如下:onCreate->onStart->onResume
2. 当用户打开新的Activity或者切换到桌面的时候，回调顺序如下:onPause->onStop.这里有一个特殊情况，如果新的Activity是透明的主题，那么当前的Activity不会回调onStop。
3. 当用户再次回到原来的Activity时候，回调顺序如下:onRestart->onStart->onResume
4. 当用户按back键回退的时候，回调顺序如下:onPause->onStop->onDestory
5. 当Activity被系统回收后再次打开，生命周期方法回调过程和1中一样，只是生命周期一样
6. 从整个生命周期来说，onCreate和onDestory是一对，分别对应着Activity的创建和销毁，并且只可以有一次调用；从是否可见来看,onStart和onStop是一对，可以被调用多次；从是否处于前台来说，onResum和onPause是一对，可以被调用多次。

*从Activity A跳往Activity B的过程中，先执行A 的onPause再执行Activity B的onResume*

### 异常情况下的生命周期分析

1. 资源相关的系统配置发生改变导致Activity 被杀死并重新加载
Activity由于异常情况下终止，系统会调用onSaveInstanceState来保存当前Activity的状态，在onStop之前调用。
系统只会在Activity即将被销毁并且有机会重新显示的情况下才会去调用它。
2. 资源内存不足导致低优先级的Activity被杀死
    - 前台Activity，优先级最高
    - 可见但非前台Activity,优先级次之
    - 后台Activity,优先级最低

## Activity的启动模式

### Activity的LaunchMode

任务栈：默认情况下，所有Activity所需的任务栈的名字为应用的包名

1. standard(默认模式)，每次启动一个Activity都会重新创建一个新的实例，不管这个实例是否已经存在。
2. singleTop(栈顶复用模式),如果新Activity已经位于任务栈的栈顶，那么此Activity不会被重新创建，同时它的onNewIntent方法会被回调，通过此方法的参数我们可以取出当前请求的信息。
3. singleTask(栈内复用模式)，一种单例模式，在这种模式下，只要Activity在一个栈中存在，那么多次启动此Activity都不会重新创建实例，和singleTop一样，系统也会回调其onNewIntent
4. singleInstance：具有此模式的Activity只能单独地位于一个任务栈中

**指定Activity模式的两种方法**

```
<activity
    android:name="..."
    android:configChange="..."
    android:launchMode="singleTask"
    />
```

```
startActivity(new Intent().setClass(this,...).addFlags(Intent.FLAG_ACTIVITY_NEW_TASK))；
```

第二种方法的优先级高于第一种方法

### Actvity的Flags

|名称|作用|
|:---:|:---:|
|FLAG_ACTIVITY_NEW_TASK|xml中指定启动模式singleTask|
|FLAG_ACTIVITY_SINGLE_TOP|xml中指定启动模式singleTop|
|FLAG_ACTIVITY_CLEAR_TOP|在同一个任务栈中所有位于它上面的Activity都要出栈|
|FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS|xml中指定android:excludeFromRecents="true"|

## IntentFilter的匹配规则

有一篇已经介绍过了，就不在介绍了。