---
author: zjmantou
title: （12） - SystemUI （一）
time: 2023-09-18 周一
tags:
  - Android
  - 车联网
---
## 1.前言

Android 车载应用开发与分析是一个系列性的文章，这个是第12篇，该系列文章旨在分析原生车载Android系统中核心应用的实现方式，帮助初次从事车载应用开发的同学，更好地理解车载应用开发的方式，积累android系统应用的开发经验。

注意：本文的源码分析部分非常的枯燥，最好还是下载android源码然后对着看，逐步理顺逻辑。  
本文中使用的源码基于android-11.0.0_r48  
在线源码可以使用下面的网址（基于android-11.0.0_r21）  
[http://aospxref.com/android-11.0.0_r21/xref/frameworks/base/packages/CarSystemUI/](https://links.jianshu.com/go?to=http%3A%2F%2Faospxref.com%2Fandroid-11.0.0_r21%2Fxref%2Fframeworks%2Fbase%2Fpackages%2FCarSystemUI%2F)  
[http://aospxref.com/android-11.0.0_r21/xref/frameworks/base/packages/SystemUI/](https://links.jianshu.com/go?to=http%3A%2F%2Faospxref.com%2Fandroid-11.0.0_r21%2Fxref%2Fframeworks%2Fbase%2Fpackages%2FSystemUI%2F)

## 2.车载 SystemUI

### 2.1 SystemUI 概述

SystemUI通俗的解释就是系统的 UI，在Android 系统中由SystemUI负责统一管理整个系统层的UI，它也是一个系统级应用程序（APK），但是与我们之前接触过的系统应用程序不同，SystemUI的源码在/frameworks/base/packages/目录下，而不是在/packages/目录下，这也说明了SystemUI这个应用的本质上可以归属于framework层。

- SystemUI

Android - Phone中SystemUI从源码量看就是一个相当复杂的程序，常见的如：状态栏、消息中心、近期任务、截屏以及一系列功能都是在SystemUI中实现的。

源码位置：[/frameworks/base/packages/SystemUI](https://links.jianshu.com/go?to=http%3A%2F%2Faospxref.com%2Fandroid-11.0.0_r21%2Fxref%2Fframeworks%2Fbase%2Fpackages%2FSystemUI%2F)

![](https://cdn.nlark.com/yuque/0/2023/webp/26044650/1672660244389-53f5831b-2987-44d2-8a10-600ffef718e2.webp)

- CarSystemUI

Android-AutoMotive 中的SystemUI相对手机中要简单不少，目前商用车载系统中几乎必备的顶部状态栏、消息中心、底部导航栏在原生的Android系统中都已经实现了。

源码位置：[frameworks/base/packages/CarSystemUI](https://links.jianshu.com/go?to=http%3A%2F%2Faospxref.com%2Fandroid-11.0.0_r21%2Fxref%2Fframeworks%2Fbase%2Fpackages%2FCarSystemUI%2F)

![](https://cdn.nlark.com/yuque/0/2023/webp/26044650/1672660288282-4e2f3342-3799-479a-a65d-50fc80dea39a.webp)

虽然CarSystemUI与SystemUI的源码位置不同，但是二者实际上是复用关系。通过阅读CarSystemUI的Android.bp文件可以发现CarSystemUI在编译时把SystemUI以静态库的方式引入进来了。

android.bp源码位置：[/frameworks/base/packages/CarSystemUI/Android.bp](https://links.jianshu.com/go?to=http%3A%2F%2Faospxref.com%2Fandroid-11.0.0_r21%2Fxref%2Fframeworks%2Fbase%2Fpackages%2FCarSystemUI%2FAndroid.bp)

```
android_library {
    name: "CarSystemUI-core",
    ...
    static_libs: [
        "SystemUI-core",
        "SystemUIPluginLib",
        "SystemUISharedLib",
        "SystemUI-tags",
        "SystemUI-proto",
        ...
    ],
    ...
}
```

### 2.2 SystemUI 启动流程

Android开发者应该都听说`SystemServer`，它是Android framework中关键系统的服务，由Android系统最核心的进程`Zygote`fork生成，进程名为`system_server`。我们常说的`ActivityManagerService`、`PackageManagerService`、`WindowManageService`都是由`SystemServer`启动的。

而在`ActivityManagerService`完成启动后（SystemReady），SystemServer就会去着手启动`SystemUI`。

SystemServer 的源码路径：[frameworks/base/services/java/com/android/server/SystemServer.java](https://links.jianshu.com/go?to=http%3A%2F%2Faospxref.com%2Fandroid-11.0.0_r21%2Fxref%2Fframeworks%2Fbase%2Fservices%2Fjava%2Fcom%2Fandroid%2Fserver%2FSystemServer.java)

```
mActivityManagerService.systemReady(() -> {
    Slog.i(TAG, "Making services ready");

    t.traceBegin("StartSystemUI");
    try {
        startSystemUi(context, windowManagerF);
    } catch (Throwable e) {
        reportWtf("starting System UI", e);
    }
    t.traceEnd();
}, t);
```

`startSystemUi`()代码细节如下.从这里我们可以看出，`SystemUI`本质就是一个Service，通过Pm获取到的Component 是com.android.systemui/.SystemUIService。

```
private static void startSystemUi(Context context, WindowManagerService windowManager) {
        PackageManagerInternal pm = LocalServices.getService(PackageManagerInternal.class);
        Intent intent = new Intent();
        intent.setComponent(pm.getSystemUiServiceComponent());
        intent.addFlags(Intent.FLAG_DEBUG_TRIAGED_MISSING);
        //Slog.d(TAG, "Starting service: " + intent);
        context.startServiceAsUser(intent, UserHandle.SYSTEM);
        windowManager.onSystemUiStarted();
    }
```

在`startSystemUi`()中启动`SystemUIService`，在`SystemUIService`的`oncreate`()方法中再通过`SystemUIApplication.startServicesIfNeeded()`来完成SystemUI的组件的初始化。

SystemUIService 源码位置：[/frameworks/base/packages/SystemUI/src/com/android/systemui/SystemUIService.java](https://links.jianshu.com/go?to=http%3A%2F%2Faospxref.com%2Fandroid-11.0.0_r21%2Fxref%2Fframeworks%2Fbase%2Fpackages%2FSystemUI%2Fsrc%2Fcom%2Fandroid%2Fsystemui%2FSystemUIService.java)

```
// SystemUIService
@Override
public void onCreate() {
    super.onCreate();
    Slog.e("SystemUIService", "onCreate");
    // Start all of SystemUI
((SystemUIApplication) getApplication()).startServicesIfNeeded();
    ...
}
```

在`startServicesIfNeeded`()中，通过`SystemUIFactory`获取到配置在config.xml中每个子模块的className。

SystemUIApplication 源码位置：[/frameworks/base/packages/SystemUI/src/com/android/systemui/SystemUIApplication.java](https://links.jianshu.com/go?to=http%3A%2F%2Faospxref.com%2Fandroid-11.0.0_r21%2Fxref%2Fframeworks%2Fbase%2Fpackages%2FSystemUI%2Fsrc%2Fcom%2Fandroid%2Fsystemui%2FSystemUIApplication.java)

```
// SystemUIApplication
public void startServicesIfNeeded() {
    String[] names = SystemUIFactory.getInstance().getSystemUIServiceComponents(getResources());
    startServicesIfNeeded("StartServices", names);
}

// SystemUIFactory
/** Returns the list of system UI components that should be started. */
public String[] getSystemUIServiceComponents(Resources resources) {
    return resources.getStringArray(R.array.config_systemUIServiceComponents);
}
```

```
 <!-- SystemUI Services: The classes of the stuff to start. -->
    <string-array name="config_systemUIServiceComponents" translatable="false">
        <item>com.android.systemui.util.NotificationChannels</item>
        <item>com.android.systemui.keyguard.KeyguardViewMediator</item>
        <item>com.android.systemui.recents.Recents</item>
        <item>com.android.systemui.volume.VolumeUI</item>
        <item>com.android.systemui.stackdivider.Divider</item>
        <item>com.android.systemui.statusbar.phone.StatusBar</item>
        <item>com.android.systemui.usb.StorageNotification</item>
        <item>com.android.systemui.power.PowerUI</item>
        <item>com.android.systemui.media.RingtonePlayer</item>
        <item>com.android.systemui.keyboard.KeyboardUI</item>
        <item>com.android.systemui.pip.PipUI</item>
        <item>com.android.systemui.shortcut.ShortcutKeyDispatcher</item>
        <item>@string/config_systemUIVendorServiceComponent</item>
        <item>com.android.systemui.util.leak.GarbageMonitor$Service</item>
        <item>com.android.systemui.LatencyTester</item>
        <item>com.android.systemui.globalactions.GlobalActionsComponent</item>
        <item>com.android.systemui.ScreenDecorations</item>
        <item>com.android.systemui.biometrics.AuthController</item>
        <item>com.android.systemui.SliceBroadcastRelayHandler</item>
        <item>com.android.systemui.SizeCompatModeActivityController</item>
        <item>com.android.systemui.statusbar.notification.InstantAppNotifier</item>
        <item>com.android.systemui.theme.ThemeOverlayController</item>
        <item>com.android.systemui.accessibility.WindowMagnification</item>
        <item>com.android.systemui.accessibility.SystemActions</item>
        <item>com.android.systemui.toast.ToastUI</item>
    </string-array>
```

最终在startServicesIfNeeded()中通过反射完成了每个SystemUI组件的创建，然后再调用各个SystemUI的onStart()方法来继续执行子模块的初始化。

```
private SystemUI[] mServices;

private void startServicesIfNeeded(String metricsPrefix, String[] services) {
    if (mServicesStarted) {
        return;
    }
    mServices = new SystemUI[services.length];
    ...

    final int N = services.length;
    for (int i = 0; i < N; i++) {
        String clsName = services[i];
        if (DEBUG) Log.d(TAG, "loading: " + clsName);
        try {
            SystemUI obj = mComponentHelper.resolveSystemUI(clsName);
            if (obj == null) {
                Constructor constructor = Class.forName(clsName).getConstructor(Context.class);
                obj = (SystemUI) constructor.newInstance(this);
            }
            mServices[i] = obj;
        } catch (ClassNotFoundException
                | NoSuchMethodException
                | IllegalAccessException
                | InstantiationException
                | InvocationTargetException ex) {
            throw new RuntimeException(ex);
        }

        if (DEBUG) Log.d(TAG, "running: " + mServices[i]);
        // 调用各个子模块的start()
        mServices[i].start();
        // 首次启动时，这里始终为false，不会被调用
        if (mBootCompleteCache.isBootComplete()) {
            mServices[i].onBootCompleted();
        }
    }
    mServicesStarted = true;
}
```

`SystemUIApplication`在`OnCreate`()方法中注册了一个开机广播，当接收到开机广播后会调用`SystemUI`的`onBootCompleted`()方法来告诉每个子模块Android系统已经完成开机。

```
   @Override
    public void onCreate() {
        super.onCreate();
        Log.v(TAG, "SystemUIApplication created.");
        // 设置所有服务继承的应用程序主题。
        // 请注意，在清单中设置应用程序主题仅适用于activity。这里是让Service保持与主题设置同步。
        setTheme(R.style.Theme_SystemUI);

        if (Process.myUserHandle().equals(UserHandle.SYSTEM)) {
            IntentFilter bootCompletedFilter = new IntentFilter(Intent.ACTION_BOOT_COMPLETED);
            bootCompletedFilter.setPriority(IntentFilter.SYSTEM_HIGH_PRIORITY);
            registerReceiver(new BroadcastReceiver() {
                @Override
                public void onReceive(Context context, Intent intent) {
                    if (mBootCompleteCache.isBootComplete()) return;
                    if (DEBUG) Log.v(TAG, "BOOT_COMPLETED received");
                    unregisterReceiver(this);
                    mBootCompleteCache.setBootComplete();
                    if (mServicesStarted) {
                        final int N = mServices.length;
                        for (int i = 0; i < N; i++) {
                            mServices[i].onBootCompleted();
                        }
                    }
                }
            }, bootCompletedFilter);
               ...
        } else {
            // 我们不需要为正在执行某些任务的子进程启动服务。
           ...
        }
    }
```

这里的`SystemUI`是一个抽象类，状态栏、近期任务等等模块都是继承自`SystemUI`，通过这种方式可以很大程度上简化复杂的`SystemUI`程序中各个子模块创建方式，同时我们可以通过配置资源的方式动态加载需要的`SystemUI`模块。

在实际的项目中开发我们自己的SystemUI时，这种初始化子模块的方式是值得我们学习的，不过由于原生的SystemUI使用了AOP框架 - Dagger来创建组件，所以SystemUI子模块的初始化细节就不再介绍了。

![](https://cdn.nlark.com/yuque/0/2023/png/26044650/1672660748038-b31e6c59-8509-4c43-bdc2-c4362c6b0680.png)

SystemUI的源码如下，方法基本都能见名知意，就不再介绍了。

```
public abstract class SystemUI implements Dumpable {
    protected final Context mContext;

    public SystemUI(Context context) {
        mContext = context;
    }

    public abstract void start();

    protected void onConfigurationChanged(Configuration newConfig) {
    }

    // 非核心功能，可以不用关心
    @Override
    public void dump(@NonNull FileDescriptor fd, @NonNull PrintWriter pw, @NonNull String[] args) {
    }

    protected void onBootCompleted() {
    }
```

总结一下，SystemUI的大致启动流程可以归纳如下（时序图语法并不严谨，理解即可）

![](https://cdn.nlark.com/yuque/0/2023/png/26044650/1672660779754-c708b6b2-f610-4763-8019-b3f630b0db0b.png)

### 3.CarSystemUI 的启动流程

之前也提到过`CarSystemUI`复用了手机`SystemUI`的代码，所以`CarSystemUI`的启动流程和`SystemUI`的是完全一致的。

这里就有个疑问，`CarSystemUI`中需要的功能与`SystemUI`中是有差异的，那么是这些差异化的功能是如何引入并完成初始化？以及一些手机的`SystemUI`才需要的功能是如何去除的呢？

其实很简单，在`SystemUI`的启动流程中我们得知，各个子模块的className是通过`SystemUIFactory`的`getSystemUIServiceComponents`()获取到的，那么只要继承`SystemUIFactory`并重写`getSystemUIServiceComponents`()就可以了。

```
public class CarSystemUIFactory extends SystemUIFactory {

    @Override
    protected SystemUIRootComponent buildSystemUIRootComponent(Context context) {
        return DaggerCarSystemUIRootComponent.builder()
                .contextHolder(new ContextHolder(context))
                .build();
    }

    @Override
    public String[] getSystemUIServiceComponents(Resources resources) {
        Set<String> names = new HashSet<>();
        // 先引入systemUI中的components
        for (String s : super.getSystemUIServiceComponents(resources)) {
            names.add(s);
        }
        // 再移除CarsystemUI不需要的components
        for (String s : resources.getStringArray(R.array.config_systemUIServiceComponentsExclude)) {
            names.remove(s);
        }
        // 最后再添加CarsystemUI特有的components
        for (String s : resources.getStringArray(R.array.config_systemUIServiceComponentsInclude)) {
            names.add(s);
        }

        String[] finalNames = new String[names.size()];
        names.toArray(finalNames);

        return finalNames;
    }
}
```

```
 <!-- 需要移除的Components. -->
    <string-array name="config_systemUIServiceComponentsExclude" translatable="false">
        <item>com.android.systemui.recents.Recents</item>
        <item>com.android.systemui.volume.VolumeUI</item>
        <item>com.android.systemui.stackdivider.Divider</item>
        <item>com.android.systemui.statusbar.phone.StatusBar</item>
        <item>com.android.systemui.keyboard.KeyboardUI</item>
        <item>com.android.systemui.pip.PipUI</item>
        <item>com.android.systemui.shortcut.ShortcutKeyDispatcher</item>
        <item>com.android.systemui.LatencyTester</item>
        <item>com.android.systemui.globalactions.GlobalActionsComponent</item>
        <item>com.android.systemui.SliceBroadcastRelayHandler</item>
        <item>com.android.systemui.statusbar.notification.InstantAppNotifier</item>
        <item>com.android.systemui.accessibility.WindowMagnification</item>
        <item>com.android.systemui.accessibility.SystemActions</item>
    </string-array>

    <!-- 新增的Components. -->
    <string-array name="config_systemUIServiceComponentsInclude" translatable="false">
        <item>com.android.systemui.car.navigationbar.CarNavigationBar</item>
        <item>com.android.systemui.car.voicerecognition.ConnectedDeviceVoiceRecognitionNotifier</item>
        <item>com.android.systemui.car.window.SystemUIOverlayWindowManager</item>
        <item>com.android.systemui.car.volume.VolumeUI</item>
    </string-array>
```

通过以上方式，就完成了`CarSystemUI`子模块的替换。

由于`CarSystemUI`模块的源码量极大，全部分析一遍再写成文章耗费的时间将无法估计，这里结合我个人在车载方面的工作经验，拣出了一些在商用车载项目必备的功能，来分析它们在原生系统中是如何实现的。

## 3.顶部状态栏与底部导航栏

- 顶部状态栏

状态栏是`CarSystemUI`中一个功能重要的功能，它负责向用户展示操作系统当前最基本信息，例如：时间、蜂窝网络的信号强度、蓝牙信息、wifi信息等。

![](https://cdn.nlark.com/yuque/0/2023/png/26044650/1672660883686-b1720dd7-9c14-4916-b14f-e4e314d28d54.png)

- 底部导航栏

在原生的车载Android系统中，底部的导航按钮由经典的三颗返回、主页、菜单键替换成如下图所示的七颗快捷功能按钮。从左到右依次主页、地图、蓝牙音乐、蓝牙电话、桌面、消息中心、语音助手。

![](https://cdn.nlark.com/yuque/0/2023/png/26044650/1672660894137-8068df4a-c1c6-4fdb-b6d5-1e0abfe17732.png)

### 3.1 布局方式

- 顶部状态栏  
    顶部状态栏的布局方式比较简单，如下图所示：

![](https://cdn.nlark.com/yuque/0/2023/png/26044650/1672660911384-8739986a-9583-4b6b-b452-154525226190.png)

布局文件的源码就不贴了，量比较大，而且包含了许多的自定义View，如果不是为了学习如何自定义View阅读的意义不大。

源码位置：[frameworks/base/packages/CarSystemUI/res/layout/car_top_navigation_bar.xml](https://links.jianshu.com/go?to=http%3A%2F%2Faospxref.com%2Fandroid-11.0.0_r21%2Fxref%2Fframeworks%2Fbase%2Fpackages%2FCarSystemUI%2Fres%2Flayout%2Fcar_top_navigation_bar.xml)

- 底部导航栏  
    底部状态栏的布局方式就更简单了，如下图所示：

![](https://cdn.nlark.com/yuque/0/2023/png/26044650/1672660940477-74c12e79-42b4-49e5-8c8f-16f7150e4e30.png)

不过比较有意思的是，导航栏、状态栏每个按钮对应的Action的intent都是直接定义在布局文件的xml中的，这点或许值得参考。

```
<com.android.systemui.car.navigationbar.CarNavigationButton
    android:id="@+id/grid_nav"
    style="@style/NavigationBarButton"
    systemui:componentNames="com.android.car.carlauncher/.AppGridActivity"
    systemui:highlightWhenSelected="true"
    systemui:icon="@drawable/car_ic_apps"
    systemui:intent="intent:#Intent;component=com.android.car.carlauncher/.AppGridActivity;launchFlags=0x24000000;end"
    systemui:selectedIcon="@drawable/car_ic_apps_selected" />
```

### 3.2 初始化流程

在`SystemUI`的启动流程中，`SystemUIApplication`在通过反射创建好`CarNavigationBar`后，紧接就调用了`start()`方法，那么我们就从`start()`入手，开始UI的初始化流程。

在start()方法中，首先是向`IStatusBarService`中注册一个`CommandQueue`，然后执行`createNavigationBar()`方法，并把注册的结果下发。

`CommandQueue`继承自`IStatusBar.Stub`。因此它是`IStatusBar`的服务（Bn）端。在完成注册后，这一Binder对象的客户端（Bp）端将会保存在`IStatusBarService`之中。因此它是`IStatusBarService`与`BaseStatusBar`进行通信的桥梁。

`IStatusBarService`，即系统服务`StatusBarManagerService`是状态栏导航栏向外界提供服务的前端接口，运行于system_server进程中。

注意：定制SystemUI时，我们可以不使用 IStatusBarService 和 IStatusBar 来保存 SystemUI 的状态

```
// CarNavigationBar

private final CommandQueue mCommandQueue;
private final IStatusBarService mBarService;

@Override
public void start() {
    ...
    RegisterStatusBarResult result = null;
    try {
        result = mBarService.registerStatusBar(mCommandQueue);
    } catch (RemoteException ex) {
        ex.rethrowFromSystemServer();
    }
    ...
    createNavigationBar(result);
    ...
}
```

在createNavigationBar()中依次执行buildNavBarWindows()、buildNavBarContent()、attachNavBarWindows()。

```
// CarNavigationBar
private void createNavigationBar(RegisterStatusBarResult result) {
    buildNavBarWindows();
    buildNavBarContent();
    attachNavBarWindows();
    // 如果注册成功，尝试设置导航条的初始状态。
if (result != null) {
        setImeWindowStatus(Display.DEFAULT_DISPLAY, result.mImeToken,
                result.mImeWindowVis, result.mImeBackDisposition,
                result.mShowImeSwitcher);
    }
}
```

下面依次介绍每个方法的实际作用。

- **buildNavBarWindows()** 这个方法目的是创建出状态栏的容器 - **navigation_bar_window。**

```
// CarNavigationBar
private final CarNavigationBarController mCarNavigationBarController;

private void buildNavBarWindows() {
    mTopNavigationBarWindow = mCarNavigationBarController.getTopWindow();
    mBottomNavigationBarWindow = mCarNavigationBarController.getBottomWindow();
    ...
}

// CarNavigationBarController
private final NavigationBarViewFactory mNavigationBarViewFactory;

public ViewGroup getTopWindow() {
    return mShowTop ? mNavigationBarViewFactory.getTopWindow() : null;
}

// NavigationBarViewFactory
public ViewGroup getTopWindow() {
    return getWindowCached(Type.TOP);
}

private ViewGroup getWindowCached(Type type) {
    if (mCachedContainerMap.containsKey(type)) {
        return mCachedContainerMap.get(type);
    }

    ViewGroup window = (ViewGroup) View.inflate(mContext,
            R.layout.navigation_bar_window, /* root= */ null);
    mCachedContainerMap.put(type, window);
    return mCachedContainerMap.get(type);
}
```

**navigation_bar_window** 是一个自定义View(NavigationBarFrame)，它的核心类是`DeadZone`.

`DeadZone`字面意思就是“死区”，它的作用是消耗沿导航栏顶部边缘的无意轻击。当用户在输入法上快速输入时，他们可能会尝试点击空格键、“overshoot”，并意外点击主页按钮。每次点击导航栏外的UI后，死区会暂时扩大（因为这是偶然点击更可能发生的情况），然后随着时间的推移，死区又会缩小（因为稍后的点击可能是针对导航栏顶部的）。

navigation_bar_window 源码位置：[/frameworks/base/packages/SystemUI/res/layout/navigation_bar_window.xml](https://links.jianshu.com/go?to=http%3A%2F%2Faospxref.com%2Fandroid-11.0.0_r21%2Fxref%2Fframeworks%2Fbase%2Fpackages%2FSystemUI%2Fres%2Flayout%2Fnavigation_bar_window.xml)

- **buildNavBarContent()**

这个方法目的是将状态栏的实际View添加到上一步创建出的容器中，并对触摸和点击事件进行初始化。

```
// CarNavigationBar
private void buildNavBarContent() {
    mTopNavigationBarView = mCarNavigationBarController.getTopBar(isDeviceSetupForUser());
    if (mTopNavigationBarView != null) {
        mSystemBarConfigs.insetSystemBar(SystemBarConfigs.TOP, mTopNavigationBarView);
        mTopNavigationBarWindow.addView(mTopNavigationBarView);
    }

    mBottomNavigationBarView = mCarNavigationBarController.getBottomBar(isDeviceSetupForUser());
    if (mBottomNavigationBarView != null) {
        mSystemBarConfigs.insetSystemBar(SystemBarConfigs.BOTTOM, mBottomNavigationBarView);
        mBottomNavigationBarWindow.addView(mBottomNavigationBarView);
    }
    ...
}

// CarNavigationBarController
public CarNavigationBarView getTopBar(boolean isSetUp) {
    if (!mShowTop) {
        return null;
    }

    mTopView = mNavigationBarViewFactory.getTopBar(isSetUp);
    setupBar(mTopView, mTopBarTouchListener, mNotificationsShadeController);
    return mTopView;
}

// 初始化 
private void setupBar(CarNavigationBarView view, View.OnTouchListener statusBarTouchListener,
        NotificationsShadeController notifShadeController) {
    view.setStatusBarWindowTouchListener(statusBarTouchListener);
    view.setNotificationsPanelController(notifShadeController);
    mButtonSelectionStateController.addAllButtonsWithSelectionState(view);
    mButtonRoleHolderController.addAllButtonsWithRoleName(view);
    mHvacControllerLazy.get().addTemperatureViewToController(view);
}


// NavigationBarViewFactory
public CarNavigationBarView getTopBar(boolean isSetUp) {
    return getBar(isSetUp, Type.TOP, Type.TOP_UNPROVISIONED);
}

private CarNavigationBarView getBar(boolean isSetUp, Type provisioned, Type unprovisioned) {
    CarNavigationBarView view;
    if (isSetUp) {
        view = getBarCached(provisioned, sLayoutMap.get(provisioned));
    } else {
        view = getBarCached(unprovisioned, sLayoutMap.get(unprovisioned));
    }

    if (view == null) {
        String name = isSetUp ? provisioned.name() : unprovisioned.name();
        Log.e(TAG, "CarStatusBar failed inflate for " + name);
        throw new RuntimeException(
                "Unable to build " + name + " nav bar due to missing layout");
    }
    return view;
}

private CarNavigationBarView getBarCached(Type type, @LayoutRes int barLayout) {
    if (mCachedViewMap.containsKey(type)) {
        return mCachedViewMap.get(type);
    }
    // 
    CarNavigationBarView view = (CarNavigationBarView) View.inflate(mContext, barLayout,
            /* root= */ null);
    // 在开头包括一个FocusParkingView。当用户导航到另一个窗口时，旋转控制器将焦点“停”在这里。这也用于防止wrap-around.。
view.addView(new FocusParkingView(mContext), 0);

    mCachedViewMap.put(type, view);
    return mCachedViewMap.get(type);
}
```

- **attachNavBarWindows()**

最后一步，将创建的View通过windowManger显示到屏幕上。

```
private void attachNavBarWindows() {
    mSystemBarConfigs.getSystemBarSidesByZOrder().forEach(this::attachNavBarBySide);
}

private void attachNavBarBySide(int side) {
    switch(side) {
        case SystemBarConfigs.TOP:
            if (mTopNavigationBarWindow != null) {
                mWindowManager.addView(mTopNavigationBarWindow,
                        mSystemBarConfigs.getLayoutParamsBySide(SystemBarConfigs.TOP));
            }
            break;
        case SystemBarConfigs.BOTTOM:
            if (mBottomNavigationBarWindow != null && !mBottomNavBarVisible) {
                mBottomNavBarVisible = true;
                mWindowManager.addView(mBottomNavigationBarWindow,
                        mSystemBarConfigs.getLayoutParamsBySide(SystemBarConfigs.BOTTOM));
            }
            break;
            ...
            break;
        default:
            return;
    }
}
```

简单总结一下，UI初始化的流程图如下。

![](https://cdn.nlark.com/yuque/0/2023/png/26044650/1672661188267-38e96a36-1d0d-4eea-8dd6-91d134b29c36.png)

### 3.3 关键功能

#### 3.3.1 打开/关闭消息中心

在原生车载Android中有两种方式打开消息中心分别是，1.通过点击消息中心按钮，2.通过手势下拉状态栏。

我们先来看第一种实现方式 ，通过点击按钮展开消息中心。

![](https://cdn.nlark.com/yuque/0/2023/webp/26044650/1672661206194-1f9024f6-9d10-42b3-9d51-42bc7fa07be9.webp)

`CarNavigationBarController`中对外暴露了一个可以注册监听回调的方法，`CarNavigationBarController`会把外部注册的监听事件会传递到`CarNavigationBarView`中。

```
 /** 设置切换通知面板的通知控制器。 */
public void registerNotificationController(
        NotificationsShadeController notificationsShadeController) {
    mNotificationsShadeController = notificationsShadeController;
    if (mTopView != null) {
        mTopView.setNotificationsPanelController(mNotificationsShadeController);
    }
    ...
}
```

当CarNavigationBarView中的notifications按钮被按下时，就会将打开消息中心的消息回调给之前注册进来的接口。

```
// CarNavigationBarView
@Override
public void onFinishInflate() {
    ...
    mNotificationsButton = findViewById(R.id.notifications);
    if (mNotificationsButton != null) {
        mNotificationsButton.setOnClickListener(this::onNotificationsClick);
    }
    ...
}
protected void onNotificationsClick(View v) {
    if (mNotificationsShadeController != null) {
        mNotificationsShadeController.togglePanel();
    }
}
```

消息中心的控制器在接收到回调消息后，根据需要执行展开消息中心面板的方法即可

```
// NotificationPanelViewMediator
mCarNavigationBarController.registerNotificationController(
        new CarNavigationBarController.NotificationsShadeController() {
            @Override
            public void togglePanel() {
                mNotificationPanelViewController.toggle();
            }
            
            // 这个方法用于告知外部类，当前消息中心的面板是否处于展开状态
            @Override
            public boolean isNotificationPanelOpen() {
                return mNotificationPanelViewController.isPanelExpanded();
            }
        });
```

再来看第二种实现方式 ，通过下拉手势展开消息中心，这也是我们最常用的方式。

![](https://cdn.nlark.com/yuque/0/2023/webp/26044650/1672661277406-2a595d4f-ee12-48fa-a4ef-ab5b512fd9ab.webp)

实现思路第一种方式一样，CarNavigationBarController中对外暴露了一个可以注册监听回调的方法，接着会把外部注册的监听事件会传递给CarNavigationBarView。

```
// CarNavigationBarController
public void registerTopBarTouchListener(View.OnTouchListener listener) {
    mTopBarTouchListener = listener;
    if (mTopView != null) {
        mTopView.setStatusBarWindowTouchListener(mTopBarTouchListener);
    }
}
```

这次在`CarNavigationBarView`中则是拦截了触摸事件的分发，如果当前消息中心已经展开，则`CarNavigationBarView`直接消费触摸事件，后续事件不再对外分发。如果当前消息中心没有展开，则将触摸事件分外给外部，这里的外部就是指消息中心中的`TopNotificationPanelViewMediator`。

```
// CarNavigationBarView

// 用于连接通知的打开/关闭手势
private OnTouchListener mStatusBarWindowTouchListener;

public void setStatusBarWindowTouchListener(OnTouchListener statusBarWindowTouchListener) {
    mStatusBarWindowTouchListener = statusBarWindowTouchListener;
}

@Override
public boolean onInterceptTouchEvent(MotionEvent ev) {
    if (mStatusBarWindowTouchListener != null) {
        boolean shouldConsumeEvent = mNotificationsShadeController == null ? false
                : mNotificationsShadeController.isNotificationPanelOpen();

        // 将触摸事件转发到状态栏窗口，以便在需要时拖动窗口（Notification shade）
mStatusBarWindowTouchListener.onTouch(this, ev);

        if (mConsumeTouchWhenPanelOpen && shouldConsumeEvent) {
            return true;
        }
    }
    return super.onInterceptTouchEvent(ev);
}
```

TopNotificationPanelViewMediator在初始化过程中就向CarNavigationBarController注册了触摸事件的监听。

```
.// TopNotificationPanelViewMediator
@Override
public void registerListeners() {
    super.registerListeners();
    getCarNavigationBarController().registerTopBarTouchListener(
            getNotificationPanelViewController().getDragOpenTouchListener());
}
```

最终状态栏的触摸事件会在OverlayPanelViewController中得到处理。

```
// OverlayPanelViewController
public final View.OnTouchListener getDragOpenTouchListener() {
    return mDragOpenTouchListener;
}

mDragOpenTouchListener = (v, event) -> {
    if (!mCarDeviceProvisionedController.isCurrentUserFullySetup()) {
        return true;
    }
    if (!isInflated()) {
        getOverlayViewGlobalStateController().inflateView(this);
    }

    boolean consumed = openGestureDetector.onTouchEvent(event);
    if (consumed) {
        return true;
    }
    // 判断是否要展开、收起 消息中心的面板
    maybeCompleteAnimation(event);
    return true;
};
```

#### 3.3.2 占用应用的显示区域

不知道你有没有这样的疑问，既然顶部的状态栏和底部导航栏都是通过WindowManager.addView()显示到屏幕上，那么打开应用为什么会自动“让出”状态栏占用的区域呢？

主要原因在于状态栏的Window的Type和我们平常使用的TYPE_APPLICATION是不一样的。

```
private WindowManager.LayoutParams getLayoutParams() {
    WindowManager.LayoutParams lp = new WindowManager.LayoutParams(
            isHorizontalBar(mSide) ? ViewGroup.LayoutParams.MATCH_PARENT : mGirth,
            isHorizontalBar(mSide) ? mGirth : ViewGroup.LayoutParams.MATCH_PARENT,
            mapZOrderToBarType(mZOrder),
            WindowManager.LayoutParams.FLAG_NOT_FOCUSABLE
| WindowManager.LayoutParams.FLAG_NOT_TOUCH_MODAL
| WindowManager.LayoutParams.FLAG_WATCH_OUTSIDE_TOUCH
| WindowManager.LayoutParams.FLAG_SPLIT_TOUCH,
            PixelFormat.TRANSLUCENT);
    lp.setTitle(BAR_TITLE_MAP.get(mSide));
    lp.providesInsetsTypes = new int[]{BAR_TYPE_MAP[mBarType], BAR_GESTURE_MAP.get(mSide)};
    lp.setFitInsetsTypes(0);
    lp.windowAnimations = 0;
    lp.gravity = BAR_GRAVITY_MAP.get(mSide);
    return lp;
}

private int mapZOrderToBarType(int zOrder) {
    return zOrder >= HUN_ZORDER ? WindowManager.LayoutParams.TYPE_NAVIGATION_BAR_PANEL
            : WindowManager.LayoutParams.TYPE_STATUS_BAR_ADDITIONAL;
}
```

CarSystemUI顶部的状态栏WindowType是 TYPE_STATUS_BAR_ADDITIONAL

![](https://cdn.nlark.com/yuque/0/2023/png/26044650/1672661417303-73e5ca12-cc37-4cd0-948c-1c22a3892dce.png)

底部导航栏的WindowType是 TYPE_NAVIGATION_BAR_PANEL。

![](https://cdn.nlark.com/yuque/0/2023/png/26044650/1672661426823-71f49d84-680c-487c-8a35-6fa4bdb0e058.png)

## 4. 总结

`SystemUI`在原生的车载Android系统是一个极其复杂的模块，考虑多数从手机应用转行做车载应用的开发者并对`SystemUI`的了解并不多，本篇介绍了`CarSystemUI`的启动、和状态栏的实现方式，希望能帮到正在或以后会从事`SystemUI`开发的同学。

除此以外，车载SystemUI中还有“消息中心”、“近期任务”等一些关键模块，这些内容就放到以后再做介绍吧。

  
  
作者：林栩  
链接：https://www.jianshu.com/p/bf07886c5d8f  
来源：简书  
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。