---
author: zjmantou
title: 未命名
time: 2024-08-19 周一
tags:
  - 资料
  - Android
  - 单元测试
---
今天想写一篇文章，讲一讲 Android 开发中的单元测试。

我相信我们大部分同学手上的项目工程目录，点开之后，多少都会都这么两个文件夹，一个 `androidTest`，一个 `test`。说实话，我个人以前对 Android 单元测试这块也是知之甚少，然而我敢说，国内大部分公司估计也都不太注重单元测试，换句话说，几乎没多少开发人员会往这两个文件夹里写代码。你所在的公司，你手上的项目有没有单元测试？欢迎大家打在公屏……哦不，评论区。

![image.png](file:///Users/mantou/.config/joplin-desktop/resources/9255e33049cb4c5c82bc683b057a7bfc.webp)

## 什么是单元测试？

有些同学看到这里紧张了，以为我要开始搬定义了。然而并非如此，接下来我要分享的所有都是我自己的心路历程，也是我自己对单元测试从完全不 care 到至少入门的过程。

所谓“单元测试”，顾名思义就是对软件的一个个最小单元进行模块化测试。在 [《给安卓开发小白们的unit test指南 - 这也能测？这也要测？》](https://juejin.cn/user/1468603264932014/posts "https://juejin.cn/user/1468603264932014/posts") 这篇文章里，阿庆哥给出了一张软件工程金字塔结构图，可以清楚看到单元测试直接位于这座金字塔的最底层！常言道万丈高楼平地起，可见如果单元测试做得很稳健的话，对于整个软件项目的稳健性一定是收益最大的。

好像开始逐渐晦涩了？没关系，拉回来继续说。

我在前面提到“软件的一个个最小单元”，这个怎么理解？很简单，你项目里的每一个工具类（`***Utils.kt`）、 `***Presenter.kt`、`***Controller.kt`、`***Repository.kt`、`***Dao.kt`，这些都可以被看成一个个最小单元。除此之外，再往上来到 UI 层，每一个 `Activity`、`Fragment`、`View`、`ViewModel`，这些也可以被看成一个个最小单元。

看到这里，你已经对“最小单元”有了一个基本概念了，但心里应该还是有点犯嘀咕，前面提到的这些“最小单元”，每个里面少说都有好几个方法，有些甚至几百上千行代码，能不能把“最小单元”细化到每个类里面的方法呢？

恭喜你已经学会抢答了，如果你的思维能跟到这里，那么后面你看起来将毫不费力。因为所谓的单元测试，就是对这些“最小单元”里面的方法，去一个个进行测试，而这些测试，被称为一条条 `test case`（测试用例）。

## 为什么需要单元测试？

对于这个问题，我自己都在内心问过我无数次，因为传统观念里，尤其对于我们 Android 开发而言，把业务需求完成，拿手点一点，最多再考虑一下边界情况，符合预期，不崩溃，似乎就可以提交代码了。然而事实真的如此吗？

我想到一个很好的例子来解释这个问题，以下是一个真实的案例：

> 假设有一个需求：`displayName` 为系统版本名称，是一个 `String`，例如 `"Android OS 14"` `"Android OS 15"`，并且运营同学拍着胸脯告诉你，`"Android OS "`开头一定不会变，要求 app 根据 Android 版本 `14`，还是 `15`，这样类似的，来做出逻辑上的区分，换句话说，你要解析出 `displayName` 里面`14`、`15` 这样的整数型。

于是你的项目里一定会多出工具类 `VersionUtils`，里面有一个 `parseAndroidVersion(displayName: String)` 方法来实现这个需求，代码如下：

```
object VersionUtils {  
  
    fun parseAndroidVersion(displayName: String): Int {  
        val regex = "\\b(\\d+)\\b"  
        val pattern = Pattern.compile(regex)  
        val matcher = pattern.matcher(displayName)  
        // 查找匹配  
        return if (matcher.find()) {  
            // 将匹配到的字符串转换为整数并返回  
            Integer.parseInt(matcher.group(1));  
        } else {  
            // 未找到匹配，返回默认值或抛出异常，这里返回 -1 作为默认值  
            -1;  
        }  
    }
  
}
```

有一天，组里来了一个新同事，一看这个方法，他感觉太复杂了，这么小一个需求，还把正则搬出来呢，有这个必要？何况运营都说了`"Android OS "`开头肯定不会变，那不直接 `displayName.split("Android OS ")`劈开，再把第二个结果 `parseInt` 不就好了？于是他一顿操作，方法被“优化”成只有几行：

```
object VersionUtils {

    fun parseAndroidVersion(displayName: String): Int {  
        val s = displayName.split("Android OS ")  
        return try {  
            Integer.parseInt(s[1])  
        } catch (e: NumberFormatException) {  
            -1  
        }
    }
  
}
```

一个如此复杂的方法，被优化成了如此“清晰”，可读性还好，甚至还考虑到了万一 `s[1]` 不是一个整数的情况，`try-catch` 了肯定也不会崩溃，新同事很得意，刚入职就优化了项目里的代码，他很开心。

万万没想到的是，上线之后没多久就出事了。快过年了，运营同学可能是想祝大家新年快乐，编了一版固件推了出去， `displayName` 叫 `"Android OS 14（新春特别版）"`。没过多久，用户升上来了，线上的版本解析结果一下子全变成了 `-1`，业务匹配不上了，埋点的数据版本号也不对了，大年三十晚上，这个同事的电话被打爆，后面紧急回退了这个“优化”，再重新发版……

他有点不开心，开始抱怨运营坑自己，也抱怨测试为什么没测出来。

运营说自己是无辜的，这锅不背，他确实做到了 `"Android OS "`开头肯定不会变，但确实也不知道后面不能加东西，他觉得是开发写的代码不够健壮。

测试也说自己是无辜的，因为测试当时拿到这个变更的时候，觉得这么小的一个改动，肯定不会有什么影响，于是随便拿 `"Android OS 14"` `"Android OS 15"` 两个固件，试了下能解析出 `14`、`15`，就验证通过了。

## 有单元测试会怎样？

上面这个场景是我曾经经历过的真实案例。

我们平时在开发过程中，经常遇到这种发版的时候信心满满，发出去之后才发现“项目被负优化了”，又或者“按下葫芦浮起瓢”的情况。每次遇到的时候，开发解释说“我没想到还有 xxx 场景”，测试解释说“我们想到这个改动会影响到 xxx 场景，所以没测到”。

都说自己没想到，那么有没有一个办法，让程序来代替人工去想到这些场景呢？

回到上面这个案例，让我们来看看如果有单元测试，会是什么情况。

```
@RunWith(RobolectricTestRunner::class) 
class VersionUtilsTest { 
    @Test 
    fun testParseAndroidVersion() { 
        // 正常场景
        val version = VersionUtils.parseAndroidVersion("Android OS 14") 
        assertEquals(14, version) 
        
        // 有后缀场景
        val version2 = VersionUtils.parseAndroidVersion("Android OS 14（某某版）") 
        assertEquals(14, version2) 
   } 
}
```

假设最开始写 `VersionUtils.kt` 的开发同学，在写这个工具类的时候，顺手提交了 `VersionUtilsTest.kt` 类，并且在 `testParseAndroidVersion()` 方法里对 `parseAndroidVersion()` 方法进行了测试。

测试包含 2 条用例，一条是正常情况，一条是异常情况，对于原始版本的`parseAndroidVersion()` 方法，一定是能通过单元测试的。此时后面新入职的同学，“自作聪明”地做出相应优化，再跑单元测试的时候，`testParseAndroidVersion()` 方法就会报错，通过查看日志，我们一下子就可以看到错误在哪：

![](file:///Users/mantou/.config/joplin-desktop/resources/bea925e2b1ca4d6e8cbd6ee4838cd782.webp)

测试用例会告诉我们，这个地方原本应该照样能解析出 14，但是结果却返回了 -1，这显然不符合预期。

可惜没有如果，原始的开发同学没有留下单元测试，后续的维护者不清楚这块的逻辑。原始的开发同学、测试又都离职了，没办法打听，只能硬着头皮去改，以为不影响，没想到上线后出了事。（这一刻是不是好熟悉？因为在企业内部太常见了。）

一般来讲，大部分企业都会把跑单元测试这个行为整合到 CI 流程，CI 跑不过，代码自然合不进去，这个时候开发必然会回过头来看为什么失败，然后就会一下子发现“哦不能这样改，原来这边还要考虑 `displayName` 的值后面包含括号的情况！”

## 单元测试解决了什么？

上面这个例子直观地展现了，如果有单元测试，我们就可以避免掉这个本不该发生的错误。

我在带团队的时候，经常会跟团队成员说：“我们在写代码的时候，有很多变更（Change line），是有且仅有我们开发同学才知道，这个地方这样改，会影响到哪些其它地方的，所以提测的时候，一定要跟测试同学说清楚，把可能影响到的地方重点回归测试一下，因为开发不说，测试是不可能知道的。”

道理很简单。在一个团队里，测试同学不碰代码，不知道你写的这个逻辑有哪些地方调到；运营同学不关心技术，更加不知道你这个地方可能跟某个数据不兼容。而我们作为开发本身，除了要在写代码的时候，考虑到健壮性（空安全、边界数据、异常处理）之外，完全可以运用单元测试，把你作为开发能想到的场景，丰富到单元测试的用例里面去，这样当后续有他人维护这块逻辑的时候，才能够更有信心地进行重构或添加新的功能，因为你知道他如果不小心破坏了什么，测试就会失败。这样就可以在问题发生的早期就发现问题，从而更早地修复它们。

大家都知道，aosp 一直都有一个内部分支（`internal`）和外部分支（`main`）。

Google 内部会持续在`internal`分支开发，定期把代码向`main`分支合并。而外部开发者如果有提交，则是直接提交到 `main` 分支，每次提交的时候 Google 会检查这个提交与 `internal`分支是否冲突，并确保提交不会产生破坏，如果一切 OK，经过 review 之后就能 merge。

文字可能不太好理解，我画一个简单的流程图：

```
    +--------------------+             +--------------------+
  |                    |             |                    |
  |  aosp internal 分支 |             |       外部提交       +<---+ 外部开发者
  |       合 并         |             |                    |     向 aosp 提交
  |                    |             |                    |
  +--------^-----------+             +--------+-----------+
           |                                  |
      同时操作内部提交                        CI 生成内部提交
           |                                  |
  +--------+-----------+             +--------v-----------+
  |                    |             |                    |
  |   aosp main 分支    +<------------+       内部提交       |
  |       合 并         |   CI 检查    |                    |
  |                    |   &         +--------------------+
  +--------------------+   人工 review 通过
```

大家都知道 Android 很大，模块很多，再加上内部外部都会持续演进，如果没有 CI 对每笔提交进行把关，很容易会变得磕磕绊绊。而 CI 究竟是如何进行把关的呢？很重要的一点就是单元测试，在 aosp 里，单元测试被整合进了一个叫 [atest](https://link.juejin.cn/?target=https%3A%2F%2Fsource.android.com%2Fdocs%2Fcore%2Ftests%2Fdevelopment%2Fatest "https://source.android.com/docs/core/tests/development/atest") 的套件里。

不知道有多少同学注意过，aosp 的模块里只要是可测试的项目，它的根目录一定有一个 `TEST_MAPPING` 文件，以 `Settings` 模块为例，我们打开看一下：

```
{
   "presubmit":[
      {
         "name":"SettingsSpaUnitTests"
      },
      {
         "name":"SettingsUnitTests",
         "options":[
            {
               "include-filter":"com.android.settings.password"
            },
            {
               "include-filter":"com.android.settings.biometrics"
            },
            {
               "include-filter":"com.android.settings.biometrics2"
            }
         ]
      }
   ],
   "postsubmit":[
      {
         "name":"SettingsUnitTests",
         "options":[
            {
               "exclude-annotation":"androidx.test.filters.FlakyTest"
            }
         ]
      },
      {
         "name":"SettingsPerfTests"
      }
   ]
}
```

如果熟悉 Google 软件开发流程的话，对于 `presubmit` 这个单词就会很熟悉，这是 Google 内部一直践行的一套代码提交前的预检查流程，它与 CL、Code Review 流程紧密结合。每个项目都可以根据实际情况配置自己的预检查流程，CI 会按照配置对每一笔提交去跑这些流程，如果全部 PASS，会在 Code Review 给提交 +1（有 `warning`） 或者 +2，否则就 -1（单元测试有 `error`） 或者 -2（编不过）。对 `presubmit` 流程感兴趣的，推荐阅读 [Efficacy Presubmit](https://link.juejin.cn/?target=https%3A%2F%2Ftesting.googleblog.com%2F2018%2F09%2Fefficacy-presubmit.html "https://testing.googleblog.com/2018/09/efficacy-presubmit.html")。

从上面的配置文件可以看到，Settings 项目定义了大量的单元测试用例，我们也可以在 [packages/apps/Settings/tests/](https://link.juejin.cn/?target=https%3A%2F%2Fcs.android.com%2Fandroid%2Fplatform%2Fsuperproject%2Fmain%2F%2B%2Fmain%3Apackages%2Fapps%2FSettings%2Ftests%2F "https://cs.android.com/android/platform/superproject/main/+/main:packages/apps/Settings/tests/") 找到这些用例的代码。这些用例，对于开发而言，避免了“不知道修改会不会有什么其它影响”的窘境；对测试而言，避免了“不知道会不会漏测某种场景”的情况。可以说，单元测试可以极大程度保证项目健康度，并且节省了大量的测试手工测试的时间。

## 所以，单元测试能替代手动测试吗？

答案一定是——不能。

相信大家也感受到了，单元测试主要用于检查代码的各个单元是否正常工作，它主要关注的是代码纯逻辑层面的功能，以及一些边界情况和异常处理是否正确。换句话说，如果一个方法是要计算 `1 + 1 = 2`，单元测试可以保证不管你怎么改，只要 `1 + 1 ≠ 2` 了，第一时间就告诉你，而不用拖到上线才知道。由此带来的好处也是显而易见的，既可以节省不少人力成本，节约大量时间，也能保持项目维持在一个不错的健康度。

而手动测试则可以评估应用的用户体验，例如用户界面是否友好，用户交互是否流畅等。这些是单元测试无法覆盖的。此外，手动测试还可以更好地模拟用户的行为和使用场景，检查应用在各种情况下是否都能正常工作。例如，网络不稳定时，应用是否能正常工作？当用户同时打开多个应用，或者在应用中进行复杂操作时，应用是否仍然稳定？尤其在 Android 开发过程中，有很多场景可能更需要人工去点击测试，靠主观来感受是否流畅，是否 anr 等。

所以，单元测试和手动测试应该同时使用，而不是互相替代。通过结合两种测试方法，你可以确保你的应用在功能、性能和用户体验上都达到了预期。

当然了，随着很多 Android 平台测试框架的出现，现在有很多框架也可以开始模拟卡顿，弱网，或者很多以前需要人工去制造的场景，来进行单元测试，从而更好地量化一些指标，避免主观判断不准确。至于这些框架如何使用，我会在后面的文章中陆续和大家进行分享。

## 总结

在这篇文章里，我向大家简单解释了什么是单元测试，用一个最简单的场景说明了为什么需要单元测试，它的好处，以及 aosp 是如何运用单元测试的。

过去的一年半，我由于参与 aosp 和 androidx，也算是从小白到入门了单元测试。在后续的一些个人项目里，我逐渐也加入了单元测试，慢慢感受到了它的好处，因此想在这里分享给大家。时间允许的话，我会在后续与大家分享 android 单元测试的更多用法！

希望大家也能从今年开始慢慢用起来，别再冷落了项目里的 `androidTest` 文件夹啦。