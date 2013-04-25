---
author: labuxiaojue
comments: true
date: 2011-06-19 20:06:19
layout: post
slug: google-toolbar-support-to-firefox-5-0
title: 让旧版Google工具栏临时支持Firefox5.0
wordpress_id: 164
categories:
- Notes
tags:
- Firefox
- Google
---

> 7月22日报道：近日，谷歌宣布停止支持Firefox新版本上的Google工具栏，也就是说，用于Firefox 4及更早版本的Google工具栏，将不再为新版本Firefox 5提供支持。谷歌解释，此举是因为Google工具栏曾为Firefox浏览器提供的多种功能，现在已经内置到浏览器，所以谷歌决定放弃为Firefox提供这个已经没有多少附加价值的工具栏。


Firefox5.0正式版几天前发布了，但是一直让人诟病的不向下兼容旧版插件的机制还是没有改善，升级后发现Google工具栏尚不支持新版的火狐浏览器，<del>谷歌官方还未更新</del>，Google不再更新，如有童鞋还想继续在新版火狐中使用Google工具栏，下文提供一个临时解决办法：
<!-- more -->
打开你的火狐插件安装目录，比如(本机是Win7系统)


> C:用户名WorkAppDataRoamingMozillaFirefoxProfileslot9ytgx.defaultextensions{3112ca9c-de6d-4884-a869-9855de68056c}


以记事本方式编辑**install.rdf**文件，
把


> <em:maxVersion>4.0.*</em:maxVersion>


更改成


> <em:maxVersion>5.0.*</em:maxVersion>


重启火狐即可发现可以运行Google工具栏了，其它插件类似。
不过
`chrome://google-toolbar/content/new-tab.html`
并不能正常显示，期待官方更新。
