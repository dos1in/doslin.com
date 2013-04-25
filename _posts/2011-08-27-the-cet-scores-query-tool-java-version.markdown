---
author: labuxiaojue
comments: true
date: 2011-08-27 10:15:10
layout: post
slug: the-cet-scores-query-tool-java-version
title: CET成绩查询工具Java版
wordpress_id: 181
categories:
- Java
tags:
- Java
---

前段时间公布了CET的成绩，查了一下，从去年的强迫观看30秒广告缩短到了15秒，谎称正在查询的过程实在令人不齿。

抽空用Java写了一个查询CET成绩的小工具，因为向服务器发送的请求都是一样的，也可以查询日语等其它科目成绩。而今年的验证措施是准考证加姓名，所以批量查询整个考场的成绩暂时无法实现，但查单人成绩可以略去那15秒。可能以后能完成批量查询的功能，所以程序界面底部预留了一点空间——看起来有多行的表格。

伪造查询请求的核心代码主要用到HttpURLConnection类，实现Post很方便，以下是程序运行截图：

[![](/assets/post_img/the-cet-scores-query-tool-java-version/cet.jpg)](/assets/post_img/the-cet-scores-query-tool-java-version/cet.jpg)

下载地址:[http://u.115.com/file/dnhrxjbn](http://u.115.com/file/dnhrxjbn)
因为加了仿mac风格的界面，所以程序比较大，请见谅。

原本想连同jre一起打包封装，但是手动精简后体积多大40M，不甚理想，故放弃。

程序启动时会检测运行环境，没有安装jre的童鞋程序会引导你去下载页下载安装，安装成功后即可正常运行。
