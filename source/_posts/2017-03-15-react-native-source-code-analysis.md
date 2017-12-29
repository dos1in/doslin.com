---
title: 20分钟理解React Native For Android原理
date: 2017-03-15 18:07:06
tags:
    - react-native
    - android
categories:
    - react-native
---

## 前言
* 文中所有 RN 缩写指代React Native For Android
* 分析的 RN 代码基于

``` json
{
 "react": "15.4.1",
 "react-native": "0.39.2"
}
```
* 本文主要分析 Java 层实现，对 C++ 和 JS 笔墨较少。
* 阅读正文将花费您大约20分钟。

## 背景
公司内几个 APP 已经接入并上线了多个 RN 模块，后续规划的定制化需求及性能优化需要我们对 RN 底层原理有更深入的理解。下面通过研读源代码来分析和总结下 Android 中的 RN 实现原理。

## 从示例入手
之前写过一篇[弹射起步：Android原生项目集成React Native模块](/2016/10/08/embedded-react-native-in-app-android/)。

示例代码如下：

``` java
public class MainActivity extends ReactActivity {
    @Override
    protected String getMainComponentName() {
        return "RN_Demo";
    }
}
```

可以发现 RN 容器外层本质也是一个 **Activity** ，继承了 **ReactActivity** ，需要我们覆写 `getMainComponentName()` 方法，更改其返回值为组件名。

<!--more-->

### ReactActivity

接着跟踪到 `ReactActivity` 中，类结构如下：

![ReactActivity的类结构图](/images/2017/03/rn-code-ReactActivity-strut.png)

根据上述的结构转换成 UML 图如下（后面相关类将直接给出 UML ）：

![ReactActivity的UML图](/images/2017/03/rn-code-ReactActivity-uml.png)

