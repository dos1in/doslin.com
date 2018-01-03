---
title: JNI与NDK入门之四
date: 2018-01-03 17:19:10
tags:
    - jni
    - ndk
    - android
categories:
    - android
---

# 概述

通过上述三篇文章的学习，我们初步掌握了 JNI 的编程方式。
下面重新复习一下 Java 代码调用本地函数的步骤，如下：

> 将需要本地实现的 Java 方法加上 native 声明；
使用 javac 命令编译 Java 类；
使用 javah 命令生成头文件；
在本地代码中实现头文件中声明的 native 方法；
编译本地代码，生成动态链接库
在 Java 类中加载动态链接库病调用 native 方法。

前面我们都通过 gcc 来生成动态链接库，可以看见生成步骤比较复杂，而在 Android 中我们就要借助 NDK 的帮助，Android NDK 是 Google 提供的开发工具集，通过它能让我们在 Android 中开发出具有 JNI 机制的应用。而新版的 NDK 使用了 `Gradle` 与 `CMake` 来帮助我们简化了一些步骤，比如手动生成头文件。

<!--more-->

NDK 提供了如下工具集与功能：

1. 包含将 C/C++ 源代码编译成本地库的工具（编译器、连接器等）
2. 提供将编译好的本地库插入到 Android 包文件（.apk）中的功能
3. 在生成本地库时，Android 平台可支持的系统头文件与库
4. NDK 开发相关的文档、示例、规范

常见的系统头文件与库如下：

1. libc(C library) headers
2. libm(math library) headers
3. JNI interface headers
4. libz(Zlib compression) headers
5. liblog(Android logging) header
6. OpenGL ES1.1 and OpenGL ES 2.0(3D graphics library) headers
7. libjnigraphics(Pixel buffer access) header(for Android 2.2 and above)
8. A Minimal set of headers for C++ support

如下图所示，Android NDK 先编译 C/C++ 本地代码，生成本地库后，将其插入到 Android 应用程序包中，但调用 JNI 编写本地代码还是由开发者来实现。

![NDK的角色](/images/2018/01/jni-ex4_1.png)

# 开发流程
下面我们将会编写一个使用了 NDK 的 Android 应用程序。整个程序大致分为两个部分：一部分是使用 Java 代码编写的 Android 应用，另一部分是使用 C 语言编写的求两数之和的蹦迪库。Java 代码将会调用本地方法 `add()` 求和，之后将结果输出到 `TextView` 中。
`libndk-sum.so` 共享库中来实现 `add()` 的具体逻辑，它由 `first.c` 与
`second.c` 两个源文件生成。而 `add()` 本地方法将会通过 JNI 与`second.c` 文件中的 `Java_com_doslin_ndksum_MainActivity_add()` 函数映射起来，如下图：

![调用关系](/images/2018/01/jni-ex4_2.png)

