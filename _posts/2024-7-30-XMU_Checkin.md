---
layout: post
title: 如何爬取厦大智慧教务签到码
description: 记录
tag: 随笔
---

## 前言

本文是作者在学习计算机网络八股文的过程中做的小实践，**仅供学习使用**，请勿使用此方法逃避考勤。



## 操作过程

### 爬取签到码前需要准备什么？

1. 一台可以联网并且安装有微信的电脑
2. 一个可以抓取网络流量包的软件（我使用的是Proxyman）



### 抓包过程

使用电脑登陆微信，搜索厦大智慧教务小程序并打开

![1.png](https://s2.loli.net/2024/11/27/s3yGPUxkJgRMCwK.png)



点击课程签到，进入课程签到界面

![1.png](https://s2.loli.net/2024/11/27/PK5yJuTpewAV1iX.png)

![1.png](https://s2.loli.net/2024/11/27/5Wi9pVkh1AQIR4Z.png)

然后电脑打开 Proxyman，并打开监听

![1.png](https://s2.loli.net/2024/11/27/tucHM8NoeXOaI7k.png)

之后回到小程序，点击“启动签到”，小程序会进入输入签到码并获取定位的界面，此时回到 Proxyman，找到 **`getXsQdInfo`** 这个包，打开 Response 的 Body ，找到 `"klHm"` 这个 key ，它对应的 value 就是签到码

![1.png](https://s2.loli.net/2024/11/27/kUOa9cpDBnNCWP5.png)

