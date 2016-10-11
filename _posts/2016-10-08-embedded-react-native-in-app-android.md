---
date: 2016-10-08 14:57:22
layout: post
title: "弹射起步：Android原生项目集成React Native模块"
tags:
    - react-native android
categories:
    - react-native
---

------
## 说明和前提

1、由于通过命令`react-native init`初始化的**React Native**（以下简称RN）项目结构与原生项目有区别，所以不能单纯直接在已有项目的根目录下运行该命令。

2、RN需要Android4.1或以上的环境，故集成RN的Android项目的`minSdkVersion`需设置为**API16**或以上。

3、本机安装NPM支持：
新版的RN依赖都将发布在**NPM**（包管理工具，如同Maven仓库）中，故需安装NPM。

4、本文安装环境为**MAC**，Windows或Linux上的命令差异请参考[官方文档](http://facebook.github.io/react-native/docs/getting-started.html)。

## 集成步骤
### 1. 安装NPM

之后于同时需要NodeJS服务器，故只需安装NodeJS，NPM会随同安装。

```
brew install node
brew install watchman
```

### 2. 添加NPM依赖

进入Android项目根目录，新建文件`package.json`，内容如下：

```
{
  "name": "react-native-module",
  "version": "0.0.1",
  "private": true,
  "scripts": {
    "start": "node node_modules/react-native/local-cli/cli.js start"
  },
  "dependencies": {
    "react": "15.3.2",
    "react-native": "0.34.1"
  }
}
```

之后拉取NPM组件，在`package.json`同目录位置下运行
```
npm install
```
完成后将出现`node_modules`文件夹。

### 3.添加页面文件

在根目录下，新建文件`index.android.js`，内容如下：

```
import React, { Component } from 'react';
import {
  AppRegistry,
  StyleSheet,
  Text,
  View
} from 'react-native';

class RN_Demo extends Component {
  render() {
    return (
      <View style={styles.container}>
        <Text style={styles.welcome}>
          Welcome to React Native!
        </Text>
        <Text style={styles.instructions}>
          To get started, edit index.android.js
        </Text>
        <Text style={styles.instructions}>
          Double tap R on your keyboard to reload,{'\n'}
          Shake or press menu button for dev menu
        </Text>
      </View>
    );
  }
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#F5FCFF',
  },
  welcome: {
    fontSize: 20,
    textAlign: 'center',
    margin: 10,
  },
  instructions: {
    textAlign: 'center',
    color: '#333333',
    marginBottom: 5,
  },
});

AppRegistry.registerComponent('RN_Demo', () => RN_Demo);
```

记住上面的组件名**RN_Demo**。

### 4.添加RN依赖

编辑`app`目录下的`build.gradle`，添加

```
compile 'com.facebook.react:react-native:0.34.1'  // From node_modules
```

之所以此处不根据官方文档的配置（compile 'com.facebook.react:react-native:+'），你会发现通过**Gradle Sync**进行同步后会报错，所以需要在项目下的`build.gradle`中添加

```
allprojects {
    repositories {
        jcenter()
        maven {
            // All of React Native (JS, Android binaries) is installed from npm
            url "$projectDir/../node_modules/react-native/android"
        }
    }
}
```

否则未写明版本的RN依赖可能是**jcenter**中的旧版本。

### 5.新建Activity与Application

新建一个继承自`ReactActivity`的`activity`，内容如下：

```
import com.facebook.react.ReactActivity;

public class MainActivity extends ReactActivity {

    @Override
    protected String getMainComponentName() {
        return "RN_Demo";
    }
}
```

注意此处实现的`getMainComponentName`方法返回的字符串与`index.android.js`中注册的`Component`名的要一致。

之后编辑自定义`Application`，实现`ReactApplication`接口，内容如下：

```
public class MainApplication extends Application implements ReactApplication {

    private final ReactNativeHost mReactNativeHost = new ReactNativeHost(this) {
        @Override
        // 表示是否启用开发者模式
        protected boolean getUseDeveloperSupport() {
            return BuildConfig.DEBUG;
        }

        @Override
        // 模块列表，可添加原生Java模块
        protected List<ReactPackage> getPackages() {
            return Arrays.<ReactPackage>asList(
                    new MainReactPackage()
            );
        }
    };

    @Override
    public ReactNativeHost getReactNativeHost() {
        return mReactNativeHost;
    }
}
```

### 6.编辑`AndroidManifest.xml`

添加应用的网络访问权限，以及刚刚新建的`Activity`及`Application`：

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.doslin.rndemo">

    <uses-permission android:name="android.permission.INTERNET" />

    <application
        android:name=".MainApplication"
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:supportsRtl="false"
        android:theme="@style/AppTheme">
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

        <activity android:name="com.facebook.react.devsupport.DevSettingsActivity" />
    </application>

</manifest>
```

### 7.添加NDK支持

在`gradle.properties`中添加如下内容：

```
android.useDeprecatedNdk=true
```

在**app**下的`build.gradle`中添加如下内容：

```
ndk {
    abiFilters 'armeabi', 'armeabi-v7a'
}
```

如果你的app需要在x86下运行，请额外进行配置`x86`。

### 8.启动**Packager Server**

在`package.json`同级目录下运行
```react-native start```
之后为了防止出现`error “Could not get BatchedBridge, make sure your bundle is packaged properly” on start of app`的异常，先运行

```
curl "http://localhost:8081/index.android.bundle?platform=android" -o "./app/src/main/assets/index.android.bundle"
```

将在**assets**目录下下载一份`index.android.bundle`文件。

如果你是用的手机而非模拟器测试，还需要运行如下命令设置反向代理：

```
adb reverse tcp:8081 tcp:8081
```

最后执行**AS**的绿色**RUN**按钮等待**apk**上传并运行。
在主界面摇晃手机会出现调试界面，注意开启**显示悬浮窗**权限。

------
## 参考链接

[1] [http://facebook.github.io/react-native/docs/native-components-android.html](http://facebook.github.io/react-native/docs/native-components-android.html)

[2] [http://gold.xitu.io/entry/57a0d03ed342d300572e9ffc](http://gold.xitu.io/entry/57a0d03ed342d300572e9ffc)

[3] [http://www.imbeta.cn/react-native-for-android-huan-jing-da-jian-ji-cai-keng.html](http://www.imbeta.cn/react-native-for-android-huan-jing-da-jian-ji-cai-keng.html)

[4] [http://acgtofe.com/posts/2016/06/react-native-embedding-android](http://acgtofe.com/posts/2016/06/react-native-embedding-android)

[5] [http://stackoverflow.com/questions/38870710/error-could-not-get-batchedbridge-make-sure-your-bundle-is-packaged-properly](http://stackoverflow.com/questions/38870710/error-could-not-get-batchedbridge-make-sure-your-bundle-is-packaged-properly)

## 相关资料

1.[React-Native学习指南](https://github.com/reactnativecn/react-native-guide)

2.[react-native-android-guide](https://github.com/xujinyang/react-native-android-guide#react-native-android-guide)
