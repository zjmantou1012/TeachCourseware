---
author: zjmantou
title: Android的MVI架构最佳实践(二)：View层封装和repeatOnLifecycle的使用
time: 2024-02-29 周四
tags:
  - 资料
  - Android
  - MVI
---
## 前言

我们已经搭建了MVI中的M和I，对于View封装主要为了简化监听ViewModel的数据的样板代码。需要定义BaseActivity或者BaseFragment，很多同学并不喜欢使用BaseX的封装。因此在这个实践指导中这个环节并不是很重要，你可以随性直接看下一个小节。

## BaseFragment

在现在构建一个Android应用都是Single Activity的，所以我们也只封装下Fragment来消除一些模版代码。由于是单线数据流不需要DataBinding, layout我们首选ViewBinding；

```kotlin
abstract class BaseFragment<VB : ViewBinding, VM : BaseViewModel<*, *, *>>(
    val viewBinding: (LayoutInflater, ViewGroup?, Boolean) -> VB
) : Fragment() {
​
    protected abstract val viewModel: VM
​
    private var _binding: VB? = null
​
    protected val binding: VB
        get() = requireNotNull(_binding) { "The property of binding has been destroyed." }
​
    override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?, savedInstanceState: Bundle?
    ): View? {
        activity?.window?.setSoftInputMode(WindowManager.LayoutParams.SOFT_INPUT_ADJUST_PAN)
        binding = viewBinding(inflater, container, false)
        return binding?.run {
            initRenderers(this)
            root
        }
    }
​
    abstract fun initRenderers(binding: VB)
​
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        observeStateOrEvent(viewModel)
    }
​
    abstract fun observeStateOrEvent(viewModel: VM)
​
    override fun onDestroyView() {
        super.onDestroyView()
        _binding = null
    }
}
```

创建一个Fragment看看

```kotlin
class EditFragment :BaseFragment<FragmentEditBinding, EditViewModel>(  
   FragmentEditBinding::inflate  
) {  
​  
   override val viewModel by viewModels<EditViewModel>()  
​  
   override fun initRenderers(binding: FragmentEditBinding) {  
       with(binding) {  
           title.text = "aaa"  
      }  
  }  
​  
   override fun observeStateOrEvent(viewModel: EditViewModel) {  
       viewLifecycleOwner.lifecycleScope.launchWhenResumed {  
           viewModel.state.collect {

}  
      }  
  }  
}
```

很快啊，我们发现一些问题，再一个一个解决：

- `BaseFragment`已经在**泛型**中定义了`FragmentEditBinding`但是还是需要`FragmentEditBinding`调用方法才能真正注入binding对象，这不够优雅。
    
- `launchWhenResumed` 被标记为过时方法，推荐使用`repeatOnLifecycle`。原来StateFlow是无生命周期感知的，在应用进入后台时候Flow还是在collect数据，只要collect在当前的`lifecycleScope`调用了就不会停止，除非到scope生命周期结束。
    

## 优雅的创建ViewBinding

所有ViewBinding生成的代码都会实现一个接口`androidx.viewbinding.ViewBinding`, 并且自动生成的代码中都会有`inflate()的重载静态方法`，那我们可以通过反射获取并且调用方法。

```java
public interface ViewBinding {  
   @NonNull  
   View getRoot();  
}
```

1. `Class.getGenericSuperclass()` 转换为`ParameterizedType`
    
2. 通过`Type.getActualTypeArguments()`获取到所有的泛型定义
    
3. 然后`if(ViewBinding::class.java.isAssignableFrom(result))`找到是实现ViewBinding的类class
    
4. 反射
    
    inflate()
    
    创建ViewBinding

```kotlin
internal fun <V : ViewBinding?> withGenericViewBinding(aClass: Class<*>): V {
     return try {
         val genericSuperclass = aClass.genericSuperclass
         if (genericSuperclass is ParameterizedType) {
             genericSuperclass.actualTypeArguments.forEach {
                 val result: Class<*>
                 try {
                     result = it as Class<V>
                 } catch (_: Exception) {
                     return@forEach
                 }
                 if (ViewBinding::class.java.isAssignableFrom(result)) {
                     return result.getMethod(
                             "inflate", LayoutInflater::class.java, ViewGroup::class.java, Boolean::class.java
                     ).invoke(null, inflater, container, false)
                 }
             }
         }
         withGenericViewBinding(aClass.superclass, block)
     } catch (e: Exception) {
         e.printStackTrace()
         throw IllegalArgumentException("There is no generic of ViewBinding.")
     }
 }
```

