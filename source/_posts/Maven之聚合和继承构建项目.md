---
title: Maven之聚合和继承构建项目
date: 2016-03-16 16:46:43
tags: [Java,Maven]
categories: coding
---

如果你是一个JAVA攻城狮，那么你还是有很大几率会接触到Maven的，Maven是apache旗下推出的项目构建和包管理工具，如果你对Maven是什么不太了解，我推荐你还是可以去看看它的[官网](http://maven.apache.org/)。这里我们就不再赘述（也许以后我还会出关于Maven的博客)。

### 构建步骤
#### 第一步
新建一个父maven工程，作为一个单独的maven的project工程，不包含源代码以及配置文件，pom文件中，package方式设置为pom方式，在ｐｏｍ中添加的依赖库将被子程序继承.配置如下：
```xml
    <groupId>com.changyu.partent</groupId>
    <artifactId>partent_demo</artifactId>
    <packaging>pom</packaging>
    <version>0.0.1-SNAPSHOT</version>
    <name>partent_demo Maven Webapp</name>
    <url>http://maven.apache.org</url>
     <modules>   
          <module>child-task</module>
          <module>child_web</module>
          <module>common-changyu</module>
     </modules>
 ```

#### 第二步
新建Maven的ｍodule子项目，groudId应该和父maven项目一致（最好一致）,package
方式应该为war包形式，pom中的配置如下：
```xml
<parent>
    <artifactId>partent_demo</artifactId>
    <groupId>com.changyu.partent</groupId>
    <version>0.0.1-SNAPSHOT</version>
</parent>
<groupId>com.changyu.partent</groupId>
<artifactId>child-task</artifactId>
<version>0.0.1-SNAPSHOT</version>
<packaging>war</packaging>
<name>child-task Maven Webapp</name>
<url>http://maven.apache.org</url>
 ```

### 第三步
新建一个module子项目，为别的子项目提供服务，为了方便，我们称呼为ｃｏｍｍｏｎ项目，该maven项目中包含一些常用的工具包，一些基本dao操作，打包方式为jar形式，配置方式如下：
```xml
<parent>
  <artifactId>partent_demo</artifactId>
  <groupId>com.changyu.partent</groupId>
  <version>0.0.1-SNAPSHOT</version>
</parent>
<groupId>com.changyu.partent</groupId>
<artifactId>common-changyu</artifactId>
<version>0.0.1-SNAPSHOT</version>
<packaging>jar</packaging>
<name>common-changyu Maven Webapp</name>
<url>http://maven.apache.org</url>
```

#### 第四步：
将common项目用maven编译并提交到本地仓库，执行的命令为mvn:clean install
在eclipse中，右键点击工程，run as->maven build->goal中输入clean install
即可．这样我们在子别的子项目中即可添加对common包中的依赖了.


#### 第五步
在其余的子项目中添加对common项目的依赖，在pom文件中添加common项目的坐标．配置如下：
```xml
<dependency>
   <groupId>com.changyu.partent</groupId>
   <artifactId>common-changyu</artifactId>
  　<version>0.0.1-SNAPSHOT</version>
</dependency>
```

这样即可对common项目中的包进行正常调用了．

#### 第六步：
对整个项目进行打包，同common项目中install一样，对父工程执行mvn:clean package，在 此之后会在各个打包方式为war的工程目录下，生成一个war文件，然后将该war包发布在服务器 下即可．


  谢谢！
