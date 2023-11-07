创建Application之后调用AppWatcher的manualInstall方法，默认实现四种watcher：
- ActivityWatcher：监测Activity
- FragmentAndViewModelWatcher：监测FragmentAndViewModel
- RootViewWatcher：RootView
- ServiceWatcher：Service


# 流程图

![image.png](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/202311071718807.png)


# 总结
它使用了一个名为 RefWatcher 的类来监视对象的引用。当一个对象被创建时，它会被添加到 RefWatcher 中。当这个对象不再被引用时，RefWatcher 会检测它是否被正确地回收。如果对象没有被回收，LeakCanary 会生成一个报告，告诉开发人员哪些对象没有被正确地回收。开发人员可以使用报告中提供的信息来修复内存泄漏问题，从而提高应用程序的性能和稳定性。

#### 参考资料
[Android内存泄漏&leakcanary🐤2.7原理](https://juejin.cn/post/7028491809185595399)

[Android主流三方库源码分析（六、深入理解Leakcanary源码）](https://juejin.cn/post/6844904070977749005)


