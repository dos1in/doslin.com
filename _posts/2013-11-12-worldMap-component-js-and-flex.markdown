---
date: 2013-11-12 17:18:20
title: WorldMap-JS的Flash版
layout: post
tags:
    - WorldMap
    - JavaScript
    - Flex
    - Flash
categories:
    - component
---

去年写了一个地图组件[worldMap-js](https://github.com/JChord/worldMap-js) ，因为用JS实现的，众所周知，IE6-8对JS的执行效率实在让人捉急，而国内浏览器中IE占用率又是个大头，所以萌生了用Flash重现实现这个组件的念头，使用Flex编写，目前还不完善，大体实现了地图的绘制和提示信息的展示，绘制下级地图的功能还没完成，同时MapData目前只有中国地图。

以下是Demo演示：

<script type="text/javascript" src="../assets/demo/worldMap-flex/swfobject.js"></script>
<script type="text/javascript">
// For version detection, set to min. required Flash Player version, or 0 (or 0.0.0), for no version detection. 
var swfVersionStr = "11.4.0";
// To use express install, set to playerProductInstall.swf, otherwise the empty string. 
var xiSwfUrlStr = "../assets/demo/worldMap-flex/playerProductInstall.swf";
var flashvars = {};
var params = {};
params.quality = "high";
params.bgcolor = "#ffffff";
params.allowscriptaccess = "sameDomain";
params.allowfullscreen = "true";
var attributes = {};
attributes.id = "FlashMap";
attributes.name = "FlashMap";
attributes.align = "middle";
swfobject.embedSWF(
"../assets/demo/worldMap-flex/FlashMap.swf", "flashContent", 
"560", "470", 
swfVersionStr, xiSwfUrlStr, 
flashvars, params, attributes);
// JavaScript enabled so display the flashContent div in case it is not replaced with a swf object.
swfobject.createCSS("#flashContent", "display:block;text-align:left;");
</script>
<div id="flashContent">
<p>
To view this page ensure that Adobe Flash Player version 
11.4.0 or greater is installed. 
</p>
<script type="text/javascript"> 
var pageHost = ((document.location.protocol == "https:") ? "https://" : "http://"); 
document.write("<a href='http://www.adobe.com/go/getflashplayer'><img src='" 
+ pageHost + "www.adobe.com/images/shared/download_buttons/get_flash_player.gif' alt='Get Adobe Flash player' /></a>" ); 
</script> 
</div>

你可以在这里找到源码：[worldMap-flex](https://github.com/JChord/worldMap-flex)

