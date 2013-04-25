---
author: labuxiaojue
comments: true
date: 2011-05-26 15:34:08
layout: post
slug: fedora-install-nvidia-dirve
title: Fedora安装N卡驱动全过程
wordpress_id: 117
categories:
- Linux
tags:
- Fedora
- Linux
---

折腾了一个上午，搞定了N卡驱动安装，现将安装步骤整理如下：
1、首先访问Nvidia官网下载最新的Linux驱动：http://www.nvidia.cn/Download/index.aspx?lang=cn
比如我下载到的文件为NVIDIA-Linux-x86-270.41.06.run。

2、阻止 nouveau 模块的加载：
终端切换到root：`su`
编辑 /etc/modprobe.d/blacklist.conf 文件：
`vi /etc/modprobe.d/blacklist.conf `
在文件末尾添加blacklist nouveau。

3、编辑/boot/grub/grub.conf文件
`vi /boot/grub/grub.conf`
在rhgh quiet 后面添加’nouveau.modeset=0’(没有引号)

4、降低selinux级别，解除selinux阻止加载nvidia驱动模块，运行命令：
`setsebool -P allow_execstack on`

5、重启在grub界面按e键进入，选择带有Kernel的第二项，继续按住e键编辑该项目。在quiet的后面添加 3 这个参数（记得有个空格），然后回车退出。按b引导进入文字终端。

6、进入root，切换到进入放置驱动文件的目录，进行安装：
`sh ./NVIDIA-Linux-x86-270.41.06.run`
至此reboot后可以看到nvidia logo说明安装成功。

p.s.安装nvidia官方驱动后，发现fedora开机画面变成黑屏和蓝带进度条了。

只要安装以下rpm即可：
`yum install bootchart bootconf bootconf-gui`

接着运行bootconf，选择分辨率，之后reboot，就看到原来的开机画面了。比较郁闷的是没有1366*769的分辨率。
