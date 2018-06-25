---
title: RxJava2.0教程
date: 2018-05-23 10:14:30
tags:	RxJava2.0
---

# 本文章全部摘抄于 https://www.jianshu.com/p/464fa025229e

# 观察者模式

<center>![](http://p2g00vblr.bkt.clouddn.com/RxJava1.png)</center>

上面的管道，我们称它为`上游`，下面的管道我们称之为`下游`。在RxJava中分别对应着`observable`(被观察者)和`observer`(观察者)。被观察者/观察者之间通过订阅(`subscribe()`)来进行关联。

<!--more-->

举个栗子：
```
public static void demo1() {
        //创建一个上游 Observable：
        Observable<Integer> observable = Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
                emitter.onNext(1);
                emitter.onNext(2);
                emitter.onNext(3);
                emitter.onComplete();
            }
        });
        //创建一个下游 Observer
        Observer<Integer> observer = new Observer<Integer>() {
            @Override
            public void onSubscribe(Disposable d) {
                Log.i(TAG, "subscribe");
            }

            @Override
            public void onNext(Integer value) {
                Log.i(TAG, "" + value);
            }

            @Override
            public void onError(Throwable e) {
                Log.i(TAG, "error");
            }

            @Override
            public void onComplete() {
                Log.i(TAG, "complete");
            }
        };
        //建立连接
        observable.subscribe(observer);
    }
```

输出结果
```
I/TAG: subscribe
I/TAG: 1
I/TAG: 2
       3
       complete
```

*subscribe-->被观察者发送数据-->观察者接收数据-->complete*

将上面的程序换成RxJava的链式操作：
```
public static void demo2() {
        Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
                emitter.onNext(1);
                emitter.onNext(2);
                emitter.onNext(3);
                emitter.onComplete();
            }
        }).subscribe(new Observer<Integer>() {
            @Override
            public void onSubscribe(Disposable d) {
                Log.i(TAG, "subscribe");
            }

            @Override
            public void onNext(Integer value) {
                Log.i(TAG, "" + value);
            }

            @Override
            public void onError(Throwable e) {
                Log.i(TAG, "error");
            }

            @Override
            public void onComplete() {
                Log.i(TAG, "complete");
            }
        });
    }
```

运行结果：
```
I/TAG: subscribe
    1
    2
    3
    complete
```

# 在上面程序中出现了`ObservableEmitter`和`Disposable`

## ObservableEmitter:被观察者用来发射事件，通常调用emitter的Next事件`onNext(Tvalue)` complete事件`onComplete()`和error事件`onError(Throwable error)`

被观察者发送事件的一些规则：
1. 被观察者可以发送无限多个`onNext`事件，观察者也可以接收无限多个`onNext`事件
2. 当被观察者发送了`onComplete`事件之后，被观察者还可以发送`onComplete`之后的事件，但是观察者是不会再继续接收`onComplete`之后的事件
3. 当被观察者发送了`onError`事件之后,被观察者还可以发送`onError`之后的事件，但是观察者是不会再继续接收`onError`之后的事件
4. 被观察者可以不发送`onComplete`和`onError`事件
5. 从2和3 可以看出`onComplete`和`onError`事件是唯一且互斥的，也就是说不能发送多个`onComnplete`事件/`onError`事件,而且在一个发送事件中只能存在一个`onComplete`或者`onError`事件

>注: 关于onComplete和onError唯一并且互斥这一点, 是需要自行在代码中进行控制, 如果你的代码逻辑中违背了这个规则, **并不一定会导致程序崩溃.** 比如发送多个onComplete是可以正常运行的, 依然是收到第一个onComplete就不再接收了, 但若是发送多个onError, 则收到第二个onError事件会导致程序会崩溃.

只发送onNext事件
<center>![](http://p2g00vblr.bkt.clouddn.com/%E5%8F%91%E9%80%81onNext%E4%BA%8B%E4%BB%B6.png)</center>

发送onComplete事件
<center>![](http://p2g00vblr.bkt.clouddn.com/%E5%90%AB%E6%9C%89onComplete.png)</center>

发送onError事件
<center>![](http://p2g00vblr.bkt.clouddn.com/%E5%90%AB%E6%9C%89onError.png)</center>

## Disposable:字面意思(一次性用品，用完可丢弃)，在Rxjava中，当调用`dispose()`方法的时候，它会将上面所讲的管道切断，观察者接收不到事件。

举个栗子
```
//调用dispose()并不会导致上游不再继续发送事件, 上游会继续发送剩余的事件.
    public static void demo3() {
        Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
                Log.i(TAG, "emit 1");
                emitter.onNext(1);
                Log.i(TAG, "emit 2");
                emitter.onNext(2);
                Log.i(TAG, "emit 3");
                emitter.onNext(3);
                Log.i(TAG, "emit complete");
                emitter.onComplete();
                Log.i(TAG, "emit 4");
                emitter.onNext(4);
            }
        }).subscribe(new Observer<Integer>() {
            private Disposable mDisposable;
            private int i;

            @Override
            public void onSubscribe(Disposable d) {
                Log.i(TAG, "subscribe");
                mDisposable = d;
            }

            @Override
            public void onNext(Integer value) {
                Log.i(TAG, "onNext: " + value);
                i++;
                if (i == 2) {
                    Log.i(TAG, "dispose");
                    mDisposable.dispose();
                    Log.i(TAG, "isDisposed : " + mDisposable.isDisposed());
                }
            }

            @Override
            public void onError(Throwable e) {
                Log.i(TAG, "error");
            }

            @Override
            public void onComplete() {
                Log.i(TAG, "complete");
            }
        });
    }
```

输出结果
```
I/TAG: subscribe
    emit 1
I/TAG: onNext: 1
    emit 2
    onNext: 2
    dispose
    isDisposed : true
    emit 3
    emit complete
    emit 4
```

在发送2之后，我们调用`dispose()`切断水管，但是被观察者还是在继续发送事件。

# subscribe()有多个重载的方法:
```
    public final Disposable subscribe() {}
    public final Disposable subscribe(Consumer<? super T> onNext) {}
    public final Disposable subscribe(Consumer<? super T> onNext, Consumer<? super Throwable> onError) {} 
    public final Disposable subscribe(Consumer<? super T> onNext, Consumer<? super Throwable> onError, Action onComplete) {}
    public final Disposable subscribe(Consumer<? super T> onNext, Consumer<? super Throwable> onError, Action onComplete, Consumer<? super Disposable> onSubscribe) {}
    public final void subscribe(Observer<? super T> observer) {}
```

1. 不带任何参数的subscribe() 表示下游不关心任何事件,你上游尽管发你的数据去吧, 老子可不管你发什么.
2. 带有一个`Consumer`参数的方法表示下游只关心onNext事件, 其他的事件我假装没看见, 因此我们如果只需要`onNext`事件可以这么写:
```
Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
                Log.d(TAG, "emit 1");
                emitter.onNext(1);
                Log.d(TAG, "emit 2");
                emitter.onNext(2);
                Log.d(TAG, "emit 3");
                emitter.onNext(3);
                Log.d(TAG, "emit complete");
                emitter.onComplete();
                Log.d(TAG, "emit 4");
                emitter.onNext(4);
            }
        }).subscribe(new Consumer<Integer>() {
            @Override
            public void accept(Integer integer) throws Exception {
                Log.d(TAG, "onNext: " + integer);
            }
        });
```

# RxJava 线程控制

当我们在主线程中创建一个被观察者Observable来发送事件，那么被观察者就会默认在主线程发送事件，同理在主线程创建一个观察者Observer来接收事件，那么观察者就默认在主线程中接收事件。
```
    //同一线程下的
    public static void demo1() {
        Observable<Integer> observable = Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
                Log.i(TAG, "Observable thread is : " + Thread.currentThread().getName());
                Log.i(TAG, "emit 1");
                emitter.onNext(1);
            }
        });

        Consumer<Integer> consumer = new Consumer<Integer>() {
            @Override
            public void accept(Integer integer) throws Exception {
                Log.i(TAG, "Observer thread is :" + Thread.currentThread().getName());
                Log.i(TAG, "onNext: " + integer);
            }
        };

        observable.subscribe(consumer);
    }
```

输出结果
```
I/TAG: Observable thread is : main
                        emit 1
       Observer thread is :main
                        onNext: 1
```

如果我们希望在子线程中做耗时操作，然后回到主线程中更新UI

<center>![](http://p2g00vblr.bkt.clouddn.com/RxJava%E4%B8%BB%E7%BA%BF%E7%A8%8B%E5%92%8C%E5%AD%90%E7%BA%BF%E7%A8%8B.png)</center>

黄色的管道代表子线程，蓝色管道代表主线程。

想要达到这样的效果，那么我们就需要先去改变被观察者所处的线程，再去改变观察者的线程。
```
public static void demo2() {
        Observable<Integer> observable = Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
                Log.i(TAG, "Observable thread is : " + Thread.currentThread().getName());
                Log.i(TAG, "emit 1");
                emitter.onNext(1);
            }
        });

        Consumer<Integer> consumer = new Consumer<Integer>() {
            @Override
            public void accept(Integer integer) throws Exception {
                Log.i(TAG, "Observer thread is :" + Thread.currentThread().getName());
                Log.i(TAG, "onNext: " + integer);
            }
        };

        //指定在不同的线程下操作
        observable.subscribeOn(Schedulers.newThread())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(consumer);
    }
```

输出结果
```
I/TAG: Observable thread is : RxNewThreadScheduler-1
    emit 1
I/TAG: Observer thread is :main
    onNext: 1
```
在上面的输出结果中我们可以看到，被观察者处于的线程是`RxNewThreadScheduler-1`,观察者的线程是`Observer thread is :main`

起作用的代码
```
        //指定在不同的线程下操作
        observable.subscribeOn(Schedulers.newThread())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(consumer);
```
上面的代码中第二行指定了被观察者的线程，第三行指定了观察者的线程。
多次指定被观察者的线程只有第一次指定的有效，如果多次指定下游线程，那么在每调用一次`observeOn()`，观察者的线程就会切换一次
```
public static void demo3() {
        Observable<Integer> observable = Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
                Log.i(TAG, "Observable thread is : " + Thread.currentThread().getName());
                Log.i(TAG, "emit 1");
                emitter.onNext(1);
            }
        });

        Consumer<Integer> consumer = new Consumer<Integer>() {
            @Override
            public void accept(Integer integer) throws Exception {
                Log.i(TAG, "Observer thread is :" + Thread.currentThread().getName());
                Log.i(TAG, "onNext: " + integer);
            }
        };

        //多次指定被观察者的线程，以第一次有效
        //多次指定观察者的线程，都有效
        observable.subscribeOn(Schedulers.newThread())
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .doOnNext(new Consumer<Integer>() {
                    @Override
                    public void accept(Integer integer) throws Exception {
                        Log.i(TAG, "After observeOn(mainThread), current thread is: " + Thread.currentThread()
                                .getName());
                    }
                })
                .observeOn(Schedulers.io())
                .doOnNext(new Consumer<Integer>() {
                    @Override
                    public void accept(Integer integer) throws Exception {
                        Log.i(TAG, "After observeOn(io), current thread is : " + Thread.currentThread().getName());
                    }
                })
                .subscribe(consumer);
    }
```
输出结果
```
Observable thread is : RxNewThreadScheduler-1
emit 1
After observeOn(mainThread), current thread is: main
After observeOn(io), current thread is : RxCachedThreadScheduler-2
Observer thread is :RxCachedThreadScheduler-2
onNext: 1
```

在RxJava中，已经内置了很多线程选项供我们选择
1. `Schedulers.io()`代表io操作的线程，通常用于网络，读写文件等io密集型的操作
2. `Schedules.computation()`代表CPU计算密集型的操作，例如需要大量计算的操作
3. `Schedulers.newThread()`代表一个常规的新线程
4. `AndroidSchedulers.mainThread()`代表Android的主线程

# RxJava 变换操作符map

map操作符:对被观察者发送的每一个事件应用一个函数，使的每一个事件都按照指定的函数去变化。
<center>![](http://p2g00vblr.bkt.clouddn.com/RxJavamap%E6%93%8D%E4%BD%9C%E7%AC%A6.png)</center>

举个栗子
```
//map操作符Integer->String
    public static void demo1() {
        Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
                emitter.onNext(1);
                emitter.onNext(2);
                emitter.onNext(3);
            }
        }).map(new Function<Integer, String>() {
            @Override
            public String apply(Integer integer) throws Exception {
                return "This is result " + integer;
            }
        }).subscribe(new Consumer<String>() {
            @Override
            public void accept(String s) throws Exception {
                Log.i(TAG, s);
            }
        });
    }
```

输出结果
```
This is result 1
This is result 2
This is result 3
```
# RxJava flatMap操作符
flatMap操作符：将一个发送事件的被观察者变换为多个发送事件的被观察者，然后将他们发送的事件合并后放进一个单独的被观察者中。
很难理解是吧，我也看不懂，直接看图
<center>![](http://p2g00vblr.bkt.clouddn.com/RxJavaflatMap%E6%93%8D%E4%BD%9C%E7%AC%A6.png)</center>

中间flatMap的作用是将圆形的事件转换为一个发送矩形事件和三角形事件的新的被观察者Observable.

<center>![](http://p2g00vblr.bkt.clouddn.com/RxJavaflatMap2.png)</center>

被观察者每发送一个事件，flatMap都会新建一个管道，然后发送转换之后的新事件，观察者收到的就是新的管道发送的数据。**faltMap并不保证事件的顺序**，如果想保证顺序的使用`concatMap`

举个栗子：
```
//flatMap:Integer->Integer+String,不保证顺序
    public static void demo2() {
        Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
                emitter.onNext(1);
                emitter.onNext(2);
                emitter.onNext(3);
            }
        }).flatMap(new Function<Integer, ObservableSource<String>>() {
            @Override
            public ObservableSource<String> apply(Integer integer) throws Exception {
                final List<String> list = new ArrayList<>();
                for (int i = 0; i < 3; i++) {
                    list.add("I am value " + integer);
                }
                return Observable.fromIterable(list).delay(10, TimeUnit.MILLISECONDS);
            }
        }).subscribe(new Consumer<String>() {
            @Override
            public void accept(String s) throws Exception {
                Log.i(TAG, s);
            }
        });
    }
```

输出结果
```
I am value 1
I am value 1
I am value 1
I am value 2
I am value 2
I am value 3
I am value 3
I am value 3
I am value 2
```
# RxJava concatMap操作符
与flatMap作用一样，只是保证了顺序不变

举个栗子：
```
//保证顺序的flatMap
    public static void demo3() {
        Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
                emitter.onNext(1);
                emitter.onNext(2);
                emitter.onNext(3);
            }
        }).concatMap(new Function<Integer, ObservableSource<String>>() {
            @Override
            public ObservableSource<String> apply(Integer integer) throws Exception {
                final List<String> list = new ArrayList<>();
                for (int i = 0; i < 3; i++) {
                    list.add("I am value " + integer);
                }
                return Observable.fromIterable(list).delay(10, TimeUnit.MILLISECONDS);
            }
        }).subscribe(new Consumer<String>() {
            @Override
            public void accept(String s) throws Exception {
                Log.i(TAG, s);
            }
        });
    }
```

运行结果
```
I am value 1
I am value 1
I am value 1
I am value 2
I am value 2
I am value 2
I am value 3
I am value 3
I am value 3
```

# RxJava Zip操作符
`Zip`通过一个函数将多个被观察者发送的事件结合在一起，然后将这些组合在一起的事件发送。它将严格按照顺序发送与发送数据项最少的那个被观察者一样多的数据。

<center>![](http://p2g00vblr.bkt.clouddn.com/RxJavaZip.png)</center>
<center>![](http://p2g00vblr.bkt.clouddn.com/RxJavaZip2.png)</center>
过程：
1. 首先分别从两根管道中各取出一个事件来进行组合，并且一个事件只能被使用一次组合的顺序是严格按照事件发送的顺序来进行的。
2. 最终，观察者接收到的事件数量是和被观察者中发送事件最少的那一根管道的事件数量相同。

举个栗子：
```
//zip操作符:Integer + String -> 第三种类型（组合操作，以少的一方为标准）
    public static void demo1() {
        Observable<Integer> observable1 = Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
                Log.i(TAG, "emit 1");
                emitter.onNext(1);
                Log.i(TAG, "emit 2");
                emitter.onNext(2);
                Log.i(TAG, "emit 3");
                emitter.onNext(3);
                Log.i(TAG, "emit 4");
                emitter.onNext(4);
                Log.i(TAG, "emit complete1");
                emitter.onComplete();
            }
        });

        Observable<String> observable2 = Observable.create(new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(ObservableEmitter<String> emitter) throws Exception {
                Log.i(TAG, "emit A");
                emitter.onNext("A");
                Log.i(TAG, "emit B");
                emitter.onNext("B");
                Log.i(TAG, "emit C");
                emitter.onNext("C");
                Log.i(TAG, "emit complete2");
                emitter.onComplete();
            }
        });

        Observable.zip(observable1, observable2, new BiFunction<Integer, String, String>() {
            @Override
            public String apply(Integer integer, String s) throws Exception {
                return integer + s;
            }
        }).subscribe(new Observer<String>() {
            @Override
            public void onSubscribe(Disposable d) {
                Log.i(TAG, "onSubscribe");
            }

            @Override
            public void onNext(String value) {
                Log.i(TAG, "onNext: " + value);
            }

            @Override
            public void onError(Throwable e) {
                Log.i(TAG, "onError");
            }

            @Override
            public void onComplete() {
                Log.i(TAG, "onComplete");
            }
        });
    }
```

运行结果
```
onSubscribe
emit 1
emit 2
emit 3
emit 4
emit complete1
emit A
onNext: 1A
emit B
onNext: 2B
emit C
onNext: 3C
emit complete2
onComplete
```
因为两个管道都是出于一个线程中，所以是先发送完管道1，再发送管道2。
如果两个管道不在一个线程：
```
 public static void demo3() {
        Observable<Integer> observable1 = Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
                Log.i(TAG, "emit 1");
                emitter.onNext(1);
                Thread.sleep(1000);

                Log.i(TAG, "emit 2");
                emitter.onNext(2);
                Thread.sleep(1000);

                Log.i(TAG, "emit 3");
                emitter.onNext(3);
                Thread.sleep(1000);

                Log.i(TAG, "emit 4");
                emitter.onNext(4);
                Thread.sleep(1000);

                Log.i(TAG, "emit complete1");
                emitter.onComplete();
            }
        }).subscribeOn(Schedulers.io());

        Observable<String> observable2 = Observable.create(new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(ObservableEmitter<String> emitter) throws Exception {
                Log.i(TAG, "emit A");
                emitter.onNext("A");
                Thread.sleep(1000);

                Log.i(TAG, "emit B");
                emitter.onNext("B");
                Thread.sleep(1000);

                Log.i(TAG, "emit C");
                emitter.onNext("C");
                Thread.sleep(1000);

                Log.i(TAG, "emit complete2");
                emitter.onComplete();
            }
        }).subscribeOn(Schedulers.io());

        Observable.zip(observable1, observable2, new BiFunction<Integer, String, String>() {
            @Override
            public String apply(Integer integer, String s) throws Exception {
                return integer + s;
            }
        }).subscribe(new Observer<String>() {
            @Override
            public void onSubscribe(Disposable d) {
                Log.i(TAG, "onSubscribe");
            }

            @Override
            public void onNext(String value) {
                Log.i(TAG, "onNext: " + value);
            }

            @Override
            public void onError(Throwable e) {
                Log.i(TAG, "onError");
            }

            @Override
            public void onComplete() {
                Log.i(TAG, "onComplete");
            }
        });
    }
```
运行结果
```
onSubscribe
emit A
emit 1
onNext: 1A
emit B
emit 2
onNext: 2B
emit C
emit 3
onNext: 3C
emit complete2
onComplete
```

# Zip详解
在上一节中，我们说到`Zip`操作符的作用是将多个被观察者发送的事件组合起来发送给观察者。那如果在被观察着中，有一个管道发送事件特别快，一个特别慢，会出现什么情况？在`Zip`中使用一个缓冲区来进行缓冲。
<center>![](http://p2g00vblr.bkt.clouddn.com/RxJavaZip3.png)</center>

如果在被观察者中有一个管道无限循环发送事件，另一个管道只发送少量事件，会出现什么情况？
```
/解决一方发送事件过快导致OOM
    public static void demo1() {
        Observable<Integer> observable1 = Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
                for (int i = 0; ; i++) {
                    emitter.onNext(i);
                }
            }
        }).subscribeOn(Schedulers.io());

        Observable<String> observable2 = Observable.create(new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(ObservableEmitter<String> emitter) throws Exception {
                emitter.onNext("A");
            }
        }).subscribeOn(Schedulers.io());

        Observable.zip(observable1, observable2, new BiFunction<Integer, String, String>() {
            @Override
            public String apply(Integer integer, String s) throws Exception {
                return integer + s;
            }
        }).observeOn(AndroidSchedulers.mainThread()).subscribe(new Consumer<String>() {
            @Override
            public void accept(String s) throws Exception {
                Log.d(TAG, s);
            }
        }, new Consumer<Throwable>() {
            @Override
            public void accept(Throwable throwable) throws Exception {
                Log.w(TAG, throwable);
            }
        });
    }
```
运行结果
<center>![](http://p2g00vblr.bkt.clouddn.com/RxJavaZip4.gif)</center>

如果被观察者和观察者处于同一个线程，而且观察者在每次接收事件前延时2秒
```
public static void demo2() {
        Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
                for (int i = 0; ; i++) {
                    emitter.onNext(i);
                }
            }
        }).subscribe(new Consumer<Integer>() {
            @Override
            public void accept(Integer integer) throws Exception {
                Thread.sleep(2000);
                Log.d(TAG, "" + integer);
            }
        });
    }
```
运行结果
<center>![](http://p2g00vblr.bkt.clouddn.com/RxJavaZip5.gif)</center>
如果两者不是同一个线程的话
```
public static void demo3() {
        Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
                for (int i = 0; ; i++) {
                    emitter.onNext(i);
                }
            }
        }).subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Consumer<Integer>() {
                    @Override
                    public void accept(Integer integer) throws Exception {
                        Log.d(TAG, "" + integer);
                    }
                });
    }
```
运行结果
<center>![](http://p2g00vblr.bkt.clouddn.com/RxJavaZip6.gif)</center>
同样还是出现了oom

同步的时候
<center>![](http://p2g00vblr.bkt.clouddn.com/RxJavaZip%E5%90%8C%E6%AD%A5.png)</center>
异步的时候
<center>![](http://p2g00vblr.bkt.clouddn.com/RxJavaZip%E5%BC%82%E6%AD%A5.png)</center>

只要有缓冲区的存在就会出现OOM

# 解决Zip中出现OOM的情况

我们首先在被观察者中增加一个`filter`，只允许能被10整除的事件通过
```
public static void demo2() {
        Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
                for (int i = 0; ; i++) {
                    emitter.onNext(i);
                }
            }
        }).subscribeOn(Schedulers.io())
                .filter(new Predicate<Integer>() {
                    @Override
                    public boolean test(Integer integer) throws Exception {
                        return integer % 10 == 0;
                    }
                })
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Consumer<Integer>() {
                    @Override
                    public void accept(Integer integer) throws Exception {
                        Log.d(TAG, "" + integer);
                    }
                });
    }
```
运行结果
<center>![](http://p2g00vblr.bkt.clouddn.com/RxJavaZip%E6%95%B0%E9%87%8F%E8%A7%A3%E5%86%B3.gif)</center>

在被观察者中加入一个`sample`操作符,每隔指定时间从被观察者中取出一个事件发送给观察者
```
public static void demo3() {
        Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
                for (int i = 0; ; i++) {
                    emitter.onNext(i);
                }
            }
        }).subscribeOn(Schedulers.io())
                .sample(2, TimeUnit.SECONDS)
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Consumer<Integer>() {
                    @Override
                    public void accept(Integer integer) throws Exception {
                        Log.d(TAG, "" + integer);
                    }
                });
    }
```
运行结果
<center>![](http://p2g00vblr.bkt.clouddn.com/RxJavaZipsample.gif)</center>

前面这两种方法归根到底其实就是减少放进水缸的事件的数量, 是以**数量**取胜, 但是这个方法有个缺点, 就是丢失了大部分的事件

现在我们从速度上取胜，每次被观察者发送完事件后都延时2秒
```
public static void demo4() {
        Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
                for (int i = 0; ; i++) {
                    emitter.onNext(i);
                    Thread.sleep(2000);
                }
            }
        }).subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Consumer<Integer>() {
                    @Override
                    public void accept(Integer integer) throws Exception {
                        Log.d(TAG, "" + integer);
                    }
                });
    }
```
运行结果
<center>![](http://p2g00vblr.bkt.clouddn.com/RaJavaZip%E9%80%9F%E5%BA%A6.gif)</center>

我们分别利用从数量和速度的方法来解决Zip中出现的OOM问题
数量上解决
```
public static void demo5() {
        Observable<Integer> observable1 = Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
                for (int i = 0; ; i++) {
                    emitter.onNext(i);
                }
            }
        }).subscribeOn(Schedulers.io()).sample(2, TimeUnit.SECONDS);

        Observable<String> observable2 = Observable.create(new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(ObservableEmitter<String> emitter) throws Exception {
                emitter.onNext("A");
            }
        }).subscribeOn(Schedulers.io());

        Observable.zip(observable1, observable2, new BiFunction<Integer, String, String>() {
            @Override
            public String apply(Integer integer, String s) throws Exception {
                return integer + s;
            }
        }).observeOn(AndroidSchedulers.mainThread()).subscribe(new Consumer<String>() {
            @Override
            public void accept(String s) throws Exception {
                Log.d(TAG, s);
            }
        }, new Consumer<Throwable>() {
            @Override
            public void accept(Throwable throwable) throws Exception {
                Log.w(TAG, throwable);
            }
        });
    }
```
运行结果
<center>![](http://p2g00vblr.bkt.clouddn.com/RxJava%E6%95%B0%E9%87%8F%E8%A7%A3%E5%86%B3.gif)</center>

速度解决
```
public static void demo6() {
        Observable<Integer> observable1 = Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
                for (int i = 0; ; i++) {
                    emitter.onNext(i);
                    Thread.sleep(2000);
                }
            }
        }).subscribeOn(Schedulers.io());

        Observable<String> observable2 = Observable.create(new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(ObservableEmitter<String> emitter) throws Exception {
                emitter.onNext("A");
            }
        }).subscribeOn(Schedulers.io());

        Observable.zip(observable1, observable2, new BiFunction<Integer, String, String>() {
            @Override
            public String apply(Integer integer, String s) throws Exception {
                return integer + s;
            }
        }).observeOn(AndroidSchedulers.mainThread()).subscribe(new Consumer<String>() {
            @Override
            public void accept(String s) throws Exception {
                Log.d(TAG, s);
            }
        }, new Consumer<Throwable>() {
            @Override
            public void accept(Throwable throwable) throws Exception {
                Log.w(TAG, throwable);
            }
        });
    }
```
运行结果
<center>![](http://p2g00vblr.bkt.clouddn.com/RxJava%E9%80%9F%E5%BA%A6.gif)</center>

# RxJava Flowable操作符
基本用法
```
 public static void demo1() {
        Flowable<Integer> upstream = Flowable.create(new FlowableOnSubscribe<Integer>() {
            @Override
            public void subscribe(FlowableEmitter<Integer> emitter) throws Exception {
                Log.i(TAG, "emit 1");
                emitter.onNext(1);
                Log.i(TAG, "emit 2");
                emitter.onNext(2);
                Log.i(TAG, "emit 3");
                emitter.onNext(3);
                Log.i(TAG, "emit complete");
                emitter.onComplete();
            }
        }, BackpressureStrategy.ERROR);

        Subscriber<Integer> downstream = new Subscriber<Integer>() {

            @Override
            public void onSubscribe(Subscription s) {
                Log.i(TAG, "onSubscribe");
                s.request(Long.MAX_VALUE);
            }

            @Override
            public void onNext(Integer integer) {
                Log.i(TAG, "onNext: " + integer);
            }

            @Override
            public void onError(Throwable t) {
                Log.i(TAG, "onError: ", t);
            }

            @Override
            public void onComplete() {
                Log.i(TAG, "onComplete");
            }
        };

        upstream.subscribe(downstream);
    }
```
运行结果
```
onSubscribe
emit 1
onNext: 1
emit 2
onNext: 2
emit 3
onNext: 3
emit complete
onComplete
```
与之前的不同之处
1. 被观察者改为`Flowable`
2. 观察者改为`Subscriber`
3. 创建`Flowable`的时候增加一个参数，这个参数是用来选择背压的，就是在出现发送事件和接收事件速度不一致的情况下处理的办法，这里我们直接使用`BackpressureStrategy.ERROR`,不一致的话就直接抛异常
4. 在观察者的`onSubscribe()`中，传给我们的是`Subscription`而不是`Disposable`。两者之间的区别是`Subscription`增加了一个`void request(long n)`
没有设置`void request(long n)`
```
public static void demo2() {
        Flowable.create(new FlowableOnSubscribe<Integer>() {
            @Override
            public void subscribe(FlowableEmitter<Integer> emitter) throws Exception {
                Log.i(TAG, "emit 1");
                emitter.onNext(1);
                Log.i(TAG, "emit 2");
                emitter.onNext(2);
                Log.i(TAG, "emit 3");
                emitter.onNext(3);
                Log.i(TAG, "emit complete");
                emitter.onComplete();
            }
        }, BackpressureStrategy.ERROR).subscribe(new Subscriber<Integer>() {

            @Override
            public void onSubscribe(Subscription s) {
                Log.i(TAG, "onSubscribe");
            }

            @Override
            public void onNext(Integer integer) {
                Log.i(TAG, "onNext: " + integer);

            }

            @Override
            public void onError(Throwable t) {
                Log.i(TAG, "onError: ", t);
            }

            @Override
            public void onComplete() {
                Log.i(TAG, "onComplete");
            }
        });
    }
```

运行结果
```
onSubscribe
emit 1
onError: 
                              io.reactivex.exceptions.MissingBackpressureException: create: could not emit value due to lack of requests
                                  at io.reactivex.internal.operators.flowable.FlowableCreate$ErrorAsyncEmitter.onOverflow(FlowableCreate.java:411)
                                  at io.reactivex.internal.operators.flowable.FlowableCreate$NoOverflowBaseAsyncEmitter.onNext(FlowableCreate.java:377)
                                  at zlc.season.rxjava2demo.demo.ChapterSeven$3.subscribe(ChapterSeven.java:77)
                                  at io.reactivex.internal.operators.flowable.FlowableCreate.subscribeActual(FlowableCreate.java:72)
                                  at io.reactivex.Flowable.subscribe(Flowable.java:12218)
                                  at zlc.season.rxjava2demo.demo.ChapterSeven.demo2(ChapterSeven.java:111)
                                  at zlc.season.rxjava2demo.MainActivity$2.onClick(MainActivity.java:36)
                                  at android.view.View.performClick(View.java:5637)
                                  at android.view.View$PerformClick.run(View.java:22429)
                                  at android.os.Handler.handleCallback(Handler.java:751)
                                  at android.os.Handler.dispatchMessage(Handler.java:95)
                                  at android.os.Looper.loop(Looper.java:154)
                                  at android.app.ActivityThread.main(ActivityThread.java:6119)
                                  at java.lang.reflect.Method.invoke(Native Method)
                                  at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:886)
                                  at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:776)
emit 2
emit 3
emit complete
```
抛出异常
为什么上游发送第一个事件后下游就抛出了MissingBackpressureException异常, 这是因为下游没有调用request, 上游就认为下游没有处理事件的能力, 而这又是一个同步的订阅, 既然下游处理不了, 那上游不可能一直等待吧, 如果是这样, 万一这两根水管工作在主线程里, 界面不就卡死了吗, 因此只能抛个异常来提醒我们. 那如何解决这种情况呢, 很简单啦, 下游直接调用request(Long.MAX_VALUE)就行了, 或者根据上游发送事件的数量来request就行了, 比如这里request(3)就可以了.

我们看一下在异步情况下，不设置`void request(long n)`
```
public static void demo3() {
        Flowable.create(new FlowableOnSubscribe<Integer>() {
            @Override
            public void subscribe(FlowableEmitter<Integer> emitter) throws Exception {
                Log.i(TAG, "emit 1");
                emitter.onNext(1);
                Log.i(TAG, "emit 2");
                emitter.onNext(2);
                Log.i(TAG, "emit 3");
                emitter.onNext(3);
                Log.i(TAG, "emit complete");
                emitter.onComplete();
            }
        }, BackpressureStrategy.ERROR).subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Subscriber<Integer>() {

                    @Override
                    public void onSubscribe(Subscription s) {
                        Log.i(TAG, "onSubscribe");
                        mSubscription = s;
                    }

                    @Override
                    public void onNext(Integer integer) {
                        Log.i(TAG, "onNext: " + integer);
                    }

                    @Override
                    public void onError(Throwable t) {
                        Log.i(TAG, "onError: ", t);
                    }

                    @Override
                    public void onComplete() {
                        Log.i(TAG, "onComplete");
                    }
                });
    }
```

运输结果
```
onSubscribe
emit 1
emit 2
emit 3
emit complete
```
被观察者正常发送事件，但是观察者接收不到事件
因为在Flowable里默认有一个大小为128的缓冲区, 当被观察者和观察者工作在不同的线程中时, 被观察者就会先把事件发送到这个缓冲区中, 因此, 观察者虽然没有调用request, 但是被观察者在缓冲区中保存着这些事件, 只有当观察者调用request时, 才从缓冲区里取出事件发给观察者.

设置`void request(long n)`的作用是观察者能处理多少个事件就告诉被观察者，这样被观察者根据观察者的处理能力来决定发送多少个事件。

# RxJava Flowable操作符详解
在上面我们说被观察者如果一次性发送128个事件是没有异常的，但是一旦超过128个的话就会抛出`MissingBackpressureException`,这是在提示被观察者发送了太多的事件，观察者还没有处理过来。
解决方法：
发送128个事件不出现异常，是因为在`Flowable`的内部有一个大小为128的缓冲区，超过128的时候就会溢出。，如果我们换个缓冲区大的话，会不会解决之前的异常
```
public static void demo1() {
        Flowable.create(new FlowableOnSubscribe<Integer>() {
            @Override
            public void subscribe(FlowableEmitter<Integer> emitter) throws Exception {
                for (int i = 0; i < 1000; i++) {
                    Log.i(TAG, "emit " + i);
                    emitter.onNext(i);
                }
            }
        }, BackpressureStrategy.BUFFER).subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Subscriber<Integer>() {

                    @Override
                    public void onSubscribe(Subscription s) {
                        Log.i(TAG, "onSubscribe");
                        mSubscription = s;
                    }

                    @Override
                    public void onNext(Integer integer) {
                        Log.i(TAG, "onNext: " + integer);
                    }

                    @Override
                    public void onError(Throwable t) {
                        Log.i(TAG, "onError: ", t);
                    }

                    @Override
                    public void onComplete() {
                        Log.i(TAG, "onComplete");
                    }
                });
    }
```
运行结果
```
onSubscribe
emit 0
emit 1
emit 2
...
emit 997
emit 998
emit 999
```
这不是和`Observable`一样吗。但是要注意如果被观察者一直发送事件，而观察者没有去处理事件的话，一样会造成OOM
```
public static void demo2() {
        Flowable.create(new FlowableOnSubscribe<Integer>() {
            @Override
            public void subscribe(FlowableEmitter<Integer> emitter) throws Exception {
                for (int i = 0; ; i++) {
                    emitter.onNext(i);
                }
            }
        }, BackpressureStrategy.BUFFER).subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Subscriber<Integer>() {

                    @Override
                    public void onSubscribe(Subscription s) {
                        Log.i(TAG, "onSubscribe");
                        mSubscription = s;
                    }

                    @Override
                    public void onNext(Integer integer) {
                        Log.i(TAG, "onNext: " + integer);
                    }

                    @Override
                    public void onError(Throwable t) {
                        Log.i(TAG, "onError: ", t);
                    }

                    @Override
                    public void onComplete() {
                        Log.i(TAG, "onComplete");
                    }
                });
    }
```
运行结果
<center>![](http://p2g00vblr.bkt.clouddn.com/RxJavaFlowable.gif)</center>

`Flowable`在这个时候的性能还不如`Observable`

`Flowable`从数量上解决被观察者发送过快的解决方法
1. `BackpressureStrategy.DROP`:把存不下的事件丢弃
```
public static void demo3() {
        Flowable.create(new FlowableOnSubscribe<Integer>() {
            @Override
            public void subscribe(FlowableEmitter<Integer> emitter) throws Exception {
                for (int i = 0; ; i++) {
                    emitter.onNext(i);
                }
            }
        }, BackpressureStrategy.DROP).subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Subscriber<Integer>() {

                    @Override
                    public void onSubscribe(Subscription s) {
                        Log.i(TAG, "onSubscribe");
                        mSubscription = s;
                    }

                    @Override
                    public void onNext(Integer integer) {
                        Log.i(TAG, "onNext: " + integer);
                    }

                    @Override
                    public void onError(Throwable t) {
                        Log.i(TAG, "onError: ", t);
                    }

                    @Override
                    public void onComplete() {
                        Log.i(TAG, "onComplete");
                    }
                });
    }
```
运行结果
ChapterEight.demo3();
```
onSubscribe
onNext: 0
onNext: 1
onNext: 2
onNext: 3
onNext: 4
onNext: 5
...
onNext: 125
onNext: 126
onNext: 127
```
ChapterEight.request(1000);
```
onSubscribe
onNext: 0
onNext: 1
onNext: 2
onNext: 3
onNext: 4
onNext: 5
...
onNext: 708
onNext: 709
onNext: 710
onNext: 711
onNext: 712
onNext: 713
onNext: 714
onNext: 715
onNext: 716
onNext: 717
onNext: 718
onNext: 719
onNext: 720
```
第一次request的时候, 下游的确收到的是0-127这128个事件, 但第二次request的时候就不确定了, 因为上游一直在发送事件. 内存占用也很正常
2. `BackpressureStrategy.LATEST`:只保留最新的事件
```
public static void demo4() {
        Flowable.create(new FlowableOnSubscribe<Integer>() {
            @Override
            public void subscribe(FlowableEmitter<Integer> emitter) throws Exception {
                for (int i = 0; ; i++) {
                    emitter.onNext(i);
                }
            }
        }, BackpressureStrategy.LATEST).subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Subscriber<Integer>() {

                    @Override
                    public void onSubscribe(Subscription s) {
                        Log.i(TAG, "onSubscribe");
                        mSubscription = s;
                    }

                    @Override
                    public void onNext(Integer integer) {
                        Log.i(TAG, "onNext: " + integer);
                    }

                    @Override
                    public void onError(Throwable t) {
                        Log.i(TAG, "onError: ", t);
                    }

                    @Override
                    public void onComplete() {
                        Log.i(TAG, "onComplete");
                    }
                });
    }
```
运行结果
ChapterEight.demo4();
ChapterEight.demo3();
```
onSubscribe
onNext: 0
onNext: 1
onNext: 2
onNext: 3
onNext: 4
onNext: 5
...
onNext: 125
onNext: 126
onNext: 127
```
ChapterEight.request(128);
```
onNext: 9999
```
<center>![](http://p2g00vblr.bkt.clouddn.com/RxJavaFlowable2.gif)</center>

drop和latest两者的改良版
```
Flowable.create(new FlowableOnSubscribe<Integer>() {
            @Override
            public void subscribe(FlowableEmitter<Integer> emitter) throws Exception {
                for (int i = 0; i < 10000; i++) {  //只发1w个事件
                    emitter.onNext(i);
                }
            }
        }, BackpressureStrategy.DROP).subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Subscriber<Integer>() {

                    @Override
                    public void onSubscribe(Subscription s) {
                        Log.d(TAG, "onSubscribe");
                        mSubscription = s;
                        s.request(128);  //一开始就处理掉128个事件
                    }

                    @Override
                    public void onNext(Integer integer) {
                        Log.d(TAG, "onNext: " + integer);
                    }

                    @Override
                    public void onError(Throwable t) {
                        Log.w(TAG, "onError: ", t);
                    }

                    @Override
                    public void onComplete() {
                        Log.d(TAG, "onComplete");
                    }
                });
```
运行结果
<center>![](http://p2g00vblr.bkt.clouddn.com/RxJavaFlowable3.gif)</center>
 一开始下游就处理掉了128个事件, 当我们再次request的时候, 只得到了第3317的事件, 后面的事件直接被抛弃了.

```
public static void demo4() {
        Flowable.create(new FlowableOnSubscribe<Integer>() {
            @Override
            public void subscribe(FlowableEmitter<Integer> emitter) throws Exception {
                for (int i = 0; i < 10000; i++) {
                    emitter.onNext(i);
                }
            }
        }, BackpressureStrategy.LATEST).subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Subscriber<Integer>() {

                    @Override
                    public void onSubscribe(Subscription s) {
                        Log.i(TAG, "onSubscribe");
                        mSubscription = s;
                        s.request(128);
                    }

                    @Override
                    public void onNext(Integer integer) {
                        Log.i(TAG, "onNext: " + integer);
                    }

                    @Override
                    public void onError(Throwable t) {
                        Log.i(TAG, "onError: ", t);
                    }

                    @Override
                    public void onComplete() {
                        Log.i(TAG, "onComplete");
                    }
                });
    }
```
运行结果
<center>![](http://p2g00vblr.bkt.clouddn.com/RxJavaFlowable4.gif)</center>
除去前面128个事件, 与Drop不同, Latest总是能获取到最后最新的事件, 例如这里我们总是能获得最后一个事件9999.

budong 这些FLowable是我自己创建的, 所以我可以选择策略, 那面对有些FLowable并不是我自己创建的, 该怎么办呢? 比如RxJava中的interval操作符, 这个操作符并不是我们自己创建的, 来看下面这个例子吧:
```
Flowable.interval(1, TimeUnit.MICROSECONDS)
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Subscriber<Long>() {
                    @Override
                    public void onSubscribe(Subscription s) {
                        Log.d(TAG, "onSubscribe");
                        mSubscription = s;
                        s.request(Long.MAX_VALUE);
                    }

                    @Override
                    public void onNext(Long aLong) {
                        Log.d(TAG, "onNext: " + aLong);
                        try {
                            Thread.sleep(1000);  //延时1秒
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }

                    @Override
                    public void onError(Throwable t) {
                        Log.w(TAG, "onError: ", t);
                    }

                    @Override
                    public void onComplete() {
                        Log.d(TAG, "onComplete");
                    }
                });
```
运行结果
```
onSubscribe
onError: 
                              io.reactivex.exceptions.MissingBackpressureException: Can't deliver value 128 due to lack of requests
                                  at io.reactivex.internal.operators.flowable.FlowableInterval$IntervalSubscriber.run(FlowableInterval.java:87)
                                  at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:428)
                                  at java.util.concurrent.FutureTask.runAndReset(FutureTask.java:278)
                                  at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:273)
                                  at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1133)
                                  at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:607)
                                  at java.lang.Thread.run(Thread.java:761)
```

RxJava给我们提供了其他的方法:
- onBackpressureBuffer()
- onBackpressureDrop()
- onBackpressureLatest()

举个栗子
```
Flowable.interval(1, TimeUnit.MICROSECONDS)
                .onBackpressureDrop()  //加上背压策略
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Subscriber<Long>() {
                    @Override
                    public void onSubscribe(Subscription s) {
                        Log.d(TAG, "onSubscribe");
                        mSubscription = s;
                        s.request(Long.MAX_VALUE);
                    }

                    @Override
                    public void onNext(Long aLong) {
                        Log.d(TAG, "onNext: " + aLong);
                        try {
                            Thread.sleep(1000);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }

                    @Override
                    public void onError(Throwable t) {
                        Log.w(TAG, "onError: ", t);
                    }

                    @Override
                    public void onComplete() {
                        Log.d(TAG, "onComplete");
                    }
                });
```

# RxJava Flowable响应式拉取
大家都知道`Flowable`是采用了`响应式拉取`的方式。那下面我们来讲解一下`Flowable`是如何响应式拉取的。
```
Flowable.create(new FlowableOnSubscribe<Integer>() {
            @Override
            public void subscribe(FlowableEmitter<Integer> emitter) throws Exception {
                Log.d(TAG, "emit 1");
                emitter.onNext(1);
                Log.d(TAG, "emit 2");
                emitter.onNext(2);
                Log.d(TAG, "emit 3");
                emitter.onNext(3);
                Log.d(TAG, "emit complete");
                emitter.onComplete();
            }
        }, BackpressureStrategy.ERROR).subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Subscriber<Integer>() {

                    @Override
                    public void onSubscribe(Subscription s) {
                        Log.d(TAG, "onSubscribe");
                        mSubscription = s;  
                        s.request(1);
                    }

                    @Override
                    public void onNext(Integer integer) {
                        Log.d(TAG, "onNext: " + integer);
                        mSubscription.request(1);
                    }

                    @Override
                    public void onError(Throwable t) {
                        Log.w(TAG, "onError: ", t);
                    }

                    @Override
                    public void onComplete() {
                        Log.d(TAG, "onComplete");
                    }
                });
```
我们知道在异步的情况下，当观察者每次处理掉一个事件之后才去调用`request(1)`去请求下一个事件，被观察者并不是当观察者请求一个才发送一个事件，而是一开始就发送所有的事件。

如果观察者调用`Subscription.request(n)`可以告诉被观察者自己可以处理多少事件，那么被观察者就可以根据观察者的处理能力发送事件，下面我们来看看被观察者是如何知道观察者的处理能力的？

先看一下`FlowableEmitter`的源码
```
public interface FlowableEmitter<T> extends Emitter<T> {

    void setDisposable(@Nullable Disposable s);

    void setCancellable(@Nullable Cancellable c);

    /**
     * The current outstanding request amount.
     * 当前外部请求的数量
     * <p>This method is thread-safe.
     * @return the current outstanding request amount
     */
    long requested();//重点

    boolean isCancelled();

    FlowableEmitter<T> serialize();

    boolean tryOnError(@NonNull Throwable t);
}
```
我们先看一下同步的情况
```
public static void demo1() {
        Flowable.create(new FlowableOnSubscribe<Integer>() {
                    @Override
                    public void subscribe(FlowableEmitter<Integer> emitter) throws Exception {
                        Log.i(TAG, "current requested: " + emitter.requested());
                    }
                }, BackpressureStrategy.ERROR)
                .subscribe(new Subscriber<Integer>() {

                    @Override
                    public void onSubscribe(Subscription s) {
                        Log.i(TAG, "onSubscribe");
                        mSubscription = s;
                        s.request(10);
                        s.request(100);
                    }

                    @Override
                    public void onNext(Integer integer) {
                        Log.i(TAG, "onNext: " + integer);
                    }

                    @Override
                    public void onError(Throwable t) {
                        Log.i(TAG, "onError: ", t);
                    }

                    @Override
                    public void onComplete() {
                        Log.i(TAG, "onComplete");
                    }
                });
    }
```
没有`s.request();`运行结果是
```
onSubscribe
current requested: 0
```
`s.request(10);`运行结果是
```
onSubscribe
current requested: 10
```
`s.request(10);s.request(100);`运行结果是
```
onSubscribe
current requested: 110
```

当被观察者发送事件的时候，`current requested`就会减少
举个栗子
```
public static void demo2() {
        Flowable
                .create(new FlowableOnSubscribe<Integer>() {
                    @Override
                    public void subscribe(final FlowableEmitter<Integer> emitter) throws Exception {
                        Log.i(TAG, "before emit, requested = " + emitter.requested());

                        Log.i(TAG, "emit 1");
                        emitter.onNext(1);
                        Log.i(TAG, "after emit 1, requested = " + emitter.requested());

                        Log.i(TAG, "emit 2");
                        emitter.onNext(2);
                        Log.i(TAG, "after emit 2, requested = " + emitter.requested());

                        Log.i(TAG, "emit 3");
                        emitter.onNext(3);
                        Log.i(TAG, "after emit 3, requested = " + emitter.requested());

                        Log.i(TAG, "emit complete");
                        emitter.onComplete();

                        Log.i(TAG, "after emit complete, requested = " + emitter.requested());
                    }
                }, BackpressureStrategy.ERROR)
                .subscribe(new Subscriber<Integer>() {

                    @Override
                    public void onSubscribe(Subscription s) {
                        Log.i(TAG, "onSubscribe");
                        mSubscription = s;
                        s.request(10);
                    }

                    @Override
                    public void onNext(Integer integer) {
                        Log.i(TAG, "onNext: " + integer);
                    }

                    @Override
                    public void onError(Throwable t) {
                        Log.i(TAG, "onError: ", t);
                    }

                    @Override
                    public void onComplete() {
                        Log.i(TAG, "onComplete");
                    }
                });
    }
```
运行结果
```
onSubscribe
before emit, requested = 10
emit 1
onNext: 1
after emit 1, requested = 9
emit 2
onNext: 2
after emit 2, requested = 8
emit 3
onNext: 3
after emit 3, requested = 7
emit complete
onComplete
after emit complete, requested = 7
```
观察者调用request(n) 告诉被观察者它的处理能力，被观察者每发送一个next事件之后，requested就减一，**注意是next事件，complete和error事件不会消耗requested，**当减到0时，则代表观察者没有处理能力了，这个时候你如果继续发送事件，会发生什么后果呢？当然是MissingBackpressureException啦
```
onSubscribe
    before emit, requested = 2
    emit 1
    onNext: 1
    after emit 1, requested = 1
    emit 2
    onNext: 2
    after emit 2, requested = 0
    emit 3
    onError: 
    io.reactivex.exceptions.MissingBackpressureException: create: could not emit value due to lack of requests
        at io.reactivex.internal.operators.flowable.FlowableCreate$ErrorAsyncEmitter.onOverflow(FlowableCreate.java:432)
        at io.reactivex.internal.operators.flowable.FlowableCreate$NoOverflowBaseAsyncEmitter.onNext(FlowableCreate.java:398)
        at zlc.season.rxjava2demo.demo.ChapterNine$4.subscribe(ChapterNine.java:82)
        at io.reactivex.internal.operators.flowable.FlowableCreate.subscribeActual(FlowableCreate.java:72)
        at io.reactivex.Flowable.subscribe(Flowable.java:14319)
        at io.reactivex.Flowable.subscribe(Flowable.java:14268)
        at zlc.season.rxjava2demo.demo.ChapterNine.demo2(ChapterNine.java:91)
        at zlc.season.rxjava2demo.MainActivity.onCreate(MainActivity.java:53)
        at android.app.Activity.performCreate(Activity.java:6078)
        at android.app.Instrumentation.callActivityOnCreate(Instrumentation.java:1109)
        at android.app.ActivityThread.performLaunchActivity(ActivityThread.java:2381)
        at android.app.ActivityThread.handleLaunchActivity(ActivityThread.java:2490)
        at android.app.ActivityThread.access$1200(ActivityThread.java:159)
        at android.app.ActivityThread$H.handleMessage(ActivityThread.java:1363)
        at android.os.Handler.dispatchMessage(Handler.java:102)
        at android.os.Looper.loop(Looper.java:135)
        at android.app.ActivityThread.main(ActivityThread.java:5569)
        at java.lang.reflect.Method.invoke(Native Method)
        at java.lang.reflect.Method.invoke(Method.java:372)
        at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:931)
        at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:726)
    after emit 3, requested = 0
    emit complete
    after emit complete, requested = 0
```

同步的情况已经完成了
<center>![](http://p2g00vblr.bkt.clouddn.com/RxjavaFlowable%E5%90%8C%E6%AD%A5%E8%AF%B7%E6%B1%82.png)</center>

让我们来看看异步下的情况
```
public static void demo3() {
        Flowable
                .create(new FlowableOnSubscribe<Integer>() {
                    @Override
                    public void subscribe(FlowableEmitter<Integer> emitter) throws Exception {
                        Log.i(TAG, "current requested: " + emitter.requested());
                    }
                }, BackpressureStrategy.ERROR)
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Subscriber<Integer>() {

                    @Override
                    public void onSubscribe(Subscription s) {
                        Log.i(TAG, "onSubscribe");
                        mSubscription = s;
                        s.request(1000);
                    }

                    @Override
                    public void onNext(Integer integer) {
                        Log.i(TAG, "onNext: " + integer);
                    }

                    @Override
                    public void onError(Throwable t) {
                        Log.i(TAG, "onError: ", t);
                    }

                    @Override
                    public void onComplete() {
                        Log.i(TAG, "onComplete");
                    }
                });
    }
```
被观察者和观察者什么都没有做的情况下，运行结果
```
onSubscribe
current requested: 128
```
当我们`s.request(1000);`,运行结果
```
onSubscribe
current requested: 128
```
还是和上面一样...
异步情况下的请求情况
<center>![](http://p2g00vblr.bkt.clouddn.com/RxjavaFlowable%E5%BC%82%E6%AD%A5%E8%AF%B7%E6%B1%82.png)</center>
在异步的情况下，每一个线程里都有一个`requested`，当我们观察者`request(1000)`的时候，实际上是改变观察者所在线程中的`requested`，而被观察者中的`requested`的值所有`RxJava`内部调用`request(n)`来设置的，这个调用会在合适的时候触发。
我们来看一下是在什么合适的时候触发的？
```
public static void demo4() {
        Flowable
                .create(new FlowableOnSubscribe<Integer>() {
                    @Override
                    public void subscribe(FlowableEmitter<Integer> emitter) throws Exception {
                        Log.i(TAG, "First requested = " + emitter.requested());
                        boolean flag;
                        for (int i = 0; ; i++) {
                            flag = false;
                            while (emitter.requested() == 0) {
                                if (!flag) {
                                    Log.i(TAG, "Oh no! I can't emit value!");
                                    flag = true;
                                }
                            }
                            emitter.onNext(i);
                            Log.i(TAG, "emit " + i + " , requested = " + emitter.requested());
                        }
                    }
                }, BackpressureStrategy.ERROR)
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Subscriber<Integer>() {

                    @Override
                    public void onSubscribe(Subscription s) {
                        Log.i(TAG, "onSubscribe");
                        mSubscription = s;
                    }

                    @Override
                    public void onNext(Integer integer) {
                        Log.i(TAG, "onNext: " + integer);
                    }

                    @Override
                    public void onError(Throwable t) {
                        Log.i(TAG, "onError: ", t);
                    }

                    @Override
                    public void onComplete() {
                        Log.i(TAG, "onComplete");
                    }
                });
    }
```
首先我们观察者不去`request`，运行结果
```
    onSubscribe
    First requested = 128
    emit 0 , requested = 127
    emit 1 , requested = 126
    emit 2 , requested = 125
    ...
    emit 120 , requested = 7
    emit 121 , requested = 6
    emit 122 , requested = 5
    emit 123 , requested = 4
    emit 124 , requested = 3
    emit 125 , requested = 2
    emit 126 , requested = 1
    emit 127 , requested = 0
    Oh no! I can't emit value!
```
现在我们`request(96)`，运行结果
```
D/TAG: onNext: 0
D/TAG: onNext: 1
  ...
D/TAG: onNext: 92
D/TAG: onNext: 93
D/TAG: onNext: 94
D/TAG: onNext: 95
D/TAG: emit 128 , requested = 95
D/TAG: emit 129 , requested = 94
D/TAG: emit 130 , requested = 93
D/TAG: emit 131 , requested = 92
  ...
D/TAG: emit 219 , requested = 4
D/TAG: emit 220 , requested = 3
D/TAG: emit 221 , requested = 2
D/TAG: emit 222 , requested = 1
D/TAG: emit 223 , requested = 0
D/TAG: Oh no! I can't emit value!
```
当观察者消费第96个事件之后，被观察者又开始发送事件了，而且可以看到当前被观察者的requested的值是96(打印出来的95是已经发送了一个事件减一之后的值)，最终发出了第223个事件之后又进入了等待区，而223-127 正好等于 96。