这里使用了[委托模式](https://zh.wikipedia.org/wiki/委托模式)将 `Activity` 的生命周期及事件传递委托给 `ReactActivityDelegate` 的实例对象 `mDelegate` 进行处理，之所以使用这种形式是为了让 `ReactFragmentActivity` 也能复用该处理逻辑。
此外如果你有自定义的委托实现，可以在自己的 `Activity` 中覆写 `createReactActivityDelegate()` 方法。这个方法将在 `ReactActivity` 的构造函数中调用生成 `mDelegate` 实例，之后在 `onCreate()` 方法调用这个委托对象的执行入口，也就是`loadApp()`方法。

### ReactActivityDelegate

``` java
public class ReactActivityDelegate {
    protected void onCreate(Bundle savedInstanceState) {
        // 弹窗权限判断逻辑
        if (getReactNativeHost().getUseDeveloperSupport() && Build.VERSION.SDK_INT >= 23) {
        // 省略具体实现
        }
        // 组件加载逻辑
        if (mMainComponentName != null) {
          loadApp(mMainComponentName);
        }
        // 双击判断工具类 DoubleTapReloadRecognizer ，省略代码
      }

      protected void loadApp(String appKey) {
        // 省略判空代码
        
        // 创建 RN 容器根视图，父类为 FrameLayout
        mReactRootView = createRootView();
        mReactRootView.startReactApplication(
          getReactNativeHost().getReactInstanceManager(),
          appKey,
          getLaunchOptions());
        // 将 RN 视图放入 Activity
        getPlainActivity().setContentView(mReactRootView);
      }
}
```

### ReactRootView

因此可以认为所谓的 RN 其实就是一个特殊的“自定义 View ”-- `ReactRootView` 。

![RN视图](/images/2017/03/rn-code-view.png)

这里的关键调用则是 `startReactApplication()` 方法。下面是该方法需要的三个参数：

| 形参 | 类型 | 功能描述 |
| :------|:------ | :------ |
| reactInstanceManager | ReactInstanceManager | 用来创建及管理 `CatalyInstance` （提供原生与JS互调环境）的实例，同时连接调试功能，其生命周期与 `ReactRootView` 所在 `Activity` 保持一致。 |
| moduleName | String | 即实参 `appKey` ，需要保证JS中的 `AppRegistry.registerComponent` 参数值与 `Acitvity` 中的 `getMainComponentName` 返回值一致。 |
| launchOptions | Bundle（后续版本可能更改为POJO） | 默认为null，如果你需要传 [Props](https://facebook.github.io/react-native/docs/props.html) 给 JS 的话，请覆写 `createReactActivityDelegate()` 方法，并覆写 `getLaunchOptions()` 的返回值即可。 |

![ReactInstanceManager的 UML 图](/images/2017/03/rn-code-ReactInstanceManager-uml.png)

``` java
public void startReactApplication(
      ReactInstanceManager reactInstanceManager,
      String moduleName,
      @Nullable Bundle launchOptions) {
    // 确保在 UI 线程执行
    
    // 判断 ReactContext 是否已初始化，没有就异步在后台线程创建
    if (!mReactInstanceManager.hasStartedCreatingInitialContext()) {
      mReactInstanceManager.createReactContextInBackground();
    }
    // 宽高计算完成后添加布局监听
  }
```

 `startReactApplication()` 将调用其中的 `createReactContextInBackground()` 方法，我们下面来看看这个通过[构造者模式](https://zh.wikipedia.org/wiki/生成器模式)创建的实现类-- `XReactInstanceManagerImpl` 。

### ReactInstanceManager

``` java
/**
* 在后台线程异步初始化，这个方法只会在 Application 创建时调用一次，
* 重新加载 JS 时将会调用 recreateReactContextInBackground 方法
*/
@Override
public void createReactContextInBackground() {
    // 首次执行判断逻辑
    mHasStartedCreatingInitialContext = true;
    recreateReactContextInBackgroundInner();
  }
```

可以看到不管是 `createReactContextInBackground()` 还是 `recreateReactContextInBackground()` ，都是通过 `recreateReactContextInBackgroundInner()` 来初始化 `ReactContext` 的。

``` java
private void recreateReactContextInBackgroundInner() {
    // 确保在UI线程执行
    
    // 调试模式，从服务器加载 bundle
    if (mUseDeveloperSupport && mJSMainModuleName != null) {
      // 省略代码
      return;
    }
	
    // 正式环境，从本地加载 bundle
    recreateReactContextInBackgroundFromBundleLoader();
  }
  
  private void recreateReactContextInBackgroundFromBundleLoader() {
    recreateReactContextInBackground(
        new JSCJavaScriptExecutor.Factory(mJSCConfig.getConfigMap()),
        mBundleLoader);
  }
}
```

| 形参 | 类型 | 功能描述 |
| :------|:------ | :------ |
| jsExecutorFactory | JavaScriptExecutor.Factory | 管理Webkit 的 JavaScriptCore，JS与C++的双向通信在这里中转  |
| jsBundleLoader | JSBundleLoader | bundle加载器，根据`ReactNativeHost`中的配置决定从哪里加载bundle文件 |

### XReactInstanceManagerImpl

``` java
private void recreateReactContextInBackground(
      JavaScriptExecutor.Factory jsExecutorFactory,
      JSBundleLoader jsBundleLoader) {
 
    // 省略代码

    / 把两个参数封装成ReactContextInitParams对象
    ReactContextInitParams initParams =
        new ReactContextInitParams(jsExecutorFactory, jsBundleLoader);
    if (mReactContextInitAsyncTask == null) {
      // 核心代码，创建后台线程，初始化 ReactContext
      mReactContextInitAsyncTask = new ReactContextInitAsyncTask();
      mReactContextInitAsyncTask.executeOnExecutor(AsyncTask.THREAD_POOL_EXECUTOR, initParams);
    } else {
      // 省略代码
    }
  }
```

### ReactContextInitAsyncTask

``` java
private ReactApplicationContext createReactContext(
      JavaScriptExecutor jsExecutor,
      JSBundleLoader jsBundleLoader) {
    // 省略代码
    
    // 注册JS层模块，通过它把所有的 JavaScriptModule 注册到 CatalystInstance，将 JS 的可调用 API 暴露给 Java。
    JavaScriptModuleRegistry.Builder jsModulesBuilder = new JavaScriptModuleRegistry.Builder();

    // 包装 ApplicationContext 为 ReactApplicationContext
    final ReactApplicationContext reactContext = new ReactApplicationContext(mApplicationContext);
    // 调试模式
    if (mUseDeveloperSupport) {
    // 调试模式下 ReactApplicationContext 中崩溃信息由 mDevSupportManager 进行拦截处理（红色背景的错误页）
    reactContext.setNativeModuleCallExceptionHandler(mDevSupportManager);
    }

    // 省略代码
    
    try {
      // 加载给定的大量核心 ReactPackage，详细列表请查看后文的表格
      CoreModulesPackage coreModulesPackage =
        new CoreModulesPackage(this, mBackBtnHandler, mUIImplementationProvider);
      processPackage(
        coreModulesPackage,
        reactContext,
        moduleSpecs,
        reactModuleInfoMap,
        jsModulesBuilder);
    } finally {
      // 省略代码
    }

    // 加载 MainPackage 和开发者自定义的 ReactPackage，逻辑同 CoreModulesPackage
    for (ReactPackage reactPackage : mPackages) {
      // 省略代码
      
      try {
        processPackage(
          reactPackage,
          reactContext,
          moduleSpecs,
          reactModuleInfoMap,
          jsModulesBuilder);
      } finally {
        // 省略代码
      }
    }
    
    // 省略代码
    
    // 注册 Java 层模块，通过它把所有的 NativeModule 注册到 CatalystInstance ，将 Java 的可调用 API 暴露给 JS。
    NativeModuleRegistry nativeModuleRegistry;
    try {
       nativeModuleRegistry = new NativeModuleRegistry(moduleSpecs, reactModuleInfoMap);
    } finally {
      // 省略代码
    }

    // 异常处理器选择逻辑
    NativeModuleCallExceptionHandler exceptionHandler = mNativeModuleCallExceptionHandler != null
        ? mNativeModuleCallExceptionHandler
        : mDevSupportManager;
        
    // 核心逻辑， Builder 模式创建 CatalystInstance 实例
    CatalystInstanceImpl.Builder catalystInstanceBuilder = new CatalystInstanceImpl.Builder()
        .setReactQueueConfigurationSpec(ReactQueueConfigurationSpec.createDefault())
        // 将 JS 执行通信类传给 CatalystInstance
        .setJSExecutor(jsExecutor)
        // 将 Java 模块映射表传给 CatalystInstance
        .setRegistry(nativeModuleRegistry)
        // 将 JS 模块映射表传给 CatalystInstance
        .setJSModuleRegistry(jsModulesBuilder.build())
        // 将 Bundle 加载工具类传给 CatalystInstance
        .setJSBundleLoader(jsBundleLoader)
        // 将异常处理器传给 CatalystInstance
        .setNativeModuleCallExceptionHandler(exceptionHandler);

    // 省略代码
   
    final CatalystInstance catalystInstance;
    try {
      catalystInstance = catalystInstanceBuilder.build();
    } finally {
      // 省略代码
    }

    if (mBridgeIdleDebugListener != null) {
      catalystInstance.addBridgeIdleDebugListener(mBridgeIdleDebugListener);
    }
	
    // 使用 CatalystInstance 实例初始化 ReactContext 部分成员变量
    reactContext.initializeWithInstance(catalystInstance);
    
    // 解析 Bundle 文件
    catalystInstance.runJSBundle();

    return reactContext;
  }
```

| 原生模块  | 功能描述 |
| :------ | :------ |
| AndroidInfoModule | 获取Android版本号和本地服务器地址 |
| AnimationsDebugModule | 监听动画过渡性能 |
| DeviceEventManagerModule | 事件监听，比如后退键 |
| ExceptionsManagerModule | 异常处理 |
| HeadlessJsTaskSupportModule | 通知部分JS任务执行完成 |
| SourceCodeModule | 传递Bundle文件地址 |
| Timing | 在绘制帧率变化时触发JS定时器 |
| UIManagerModule | 提供JS去创建和更新原生视图的能力 |
| DebugComponentOwnershipModule | 调试功能：异步请求视图结构 |
| JSCHeapCapture | 调试功能：获取堆内存信息 |
| JSCSamplingProfiler | 调试功能：dump工具 |

### CatalystInstance

``` java

private CatalystInstanceImpl(
      final ReactQueueConfigurationSpec ReactQueueConfigurationSpec,
      final JavaScriptExecutor jsExecutor,
      final NativeModuleRegistry registry,
      final JavaScriptModuleRegistry jsModuleRegistry,
      final JSBundleLoader jsBundleLoader,
      NativeModuleCallExceptionHandler nativeModuleCallExceptionHandler) {
    // 省略代码
    
    // Android UI 线程，JS 线程和 NativeModulesQueue 线程
    mReactQueueConfiguration = ReactQueueConfigurationImpl.create(
        ReactQueueConfigurationSpec,
        new NativeExceptionHandler());
        
    // 调用 C++ 层代码进行初始化
    initializeBridge(
      new BridgeCallback(this),
      jsExecutor,
      mReactQueueConfiguration.getJSQueueThread(),
      mReactQueueConfiguration.getNativeModulesQueueThread(),
      // 将所有 Java Module 传递给 C++ 层
      mJavaRegistry.getModuleRegistryHolder(this));
    mMainExecutorToken = getMainExecutorToken();
  }
  
@Override
  public void runJSBundle() {
    // 省略代码
    
    mJSBundleHasLoaded = true;
    // Bundle 的加载逻辑请参看下文的流程图
    mJSBundleLoader.loadScript(CatalystInstanceImpl.this);

    // 省略代码
  }
```

![Bundle 加载流程图](/images/2017/03/rn-code-bundle.png)

``` java
private native void initializeBridge(ReactCallback callback,
                                       JavaScriptExecutor jsExecutor,
                                       MessageQueueThread jsQueue,
                                       MessageQueueThread moduleQueue,
                                       ModuleRegistryHolder registryHolder);
                                       
native void loadScriptFromAssets(AssetManager assetManager, String assetURL);
native void loadScriptFromFile(String fileName, String sourceURL);
native void loadScriptFromOptimizedBundle(String path, String sourceURL, int flags);
```

此时已经发现Java层的逻辑已经走完，不管是 `CatalystInstance` 实例的初始化还是 **Bundle** 的加载逻辑都将由 `C++` 层进行处理。

### CatalystInstance的创建

`CatalystInstanceImpl.cpp` 中是空实现，具体实现在 `Instance.cpp` 中。

``` C++

void CatalystInstanceImpl::initializeBridge(
	// JInstanceCallback 实现类，父类在 cxxreact/Instance.h 中。
    std::unique_ptr<InstanceCallback> callback,
    // 对应 Java 中的 JavaScriptExecutor
    std::shared_ptr<JSExecutorFactory> jsef,
    // C++ 的 JMessageQueueThread 。
    std::shared_ptr<MessageQueueThread> jsQueue,
    // C++ 的 JMessageQueueThread 。
    std::unique_ptr<MessageQueueThread> nativeQueue,
    // C++ 的 ModuleRegistryHolder 的 getModuleRegistry() 方法
    std::shared_ptr<ModuleRegistry> moduleRegistry) {
  callback_ = std::move(callback);

  jsQueue->runOnQueueSync(
    [this, &jsef, moduleRegistry, jsQueue,
     nativeQueue=folly::makeMoveWrapper(std::move(nativeQueue))] () mutable {
      // 创建 NativeToJsBridge 对象
      nativeToJsBridge_ = folly::make_unique<NativeToJsBridge>(
          jsef.get(), moduleRegistry, jsQueue, nativeQueue.move(), callback_);

      std::lock_guard<std::mutex> lock(m_syncMutex);
      m_syncReady = true;
      m_syncCV.notify_all();
    });

  CHECK(nativeToJsBridge_);
}

```

### Bundle 的加载

``` C++
void CatalystInstanceImpl::loadScriptFromAssets(jobject assetManager,
                                                const std::string& assetURL) {
  const int kAssetsLength = 9;  // strlen("assets://");
  // 获取 source 路径名，不包含前缀，默认值为 index.android.bundle
  auto sourceURL = assetURL.substr(kAssetsLength);
  // assetManager 是Java 传递的 AssetManager 。
  // extractAssetManager 是JSLoader.cpp 中通过系统动态链接库 android/asset_manager_jni.h的AAssetManager_fromJava 方法来获取 AAssetManager 对象的。
  auto manager = react::extractAssetManager(assetManager);
  // //通过 JSLoader 对象的 loadScriptFromAssets 方法读文件，得到大字符串 script（即index.android.bundle文件内容）。
  auto script = react::loadScriptFromAssets(manager, sourceURL);
  // 判断是否为 Unbundle
  if (JniJSModulesUnbundle::isUnbundle(manager, sourceURL)) {
    instance_->loadUnbundle(
      folly::make_unique<JniJSModulesUnbundle>(manager, sourceURL),
      std::move(script),
      sourceURL);
    return;
  } else {
    // instance_ 为 ReactCommon 目录下 Instance.h 中类的实例
    instance_->loadScriptFromString(std::move(script), sourceURL);
  }
}
```

![RN 三层通信图](/images/2017/03/rn-code-invoke.png)

至此 `C++` 层的调用逻辑到此为止，感兴趣的同学可以继续跟踪进入，或者可以参考文末的资料，我们下面假设底层执行完成，回到 `ReactContextInitAsyncTask` 的 `onPostExecute` 方法。

``` Java
@Override
protected void onPostExecute(Result<ReactApplicationContext> result) {
      // 省略代码
      
      setupReactContext(result.get());
      
      // 省略代码

      // 处理队列再次初始化 ReactContext 的逻辑代码
}
    
private void setupReactContext(ReactApplicationContext reactContext) {
    // 省略代码
    
    // 确保在 UI 线程中执行并且当前 ReactContext 为空
    Assertions.assertCondition(mCurrentReactContext == null);
    
    // 确保当前 CatalystInstance 实例非空
    CatalystInstance catalystInstance =
        Assertions.assertNotNull(reactContext.getCatalystInstance());
    // 初始化所有 Native Module
    catalystInstance.initialize();
    
    mDevSupportManager.onNewReactContextCreated(reactContext);
    // 添加内存警告的回调
    mMemoryPressureRouter.addMemoryPressureListener(catalystInstance);
    
    moveReactContextToCurrentLifecycleState();

    for (ReactRootView rootView : mAttachedRootViews) {
      // 核心代码，给 RootView 添加 view
      attachMeasuredRootViewToInstance(rootView, catalystInstance);
    }
    // 省略代码
}
  
private void attachMeasuredRootViewToInstance(
      ReactRootView rootView,
      CatalystInstance catalystInstance) {
      // 省略代码
      
      // 确保在 UI 线程执行

    // 重置视图内容
    rootView.removeAllViews();
    rootView.setId(View.NO_ID);

    // 设置 RootView
    UIManagerModule uiManagerModule = catalystInstance.getNativeModule(UIManagerModule.class);
    int rootTag = uiManagerModule.addMeasuredRootView(rootView);
    rootView.setRootViewTag(rootTag);
    @Nullable Bundle launchOptions = rootView.getLaunchOptions();
    WritableMap initialProps = Arguments.makeNativeMap(launchOptions);
    String jsAppModuleName = rootView.getJSModuleName();

    // 获取 AppRegistry 的代理对象
    WritableNativeMap appParams = new WritableNativeMap();
    appParams.putDouble("rootTag", rootTag);
    appParams.putMap("initialProps", initialProps);
    // AppRegistry 是 JS 暴露给 Java 层的 API
    catalystInstance.getJSModule(AppRegistry.class).runApplication(jsAppModuleName, appParams);
    
    // 省略代码
}
```

### AppRegistry

``` java
public interface AppRegistry extends JavaScriptModule {
  void runApplication(String appKey, WritableMap appParameters);
  void unmountApplicationComponentAtRootTag(int rootNodeTag);
  void startHeadlessTask(int taskId, String taskKey, WritableMap data);
}
```

`AppRegistry` 的 `runApplication()` 方法成为了加载 `index.android.js` 的主入口，而所有的 JS 方法调用都会经过 `JavaScriptModuleInvocationHandler` 。

### JavaScriptModuleInvocationHandler

``` java
@Override
public @Nullable Object invoke(Object proxy, Method method, @Nullable Object[] args) throws Throwable {
  ExecutorToken executorToken = mExecutorToken.get();
      
  // 省略代码
      
  NativeArray jsArgs = args != null ? Arguments.fromJavaArgs(args) : new WritableNativeArray();
  // 调用 C++ 层的 callFunction
  mCatalystInstance.callFunction(
        executorToken,
        mModuleRegistration.getName(),
        method.getName(),
        jsArgs
      );
      return null;
}
```

以上就是 RN Android 的执行主流程，如有疏漏，欢迎留言。

## 参考资料
1. https://github.com/facebook/react-native
2. [ReactNative Android源码分析](http://www.jianshu.com/p/02be425d7b13)
3. [React Native Android 源码框架浅析（主流程及 Java 与 JS 双边通信）](http://blog.csdn.net/yanbober/article/details/53157456)
4. [React Native通讯原理](http://www.jianshu.com/p/17d6f6c57a5c)
5. [ReactNativeAndroid源码分析-Js如何调用Native的代码](https://zhuanlan.zhihu.com/p/20464825)