---
author: labuxiaojue
comments: true
date: 2011-05-12 10:17:40
layout: post
slug: win7-share-wifi-for-android
title: Win7共享WIFI热点让Android手机上网
wordpress_id: 103
categories:
- Notes
tags:
- Android
- Windows
---

Win7自带的无线网络分享功能建立的是Ad hoc网络，而Android的手机(官方ROM)默认不支持该网络，这也就是为什么明明构造了模拟热点，在手机中却搜不到该网络。




以下通过命令行开启Win7隐藏功能开启虚拟无线AP模式可让你的安卓机分享Win7的网络（注意台式机需要无线网卡）。




1、首先以管理员身份运行CMD（命令行程序）；




2、输入


`netsh wlan set hostednetwork mode=allow ssid=要广播的名字 key=你的连接密码`
<!-- more -->

使用完毕需要禁用网络的话只需要把_mode=allow_改为_mode=disallow_即可




上述命令运行成功后你会发现你的网络共享中心-更改适配器设置中会多出一个名为“Microsoft Virtual WiFi Miniport Adapter”的连接。




3、以下进行设置Internet连接共享：


在“网络连接”窗口中，右键打开本机已经拨号的网络连接（如“宽带连接”或其他），选择“属性”→“共享”，勾上“允许其他······连接(N)”并选择“Microsoft Virtual WiFi Miniport Adapter”。确定之后，提供共享的网卡图标旁会出现“共享的”字样，表示“宽带连接”已共享至“Microsoft Virtual WiFi Miniport Adapter”。


4、命令提示符中运行：`netsh wlan start hostednetwork`




如需关闭该功能只需运行 `netsh wlan stop hostednetwork`




至此你可以发现你的安卓机可搜到刚刚共享的WIFI了。




_以上教程更改自网络，些许步骤按个人实际操作过程中进行了改动，如有错误，敬请斧正。_
