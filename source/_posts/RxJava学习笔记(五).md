---
title: RxJava学习笔记5
date: 2015-12-15 13:42:20
tags: [Android, RxJava]
categories: 施博文
---
# RxJava笔记5
## 场景举例
### 场景1
#### 需求：
1.有一字符串，每个字符都是一个数字

2.将字符串转化为数字

3.过滤掉小于1的元素

4.去重

5.取前三个元素

6.求和
#### 代码
```Java
Subscriber<Integer> subscriber = new Subscriber<Integer>() {
            @Override
            public void onCompleted() {
                Log.i("TAG","onCompleted");
            }

            @Override
            public void onError(Throwable e) {

            }

            @Override
            public void onNext(Integer integer) {
                Log.i("TAG",""+integer);//最后输出为9
            }
        };

        Observable.just("1","2","2","3","4","5")
                .map(new Func1<String, Integer>() {
                    @Override
                    public Integer call(String s) {
                        return Integer.parseInt(s);
                    }
                })//map操作符将String转化为integer
                .filter(new Func1<Integer, Boolean>() {
                    @Override
                    public Boolean call(Integer integer) {
                        return integer>1;
                    }
                })//filter操作符过滤<=1的数字
                .distinct()//去重
                .take(3)//前三个
                .reduce(new Func2<Integer, Integer, Integer>() {
                    @Override
                    public Integer call(Integer integer, Integer integer2) {
                        return integer+integer2;
                    }
                })//reduce操作符 求和
                .subscribe(subscriber);
```
因为我Android Studio打死也没装上lamda表达式插件，导致不能用Java8新特性，否则代码为更加整齐简介，这个是大头鬼给出的Java8写法，纯手打，没有经过测验
```Java
Observable.just("1","2","2","3","4","5")
        .map(Integer::parseInt)//map操作符将String转化为integer
        .filter(s -> s>1)//filter操作符过滤<=1的数字
        .distinct()//去重
        .take(3)//前三个
        .reduce((integer,integer2) ->integer.intValue()+integer2.intValue())//reduce操作符 求和
        .subscribe(subscriber);
```
再想想如果不用RxJava，各种嵌套if else，说实话这是我第一次体会到RxJava的好处。另外Java8也是神器，原本Java繁琐的语法一下变得简单，下一步就是要去学习Java8新特性了。

## 小结
到这总算有了一个还不错的使用场景（还不错是指接近实际生产环境，且能够体现出RxJava优势的使用场景），应该来说对RxJava能有个比较清晰的认识了。另外根据大头鬼的总结，RxJava的使用场景为：
1.出现多层嵌套回调

2.复杂的数据处理

3.响应式UI

4.复杂的线程切换

## 视频地址
[大头鬼的RxJava视频](http://boolan.com/lecture/1000001243#0-tsina-1-68759-397232819ff9a47a7b7e80a40613cfe1)
