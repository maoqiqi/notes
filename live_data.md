# LiveData

LiveData是一个可观察的数据持有者类。与常规observable不同，LiveData是生命周期感知的，这意味着它尊重其他应用程序组件的生命周期，例如活动，片段或服务。
此感知确保LiveData仅更新处于活动生命周期状态的应用程序组件观察者。


## 目录

* [概述](#概述)
* [使用LiveData](#使用LiveData)
   * [创建LiveData对象](#创建LiveData对象)
   * [观察LiveData对象](#观察LiveData对象)
   * [更新LiveData对象](#更新LiveData对象)
   * [与Room一起使用LiveData](#与Room一起使用LiveData)
* [扩展LiveData](#扩展LiveData)
* [转换LiveData](#转换LiveData)
* [合并多个LiveData](#合并多个LiveData)


## 概述

LiveData认为，如果一个观察者的生命周期处于STARTED或RESUMED状态，那么这个观察者(由observer类表示)就处于活动状态。
LiveData只将更新通知活动观察者。注册为监视LiveData对象的非活动观察者不会收到有关更改的通知。

您可以注册一个与实现LifecycleOwner接口的对象配对的观察者。当相应的Lifecycle对象的状态更改为DESTROYED时，此关系允许删除观察者。
这对于活动和片段特别有用，因为它们可以安全地观察LiveData对象，而不用担心泄漏——当活动和片段的生命周期被破坏时，它们会立即取消订阅。

使用LiveData具有以下优势：

* 确保您的UI符合您的数据状态

  LiveData遵循观察者模式。当生命周期状态发生变化时，LiveData通知Observer对象。您可以合并代码来更新这些观察者对象中的UI。
  您的观察者可以在每次发生更改时更新UI，而不是每次应用程序数据更改时都更新UI。

* 没有内存泄漏

  观察者绑定Lifecycle对象并在其相关生命周期被销毁后自行清理。

* 停止活动不会导致崩溃

  如果观察者的生命周期处于非活动状态（例如，在后台堆栈中的活动的情况下），则它不会接收任何LiveData事件。

* 不再需要手动生命周期处理

  UI组件只观察相关数据，不停止或恢复观察。LiveData自动管理所有这些，因为它在观察过程中知道相关的生命周期状态变化。

* 始终保持最新数据

  如果生命周期变为非活动状态，它将在再次变为活动状态时接收最新数据。例如，后台活动在返回前台后立即接收最新数据。

* 适当的配置更改

  如果某个活动或片段由于配置更改(如设备旋转)而重新创建，它将立即接收最新可用数据。

* 共享资源

  您可以使用单例模式扩展LiveData对象来包装系统服务，以便在您的应用程序中共享它们。LiveData对象连接到系统服务一次，然后任何需要该资源的观察者都可以查看LiveData对象。


## 使用LiveData

请按照以下步骤处理 LiveData对象：

1. 创建LiveData实例来保存特定类型的数据。这通常在ViewModel类中完成。
2. 创建一个Observer对象，该对象定义onChanged()方法，该方法控制LiveData对象持有的数据更改时发生的情况。您通常在UI控制器中创建一个Observer对象，例如一个活动或片段。
3. 使用observe()方法将观察者对象附加到LiveData对象。方法接受LifecycleOwner对象。这订阅观察者对象到LiveData对象，以便在发生更改时通知它。
   通常将观察者对象附加到UI控制器中，例如活动或片段。

> **注意**：您可以使用observeForever(observer)方法注册一个没有关联LifecycleOwner对象的观察者。
在这种情况下，观察者总是被认为是活跃的，因此总是被通知修改。您可以删除调用removeObserver(Observer)方法的这些观察者。

当您更新LiveData对象中存储的值时，只要附加的LifecycleOwner处于活动状态，它就会触发所有已注册的观察者。

LiveData允许UI控制器观察者订阅更新。当LiveData对象所持有的数据发生更改时，UI将自动进行更新。

### 创建LiveData对象

### 观察LiveData对象

### 更新LiveData对象

### 与Room一起使用LiveData












## 扩展LiveData

## 转换LiveData

## 合并多个LiveData
