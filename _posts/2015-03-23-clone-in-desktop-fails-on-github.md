---
date: 2015-03-23 9:37:15
layout: post
title: "解决Clone in Desktop失败的问题"
tags:
    - github
categories:
    - github
---

有时候会出现github客户端在Mac下点击`Clone in Desktop`时会带你去GitHub客户端的下载页面。

根据官方写明的[调用原理](https://help.github.com/articles/github-conduit/)，只要确保
在浏览器中访问[https://ghconduit.com:25035/status](https://ghconduit.com:25035/status)能正常返回
`{"capabilities":["status","unique_id","url-parameter-filepath"],"running":false,"server_version":"5"}`
类似的信息即可。

如果访问失败，可以通过以下几种形式定位问题：

* 进程清单中是否有`Github Conduit`；
* 浏览器是否开了代理；
* 尝试在`/etc/hosts`中加入`127.0.0.1 ghconduit.com`后重新访问。

当链接正常时刷新页面重新点击`Clone`就能生效了。
