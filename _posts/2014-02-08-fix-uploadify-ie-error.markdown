---
date: 2014-02-08 14:26:55
title: 修正IE6下刷新页面时Uploadify引起的报错问题
layout: post
tags:
    - uploadify
    - IE
categories:
    - JavaScript
---

[Uploadify](http://www.uploadify.com/)作为一个比较常用的页面上传控件，最近也是在上面遇到了一个问题。即使用了该控件的页面在IE6下刷新会出现下图错误：

[![](/assets/post_img/fix-uploadify-ie-error/ie6-err.png)](/assets/post_img/fix-uploadify-ie-error/ie6-err.png)

调查结果是因为IE6下该控件的Object未能销毁的缘故引起的页面报错`document.getElementById("SWFUpload_0").SetReturnValue("<undefined/>");`和内存飙高。所以解决办法就是可以在每次刷新页面前进行对象释放。由于dwz也使用了该控件，我们可以在每次进行标签操作时进行该动作，在`dwz.navTab.js`文件的`_switchTab`函数加入如下语句：

	_switchTab: function(iTabIndex){
		for(var i=SWFUpload.movieCount; i>=0; i--) { // 手动释放SWFUpload对象，修正IE6下报错问题
			if(SWFUpload.instances["SWFUpload_"+i]){
				SWFUpload.instances["SWFUpload_"+i].destroy();
			}
		}
		// ... 
	},

**_参考资料:_**  
_(1)[http://blog.csdn.net/zhichao2001/article/details/9468023](http://blog.csdn.net/zhichao2001/article/details/9468023)_  
_(2)[http://my.opera.com/justnewbee/blog/fix-swfupload-destroy-ie-js-error](http://my.opera.com/justnewbee/blog/fix-swfupload-destroy-ie-js-error)_