---
date: 2013-05-17 12:48:49
title: 用zTree替换DWZ(j-UI)中的jTree
layout: post
tags:
    - JavaScript
    - DWZ
    - zTree
categories:
    - JavaScript
---

组件版本
-------

*  DWZ: [v1.4.5](https://code.google.com/p/jquerytree/downloads/list)

*  zTree: [v3.5.12](https://code.google.com/p/jquerytree/downloads/list)

替换步骤
-------
>###**样式文件替换**

定位dwz的themes/css下的`core.css`文件，在184行~216行中注释JTree样式。

提取zTree的样式文件`zTreeStyle.css`（位于css/zTreeStyle）及图标资源img文件夹。前者代码置于dwz的`core.css`的底部以防止dwz预定义的样式破坏zTree的样式。同时将img文件夹与`core.css`放在同一个文件夹下（themes/css/）。

>###**脚本文件替换与修改**

找到DWZ的js文件夹下的`dwz.tree.js`文件，用zTree的js文件`jquery.ztree.all-3.5.js`中的内容替换（如果不需要zTree的excheck + exedit扩展，可以使用`jquery.ztree.core-3.5.js`）。

脚本内容替换完成后，需要修改`dwz.ui.js`与`dwz.tree.js`（现在它的脚本语句是zTree的）：

*  dwz.ui.js

注释掉77行的调用jTree语句。

    $("ul.tree", $p).jTree();


*  dwz.tree.js

1、树节点添加DWZ中的rel属性（添加DWZ中的external属性同理）：

找到`makeDOMNodeNameBefore`方法（当前版本位于1112行），在html.push方法后添加：

	if (node.rel) {
		html.push("' rel='", node.rel);
	}

2、更改节点的默认Target为`navTab`：

找到`makeNodeTarget`方法（当前版本位于1178行），return语句修改为`return (node.target || "navTab");`

3、修改节点属性更新方法`updateNode`（当前版本位于1639行），追加`view.setNodeRel(this.setting, node)`。

setNodeRel方法定义如下：

	setNodeRel: function(setting, node) {
			var aObj = $("#" + node.tId + consts.id.A),
			rel = treeNode.rel;
			if (rel == null || rel.length == 0) {
				aObj.removeAttr("rel");
			} else {
				aObj.attr("rel", rel);
			}
		}

## 调用方式
原先的jTree调用方式

	<ul class="tree treeFolder">
		<li>
			<a href="tabsPage.html" target="navTab">主框架面板</a>
			<ul>
				<li><a href="main.html" target="navTab" rel="main">我的主页</a></li>
				<li><a href="http://www.baidu.com" target="navTab" rel="page1">页面一(外部页面)</a></li>
				<li><a href="demo_page2.html" target="navTab" rel="external" external="true">iframe navTab页面</a></li>
			</ul>
		</li>
	</ul>

现在的zTree调用方式：
将上述树DOM内容更改为

	<div class="zTreeDemoBackground left">
		<ul id="treeDemo" class="ztree"></ul>
	</div>

添加如下调用脚本：

	var setting = {
			data: {
				simpleData: {
					enable: true
				}
			}
	};

	var zNodes =[
			{ id:1, pId:0, name:"主框架面板", url:"tabsPage.html", target:"navTab", open:true},
			{ id:2, pId:1, name:"我的主页", url:"main.html", target:"navTab", rel:"main"},
			{ id:3, pId:1, name:"iframe navTab页面", url:"demo_page2.html", target:"navTab", rel:"external", external:"true"}
		];

		$(document).ready(function(){
			$.fn.zTree.init($("#treeDemo"), setting, zNodes);
		});

_注意：原先超链接中href属性名要更改为url。_
