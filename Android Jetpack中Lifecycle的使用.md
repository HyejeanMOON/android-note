为了应用的安全，经常会有从后台切回前台时需要进行检测各种安全状态。
我在开发一个项目的时候就有从后台切回前台时需要进行app可用性的检测。
以前这个功能实现起来还很麻烦。
但是自从Android Jetpack中有了Lifecycle的组件，实现起来容易了很多。

---
##### LifeCycle的简单介绍
Lifecycle时Android JectPack中的组件之一。
他是为生命感知组件进行生命周期处理。 
你不需要在onStart,onStart等生命周期方法中写入这种方法。
可以直接写一个方法，把该方法传入给Lifecycle，让Lifecycle帮你处理。 
这样可以使代码更有条理性，也更容易阅读。 

---

##### Lifecycle的作用
如果把Lifecycle写在Application类中， 则它帮你处理的是整个app的生命周期。
app启动时，结束时，切回前后台。
如果把Lifecycle写在Activity后者Fragment中，则它帮你处理的是该context的生命周期。
比如，新activity的创建和销毁。

---

##### Application中的Lifecycle
在这里我们尝试实现一下切回前后台时感知生命周期。

首先需要实现被监视者。比如我们需要进行应用检测，我们就创建一个AppCheckLifecycleObserver。 引入的参数时一个Callback。
在被监视类的方法中需要标记执行哪个阶段的生命周期。
具体方法是在具体方法上添加`@OnLifecycleEvent(Lifecycle.Event.ON_START)`。

```kotlin
class AppCheckLifecycleObserver(val callback: AppCheckCallback) :
    LifecycleObserver {

    /**
     * onStart
     */
    @OnLifecycleEvent(Lifecycle.Event.ON_START)
    fun onStart() {
        callback.onStart()
    }

    /**
     * onStop
     */
    @OnLifecycleEvent(Lifecycle.Event.ON_STOP)
    fun onStop() {
        callback.onStop()
    }
}

interface AppCheckCallback {

    fun onStart()

    fun onStop()
}

```

```kotlin
// 在Application的onCreate方法中加入监视代码
ProcessLifecycleOwner.get()
            .lifecycle.addObserver(AppCheckLifecycleObserver(object :
            AppCheckCallback {
            override fun onStart() {
                Log.d("android application","Foreground")
            }

            override fun onStop() {
                Log.d("android application","Background")
            }
        }))
```

ProcessLifecycleOwner是拥有感知整个应用生命周期的类。
把该类加入到Application中可以感知整个生命周期，但是加入到activity或Fragment中，只能感知activity或fragment的生命周期。