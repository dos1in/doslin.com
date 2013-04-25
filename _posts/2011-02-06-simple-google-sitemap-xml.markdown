---
author: labuxiaojue
comments: true
date: 2011-02-06 17:44:49
layout: post
slug: simple-google-sitemap-xml
title: Simple Google Sitemap XML插件的安装
wordpress_id: 6
categories:
- WordPress
tags:
- Wordpress
---

刚换的空间商，所以把以前的文章都清空了。
重新安装wordpress一切顺利,后来安装Simple Google Sitemap XML插件时,提示


> 插件无法被启用因为触发了一个严重错误。

> Warning:fopen(xxx/xxx/.../wpcontent/plugins/simple-google-sitemap-xml/sitemap.xml) [function.fopen]:sitemap-xml/imple-google-sitemap-xml.php on line 34


解决方案如下:到主机后台/wpcontent/plugins/simple-google-sitemap-xml/目录下把simple-google-sitemap-xml.php的权限更改为755即可。

_说明：755权限表示要开放档案给任何人执行，自己可以更改档案，但是不希望其它人更改您的档案_
