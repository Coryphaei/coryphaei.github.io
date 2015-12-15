---
title: RxJava学习笔记1
date: 2015-12-15 12:27:20
tags: [Android][RxJava]
categories: 施博文
---
# RxJava笔记1

## 写在前面
很早以前就想去学一学RxJava这个既神秘又高大上的库，看过大头鬼的文章，限于自身水平原因一直看的迷迷糊糊晕头转向。刚找到扔物线写的文章，感觉突然茅塞顿开。 [扔物线文章地址](http://gank.io/post/560e15be2dca930e00da1083#toc_5)

## RxJava是什么
一个实现异步操作的库（这句话只是先写在这，因为这句话具体含义还没理解）

## RxJava优点
实现链式调用，逻辑清晰简单

## 准备工作
[RxJava](https://github.com/ReactiveX/RxJava)

[RxAndroid](https://github.com/ReactiveX/RxAndroid)

```gradle
compile 'io.reactivex:rxandroid:1.0.1'
compile 'io.reactivex:rxjava:1.0.16'
```


## 基本实现
### 创建观察者Observer
```Java
Observer<String> observer = new Observer<String>() {
            @Override
            public void onCompleted() {
                Log.i("TAG","onCompleted");
            }

            @Override
            public void onError(Throwable e) {

            }

            @Override
            public void onNext(String s) {
                Log.i("TAG",s);
            }
        };
```
另外，Observer还有一个抽象类，Subscriber，两者使用方法基本一样，事实上，在subscibe的过程中，Observer也会被转化为一个Subscriber。
```Java
Subscriber<String> subscriber = new Subscriber<String>() {
    @Override
    public void onCompleted() {
        Log.i("TAG","onCompleted");
    }

    @Override
    public void onError(Throwable e) {

    }

    @Override
    public void onNext(String s) {
        Log.i("TAG",s);
    }
};
```
### 创建被观察者 Observable
```Java
Observable observable = Observable.create(new Observable.OnSubscribe<String>() {
           @Override
           public void call(Subscriber<? super String> subscriber) {
               subscriber.onNext("Hello");
               subscriber.onNext("World");
               subscriber.onNext("Android");
               subscriber.onCompleted();
           }
       });
```

### 创建被观察者的其他方法
除了以上的create()方法，其他还有

#### just
```Java
just(T...)
```
```Java
Observable observable = Observable.just("Hello","World","Android");
```
等价于
```Java
onNext("Hello");
onNext("World");
onNex("Android");
```

#### from
```Java
from(T[]);
```
```Java
String[] words = {"Hello","World","Android"};
Observable observable = Observable.from(words);
```
等价于
```Java
onNext("Hello");
onNext("World");
onNex("Android");
```

### 订阅
创建完了观察者subscriber或observer和被观察者observal以后，需要将二者绑定
```Java
observal.subscibe(subsciber);
```
#### 订阅的其他方法
subscribe()接受不完整定义的回调
```Java
Action1<String> onNextAction = new Action1<String>() {
    // onNext()
    @Override
    public void call(String s) {
        Log.d(tag, s);
    }
};
Action1<Throwable> onErrorAction = new Action1<Throwable>() {
    // onError()
    @Override
    public void call(Throwable throwable) {
        // Error handling
    }
};
Action0 onCompletedAction = new Action0() {
    // onCompleted()
    @Override
    public void call() {
        Log.d(tag, "completed");
    }
};

// 自动创建 Subscriber ，并使用 onNextAction 来定义 onNext()
observable.subscribe(onNextAction);
// 自动创建 Subscriber ，并使用 onNextAction 和 onErrorAction 来定义 onNext() 和 onError()
observable.subscribe(onNextAction, onErrorAction);
// 自动创建 Subscriber ，并使用 onNextAction、 onErrorAction 和 onCompletedAction 来定义 onNext()、 onError() 和 onCompleted()
observable.subscribe(onNextAction, onErrorAction, onCompletedAction);
```

## 小结
以上为RxJava的基础用法，Rx不在具体的使用场景下很难理解，下一节研究具体场景下的使用方法。
