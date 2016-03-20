---
title: hexo 静态博客利器
date: 2016-03-16 13:36:34
tags: [hexo]
categories: coding
---

### 写在前面
今天给大家介绍一件写静态博客的利器--<strong>[hexo](https://hexo.io)</strong>,你所看见的我的个人博客也是用hexo搭建的，简单易用，通过命令行操作，搭建在github上。本来我买了一个个人域名[twsitzheng.com](http://twistzheng.com)指向博客的,不过因为是在国外网站买的（为了免去备案的麻烦），没用多久就被大天朝墙掉了--大天朝万能的墙啊！所以如果想买个人域名的话还是在大天朝买，老老实实去备案吧。

话不多说，说说hexo吧！

### hexo环境准备
hexo博客搭建需要准备[nodejs](https://nodejs.org/en/),以及git。

hexo命令行和依赖库的安装都基于nodejs环境（不得不说nodejs包管理工具npm的强大),而git工具便于我们将博客发布到github上。

#### 安装nodejs
nodejs的安装请自行去[nodejs](https://nodejs.org/en/)官网下载,根据自己的系统环境自行下载，windows下载安装包一路next即可，而linux官网也提供nodejs的源码下载，下载之后`./configure->make&make install`即可安装成功。

安装完成后在命令行窗口输入`node --version`和`npm --version`查看nodejs以及npm包管理工具是否安装成功.如下图所示.

![node npm](/images/node.jpg)

如果windows下提示安装成功，但敲击命令行不出现版本号的话，请检查nodejs和npm的exe文件路径是否在环境变量中。

#### git工具安装
在ubuntu下运行`sudo apt-get install git`即可，windows下的话我推荐大家下载<em>[Github for widows](http://github-for-windows.en.softonic.com/)</em>,这是github官方出的git工具，一下<em>hexo命令</em>，请在<em>git shell</em>里面运行。

#### 安装hexo
使用npm包工具安装hexo-cli命令行工具
```
  npm install -g hexo-cli
```
安装完成后，输入`hexo`显示如下则安装成功
![node hexo-cli](/images/hexo-cli.jpg)

运行`hexo init blog`,则会在当前目录下自动生成hexo相关文件,如图示：
![hexo blog](/images/hexo-blog.jpg)

运行命令`npm install`,安装hexo依赖包(npm 包管理器将依赖包安装在node_modules下面)
```json
{
  "name": "twist",
  "version": "0.0.0",
  "private": true,
  "hexo": {
    "version": "3.2.0"
  },
  "dependencies": {
    "hexo": "^3.1.1",
    "hexo-generator-archive": "^0.1.4",
    "hexo-generator-category": "^0.1.3",
    "hexo-generator-index": "^0.2.0",
    "hexo-generator-tag": "^0.2.0",
    "hexo-renderer-ejs": "^0.1.1",
    "hexo-renderer-stylus": "^0.3.0",
    "hexo-renderer-marked": "^0.2.9",
    "hexo-server": "^0.1.3",
    "hexo-deployer-git": "^0.1.0"
  }
}
```
`dependencies`下的列表即为依赖包
依赖安装成功之后，运行`hexo server`,在本地会创建一个node服务器，端口号为4000,
服务器启动之后，在浏览器中输入[localhost:4000](http://localhost:4000),即可看见你搭建在本机的博客了。

如果本地node服务器起不来，请检查4000端口是否被占用.

#### 将博客搭建在github上
1. 在github上创建一个公有仓库，取名为您的`用户名+github.io`，如我的为`xiaowei1118.github.io`
2. 确保您的电脑有github的密钥，如果没有在github工具上登陆一下您的账号即可，linux上使用ssh创建密钥后将公钥放到github上即可，具体请查找教程。
3. 在blog根目录下修改配置文件,请自行根据您的git修改.
```
deploy:
  type: git
  repo: git@github.com:xiaowei1118/xiaowei1118.github.io
  branch: master
```
4. 运行命令
```bash
  hexo clean   
  hexo generate   
  hexo deploy
```
  如果运行报错`ERROR Deployer not found`,即<em>hexo-deployer-git</em>依赖没有装（hexo init生成的文件中默认没有该依赖),运行`npm install hexo-deployer-git --save`,然后执行4操作即可.

  步骤4中的操作可以简化为`hexo d -g`，渲染和发布一个命令完成。
  发布成功之后在浏览器输入`git用户名+github.io`查看博客，恭喜你，到这里hexo静态博客的搭建就成功了。

### 写在最后
  hexo博客，我个人还是蛮喜欢的，简洁美观易用是hexo的主要特点。使用markdown来写文章也是我喜欢的一种方式，后期可能也会写关于markdown的博客。除此之外，hexo有很多第三方的主题都很不错，到文章发表之日，我用的这款主题是next（当然不排除以后会更换）。[hexo主题](https://hexo.io/themes/)点击这里，可以尝试更多漂亮的主题。

  快去试试搭建属于你自己的博客吧！
