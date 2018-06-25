---
title: Android notes
date: 2018-01-20 18:38:11
tags: [Android]
---

1. Activity生命周期
 <center>![Activity生命周期](http://p2g00vblr.bkt.clouddn.com/activity%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F.png)</center>
 <!-- more -->
- 启动Activity:onCreate()->onStart()->onResume(),Activity进入运行状态
- Activity退居后台:当前Activity转到新的Activity界面或按Home键回到主屏:onPause()->onResume(),再次回到运行状态
- Activity返回前台：onRrstart()->onStart()->onResume(),再次回到运行状态
- Activity退居回台，且系统内存不足，系统会杀死这个后台状态的Activity（此时这个Activity引用仍然处在任务栈中，只是这个时候引用指向的对象已经是null),若再次回到这个Activity，则会走onCreate()->onStart()->onResume()
- 锁屏:onPause()->onStop()
- 解锁:onStart()->onResume()

2.android任务栈

- standard:标准模式，每次启动Activity都会创建一个新的Activity实例，并且将其压入任务栈栈顶，而不管这个Activity是否已经存在。Activity的启动三个回调函数(onCreate()->onStart()->onRrsume())都会执行
<center>![standard模式](http://p2g00vblr.bkt.clouddn.com/standard%E6%A8%A1%E5%BC%8F.png)</center>
*从上图可以看出，在standard模式下启动了三次MainActivity后，都生成了不同的新实例，并天剑到同一个任务栈中*

- singleTop:栈顶复用模式。这种模式下，如果新的Activity已经位于任务栈的栈顶，那么此Activity不会重新被创建，所以他的启动三个回调函数不会被执行，同事Activity的onNewIntent()方法会被回调。如果Activity已经存在但是不存在栈顶，那么作用与standard模式一样
<center>![singleTop模式Activity位于栈顶](http://p2g00vblr.bkt.clouddn.com/singleTop%E6%A8%A1%E5%BC%8F%E4%BD%8D%E4%BA%8E%E6%A0%88%E9%A1%B6.png)</center>
*singleTop模式Activity位于栈顶，当需要新创建的MainActivity位于栈顶时，MainActivity并没有创建*
<center>![singleTop模式Actvity没有位于栈顶](http://p2g00vblr.bkt.clouddn.com/singleTop%E6%A8%A1%E5%BC%8F%E6%B2%A1%E6%9C%89%E4%BD%8D%E4%BA%8E%E6%A0%88%E9%A1%B6.png)</center>
*需要创建的MainActivity没有位于栈顶时，在栈顶新建一个MainActivity*
- singTask:栈内复用模式。创建这样的Activity的时候，系统会先确认他所需任务栈是否已经创建，否则会先创建任务栈，然后放入Activity。如果栈中已经有一个Activity实例，那么这个Activity就会被调到栈顶，onNewIntent(),并且singleTask会清理在当前Activity上面的所有Activity
<center>![singleTask模式](http://p2g00vblr.bkt.clouddn.com/singleTask%E6%A8%A1%E5%BC%8F.png)</center>
*当我们再次启动MainActivity时，由于MainActivity位于栈中，所以系统直接将其置于栈顶，并移除其上方的所有Activity。当然如果所需要的MainActivity不存在栈中，则会创建新的Activity并添加到栈中。*

- singleInstance：加强版的singleTask模式，这种模式的Activity只能单独位于一个任务栈内，由于栈内复用的特性，后续请求均不会创建新的Activity，除非这个独特的任务栈被系统销毁.换句话说，，A应用需要启动的MainActivity 是singleInstance模式，当A启动后，系统会为它创建一个新的任务栈，然后A单独在这个新的任务栈中，如果此时B应用也要激活MainActivity，由于栈内复用的特性，则不会重新创建，而是两个应用共享一个Activity的实例。
<cneter>![SingleInstance](http://p2g00vblr.bkt.clouddn.com/singleInstance%E6%A8%A1%E5%BC%8F.png)</center>

- 来自[francistao](https://github.com/francistao/LearningNotes/blob/master/Part1/Android/Android%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86.md)
- 来自[javazejian](http://blog.csdn.net/javazejian/article/details/52071885)


