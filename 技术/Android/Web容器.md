# 发生卡顿的情况
1.  JavaScript执行过长；
2. 大量的DOM操作，可以考虑使用虚拟DOM；
3. 大量网络请求，下载大量资源的时候；
4. JS脚本阻塞了渲染进程；

# 首屏优化方案
1. 分包、压缩、删除无用代码；
2. 静态资源分离，并行加载；
3. JS脚本非阻塞加载，defer、async；
4. 缓存策略；响应头中的Cache-Control和Expires等设置缓存策略；
5. SSR（服务器端渲染Server Side Rendering），在服务端生成HTML文档；
6. 预置Loading、骨架屏提升用户体验；

# 渲染优化
1. GPU加速；
	- CSS3的transform和opacity等属性来开启GPU加速
2. 减少回流、重绘；
	- 可以通过避免使用影响布局的属性、批量修改DOM元素等技术来减少回流和重绘操作
3. 离屏渲染
	- 离屏渲染是将页面中的部分内容在单独的图层中进行渲染，从而减少对主渲染线程的阻塞。可以使用CSS3的`transform`和`position`等属性来开启离屏渲染。
4. 懒加载
	- 将页面中的非必要资源（如图片和视频等）延迟加载，可以加快页面的加载速度。可以使用Intersection Observer和Lazyload等技术来实现懒加载

# JS优化
1. 防止内存优化
	- 场景：
		- ![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309261551582.png)

	- 内存泄漏会导致不必要的内存占用和程序崩溃。可以使用let和const关键字声明变量，避免变量污染和内存泄漏；
2. 循环尽早break；
3. 合理使用闭包；
	- 闭包可以在函数中保存局部变量和参数，避免全局变量的污染和泄漏。但是，如果使用不当，也会导致内存泄漏和性能下降；
4. 减少Dom访问
	- DOM操作是JavaScript性能的一个瓶颈。可以使用缓存和批量操作等技术来减少DOM访问次数，从而提高JavaScript的性能；
5. 防抖、节流
	- ![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309261553372.png)
	- 防抖和节流是用来控制函数调用频率的技术。可以使用setTimeout和requestAnimationFrame等API来实现防抖和节流(或者用第三方库也行)
6. Web Workers
	- Web Workers是一种在后台线程中执行JavaScript代码的技术。可以将耗时的计算任务和数据处理等操作放到Web Workers中执行，避免阻塞主线程，提高页面的响应速度

# 跨端容器
- UI组件
- 渲染引擎
- 逻辑控制引擎
- 通信桥梁
- 底层API抹平表面差异
![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309261631287.png)

# 跨端方案对比
![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202309261631930.png)


# WebView容器框架
在Android开发中，创建一个webview容器框架需要考虑以下因素：

1. **内存限制**：Android系统分配给每个应用程序的内存是有限的，因此，每个WebView占用的内存也受到限制。如果一个应用程序有多个WebView，就需要特别注意内存使用情况，以避免内存溢出的问题。
2. **性能优化**：WebView的性能取决于很多因素，包括网络状况、网页内容、JavaScript代码等。因此，需要采取一些措施来优化WebView的性能，例如，使用缓存、减少网络请求、优化JavaScript代码等。
3. **安全性**：WebView可以加载和执行网页代码，因此需要注意安全性问题。需要对输入的URL进行验证和过滤，以避免恶意代码的执行。同时，还需要对JavaScript代码进行限制，以避免可能的攻击。
4. **用户体验**：WebView的用户体验也非常重要。需要确保WebView的流畅性和响应速度，同时还需要提供必要的功能和控件，以方便用户的使用。
5. **跨进程**：如果一个应用程序需要同时打开多个WebView，可以考虑使用跨进程的方式来实现。这样可以有效地利用系统资源，提高应用程序的性能和稳定性。
6. **API接口**：在开发WebView容器框架时，需要提供相应的API接口，以方便开发者的使用。这些API接口应该简单易用，同时还需要提供相应的文档和示例代码等参考资料。

总之，在开发Android WebView容器框架时需要考虑多方面的因素，包括内存限制、性能优化、安全性、用户体验、跨进程和API接口等。这些因素可能会影响到应用程序的性能和稳定性，因此需要仔细考虑和评估。


# 参考链接
[Android webView统一容器SDK设计](https://juejin.cn/post/7117634798511718431?searchId=20230926171356B7A52A28EFB9C531784B)
