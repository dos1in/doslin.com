---
title: Tinker 热修复原理及手写实现
date: 2018-01-25 14:29:36
tags:
    - hotfix
    - android
categories:
    - android
---

# 概述

移动端 APP 的热修复技术已经出来了很多年，Android 方面的解决方案也有很多（iOS 已经被苹果官方叫停），比如手淘的 [Sophix](https://help.aliyun.com/document_detail/53240.html)，微信的 [Tinker](https://github.com/Tencent/tinker) 等等，他们都能做到代码修复、资源修复以及 SO 文件修复，这里的「修复」也即意味着「替换」。在接入细节、即时生效或者性能损耗等等的一些方面上，两者存在着一些差异，我们这里不做探讨，本文主要关注的是类文件是如何被替换的并且给出一个简易的实现方案。

# 原理

目前实现代码修复主要有两种方式：native 底层替换和类加载替换。

前者的特点是修复粒度小，但有稳定性问题（Sophix 已解决），局限性在于不适用于类结构发生变化的修改和修复了的非静态方法被反射调用，它的优点是能即时生效。原理是通过对底层的 ArtMethod 结构体进行完整替换来实现。

后者的特性是修复粒度大，稳定性和兼容性较好，但改动需要冷启动才能生效。原理是在 APP 重新启动后让类加载器 Classloader 去加载新替换的类来实现改动。本文的热修复就是这种思路来实现。

那么问题来了，Android 中的类加载器是什么呢？

<!--more-->

# ClassLoader

通过参考 Android 开发者官网上的 [ClassLoader](https://developer.android.com/reference/java/lang/ClassLoader.html) 的文档说明我们可以一窥究竟。

父类是 [ClassLoader](http://androidxref.com/8.0.0_r4/xref/libcore/ojluni/src/main/java/java/lang/ClassLoader.java)，它的子类有点多，我们来画个图捋一捋关系。

![ClassLoader关系图](/images/2018/01/hotfix_1.png)

注意图中我们高亮的部分，我们主要用到了两种类加载器 [PathClassLoader](http://androidxref.com/8.0.0_r4/xref/libcore/dalvik/src/main/java/dalvik/system/PathClassLoader.java) 和 [DexClassLoader](http://androidxref.com/8.0.0_r4/xref/libcore/dalvik/src/main/java/dalvik/system/DexClassLoader.java)。

源码很简单：

```java
// PathClassLoader
public class PathClassLoader extends BaseDexClassLoader {
    public PathClassLoader(String dexPath, ClassLoader parent) {
        super(dexPath, null, null, parent);
    }

    public PathClassLoader(String dexPath, String librarySearchPath, ClassLoader parent) {
        super(dexPath, null, librarySearchPath, parent);
    }
}

// DexClassLoader
public class DexClassLoader extends BaseDexClassLoader {
    public DexClassLoader(String dexPath, String optimizedDirectory,
            String librarySearchPath, ClassLoader parent) {
        super(dexPath, new File(optimizedDirectory), librarySearchPath, parent);
    }
}

```
他们的区别是：

1. PathClassLoader：是 Android 应用中默认的类加载器，只能加载已经安装到 Android 系统中的apk文件（/data/app目录下，解压为 dex 后优化为 odex）。
2. DexClassLoader：可以加载路径下的 dex/jar/apk/zip 文件，比 PathClassLoader 更灵活，是实现热修复的关键。

那我们如何将新的 DEX 文件插入到上述的类加载器中呢？观察它们的父类[BootClassLoader](http://androidxref.com/8.0.0_r4/xref/libcore/ojluni/src/main/java/java/lang/ClassLoader.java)，可以发现有个成员变量 [DexPathList](http://androidxref.com/8.0.0_r4/xref/libcore/dalvik/src/main/java/dalvik/system/DexPathList.java)，它的属性中有一个 dex 数组，也就是说我们只要将替换的类添加到 `dexElements` 前面即可，这样类加载器跟据双亲委托模型（Parent Delegation Model）的机制就会使用先找到的类。

# 双亲委托模型

当类加载器收到加载类或资源的请求时，通常都是先委托给父类加载器加载，也就是说只有当父类加载器找不到指定类或资源时，自身才会执行实际的类加载过程，源码如下：

```java
    protected Class<?> loadClass(String className, boolean resolve) throws ClassNotFoundException {
        // 首先从已经加载的类中查找
        Class<?> clazz = findLoadedClass(className);
        if (clazz == null) {
            ClassNotFoundException suppressed = null;
            try {
                // 如果没有加载过，先调用父加载器的 loadClass
                clazz = parent.loadClass(className, false);
            } catch (ClassNotFoundException e) {
                suppressed = e;
            }
            if (clazz == null) {
                try {
                    // 父加载器都没有加载，则尝试自己去加载
                    clazz = findClass(className);
                } catch (ClassNotFoundException e) {
                    e.addSuppressed(suppressed);
                    throw e;
                }
            }
        }
        return clazz;
    }
```

一句话概括：ClassLoader 加载类时，先查看自身是否已经加载过该类，如果没有加载过会首先让父加载器去加载，如果父加载器无法加载该类时才会调用自身的 `findClass` 方法加载，该逻辑避免了类的重复加载。所以我们所要实现的就是把要替换的类可见性提前，这样类加载器就会优先找到修复过的类。

# 实现思路

好了，根据上述的铺垫，我们总结一下热修复的实现过程：

1、PathClassLoader 作为默认的类加载器，也就是第一个 DEX 文件是 PathClassLoader 自动加载的。可以通过下述代码验证：

```java
ClassLoader classLoader = getClassLoader();
int i = 0;
while (classLoader != null){
    Log.d(TAG, "ClassLoader " + (i++) + " : " + classLoader.toString());
    classLoader = classLoader.getParent();
}
```

控制台输出为：

```
ClassLoader 0 : dalvik.system.PathClassLoader[DexPathList[[zip file.....]]]
ClassLoader 1 : java.lang.BootClassLoader@4cf7d56
```

可以看到我们刚才提到的 DexPathList。所以，我们需要做的就是将其他的 DEX 文件注入到 PathClassLoader 中。

2、通过前面的分析来看，我们知道 PathClassLoader 里面的 DEX 文件是存放在一个 Element 数组中，可以包含多个 DEX 文件，所以我们只需要通过反射获取 PathClassLoader 中的 DexPathList 中的Element数组（已加载了第一个dex包，由系统加载），将要替换的 DEX 文件放置到这个数组中去。

3、将两个Element数组合并之后，再将其赋值给 PathClassLoader 的 Element 数组。

刚才我们提到了进行热修复的原理就是要替换类文件，Android 的虚拟机（基于寄存器）与 Java 的虚拟机（基于栈）有些不同，它加载的不是 .class 字节码，而是 .dex 文件（可以通过 Google 提供的 dx 工具对 .class 文件转换得到），所以上面提到的替换类文件就是等于替换 DEX 文件。那么如何将要修复的 DEX 和主工程 DEX 分开呢？那就要用到 Multidex 进行处理。

# Multidex 分包

由于 DexFormat.MAX_MEMBER_IDX 的最大值（0xFFFF）限制，Android 项目中的方法数目（包括 Android 框架、库函数、自己代码中的方法）不能超过 65536，而由于这个值等于 64 X 1024，所以又称之为 64k 问题。 一旦方法数过多，我们就需要进行分包处理，那么代码会分散到不同的 DEX 包中去。命名规则为 `classes.dex，classes2.dex，classes3.dex ...`。

根据上述 Android 支持分包加载的特性，我们有没有可能手动指定分包逻辑呢？也就是把核心代码放到第一个 `classes.dex` 中，其余的业务代码和开源库放到剩下的 DEX 包中，由系统去处理。

答案当然是肯定的，参考官方解决 64k 问题的文档，我们可以发现 Google 提供了 在 Gradle 中显式配置`multiDexKeepFile` 文件的方式，来实现指定哪些类我们需要保存到首个 DEX 文件中。


详细过程不再赘述，请参考[官方文档](https://developer.android.com/studio/build/multidex.html )，我们对比下分包前后的效果：

> 分包前：

![分包前的结构](/images/2018/01/hotfix_2.png)

> 分包后：

![分包后的结构](/images/2018/01/hotfix_3.png)

# 核心代码

``` java
                // 系统的ClassLoader
                PathClassLoader pathClassLoader = (PathClassLoader) context.getClassLoader();
                Class baseDexClazzLoader = Class.forName("dalvik.system.BaseDexClassLoader");
                Field pathListFiled = baseDexClazzLoader.getDeclaredField("pathList");
                pathListFiled.setAccessible(true);
                Object pathListObject = pathListFiled.get(pathClassLoader);

                Class systemPathClazz = pathListObject.getClass();
                Field systemElementsField = systemPathClazz.getDeclaredField("dexElements");
                systemElementsField.setAccessible(true);
                Object systemElements = systemElementsField.get(pathListObject);

                // 自定义 ClassLoader
                DexClassLoader dexClassLoader = new DexClassLoader(dex.getAbsolutePath(), fopt.getAbsolutePath(), null, context.getClassLoader());
                Class myDexClazzLoader = Class.forName("dalvik.system.BaseDexClassLoader");
                Field myPathListFiled = myDexClazzLoader.getDeclaredField("pathList");
                myPathListFiled.setAccessible(true);
                Object myPathListObject = myPathListFiled.get(dexClassLoader);

                Class myPathClazz = myPathListObject.getClass();
                Field myElementsField = myPathClazz.getDeclaredField("dexElements");
                myElementsField.setAccessible(true);
                Object myElements = myElementsField.get(myPathListObject);

                // 合并数组
                Class<?> sigleElementClazz = systemElements.getClass().getComponentType();
                int systemLength = Array.getLength(systemElements);
                int myLength = Array.getLength(myElements);
                int newSystenLength = systemLength + myLength;
                // 生成一个新的 数组，类型为Element类型
                Object newElementsArray = Array.newInstance(sigleElementClazz, newSystenLength);
                for (int i = 0; i < newSystenLength; i++) {
                    if (i < myLength) {
                        Array.set(newElementsArray, i, Array.get(myElements, i));
                    } else {
                        Array.set(newElementsArray, i, Array.get(systemElements, i - myLength));
                    }
                }
                // 覆盖新数组
                Field elementsField = pathListObject.getClass().getDeclaredField("dexElements");
                elementsField.setAccessible(true);
                elementsField.set(pathListObject, newElementsArray);
```

# 修复过程

## 转换 DEX

那么如何得到修复后的 DEX 文件呢？Android SDK 中提供了 dx 工具类（放置在 /build-tools/ 下）。

命令格式为：

```
dx --dex --output ./out.dex ./WaitToFix.class
```

可能会遇到下述问题：

> PARSE ERROR:
> class name (com/doslin/hotfix/WaitToFix) does not match path (WaitToFix.class)
> ...while parsing WaitToFix.class
> 1 error; aborting

请将 WaitToFix 放置在相应的目录结构下（与包名一致）。

```
dx --dex --output ./out.dex ./com/doslin/hotfix/WaitToFix.class
```

## 测试

通过 adb 命令将 DEX 文件放置到存储卡中，然后点击 「Fix」，结束应用后重新打开即可发现修复完成。示例工程代码可以参考 [https://github.com/JChord/demo_hotfix](https://github.com/JChord/demo_hotfix)。

```
adb push out.dex /sdcard/
```

# 其它

由于 Tinker 已开源，想了解更多实现细节的同学可去它的代码仓库上浏览。虽然 Sophix 还没开源，但它的一些技术细节已经在阿里的「深入探索 Android 热修复技术原理」这个手册中已给出，可以通读一番。

# 参考

https://developer.android.com/studio/build/multidex.html
https://source.android.com/devices/tech/dalvik/dalvik-bytecode
http://google.github.io/android-gradle-dsl/current/
https://github.com/Tencent/tinker/wiki
https://jaeger.itscoder.com/android/2016/08/27/android-classloader.html
https://www.jianshu.com/p/a67a560903fa
http://jayfeng.com/2016/03/10/由Android-65K方法数限制引发的思考/
https://juejin.im/entry/5a370a6d51882521033456af
https://juejin.im/post/5a42f29ef265da43333eaba0


