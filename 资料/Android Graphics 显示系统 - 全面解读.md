---
author: zjmantou
title: Android Graphics 显示系统 - 全面解读
time: 2024-08-09 周五
tags:
  - Android
  - Framework
---
**01**

**—**

**前言**

Android Graphics显示系统系列文章从简单的示例入手，着眼于理论与实践，帮助开发人员快速构建安卓图像显示的基本概念和理论认知。

Android每年一个大版本的更新，图像显示系统的源码在架构和写作方式上也几经调整与修改，逻辑细节虽发生了很大的变化，但核心原理不变，本系列试图让大家理解这些“核心原理”，以不变应万变，帮助大家轻松胜任各个版本的开发及调试工作。

**作为一个业余爱好者**，学习的过程是漫长的，写作的过程更痛苦，该系列的笔记也不会一蹴而就或在短时间内一次完成，所以在学习过程中，我会不断的把新的笔记、新的收获更新上来。**该系列笔记会在动态中不断更新，有兴趣的可以关注公众号获得持续更新！**

**文章难免错误，请带着审慎与批判的态度去阅读，阅读中请保持独立思考。**

**02**

**—**

**主题简介  
  
**

该系列文章聚焦Graphics知识，从简单的Demo入手，分析图形显示基本框架和运作流程。涉及内容众多，比如SurfaceFlinger的运行机制，VSYNC信号的产生与分发，BufferQueue的工作原理，Mapper&Allocator，Fence同步机制，HWC基础框架，多屏显示，各种实用工具 ....

**比如 生产者消费者模型**

