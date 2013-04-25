---
author: labuxiaojue
comments: true
date: 2011-05-26 10:31:21
layout: post
slug: fedora-witn-virtualbox-etcinit-dvboxdrv-setup
title: 解决Fedora安装Virtualbox后无法运行：/etc/init.d/vboxdrv setup
wordpress_id: 113
categories:
- Linux
tags:
- Fedora
- Linux
- Virtualbox
---

在Fedora下安装了Virtualbox，发现运行时出现以下问题：


> Kernel driver not installed (rc=-1908)

The VirtualBox Linux kernel driver (vboxdrv) is either not loaded or there is a permission problem with /dev/vboxdrv. Please reinstall the kernel module by executing

'/etc/init.d/vboxdrv setup'

as root. Users of Ubuntu, Fedora or Mandriva should install the DKMS package first. This package keeps track of Linux kernel changes and recompiles the vboxdrv kernel module if necessary.


<!-- more -->
根据提示运行以root身份运行`/etc/init.d/vboxdrv setup`
结果提示：


> Stopping VirtualBox kernel modules                         [确定]
Uninstalling old VirtualBox DKMS kernel modules            [确定]
Trying to register the VirtualBox kernel modules using DKMSError! Bad return status for module build on kernel: 2.6.38.6-27.fc15.i686 (i686)
Consult /var/lib/dkms/vboxhost/4.0.8/build/make.log for more information.
[失败]
(Failed, trying without DKMS)
Recompiling VirtualBox kernel modules                      [失败]
(Look at /var/log/vbox-install.log to find out what went wrong)


应该是编译问题，故安装gcc：`yum install gcc`
提示：


> 已加载插件：langpacks, presto, refresh-packagekit
http://download.virtualbox.org/virtualbox/rpm/fedora/15/i386/repodata/repomd.xml: [Errno 14] HTTP Error 404 - Not Found : http://download.virtualbox.org/virtualbox/rpm/fedora/15/i386/repodata/repomd.xml
尝试其他镜像。
http://download.virtualbox.org/virtualbox/rpm/fedora/15/i386/repodata/repomd.xml: [Errno 14] HTTP Error 404 - Not Found : http://download.virtualbox.org/virtualbox/rpm/fedora/15/i386/repodata/repomd.xml
尝试其他镜像。
错误：Cannot retrieve repository metadata (repomd.xml) for repository: virtualbox. Please verify its path and try again


源出问题？想起刚安装Virtualbox时，从官网下载到virtualbox.repo，于是切换到/yum.repos.d，删除virtualbox.repo。

之后成功运行`yum install gcc，`




> 已加载插件：fastestmirror, langpacks, presto, refresh-packagekit
Determining fastest mirrors
 * fedora: ftp.cuhk.edu.hk
 * updates: ubuntu.cn99.com





接着运行`/etc/init.d/vboxdrv setup`




> Stopping VirtualBox kernel modules                         [确定]
Uninstalling old VirtualBox DKMS kernel modules            [确定]
Trying to register the VirtualBox kernel modules using DKMS[确定]
Starting VirtualBox kernel modules                         [确定]



问题解决，能正常打开虚拟机啦。
