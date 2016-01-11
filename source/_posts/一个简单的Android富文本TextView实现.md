﻿---
title: 一个简单的Android富文本TextView实现
date: 2016-1-11 11:12:00
tags: [Android,Java,RichTextview]
categories: Basti
---

# Android客户端富文本

## 现阶段问题
在Android客户端展现一个普通数据非常的方便，直接调用textview.setText()方法，但是当TextView比较复杂时（例如存在@用户，##话题，部分字符样式、网址链接等），普通的TextView就无法完成需求，需要自己封装一个富文本TextView。

## Demo
Coding 冒泡示例

![image](http://7xpvut.com1.z0.glb.clouddn.com/coding.png)
##

## 分析
1.一个TextView中不同类别的信息，需要由不同样式的显示，一般的用法textview.setTextColor(color)会将所有的文字变成同一种颜色，这显然是不符合要求的。为了实现这一个需求，我们将会用到SpannableStringBuilder这个类。

## 预备知识
### SpannableStringBuilder：
#### 基本概念
SpannableStringBuilder 和 StringBuilder类似，都可以存储字符串，不同的是SpannableStringBuilder有一个setSpan()函数，可以给存储的String添加不同的样式。如加下划线、背景色、字体颜色、字体大小等。

另外需要注意的是，当SpannableStringBuilder中存储了一个有样式的String，当把spannableStringBuilder展示在TextView、EditTextView中时，能显示这些样式；当展示在canvas上时，因为Canvas不支持SpannableStringBuilder的额外信息，所以会退化成一个普通的String,不显示样式信息。
#### setSpan()函数
void setSpan（Object what,int startIndex,int endIndex,int flag）;

说明：

|参数 |说明 |
|---|:---|:---|
|Object what|设置Span样式|
|int startIndex|样式开始的Index|
|int endIndex|样式结束的Index|
|int flag|新插入字符的样式设置|

注意点：
1. endIndex:字体样式结束的Index，该Index对应的字符不使用样式，比如有一个字符串为s = "abcd"，s.setSpan(span,0,2,flag),此时第0、1个字符ab使用了样式span，endIndex对应的字符c不使用。
2. flag:取值如下

|取值|说明|
|---|:---|
|Spannable.SPAN_EXCLUSIVE_EXCLUSIVE|前后都不包括，即在指定范围的前面和后面插入新字符都不会应用新样式|
|Spannable.SPAN_EXCLUSIVE_INCLUSIVE|前面不包括，后面包括。即仅在范围字符的后面插入新字符时会应用新样式|
|Spannable.SPAN_INCLUSIVE_EXCLUSIVE	|前面包括，后面不包括。|
|Spannable.SPAN_INCLUSIVE_INCLUSIVE	|前后都包括|

#### 简单示例

```java
//设置字体颜色
textview1 = (TextView) findViewById(R.id.text1);
SpannableStringBuilder spannableStringBuilder1 = new SpannableStringBuilder("Android");
ForegroundColorSpan foregroundColorSpan = new ForegroundColorSpan(Color.BLUE);
spannableStringBuilder1.setSpan(foregroundColorSpan, 0, 3, Spannable.SPAN_EXCLUSIVE_INCLUSIVE);
textview1.setText(spannableStringBuilder1);
```
效果：
![image](http://7xpvut.com1.z0.glb.clouddn.com/demo1.png)

#### 多种Span
由以上的简单示例我们可以看出，设置一个样式的一般步骤是：
1. 构造一个SpannableStringBuilder
2. 构造一个Span，并设置在SpannableStringBuilder上
3. 将SpannableStringBuilder绑定在TextView上展示

##### 设置字体颜色
这个已经在简单示例中展示

##### 设置字体背景颜色
```java
//设置字体背景颜色
textview2 = (TextView) findViewById(R.id.text2);
SpannableStringBuilder spannableStringBuilder2 = new SpannableStringBuilder("Android");
BackgroundColorSpan backgroundColorSpan = new BackgroundColorSpan(Color.RED);
spannableStringBuilder2.setSpan(backgroundColorSpan, 0, 3, Spannable.SPAN_EXCLUSIVE_INCLUSIVE);
textview2.setText(spannableStringBuilder2);
```
效果：
![image](http://7xpvut.com1.z0.glb.clouddn.com/demo2.png)

##### 设置字体大小
```java
//设置字体大小
textview3 = (TextView) findViewById(R.id.text3);
SpannableStringBuilder spannableStringBuilder3 = new SpannableStringBuilder("Android");
AbsoluteSizeSpan absoluteSizeSpan = new AbsoluteSizeSpan(30);
spannableStringBuilder3.setSpan(absoluteSizeSpan, 0, 3, Spannable.SPAN_EXCLUSIVE_INCLUSIVE);
textview3.setText(spannableStringBuilder3);
```
效果：
![image](http://7xpvut.com1.z0.glb.clouddn.com/demo3.png)

##### 设置字体
```java
//设置字体(加粗斜体)
textview4 = (TextView) findViewById(R.id.text4);
SpannableStringBuilder spannableStringBuilder4 = new SpannableStringBuilder("Android");
StyleSpan styleSpan = new StyleSpan(Typeface.BOLD_ITALIC);
spannableStringBuilder4.setSpan(styleSpan, 0, 3, Spannable.SPAN_EXCLUSIVE_INCLUSIVE);
textview4.setText(spannableStringBuilder4);
```
效果：
![image](http://7xpvut.com1.z0.glb.clouddn.com/demo4.png)

##### 设置下划线
```java
//设置下划线
textview5 = (TextView) findViewById(R.id.text5);
SpannableStringBuilder spannableStringBuilder5 = new SpannableStringBuilder("Android");
UnderlineSpan underlineSpan = new UnderlineSpan();
spannableStringBuilder5.setSpan(underlineSpan, 0, 3, Spannable.SPAN_EXCLUSIVE_INCLUSIVE);
textview5.setText(spannableStringBuilder5);
```
效果：
![image](http://7xpvut.com1.z0.glb.clouddn.com/demo5.png)

##### 设置下划线
```java
//设置删除线
textview6 = (TextView) findViewById(R.id.text6);
SpannableStringBuilder spannableStringBuilder6 = new SpannableStringBuilder("Android");
StrikethroughSpan strikethroughSpan = new StrikethroughSpan();
spannableStringBuilder6.setSpan(strikethroughSpan, 0, 3, Spannable.SPAN_EXCLUSIVE_INCLUSIVE);
textview6.setText(spannableStringBuilder6);
```
效果：
![image](http://7xpvut.com1.z0.glb.clouddn.com/demo6.png)

##### 设置多种样式
除了设置一个样式以外，SpannableStringBuilder还支持设置多个样式。
```java
//设置多种样式
textview7 = (TextView) findViewById(R.id.text7);
SpannableStringBuilder spannableStringBuilder7 = new SpannableStringBuilder("Android");
spannableStringBuilder7.setSpan(foregroundColorSpan, 0, 3, Spannable.SPAN_EXCLUSIVE_INCLUSIVE);
spannableStringBuilder7.setSpan(backgroundColorSpan, 0, 3, Spannable.SPAN_EXCLUSIVE_INCLUSIVE);
spannableStringBuilder7.setSpan(underlineSpan, 0, 3, Spannable.SPAN_EXCLUSIVE_INCLUSIVE);
spannableStringBuilder7.setSpan(absoluteSizeSpan, 3, 6, Spannable.SPAN_EXCLUSIVE_INCLUSIVE);
spannableStringBuilder7.setSpan(strikethroughSpan, 3, 6, Spannable.SPAN_EXCLUSIVE_INCLUSIVE);
textview7.setText(spannableStringBuilder7);
```
效果：
![image](http://7xpvut.com1.z0.glb.clouddn.com/demo7.png)

##### 点击事件
实例代码
```java
SpannableStringBuilder spannableStringBuilder = new SpannableStringBuilder("Android");

spannableStringBuilder.setSpan(
    new ClickableSpan() {
        @Override
        public void onClick(View widget) {
            //do something
        }

        @Override
        public void updateDrawState(TextPaint ds) {
            //设置一些样式
            //ds.setUnderlineText(false);
            //ds.setColor(color);
        }
    }, startIndex, endIndex,
    Spannable.SPAN_EXCLUSIVE_EXCLUSIVE
);
```

#### 简单实现
1. 以Listview为例，我们需要实现的是每一个Item和Adapter中的getView()函数。
2. 布局文件
```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent" android:layout_height="wrap_content">

    <ImageView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:src="@mipmap/ic_launcher"
        android:layout_marginTop="16dp"
        android:layout_marginLeft="16dp"
        android:id="@+id/userhead"/>

    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="16dp"
        android:layout_marginLeft="8dp"
        android:layout_marginRight="16dp"
        android:layout_toRightOf="@id/userhead"
        android:id="@+id/content"/>
</RelativeLayout>
```
ImageView表示该Item的发布人，和富文本没有关系，在Demo中直接写死数据。
主要内容在TextView中
3. 因为每一个TextView中可能有多个不同样式的文本（#话题#，@其他用户，文本内容等），所以先封装一个文本类：
```java
public class TextObject {

    private int mStartIndex;//样式的开始字符
    private int mEndIndex;//结束字符
    //private SpannableStringBuilder mSpannableStringBuilder;
    private String content;//文本内容
    private int mOptionType;//操作符
    private int color;//字体颜色
    private boolean isUnderline;//是否有下划线
    //...其他需要的属性可继承TextObject自行添加
  }
```
这个类封装的是一个Item中的某一个文本样式，所以我们还需要封装一个Item中的文本类：
```java
public class Content {

    private List<TextObject> mList;

    private SpannableStringBuilder mSpannableStringBuilder;
  }
```
这个类中有一个List成员变量，存放着该Item中所有的样式文本。
对于成员变量SpannableStringBuilder，有一个init函数，用于根据List中的TextObject拼接StringBuilder
```java
//拼接SpannableStringBuilder
public void initStringBuilder(){

    SpannableStringBuilder spannableStringBuilder = new SpannableStringBuilder();
    for (int i = 0;i<mList.size();i++){
        spannableStringBuilder.append(mList.get(i).getContent());
    }
    setmSpannableStringBuilder(spannableStringBuilder);
}
```
到现在，数据准备工作已经全部完成，现在就要显示这些数据了。
4. getView()函数

getview函数中就可以通过content.getSpannableStringBuilder()函数获取到正文内容，然后即可根据上面的预备知识对其进行设置不同的样式和点击事件。
设置完成后，需要调用
```java
viewHolder.textview.setMovementMethod(LinkMovementMethod.getInstance());
```
使其响应点击事件。

##### Demo效果：

![image](http://7xpvut.com1.z0.glb.clouddn.com/demo.gif)

#### Demo地址
[Github项目地址](https://github.com/basti-shi031/RichTextView)
