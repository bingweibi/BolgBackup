---
title: Android基础---Service
date: 2018-04-18 10:57:29
tags: 
	- Android
---

# 介绍Service

1. Service是Android中应用程序后台运行的解决方案，适合在执行一些不需要用户交互但是还需要长时间运行的任务上。
2. Service运行在UI线程上，所以不建议在服务中进行耗时操作，当然你还是可以再服务中开辟一个子线程进行一些耗时操作
3. Service不依赖于用户界面，当程序被切换到后台的话，依然是可以正常运行。当程序被杀死的时候，服务也会停止运行

<!--more-->

# 普通的Service

1. 新建服务
2. 启动服务
3. 停止服务

```
public class MyService extends Service {

    public MyService() {
    }

    @Override
    public void onCreate() {
        super.onCreate();
    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        doSomeWork();
        return super.onStartCommand(intent, flags, startId);
    }

    //逻辑操作
    private void doSomeWork() {
        new Thread(new Runnable() {
            @Override
            public void run() {

            }
        }).start();
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
    }

    @Override
    public IBinder onBind(Intent intent) {
        // TODO: Return the communication channel to the service.
        throw new UnsupportedOperationException("Not yet implemented");
    }
}
```

```
startService(new Intent(MainActivity.this, MyService.class));
```

```
stopService(new Intent(MainActivity.this,MyService.class));
```

1. onCreate方法在服务第一次创建的时候被调用，onStartCommand方法在每次服务启动的时候都会调用，onDestory方法在服务销毁的时候调用

# 前台服务

前台服务被回收的优先级较低，不容易被回收
```
//前台服务
Intent intent = new Intent(this, MainActivity.class);
PendingIntent pendingIntent = PendingIntent.getActivity(this,0,intent,0);
if (android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.JELLY_BEAN) {
    Notification notification = new Notification.Builder(this)
            .setContentTitle("content title")
            .setContentText("content text")
            .setWhen(System.currentTimeMillis())
            .setSmallIcon(R.mipmap.ic_launcher)
            .setLargeIcon(BitmapFactory.decodeResource(getResources(),R.mipmap.ic_launcher))
            .setContentIntent(pendingIntent)
            .build();
    startForeground(1,notification);
}
```

1. 使用startForeground方法，将Service设置为前台服务，并在系统的状态栏显示出来

# IntentService

1. `IntentService`可以在结束的时候自动结束Service
2. 不在主线程中
3. 处理耗时操作在`onHandlerIntent`中

```
public class MyIntentService extends IntentService {


    public MyIntentService(String name) {
        super(name);
    }

    @Override
    protected void onHandleIntent(@Nullable Intent intent) {
        //耗时的操作
    }
}
```

# Activity和Service进行通信

1. 在Activity中与Service 取得联系，并进行操作
2. 随之进行绑定或者解绑

```
//和服务进行关联,进行逻辑操作
    private ServiceConnection mServiceConnection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            mDownloadBinder = (MyService.DownloadBinder) service;
            mDownloadBinder.getProgress();
            mDownloadBinder.startDownload();
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {

        }
    };
```

```
bindService(new Intent(MainActivity.this,MyService.class),mServiceConnection,BIND_AUTO_CREATE);

unbindService(mServiceConnection);
```

# Service生命周期

1. startService
onCreate-->onStartCommand-->onStopService-->onDestory
2. bindService
onCreate-->onBind-->unBind-->onDestory