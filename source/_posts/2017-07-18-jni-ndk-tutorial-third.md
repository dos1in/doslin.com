---
title: JNI与NDK入门之三
date: 2017-07-18 16:12:33
tags:
    - jni
    - ndk
    - android
categories:
    - android
---

# 概述

前面两篇文章的示例代码的主程序都是用 Java 代码编写的，我们看到了如何在 Java 代码中使用 JNI 的方式调用 C 函数。下面我们将学习如何在由 C/C++ 编写的主程序中来运行 Java 类。

我们知道由 Java 类编译生成的字节码需要运行在 Java 虚拟机上，那么在 C/C++ 中运行 Java 代码也需要虚拟机环境吗？答案是肯定的。JNI 为这种情况提供了一套 Invocation API ，它允许本地代码在自身内存区域内加载 Java 虚拟机。编写并运行 Java 代码的 `java` 命令是由 C 语言编写的，它也是通过 `Invocation API` 来接收命令参数。

除此之外， Android 系统的 `dalvikvm` 虚拟机的主入口（ `dalvikvm-dalvik/dalvikvm/main.c` ）也是通过 Invocation API 进行工作的。在 Android 启动时， `app_process` 调用 `JNI invocation API` 在自身程序域内加载 `dalvikvm` 虚拟机，而后调用 `ZygoteInit` 类的 `main()` 方法，从而运行 `Zygote` 进程。

<!--more-->

# 使用情形

1. 需要在 C/C++ 中使用标准 Java 类库。
2. 需要在 C/C++ 中访问 Java 编写的代码。

# 功能描述

我们下面要实现一个从 C 层调用 Java 层函数的简单程序，调用流程如下图：

![调用流程图](/images/2017/07/jni-ex3-demo.png)

1. 主程序 `invocationApi.c` 使用 Invocation API 加载虚拟机。
2. 之后通过 JNI 函数加载 `InvocationTest` 类。
3. 执行被加载类的 `main()` 方法，并传参。

# 编写 Java 代码

```java
public class InvocationTest {
	public static void main(String[] args) {
		System.out.println(args[0]);
	}
}
```

该 Java 类只有一个简单的 `main()` 函数，它是一个静态方法，接收的参数是一个字符串对象数组，在方法体内有且仅有一条输出语句，用来在控制台打印字符串数组的第一个数组元素。

# 编写 C 代码

```c
// 包含各种 JNI 必须的变量及函数
#include <jni.h>
#include <stdlib.h>

int main() {
	JNIEnv *env;
	JavaVM *vm;
	JavaVMInitArgs vm_args;
	JavaVMOption* options = malloc(1 * sizeof(JavaVMOption));
	jint res;
	jclass clazz;
	jmethodID mid;
	jstring jstr;
	jclass strClass;
	jobjectArray argsAry;

	// 初始化虚拟机配置
	// 将虚拟机要加载的类读取目录设置为当前目录
	options[0].optionString = "-Djava.class.path=.";
	vm_args.version = JNI_VERSION_1_8;
	vm_args.options = options; // JavaVMOption 结构体的地址
	vm_args.nOptions = 1; // options 结构体数组元素的个数
	// 该参数指定是否忽略错误的配置参数
	// 如果填JNI_FLASE，当遇到非标准参数时，JNI_CreateJavaVM 会返回 JNI_ERR 后终止执行
	vm_args.ignoreUnrecognized = JNI_TRUE;

	// 创建虚拟机，返回值表示虚拟机是否创建成果
	res = JNI_CreateJavaVM(&vm, (void**)&env, &vm_args);

	if (res != JNI_OK) {  
        printf("Can't create Java VM: %d\n", res);  
		return -1;
	}  
	// 查找并加载类
	clazz = (*env)->FindClass(env, "InvocationTest");

	// 获取 main() 方法的ID
	mid = (*env)->GetStaticMethodID(env, clazz, "main", "([Ljava/lang/String;)V");

	// 生成字符串数组，作为 main() 方法的参数
	jstr = (*env)->NewStringUTF(env, "Hello World From C !");
	strClass = (*env)->FindClass(env, "java/lang/String");
	argsAry = (*env)->NewObjectArray(env, 1, strClass, jstr);

	// 调用 main() 方法
	(*env)->CallStaticVoidMethod(env, clazz, mid, argsAry);

	// 释放虚拟机
	(*vm)->DestroyJavaVM(vm);

	return 0;
}
```

1. `#include` 命令将 `jni.h` 头文件包含到本文件中。该头文件包含了 C 代码使用 JNI 必须的变量类型与函数定义。
2. 前面配置一些虚拟机的运行参数，参数相关内容可参考[官方文档](http://docs.oracle.com/javase/8/docs/technotes/guides/jni/spec/invocation.html)。

# 编译及运行

首先编译 Java 代码，如下：

```shell
javac InvocaitonTest.java
```

接着编译 C 代码，这里需要除了指定头文件 `<jni.h>` 与 `<jni_md.h>` 外，还需要指定 `jre` 的路径（注意不要使用 $JAVA_HOME/Contents/Home/jre/lib/server/ 及其配对参数 `-ljvm` ，否则会提示 Java 版本过低）。

```shell
gcc "-I/Library/Java/JavaVirtualMachines/jdk1.8.0_92.jdk/Contents/Home/include" \
    "-I/Library/Java/JavaVirtualMachines/jdk1.8.0_92.jdk/Contents/Home/include/darwin" \
    "-L/Library/Java/JavaVirtualMachines/jdk1.8.0_92.jdk/Contents/Home/jre/lib/jli" \
    -o invocationApi invocationApi.c -ljli
```

在运行前，指定 `libjli.dylib` 路径：

```shell
export LD_LIBRARY_PATH=/Library/Java/JavaVirtualMachines/jdk1.8.0_92.jdk/Contents/Home/jre/lib/jli/

./invocationApi
```

结果如下：

![运行结果](/images/2017/07/jni-ex3-result.png)
