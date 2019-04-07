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

LiveData是一个包装器，可以用于任何数据，包括实现Collections的对象，比如List。LiveData对象通常存储在ViewModel对象中，并通过getter方法访问，如下例所示:

```
public class NameViewModel extends ViewModel {

// Create a LiveData with a String
private MutableLiveData<String> currentName;

    public MutableLiveData<String> getCurrentName() {
        if (currentName == null) {
            currentName = new MutableLiveData<String>();
        }
        return currentName;
    }

// Rest of the ViewModel...
}
```

最初，LiveData未设置对象中的数据。

> 注意:确保将更新UI的LiveData对象存储在ViewModel对象中，而不是存储在活动或片段中，原因如下:
> * 避免臃肿的活动和碎片。现在，这些UI控制器负责显示数据但不保持数据状态。
> * 将LiveData实例与特定活动或片段实例分离，并允许LiveData对象在配置更改后继续存在。

您可以在[ViewModel](view_model.md)中阅读更多关于ViewModel类的好处和用法的信息。

### 观察LiveData对象

在大多数情况下，app组件的onCreate()方法是开始观察LiveData对象的正确位置， 原因如下：

* 以确保系统不会从活动或fragment的onResume()方法发出冗余调用。
* 确保活动或片段具有数据，可以在活动时立即显示这些数据。一旦应用程序组件处于STARTED状态，它就会从它所观察的LiveData对象接收到最近的值。
  只有在设置了要观察的LiveData对象时才会发生这种情况。

通常，LiveData仅在数据更改时才提供更新，并且仅在活动观察者时提供更新。此行为的一个例外是观察者在从非活动状态更改为活动状态时也会收到更新。
此外，如果观察者第二次从非活动状态更改为活动状态，则只有在自上次活动状态以来该值发生更改时才会收到更新。

以下示例代码说明了如何开始观察LiveData 对象：

```
public class NameActivity extends AppCompatActivity {

    private NameViewModel model;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        // Other code to setup the activity...

        // Get the ViewModel.
        model = ViewModelProviders.of(this).get(NameViewModel.class);


        // Create the observer which updates the UI.
        final Observer<String> nameObserver = new Observer<String>() {
            @Override
            public void onChanged(@Nullable final String newName) {
                // Update the UI, in this case, a TextView.
                nameTextView.setText(newName);
            }
        };

        // Observe the LiveData, passing in this activity as the LifecycleOwner and the observer.
        model.getCurrentName().observe(this, nameObserver);
    }
}
```

使用传递的参数nameObserver调用observe()后，立即调用onChanged()，提供存储在mCurrentName中的最新值。如果LiveData对象没有在mCurrentName中设置值，则不会调用onChanged()。

### 更新LiveData对象

LiveData没有公开可用的方法来更新存储的数据。MutableLiveData类公开了setValue(T)和postValue(T)方法，如果需要编辑存储在LiveData对象中的值，则必须使用这些方法。
通常在ViewModel中使用MutableLiveData，然后ViewModel只向观察者公开不可变的LiveData对象。

设置好观察者关系后，就可以更新LiveData对象的值，如下面的例子所示，当用户点击一个按钮时，它就会触发所有的观察者:

```
button.setOnClickListener(new OnClickListener() {
    @Override
    public void onClick(View v) {
        String anotherName = "John Doe";
        model.getCurrentName().setValue(anotherName);
    }
});
```

在示例中调用setValue(T)会导致观察者使用值John Doe调用他们的onChanged()方法。该示例显示一个按钮按下，但是可以调用setValue()或postValue()来更新mName，
原因有很多，包括响应网络请求或完成数据库加载;在所有情况下，对setValue()或postValue()的调用都会触发观察者并更新UI。

> **注意**:必须调用setValue(T)方法才能从主线程更新LiveData对象。如果代码在工作线程中执行，则可以使用postValue(T)方法来更新LiveData对象。

### 与Room一起使用LiveData

Room持久性库支持可观察的查询，这些查询返回LiveData对象。可观察查询是作为数据库访问对象(DAO)的一部分编写的。

在更新LiveData数据库时，Room会生成更新对象所需的所有代码。生成的代码在需要时在后台线程上异步运行查询。
此模式对于使UI中显示的数据与存储在数据库中的数据保持同步非常有用。您可以在[Room](room.md)中阅读有关Room和DAO的更多信息。


## 扩展LiveData

## 转换LiveData

## 合并多个LiveData
