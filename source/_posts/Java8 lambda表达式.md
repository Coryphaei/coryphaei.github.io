---
title: Java8 lambda表达式
date: 2015-10-23 13:46:00
tags: [Android,Java,Java8,Lambda]
categories: Basti
---
# Java8 lambda表达式

## 前言
RxJava的逻辑清晰的特点在Java8 lambda表达式下更加突出，学好RxJava必须熟悉lambda表达式。
## 本质
lambda表达式的本质是一个匿名方法
### 举例
问题：求两个int型数据的和，通常的写法是
```java
public int add(int x,int y){
  return x+y;
}
```

转成lambda以后
```java
(int x,int y)->x+y;
```
或者，省略参数类型，java编译器会根据上下文推断
```java
(x,y)->x+y;
```
或者,可以显示的指明返回值
```java
(x,y)->{return x+y;}
```
###小结
lambda表达式由三部分组成：参数列表，箭头，表达式/语句块、

无参数的情况
```java
()->{Log.i("TAG","Message");}
```
如果只有一个参数且可以被java推断出类型，那么，参数列表的括号也可以省略：
c->{return c.size;}

## 类型
lambda表达式的目标类型是“函数接口”，定义是：一个接口，如果只有一个显示声明的抽象方法，那么它就是一个函数接口，最简单的：
```java
public interface MyInterface{
  int getInt();
}
```
或者
```java
public interface MyInterface{
  void doSomething();
}
```
特殊情况
```java
public interface Comparator<T>{
  int compare(T o1,T o2);
  boolean equals(Object obj);
}
```
注意接口Comparator，虽然声明了两个方法，貌似与定义不符（只有一个声明的抽象方法），但它的确是一个函数接口，这是因为equals方法是Object的。事实上所有的接口都会声明Object的public方法（大多是隐式的）。

## 赋值
```java
MyInterface myInterface = ()->{Log.i("Tag","Message");}
```
然后再赋值给一个Object:
```java
Object obj = myInterface;
```

但是不能这样赋值
```java
Object obj = ()->{Log.i("Tag","Message");}
```
必须显示的转型成为一个函数接口才可以
```java
Object obj = (myInterface)()->{Log.i("Tag","Message");}
```
## 使用
### lambda表达式用在何处
举例：未使用Lambda
```java
Thread t = new Thread( new Runnable () {
  @Override
  public void run() {
      Log.i("Tag","Message");
  }
});
```
使用lambda
```java
Thread t = new Thread{()->{Log.i("Tag","Message");}};
```
注意第二个线程里的lambda表达式，并不需要将其转成Runnable,因为Java能够根据上下文自动推断，
一个Thread的构造函数接受一个Runnable参数，且传入的Lambda表达式正好符合其run()函数，所以java编译器可以推断为Runnable。
### 集合批处理操作
```java
for (Object o:list ) {
  Log.i("TAG",o);
}
```
可转换为
```java
list.forEach(o->{Log.i("TAG",o)};)
```

### 流
一个流通常以一个集合类实例为其数据源，然后在其上定义各种操作。流的API设计使用了管道模式，对流的一次操作为返回另一个流，从而不同的操作可以串起来。
```java
List<Shape> shapes = ...;
shape.stream()
  .filter(s->s.getColor() == BLUE)
  .forEach(s->s.setColor(RED));
```
首先调用stream()方法，以集合类对象shapes里面的元素为数据源，生成一个流，然后在这个流上调用filter方法，跳出蓝色的，返回一个流，最后调用forEach方法将这些蓝色的shape换成红色。
另一个例子：一个典型的数据处理方法
```java
public void distinctPrimary(String... numbers) {
       List<String> list = Arrays.asList(numbers);
       List<Integer> r = list.stream()
               .map(e->{new Integer(e)})
               .filter(e->Primes.isPrime(e))
               .distinct()
               .collect(Collectors.toList());
       System.out.println("distinctPrimary result is: " + r);
   }
```

1.传入一串String，转成list。用steram()方法生成流
2.map方法将每个元素由String转为Integer得到一个新的流。
3.filter方法过滤不是素数的数字，并形成一个新的流
4.distinct 去重
5.collect 将最终结果收集到List中。
对于效率问题，前面的map filter distinct都是懒方法，collect是急方法，在遇到急方法之前，懒方法不会执行。
当遇到急方法时，前面的懒方法才会开始执行，且是管道式执行。例如对于字符串 "3",首先将"3"转化为3，在判断他是否是素数，
如果是素数，保留3进入distinct方法，如果在队列中已经有3了，则把3丢弃，否则加入到队列中，实际上，"3"只进行了一次遍历操作，
而不是像代码中表达的，先把所有字符串转化为数字（第一次循环），再将数字过滤（第二次循环），去重（第三次循环），收集（第四次循环）。

### 收集器
上述代码中，collect就是一个收集器，即将所有经过懒方法后被保留下来的数据收集起来。
常见的收集器， groupingBy:
```java
//给出一个String类型的数组，找出其中各个素数，并统计其出现次数
List<String> l = Arrays.asList(String... numbers);
Map<Integer,Integer> r
        = l.stream()
            .map(e->new Integer(e))
            .filter(e->Prime.isPrime(e))
            .collect(Collectors.groupingBy(p->p,Collectors.summingInt(p->1)));
```
相当于将所有收集到的数存放在一个Map中，以元素为Key，出现的次数作为Value。

```java
 //给出一个String类型的数组，求其中所有不重复素数的和
List<String> l = Arrays.asList(numbers);
int sum = l.stream()
            .map(e->new Integer(e))
            .filter(e->Prime.isPrime(e))
            .distinct()
            .reduce(0,(x,y)->x+y);
```
reduce方法用来产生单一的一个最终结果
```java
// 统计年龄在25-35岁的男女人数
//List<Person> persons
Map<Integer,Integer> result = persons.parallelStream()
                                      .filter(p->p.getAge>=25&&p.getAge()<=35)
                                      .collect(Collectors.groupingBy(p->p.getSex(),Collectors.summingInt(p->1));
```
### lambda表达式的更多用法
```java
//嵌套的lambda表达式
Callable<Runnable> c1 = ()->()->{System.out.println("Nested lambda");}
```
```java
//用在条件表达式中
Callable c2 = true ? (()->42);(()->24);
```
```java
// 定义一个递归函数，注意须用this限定
protected UnaryOperator<Integer> factorial = i -> i == 0 ? 1 : i * this.factorial.apply( i - 1 );
```
### 方法引用
```java
Integer::parseInt//静态方法引用
System.out::print//实例方法引用
Person::new//构造器引用
```
举例：
```java
    //c1 与 c2 是一样的（静态方法引用）
    Comparator<Integer> c2 = (x, y) -> Integer.compare(x, y);
    Comparator<Integer> c1 = Integer::compare;

    //下面两句是一样的（实例方法引用1）
    persons.forEach(e -> System.out.println(e));
    persons.forEach(System.out::println);

    //下面两句是一样的（实例方法引用2）
    persons.forEach(person -> person.eat());
    persons.forEach(Person::eat);

    //下面两句是一样的（构造器引用）
    strList.stream().map(s -> new Integer(s));
    strList.stream().map(Integer::new);
```
### 生成器函数
生成器函数会产生一系列元素，供给一个流。
```java
Stream.generate(Supplier<T> s)
```
例如：
```java
Stream.generate(Math::random)
      .limit(5)
      .forEach(System.out::println);
```
limit(5)，如果没有这个调用，那么这条语句会永远地执行下去。也就是说这个生成器是无穷的。这种调用叫做终结操作，或者短路操作。
