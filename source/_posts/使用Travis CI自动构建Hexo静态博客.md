---
title: 使用Travis CI自动构建Hexo静态博客
date: 2015-12-11 21:34:52
tags: [hexo, Travis CI]
categories: 叶帆
---
# 前言
随着现在open source越来越火，更多的人开始注重技术知识的获取。不可否认，目前的信息传播速度非常之快，渠道也非常之多，对于知识获取的整理和理解是很多人都在做的一件事情。在这种情况下更多的人开始选择写博客，把自己认知记录下来，一是为了检验自己对于技术的理解，二是为了让更多的人去从你的博客中获取到有用的信息。

我2014年的时候自己买了域名，用`jekyll + GitHub Pages`搭建了自己的博客，有兴趣的可以访问我的[博客](http://yeziahehe.com)。（目前正在考虑换到Hexo，而Coryphaei技术博客就是采用的Hexo）

# 技术
Coryphaei技术博客采用的是`Hexo + GitHub Pages + Travis CI`的技术方案，实现了多人同时更新博客并且自动化构建。

## Hexo
[Hexo](https://hexo.io/)是一款高效、简洁的静态博客框架，目前整个项目都开源在[GitHub](https://github.com/hexojs/hexo)上。因为部署极其简单，而且不需要数据库的支持，纯静态的模式，使得目前静态博客被越来越多的选择。关注与文章本身，创造出更有价值的文章才是每个写博客的人的初衷。

Hexo是由[Node.js](nodejs.org)完成，需要集成Node.js的开发环境，这里不再赘述。接下来开始集成Hexo的开发环境，因为我是OS X，所以一下所有的均是基于OS X环境的搭建教程。

首先，需要配置基本的环境。

### cnpm
> 注意：npm因为qiang的原因，经常会出问题，我换成了taobao的cmpn镜像，taobao的cnpm镜像这是一个完整 npmjs.org 镜像，你可以用此代替官方版本(只读)，同步频率目前为 10分钟 一次以保证尽量与官方服务同步。

安装方式
```bash
$ npm install -g cnpm --registry=https://registry.npm.taobao.org
```

### Hexo基本环境
Hexo基本环境的配置，步骤比较简单。

#### 安装
```bash
$ cnpm install hexo-cli -g
$ hexo init blog
$ cd blog
$ npm install
```

启动本地的服务器看下是否安装成功`hexo s`，浏览器打开`http://localhost:4000`。

#### 主题 Next
我采用的是[Next](https://github.com/iissnan/hexo-theme-next)主题，这个主题是国人写的，因为其简洁的特点，深受大家的喜欢。Next有官方出的[使用说明](http://theme-next.iissnan.com/)，大家有问题可以先去浏览使用说明。

安装非常简单
```bash
$ cd blog
$ git clone https://github.com/iissnan/hexo-theme-next themes/next
```

启用 NexT 主题
克隆/下载 完成后，打开根目录下的_config.yml，找到`theme`字段，并将其值更改为`next`。
启动本地的服务器看下是否安装成功`hexo s`，浏览器打开`http://localhost:4000`。

#### 基本配置的一些修改
对于博客的基本配置的个性化定制，完全可以参照Next官方出的[使用说明](http://theme-next.iissnan.com/)，我也附上我的 [_config.yml](https://github.com/Coryphaei/coryphaei.github.io/blob/blog/_config.yml) 和theme目录下的[themes/next/_config.yml](https://github.com/Coryphaei/coryphaei.github.io/blob/blog/themes/next/_config.yml)供大家参考。

到目前为止，整个Hexo的博客就搭建完毕。

## GitHub Pages
有关于GitHub Pages的问题，网上教程太多，大家可以自行google，这边就不在赘述。

## Travis CI
目前，自动化构建、持续集成的理念在整个计算行业非常的流行，大家更愿意去使用自动化代替手动，从而提高效率。

`持续集成`的概念
> 持续集成是一种软件开发实践。在持续集成中，团队成员频繁集成他们的工作成果，每人每天可能集成一次，甚至多次。每次集成会经过自动构建（包括自动测试）的检验，以尽快发现集成错误。许多团队发现这种方法可以显著减少集成引起的问题，并可以加快团队合作软件开发的速度。

自动构建工具则是持续集成的一种出色实践。代码提交后，由软件自动完成代码的测试、构建，并将过程中状态与构建物产出才是持续集成的意义。

[Travis CI](https://travis-ci.org/)就是一个在线的、分布式的持续集成服务，用来构建及测试在GitHub托管的代码。利用Travis CI 会在每一次push后生成一个虚拟机来执行事先安排好的自动构建任务，从来进行发布。

### 为什么使用
Travis CI本身已经是很好的自动构建的工具，而这里使用的原因，本质上是因为Hexo本身并不能进行多人合作。Hexo的`hexo generate`和`hexo deploy`会自动渲染并提交到GitHub上，所以当你从别的电脑上clone的时候，clone下来的是渲染好的html的文章。就算我在两个电脑上同时搭建了环境，但是每次渲染的时候只会渲染本地的markdown文章，依然不能进行同步。有些人选择了使用百度云进行同步，每次写之前下载下来并覆盖，就能进行同步。不否认，这个方法对于一个人写博客，在工作和家的电脑还算是比较方便的，因为始终是一个人进行操作。而我们的博客是多人共同写的，所以说会存在各种冲突问题，于是想到了用Travis CI。

### 原理
![travis-hexo-flowing](http://7xp57v.com1.z0.glb.clouddn.com/coryphaei/travis-hexo-flowing.png)
> 图片引用自v2cc的[博客](http://v2cc.github.io/2015/09/02/unbelievable-workflow-autodeploy-hexo-by-travis/)，并且其对于流程的讲解也对我产生了很大的帮助，非常感谢。

分析下思路：
前提：我们在之前博客搭建的repo下面，新建一个blog的分支，这个分支用来进行环境代码的备份，并且配置出`.travis.yml`进行自动化构建。
#### User - push -> branch blog
将代码push到在GitHub上的博客中的blog分支。

#### Dev repo - sync -> Travis CI
在branch blog中配置`.travis.yml`文件，在Travis CI中开启branch blog的同步开关，并启用`Build only if .travis.yml is present`项，这样能在repo中有多个branch时，让Travis CI只构建放置了`.travis.yml`文件的branch。

#### Travis CI - build and push -> Pages repo
Travis CI 的自动化构建完全依靠唯一的`.travis.yml`脚本文件。需要在此文件中添加构建环境、构建Hexo、生成博客及后续push到Pages repo的全部脚本。

- 生成SSH Key
要做到Travis CI向Pages repo自动推送就必须用到Github SSH Key，这样做的目的是免去Hexo部署时候输入密码的步骤。生成SSH Key的操作参照GitHub的官网即可：[Github SSH Key](https://help.github.com/articles/generating-ssh-keys/)。
这样会得到`id_rsa.pub`和`id_rsa`两个秘钥，我们将`id_rsa.pub`添加到了github，下面要加密`id_rsa`这个私钥并且上传到Travis。
> 注意：这个SSH key不应该是你账号的全局SSH Key，这样Travis CI就获得了你所有代码库的提交权限。仅仅只需要把SSH Key添加到当前repo的setting中的key下面即可。

- Travis CI 环境
```bash
$ sudo cp ~/.ssh/id_rsa / #将上一步得到的`id_rsa`复制到根目录下
$ vim .travis.yml #创建.travis.yml
$ gem install travis #安装Travis CI
$ travis login --auto #登录Travis CI，需要输入GitHub的账号密码
$ travis encrypt-file ssh_key --add #加密私钥并上传至Travis
```
  生成加密过得新秘钥`id_rsa.enc`, 并自动将branch blog中git的信息及解密秘钥的相关信息添加到`.travis.yml`中。然后手动删除私钥文件`id_rsa`， 以保证代码仓库的安全。

- SSH的设置
在当前目录下新建文件`ssh_config`，内容为
```bash
Host github.com
  User git
  StrictHostKeyChecking no
  IdentityFile ~/.ssh/id_rsa
  IdentitiesOnly yes
```
  修改`.travis.yml`中的命令，指定openssl解密后的生成位置，xxxxxxxxxx部分就是你的解密参数，不要去改动它。
```bash
- openssl aes-256-cbc -K $encrypted_xxxxxxxxxx_key -iv $encrypted_xxxxxxxxxx_iv
  -in travis.enc -out ~/.ssh/id_rsa -d
```

- 修改目录权限
紧接着在`.travis.yml`中修改目录权限
```bash
- chmod 600 ~/.ssh/id_rsa
```

- 将密钥加入系统
紧接着在`.travis.yml`中将密钥加入系统
```bash
- eval $(ssh-agent)
- ssh-add ~/.ssh/id_rsa
```

- 修改git信息
```bash
- cp ssh_config ~/.ssh/config
- git config --global user.name "username"
- git config --global user.email username@example.com
```

- 添加分支信息
```bash
branches:
  only:
  - blog
```

- 配置Hexo
```bash
install:
- npm install hexo-cli -g
- npm install hexo --save
- npm install

script:
- hexo clean
- hexo d
- hexo g
```
  这样就完成了`.travis.yml`的设置，这里是我的源文件[.travis.yml](https://github.com/Coryphaei/coryphaei.github.io/blob/blog/.travis.yml)。
```bash
language: node_js
node_js:
- '0.12'
branches:
  only:
  - blog
before_install:
- openssl aes-256-cbc -K $encrypted_b83a281ef741_key -iv $encrypted_b83a281ef741_iv
  -in id_rsa.enc -out ~/.ssh/id_rsa -d
- chmod 600 ~/.ssh/id_rsa
- eval $(ssh-agent)
- ssh-add ~/.ssh/id_rsa
- cp ssh_config ~/.ssh/config
- git config --global user.name "叶帆"
- git config --global user.email yeziahehe@gmail.com
- git clone -b master git@github.com:Coryphaei/coryphaei.github.io.git .deploy_git
install:
- npm install hexo-cli -g
- npm install
- npm install hexo-generator-feed --save
- npm install hexo-deployer-git --save
script:
- hexo clean
- hexo g
- hexo g
- hexo d
```
  这个时候应该将其push到blog分支。
> 注意，要删除themes/next/.git文件，否则会导致主题传不上去，渲染后首页空白的问题。

#### View the pages
打开首页，就能看到已经发布的最新博客。

# 问题
上述的所有步骤完成后，应该就已经成功了。我这边列举下我遇到的一些问题，希望能帮到大家。大家有任何问题也可以直接评论，我会第一时间回复。

- `.travis.yml`中的注释问题
![travis_yml_comment](http://7xp57v.com1.z0.glb.clouddn.com/coryphaei/travis_yml_comment.png)
一开始的时候我在`.travis.yml`中的注释用的是`//`，结果一直导致`missing config`。后来才知道YAML中注释应该用`#`。

- 首页无内容
一开始的时候我的首页一片空白，index.html中也是空的，原因就是因为主题Next是从GitHub上clone下来的，里面会存在`.git`文件，所以push到blog分支的时候千万要注意删除掉next文件夹中的`.git`文件。
```bash
cd blog/themes/next #到next主题文件夹下
ls -a #显示所有文件
rm .git #删除.git文件
ls -a #确认删除
```

- next主题会导致首页只显示最新的文章
很多人遇到发布后首页只显示最新的一篇文章，next主题[issue](https://github.com/iissnan/hexo-theme-next/issues/535)中也有提到这个。
解决办法就是`hexo g`命令做两遍，这个也是为什么我`.travis.yml`中的Hexo配置命令写了两遍的原因。被这个问题纠缠了很久，希望写出来能帮到大家，如果你没有问题就不需要在`.travis.yml`中写两遍命令。

# 结语
这个是我搭建这个博客写的第一篇文章，我也发现我这次解决问题会去弄个明白，回想之前写的博客，其实干货真的很少，知识也很肤浅，这次搭建博客--发现问题--解决问题给了我很好的体验，也让我学到了很多，我会尽可能的去写很多的干货去和大家分享！共勉！
