---
title: 发现了阿里云 APP 的一个小 BUG
urlname: 2019-07-17-bugs-of-aliyun-app
author: 章鱼猫先生
date: 2019-07-17
updated: "2021-06-25 10:48:40"
---

前几天在华为手机上使用阿里云 APP，从 oss bucket 中下载了一张图片，想要通过微信把它发给一个朋友时，发现在打开微信选择照片时却无法找到新下载的图片。同样在打开的 Android 系统相册中也完全找不到这个照片，虽然这个图片确确实实已经下载到了手机上。

- 手机系统：EMUI 8.0.0
- 阿里云 APP：V4.11.0

后来谷歌了一下，最后找出问题所在：
Android 只会在每次启动的时候扫描系统相册，并将扫描到的信息存储在数据库（MediaStore）。然后系统相册将直接调用数据库中的数据，所以当新的图片存到相册后，并没有将数据写入到数据库，所以在微信扫描里面自然就找不到这张图片了，所以解决的办法就是更新这个数据库。

为了验证问题，我在手机的文件管理中找到了这张图片所在的目录，进去重新刷新该文件夹，然后打开系统相册，果然就可以看到这张下载的图片了。

后来，向阿里云反馈也的确证实了该问题：
![](https://shub.weiyan.tech/yuque/elog-cookbook-img/FnfRQ6bjCk6drR_BULl4yyzFX0fK.png)

虽然本人对 Android 开发完全不懂， 但也想了解一下，在 Android 手机中下载图片怎么样才能实时刷新系统相册呢？

首先，是在 Adnroid 中保存图片。这里面会涉及到是否能指定路径保存和名称，方法可能有如下两种：

- 一，自己写方法。
- 二，调用系统提供的插入图库的方法保存图片。

其次，是更新系统图库。这里有三种策略：

- 重新扫描整个存储空间，将数据库更新。扫描整个 sd 卡的广播，如果 sd 卡里面东西多会扫描很久，用户体验差。
- 将新的数据加到数据库。样操作对数据的安全性要求很高。
- 直接扫描新添加的文件（或者是该文件所在的文件夹）。

关于在代码层面刷新系统 Media，网络上面主要几种方法：

- 通过操作 MediaStore 类。
- 发送广播更新 MediaStore。
- 通过操作 MediaScannerConnection 类。

当然或许会有更多的解决方法，小编作为一个 Adnroid 的小白，就不在这里啰嗦了。