## 下载并配置 NDK
登录[官网](https://developer.android.com/ndk/downloads/index.html)下载最新版本的 NDK 并将其解压到某个目录，之后将这个目录地址配置到系统环境变量中。

Mac 中的配置方式如下：
修改 `.bashrc` 文件，追加 `export PATH=<NDK_HOME>:$PATH` ，其中`<NDK_HOME>`请更改为刚解压的NDK目录。添加完毕后执行 `source ~/.bashrc` 后运行 `ndk-build` 测试是否配置成功。

## 建立一个 Android 项目，并声明 native 方法
编辑 `MainActivity.java` 文件，内容如下：

```java
package com.doslin.ndksum;

import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.widget.TextView;

public class MainActivity extends AppCompatActivity {

    // Used to load the 'ndk-sum' library on application startup.
    static {
        // 在 CMakeLists.txt 中指定生成的 so 库名称
        System.loadLibrary("ndk-sum");
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        TextView tv = findViewById(R.id.sample_text);
        tv.setText(add(2, 3));
    }

    // 本地方法声明
    public native String add(int x, int y);
}
```

Java 代码编写完成后，下面开始编写 `second.c` 文件，该文件包含本地方法 `add()` 的具体实现。

## 创建 JNI 源码目录
在 app 文件夹下新建一个 JNI 目录，创建成功后会出现在 main 目录下。将刚才生成的头文件移到该目录中。

![菜单入口](/images/2018/01/jni-ex4_3.png)


### 编写 second.c 文件
在刚才创建的 JNI 目录下创建 `second.c` 文件，实现调用名称为 `sum()` 函数的逻辑，具体代码如下：

```c
#include <stdio.h>
#include <jni.h>
#include "first.h"

jstring JNICALL Java_com_doslin_ndksum_MainActivity_add
  (JNIEnv *env, jobject this, jint x, jint y) {
    char char_arr [100];
    int res = sum(x,y);
    sprintf(char_arr, "%d", res);
    return (*env)->NewStringUTF(env, char_arr);
}
```

上述代码未实现求和运算，真正的实现在 `first.c` 的 `sum()`函数中实现。

### 编写 first.h 与 first.c

```c
// first.h

#ifndef FIRST_H
#define FIRST_H

extern int sum(int x, int y);

#endif /* FIRST_H */```

```c
// first.c

#include "first.h"

int sum(int x, int y) {
    return x + y;
}
```

## 创建 CMakeLists.txt 文件
在项目根目录下创建脚本文件 `CMakeLists.txt`，它配置了 NDK 编译系统所需的各种信息（源代码的位置，本地库的名称等）。内容如下：

```
# For more information about using CMake with Android Studio, read the
# documentation: https://d.android.com/studio/projects/add-native-code.html

# Sets the minimum version of CMake required to build the native library.

cmake_minimum_required(VERSION 3.4.1)

# Creates and names a library, sets it as either STATIC
# or SHARED, and provides the relative paths to its source code.
# You can define multiple libraries, and CMake builds them for you.
# Gradle automatically packages shared libraries with your APK.

add_library( # Sets the name of the library.
             ndk-sum

             # Sets the library as a shared library.
             SHARED

             # Provides a relative path to your source file(s).
             src/main/jni/first.c
             src/main/jni/second.c )

# Searches for a specified prebuilt library and stores the path as a
# variable. Because CMake includes system libraries in the search path by
# default, you only need to specify the name of the public NDK library
# you want to add. CMake verifies that the library exists before
# completing its build.

find_library( # Sets the name of the path variable.
              log-lib

              # Specifies the name of the NDK library that
              # you want CMake to locate.
              log )

# Specifies libraries CMake should link to your target library. You
# can link multiple libraries, such as libraries you define in this
# build script, prebuilt third-party libraries, or system libraries.

target_link_libraries( # Specifies the target library.
                       ndk-sum

                       # Links the target library to the log library
                       # included in the NDK.
                       ${log-lib} )
```

## 配置 Gradle
将   `externalNativeBuild {}` 块添加到模块级 `build.gradle` 文件中，并使用 `cmake {}` 或 `ndkBuild {}` 对其进行配置：

```
android {
  ...
  defaultConfig {...}
  buildTypes {...}

  // Encapsulates your external native build configurations.
  externalNativeBuild {
    // Encapsulates your CMake build configurations.
    cmake {
      // Provides a relative path to your CMake build script.
      path "CMakeLists.txt"
    }
  }
}
```

## 运行项目
不出意外的话，你可以看到输出结果为 5。

![执行结果](/images/2018/01/jni-ex4_4.png)

# 进一步学习
Google 的 [Github](https://github.com/googlesamples/android-ndk) 中有很多 NDK 的例子，里面都是使用了 NDK 编写的示例程序，有条件的话可以参考这些示例进行进一步学习。

* hello-jni: 调用本地库，接收 「Hello from JNI」字符串，并输出到 TextView 。
* two-libs: 调用本地库，返回两数字之和，并将结果输出到 TextView。
* san-angeles: 调用本地的 OpenGL ES API，渲染 3D 画面。
* hello-gl2: 调用 OpenGL ES 2.0，渲染三角形。
* bitmap-plasma: 使用本地代码访问 Android Bitmap 对象的像素缓冲区。

更多详细配置可参考[官方文档说明](https://developer.android.com/studio/projects/add-native-code.html)。

