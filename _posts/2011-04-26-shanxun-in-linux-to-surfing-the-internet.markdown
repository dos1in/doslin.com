---
author: labuxiaojue
comments: true
date: 2011-04-26 10:49:01
layout: post
slug: shanxun-in-linux-to-surfing-the-internet
title: 闪讯钳制下Linux系统上网解决方案
wordpress_id: 90
categories:
- Notes
tags:
- Linux
- Ubuntu
---

很多童鞋是的学校是用电信的闪讯客户端拨号上网的，而目前电信只有的Mac和Windows版本的闪讯，Linux版遥遥无期。前不久看到如何在使用闪讯的情况上使用路由器共享上网，联想到了宿主机与虚拟机之间的网络状态类似上述情形，于是照着这个想法鼓捣了一下，发现本机的Ubuntu可以上网了。到目前为止使用了将近一个月无鸭梨。现在撰之成文分享一下。P.S.这个方法的思路应该也适用于使用其他第三方拨号客户端程序才能上网的情形。

<!-- more -->

下面进入正题：

一、准备工作，你需要有：

1、可以正常工作的Linux(本机为Ubuntu)，已经安装Vitualbox并且安装好Windows(下文特指XP)

2、闪讯官方最新版客户端程序

3、布闪廖(另：闪讯终结者)：http://u.115.com/file/f3cba705fe

二、真正的正题：

1、进入Ubuntu，打开XP虚拟机，更改网络连接方式为Bridged Adapter

[![](/assets/post_img/shanxun-in-linux-to-surfing-the-internet/1.png)](/assets/post_img/shanxun-in-linux-to-surfing-the-internet/1.png)

2、XP下打开闪讯拨号，保证虚拟机可以上网。

3、进入控制面板中的网络连接，右键“ChinaNetSNWide”打开“属性”，点击“高级”标签，勾选“允许其他网络用户通过此计算机的Internet连接来连接”，点击“确定”保存退出。（见下图）

[![](/assets/post_img/shanxun-in-linux-to-surfing-the-internet/2.png)](/assets/post_img/shanxun-in-linux-to-surfing-the-internet/2.png)

４、右键打开本地连接属性。在“常规”选项卡中选中“Internet协议(TCP/IP)”并打开其属性。

为了说明简单点，直接按下图设置。注意：DNS服务器IP的值请打开CMD，输入ipconfig /all回车后得到你的地址，然后输入。

[![](/assets/post_img/shanxun-in-linux-to-surfing-the-internet/3.png)](/assets/post_img/shanxun-in-linux-to-surfing-the-internet/3.png)

5、此时打开“布闪廖”，将闪讯用户名及密码输入，连接名默认为ChinaNetSNWide。然后点“连接”，这时软件自动关闭。最后请去虚拟机退出官方闪讯程序。

6、切换回Ubuntu，进入网络连接设置按图配置IP即可。注意网关地址一定要和刚在XP下设置的IP一致(如文中为192.168.1.111)。

[![](/assets/post_img/shanxun-in-linux-to-surfing-the-internet/4.png)](/assets/post_img/shanxun-in-linux-to-surfing-the-internet/4.png)

7、这时你惊喜地发现你的ubuntu可以sudo apt-get update了。

tips：

1、为了减少每次上网都有打开虚拟机的开销，可以禁用XP不必要的服务+主题特效。

2、为了避免不必要的每次上网前的繁琐操作，上网结束后，可以休眠虚拟机，就不需要总设置了。


[![](/assets/post_img/shanxun-in-linux-to-surfing-the-internet/5.png)](/assets/post_img/shanxun-in-linux-to-surfing-the-internet/5.png)
