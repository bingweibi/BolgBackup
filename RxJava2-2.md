---
title: RxJava2(2)
date: 2018-06-25 17:09:55
tags: RxJava2
---

# 变换操作符

<center>![](https://rxjava.yuxingxin.com/images/chapter5_1.png)</center>

## map

### 作用

将观察者发送的数据类型转变为其他的数据类型

### 使用

```
        Observable.just(1,2,3)
                .map(new Function<Integer, String>() {
                    @Override
                    public String apply(Integer integer){
                        return "string: " + integer;
                    }
                }).subscribe(new Consumer<String>() {
            @Override
            public void accept(String s){
                Log.i(TAG, "accept: " + "string: " + s);
            }
        });
```

### 输出结果

```
accept: string: 1
accept: string: 2
accept: string: 3
```

<!--more-->

## flatMap

<center>![](https://rxjava.yuxingxin.com/images/chapter5_2.png)</center>

### 作用

将一个发送事件的被观察者转变为多个发送事件的被观察者，然后将他们发送的事件合并后放在一个单独的被观察者中，在flatMap中不保证事件的顺序。

### 使用

```
        Observable.just(1,2,3,4,5,6)
                .flatMap(new Function<Integer,ObservableSource<String>>() {
                    @Override
                    public ObservableSource<String> apply(Integer integer) {
                        List<String> l = new ArrayList<>();
                        for (int i=0;i<2;i++){
                            l.add("string: " + integer);
                        }
                        //顺序不一致
                        return     Observable.fromIterable(l).delay(10, TimeUnit.MILLISECONDS);
                    }
                }).subscribe(new Consumer<String>() {
            @Override
            public void accept(String s) {
                Log.i(TAG, "accept: " + s);
            }
        });
```

### 输出结果

```
    accept: string: 1
    accept: string: 1
    accept: string: 2
    accept: string: 2
    accept: string: 3
    accept: string: 3
    accept: string: 4
    accept: string: 6
    accept: string: 6
    accept: string: 4
    accept: string: 5
    accept: string: 5
```

## concatMap()

<center>![](https://rxjava.yuxingxin.com/images/chapter5_3.png)</center>

### 作用

与flatMap作用一样，但是保证了顺序不变

### 使用

```
        //concatMap
        Observable.just(1,2,3,4,5,6)
                .concatMap(new Function<Integer, ObservableSource<String>>() {
                    @Override
                    public ObservableSource<String> apply(Integer integer) {
                        List<String> l = new ArrayList<>();
                        for (int i=0;i<2;i++){
                            l.add("string: " + integer);
                        }
                        return Observable.fromIterable(l).delay(10, TimeUnit.MILLISECONDS);
                    }
                }).subscribe(new Consumer<String>() {
            @Override
            public void accept(String s) {
                Log.i(TAG, "accept: " + s);
            }
        });
```

### 输出结果

```
accept: string: 1
accept: string: 1
accept: string: 2
accept: string: 2
accept: string: 3
accept: string: 3
accept: string: 4
accept: string: 4
accept: string: 5
accept: string: 5
accept: string: 6
accept: string: 6
```

## buffer()

<center>![](https://rxjava.yuxingxin.com/images/chapter5_10.png)</center>

<center>![](https://rxjava.yuxingxin.com/images/chapter5_11.png)</center>

<center>![](https://rxjava.yuxingxin.com/images/chapter5_12.png)</center>

### 作用

从需要发送的事件中获取一定数量的事件，并将这些事件放置在缓冲区中一起发出

count: 缓冲区的大小
skip: 缓冲区满了之后，发送下一个时间跳过多少个元素
timespan： 每隔timespan时间段发送一次

### 使用

```
            Observable.just(1,2,3,4,5,6)
                .buffer(3)
                .subscribe(new Consumer<List<Integer>>() {
                    @Override
                    public void accept(List<Integer> integers) throws Exception {
                        Log.i(TAG, "accept: " + integers);
                    }
                });
                
                //输出
                accept: [1, 2, 3]
                accept: [4, 5, 6]
```

```
            Observable.just(1,2,3,4,5,6)
                .buffer(3,1)
                .subscribe(new Consumer<List<Integer>>() {
                    @Override
                    public void accept(List<Integer> integers) throws Exception {
                        Log.i(TAG, "accept: " + integers);
                    }
                });
                
                //输出
                accept: [1, 2, 3]
                accept: [2, 3, 4]
                accept: [3, 4, 5]
                accept: [4, 5, 6]
                accept: [5, 6]
                accept: [6]
```

```
            Observable.just(1,2,3,4,5,6)
                .buffer(1,TimeUnit.MILLISECONDS,2)
                .subscribe(new Consumer<List<Integer>>() {
                    @Override
                    public void accept(List<Integer> integers) throws Exception {
                        Log.i(TAG, "accept: " + integers);
                    }
                });
                
                //输出
                accept: [1, 2]
                accept: [3, 4]
                accept: [5, 6]
                accept: []
```

## groupBy()

<center>![](https://rxjava.yuxingxin.com/images/chapter5_8.png)</center>

### 作用

将发送的数据根据需要进行分组，每个分组再返回给被观察者

### 使用

```
            Observable.just(1,2,3,4,5,6)
                .groupBy(new Function<Integer, Integer>() {
                    @Override
                    public Integer apply(Integer integer) throws Exception {
                        return integer % 3;
                    }
                }).subscribe(new Consumer<GroupedObservable<Integer, Integer>>() {
            @Override
            public void accept(GroupedObservable<Integer, Integer> integerIntegerGroupedObservable) throws Exception {
                Log.i(TAG, "group" + integerIntegerGroupedObservable.getKey() + ": ");
            }
        });
```

### 输出结果

```
    group1
    group2
    group0
    //被分为3组
```

## window()

<center>![](https://rxjava.yuxingxin.com/images/chapter5_13.png)</center>

### 作用

被观察者发送一定数量的事件，按照count参数的大小，将发送的事件分组

### 使用

```
            Observable.just(1,2,3,4,5,6)
                .window(2)
                .subscribe(new Consumer<Observable<Integer>>() {
                    @Override
                    public void accept(final Observable<Integer> integerObservable) throws Exception {
                        integerObservable.subscribe(new Consumer<Integer>() {
                            @Override
                            public void accept(Integer integer) throws Exception {
                                Log.i(TAG, integerObservable + "accept: " + integer);
                            }
                        });
                    }
                });
```

### 输出结果

```
    io.reactivex.subjects.UnicastSubject@2c02ead8accept: 1
    io.reactivex.subjects.UnicastSubject@2c02ead8accept: 2
    io.reactivex.subjects.UnicastSubject@1bdbfe31accept: 3
    io.reactivex.subjects.UnicastSubject@1bdbfe31accept: 4
    io.reactivex.subjects.UnicastSubject@2f045916accept: 5
    io.reactivex.subjects.UnicastSubject@2f045916accept: 6
    //看得出来分为三组
```