![](https://mmbiz.qpic.cn/mmbiz_png/fPpwWIqVn41VOFriawBM2picT2z3DjPSzR9a6dge7VSx68MoJhxPY7hYS8NqLLMk4GWP7p0hyj4PBFVpqdSRma2Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp)

**比如 BufferQueue的工作机制**

![](https://mmbiz.qpic.cn/mmbiz_png/fPpwWIqVn41VOFriawBM2picT2z3DjPSzR20ryylmI6puycMxQ4wtx7VjDIwxQicSmnoj9icZMmibAMwJ9wBKbwrZuQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp)

**比如 图形显示合成的基本流程**

![](https://mmbiz.qpic.cn/mmbiz_png/fPpwWIqVn41VOFriawBM2picT2z3DjPSzRhTNSBVkpbQ795NdT0pu7xzDicoJcE9KVaIJFykiclHo4qEXxvzkUwSRw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp)

**比如 多屏显示**

![](https://mmbiz.qpic.cn/mmbiz_gif/fPpwWIqVn43zvf97Qia4xibXwicRJWlGiaFpjvTGv4ESbdLN2sYNJMzgdBKiaRLzgY6YkDeecKhaNDAuSfUjGRuQ5Fg/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1) ![](https://mmbiz.qpic.cn/mmbiz_gif/fPpwWIqVn42Bk0hj6OqlUbHC0mXH6blWibia9j7qOtrQwmlrgmNibbIhsIpDpSGEjgyWxeVsDFXaAbgMbmxbWslTw/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

**03**

**—**

**文章系列  
  
**

系列文章持续更新，更多内容可以订阅 **“Android Graphics”** 专栏，目前已发表的文章如下：

**基础篇**

[Android Graphics 显示系统 - 开篇](http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484417&idx=1&sn=365eaac3802d874d21bd6fa96862bdf8&chksm=f9ccb5cacebb3cdc7137c87d7402950f9d624d76ce7049f372f4bf5c1d11fdc7a340ff8b0511&scene=21#wechat_redirect "http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484417&idx=1&sn=365eaac3802d874d21bd6fa96862bdf8&chksm=f9ccb5cacebb3cdc7137c87d7402950f9d624d76ce7049f372f4bf5c1d11fdc7a340ff8b0511&scene=21#wechat_redirect")

[Android Graphics 显示系统 - 基本组件（一）](http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484418&idx=1&sn=c206cd0dd42d2f89ec83f9dbfaf5d22d&chksm=f9ccb5c9cebb3cdf67f31a9f5928d4b9ba64d0833b44cea53818aac64f978a0a9870f2d86d5c&scene=21#wechat_redirect "http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484418&idx=1&sn=c206cd0dd42d2f89ec83f9dbfaf5d22d&chksm=f9ccb5c9cebb3cdf67f31a9f5928d4b9ba64d0833b44cea53818aac64f978a0a9870f2d86d5c&scene=21#wechat_redirect")

[Android Graphics 显示系统 - 基本组件（二）](http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484419&idx=1&sn=90ef267e6d590ab2b2215b5a7ab9da6e&chksm=f9ccb5c8cebb3cdeef102255941447c213da39ce1b38950a54726a879dff1d3b0ef66ea3dc39&scene=21#wechat_redirect "http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484419&idx=1&sn=90ef267e6d590ab2b2215b5a7ab9da6e&chksm=f9ccb5c8cebb3cdeef102255941447c213da39ce1b38950a54726a879dff1d3b0ef66ea3dc39&scene=21#wechat_redirect")

[Android Graphics 显示系统 - 基本组件（三）](http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484420&idx=1&sn=051a1f1cea3180a4ca277317f570e092&chksm=f9ccb5cfcebb3cd9efa19f390729e026a7428aa2587d1f974986396bb9ad69b6430235586902&scene=21#wechat_redirect "http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484420&idx=1&sn=051a1f1cea3180a4ca277317f570e092&chksm=f9ccb5cfcebb3cd9efa19f390729e026a7428aa2587d1f974986396bb9ad69b6430235586902&scene=21#wechat_redirect")

[Android Graphics 显示系统 - Surface绘图示例（四）](http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484421&idx=1&sn=ad36e726b4ca90fff330de4f9758432c&chksm=f9ccb5cecebb3cd811ea3793d78dffa54328084ee7e7cc0a092ddc1ad812548b0c729c842851&scene=21#wechat_redirect "http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484421&idx=1&sn=ad36e726b4ca90fff330de4f9758432c&chksm=f9ccb5cecebb3cd811ea3793d78dffa54328084ee7e7cc0a092ddc1ad812548b0c729c842851&scene=21#wechat_redirect")

[Android Graphics 显示系统 - Surface绘图示例（五）](http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484422&idx=1&sn=ca754d19c48cb94662cbe17cc4e1b327&chksm=f9ccb5cdcebb3cdb79cc4f94329a567273e839e51558272d580f6ad9915b8608bfe4bdfe6c29&scene=21#wechat_redirect "http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484422&idx=1&sn=ca754d19c48cb94662cbe17cc4e1b327&chksm=f9ccb5cdcebb3cdb79cc4f94329a567273e839e51558272d580f6ad9915b8608bfe4bdfe6c29&scene=21#wechat_redirect")

[Android Graphics 显示系统 - 建立SurfaceFlinger通信的流程（六）](http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484423&idx=1&sn=9f39503685f34683476c9a86e7cf6d90&chksm=f9ccb5cccebb3cda45b0a0fb07beb6616d9f5c25896b4e323dae2addd98ddeef8a688489b15f&scene=21#wechat_redirect "http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484423&idx=1&sn=9f39503685f34683476c9a86e7cf6d90&chksm=f9ccb5cccebb3cda45b0a0fb07beb6616d9f5c25896b4e323dae2addd98ddeef8a688489b15f&scene=21#wechat_redirect")

[Android Graphics 显示系统 - SurfaceFlinger的启动与初始化（七）](http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484424&idx=1&sn=e178f3fea9512f31d54590098267806a&chksm=f9ccb5c3cebb3cd54475f72df1e703828c921387b88ac4fe53a96c09c3c974a0d8a5d81bb290&scene=21#wechat_redirect "http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484424&idx=1&sn=e178f3fea9512f31d54590098267806a&chksm=f9ccb5c3cebb3cd54475f72df1e703828c921387b88ac4fe53a96c09c3c974a0d8a5d81bb290&scene=21#wechat_redirect")

[Android Graphics 显示系统 - SurfaceFlinger MessageQueue机制（八）](http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484425&idx=1&sn=e84284f97e1583c45ef7f3be7a1488c7&chksm=f9ccb5c2cebb3cd4cfe6cb48fdea40196ed9c979c7e52df8b8cc2b4ad5492afb6ddeb24283c2&scene=21#wechat_redirect "http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484425&idx=1&sn=e84284f97e1583c45ef7f3be7a1488c7&chksm=f9ccb5c2cebb3cd4cfe6cb48fdea40196ed9c979c7e52df8b8cc2b4ad5492afb6ddeb24283c2&scene=21#wechat_redirect")

[Android Graphics 显示系统 - 创建Surface流程（九）](http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484426&idx=1&sn=dc7fc191bbbab3495cd57aac6e5370d7&chksm=f9ccb5c1cebb3cd706a7bd1d50de2adc3128f0257ac3ca31c78f9851c67ff6586a1916477f6a&scene=21#wechat_redirect "http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484426&idx=1&sn=dc7fc191bbbab3495cd57aac6e5370d7&chksm=f9ccb5c1cebb3cd706a7bd1d50de2adc3128f0257ac3ca31c78f9851c67ff6586a1916477f6a&scene=21#wechat_redirect")

[Android Graphics 显示系统 - 初识BufferQueue（十）](http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484427&idx=1&sn=7f5689c2f71329a436c67643b49f1ea5&chksm=f9ccb5c0cebb3cd6c8acd8f8512314be40d046896a3bef1878976ebe4c4d6bda6c25e0fc8ed2&scene=21#wechat_redirect "http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484427&idx=1&sn=7f5689c2f71329a436c67643b49f1ea5&chksm=f9ccb5c0cebb3cd6c8acd8f8512314be40d046896a3bef1878976ebe4c4d6bda6c25e0fc8ed2&scene=21#wechat_redirect")

[Android Graphics 显示系统 - ANativeWindow/Surface/SurfaceControl（十一）](http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484428&idx=1&sn=96821e38b85924eba44b9599704e4771&chksm=f9ccb5c7cebb3cd1712e4997e7af18116f8d899412022bd5a8a34d26177b513541ca435e7151&scene=21#wechat_redirect "http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484428&idx=1&sn=96821e38b85924eba44b9599704e4771&chksm=f9ccb5c7cebb3cd1712e4997e7af18116f8d899412022bd5a8a34d26177b513541ca435e7151&scene=21#wechat_redirect")

[Android Graphics 显示系统 - BufferQueue的工作流程（十二）](http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484429&idx=1&sn=ed6c90c31cda6b57f96df91c77c1505b&chksm=f9ccb5c6cebb3cd02ac3f090191fc58311824ff8fe6f77b926cab1730d8f3eeee96c76c37489&scene=21#wechat_redirect "http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484429&idx=1&sn=ed6c90c31cda6b57f96df91c77c1505b&chksm=f9ccb5c6cebb3cd02ac3f090191fc58311824ff8fe6f77b926cab1730d8f3eeee96c76c37489&scene=21#wechat_redirect")

[Android Graphics 显示系统 - BufferQueue的工作流程（十三）](http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484430&idx=1&sn=ac29597d1c4d6492a110ac28baced999&chksm=f9ccb5c5cebb3cd3a7cdbdc71ead670c75a79e2aade7eefac8fec57952976caa21704ee66f2e&scene=21#wechat_redirect "http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484430&idx=1&sn=ac29597d1c4d6492a110ac28baced999&chksm=f9ccb5c5cebb3cd3a7cdbdc71ead670c75a79e2aade7eefac8fec57952976caa21704ee66f2e&scene=21#wechat_redirect")

[Android Graphics 显示系统 - BufferQueue的工作流程（十四）](http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484431&idx=1&sn=50062cecf8db63ecb4c6ab553c70accb&chksm=f9ccb5c4cebb3cd25779b1302b2530480f7b20d923a19ce2a1e87f7d71aaeece6f3935adf885&scene=21#wechat_redirect "http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484431&idx=1&sn=50062cecf8db63ecb4c6ab553c70accb&chksm=f9ccb5c4cebb3cd25779b1302b2530480f7b20d923a19ce2a1e87f7d71aaeece6f3935adf885&scene=21#wechat_redirect")

[Android Graphics 显示系统 - BufferQueue的工作流程（十五）](http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484432&idx=1&sn=943008fbb9bf43dd399d15f76dfa8576&chksm=f9ccb5dbcebb3ccda7652378ce75d3993c57fc6256cfa867fe3324f8f0afad4d23fba9101045&scene=21#wechat_redirect "http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484432&idx=1&sn=943008fbb9bf43dd399d15f76dfa8576&chksm=f9ccb5dbcebb3ccda7652378ce75d3993c57fc6256cfa867fe3324f8f0afad4d23fba9101045&scene=21#wechat_redirect")

[Android Graphics 显示系统 - Surface补充知识（十六）](http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484433&idx=1&sn=57e262d2c555e579943e2d4504de2b2c&chksm=f9ccb5dacebb3cccc91fb3c0a785bc0b71ce8a16754de1745005e10a95e34bd1ad6d3e280d63&scene=21#wechat_redirect "http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484433&idx=1&sn=57e262d2c555e579943e2d4504de2b2c&chksm=f9ccb5dacebb3cccc91fb3c0a785bc0b71ce8a16754de1745005e10a95e34bd1ad6d3e280d63&scene=21#wechat_redirect")

[Android Graphics 显示系统 - SurfaceView与BufferQueue关系（十七）](http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484434&idx=1&sn=fd4bc484bb732e409138f9113adaf3ae&chksm=f9ccb5d9cebb3ccf0d91bdd3217f7eee1d5d31a8a6facd8fda30ed545a1d405769f9f2efc11c&scene=21#wechat_redirect "http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484434&idx=1&sn=fd4bc484bb732e409138f9113adaf3ae&chksm=f9ccb5d9cebb3ccf0d91bdd3217f7eee1d5d31a8a6facd8fda30ed545a1d405769f9f2efc11c&scene=21#wechat_redirect")

[Android Graphics 显示系统 - Gralloc架构及GraphicBuffer创建/传递/释放（十八）](http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484435&idx=1&sn=99d907664158e4f44d0f95dbeb620781&chksm=f9ccb5d8cebb3cceecc995718593554e16e853cabe4a9120e33c050eacd79616a64b4c9a534e&scene=21#wechat_redirect "http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484435&idx=1&sn=99d907664158e4f44d0f95dbeb620781&chksm=f9ccb5d8cebb3cceecc995718593554e16e853cabe4a9120e33c050eacd79616a64b4c9a534e&scene=21#wechat_redirect")

[Android Graphics 显示系统 - 简述Allocator/Mapper服务的获取流程（十九）](http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484436&idx=1&sn=b14cd25c72600fa9adeef63112719a6b&chksm=f9ccb5dfcebb3cc93b1dcafc2211a3de477e2d046d8a61ab11cb28c10d96130b1c4490566d05&scene=21#wechat_redirect "http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484436&idx=1&sn=b14cd25c72600fa9adeef63112719a6b&chksm=f9ccb5dfcebb3cc93b1dcafc2211a3de477e2d046d8a61ab11cb28c10d96130b1c4490566d05&scene=21#wechat_redirect")

[Android Graphics 显示系统 - GraphicBuffer同步机制-Fence（二十）](http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484437&idx=1&sn=3bf8866bdbdba558965c4dd76a1fe9b2&chksm=f9ccb5decebb3cc88620093e3efaaa2151eebda68ddfb2e56dbcd0985ae9d72c8e8e42048c56&scene=21#wechat_redirect "http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484437&idx=1&sn=3bf8866bdbdba558965c4dd76a1fe9b2&chksm=f9ccb5decebb3cc88620093e3efaaa2151eebda68ddfb2e56dbcd0985ae9d72c8e8e42048c56&scene=21#wechat_redirect")

[Android Graphics 显示系统 - SurfaceFlinger的GPU合成（廿一）](http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484438&idx=1&sn=7b374447e55ecbe42a11ec0a735402f8&chksm=f9ccb5ddcebb3ccb07d64246fd39f40a52c152abf93d6e100034d1bf37029da227313affd424&scene=21#wechat_redirect "http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484438&idx=1&sn=7b374447e55ecbe42a11ec0a735402f8&chksm=f9ccb5ddcebb3ccb07d64246fd39f40a52c152abf93d6e100034d1bf37029da227313affd424&scene=21#wechat_redirect")

[Android Graphics 显示系统 - 导出图层数据(dump graphic raw data)（廿二）](http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484439&idx=1&sn=7391f55281d330bcd00be144c4ef4fe6&chksm=f9ccb5dccebb3ccadb2ccb1ac84aca63efcde5d2ebf3f8e7ef840a3b916bd05dfd13f62bb48f&scene=21#wechat_redirect "http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484439&idx=1&sn=7391f55281d330bcd00be144c4ef4fe6&chksm=f9ccb5dccebb3ccadb2ccb1ac84aca63efcde5d2ebf3f8e7ef840a3b916bd05dfd13f62bb48f&scene=21#wechat_redirect")

[Android Graphics 显示系统 - 基础知识之 BitTube（廿三）](http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484440&idx=1&sn=19cd145157cda46bae00a332009eb529&chksm=f9ccb5d3cebb3cc5a6947fa902d6078c4523ad602ce07d4dd67d6c3a90fb00ea77f09b7ecbf5&scene=21#wechat_redirect "http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484440&idx=1&sn=19cd145157cda46bae00a332009eb529&chksm=f9ccb5d3cebb3cc5a6947fa902d6078c4523ad602ce07d4dd67d6c3a90fb00ea77f09b7ecbf5&scene=21#wechat_redirect")

[Android Graphics 显示系统 - SurfaceFlinger之VSync-1（廿四）](http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484441&idx=1&sn=ace4bbee715c2ac41d137e75343e0af9&chksm=f9ccb5d2cebb3cc482ed39968a3f8d1378eea71b5fb5e2cdd93412d5bf1828e38754bf2945d8&scene=21#wechat_redirect "http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484441&idx=1&sn=ace4bbee715c2ac41d137e75343e0af9&chksm=f9ccb5d2cebb3cc482ed39968a3f8d1378eea71b5fb5e2cdd93412d5bf1828e38754bf2945d8&scene=21#wechat_redirect")

[Android Graphics 显示系统 - SurfaceFlinger之VSync-2（廿五）](http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484442&idx=1&sn=6cdccb021363f5d983608cb5b8091dbc&chksm=f9ccb5d1cebb3cc7bc099cab07fe8d54b808cd88c9a372a0a46db6babf8cc689f06df056b362&scene=21#wechat_redirect "http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484442&idx=1&sn=6cdccb021363f5d983608cb5b8091dbc&chksm=f9ccb5d1cebb3cc7bc099cab07fe8d54b808cd88c9a372a0a46db6babf8cc689f06df056b362&scene=21#wechat_redirect")

[Android Graphics 显示系统 - SurfaceFlinger之VSync-3（廿六）](http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484443&idx=1&sn=d42e04fc64270f6e5293858b1d495074&chksm=f9ccb5d0cebb3cc6d7608f3d539690da525ac5b3303fe532f02685fe973f462a4130754cf177&scene=21#wechat_redirect "http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484443&idx=1&sn=d42e04fc64270f6e5293858b1d495074&chksm=f9ccb5d0cebb3cc6d7608f3d539690da525ac5b3303fe532f02685fe973f462a4130754cf177&scene=21#wechat_redirect")

[Android Graphics 显示系统 - HWC HAL的初始化（廿七）](http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484444&idx=1&sn=4c3e3b291001ed9319eb4d2b739b897f&chksm=f9ccb5d7cebb3cc1ce9889d2abe2c094ab146b6c3b7f01a7d7acb85bb4419169c9b12ce14875&scene=21#wechat_redirect "http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484444&idx=1&sn=4c3e3b291001ed9319eb4d2b739b897f&chksm=f9ccb5d7cebb3cc1ce9889d2abe2c094ab146b6c3b7f01a7d7acb85bb4419169c9b12ce14875&scene=21#wechat_redirect")

[Android Graphics 显示系统 - 聊聊屏幕刷新机制（廿八）](http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484445&idx=1&sn=529022ec3f4cb3f00bd9f9814ef83315&chksm=f9ccb5d6cebb3cc0b8bca4e4acb1bfab671a9bbce07558575605388b2bc9e0639e1151402f2c&scene=21#wechat_redirect "http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484445&idx=1&sn=529022ec3f4cb3f00bd9f9814ef83315&chksm=f9ccb5d6cebb3cc0b8bca4e4acb1bfab671a9bbce07558575605388b2bc9e0639e1151402f2c&scene=21#wechat_redirect")

[Android Graphics 显示系统 - HWC 探秘 - 1（廿九）](http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484455&idx=1&sn=d9f33c8c3d82e1f48f67142a475132ae&chksm=f9ccb5eccebb3cfaa3de454892c81ac954e34e60dd4ae61fbe2fc35e7c6027138351c4236525&scene=21#wechat_redirect "http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484455&idx=1&sn=d9f33c8c3d82e1f48f67142a475132ae&chksm=f9ccb5eccebb3cfaa3de454892c81ac954e34e60dd4ae61fbe2fc35e7c6027138351c4236525&scene=21#wechat_redirect")

[Android Graphics 显示系统 - HWC 探秘 - 2（三十）](http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484459&idx=1&sn=92b39e350a0b26cf02a970f2e8260a5d&chksm=f9ccb5e0cebb3cf683b530db9db8f217afabb1a086ba912eb1e9168226b8bd0cd7d15266d073&scene=21#wechat_redirect "http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484459&idx=1&sn=92b39e350a0b26cf02a970f2e8260a5d&chksm=f9ccb5e0cebb3cf683b530db9db8f217afabb1a086ba912eb1e9168226b8bd0cd7d15266d073&scene=21#wechat_redirect")

[Android Graphics 显示系统 - HWC 探秘 - 3（三一）](http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484490&idx=1&sn=f0527daac880528e389fe0f79c73ad02&chksm=f9ccb581cebb3c9735563e9fcc986781efaaa28e7e38803c2eb57df814c83a0de66113e3f600&scene=21#wechat_redirect "http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484490&idx=1&sn=f0527daac880528e389fe0f79c73ad02&chksm=f9ccb581cebb3c9735563e9fcc986781efaaa28e7e38803c2eb57df814c83a0de66113e3f600&scene=21#wechat_redirect")

[Android Graphics 显示系统 - 解读Source Crop和Display Frame(三二)](http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484679&idx=1&sn=ce4bd3920b40d65635fd376bfe81eef4&chksm=f9ccb4cccebb3ddabc6ea35309fd17cf8e124b51b7626358f1e7f1fc91acd588e55dfba97735&scene=21#wechat_redirect "http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484679&idx=1&sn=ce4bd3920b40d65635fd376bfe81eef4&chksm=f9ccb4cccebb3ddabc6ea35309fd17cf8e124b51b7626358f1e7f1fc91acd588e55dfba97735&scene=21#wechat_redirect")

[Android Graphics 显示系统 - Android 14(U)编译、运行Surface绘图示例](http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484745&idx=1&sn=d60e65788d8a7f4cb44245cc9918b13f&chksm=f9ccb482cebb3d94c16f737f07186067067afe05573f4e923deecbeef026ba2ad1d3d111968d&scene=21#wechat_redirect "http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484745&idx=1&sn=d60e65788d8a7f4cb44245cc9918b13f&chksm=f9ccb482cebb3d94c16f737f07186067067afe05573f4e923deecbeef026ba2ad1d3d111968d&scene=21#wechat_redirect")

[Android Graphics 显示系统 - Android Jank detection with FrameTimeline](http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484743&idx=1&sn=fc2711b8afadd5667927456e86e6d7d9&chksm=f9ccb48ccebb3d9a15cb1c5bbc425ffd5858c2a2f116623fb4e31de9d06ee5f577f227b4e811&scene=21#wechat_redirect "http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484743&idx=1&sn=fc2711b8afadd5667927456e86e6d7d9&chksm=f9ccb48ccebb3d9a15cb1c5bbc425ffd5858c2a2f116623fb4e31de9d06ee5f577f227b4e811&scene=21#wechat_redirect")

[Android Graphics 显示系统 - 计算FPS的原理与探秘Present Fence](http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484884&idx=1&sn=c8fe89f637924900fb39baf327cafe92&chksm=f9ccb41fcebb3d09711c5ecf6e02f2e03396b16293a27cc9786f68b1c9827164205b73b9bfca&scene=21#wechat_redirect "http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484884&idx=1&sn=c8fe89f637924900fb39baf327cafe92&chksm=f9ccb41fcebb3d09711c5ecf6e02f2e03396b16293a27cc9786f68b1c9827164205b73b9bfca&scene=21#wechat_redirect")

[Android Graphics 显示系统 - 导出指定图层Layer数据与图层合成探秘](http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484798&idx=1&sn=ce4b31b8dd899546c91e2ede27e90079&chksm=f9ccb4b5cebb3da3fc7a0ec7d7170454c90e365d6cb308d52984faaa410d5d66788cb0cd3461&scene=21#wechat_redirect "http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484798&idx=1&sn=ce4b31b8dd899546c91e2ede27e90079&chksm=f9ccb4b5cebb3da3fc7a0ec7d7170454c90e365d6cb308d52984faaa410d5d66788cb0cd3461&scene=21#wechat_redirect")

**未完待续...**

**多屏篇**

[Android Graphics 多屏同显/异显 - 开篇](http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484645&idx=1&sn=93ba0723f4518fbba59d3cb49952eb2d&chksm=f9ccb52ecebb3c38d3e45522f87cefd57c05fda358a911defe0411ce53e42ce9abdde1fe4d3e&scene=21#wechat_redirect "http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484645&idx=1&sn=93ba0723f4518fbba59d3cb49952eb2d&chksm=f9ccb52ecebb3c38d3e45522f87cefd57c05fda358a911defe0411ce53e42ce9abdde1fe4d3e&scene=21#wechat_redirect")

[Android Graphics 多屏同显/异显 - C++示例程序(标准版)](http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484647&idx=1&sn=562692c6b3c5f0d7c3600d2f3398cd07&chksm=f9ccb52ccebb3c3a9f48ebc7c61930528c1ace5bddf0159552785f8ec4213047c964b2f834f6&scene=21#wechat_redirect "http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484647&idx=1&sn=562692c6b3c5f0d7c3600d2f3398cd07&chksm=f9ccb52ccebb3c3a9f48ebc7c61930528c1ace5bddf0159552785f8ec4213047c964b2f834f6&scene=21#wechat_redirect")

[Android Graphics 多屏同显/异显 - C++示例程序(升级版)](http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484648&idx=1&sn=9682ad03509b7427c4ecb89affde624e&chksm=f9ccb523cebb3c35acea57d455cbcbbbce48ffe5ed43b90c4a24f375446e6c4a6d69bb42cbfa&scene=21#wechat_redirect "http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484648&idx=1&sn=9682ad03509b7427c4ecb89affde624e&chksm=f9ccb523cebb3c35acea57d455cbcbbbce48ffe5ed43b90c4a24f375446e6c4a6d69bb42cbfa&scene=21#wechat_redirect")

[Android Graphics 多屏同显/异显 - Demo源码分析(1)](http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484649&idx=1&sn=2ac4e1e6915bbf9fdacc0114aa7d021d&chksm=f9ccb522cebb3c3484be07e1968d0a13329cbe2fa7dd24cb3396807248574361bd57137f8999&scene=21#wechat_redirect "http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484649&idx=1&sn=2ac4e1e6915bbf9fdacc0114aa7d021d&chksm=f9ccb522cebb3c3484be07e1968d0a13329cbe2fa7dd24cb3396807248574361bd57137f8999&scene=21#wechat_redirect")

[Android Graphics 多屏同显/异显 - Demo源码分析(2)](http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484650&idx=1&sn=cdfaa16f29516592498002dc862d2cff&chksm=f9ccb521cebb3c37d6106409746a2e598d18a6c5525aa7ef6ce2ee36705ec03ee8276b6e432b&scene=21#wechat_redirect "http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484650&idx=1&sn=cdfaa16f29516592498002dc862d2cff&chksm=f9ccb521cebb3c37d6106409746a2e598d18a6c5525aa7ef6ce2ee36705ec03ee8276b6e432b&scene=21#wechat_redirect")

[Android Graphics 多屏同显/异显 - Demo源码分析(3)](http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484651&idx=1&sn=abb32c7326313e4fbaaaf1ab10e38fd3&chksm=f9ccb520cebb3c369671aad62cfc5b216f0acfc823be954e0b4c7d776135cd92ebcfbdaad23b&scene=21#wechat_redirect "http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484651&idx=1&sn=abb32c7326313e4fbaaaf1ab10e38fd3&chksm=f9ccb520cebb3c369671aad62cfc5b216f0acfc823be954e0b4c7d776135cd92ebcfbdaad23b&scene=21#wechat_redirect")

[Android Graphics 多屏同显/异显 - mirrorSurface图层镜像](http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484722&idx=1&sn=2df9fd9eada3b70da4fa7b39ba66a5cb&chksm=f9ccb4f9cebb3def5c91892e6d43cc55f77b4fbacb365ba67c0c9e80200b8490758bad4f6e43&scene=21#wechat_redirect "http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484722&idx=1&sn=2df9fd9eada3b70da4fa7b39ba66a5cb&chksm=f9ccb4f9cebb3def5c91892e6d43cc55f77b4fbacb365ba67c0c9e80200b8490758bad4f6e43&scene=21#wechat_redirect")

[Android Graphics 多屏同显/异显 - 多屏时用于GPU合成的GraphicBuffer数量](http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484726&idx=1&sn=520d8b338501186b348d16571c2f704e&chksm=f9ccb4fdcebb3deb16a76ac78cbbefc2c41985671dfbc07a321f5136dea38bdd4476d28cce19&scene=21#wechat_redirect "http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484726&idx=1&sn=520d8b338501186b348d16571c2f704e&chksm=f9ccb4fdcebb3deb16a76ac78cbbefc2c41985671dfbc07a321f5136dea38bdd4476d28cce19&scene=21#wechat_redirect")

[Android Graphics 显示系统 - Android 14(U)编译/运行多屏同显/异显示例](http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484754&idx=1&sn=5451950554e1edbe15428d9f29c7f480&chksm=f9ccb499cebb3d8f2831c70ca694257203fb1dd9a3e08756cc35611c375990f1121b39d8a423&scene=21#wechat_redirect "http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484754&idx=1&sn=5451950554e1edbe15428d9f29c7f480&chksm=f9ccb499cebb3d8f2831c70ca694257203fb1dd9a3e08756cc35611c375990f1121b39d8a423&scene=21#wechat_redirect")

**未完待续...**

**工具篇**

[Android Graphics 显示系统 - 如何模拟多(物理)显示屏？](http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484508&idx=1&sn=078d92e0e8121190c3da024444d24430&chksm=f9ccb597cebb3c818d846afcb978e6ae424bafabbdaf143ac473669d2767f518da10b1fc0369&scene=21#wechat_redirect "http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484508&idx=1&sn=078d92e0e8121190c3da024444d24430&chksm=f9ccb597cebb3c818d846afcb978e6ae424bafabbdaf143ac473669d2767f518da10b1fc0369&scene=21#wechat_redirect")

[Android Graphics 显示系统 - adb shell service命令与SurfaceFlinger调试](http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484765&idx=1&sn=845efc6b4bc9c63ab3a8e56ba4ec947d&chksm=f9ccb496cebb3d8011405ea499d6085826fa7470a664aaa9bb0ef2b59183fb5ef60765e0e310&scene=21#wechat_redirect "http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484765&idx=1&sn=845efc6b4bc9c63ab3a8e56ba4ec947d&chksm=f9ccb496cebb3d8011405ea499d6085826fa7470a664aaa9bb0ef2b59183fb5ef60765e0e310&scene=21#wechat_redirect")

[Android Graphics 显示系统 - BufferQueue的状态监测](http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484808&idx=1&sn=dde95e8bc731ac1f1914a613a5df8367&chksm=f9ccb443cebb3d552336868c023e6dc5055b13acff6accec393f4ddb61036a2ed83eef186445&scene=21#wechat_redirect "http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484808&idx=1&sn=dde95e8bc731ac1f1914a613a5df8367&chksm=f9ccb443cebb3d552336868c023e6dc5055b13acff6accec393f4ddb61036a2ed83eef186445&scene=21#wechat_redirect")

[Android Graphics 显示系统 - 监测、计算FPS的工具及设计分析](http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484855&idx=1&sn=8ec4bba81c6e4b968314ccac2f7ce637&chksm=f9ccb47ccebb3d6ac6a8929b547b3f6e2dbee3b4dcdd75962a7fe94e06a6213fa2b11ed5baa3&scene=21#wechat_redirect "http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484855&idx=1&sn=8ec4bba81c6e4b968314ccac2f7ce637&chksm=f9ccb47ccebb3d6ac6a8929b547b3f6e2dbee3b4dcdd75962a7fe94e06a6213fa2b11ed5baa3&scene=21#wechat_redirect")

[Android Graphics 显示系统 - 通过dumpsys SurfaceFlinger --latency计算FPS](http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484860&idx=1&sn=5734b6ab18865d8cd611a3e5c3feb673&chksm=f9ccb477cebb3d61ac2b26bdad38f2f014c6ac1b15af2cc42662f5c92e1a24d508e0c3a9f01e&scene=21#wechat_redirect "http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484860&idx=1&sn=5734b6ab18865d8cd611a3e5c3feb673&chksm=f9ccb477cebb3d61ac2b26bdad38f2f014c6ac1b15af2cc42662f5c92e1a24d508e0c3a9f01e&scene=21#wechat_redirect")

[Android Graphics 显示系统 - 使用WinScope跟踪窗口转换](http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484695&idx=1&sn=198fd2b9a17811ac7eed1dc0e3f36701&chksm=f9ccb4dccebb3dca229773c0e0a403f436cc955a0c851655a1c4422813f566c866016e04d036&scene=21#wechat_redirect "http://mp.weixin.qq.com/s?__biz=MzUyMjI5OTU1Ng==&mid=2247484695&idx=1&sn=198fd2b9a17811ac7eed1dc0e3f36701&chksm=f9ccb4dccebb3dca229773c0e0a403f436cc955a0c851655a1c4422813f566c866016e04d036&scene=21#wechat_redirect")

**未完待续...**