5. 配置consumerProguardFiles, 配置的文件会把aar的混淆规则最终应用到APP中，需要检查lib的`build.gradle`中如下：
```groovy
defaultConfig {
    ....
    consumerProguardFiles "consumer-rules.pro"
}
```

6. 添加混淆规则到`consumer-rules.pro`
```properties
# consumer-rules.pro 中混淆规则  
-keep public interface androidx.viewbinding.ViewBinding  
-keep class * implements androidx.viewbinding.ViewBinding {  
public static * inflate(android.view.LayoutInflater);  
public static * inflate(android.view.LayoutInflater, android.view.ViewGroup, boolean);  
}
```

去掉ViewBinding调用方法最终效果

```kotlin
class EditFragment :BaseFragment<FragmentEditBinding, EditViewModel>() {

    override val viewModel by viewModels<EditViewModel>()

    override fun initRenderers(binding: FragmentEditBinding) {
        with(binding) {
            title.text = "aaa"
        }
    }

    override fun observeStateOrEvent(viewModel: EditViewModel) {
        viewLifecycleOwner.lifecycleScope.launchWhenResumed {
            viewModel.state.collect {
                // binding.title.text = it.content
            }
        }
    }
}
```

## repeatOnLifecycle

对比LiveData，Flow是无生命周期感知的，也就是在应用后台时候Flow还是在collect数据，只要collect调用了当前的lifecycleScope就不会停止，除非到生命周期结束。因此官方提供了一个`repeatOnLifecycle`在应用进入后台时候停止collect，当进入设置的生命周期会在此重新collect。使用此函数需要依赖 `androidx.lifecycle:lifecycle-runtime-ktx:2.6.1`, 同时这个包已经支持Flow的扩展函数

```kotlin
public fun <T> Flow<T>.flowWithLifecycle(
    lifecycle: Lifecycle,
    minActiveState: Lifecycle.State = Lifecycle.State.STARTED
): Flow<T> = callbackFlow {
    lifecycle.repeatOnLifecycle(minActiveState) {
        this@flowWithLifecycle.collect {
            send(it)
        }
    }
    close()
}
```

## 封装下fragment的flowWithLifecycle

```kotlin
fun <T> Flow<T>.observeWithLifecycle(
    fragment: androidx.fragment.app.Fragment,
    minActiveState: Lifecycle.State = Lifecycle.State.STARTED,
    collector: FlowCollector<T>
): Job = fragment.viewLifecycleOwner.lifecycleScope.launch {
    flowWithLifecycle(fragment.viewLifecycleOwner.lifecycle, minActiveState).collect(collector)
}

//Sample
class SampleFragment : Fragment() {
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        flow.observeWithLifecycle(this){
            // Consume flow emissions
        }
    }
}
```

## 总结

此篇中我们对BaseFragment进行封装，并使用反射优雅的创建ViewBinding。在Flow的使用中，为了实现和LiveData一样的生命周期感知，了解了官方提供的核心方法`repeatOnLifecycle`。 下篇中我们会把MVI移植到Compose中，并搭建一个快速手脚架，毕竟Compose才是MVI架构的未来舞台。

# 原文链接
[Android的MVI架构最佳实践(二)：View层封装和repeatOnLifecycle的使用](https://hoooopy.com/index.php/2023/10/25/android%e7%9a%84mvi%e6%9e%b6%e6%9e%84%e6%9c%80%e4%bd%b3%e5%ae%9e%e8%b7%b5%e4%ba%8c%ef%bc%9aview%e5%b1%82%e5%b0%81%e8%a3%85%e5%92%8crepeatonlifecycle%e7%9a%84%e4%bd%bf%e7%94%a8/)