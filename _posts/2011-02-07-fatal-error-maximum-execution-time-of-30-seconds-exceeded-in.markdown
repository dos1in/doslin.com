---
author: labuxiaojue
comments: true
date: 2011-02-07 20:18:33
layout: post
slug: fatal-error-maximum-execution-time-of-30-seconds-exceeded-in
title: 'Fatal error: Maximum execution time of 30 seconds exceeded in的解决方法'
wordpress_id: 11
categories:
- WordPress
tags:
- Wordpress
---

重新安装wordpress自然免不了安装一堆必要的插件，结果添加某些插件时在下载页面提示：

> Fatal error: Maximum execution time of 30 seconds exceeded in.....

找到一个简单的解决方法：修改wp-config.php文件：

> 在WordPress配置文件wp-config.php最下边中添加set_time_limit(0); 然后上传覆盖wp-config.php。值得注意的是如果是用记事本修改wp-config.php文件在保存的时候记得保存为ANSI编码。安装完毕后可删除添加的语句。
