title: Ajax
tags:
  - 闲聊
categories:
  - chet
author: 乔丁
date: 2018-10-15 09:31:00
description: hexo备份问题小结
photos: http://p7wm7amg2.bkt.clouddn.com/hexo.jpg
---

## balabala
hexo作为搭建博客十分美观方便，主要问题就是如何同步源码多端写作。这里总结了一下网上的一些方案并实践了一下。

## 方案一（我觉得不错的方案）
好处：用其他git仓库保存，配置最简<br>
核心：hexo-git-backup <br>
先贴上它的git地址：https://github.com/coneycode/hexo-git-backup

安装说明里面有。我遇到的问题是fatal: 'github' does not appear to be a git repository。解决办法在issue里面有：https://github.com/coneycode/hexo-git-backup/issues/8

多提一嘴，在选择开源库的时候多看几眼很重要，比如issue多少，作者解答情况还有更新的版本问题。

## 方案二
在原仓库里开一个备份分支<br>
master分支是用来放编译后静态网页代码的，那就在开一个备份分支。
1、把备份分支作为默认分支<br>
2、clone到本地，_config.yml，themes/，source，scffolds/，package.json，.gitignore复制到你克隆下来的仓库文件夹<br>
3、注意要将主题中的.git删掉，要不然推不上去<br>
4、检查执行git命令的分支是不是备份分支，然后直接add、commit、push<br>
5、不影响hexo d

首先，一个仓库里两个项目这种想法的确很逆天。其次，命令一堆有点烦，不过可以写个脚本执行

## 方案三
https://www.jianshu.com/p/53f37196f6e7
这里算是以上两种结合，且有一个自动跑的脚本，不用一堆命令。<br>
但也有问题，多端时也需要配置相关环境。

## In a Word
看过不如试过，试过不如自己搞一个