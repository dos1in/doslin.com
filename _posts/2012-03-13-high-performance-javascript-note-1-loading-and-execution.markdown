---
author: labuxiaojue
date: 2012-03-13 22:26:30
layout: post
slug: high-performance-javascript-note-1-loading-and-execution
title: 《高性能JavaScript》读书笔记[1]-加载与执行
categories:
- JavaScript
tags:
- JavaScript
---

>###**优化的必要性**

目前的应用比早期Web应用（05年以前）相比更庞大复杂，JavaScript代码量指数级增长。IE6引擎的“静态垃圾回收机制”难以应付JS代码产生的越来越多的对象。
因为JS作为解释性代码以及解释器很少有优化，往往意味着代码怎么写，就被怎么执行。相较于编译性的代码由编译处理器优化，在JS中通常需要我们开发人员自己完成优化工作。
  
>###**JS的特点-阻塞**

无论JS代码是内嵌（以`<script>`标签包含）还是从外部文件引入（以`<script>`标签的src属性引入），该标签一出现就会让页面的下载和渲染暂时停止，等待该脚本解析和执行完成才能继续解析和渲染页面。
  
优化方案
-------
>###**将脚本放在页面底部**

推荐将所有的`<script>`标签尽可能放到`<body>`标签底部，来减少对整个页面下载的影响。
浏览器为了确保内嵌脚本在执行时能获得最精准的样式信息，所以会出现如果内嵌脚本处于引用外链样式表的`<link>`标签之后会导致页面阻塞去等待样式表下载完成，所以不要把内嵌脚本跟在`<link>`标签后。
  
>###**减少<scirpt>标签数量**

因为HTTP请求的额外开销，下载单个100KB的文件比下载4个25KB的文件更块！请试着合并JS文件。
因为上文提到不管内嵌或外链JS，`<script>`标签元素都会阻塞浏览器，所以内嵌`<script>`标签数量同样也要限制。

>###**使用无阻塞脚本（页面加载结束后才去加载执行JS）**

合并JS文件可能使得单个JS变大，虽然只产生一个HTTP请求，但会锁死浏览器一大段时间，为了避免此情形，需要采用无阻塞脚本方案逐步加载JS文件。无阻塞即window对象的onload事件触发后才下载脚本。

>###**延迟脚本**

使用 `<script>` 的 defer 属性(仅适用于IE和Firefox3.5+)，该属性指明本元素所含的脚本不会修改DOM，因此代码能安全地延迟执行；

    <script defer="defer" type="text/javascript">
        alert("defer");
    </script>
    <script type="text/javascript">
        alert("script");
    </script>
    <script type="text/javascript">
        windows.onload=function(){
        alert("load");
    };
    </script>

* 动态脚本元素

使用动态创建的 `<script>` 元素来下载并执行代码；

    function loadScript(url, callback){
        var script = document.createElement("script");
        script.type = "text/javascript";
        if (script.readyState) { // IE
            script.onreadystatechange = function() {
                if (/loaded|complete/.test(script.readyState)) {
                    script.onreadystatechange = null; // 避免事件执行两次
                    callback();
                }
            };
        } else {// 其它浏览器
            script.onload = function() {
            callback();
        };
    }
    
    script.src = url;
    document.getElementsByTagName("head")[0].appendChild(script);
    }

* XMLHttpRequest脚本注入

使用 XMLHttpRequest 对象下载js代码并注入到页面；
    
    var xhr = new XMLHttpRequest();
    xhr.open("get","file1.js",true);
    xhr.onreadystatechange = function(){
        if(xhr.readyState == 4){
        //2XX表示有响应，304意味着从缓存读取
            if(xhr.status >= 200 && xhr.status < 300 || xhr.status == 304){
                var script = document.createElement("script");
                script.type = "text/javascript";
                script.text = xhr.responseText;
                document.body.appendChild(script);
    
            }
        }
    };
    
    xhr.send(null);
    
这种方法的主要优点是可以下载JS代码但不会立即执行。由于代码时在`<script>`标签之外返回的，我们可以把脚本的执行推迟到准备好的时候而且它可跨浏览器。局限性在于它不能跨域执行。