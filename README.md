# Coryphaei技术博客使用方法
`目前仅限于团队使用，其他人提交应该是不成功的。`

基于[Travis CI](https://travis-ci.org/)进行自动化构建。

# 步骤
合作者按照如下步骤进行博客提交和编写。

## Step 1
第一次clone

```bash
https://github.com/Coryphaei/coryphaei.github.io.git
```

否则先

```bash
git pull origin blog
```

## Step 2
进入`source/_post`文件中，新建markdown文件。

markdown格式如下:

```
---
title: 文章题目
date: 2015-10-12 16:53:25
tags: [iOS, Objective-C]
categories: Ivan
---
# 开始
```

`tags` 代表文章的标签，如iOS, Android，`categories` 成员写自己的名字就可以。

## Step 3
写完之后提交
```bash
git add .
git commit -m "名称-文章名字"
git push origin blog
```

# License
Write by [Ivan](https://github.com/yeziahehe)

Support By [Hexo](https://github.com/hexojs/hexo)

Available under the MIT license. See the LICENSE file for more info.
