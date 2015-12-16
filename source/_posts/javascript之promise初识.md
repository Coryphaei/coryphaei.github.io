title: javascript之promise初识
date: 2015-12-16 17:30:34
tags: [javascript,nodejs]
categories: twist.zheng
---

### promise是什么？
在javascript promise机制出现之前，基于javascript的异步回调机制，当我们要顺序执行一段代码时，需要在异步回调里面执行代码，有时候需要在回调里面添加回调函数，如此一来，以前写代码之前代码是这样的：
```
firstAsrc(function(item){          //第一个异步回调函数
    //coding do something
    secondASrc(function(item){    //第二个回掉函数
        //coding do something
      });
  });
```
这还是只有两个回调函数，如果有十几个呢，我相信你就会疯了，至少我会。
所以，promise是用来解决javascript中层层异步调用，实现链式调用模式的一种机制。

### promise的用法
promise在es6中已经得到了完美支持，而nodejs中也有promise的三方类库支持。现在我们来看看promise到底要怎么使用。

现在我们使用promise来实现函数的顺序调用
```
function firstAsrc(resolve,reject){   //接受两个函数作为参数，处理正常事务和异常的事务
  //do something
   console.log("step 1");
   if(a==1){
      resolve(data);
   }else{
     reject(data);
   }
}

function secondAsrc(resolve,reject){
  //do something
  console.log("step 2");
  if(a==1){
     resolve(data);
  }else{
    reject(data);
  }
}

function thirdAsrc(resolve,reject){
  //do something
  console.log("step 3");
  if(a==1){
     resolve(data);
  }else{
    reject(data);
  }
}

new Promise(firstAsrc).then(function(){
    console.log("firstAsrc success");
    return new Promise(secondAsrc);
  },function(data){
    console.log("firstAsrc error");
    return new Promise(secondAsrc);
  }).then(function(data){
    console.log("secondASrc success");
    return new Promise(thirdAsrc);
  },function(data){
    console.log("secondASrc error");
    return new Promise(thirdAsrc);
  }).then(function(data){
    console.log("thirdAsrc success");
    return data;
  },function(data){
    console.log("thirdAsrc error");
    return data;
  })

```
打印顺序:
```
   step 1
   firstAsrc success
   step 2
   secondASrc success
   step 3
   thirdAsrc success
```
现在我们来看看上面的代码,`Promise` 类是一个对象，这个对象有一个`then`方法，then方法有两个参数，这两个参数传入两个函数resolve和reject,所有的回调只执行者两个函数中的任意一个，其中resolve用于处理成功的回调，reject用户处理异常的回调。一般的形式为`promise.then(resolve,reject)`。

我们注意到then函数执行完后返回的还是一个Promise对象`return new Promise(secondAsrc)`,Promise对象的构造函数接收一个带有异步逻辑的函数作为参数。既然then方法里面返回的依然是promise对象，那么是否意味着我们可以继续调用then方法呢？如上，确实如此。这样我们就可以将异步调用以链式的形式实现，
