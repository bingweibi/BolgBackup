---
title: Android基础---广播机制
date: 2018-04-18 08:55:25
tags: 
	- Android
---

# 介绍

Android上的每一个应用程序都可以对自己感兴趣的广播进行注册，这样这个程序就可以接收到自己感兴趣的广播，广播可以是来自于系统或者来自于其他应用程序。
发送广播需要借助于Intent，接受广播则借助于BroadCastReceiver.

## 广播分类

1. 标准广播
异步进行，广播发出之后，所有的广播接收器几乎在同一时间接收到广播，无法被截断
2. 有序广播
同步执行，广播发出之后，优先级高的广播先接收到广播，优先级低的广播后接收到广播。优先级高的广播可以截断正在传递的广播，这样的话后面的广播就没有办法接收到广播。

<!--more-->

# 接收系统广播

## 注册广播分类

### 动态注册

在代码中注册，程序启动才有效，程序未启动，失效

1. 创建广播过滤器new IntentFilter(),添加一个action `android.net.conn.CONNECTIVITY_CHANGE `这样当网络发生变化的时候，系统就会发出`android.net.conn.CONNECTIVITY_CHANGE`的广播
2. 创建广播接收者`NetworkChangeReceiver`，重写`OnReceive`方法，当网络发生变化的时候就可以作出相应的逻辑
3. 注册，并绑定广播的接受者和过滤器
4. 动态注册的广播一定要记得取消注册

```
public class MainActivity extends AppCompatActivity {

    private IntentFilter mIntentFilter;
    private NetworkChangeReceiver mNetworkChangeReceiver;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mIntentFilter = new IntentFilter();
        mIntentFilter.addAction("android.net.conn.CONNECTIVITY_CHANGE");
        mNetworkChangeReceiver = new NetworkChangeReceiver();
        registerReceiver(mNetworkChangeReceiver,mIntentFilter);
    }

    @Override
    protected void onResume() {
        super.onResume();
    }

    private class NetworkChangeReceiver extends BroadcastReceiver{
        @Override
        public void onReceive(Context context, Intent intent) {
            Toast.makeText(MainActivity.this,"network change",Toast.LENGTH_SHORT).show();
        }
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        unregisterReceiver(mNetworkChangeReceiver);
    }
}

```

### 静态注册

在AndroidManifest.xml中注册，只需要手机上下载了这个应用程序之后就可以接收广播，不用管应用程序是否启动

1. 注册广播
2. 添加广播过滤器
3. 申请权限

```
<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>

<receiver
    android:name=".BootCompleteReceiver"
    android:enabled="true"
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.BOOT_COMPLETED"/>
    </intent-filter>
</receiver>
```
```
public class BootCompleteReceiver extends BroadcastReceiver {

    @Override
    public void onReceive(Context context, Intent intent) {
        Toast.makeText(context,"boot complete",Toast.LENGTH_SHORT).show();
    }
}
```

不建议在onReceive()方法中进行耗时操作，广播接收器中不允许开启线程

# 发送广播

## 发送标准广播

1. 定义一个广播接受器
2. 修改AndroidManifest.xml文件
3. 发出广播

```
mIntentFilter.addAction("com.example.bibingwei.reset.LOCAL_BROADCAST");
```

```
public class MyBroadcastReceiver extends BroadcastReceiver {

    @Override
    public void onReceive(Context context, Intent intent) {
        // TODO: This method is called when the BroadcastReceiver is receiving
        // an Intent broadcast.
        Toast.makeText(context,"接收到广播", Toast.LENGTH_SHORT).show();
    }
}
```

```
<receiver
    android:name=".MyBroadcastReceiver"
    android:enabled="true"
    android:exported="true">
    <intent-filter>
        <action android:name="com.example.bibingwei.reset.MY_BROADCAST"/>
    </intent-filter>
</receiver>
```

```
button4.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                sendBroadcast(new Intent("com.example.bibingwei.reset.MY_BROADCAST"));
            }
        });
```

## 发送有序广播

类似标准广播
只是发送的时候语句是`sendOrderedBroadcast(intent,null);`
当然可以在过滤器中设置优先级

# 本地广播

前面的属于全局广播，发出的广播可以被其他应用程序接收(ipc)，这样就带来安全问题
所以有本地广播

利用`LocalBroadcastManager`类对广播进行管理

```
mLocalBroadcastManager = LocalBroadcastManager.getInstance(this);
mLocalReceiver = new LocalReceiver();
```

```
button5.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                mLocalBroadcastManager.sendBroadcast(new Intent("com.example.bibingwei.reset.LOCAL_BROADCAST"));
            }
        });
        
mLocalBroadcastManager.registerReceiver(mLocalReceiver,mIntentFilter);
```

```
mLocalBroadcastManager.unregisterReceiver(mLocalReceiver);
```

```
private class LocalReceiver extends BroadcastReceiver {
        @Override
        public void onReceive(Context context, Intent intent) {
            Toast.makeText(context,"接收到了本地广播",Toast.LENGTH_SHORT).show();
        }
    }
```

本地广播是无法通过静态注册的来接收
