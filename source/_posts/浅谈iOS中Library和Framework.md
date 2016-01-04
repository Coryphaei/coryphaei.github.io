---
title: 浅谈iOS中Library和Framework
date: 2015-12-30 10:40:25
tags: [iOS, Framework, Objective-C, Swift]
categories: 叶帆
---
# 写在前面
有关于库的出现场景，其实很简单的回答就是，不愿意把实现的源码暴露给其他人。虽然我是脑残的开源爱好者，但是总有些场景下，有这样的需求，比如外包公司的框架，比如我这次写NativeScript，需要自己将三方库打包然后使用在js中调用（当然后来我看到支持CocoaPods的时候，喷了一口老血）。不过这个是客观的一些原因，还有一些有想法的程序员，为了减少编译的时间，也会选择将改动不大的代码进行打包。打包好的代码是编译完成的二进制文件，在项目进行编译的时候链接上，确实一定程度的减少编译时间。

# 库的本质
库的本质，我觉得非常好理解。我们首先要知道对于所有的库，都是进行编译的，而编译生成的其实是一段二进制代码。所以我们可以给库下个定义：
> 提供头文件的编译好的二进制代码。

对于库的链接，分为了动态链接和静态链接，这也就产生了iOS中的静态库和动态库。

# 静态库和动态库
静态库（Windows 下的 .lib，Linux 和 Mac 下的 .a）。
动态库（Windows 下的 .dll，Linux 下的 .so，Mac 下的 .dylib）。

和所有平台所理解静态链接库、动态链接库一样，静态和动态都是相对于编译和运行来讲的：静态库在编译的时候就会被拷贝到目标程序中，运行的时候就不会在改变了；动态库在编译的时候是不会拷贝到目标程序中，在运行的时候会将库加载进来。

## 编译和运行
区分下编译和运行的概念。在Xcode中我们进行CMD+R的时候其实做了很多的步骤，编译和运行都包括在其中。

编译：如果我们自己打包的时候，使用的是CMD+B的命令，其实是调用了LLVM编译器，进行编译，编程计算机能识别的二进制码。
运行：将编译生成的文件链接为可执行文件并进行运行。

静态库就是在目标程序编译的时候已经存在了编译好了的二进制代码，所以说目标程序编译的时候不需要对这段代码进行改变，而且还减少了编译时间。
动态库就是在目标程序编译的时候不去链接，而是创建了引用，在运行的时候，进行对动态库的链接和编译。

## 优缺点
静态库的好处很明显，编译完成之后，库文件实际上就没有作用了。目标程序没有外部依赖，直接就可以运行。当然其缺点也很明显，就是会使用目标程序的体积增大。

动态库的优点，不需要拷贝到目标程序中，不会影响目标程序的体积，而且同一份库可以被多个程序使用（因为这个原因，动态库也被称作共享库）。同时，编译时才载入的特性，也可以让我们随时对库进行替换，而不需要重新编译代码，这样就可以实现动态更新。动态库带来的问题主要是，动态载入会带来一部分性能损失，使用动态库也会使得程序依赖于外部环境。如果环境缺少动态库或者库的版本不正确，就会导致程序无法运行。

# Framework
Framework是Mac OS/iOS平台特有的一种打包方式，将编译生成的二进制文件、头文件、资源文件统一打包。可以包含如下的东西：
- 共享库
- 描述API的头文件
- 文档
- 资源文件（UI，Assets，配置文件）

