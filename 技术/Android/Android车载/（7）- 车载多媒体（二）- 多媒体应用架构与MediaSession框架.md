---
author: zjmantou
title: （7）- 车载多媒体（二）- 多媒体应用架构与MediaSession框架
time: 2023-09-18 周一
tags:
  - Android
  - 车联网
---
参考资料  
[媒体应用架构概览 | Android 开发者 | Android Developers](https://links.jianshu.com/go?to=https%3A%2F%2Fdeveloper.android.google.cn%2Fguide%2Ftopics%2Fmedia-apps%2Fmedia-apps-overview%3Fhl%3Dzh_cn)  
[MediaSession | Android Developers](https://links.jianshu.com/go?to=https%3A%2F%2Fdeveloper.android.google.cn%2Freference%2Fandroid%2Fmedia%2Fsession%2FMediaSession%3Fhl%3Den)  
[MediaSession框架全解析_qzns木雨的博客-CSDN博客_mediasession](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fweixin_42229694%2Farticle%2Fdetails%2F89315026)

## 1. 多媒体应用架构

### 1.1 传统应用架构

播放音频或视频的多媒体应用通常由两部分组成：

- **播放器**：接收传入的数据多媒体，并输出音频或视频。可以是MediaPlayer、ExoPlayer或其他Player。
- **界面**：用于显示、控制播放器状态界面。

![](https://cdn.nlark.com/yuque/0/2022/png/26044650/1670161018766-79360ee9-5ac0-4caa-80d4-bd04475104e3.png)

  
众所周知，如果需要在应用的后台继续播放音频，我们就需要把Player放置在Service中，那么**界面**与**播放器**之间通信就非常值得研究了。很长一段时间里，都是由Service提供一个Binder来实现与**播放器**之间的通信。但是往往**下拉的状态栏**和**桌面的Widget**都需要与Service之间进行通信，这时候Service就不得不通过实现一系列AIDL接口/广播/ContentProvider完成与其它应用之间的通信，而这些通信手段既增加了应用开发者之间的沟通成本，也增加了应用之间的耦合度。

为了解决上面的问题，Android官方从Android5.0开始提供了**MediaSession**框架。

### 1.2 MediaSession 框架

MediaSession框架规范了音视频应用中**界面**与**播放器**之间的通信接口，实现界面与播放器之间的完全解耦。框架定义了两个重要的类**媒体会话**和**媒体控制器**，它们为构建多媒体播放器应用提供了一个完善的结构。

**媒体会话**和**媒体控制器**通过以下方式相互通信：使用与标准播放器操作（播放、暂停、停止等）相对应的预定义回调，以及用于定义应用独有的特殊行为的可扩展自定义调用。  
![](https://cdn.nlark.com/yuque/0/2022/png/26044650/1670161046293-246f85fb-19b0-487f-97d5-f7b28d2819d7.png)

## 2. MediaSession 介绍

**MediaSession**框架属于典型的**C/S**架构，有四个常用的成员类，是整个MediaSession框架流程控制的核心。

### **2.1 客户端媒体浏览器 - MediaBrowser**

**媒体浏览器**，用来**连接**`MediaBrowserService`和**订阅数据**，通过它的回调接口我们可以获取与**Service**的连接状态以及获取在**Service**中的音乐库数据。在**客户端**（也就是上文我们提到的**界面**，或者说是**控制端**）中创建。  
**媒体浏览器**不是线程安全的。所有调用都应在构造`MediaBrowser`的线程上进行。

```kotlin
@RequiresApi(Build.VERSION_CODES.M)
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)

    val component = ComponentName(this, MediaService::class.java)
    mMediaBrowser = MediaBrowser(this, component, connectionCallback, null);
    mMediaBrowser.connect()
}
```

#### 2.1.1 MediaBrowser.ConnectionCallback

用于接收与**MediaBrowserService**连接事件的回调，在创建MediaBrowser时传入。

```kotlin
@RequiresApi(Build.VERSION_CODES.M)
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)

    val component = ComponentName(this, MediaService::class.java)
    mMediaBrowser = MediaBrowser(this, component, connectionCallback, null);
    mMediaBrowser.connect()
}

private val connectionCallback = object : MediaBrowser.ConnectionCallback() {

    override fun onConnected() {
        super.onConnected()
    }

    override fun onConnectionFailed() {
        super.onConnectionFailed()
    }

    override fun onConnectionSuspended() {
        super.onConnectionSuspended()
    }
}
```

#### 2.1.2 MediaBrowser.ItemCallback

用于返回MediaBrowser.getItem()的结果。

```kotlin
private val connectionCallback = object : MediaBrowser.ConnectionCallback() {

    override fun onConnected() {
        super.onConnected()
        // ...
        if(mMediaBrowser.isConnected) {
            val mediaId = mMediaBrowser.root
            mMediaBrowser.getItem(mediaId, itemCallback)
        }
    }
}

@RequiresApi(Build.VERSION_CODES.M)
private val itemCallback = object : MediaBrowser.ItemCallback(){

    override fun onItemLoaded(item: MediaBrowser.MediaItem?) {
        super.onItemLoaded(item)
    }

    override fun onError(mediaId: String) {
        super.onError(mediaId)
    }
}
```

#### 2.1.3 MediaBrowser.MediaItem

包含有关单个媒体项的信息，用于浏览/搜索媒体。MediaItem依赖于**服务端**提供，因此框架本身无法保证它包含的值都是正确的。

#### 2.1.4 MediaBrowser.SubscriptionCallback

用于订与**MediaBrowserService**中**MediaBrowser.MediaItem**列表变化的回调。

```kotlin
private val connectionCallback = object : MediaBrowser.ConnectionCallback() {

    override fun onConnected() {
        super.onConnected()
        // ...
        if(mMediaBrowser.isConnected) {
            val mediaId = mMediaBrowser.root
            // 需要先取消订阅
            mMediaBrowser.unsubscribe(mediaId)
            // 服务端会调用onLoadChildren
            mMediaBrowser.subscribe(mediaId, subscribeCallback)
        }
    }
}

private val subscribeCallback = object : MediaBrowser.SubscriptionCallback(){
    override fun onChildrenLoaded(
        parentId: String,
        children: MutableList<MediaBrowser.MediaItem>
    ) {
        super.onChildrenLoaded(parentId, children)
    }

    override fun onChildrenLoaded(
        parentId: String,
        children: MutableList<MediaBrowser.MediaItem>,
        options: Bundle
    ) {
        super.onChildrenLoaded(parentId, children, options)
    }

    override fun onError(parentId: String) {
        super.onError(parentId)
    }

    override fun onError(parentId: String, options: Bundle) {
        super.onError(parentId, options)
    }
}
```

### **2.2 客户端媒体控制器 - MediaController**

**媒体控制器**，用来向服务端发送控制指令，例如：播放、暂停等等，在客户端中创建。媒体控制器是线程安全的。MediaController还有一个关联的权限_**android.permission.MEDIA_CONTENT_CONTROL**_（不是必须加的权限）必须是系统级应用才可以获取，幸运的是车载应用一般都是系统级应用。  
MediaController必须在MediaBrowser连接成功后才可以创建。

```kotlin
private val connectionCallback = object : MediaBrowser.ConnectionCallback() {

    override fun onConnected() {
        super.onConnected()
        // ...
        if(mMediaBrowser.isConnected) {
            val sessionToken = mMediaBrowser.sessionToken
            mMediaController = MediaController(applicationContext,sessionToken)
        }
    }
}
```

#### 2.2.1 MediaController.Callback

用于从MediaSession接收回调。使用方式如下：

```kotlin
private val connectionCallback = object : MediaBrowser.ConnectionCallback() {

    override fun onConnected() {
        super.onConnected()
        // ...
        if(mMediaBrowser.isConnected) {
            val sessionToken = mMediaBrowser.sessionToken
            mMediaController = MediaController(applicationContext,sessionToken)
            mMediaController.registerCallback(controllerCallback)
        }
    }
}

private val controllerCallback = object : MediaController.Callback() {

    override fun onAudioInfoChanged(info: MediaController.PlaybackInfo?) {
        super.onAudioInfoChanged(info)
    }

    override fun onExtrasChanged(extras: Bundle?) {
        super.onExtrasChanged(extras)
    }
    // ...
}
```

#### 2.2.2 MediaController.PlaybackInfo

保存有关当前播放以及如何处理此会话的音频的信息。使用方式如下：

```kotlin
// 获取当前回话播放的音频信息
val playbackInfo = mMediaController.playbackInfo
```

#### 2.2.3 MediaController.TransportControls

用于控制会话中媒体播放的接口。这允许客户端向Session发送媒体控制命令。使用方式如下：

```kotlin
private val connectionCallback = object : MediaBrowser.ConnectionCallback() {

    override fun onConnected() {
        super.onConnected()
        // ...
        if(mMediaBrowser.isConnected) {
            val sessionToken = mMediaBrowser.sessionToken
            mMediaController = MediaController(applicationContext,sessionToken)
            // 播放媒体
            mMediaController.transportControls.play()
            // 暂停媒体
            mMediaController.transportControls.pause()
        }
    }
}
```

### **2.3 服务端媒体浏览服务 - MediaBrowserService**

**媒体浏览器服务**，继承自`Service`，`MediaBrowserService`属于**服务端**，也是承载**播放器**（如MediaPlayer、ExoPlayer等）和`MediaSession`的容器。  
实现`MediaBrowserService`时会要求复写`onGetRoot`和`onLoadChildren`两个方法。  
`onGetRoot`通过的返回值决定是否允许客户端的`MediaBrowser`连接到`MediaBrowserService`。  
当客户端调用`MediaBrowser.subscribe`时会触发`onLoadChildren`方法。

```kotlin
const val FOLDERS_ID = "__FOLDERS__"
const val ARTISTS_ID = "__ARTISTS__"
const val ALBUMS_ID = "__ALBUMS__"
const val GENRES_ID = "__GENRES__"
const val ROOT_ID = "__ROOT__"

class MediaService : MediaBrowserService() {

    override fun onGetRoot(
        clientPackageName: String,
        clientUid: Int,
        rootHints: Bundle?
    ): BrowserRoot? {
        // 由MediaBrowser.connect触发，可以通过返回null拒绝客户端的连接。
        return BrowserRoot(ROOT_ID, null)
    }

    override fun onLoadChildren(
        parentId: String,
        result: Result<MutableList<MediaBrowser.MediaItem>>
    ) {
    // 由MediaBrowser.subscribe触发
        when (parentId) {
            ROOT_ID -> {
                // 查询本地媒体库
                // ...
                // 将此消息与当前线程分离，并允许稍后进行sendResult调用
                result.detach()
                // 设定到 result 中
                result.sendResult()
            }
            FOLDERS_ID -> {

            }
            ALBUMS_ID -> {

            }
            ARTISTS_ID -> {

            }
            GENRES_ID -> {

            }
            else -> {

            }
        }
    }
}
```

然后还需要在manifest中注册这个Service。

```xml
<service
  android:name=".MediaService"
  android:label="@string/service_name">
  <intent-filter>
    <action android:name="android.media.browse.MediaBrowserService" />
  </intent-filter>
</service>
```

#### 2.3.1 MediaBrowserService.BrowserRoot

包含**浏览器服务**首次连接时需要返回给**客户端**的信息。

**MediaBrowserService.BrowserRoot API 列表**

|   |   |
|---|---|
|方法名|备注|
|Bundle getExtras()|获取有关浏览器服务的附加信息。|
|String getRootId()|获取用于浏览的根 ID。|

#### 2.3.2 `MediaBrowserService.Result<T>`

包含**浏览器服务**返回给**客户端**的结果集。通过调用`sendResult`()将结果返回给调用方，但是在此之前需要调用`detach`()。

**MediaBrowserService.Result API 列表**

|   |   |
|---|---|
|方法名|备注|
|void detach()|将此消息与当前线程分离，并允许稍后进行调用sendResult(T)|
|void sendResult(T result)|将结果发送回调用方。|

### **2.4 服务端媒体会话 - MediaSession**

**媒体会话**，即**受控端。**通过设定`MediaSession.Callback`回调来接收媒体控制器`MediaController`发送的指令。  
创建`MediaSession`后还需要调用`setSessionToken`()方法设置用于和**控制器配对的令牌。使用方式如下：

```kotlin
const val FOLDERS_ID = "__FOLDERS__"
const val ARTISTS_ID = "__ARTISTS__"
const val ALBUMS_ID = "__ALBUMS__"
const val GENRES_ID = "__GENRES__"
const val ROOT_ID = "__ROOT__"

class MediaService : MediaBrowserService() {

    private lateinit var mediaSession: MediaSession;

    override fun onCreate() {
        super.onCreate()
        mediaSession = MediaSession(this, "TAG")
        mediaSession.setCallback(callback)
        sessionToken = mediaSession.sessionToken
    }

    // 与MediaController.transportControls中的大部分方法都是一一对应的
    // 在该方法中实现对 播放器 的控制，
    private val callback = object : MediaSession.Callback() {

        override fun onPlay() {
            super.onPlay()
            // 处理 播放器 的播放逻辑。
            // 车载应用的话，别忘了处理音频焦点
        }

        override fun onPause() {
            super.onPause()
        }

    }
    override fun onGetRoot(
            clientPackageName: String,
            clientUid: Int,
            rootHints: Bundle?
                ): BrowserRoot? {
            Log.e("TAG", "onGetRoot: $rootHints")
            return BrowserRoot(ROOT_ID, null)
        }

    override fun onLoadChildren(
        parentId: String,
        result: Result<MutableList<MediaBrowser.MediaItem>>
            ) {
        Log.e("TAG", "onLoadChildren: $parentId")
        result.detach()
        when (parentId) {
            ROOT_ID -> {
                result.sendResult(null)
            }
            FOLDERS_ID -> {

            }
            ALBUMS_ID -> {

            }
            ARTISTS_ID -> {

            }
            GENRES_ID -> {

            }
            else -> {

            }
        }
    }

    override fun onLoadItem(itemId: String?, result: Result<MediaBrowser.MediaItem>?) {
        super.onLoadItem(itemId, result)
        Log.e("TAG", "onLoadItem: $itemId")
    }
}
```

#### 2.4.1 MediaSession.Callback

接收来自控制器和系统的媒体按钮、传输控件和命令。与MediaController.transportControls中的大部分方法都是一一对应的。使用方式如下：

```kotlin
override fun onCreate() {
    super.onCreate()
    mediaSession = MediaSession(this, "TAG")
    mediaSession.setCallback(callback)
    sessionToken = mediaSession.sessionToken
}

// 与MediaController.transportControls中的方法是一一对应的。
// 在该方法中实现对 播放器 的控制，
private val callback = object : MediaSession.Callback() {

    override fun onPlay() {
        super.onPlay()
        // 处理 播放器 的播放逻辑。
        // 车载应用的话，别忘了处理音频焦点
        // ...
        if (!mediaSession.isActive) {
            mediaSession.isActive = true
        }
        // 更新播放状态.
        val state = PlaybackState.Builder()
            .setState(
                PlaybackState.STATE_PLAYING,1,1f
            )
            .build()
        // 此时MediaController.Callback.onPlaybackStateChanged会回调
        mediaSession.setPlaybackState(state)
    }

    override fun onPause() {
        super.onPause()
    }

    override fun onStop() {
        super.onStop()
    }

}
```

#### 2.4.2 MediaSession.QueueItem

作为播放队列一部分的单个项目。它包含队列中项目及其 ID 的说明。  
**MediaSession.QueueItem API 列表**

|   |   |
|---|---|
|方法名|备注|
|[MediaDescription](https://links.jianshu.com/go?to=https%3A%2F%2Fdeveloper.android.google.cn%2Freference%2Fandroid%2Fmedia%2FMediaDescription)<br><br>getDescription()|返回介质的说明。包含媒体的基础信息如：标题、封面等等。|
|long getQueueId()|获取此项目的队列 ID。|

#### 2.4.3 MediaSession.Token

表示正在进行的会话。这可以通过会话所有者传递给**客户端**，以允许客户端与服务端之间建立通信。

### **2.5 播放器状态 - PlaybackState**

用于承载播放状态的类。如当前播放位置和当前控制功能。  
在`MediaSession.Callback`更改状态后需要调用`MediaSession.setPlaybackState`把状态同步给**客户端**。使用方式如下：

```kotlin
private val callback = object : MediaSession.Callback() {

    override fun onPlay() {
        super.onPlay()
        // ...
        // 更新状态
        val state = PlaybackState.Builder()
            .setState(
                PlaybackState.STATE_PLAYING,1,1f
            )
            .build()
        mediaSession.setPlaybackState(state)
    }
}
```

#### 2.5.1 PlaybackState.Builder

基于建造者模式来生成PlaybackState对象。使用方式如下：

```Java
PlaybackState state = new PlaybackState.Builder()
    .setState(PlaybackState.STATE_PLAYING,
              mMediaPlayer.getCurrentPosition(), PLAYBACK_SPEED)
    .setActions(PLAYING_ACTIONS)
    .addCustomAction(mShuffle)
    .setActiveQueueItemId(mQueue.get(mCurrentQueueIdx).getQueueId())
    .build();
```

#### 2.5.2 PlaybackState.CustomAction

`CustomActions`可用于通过将特定于应用程序的操作发送给`MediaControllers`，这样就可以扩展标准传输控件的功能。使用方式如下：

```Java
CustomAction action = new CustomAction
    .Builder("android.car.media.localmediaplayer.shuffle",
             mContext.getString(R.string.shuffle),
             R.drawable.shuffle)
    .build();

PlaybackState state = new PlaybackState.Builder()
    .setState(PlaybackState.STATE_PLAYING,
              mMediaPlayer.getCurrentPosition(), PLAYBACK_SPEED)
    .setActions(PLAYING_ACTIONS)
    .addCustomAction(action)
    .setActiveQueueItemId(mQueue.get(mCurrentQueueIdx).getQueueId())
    .build();
```

**PlaybackState.CustomAction API 说明**

|   |   |
|---|---|
|方法名|备注|
|String getAction()|返回CustomAction的action。|
|Bundle getExtras()|返回附加项，这些附加项提供有关操作的其他特定于应用程序的信息，如果没有，则返回 null。|
|int getIcon()|返回package中图标的资源 ID。|
|CharSequence getName()|返回此操作的显示名称。|

### **2.6 元数据类 - MediaMetadata**

包含有关项目的基础数据，例如标题、艺术家等。一般需要**服务端**从本地数据库或远端查询出原始数据在封装成MediaMetadata再通过MediaSession.setMetadata(metadata)返回到**客户端**的MediaController.Callback.onMetadataChanged中。

**MediaMetadata API 说明**

|   |   |
|---|---|
|方法名|备注|
|boolean containsKey(String key)|如果给定的key包含在元数据中，则返回 true|
|int describeContents()|描述此可打包实例的封送处理表示中包含的特殊对象的种类。|
|Bitmap getBitmap(String key)|返回给定的key的Bitmap;如果给定key不存在位图，则返回 null。|
|int getBitmapDimensionLimit()|获取创建此元数据时位图的宽度/高度限制（以像素为单位）。|
|MediaDescription getDescription()|获取此元数据的简单说明以进行显示。|
|long getLong(String key)|返回与给定key关联的值，如果给定key不再存在，则返回 0L。|
|Rating getRating(String key)|对于给定的key返回Rating;如果给定key不存在Rating，则返回 null。|
|String getString(String key)|以 String 格式返回与给定key关联的文本值，如果给定key不存在所需类型的映射，或者null值显式与该key关联，则返回 null。|
|CharSequence getText(String key)|返回与给定键关联的值，如果给定键不存在所需类型的映射，或者与该键显式关联 null 值，则返回 null。|
|`Set<String>` keySet()|返回一个 Set，其中包含在此元数据中用作key的字符串。|
|int size()|返回此元数据中的字段数。|

**MediaMetadata 常用Key**

|   |   |
|---|---|
|方法名|备注|
|METADATA_KEY_ALBUM|媒体的唱片集标题。|
|METADATA_KEY_ALBUM_ART|媒体原始来源的相册的插图，Bitmap格式|
|METADATA_KEY_ALBUM_ARTIST|媒体原始来源的专辑的艺术家。|
|METADATA_KEY_ALBUM_ART_URI|媒体原始源的相册的图稿，Uri格式（推荐使用）|
|METADATA_KEY_ART|媒体封面，Bitmap格式|
|METADATA_KEY_ART_URI|媒体的封面，Uri格式。|
|METADATA_KEY_ARTIST|媒体的艺术家。|
|METADATA_KEY_AUTHOR|媒体的作者。|
|METADATA_KEY_BT_FOLDER_TYPE|蓝牙 AVRCP 1.5 的 6.10.2.2 节中指定的媒体的蓝牙文件夹类型。|
|METADATA_KEY_COMPILATION|媒体的编译状态。|
|METADATA_KEY_COMPOSER|媒体的作曲家。|
|METADATA_KEY_DATE|媒体的创建或发布日期。|
|METADATA_KEY_DISC_NUMBER|介质原始来源的光盘编号。|
|METADATA_KEY_DISPLAY_DESCRIPTION|适合向用户显示的说明。|
|METADATA_KEY_DISPLAY_ICON|适合向用户显示的图标或缩略图。|
|METADATA_KEY_DISPLAY_ICON_URI|适合向用户显示的图标或缩略图， Uri格式。|
|METADATA_KEY_DISPLAY_SUBTITLE|适合向用户显示的副标题。|
|METADATA_KEY_DISPLAY_TITLE|适合向用户显示的标题。|
|METADATA_KEY_DURATION|媒体的持续时间（以毫秒为单位）。|
|METADATA_KEY_GENRE|媒体的流派。|
|METADATA_KEY_MEDIA_ID|用于标识内容的字符串Key。|
|METADATA_KEY_MEDIA_URI|媒体内容，Uri格式。|
|METADATA_KEY_NUM_TRACKS|媒体原始源中的曲目数。|
|METADATA_KEY_RATING|媒体的总体评分。|
|METADATA_KEY_TITLE|媒体的标题。|
|METADATA_KEY_TRACK_NUMBER|媒体的磁道编号。|
|METADATA_KEY_USER_RATING|用户对媒体的分级。|
|METADATA_KEY_WRITER|媒体作家。|
|String METADATA_KEY_YEAR|媒体创建或发布为长的年份。|

### 3. MediaSession 简单实践

MediaSession 框架核心类通信过程如下图所示。

![](https://cdn.nlark.com/yuque/0/2022/webp/26044650/1670161616894-047c19d2-031e-4aa1-a841-be7a592bf2a2.webp)

客户端源码如下所示：

```kotlin
class MainActivity : AppCompatActivity() {

    private lateinit var mMediaBrowser: MediaBrowser
    private lateinit var mMediaController: MediaController

    @RequiresApi(Build.VERSION_CODES.M)
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val component = ComponentName(this, MediaService::class.java)
        mMediaBrowser = MediaBrowser(this, component, connectionCallback, null);
        // 连接到MediaBrowserService，会触发MediaBrowserService的onGetRoot方法。
        mMediaBrowser.connect()

        findViewById<Button>(R.id.btn_play).setOnClickListener {
            mMediaController.transportControls.play()
        }
    }

    private val connectionCallback = object : MediaBrowser.ConnectionCallback() {

        override fun onConnected() {
            super.onConnected()
            if (mMediaBrowser.isConnected) {
                val sessionToken = mMediaBrowser.sessionToken
                mMediaController = MediaController(applicationContext, sessionToken)
                mMediaController.registerCallback(controllerCallback)
                // 获取根mediaId
                val rootMediaId = mMediaBrowser.root
                // 获取根mediaId的item列表，会触发MediaBrowserService.onLoadItem方法
                mMediaBrowser.getItem(rootMediaId,itemCallback)
                mMediaBrowser.unsubscribe(rootMediaId)
                // 订阅服务端 media item的改变，会触发MediaBrowserService.onLoadChildren方法
                mMediaBrowser.subscribe(rootMediaId, subscribeCallback)
            }
        }
    }

    private val controllerCallback = object : MediaController.Callback() {

        override fun onPlaybackStateChanged(state: PlaybackState?) {
            super.onPlaybackStateChanged(state)
            Log.d("TAG", "onPlaybackStateChanged: $state")
            when(state?.state){
                PlaybackState.STATE_PLAYING ->{
                    // 处理UI
                }
                PlaybackState.STATE_PAUSED ->{
                    // 处理UI
                }
                // 还有其它状态需要处理
            }
        }

        // 音频信息，音量
        override fun onAudioInfoChanged(info: MediaController.PlaybackInfo?) {
            super.onAudioInfoChanged(info)
            val currentVolume = info?.currentVolume
            // 显示在UI上
        }

        override fun onMetadataChanged(metadata: MediaMetadata?) {
            super.onMetadataChanged(metadata)
            val artUri = metadata?.getString(MediaMetadata.METADATA_KEY_ALBUM_ART_URI)
            // 显示UI上
        }

        override fun onSessionEvent(event: String, extras: Bundle?) {
            super.onSessionEvent(event, extras)
            Log.d("TAG", "onSessionEvent: $event")
        }
        // ...
    }

    private val subscribeCallback = object : MediaBrowser.SubscriptionCallback() {
        override fun onChildrenLoaded(
            parentId: String,
            children: MutableList<MediaBrowser.MediaItem>
                ) {
            super.onChildrenLoaded(parentId, children)
        }

        override fun onChildrenLoaded(
            parentId: String,
            children: MutableList<MediaBrowser.MediaItem>,
            options: Bundle
        ) {
            super.onChildrenLoaded(parentId, children, options)
        }

        override fun onError(parentId: String) {
            super.onError(parentId)
        }
    }

    private val itemCallback = object : MediaBrowser.ItemCallback() {

        override fun onItemLoaded(item: MediaBrowser.MediaItem?) {
            super.onItemLoaded(item)
        }

        override fun onError(mediaId: String) {
            super.onError(mediaId)
        }
    }
}
```

服务端源码如下所示：

```kotlin

const val FOLDERS_ID = "__FOLDERS__"
const val ARTISTS_ID = "__ARTISTS__"
const val ALBUMS_ID = "__ALBUMS__"
const val GENRES_ID = "__GENRES__"
const val ROOT_ID = "__ROOT__"

class MediaService : MediaBrowserService() {

    // 控制是否允许客户端连接，并返回root media id给客户端
    override fun onGetRoot(
        clientPackageName: String,
        clientUid: Int,
        rootHints: Bundle?
    ): BrowserRoot? {
        Log.e("TAG", "onGetRoot: $rootHints")
        return BrowserRoot(ROOT_ID, null)
    }

    // 处理客户端的订阅信息
    override fun onLoadChildren(
        parentId: String,
        result: Result<MutableList<MediaBrowser.MediaItem>>
    ) {
        Log.e("TAG", "onLoadChildren: $parentId")
        result.detach()
        when (parentId) {
            ROOT_ID -> {
                result.sendResult(null)
            }
            FOLDERS_ID -> {

            }
            ALBUMS_ID -> {

            }
            ARTISTS_ID -> {

            }
            GENRES_ID -> {

            }
            else -> {

            }
        }
    }

    override fun onLoadItem(itemId: String?, result: Result<MediaBrowser.MediaItem>?) {
        super.onLoadItem(itemId, result)
        Log.e("TAG", "onLoadItem: $itemId")
        // 根据itemId，返回对用MediaItem
        result?.detach()
        result?.sendResult(null)
    }

    private lateinit var mediaSession: MediaSession;

    override fun onCreate() {
        super.onCreate()
        mediaSession = MediaSession(this, "TAG")
        mediaSession.setCallback(callback)
        // 设置token
        sessionToken = mediaSession.sessionToken
    }

    // 与MediaController.transportControls中的方法是一一对应的。
    // 在该方法中实现对 播放器 的控制，
    private val callback = object : MediaSession.Callback() {

        override fun onPlay() {
            super.onPlay()
            // 处理 播放器 的播放逻辑。
            // 车载应用的话，别忘了处理音频焦点
            Log.e("TAG", "onPlay:")
            if (!mediaSession.isActive) {
                mediaSession.isActive = true
            }
            // 更新状态
            val state = PlaybackState.Builder()
                .setState(
                    PlaybackState.STATE_PLAYING, 1, 1f
                )
                .build()
            mediaSession.setPlaybackState(state)
        }

        override fun onPause() {
            super.onPause()
        }

        override fun onStop() {
            super.onStop()
        }

        // 还有其它方法需要复写
    }
}
```

上述的代码只是帮助理解MediaSession框架的通信过程，本身的功能非常的简陋。上一篇[Android车载应用开发与分析（6）- 车载多媒体（一）- 音视频基础知识与MediaPlayer](https://www.jianshu.com/p/232dd12e35cb)中介绍了音视频的基础知识和MediaPlayer的生命周期，再通过本篇了解了MediaSession框架的基础使用，下一篇我们就可以开始解析车载Android中的原生LocalMedia应用了。

## 4. MediaSession API 列表

### 4.1 MediaBrowser 相关组件 API 列表

#### 4.1.1 MediaBrowser

|   |   |
|---|---|
|方法名|备注|
|void connect()|连接到媒体浏览器服务。|
|void disconnect()|断开与媒体浏览器服务的连接。|
|Bundle getExtras()|获取介质服务的任何附加信息。|
|void getItem(String mediaId, MediaBrowser.ItemCallback cb)|从连接的服务中检索特定的MediaItem|
|String getRoot()|获取根ID。|
|ComponentName getServiceComponent()|获取媒体浏览器连接到的服务组件。|
|MediaSession.Token getSessionToken()|获取与媒体浏览器关联的媒体会话Token。|
|boolean isConnected()|返回浏览器是否连接到服务。|
|void subscribe(String parentId,Bundle options, MediaBrowser.SubscriptionCallback callback)|使用特定于服务的参数进行查询，以获取有关指定 ID 中包含的媒体项的信息，并订阅以在更新更改时接收更新。|
|void subscribe(String parentId, MediaBrowser.SubscriptionCallback callback)|询有关包含在指定 ID 中的媒体项的信息，并订阅以在更改时接收更新。|
|void unsubscribe(String parentId)|取消订阅指定媒体 ID 。|
|void unsubscribe(String parentId, MediaBrowser.SubscriptionCallback callback)|通过回调取消订阅对指定媒体 ID。|

#### 4.1.2 MediaBrowser.ConnectionCallback

|   |   |
|---|---|
|方法|备注|
|onConnected()|与MediaBrowserService连接成功。在调用`MediaBrowser.connect()`后才会有回调。|
|onConnectionFailed()|与MediaBrowserService连接失败。|
|onConnectionSuspended()|与MediaBrowserService连接断开。|

#### 4.1.3 MediaBrowser. ItemCallback

|   |   |
|---|---|
|方法名|备注|
|onError(String mediaId)|检索时出错，或者连接的服务不支持时回调。|
|onItemLoaded(MediaBrowser.MediaItem item)|返回Item时调用。|

#### 4.1.4 MediaBrowser. MediaItem

|   |   |
|---|---|
|方法名|备注|
|int describeContents()|描述此可打包实例的封送处理表示中包含的特殊对象的种类。|
|[MediaDescription](https://links.jianshu.com/go?to=https%3A%2F%2Fdeveloper.android.google.cn%2Freference%2Fandroid%2Fmedia%2FMediaDescription)<br><br>getDescription()|获取介质的说明。包含媒体的基础信息如：标题、封面等等。|
|int getFlags()|获取项的标志。`FLAG_BROWSABLE`：表示Item具有自己的子项。`FLAG_PLAYABLE`：表示Item可播放|
|String getMediaId()|返回此项的媒体 ID。|
|boolean isBrowsable()|返回此项目是否可浏览。|
|boolean isPlayable()|返回此项是否可播放。|

#### 4.1.5 MediaBrowser.SubscriptionCallback

|   |   |
|---|---|
|方法名|备注|
|onChildrenLoaded(String parentId, List<MediaBrowser.MediaItem> children)|在加载或更新子项列表时回调。|
|onChildrenLoaded(String parentId, List<MediaBrowser.MediaItem> children,Bundle options)|在加载或更新子项列表时回调。|
|onError(String parentId)|当 ID 不存在或订阅时出现其他错误时回调。|
|onError(String parentId, Bundle options)|当 ID 不存在或订阅时出现其他错误时回调。|

### 4.2 MediaController 相关组件 API 列表

#### 4.2.1 MediaController

|   |   |
|---|---|
|方法名|备注|
|void adjustVolume (int direction, int flags)|调整此会话正在播放的输出的音量。|
|boolean dispatchMediaButtonEvent (KeyEvent keyEvent)|将指定的媒体按钮事件发送到会话。|
|Bundle getExtras()|获取此会话的附加内容。|
|long getFlags()|获取此会话的标志。|
|MediaMetadata getMetadata()|获取此会话的当前Metadata。|
|String getPackageName()|获取会话所有者的程序包名称。|
|MediaController.PlaybackInfo getPlaybackInfo()|获取此会话的当前播放信息。|
|PlaybackState getPlaybackState()|获取此会话的当前播放状态。|
|List<MediaSession.QueueItem> getQueue()|获取此会话的当前播放队列（如果已设置）。|
|CharSequence getQueueTitle()|获取此会话的队列标题。|
|int getRatingType()|获取会话支持的评级类型。|
|PendingIntent getSessionActivity()|获取启动与此会话关联的 UI 的意图（如果存在）。|
|Bundle getSessionInfo()|获取创建会话时设置的其他会话信息。|
|MediaSession.Token getSessionToken()|获取连接到的会话的令牌。|
|String getTag()|获取会话的标记以进行调试。|
|MediaController.TransportControls getTransportControls()|获取TransportControls实例以将控制操作发送到关联的会话。|
|void registerCallback (MediaController.Callback callback, Handler handler)|注册回调以从会话接收更新。|
|void registerCallback (MediaController.Callback callback)|注册回调以从会话接收更新。|
|void sendCommand (String command, Bundle args, ResultReceiver cb)|向会话发送通用命令。|
|void setVolumeTo (int value, int flags)|设置此会话正在播放的输出的音量。|
|void unregisterCallback (MediaController.Callback callback)|注销指定的回调。|

#### 4.2.2 MediaController.Callback

|   |   |
|---|---|
|方法名|备注|
|void onAudioInfoChanged (MediaController.PlaybackInfo info)|当前音频信息发生改变。|
|void onExtrasChanged (Bundle extras)|当前附加内容发生改变。|
|void onMetadataChanged (MediaMetadata metadata)|当前Metadata发生改变。|
|void onPlaybackStateChanged(PlaybackState state)|当前播放状态发生改变。客户端通过该回调来显示界面上音视频的播放状态。|
|void onQueueChanged (List<MediaSession.QueueItem> queue)|当前队列中项目发生改变。|
|void onQueueTitleChanged (CharSequence title)|当前队列标题发生改变。|
|void onSessionDestroyed()|会话销毁。|
|void onSessionEvent (String event, Bundle extras)|MediaSession所有者发送的自定义事件。|

#### 4.2.3 MediaController. PlaybackInfo

|   |   |
|---|---|
|方法名|备注|
|AudioAttributes getAudioAttributes()|获取此会话的音频属性。|
|int getCurrentVolume()|获取此会话的当前音量。|
|int getMaxVolume()|获取可为此会话设置的最大音量。|
|int getPlaybackType()|获取影响音量处理的播放类型。|
|int getVolumeControl()|获取可以使用的音量控件的类型。|
|String getVolumeControlId()|获取此会话的音量控制 ID。|

#### 4.2.4 MediaController. TransportControls

|   |   |
|---|---|
|方法名|备注|
|void fastForward()|开始快进。|
|void pause()|请求播放器暂停播放并保持在当前位置。|
|void play()|请求播放器在其当前位置开始播放。|
|void playFromMediaId (String mediaId, Bundle extras)|请求播放器开始播放特定媒体 ID。|
|void playFromSearch (String query, Bundle extras)|请求播放器开始播放特定的搜索查询。|
|void playFromUri (Uri uri, Bundle extras)|请求播放器开始播放特定Uri。|
|void prepare()|请求播放器准备播放。|
|void prepareFromMediaId (String mediaId, Bundle extras)|请求播放器为特定媒体 ID 准备播放。|
|void prepareFromSearch (String query, Bundle extras)|请求播放器为特定搜索查询准备播放。|
|void prepareFromUri (Uri uri, Bundle extras)|请求播放器为特定Uri。|
|void rewind()|开始倒带。|
|void seekTo(long pos)|移动到媒体流中的新位置。|
|void sendCustomAction (PlaybackState.CustomAction customAction, Bundle args)|发送自定义操作以供MediaSession执行。|
|void sendCustomAction (String action,Bundle args)|将自定义操作中的 id 和 args 发送回去，以便MediaSession执行。|
|void setPlaybackSpeed (float speed)|设置播放速度。|
|void setRating(Rating rating)|对当前内容进行评级。|
|void skipToNext()|跳到下一项。|
|void skipToPrevious()|跳到上一项。|
|void skipToQueueItem(long id)|在播放队列中播放具有特定 ID 的项目。|
|void stop()|请求播放器停止播放;它可以以任何适当的方式清除其状态。|

### 4.3 MediaBrowserService 相关组件 API 列表

#### 4.3.1 MediaBrowserService

|   |   |
|---|---|
|方法名|备注|
|final Bundle getBrowserRootHints()|获取从当前连接 `MediaBrowser`的发送的根提示。|
|final MediaSessionManager.RemoteUserInfo getCurrentBrowserInfo()|获取发送当前请求的浏览器信息。|
|MediaSession.Token getSessionToken()|获取会话令牌，如果尚未创建会话令牌或已销毁会话令牌，则获取 null。|
|void notifyChildrenChanged(String parentId)|通知所有连接的媒体浏览器指定父 ID 的子级已经更改。|
|void notifyChildrenChanged(String parentId, Bundle options)|通知所有连接的媒体浏览器指定父 ID 的子级已经更改。|
|abstract MediaBrowserService.BrowserRoot onGetRoot(String clientPackageName,int clientUid, Bundle rootHints)|获取供特定客户端浏览的根信息。由`MediaBrowser.connect`触发，可以通过返回null拒绝客户端的连接。|
|abstract void onLoadChildren(String parentId, Result<List<MediaBrowser.MediaItem>> result)|获取有关媒体项的子项的信息。由`MediaBrowser.subscribe`触发。|
|void onLoadChildren(String parentId, Result<List<MediaBrowser.MediaItem>> result,Bundle options)|获取有关媒体项的子项的信息。由`MediaBrowser.subscrib`e触发。|
|void onLoadItem(String itemId, Result<MediaBrowser.MediaItem> result)|获取有关特定媒体项的信息。由`MediaBrowser.getItem`触发。|
|void setSessionToken(MediaSession.Token token)|设置媒体会话。|

#### 4.3.2 MediaBrowserService.BrowserRoot

|   |   |
|---|---|
|方法名|备注|
|Bundle getExtras()|获取有关浏览器服务的附加信息。|
|String getRootId()|获取用于浏览的根 ID。|

#### 4.3.3 MediaBrowserService.Result

|   |   |
|---|---|
|方法名|备注|
|void detach()|将此消息与当前线程分离，并允许稍后进行调用sendResult(T)|
|void sendResult(T result)|将结果发送回调用方。|

### 4.4 MediaSession 相关组件 API 列表

#### 4.4.1 MediaSession

|   |   |
|---|---|
|方法名|备注|
|MediaController getController()|获取此会话的控制器。|
|MediaSessionManager.RemoteUserInfo getCurrentControllerInfo()|获取发送当前请求的控制器信息。|
|MediaSession.Token getSessionToken()|获取此会话令牌对象。|
|boolean isActive()|获取此会话的当前活动状态。|
|void release()|当应用完成播放时，必须调用此项。|
|void sendSessionEvent (String event, Bundle extras)|将专有事件发送给监听此会话的所有MediaController。会触发MediaController.Callback.onSessionEvent。|
|void setActive(boolean active)|设置此会话当前是否处于活动状态并准备好接收命令。|
|void setCallback (MediaSession.Callback callback)|设置回调以接收媒体会话的更新。|
|void setCallback (MediaSession.Callback callback,Handler handler)|设置回调以接收媒体会话的更新。|
|void setExtras(Bundle extras)|设置一些可与MediaSession关联的附加功能。|
|void setFlags(int flags)|为会话设置标志。|
|void setMediaButtonBroadcastReceiver(ComponentName broadcastReceiver)|设置应接收媒体按钮的清单声明类的组件名称。|
|void setMediaButtonReceiver(PendingIntent mbr)|此方法在 API 级别 31 中已弃用。改用setMediaButtonBroadcastReceiver（android.content.ComponentName）。|
|void setMetadata(MediaMetadata metadata)|更新当前MediaMetadata。|
|void setPlaybackState(PlaybackState state)|更新当前播放状态。|
|void setPlaybackToLocal(AudioAttributes attributes)|设置此会话音频的属性。|
|void setPlaybackToRemote(VolumeProvider volumeProvider)|将此会话配置为使用远程音量处理。|
|void setQueue(List<MediaSession.QueueItem> queue)|更新播放队列中的项目列表。|
|void setQueueTitle(CharSequence title)|设置播放队列的标题。|
|void setRatingType(int type)|设置此会话使用的评级样式。|
|void setSessionActivity(PendingIntent pi)|设置启动此会话的Activity的Intent。|

#### 4.4.2 MediaSession.Callback

|   |   |
|---|---|
|方法名|备注|
|void onCommand(String command,Bundle args,ResultReceiver cb)|当控制器已向此会话发送命令时调用。|
|void onCustomAction(String action, Bundle extras)|当要执行MediaControllerPlaybackState.CustomAction时调用。|
|void onFastForward()|处理快进请求。|
|boolean onMediaButtonEvent(Intent mediaButtonIntent)|当按下媒体按钮并且此会话具有最高优先级或控制器向会话发送媒体按钮事件时调用。|
|void onPause()|处理暂停播放的请求。|
|void onPlay()|处理开始播放的请求。|
|void onPlayFromMediaId(String mediaId, Bundle extras)|处理播放应用提供的特定mediaId的播放请求。|
|void onPlayFromSearch(String query, Bundle extras)|处理从搜索查询开始播放的请求。|
|void onPlayFromUri(Uri uri, Bundle extras)|处理播放由URI表示的特定媒体项的请求。|
|void onPrepare()|处理准备播放的请求。|
|void onPrepareFromMediaId(String mediaId, Bundle extras)|处理应用提供的特定mediaId的准备播放请求|
|void onPrepareFromSearch(String query, Bundle extras)|处理准备从搜索查询播放的请求。|
|void onPrepareFromUri(Uri uri, Bundle extras)|处理由URI表示的特定媒体项的准备请求。|
|void onRewind()|处理倒带请求。|
|void onSeekTo(long pos)|处理跳转到特定位置的请求。|
|void onSetPlaybackSpeed(float speed)|处理修改播放速度的请求。|
|void onSetRating(Rating rating)|处理设定评级的请求。|
|void onSkipToNext()|处理要跳到下一个媒体项的请求。|
|void onSkipToPrevious()|处理要跳到上一个媒体项的请求。|
|void onSkipToQueueItem(long id)|处理跳转到播放队列中具有给定 ID 的项目的请求。|
|void onStop()|处理停止播放的请求。|

#### 4.4.3 MediaSession.QueueItem

|   |   |
|---|---|
|方法名|备注|
|[MediaDescription](https://links.jianshu.com/go?to=https%3A%2F%2Fdeveloper.android.google.cn%2Freference%2Fandroid%2Fmedia%2FMediaDescription)<br><br>getDescription()|返回介质的说明。包含媒体的基础信息如：标题、封面等等。|
|long getQueueId()|获取此项目的队列 ID。|

### 4.5 PlaybackState 相关组件 API 列表

#### 4.5.1 PlaybackState

|   |   |
|---|---|
|方法名|备注|
|long getActions()|获取此会话上可用的当前操作。|
|long getActiveQueueItemId()|获取队列中当前活动项的 ID。|
|long getBufferedPosition()|获取当前缓冲位置（以毫秒为单位）。|
|List<PlaybackState.CustomAction> getCustomActions()|获取自定义操作的列表。|
|CharSequence getErrorMessage()|获取用户可读的错误消息。|
|Bundle getExtras()|获取在此播放状态下设置的任何自定义附加内容。|
|long getLastPositionUpdateTime()|获取上次更新位置的经过的实时时间。|
|float getPlaybackSpeed()|获取当前播放速度作为正常播放的倍数。|
|long getPosition()|获取当前播放位置（以毫秒为单位）。|
|int getState()|获取当前播放状态。|
|boolean isActive()|返回是否将其视为活动播放状态。|

#### 4.5.2 PlaybackState.Builder

|   |   |
|---|---|
|方法名|备注|
|PlaybackState.Builder addCustomAction(String action, String name, int icon)|将自定义操作添加到播放状态。|
|PlaybackState.Builder addCustomAction (PlaybackState.CustomAction customAction)|将自定义操作添加到播放状态。|
|PlaybackState.Builder setActions(long actions)|设置此会话上可用的当前操作。|
|PlaybackState.Builder setActiveQueueItemId(long id)|通过指定活动项目的 id 来设置播放队列中的活动项目。|
|PlaybackState.Builder setBufferedPosition(long bufferedPosition)|设置当前缓冲位置（以毫秒为单位）。|
|PlaybackState.Builder setErrorMessage(CharSequence error)|设置用户可读的错误消息。|
|PlaybackState.Builder setExtras(Bundle extras)|设置要包含在播放状态中的任何自定义附加内容。|
|PlaybackState.Builder setState(int state, long position, float playbackSpeed)|设置当前播放状态。|
|PlaybackState.Builder setState(int state, long position, float playbackSpeed, long updateTime)|设置当前播放状态。|
|PlaybackState build()|生成并返回具有这些值的PlaybackState实例。|

#### 4.5.3 PlaybackState.CustomAction

|   |   |
|---|---|
|方法名|备注|
|String getAction()|返回CustomAction的action。|
|Bundle getExtras()|返回附加项，这些附加项提供有关操作的其他特定于应用程序的信息，如果没有，则返回 null。|
|int getIcon()|返回package中图标的资源 ID。|
|CharSequence getName()|返回此操作的显示名称。|

  
  
作者：林栩  
链接：[https://www.jianshu.com/p/41d65afd8069](https://www.jianshu.com/p/41d65afd8069)  
来源：简书  
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。