---
title: 3分钟从源码构建React Native
date: 2017-03-27 15:27:22
tags:
    - react-native
    - android
categories:
    - react-native
---

## 前言
* 本文假定读者已经安装并配置好了相应的开发环境： Gradle 、 Android SDK 、 Android NDK 及Android Studio等。
* 通过源码编译我们可以进行定制化改造和修正bug：譬如华为7.0上的边缘滑动崩溃问题。
* 阅读正文将花费您大约3分钟。

## 创建 Android 工程

虽然官方已经不再推荐 Android 命令行工具创建项目，我们还是瞄一眼它的命令格式：

``` shell
android create project --target android-22 --name MyRNApp --path /home/doslin/blog/MyRNApp 
--activity MyActivity --package com.doslin.rnapp
```

运行后会提示：
> The android command is no longer available.

> For manual SDK and AVD management, please use Android Studio.

> For command-line tools, use tools/bin/sdkmanager and tools/bin/avdmanager

<!--more-->

所以我们通过 Android Studio 创建一个项目，三步走：

1. 填入应用名及包名
2. 选择最小支持的 Android SDK
3. 选择 No Activity 

![Android Studio 创建项目](/images/2017/03/rn-build-as.png)

## 下载 React Native 源码

`NPM` 安装 `Package` 语法如下：
``` shell
npm install github:<githubname>/<githubrepo>[#<commit-ish>]

-S, --save: Package will appear in your dependencies.
```

你也可以下载最新的主干分支的代码：

```
npm install --save github:facebook/react-native#master
```
之后源码将下载到用户目录下的 node_modules/react-native 下。

你也可以下载指定版本的 React Native，假设我们现在要下载最新的 	
v0.43.0-rc.4， 它在指定的 tag 分支上，进入[https://github.com/facebook/react-native/tags](https://github.com/facebook/react-native/tags)下载即可。

## 导入 ReactAndroid Module


在刚刚创建的项目中进行操作，首先在项目级别的 `build.gradle` 中添加 `gradle-download-task` 依赖：

``` gradle
...
    buildscript {
        ...
        dependencies {
            // ↓注意这里的插件版本更改为2.2.3
            classpath 'com.android.tools.build:gradle:2.2.3'
            classpath 'de.undercouch:gradle-download-task:3.1.2'
            // NOTE: Do not place your application dependencies here; they belong
            // in the individual module build.gradle files
        }
    }
...
```

注意将 gradle 插件版本更改为 2.2.3，否则使用最新的 2.3.0会提示如下错误：

> Error:(168, 0) No such property: sdkHandler for class: com.android.build.gradle.LibraryPlugin

之后导入 React Native 源码目录下的 ReactAndroid 目录。

![Android Studio 导入模块](/images/2017/03/rn-build-module.png)

## 导入 ReactCommon

由于 ReactAndroid 也依赖了 ReactCommon，所以还需要将 React Native 源码目录下的 ReactCommon 目录复制到项目根目录下。如果你要已到其他路径或不进行移动，请更改 ReactAndroid 下的 `build.gradle`， 搜索 ReactCommon 关键字，将两处中的相应路径更改为相应目录。

``` gradle
task buildReactNdkLib(dependsOn: [prepareJSC, prepareBoost, prepareDoubleConversion, prepareFolly, prepareGlog], type: Exec) {
    inputs.file('src/main/jni/xreact')
    outputs.dir("$buildDir/react-ndk/all")
    commandLine getNdkBuildFullPath(),
            'NDK_PROJECT_PATH=null',
            "NDK_APPLICATION_MK=$projectDir/src/main/jni/Application.mk",
            'NDK_OUT=' + temporaryDir,
            "NDK_LIBS_OUT=$buildDir/react-ndk/all",
            "THIRD_PARTY_NDK_DIR=$buildDir/third-party-ndk",
            // ↓更改为指定路径下的ReactCommon
            "REACT_COMMON_DIR=$projectDir/../ReactCommon",
            '-C', file('src/main/jni/react/jni').absolutePath,
            '--jobs', project.hasProperty("jobs") ? project.property("jobs") : Runtime.runtime.availableProcessors()
}

task cleanReactNdkLib(type: Exec) {
    commandLine getNdkBuildFullPath(),
            "NDK_APPLICATION_MK=$projectDir/src/main/jni/Application.mk",
            "THIRD_PARTY_NDK_DIR=$buildDir/third-party-ndk",
            // ↓更改为指定路径下的ReactCommon
            "REACT_COMMON_DIR=$projectDir/../ReactCommon",
            '-C', file('src/main/jni/react/jni').absolutePath,
            'clean'
}
```

## 构建 React Native

更改相应的 Build Variant 后点击 build 按钮，即可在 ReactAndroid 中的 build 下得到相应的 React Native AAR，之后你可以将它发布到自己的 Maven 仓库中。

![Android Studio 构建](/images/2017/03/rn-build-var.png)

Tips：如果发现编译过慢，可能是一些第三方依赖下载过慢，可以去 ReactAndroid 下的 build.gradle 中搜索 https 替换为 http 即可。

## 参考资料

1.[Android 开发者文档](https://developer.android.com/studio/tools/help/android.html)
2.[NPM 官方文档](https://docs.npmjs.com/cli/install)
3.[React Native 官方文档](http://facebook.github.io/react-native/releases/0.43/docs/android-building-from-source.html)
