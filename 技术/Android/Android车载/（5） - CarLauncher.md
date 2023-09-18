---
author: zjmantou
title: （5） - CarLauncher
time: 2023-09-18 周一
tags:
  - Android
  - 车联网
---
在之前的[Android车载应用开发与分析（1） - Android Automotive概述与编译](https://www.jianshu.com/p/bbc02e0f6575)中了解了如何下载以及编译面向车载IVI的Android系统，一切顺利的话，运行模拟器，等待启动动画播放完毕后，我们所能看到的第一个APP就是车载android的桌面，而这就是本篇文章的重点 - CarLauncher。

本篇文章以解析Android 11 源码中`CarLauncher`为主。为了便于阅读源码，现将`CarLauncher`的源码整理成可以导入Android Studio的结构，源码地址：[https://github.com/linux-link/CarLauncher](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Flinux-link%2FCarLauncher)。由于`CarLauncher`对于源码存在依赖，该项目不能直接运行，引入jar依赖的方式也不完全正确，仅供阅读使用。

本篇文章中的功能以及源码分析基于_android-11.0.0_r43，_`CarLauncher`源码位于 _**packages/apps/Car/Launcher**_

## 1.Launcher 与 CarLauncher

Launcher是安卓系统中的桌面启动器，安卓系统的桌面UI统称为Launcher。Launcher是安卓系统中的主要程序组件之一，安卓系统中如果没有Launcher就无法启动[安卓桌面](https://links.jianshu.com/go?to=https%3A%2F%2Fbaike.baidu.com%2Fitem%2F%25E5%25AE%2589%25E5%258D%2593%25E6%25A1%258C%25E9%259D%25A2)，Launcher出错的时候，安卓系统会出现“进程 com.android.launcher 意外停止”的提示窗口。这时需要重新启动Launcher。  
《百度百科 - launcher》

`Launcher`是android系统的桌面，是用户接触到的第一个带有界面的APP。它本质上就是一个系统级APP，和普通的APP一样，它界面也是在Activity上绘制出来的。

虽然`Launcher`也是一个APP，但是它涉及到的技术点却比一般的APP要多。CarLauncher作为IVI系统的桌面，需要显示系统中所有**用户可用app**的入口，显示最近用户使用的APP，同时还需要支持在桌面上动态显示如地图、音乐在内各个APP内部的信息，在桌面显示地图并与之进行简单的交互。地图开发的工作量极大，`Launcher`显然不可能引入地图的SDK再开发一个地图应用，那么如何在不扩大工作量的前提下动态的显示地图就成了`CarLauncher`的一个技术难点。

## 2.CarLauncher功能分析

原生的Carlaunher代码并不复杂，主要是协同SystemUI完成以下两个功能。

- **显示 可以快捷操作的 『首页』**

![](https://cdn.nlark.com/yuque/0/2022/webp/26044650/1670159121833-64491769-c194-46f0-aacb-5384e46353b0.webp)

- **显示 所有APP入口的 『桌面』**
- ![](https://cdn.nlark.com/yuque/0/2022/webp/26044650/1670159160299-73b6c720-2058-4779-9257-a184d2662a72.webp)  
      
    需要注意的是，只有红框中的内容才属于CarLauncher的内容，红框之外的属于SystemUI的内容。虽然SystemUI在下方的NaviBar有6个按钮，但是只有点击**首页**和**App桌面**才会进入**CarLauncher**，点击其它按钮都会进入其它APP，所以都不在本篇文章的分析范围。

## 3.CarLauncher 源码分析

CarLauncher的源码结构如下：

![](https://cdn.nlark.com/yuque/0/2022/webp/26044650/1670159167542-5e305190-e3fe-4e33-9d11-7f187946d8f5.webp)

### 3.1 Android.dp

**CarLauncher**的android.bp相对比较简单，定义了CarLauncher的源码结构，和依赖的类库。如果你对android.bp完全不了解，可以先看一下 [Android.bp入门教程](https://www.jianshu.com/p/f23e18933122) 学习一下基础的语法，再来回过头来看**CarLauncher**的android.bp相信会容易理解很多。

```shell
android_app {
    name: "CarLauncher",
    srcs: ["src/**/*.java"],
    resource_dirs: ["res"],
    // 允许使用系统的hide api
    platform_apis: true,
    required: ["privapp_whitelist_com.android.car.carlauncher"],
    // 签名类型 ： platform
    certificate: "platform",
    // 设定apk安装路径为priv-app
    privileged: true,
    // 覆盖其它类型的Launcher
    overrides: [
        "Launcher2",
        "Launcher3",
        "Launcher3QuickStep",
    ],
    optimize: {
        enabled: false,
    },
    dex_preopt: {
        enabled: false,
    },
    // 引入静态库
    static_libs: [
        "androidx-constraintlayout_constraintlayout-solver",
        "androidx-constraintlayout_constraintlayout",
        "androidx.lifecycle_lifecycle-extensions",
        "car-media-common",
        "car-ui-lib",
    ],
    libs: ["android.car"],
    product_variables: {
        pdk: {
            enabled: false,
        },
    },
}
```

上述Android.bp中我们需要注意一个属性overrides，它表示覆盖的意思。在系统编译时Launcher2、Launcher3和Launcher3QuickStep都会被CarLauncher取代，前面三个Launcher并不是车机系统的桌面，车载系统中会用CarLauncher这个定制新的桌面取代掉其它系统的桌面。同样的，如果我们不想使用系统中自带的CarLauncher，那么也需要在overrides中覆盖掉CarLauncher。在自主开发的车载Android系统中这个属性我们会经常用到，用我们自己定制的各种APP来取代系统中默认的APP，比如系统设置等等。

### 3.2 AndroidManifest.xml

Manifest文件中我们可以看到CarLauncher所需要的权限，以及入口Activity。

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
  package="com.android.car.carlauncher">

  <uses-permission android:name="android.car.permission.ACCESS_CAR_PROJECTION_STATUS" />
  <!--  System permission to host maps activity  -->
  <uses-permission android:name="android.permission.ACTIVITY_EMBEDDING" />
  <!--  System permission to send events to hosted maps activity  -->
  <uses-permission android:name="android.permission.INJECT_EVENTS" />
  <!--  System permission to use internal system windows  -->
  <uses-permission android:name="android.permission.INTERNAL_SYSTEM_WINDOW" />
  <!--  System permissions to bring hosted maps activity to front on main display  -->
  <uses-permission android:name="android.permission.MANAGE_ACTIVITY_STACKS" />
  <!--  System permission to query users on device  -->
  <uses-permission android:name="android.permission.MANAGE_USERS" />
  <!--  System permission to control media playback of the active session  -->
  <uses-permission android:name="android.permission.MEDIA_CONTENT_CONTROL" />
  <!--  System permission to get app usage data  -->
  <uses-permission android:name="android.permission.PACKAGE_USAGE_STATS" />
  <!--  System permission to query all installed packages  -->
  <uses-permission android:name="android.permission.QUERY_ALL_PACKAGES" />
  <uses-permission android:name="android.permission.REORDER_TASKS" />
  <!--  To connect to media browser services in other apps, media browser clients
  that target Android 11 need to add the following in their manifest  -->
  <queries>
    <intent>
      <action android:name="android.media.browse.MediaBrowserService" />
    </intent>
  </queries>
  <application
    android:icon="@drawable/ic_launcher_home"
    android:label="@string/app_title"
    android:supportsRtl="true"
    android:theme="@style/Theme.Launcher">
    <activity
      android:name=".CarLauncher"
      android:clearTaskOnLaunch="true"
      android:configChanges="uiMode|mcc|mnc"
      android:launchMode="singleTask"
      android:resumeWhilePausing="true"
      android:stateNotNeeded="true"
      android:windowSoftInputMode="adjustPan">
      <meta-data
        android:name="distractionOptimized"
        android:value="true" />
      <intent-filter>
        <action android:name="android.intent.action.MAIN" />

        <category android:name="android.intent.category.HOME" />
        <category android:name="android.intent.category.DEFAULT" />
      </intent-filter>
    </activity>
    <activity
      android:name=".AppGridActivity"
      android:exported="true"
      android:launchMode="singleInstance"
      android:theme="@style/Theme.Launcher.AppGridActivity">
      <meta-data
        android:name="distractionOptimized"
        android:value="true" />
        </activity>
        </application>
        </manifest>
```

关于Manifest我们重点来了解其中一些不常用的标签即可。

#### queries

<queries/>是在Android 11 上为了收紧应用权限而引入的。用于，指定当前应用程序要与之交互的其他应用程序集，这些其他应用程序可以通过 package、intent、provider。示例：

```xml
<queries>
    <package android:name="string" />
    <intent>
        ...
    </intent>
    <provider android:authorities="list" />
    ...
</queries>
```

更多内容可以参考：[Android Developers #queries](https://links.jianshu.com/go?to=https%3A%2F%2Fdeveloper.android.google.cn%2Fguide%2Ftopics%2Fmanifest%2Fqueries-element%3Fhl%3Dcn)

#### **android:clearTaskOnLaunch = "true"**

每次启动都启动根Activity，并清理其他的Activity。

#### **android:configChanges="uiMode|mcc|mnc"**

关于android:configChanges就不废话了，直接上一张相对完整的表格供参考。

|   |   |
|---|---|
|**VALUE**|**DESCRIPTION**|
|mcc|国际移动用户识别码所属国家代号是改变了,sim被侦测到了，去更新mcc MCC是移动用户所属国家代号|
|mnc|国际移动用户识别码的移动网号码是改变了, sim被侦测到了，去更新mnc MNC是移动网号码，最多由两位数字组成，用于识别移动用户所归属的移动通信网|
|locale|用户所在区域发生变化。例如：用户切换了语言时，切换后的语言会显示出来|
|touchscreen|触摸屏发生改变|
|keyboard|键盘发生了改变。例如：用户介入了外部的键盘|
|keyboardHidden|键盘的可用性发生了改变|
|navigation|导航发生了变化|
|screenLayout|屏幕的显示发生了变化。例如：不同的显示被激活|
|fontScale|字体比例发生了变化。例如：选择了不同的全局字体|
|uiMode|用户的模式发生了变化|
|orientation|屏幕方向改变了。例如：横竖屏切换|
|smallestScreenSize|屏幕的物理大小改变了。例如：连接到一个外部的屏幕上|

#### **android:resumeWhilePausing = "true"**

**当前一个Activity还在执行onPause()方法时（即在暂停过程中，还没有完全暂停），允许该Activity显示（此时Activity不能申请任何其他额外的资源，比如相机）**

#### **android:stateNotNeeded="true"**

**这个属性默认情况为false，若设为true，则当Activity重新启动时不会调用onSaveInstanceState方法，onCreate()方法中的Bundle参数将永远为null。在一些特殊场合下，由于用户按了Home键，该属性设置为true时，可以保证不用保存原先的状态引用，一定程度上节省空间资源。**

#### **android:name="distractionOptimized"**

**设定当前Activity处于活动状态，是否导致驾驶员分心，在国外车载Android应用程序需要遵守Android官方制定《驾驶员分心指南》，这个规则在国内使用的很少，具体请参考******[Driver Distraction Guidelines | Android Open Source Project](https://links.jianshu.com/go?to=https%3A%2F%2Fsource.android.google.cn%2Fdevices%2Fautomotive%2Fdriver_distraction%2Fguidelines)

### **3.3 AppGridActivity**

`**AppGridActivity**`**用来显示系统中所有的APP，为用户的使用提供入口。**

![](https://cdn.nlark.com/yuque/0/2022/webp/26044650/1670159283559-b82d0dac-45d4-47e1-9fd5-735dcce5a2bd.webp)

作为应用开发者，我们需要关注以下两个功能是如何实现的：

- 显示系统中所有的APP，并过滤掉一些不需要显示在桌面的APP（例如：后台的Service）
- 显示最近使用的APP

#### **显示系统中所有的APP（All App）**

`CarLauncher`中用于筛选所有APP的方法都集中在`AppLauncherUtils`

```Java
/**
 * 获取我们希望在启动器中以未排序的顺序看到的所有组件，包括启动器活动和媒体服务。
 *
 * @param blackList             要隐藏的应用程序（包名称）列表（可能为空）
 * @param customMediaComponents 不应在Launcher中显示的媒体组件（组件名称）列表（可能为空），因为将显示其应用程序的Launcher活动
 * @param appTypes              要显示的应用程序类型（例如：全部或仅媒体源）
 * @param openMediaCenter       当用户选择媒体源时，启动器是否应导航到media center。
 * @param launcherApps          {@link LauncherApps}系统服务
 * @param carPackageManager     {@link CarPackageManager}系统服务
 * @param packageManager        {@link PackageManager}系统服务
 * @return 一个新的 {@link LauncherAppsInfo}
 */
@NonNull
static LauncherAppsInfo getLauncherApps(
        @NonNull Set<String> blackList,
        @NonNull Set<String> customMediaComponents,
        @AppTypes int appTypes,
        boolean openMediaCenter,
        LauncherApps launcherApps,
        CarPackageManager carPackageManager,
        PackageManager packageManager,
        CarMediaManager carMediaManager) {

    if (launcherApps == null || carPackageManager == null || packageManager == null
            || carMediaManager == null) {
        return EMPTY_APPS_INFO;
    }
    // 检索所有符合给定intent的服务
    List<ResolveInfo> mediaServices = packageManager.queryIntentServices(
            new Intent(MediaBrowserService.SERVICE_INTERFACE),
            PackageManager.GET_RESOLVED_FILTER);
    // 检索指定packageName的Activity的列表
    List<LauncherActivityInfo> availableActivities =
            launcherApps.getActivityList(null, Process.myUserHandle());

    Map<ComponentName, AppMetaData> launchablesMap = new HashMap<>(
            mediaServices.size() + availableActivities.size());
    Map<ComponentName, ResolveInfo> mediaServicesMap = new HashMap<>(mediaServices.size());

    // Process media services
    if ((appTypes & APP_TYPE_MEDIA_SERVICES) != 0) {
        for (ResolveInfo info : mediaServices) {
            String packageName = info.serviceInfo.packageName;
            String className = info.serviceInfo.name;
            ComponentName componentName = new ComponentName(packageName, className);
            mediaServicesMap.put(componentName, info);
            if (shouldAddToLaunchables(componentName, blackList, customMediaComponents,
                    appTypes, APP_TYPE_MEDIA_SERVICES)) {
                final boolean isDistractionOptimized = true;

                Intent intent = new Intent(Car.CAR_INTENT_ACTION_MEDIA_TEMPLATE);
                intent.putExtra(Car.CAR_EXTRA_MEDIA_COMPONENT, componentName.flattenToString());

                AppMetaData appMetaData = new AppMetaData(
                    info.serviceInfo.loadLabel(packageManager),
                    componentName,
                    info.serviceInfo.loadIcon(packageManager),
                    isDistractionOptimized,
                    context -> {
                        if (openMediaCenter) {
                            AppLauncherUtils.launchApp(context, intent);
                        } else {
                            selectMediaSourceAndFinish(context, componentName, carMediaManager);
                        }
                    },
                    context -> {
                        // 返回系统中所有MainActivity带有Intent.CATEGORY_INFO 和 Intent.CATEGORY_LAUNCHER的intent
                        Intent packageLaunchIntent =
                                packageManager.getLaunchIntentForPackage(packageName);
                        AppLauncherUtils.launchApp(context,
                                packageLaunchIntent != null ? packageLaunchIntent : intent);
                    });
                launchablesMap.put(componentName, appMetaData);
            }
        }
    }

    // Process activities
    if ((appTypes & APP_TYPE_LAUNCHABLES) != 0) {
        for (LauncherActivityInfo info : availableActivities) {
            ComponentName componentName = info.getComponentName();
            String packageName = componentName.getPackageName();
            if (shouldAddToLaunchables(componentName, blackList, customMediaComponents,
                    appTypes, APP_TYPE_LAUNCHABLES)) {
                boolean isDistractionOptimized =
                    isActivityDistractionOptimized(carPackageManager, packageName,
                        info.getName());

                Intent intent = new Intent(Intent.ACTION_MAIN)
                    .setComponent(componentName)
                    .addCategory(Intent.CATEGORY_LAUNCHER)
                    .setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
                // 获取app的name，和 app的图标
                AppMetaData appMetaData = new AppMetaData(
                    info.getLabel(),
                    componentName,
                    info.getBadgedIcon(0),
                    isDistractionOptimized,
                    context -> AppLauncherUtils.launchApp(context, intent),
                    null);
                launchablesMap.put(componentName, appMetaData);
            }
        }
    }

    return new LauncherAppsInfo(launchablesMap, mediaServicesMap);
}
```

通过LauncherApps.getActivityList()，获得的`List<LauncherActivityInfo>`包含了系统中所有配置了`Intent#ACTION_MAIN` 和`Intent#CATEGORY_LAUNCHER`的Activity信息。  
**String LauncherActivityInfogetLabel() : 获取app的name**  
**String LauncherActivityInfo.getComponentName() : 获取app的Mainactivity信息**  
**Drawable LauncherActivityInfo.getBadgedIcon(0) : 获取App的图标**  
最后，当用户点击图标时，虽然也是通过startActivity启动App，但是ActivityOptions可以让我们决定目标APP在哪个屏幕上启动，这对当前车载多屏系统而言非常重要。

```Java
static void launchApp(Context context, Intent intent) {
    ActivityOptions options = ActivityOptions.makeBasic();
    // 在当前的屏幕上启动目标App的Activity
    options.setLaunchDisplayId(context.getDisplayId());
    context.startActivity(intent, options.toBundle());
}
```

#### **显示最近使用的APP （Recent APP）**

Android系统中提供了`UsageStatusManager`来提供对设备使用情况历史记录和统计信息的访问，`UsageStatusManager`  
使用`android.provider.Settings#ACTION_USAGE_ACCESS_SETTINGS`，`getAppStandbyBucket()`，`queryEventsForSelf(long,long)`，方法时不需要添加额外的权限。但是除此以外的方法都需要`android.permission.PACKAGE_USAGE_STATS`权限。

```Java
/**
 * 请注意，为了从上一次boot中获得使用情况统计数据，设备必须经过干净的关闭过程。
 */
private List<AppMetaData> getMostRecentApps(LauncherAppsInfo appsInfo) {
    ArrayList<AppMetaData> apps = new ArrayList<>();
    if (appsInfo.isEmpty()) {
        return apps;
    }

    // 获取从1年前开始的使用情况统计数据，返回如下条目：
    // "During 2017 App A is last used at 2017/12/15 18:03"
    // "During 2017 App B is last used at 2017/6/15 10:00"
    // "During 2018 App A is last used at 2018/1/1 15:12"
    List<UsageStats> stats =
            mUsageStatsManager.queryUsageStats(
                    UsageStatsManager.INTERVAL_YEARLY,
                    System.currentTimeMillis() - DateUtils.YEAR_IN_MILLIS,
                    System.currentTimeMillis());

    if (stats == null || stats.size() == 0) {
        return apps; // empty list
    }

    stats.sort(new LastTimeUsedComparator());

    int currentIndex = 0;
    int itemsAdded = 0;
    int statsSize = stats.size();
    int itemCount = Math.min(mColumnNumber, statsSize);
    while (itemsAdded < itemCount && currentIndex < statsSize) {
        UsageStats usageStats = stats.get(currentIndex);
        String packageName = usageStats.mPackageName;
        currentIndex++;

        // 不包括自己
        if (packageName.equals(getPackageName())) {
            continue;
        }

        // TODO(b/136222320): 每个包都可以获得UsageStats，但一个包可能包含多个媒体服务。我们需要找到一种方法来获取每个服务的使用率统计数据。
        ComponentName componentName = AppLauncherUtils.getMediaSource(mPackageManager,
                packageName);
        // 免除媒体服务的后台和启动器检查
        if (!appsInfo.isMediaService(componentName)) {
            // 不要包括仅在后台运行的应用程序
            if (usageStats.getTotalTimeInForeground() == 0) {
                continue;
            }
            // 不要包含不支持从启动器启动的应用程序
            Intent intent = getPackageManager().getLaunchIntentForPackage(packageName);
            if (intent == null || !intent.hasCategory(Intent.CATEGORY_LAUNCHER)) {
                continue;
            }
        }

        AppMetaData app = appsInfo.getAppMetaData(componentName);
        // 防止重复条目
        // e.g. app is used at 2017/12/31 23:59, and 2018/01/01 00:00
        if (app != null && !apps.contains(app)) {
            apps.add(app);
            itemsAdded++;
        }
    }
    return apps;
}
```

### 3.4 CarLauncher

CarLauncher和普通的Android应用一样，都会有一个主布局文件，并且在Activity的onCreate中进行一系列初始化工作，所以我们先从CarLauncher的布局文件和onCreate方法开始分析。

#### CarLauncher的主布局文件

_这里我们只分析横屏的布局文件，源码中其实还包含了竖屏和多窗口的布局。_

![](https://cdn.nlark.com/yuque/0/2022/webp/26044650/1670159683446-4a6efbcd-aa12-4cad-bf9f-045075e39ef1.webp)  
  

```xml
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
  xmlns:app="http://schemas.android.com/apk/res-auto"
  xmlns:tools="http://schemas.android.com/tools"
  android:layout_width="match_parent"
  android:layout_height="match_parent"
  android:layoutDirection="ltr"
  tools:context=".CarLauncher">

  <!-- 省略 -->

  <View
    android:id="@+id/top_line"
    style="@style/HorizontalLineDivider"
    app:layout_constraintTop_toTopOf="parent" />

  <androidx.cardview.widget.CardView
    style="@style/CardViewStyle"
    // ... 
    >
    <!-- 用来显示地图的 ActivityView -->
    <android.car.app.CarActivityView
      android:id="@+id/maps"
      android:layout_width="match_parent"
      android:layout_height="match_parent" />

  </androidx.cardview.widget.CardView>

  <!-- 显示天气 -->
  <FrameLayout
    android:id="@+id/contextual"
    android:layout_width="0dp"
    android:layout_height="0dp"
    android:layout_margin="@dimen/main_screen_widget_margin"
    android:layoutDirection="locale"
    app:layout_constraintBottom_toTopOf="@+id/divider_horizontal"
    app:layout_constraintLeft_toRightOf="@+id/divider_vertical"
    app:layout_constraintRight_toLeftOf="@+id/end_edge"
    app:layout_constraintTop_toBottomOf="@+id/top_edge" />

  <!-- 显示本地音乐 -->
  <FrameLayout
    android:id="@+id/playback"
    android:layout_width="0dp"
    android:layout_height="0dp"
    android:layout_margin="@dimen/main_screen_widget_margin"
    android:layoutDirection="locale"
    app:layout_constraintBottom_toTopOf="@+id/bottom_edge"
    app:layout_constraintLeft_toRightOf="@+id/divider_vertical"
    app:layout_constraintRight_toLeftOf="@+id/end_edge"
    app:layout_constraintTop_toBottomOf="@+id/divider_horizontal" />

  <View
    android:id="@+id/bottom_line"
    style="@style/HorizontalLineDivider"
    app:layout_constraintBottom_toBottomOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

`CarLauncher`的布局文件很简单，几乎没什么值得解释的地方。

#### 初始化Android桌面

初始化Android桌面的工作大多都是在CarLuancher.onCreate方法中完成，该方法代码如下：

```Java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    // 在多窗口模式下『car_launcher_multiwindow』不显示“地图”面板。
    // 注意：拆分屏幕的CTS测试与启动器默认活动的活动视图不兼容
    if (isInMultiWindowMode() || isInPictureInPictureMode()) {
        setContentView(R.layout.car_launcher_multiwindow);
    } else {
        setContentView(R.layout.car_launcher);
    }
    // 初始化『天气』和『音乐』fragment
    initializeFragments();
    // 监听ActivityView状态，并启动地图面板
    mActivityView = findViewById(R.id.maps);
    if (mActivityView != null) {
        mActivityView.setCallback(mActivityViewCallback);
    }
    // 监听屏幕状态，并启动地图面板
    mDisplayManager = getSystemService(DisplayManager.class);
    mDisplayManager.registerDisplayListener(mDisplayListener, mMainHandler);
}
```

上述代码中的ActivityView是用来显示地图的一个ViewGroup。ActivityView的状态回调如下：

```Java
private final ActivityView.StateCallback mActivityViewCallback = new ActivityView.StateCallback() {
    @Override
    public void onActivityViewReady(ActivityView view) {
        if (DEBUG) Log.d(TAG, "onActivityViewReady(" + getUserId() + ")");
        mActivityViewReady = true;
        startMapsInActivityView();
        maybeLogReady();
    }

    @Override
    public void onActivityViewDestroyed(ActivityView view) {
        if (DEBUG) Log.d(TAG, "onActivityViewDestroyed(" + getUserId() + ")");
        mActivityViewReady = false;
    }

    @Override
    public void onTaskMovedToFront(int taskId) {
        if (DEBUG) {
            Log.d(TAG, "onTaskMovedToFront(" + getUserId() + "): started=" + mIsStarted);
        }
        try {
            if (mIsStarted) {
                ActivityManager am = (ActivityManager) getSystemService(ACTIVITY_SERVICE);
                am.moveTaskToFront(CarLauncher.this.getTaskId(), /* flags= */ 0);
            }
        } catch (RuntimeException e) {
            Log.w(TAG, "Failed to move CarLauncher to front.");
        }
    }
};
```

设定回调的作用就是在ActivityView初始化完毕后，启动地图。

```Java
private void startMapsInActivityView() {
    if (mActivityView == null || !mActivityViewReady) {
        return;
    }
    // 如果我们碰巧被重新呈现为多显示模式，我们将跳过在“Activity”视图中启动内容，因为我们无论如何都会被重新创建。
    if (isInMultiWindowMode() || isInPictureInPictureMode()) {
        return;
    }
    // 在“活动可见性测试（ActivityVisibilityTests）”的显示关闭时不要启动地图。
    if (getDisplay().getState() != Display.STATE_ON) {
        return;
    }
    try {
        mActivityView.startActivity(getMapsIntent());
    } catch (ActivityNotFoundException e) {
        Log.w(TAG, "Maps activity not found", e);
    }
}

private Intent getMapsIntent() {
    // 为应用程序的主Activity创建一个意图，不指定要运行的特定Activity，而是提供一个选择器来查找该Activity。
    return Intent.makeMainSelectorActivity(Intent.ACTION_MAIN, Intent.CATEGORY_APP_MAPS);
}
```

在onCreate中还会监听当前的屏幕状态，每当屏幕的逻辑显示属性（如大小和密度）发生更改时，都会在ActivityView中重新显示地图。

```Java
private final DisplayListener mDisplayListener = new DisplayListener() {
    @Override
    public void onDisplayAdded(int displayId) {}
    @Override
    public void onDisplayRemoved(int displayId) {}

    @Override
    public void onDisplayChanged(int displayId) {
        if (displayId != getDisplay().getDisplayId()) {
            return;
        }
        // startMapsInActivityView（）将检查显示器的状态。
        startMapsInActivityView();
    }
};
```

#### ActivityView

允许将Activity启动到自身的任务容器。本质上是一个虚拟屏幕，所以在此`ActivityView`中启动的活动受适用于在`VirtualDisplay`上启动的相同规则的限制。  
如果想在我们自己项目中使用`ActivityView`，需要从Android源码移植，Android SDK中并没有提供ActivityView相关的API。

注意：由于ActivityView中的hosted maps活动当前处于虚拟屏幕，因此系统认为该活动始终位于前面。以直接intent启动“地图”Activity将不起作用。要在real display上启动“地图”活动，需要使用{****@link*** *Intent#CATEGORY_APP_MAPS}类别将intent发送给launcher，launcher将在real display上启动Activity。

有关`VirtualDisplay`的内容可以参考这篇老哥的博客：[Android P 图形显示系统（四） Android VirtualDisplay解析](https://www.jianshu.com/p/c4ea60bc73d2)

### **参考资料**

[Android Developers | queries](https://links.jianshu.com/go?to=https%3A%2F%2Fdeveloper.android.google.cn%2Fguide%2Ftopics%2Fmanifest%2Fqueries-element%3Fhl%3Dcn)

  
  
作者：林栩  
链接：[https://www.jianshu.com/p/32793949a731  ](https://www.jianshu.com/p/32793949a731)
来源：简书  
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。