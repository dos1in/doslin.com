---
author: labuxiaojue
comments: true
date: 2013-03-28 10:36:12
layout: post
slug: make-ie-capable-of-css3-features
title: IE6+加入CSS3支持
wordpress_id: 265
categories:
- 前端
tags:
- CSS3
- IE
---

IE6~8作为老去的一代浏览器在国内还占有较高的[市场份额](http://tongji.baidu.com/data/browser)。为了使其支持一些次世代浏览器的特性，譬如CSS3的效果，我们需要通过一些hack来实现。

下面我们借助PIE.htc使得IE系列能展现CSS3的一些常见效果：如阴影、渐变等。
<!-- more -->
**使用方法：**
获取[PIE.htc](http://css3pie.com/)，在你的样式文件中引入该文件就可以了，如下

    
    border: 1px solid #696;
    padding: 60px 0;
    text-align: center; width: 200px;
    -webkit-border-radius: 8px;
    -moz-border-radius: 8px;
    border-radius: 8px;
    -webkit-box-shadow: #666 0px 2px 3px;
    -moz-box-shadow: #666 0px 2px 3px;
    box-shadow: #666 0px 2px 3px;
    background: #EEFF99;
    background: -webkit-gradient(linear, 0 0, 0 bottom, from(#EEFF99), to(#66EE33));
    background: -webkit-linear-gradient(#EEFF99, #66EE33);
    background: -moz-linear-gradient(#EEFF99, #66EE33);
    background: -ms-linear-gradient(#EEFF99, #66EE33);
    background: -o-linear-gradient(#EEFF99, #66EE33);
    background: linear-gradient(#EEFF99, #66EE33);
    -pie-background: linear-gradient(#EEFF99, #66EE33);
    behavior: url(/pie/PIE.htc);


**注意点：**



	
  * 这里behavior的url路径，是相对于HTML页面所在位置而言的，而不是样式文件所在的位置。

	
  * 要想让IE浏览器识别htc扩展名，我们需要返回“text/x-component”类型的Content-Type，对于使用tomcat的童鞋，我们只要在web.xml中加入


> `<mime-mapping>
<extension>htc<extension>
<mime-type>text/x-component<mime-type>
<mime-mapping>
`


即可。
  * 用于该文件是将借助在ie中绘制vml来模拟css3效果的，所以元素的大小可能会较原生的显示效果有差，需要重新调整宽高。


