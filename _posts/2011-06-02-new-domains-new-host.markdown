---
author: labuxiaojue
comments: true
date: 2011-06-02 22:08:11
layout: post
slug: new-domains-new-host
title: 新域名，新主机
wordpress_id: 123
categories:
- Notes
tags:
- Wordpress
---

今天折腾了大半天，终于把小站转到新主机下。

主要耗费的时间在于原主机服务商坑爹的数据库后台，根本无法登陆。_付钱之前用户是大爷，付费之后用户成孙子。_

只好用Wordpress后台的导出功能来保存数据，但导出时遇到点问题，就是无法下载xml文件，浏览器直接在新标签打开了此文件。

看错误信息发现是WP-CodeBox插件的文件，禁用后即可正常导出。

把包含原域名_www.doslin.tk_关键字都替换成了目前的域名。

转移完wp-content目录下的主题文件和插件目录，总算完成了博客的迁移。
