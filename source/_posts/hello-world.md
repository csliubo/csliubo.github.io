---
title: 为群晖 Download Station 写了一个小App
tags: [日常]
---
前段时间搞了一个DS918+，除了在上面部署了一大堆的docker服务之外，日常使用还是用来下电影和美剧居多。

### 0x00
在日常的下载中碰到一些问题：
* 找电影资源的时候经常碰到热心网友只提供一个BTIH的哈希值，没tracker很尴尬。
* 对于有tracker的也经常需要自己新增一些热门tracker来进行加速。
* 有时候不在电脑面前，又想看看下载进度，没有好用的App。iOS上的DS get被下架了；还有一个热心网友提供的App界面实在太古老了，也pass掉。

<!-- more -->

### 0x01
不服就干呗，于是就找了个周末用Flutter写了个App。在这里要吐槽一下群晖的api是什么鬼设计，新版本的api还找不到文档，只能自己抓包看。

除了看看下载进度之外，还可以在添加磁力下载任务的时候自动添加tracker服务器。tracker服务器支持编辑，也内置了一些。还增加了对magnet连接的支持，就是在网页中直接点击磁力连接可以打开app进行下载。

基本上是满足了自己的需求。

### 0x02
自己用了几天之后，想着说不定有其他人也需要呢，就吭哧吭哧去提交App Store了:)

第二天就收到App Store的审核被拒信息
```
Your app allows users to save or download music, video, or other media content without authorization from the relevant third-party sources.
```

### 0x03
当然，就这么放弃肯定是不行的，咱们要跟审核人员斗智斗勇啊。于是，我把下载入口隐藏（留了个开关），继续拿去提交。
看看过几天审核会不会通过吧:)

### 0x04
后面还是被机智的审核人员发现了，这个app就暂时自己用吧~