在iOS 8之前我们Framework其实就是静态库，原因很简单，之前Apple官方不支持动态打包，我们只能使用官方的UIKit或者Foundation等Framework。从iOS 8之后苹果开放了对动态Framework的支持，这应该是苹果为支持 Extension 这一特性而做出的选择（Extension 和 App 是两个分开的可执行文件，它们之间共享代码，所以需要 Framework 支持）。不过我们这个和系统还是有区别，系统的 Framework 不需要拷贝到目标程序中，我们自己做出来的 Framework 哪怕是动态的，最后也还是要拷贝到 App 中，因此苹果又把这种 Framework 称为Embedded Framework。这个时候所谓的动态库其实意义就是升级版的静态库，因为动态库使用的前提是项目在发布前添加到项目中，这和我们所谓的插件（即插即用，随时在自己的服务器上下载一个动态库运行，而不需要重新打包，我们可以选择在需要的时候再加载动态库）完全是两码事。当然我们可以通过方法去实现Framework的动态更新，这里不做赘述。

## 创建
应该说现在创建一个Framework非常的方便，我们基本上不需要很简单的就能够制作一个Framework。
首先我们新建一个Framwork的项目
![新建项目](http://7xkvt5.com1.z0.glb.clouddn.com/coryphaei%2Fcreate_framework.png)

然后把我们需要创建的文件加入其中，注意这边添加的是Swift的文件，因为Swift中是没有.h和.m的，所以会默认会帮你生成一个xxx-Swift.h的头文件，其中是所有public属性和方法都暴露了出来。
![添加文件](http://7xkvt5.com1.z0.glb.clouddn.com/coryphaei%2Fadd_file.png)

## 配置
接下来应该是有个配置的过程，这边需要详细讲下这些变量是什么意思。

### arm
arm代表的是使用的设备的处理器的型号，大致分为以下几种：
```
arm64 = iPhone 5s, iPad Air, Retina iPad Mini
armv7s = iPhone 5, iPhone 5c, iPad 4
armv7  = iPhone 3GS, iPhone 4, iPhone 4S, iPod 3G/4G/5G, iPad, iPad 2, iPad 3, iPad Mini   
i386 = 32 bit simulator
x86_64 = 64 bit simulator
```

### 几个设置
Architectures：该编译选项指定了工程将被编译成支持哪些指令集，支持指令集是通过编译生成对应的二进制数据包实现的，如果支持的指令集数目有多个，就会编译出包含多个指令集代码的数据包，造成最终编译的包很大。
Valid Architectures：该编译项指定可能支持的指令集。
最后生成的支持工程的指令集应该是上面两个所产生的交集。

Build Active Architecture Only：该编译项用于设置是否只编译当前使用的设备对应的arm指令集。
当这个选项设置为YES的时候，不管Architectures和Valid Architectures设置为什么，最后都只会输出支持当前使用设备对应的arm指令集。
通常debug选择YES，release选择NO。

有关于指令集的选择，因为指令集有着向下兼容的特性，所以说为了减少包的大小，我们通常选择只支持armv7，在armv7s和arm64的机器上同样可以使用，当然性能有部分损失，可以忽略不计。
![config](http://7xkvt5.com1.z0.glb.clouddn.com/coryphaei%2Fconfig.png)

## 编译
有关于编译，一共有两种方式：release和debug。
可以手动设置run的方式。
![modify_run](http://7xkvt5.com1.z0.glb.clouddn.com/coryphaei%2Fmodify_run.png)
![modify_run_release](http://7xkvt5.com1.z0.glb.clouddn.com/coryphaei%2Fmodify_run_release.png)

## 完成
Products里面生成了Framework，找到Framework的位置。
![Fframework_finish](http://7xkvt5.com1.z0.glb.clouddn.com/coryphaei%2Fframework_finish.png)

## 合并
接下来应该是需要把模拟器和真机的Framework进行合并，网上应该有脚本，我是直接实用命令行的方式。
```bash
lipo -create Debug-iphoneos/SocketIO.framework/SocketIO Debug-iphonesimulator/SocketIO.framework/SocketIO -output SocketIOLib
```
合并完成后进行查看，可以看到支持的指令集。
```bash
lipo -info ~/Library/Developer/Xcode/DerivedData/SocketIO-bwzuuvegsamvhtbdwasganalbadg/Build/Products/Debug-iphoneos/SocketIO.framework/SocketIO
```
