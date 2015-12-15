---
title: RxJava学习笔记3
date: 2015-12-15 13:40:20
tags: [Android, RxJava]
categories: 施博文
---
# RxJava笔记3

## 变换

### 概念
变换，就是将时间序列中的对象或这个序列进行加工处理，转换成不同的事件或者事件序列。
### 实例代码
#### map()
```Java
Observable.just("image/logo.png")
      .map(new Func1<String, Bitmap>() {
          @Override
          public Bitmap call(String filepate) {
              return getBitmapFromFile(filepate);
          }
      })
      .subscribe(new Action1<Bitmap>() {
          @Override
          public void call(Bitmap bitmap) {
              showBitmap(bitmap);
          }
      });
```
可以看到 Fun1和Action1非常接近，区别在于Fun1包装了一个有返回值的方法，Action1包装了一个无返回值的方法.
这样就可以把原本的参数类型转化为Bitmap。
#### flatMap()
需求1：从一个数据类型Students中，打印出学生名
```Java
Student[] students = {};

Subscriber<String> subscriber = new Subscriber<String>() {
    @Override
    public void onCompleted() {

    }

    @Override
    public void onError(Throwable e) {

    }

    @Override
    public void onNext(String s) {
        Log.i("TAG",s);
    }
};

Observable.from(students)
        .map(new Func1<Student, String>() {
            @Override
            public String call(Student student) {
                return student.getName();
            }
        })
        .subscribe(subscriber);
```

需求2：从一个数据类型students中，打印出学生选的课程名称，注意：每个学生课程数不为0
```Java
Subscriber<Student> subscriber = new Subscriber<Student>() {
            @Override
            public void onCompleted() {

            }

            @Override
            public void onError(Throwable e) {

            }

            @Override
            public void onNext(Student student) {
                List<String> courses = student.getCourses();
                for (int i = 0; i < courses.size(); i++) {
                    String course = courses.get(i);
                    Log.d("TAG", course);
                }

            }
        };

        Observable.from(students).subscribe(subscriber);
```

需求3：不使用for循环打印课程
分析：用map肯定是行的，因为map是一对一转换，现在是要一对多的转化，这时就需要flatmap()了
```Java
Subscriber<String> subscriber = new Subscriber<String>() {
            @Override
            public void onCompleted() {

            }

            @Override
            public void onError(Throwable e) {

            }

            @Override
            public void onNext(String s) {
                Log.i("TAG", s);
            }
        };

        Observable.from(students)
                .flatMap(new Func1<Student, Observable<String>>() {
                    @Override
                    public Observable<String> call(Student student) {
                        return Observable.from(student.getCourses());
                    }
                })
                .subscribe(subscriber);
```
注意到flatmap返回的是一个Observal对象，此时不发送这个observal对象，每个observal发送的对象都被汇入同一个Observal,并有这个Observal统一交给subscriber回调方法。

#### compose()对Obseval的整体变化
暂时还没看懂==

## 小结
至此，对RxJava应该能有个大概的了解了，至少能够了解一些RxJava的基本内容了。然而，一切不讲实际需求的代码教学都是耍流氓，没有一个好的应用场景很难理解RxJava的好处，我看到像大头鬼这些大神也苦于寻找到一个优秀的使用场景，RxJava之路依然艰难。
