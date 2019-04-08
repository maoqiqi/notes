# 安卓架构组件 Lifecycle

生命周期感知组件执行操作以响应另一个组件（例如活动和片段）的生命周期状态的更改。这些组件可以帮助您生成更易于组织、更容易维护的轻量级代码。


## 目录

* [概述](#概述)
* [生命周期](#生命周期)
* [LifecycleOwner](#LifecycleOwner)
* [生命周期感知组件的最佳实践](#生命周期感知组件的最佳实践)
* [生命周期感知组件的用例](#生命周期感知组件的用例)
* [处理停止事件](#处理停止事件)

## 概述

一种常见的模式是在活动和片段的生命周期方法中实现依赖组件的操作。但是，这种模式会导致代码组织不良和错误的增加。
通过使用生命周期感知组件，您可以将依赖组件的代码从生命周期方法转移到组件本身。

android.arch.lifecycle软件包提供了一些类和接口，这些类和接口允许您构建生命周期感知组件——这些组件可以根据活动或片段的当前生命周期状态自动调整它们的行为。

Android框架中定义的大多数应用程序组件都具有附加的生命周期。生命周期由运行在流程中的操作系统或框架代码管理。
它们是Android工作原理的核心，您的应用程序必须尊重它们。不这样做可能会引发内存泄漏，甚至应用程序崩溃。

假设我们有一个活动在屏幕上显示设备位置。一个常见的实现可能是这样的:

```
class MyLocationListener {
    public MyLocationListener(Context context, Callback callback) {
        // ...
    }

    void start() {
        // connect to system location service
    }

    void stop() {
        // disconnect from system location service
    }
}

class MyActivity extends AppCompatActivity {
    private MyLocationListener myLocationListener;

    @Override
    public void onCreate(...) {
        myLocationListener = new MyLocationListener(this, (location) -> {
            // update UI
        });
    }

    @Override
    public void onStart() {
        super.onStart();
        myLocationListener.start();
        // manage other components that need to respond
        // to the activity lifecycle
    }

    @Override
    public void onStop() {
        super.onStop();
        myLocationListener.stop();
        // manage other components that need to respond
        // to the activity lifecycle
    }
}
```

尽管这个示例看起来很好，但是在实际应用程序中，您最终会有太多的调用来管理UI和其他组件，以响应生命周期的当前状态。
管理多个组件会在生命周期方法中放置大量的代码，比如onStart()和onStop()，这使得它们很难维护。

此外，不能保证组件在活动或片段停止之前启动。如果我们需要执行长时间运行的操作（例如某些配置检入），则尤其如此在onStart()。
onStop()方法在onStart()之前结束，从而使组件存活的时间超过所需的时间。

```
class MyActivity extends AppCompatActivity {
    private MyLocationListener myLocationListener;

    public void onCreate(...) {
        myLocationListener = new MyLocationListener(this, location -> {
            // update UI
        });
    }

    @Override
    public void onStart() {
        super.onStart();
        Util.checkUserStatus(result -> {
            // what if this callback is invoked AFTER activity is stopped?
            if (result) {
                myLocationListener.start();
            }
        });
    }

    @Override
    public void onStop() {
        super.onStop();
        myLocationListener.stop();
    }
}
```

android.arch.lifecycle 软件包提供了类和接口，可帮助您以一种灵活和隔离的方式处理这些问题。


## 生命周期

Lifecycle是一个类，它保存关于组件(如活动或片段)生命周期状态的信息，并允许其他对象观察这个状态。

Lifecycle使用两个主要枚举来跟踪其关联组件的生命周期状态:

* Event

  从框架和Lifecycle类调度的生命周期事件 。这些事件映射到活动和片段中的回调事件。

* State

  Lifecycle对象跟踪的组件的当前状态 。

  ![lifecycle_states](images/lifecycle_states.png)

  将状态视为图的节点，将事件视为这些节点之间的边。

  类可以通过向其方法添加注释来监视组件的生命周期状态。然后你可以通过调用Lifecycle类的addObserver()方法来传递观察者的实例来添加观察者，如以下示例所示：

  ```
  public class MyObserver implements LifecycleObserver {
      @OnLifecycleEvent(Lifecycle.Event.ON_RESUME)
      public void connectListener() {
          ...
      }

      @OnLifecycleEvent(Lifecycle.Event.ON_PAUSE)
      public void disconnectListener() {
          ...
      }
  }

  myLifecycleOwner.getLifecycle().addObserver(new MyObserver());
  ```

## LifecycleOwner

LifecycleOwner是一个单一的方法接口，表示该类有一个 Lifecycle。它有一个方法getLifecycle()必须由类实现。
如果您正在尝试管理整个应用程序进程的生命周期，请参阅ProcessLifecycleOwner。

这个接口从各个类(如Fragment和AppCompatActivity)中抽象生命周期的所有权，并允许编写与之一起工作的组件。任何自定义应用程序类都可以实现LifecycleOwner接口。

实现LifecycleObserver的组件与实现LifecycleOwner的组件无缝地工作，因为所有者可以提供一个生命周期，观察者可以注册该生命周期来观察。

对于位置跟踪示例，我们可以让MyLocationListener类实现LifecycleObserver，然后在onCreate()方法中使用活动的Lifecycle初始化它。
这允许MyLocationListener类是自给自足的，这意味着响应生命周期状态变化的逻辑是在MyLocationListener中声明的，而不是在活动中声明的。
使各个组件存储自己的逻辑使得活动和片段逻辑更易于管理。

```
class MyActivity extends AppCompatActivity {
    private MyLocationListener myLocationListener;

    public void onCreate(...) {
        myLocationListener = new MyLocationListener(this, getLifecycle(), location -> {
            // update UI
        });
        Util.checkUserStatus(result -> {
            if (result) {
                myLocationListener.enable();
            }
        });
  }
}
```

一个常见的用例是，如果生命周期目前处于不好的状态，则避免调用某些回调。例如，如果回调函数在保存活动状态之后运行一个片段事务，它将触发崩溃，因此我们永远不会想要调用那个回调函数。

为了简化这个用例，Lifecycle类允许其他对象查询当前状态。

```
class MyLocationListener implements LifecycleObserver {
    private boolean enabled = false;
    public MyLocationListener(Context context, Lifecycle lifecycle, Callback callback) {
       ...
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_START)
    void start() {
        if (enabled) {
           // connect
        }
    }

    public void enable() {
        enabled = true;
        if (lifecycle.getCurrentState().isAtLeast(STARTED)) {
            // connect if not connected
        }
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_STOP)
    void stop() {
        // disconnect if connected
    }
}
```

通过这种实现，我们的LocationListener类完全可以识别生命周期。如果我们需要使用LocationListener另一个活动或片段，我们只需要初始化它。所有设置和拆卸操作都由类本身管理。

如果库提供了需要使用Android生命周期的类，我们建议您使用生命周期感知组件。您的库客户端可以轻松地集成这些组件，而无需在客户端进行手动生命周期管理。

### 实现自定义LifecycleOwner

片段和活动在支持库26.1.0和更高版本中已经实现了LifecycleOwner接口。

如果你有一个自定义类，你想让它成为LifecycleOwner，你可以使用LifecycleRegistry类，但是你需要将事件转发到该类中，如下面的代码示例所示:

```
public class MyActivity extends Activity implements LifecycleOwner {
    private LifecycleRegistry lifecycleRegistry;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        lifecycleRegistry = new LifecycleRegistry(this);
        lifecycleRegistry.markState(Lifecycle.State.CREATED);
    }

    @Override
    public void onStart() {
        super.onStart();
        lifecycleRegistry.markState(Lifecycle.State.STARTED);
    }

    @NonNull
    @Override
    public Lifecycle getLifecycle() {
        return lifecycleRegistry;
    }
}
```

## 生命周期感知组件的最佳实践

* 保持UI控制器(活动和片段)尽可能精简。他们不应该试图获取自己的数据;相反，使用一个ViewModel来实现这一点，并观察一个LiveData对象来将更改反射回视图。
* 尝试编写数据驱动的UI，其中UI控制器的职责是在数据更改时更新视图，或者将用户操作通知回ViewModel。
* 将数据逻辑放到ViewModel类中。ViewModel应该充当UI控制器和应用程序其余部分之间的连接器。不过要注意，ViewModel并不负责获取数据(例如，从网络中)。
  相反，ViewModel应该调用适当的组件来获取数据，然后将结果返回给UI控制器。
* 使用数据绑定来维护视图和UI控制器之间的干净界面。这使您可以使视图更具说明性，并最大限度地减少在活动和片段中编写所需的更新代码。
  如果您更喜欢用Java编程语言执行此操作，请使用像Butter Knife这样的库 来避免样板代码并具有更好的抽象。
* 如果您的UI很复杂，请考虑创建一个 presenter 类来处理UI修改。这可能是一项艰巨的任务，但它可以使您的UI组件更容易测试。
* 避免在ViewModel中引用View或Activity上下文。如果ViewModel比活动活得长(在配置更改的情况下)，您的活动就会泄漏，垃圾收集器也不会正确地处理它。


## 生命周期感知组件的用例

生命周期感知组件可以使您在各种情况下更容易地管理生命周期。以下是一些例子:

* 在粗粒度和细粒度位置更新之间切换。使用生命周期感知组件可在您的位置应用程序可见时启用细粒度位置更新，并在应用程序位于后台时切换到粗粒度更新。
  LiveData一个生命周期感知组件，允许您的应用在用户更改位置时自动更新UI。
* 停止并开始视频缓冲。使用生命周期感知组件尽快启动视频缓冲，但推迟播放直到应用程序完全启动。您还可以使用生命周期感知组件在销毁应用程序时终止缓冲。
* 启动和停止网络连接。使用生命周期感知组件在应用程序处于前台时启用网络数据的实时更新（流式传输），并在应用程序进入后台时自动暂停。
* 暂停并恢复动画绘制。当应用程序在后台时，使用生命周期感知组件来处理动画绘制的暂停;当应用程序在前台时，使用生命周期感知组件来恢复绘制。


## 处理停止事件

当一个Lifecycle属于AppCompatActivity或Fragment时，当调用AppCompatActivity或Fragment的onSaveInstanceState()时，
Lifecycle的状态更改为CREATED, ON_STOP事件被分派。

当通过onSaveInstanceState()保存片段或AppCompatActivity的状态时，它的UI被认为是不可变的，直到 ON_START被调用。
试图在保存状态后修改UI可能会导致应用程序导航状态不一致，这就是为什么FragmentManager在应用程序运行FragmentTransaction保存后状态时抛出异常的原因。

LiveData通过避免在至少没有启动观察者的关联生命周期的情况下调用它的观察者来防止这种情况的发生。在幕后它 isAtLeast()在决定调用其观察者之前调用。

不幸的是，AppCompatActivity的onStop()方法是在onSaveInstanceState()之后调用的，这就留下了一个空白，UI状态更改是不允许的，但是Lifecycle还没有移动到CREATED状态。

为了防止这个问题，版本beta2中的Lifecycle类将创建的状态标记为未调度事件，从而使任何检查当前状态的代码都能得到真正的值，即使直到系统调用onStop()才会分派事件。

不幸的是，这个解决方案有两个主要问题:

* 在API级别23或更低时，Android系统实际上保存了一个活动的状态，即使它被另一个活动部分覆盖。换句话说，Android系统调用onSaveInstanceState()，但它不一定调用onStop()。
  这就创建了一个潜在的很长的间隔，在这个间隔中，观察者仍然认为生命周期是活动的，即使它的UI状态不能被修改。
* 任何想要向LiveData类公开类似行为的类都必须实现Lifecycle version beta 2或更低版本提供的解决方案。

> **注意**：为了使这个流程更简单，并提供与旧版本更好的兼容性，从版本1.0.0-rc1开始，，Lifecycle对象被标记为CREATED，
当调用onSaveInstanceState()而不需要等待对onStop()方法的调用时，ON_STOP被分派。
这不太可能影响您的代码，但您需要注意这一点，因为它与API级别26或更低的Activity类中的调用顺序不匹配。





## 使用

1.在build.gradle文件中添加gradle依赖关系

```
dependencies {
    def lifecycle_version = "1.1.1"

    // ViewModel and LiveData
    implementation "android.arch.lifecycle:extensions:$lifecycle_version"
    // just ViewModel
    implementation "android.arch.lifecycle:viewmodel:$lifecycle_version" // use -ktx for Kotlin
    // just LiveData
    implementation "android.arch.lifecycle:livedata:$lifecycle_version"
    // Lifecycles only (no ViewModel or LiveData).
    // Support library depends on this lightweight import
    implementation "android.arch.lifecycle:runtime:$lifecycle_version"

    annotationProcessor "android.arch.lifecycle:compiler:$lifecycle_version"
    // if using Java8, use the following instead of compiler
    implementation "android.arch.lifecycle:common-java8:$lifecycle_version"

    // 可选 - ReactiveStreams support for LiveData
    implementation "android.arch.lifecycle:reactivestreams:$lifecycle_version"

    // 可选 - Test helpers for LiveData
    testImplementation "android.arch.core:core-testing:$lifecycle_version"
}
```

AndroidX

```
dependencies {
    def lifecycle_version = "2.0.0-alpha1"

    // ViewModel and LiveData
    implementation "androidx.lifecycle:lifecycle-extensions:$lifecycle_version"
    // just ViewModel
    implementation "androidx.lifecycle:lifecycle-viewmodel:$lifecycle_version" // use -ktx for Kotlin
    // just LiveData
    implementation "androidx.lifecycle:lifecycle-livedata:$lifecycle_version"
    // alternatively - Lifecycles only (no ViewModel or LiveData). Some UI
    // AndroidX libraries use this lightweight import for Lifecycle
    implementation "androidx.lifecycle:lifecycle-runtime:$lifecycle_version"

    annotationProcessor "androidx.lifecycle:lifecycle-compiler:$lifecycle_version"
    // if using Java8, use the following instead of lifecycle-compiler
    implementation "androidx.lifecycle:lifecycle-common-java8:$lifecycle_version"

    // 可选 - ReactiveStreams support for LiveData
    implementation "androidx.lifecycle:lifecycle-reactivestreams:$lifecycle_version" // use -ktx for Kotlin

    // 可选 - Test helpers for LiveData
    testImplementation "androidx.arch.core:core-testing:$lifecycle_version"
}
```