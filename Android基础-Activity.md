---
title: Android基础---Activity
date: 2018-04-16 22:37:51
tags: 
	- Android
---

# 介绍

1. Android四大组件(Activity、Service、BroadcastReceiver、Content Provider)
2. Android使用栈来对Activity来进行管理，所有的Activity会按照启动顺序排序进入一个叫做返回栈的里面。
3. Android中activity有四种启动模式standard、singleTop、singleTask、singleInstance
    - standard:默认模式，每当新建一个activity的时候就会将这个activity压入返回栈中
    - singleTop:如果新建一个Activity A，但是现在发现返回栈的栈顶就是Activity A，那么就直接使用栈顶的Activity A，不用新键
    - singleTask:如果现在新建一个Activity A，但是发现返回栈中出现了Activity A，那么就将返回栈中Activity A之上的所有Activity pop，且不用新建
    - singleInstance:全局单例，无论是在哪一个返回栈中启动目标Activity，都只会创建一个目标Activity实例而且这个时候还会使用一个新键的任务栈装载这个Activity的实例
<!--more-->
	
4. Activity生命周期
<center>![](http://p2g00vblr.bkt.clouddn.com/Activity%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F.png)</center>
    - 当一个新的Activity启动的时候，依次调用onCreate-->onStart-->onResume,进入可见状态
    - 当该Activity被覆盖的时候(如：锁屏或者被新的Activity覆盖的时候)就会调用onPause
    - 用户再返回到原Activity就会进入onResume;或者用户就进入到另一个Activity的话，这个时候就调用OnStop,并且保存信息和状态；再或者系统内存不足，拥有更高权限的应用需要内存，那么这个Activity的进程就可能会被，调用onPause,onStop，要想重新打开的话就得重新创建
    - 用户返回到onStop状态的Activity（重新可见），就会调用onRestart-->onStart-->onResume,重新运行
    - 在Activity结束或者系统杀死之前会调用onDestory释放占用的资源
5. Activity使用onSaveInstanceState方法来保存自己的状态和信息，以便回收后重建时恢复数据

# Intent Filter

Android中的三大组件(Activity、Service、BroadcastReceiver)是通过intent传递消息的。

## Intent Fliter的三大属性

1. Action
一个Intent Filter可以包含多个Action，Action列表用于标示Activity能接收的“动作”，
```
<intent-filter>
	<action android:name="android.intent.action.START"/>
	<category android:name="android.intent.category.DEFAULT"/>
</intent-filter>
```
在代码中可以使用下面语句来启动Intent对象：
```
startActivity(new Intent("android.intent.action.START"));
```
2. URL
在intent filter中，通过节点匹配外部数据，也就是通过URI携带外部数据给目标组件
```
<data android:mimeType="mimeType" 
	android:scheme="scheme" 
	 android:host="host"
	 android:port="port" 
	 android:path="path"/>
```
只有data的所有属性全部匹配成功的时候URI数据才会匹配成功
3. Category
为组件顶一个类别列表，当Intent中包含这个类别列表中的所有项目时才会匹配成功
```
<intent-filter . . . >
   <action android:name="code android.intent.action.MAIN" />
   <category android:name="code　android.intent.category.LAUNCHER" />
</intent-filter>
```
## Intent Filter匹配过程

1. 加载所有的Intent Filter列表
2. 去掉action匹配失败的Intent Filter
3. 去掉URI匹配失败的Intent Filter
4. 去掉Category匹配失败的Intent Filter
5. 判断剩下的Intent Filter数目是否为0，为0 查找失败返回异常，大于0 按照优先级排序，返回最高优先级的Intent Filter

## 显示Intent

```
Intent intent = new Intent(MainActivity.this,SecondActivity.class);
startActivity(intent);
```

## 隐式Intent

```
startActivity(new Intent(Intent.ACTION_VIEW).setData(Uri.parse("https://www.baidu.com")));
```

```
<intent-filter>
	<action android:name="android.intent.action.START"/>
	<category android:name="android.intent.category.DEFAULT"/>
</intent-filter>
startActivity(new Intent("android.intent.action.START"));
```
 
 ## Intent 传递数据
```
 Intent intent = new Intent(MainActivity.this,SecondActivity.class);
                intent.putExtra("data","boolean/byte/char/int/long/float/double/string/CharSequence/Parcelable/SerializableBundle");
                startActivity(intent);
```
```
//获取intent传递过来的数据
        Toast.makeText(SecondActivity.this,getIntent().getStringExtra("data"),Toast.LENGTH_LONG).show();
```