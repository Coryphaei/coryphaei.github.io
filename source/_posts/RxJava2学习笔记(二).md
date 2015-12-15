---
title: RxJava学习笔记2
date: 2015-12-15 12:32:20
tags: [Android][RxJava]
categories: 施博文
---
# RxJava笔记2

## 实际使用场景
### 打印字符串数组
```Java
String[] names = {"Hello","World","Android"};

        Observable.from(names).subscribe(new Action1<String>() {
            @Override
            public void call(String s) {
                Log.i("TAG",s);
            }
        });
```
输出结果为
```Java
12-09 03:43:47.789 19193-19193/com.bzt.rxjava I/TAG: Hello
12-09 03:43:47.789 19193-19193/com.bzt.rxjava I/TAG: World
12-09 03:43:47.789 19193-19193/com.bzt.rxjava I/TAG: Android
```

### 传递图片Id，显示图片
```Java
final ImageView imageView = (ImageView) findViewById(R.id.imageview);
final int ResourceId = R.mipmap.ic_launcher;

Observable.create(new Observable.OnSubscribe<Drawable>() {
    @Override
    public void call(Subscriber<? super Drawable> subscriber) {
        Drawable drawable = getTheme().getDrawable(ResourceId);
        subscriber.onNext(drawable);
        subscriber.onCompleted();
    }
}).subscribe(new Observer<Drawable>() {
    @Override
    public void onCompleted() {

    }

    @Override
    public void onError(Throwable e) {

    }

    @Override
    public void onNext(Drawable drawable) {
        imageView.setImageDrawable(drawable);
    }
});
```
本质都是创建出观察者Subscriber和被观察者Observable，然后将两个对象联接起来。

然而，如扔物线说的，这两个例子除了用来看看以外，并没有什么用，RxJava中，事件的发出和消费都是在同一个线程中的，而观察者模式本身的目的就是“后台处理，前台回调”的异步机制，要实现异步，则需要用的RxJava的另一个概念：Scheduler。

## 线程控制 Scheduler
线程不变原则：在哪个线程调用了subscribe(),就在哪个线程生产事件，就在哪个线程消费事件。如果需要切换线程就需要Scheduler。

### 基本API
```Java
Schedulers.immediate();//在当前线程运行，相当于不指定线程
Schedulers.io();//I/O操作（读写文件，读写数据库，网络信息交互使用Scheduler），内部是一个有无数量限制的线程，比newThread更有效率
Schedulers.newThread();//启用新线程
Schedulers.computation();//用于cpu计算操作
AndroidSchedulers.mainThread();//Android特有，在Android主线程
```
subscribeOn() 指定subscribe()发生的线程，事件产生线程
observeOn()指定Subscriber所运行的线程，事件消费线程

示例
```Java
Observable.just(1,2,3,4)
  .subscribeOn(Schedulers.io())//指定subscribe()发生在IO线程
  .observeOn(AndroidSchedulers.mainThread())//指定Subscriber的回调发生在主线程
  .subscribeOn(new Action1<Integer>){
    @Override
        public void call(Integer number) {
            Log.d(tag, "number:" + number);
        }
  )};
```
以上代码逻辑为：1，2，3，4，在IO线程发出，而数字的打印发生在主线程，满足“后台线程取数据，主线程显示”的策略

## 小结
这篇笔记比较短，主要介绍了RxJava具体的使用场景和线程问题，下一篇是重点，介绍变换。
