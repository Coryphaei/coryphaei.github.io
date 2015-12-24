title: Html5表单验证在ajax提交时失效的处理
date: 2015-12-24 16:24:24
tags: [html5,javascript]
categories: twist.zheng
---

### 问题描述
html5为我们提供了强大的校验功能，这无疑为我们手动写验证或者使用JQueryValide要方便的多，这里我们并不去缀述html5的自动校验功能，我们来说一下在ajax提交form表单的情况下验证失效的问题。众所周知，form表单的提交是带跳转的，如果不想刷新或者跳转页面的话，我们必须使用ajax来提交表单数据，这样的话又会带来一个问题，就是form表单的校验就会失效，这样的话html5强大的验证功能我们就失去了，怎么来解决这个办法呢？

### 问题分析
我们知道，html5中form表单验证的触发是在表单提交的时候，在html中提交表单只需有个type=submit的button，在点击该button之后，便会触发表单验证以及向远端提交数据。一般情况下，我们在使用ajax提交表单的时候，坚听了一个type=button的button的点击事件，然后向远端传递一个序列化的表单数据，从始至终我们都没有看到form的submit()事件的触发，这是为什么我们表单验证失效的原因。

这样，我们就有了解决问题的思路，我们能不能触发submit事件，同时又阻止form表单提交数据并跳转呢。答案是可以的。

### 解决问题
如果我们监听了form的submit事件，但是又阻止其默认的提交行为，是不是就可以达到效果？
在JQuery中有个event.preventDefault()方法，可以阻止元素发生默认的行为和事件。有了这个前提我们就可以实现了。
```
<form>
   <!--code here-->
   <input type="submit"/>
</form>

$("form").submit(function(){
    event.preventDefault();
    $.ajax({

    });
}
```
