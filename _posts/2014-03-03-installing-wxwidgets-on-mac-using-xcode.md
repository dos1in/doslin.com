---
date: 2014-03-03 16:17:10
layout: post
title: "xCode引入wxWidgets库"
tags:
    - WxWidgets
    - Xcode
categories:
    - IDE
---

安装Homebrew
------------
安装wxWidgets之前，推荐使用[Homebrew](http://brew.sh/index_zh-cn.html)管理软件包（当然，如果你熟悉[MacPorts](http://www.macports.org/)的话，也可以用它安装wxWidgets）。

在终端中键入以下命令：

	ruby -e "$(curl -fsSL https://raw.github.com/Homebrew/homebrew/go/install)"
	
安装wxWidgets
-------------
搞定Homebrew，现在安装wxWidgets库了。
输入：

	brew install wxwidgets
	//或者brew install wxmac

Homebrew将会自动为你下载好os x环境下的wxWidgets，免去编译源码的步骤。安装路径为`/usr/local/Cellar`。

接着输入：

	wx-config --version
	
终端输出`3.0.0`说明安装成功。

配置xCode
---------
我们下面要配置xCode中项目的`Build Settings`。

>###**配置Linking**

在终端中输入：

	wx-config --libs

返回：

	-L/usr/local/Cellar/wxmac/3.0.0.0/lib   -framework IOKit -framework Carbon -framework Cocoa -framework AudioToolbox -framework System -framework OpenGL -framework QuickTime -lwx_osx_cocoau-3.0 

将上面内容填入`Linking`下的`Other Linker Flags`。

>###**配置Custom Compiler Flags**

在终端中输入：

	wx-config --cxxflags

返回：
	
	-I/usr/local/Cellar/wxmac/3.0.0.0/lib/wx/include/osx_cocoa-unicode-3.0 -I/usr/local/Cellar/wxmac/3.0.0.0/include/wx-3.0 -D_FILE_OFFSET_BITS=64 -DwxDEBUG_LEVEL=0 -DWXUSINGDLL -D__WXMAC__ -D__WXOSX__ -D__WXOSX_COCOA__ 
	
如果输入命令后，终端没有上述内容，请输入`cd`切换到主目录，再输入上述命令。

将输入内容填入`Custom Compiler Flags`下的`Other C++ Flags`。

至此，你可以引入`#inlcude <wx/wx.h>`，编写你的wxWidgets程序了！
