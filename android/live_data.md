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

如果观察者的生命周期处于STARTED或RESUMED状态，LiveData认为观察者处于活动状态，下面的示例代码演示了如何扩展LiveData类:

```
public class StockLiveData extends LiveData<BigDecimal> {
    private StockManager stockManager;

    private SimplePriceListener listener = new SimplePriceListener() {
        @Override
        public void onPriceChanged(BigDecimal price) {
            setValue(price);
        }
    };

    public StockLiveData(String symbol) {
        stockManager = new StockManager(symbol);
    }

    @Override
    protected void onActive() {
        stockManager.requestPriceUpdates(listener);
    }

    @Override
    protected void onInactive() {
        stockManager.removeUpdates(listener);
    }
}
```

此示例中价格监听器的实现包括以下重要方法：

* 当LiveData对象具有活动观察者时，将调用onActive()方法。这意味着您需要开始从该方法观察股票价格更新。
* 当LiveData对象没有任何活动的观察者时，将调用onInactive()方法。因为没有观察者在听，所以没有理由保持与StockManager服务的连接。
* setValue(T)方法更新LiveData实例的值，并将更改通知任何活动的观察者。

您可以使用StockLiveData类如下:

```
public class MyFragment extends Fragment {
    @Override
    public void onActivityCreated(Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);
        LiveData<BigDecimal> myPriceListener = ...;
        myPriceListener.observe(this, price -> {
            // Update the UI.
        });
    }
}
```

方法将fragment(它是LifecycleOwner的一个实例)作为第一个参数传递。这样做意味着这个观察者被绑定到与所有者关联的Lifecycle对象上，这意味着:

* 如果Lifecycle对象没有处于活动状态，那么即使值发生了更改，也不会调用观察者。
* 毁生命周期对象后，将自动删除观察者。

LiveData对象具有生命周期感知这一事实意味着您可以在多个活动，片段和服务之间共享它们。为了简化示例，您可以将LiveData类实现为单例，如下所示：

```
public class StockLiveData extends LiveData<BigDecimal> {
    private static StockLiveData sInstance;
    private StockManager stockManager;

    private SimplePriceListener listener = new SimplePriceListener() {
        @Override
        public void onPriceChanged(BigDecimal price) {
            setValue(price);
        }
    };

    @MainThread
    public static StockLiveData get(String symbol) {
        if (sInstance == null) {
            sInstance = new StockLiveData(symbol);
        }
        return sInstance;
    }

    private StockLiveData(String symbol) {
        stockManager = new StockManager(symbol);
    }

    @Override
    protected void onActive() {
        stockManager.requestPriceUpdates(listener);
    }

    @Override
    protected void onInactive() {
        stockManager.removeUpdates(listener);
    }
}
```

您可以在片段中使用它，如下所示：

```
public class MyFragment extends Fragment {
    @Override
    public void onActivityCreated(Bundle savedInstanceState) {
        StockLiveData.get(symbol).observe(this, price -> {
            // Update the UI.
        });
    }
}
```

多个片段和活动可以观察MyPriceListener实例。LiveData仅在系统服务中的一个或多个可见且处于活动状态时才连接到系统服务。


## 转换LiveData

您可能希望在将LiveData对象分发给观察者之前对其中存储的值进行更改，或者您可能需要LiveData根据另一个实例的值返回其他实例。
该Lifecycle包提供了Transformations包含支持这些方案的辅助方法的类。

### Transformations.map()

对存储在LiveData对象中的值应用函数，并将结果传播到下游。

```
LiveData<User> userLiveData = ...;
LiveData<String> userName = Transformations.map(userLiveData, user -> {
    user.name + " " + user.lastName
});
```

### Transformations.switchMap()

类似于map()，将函数应用于存储在LiveData 对象中的值，并将结果解包并调度到下游。传递给的函数switchMap()必须返回一个LiveData对象，如下例所示：

```
private LiveData<User> getUser(String id) {
  ...;
}

LiveData<String> userId = ...;
LiveData<User> user = Transformations.switchMap(userId, id -> getUser(id) );
```

您可以使用转换方法在观察者的生命周期中传递信息。除非观察者正在观察返回的LiveData对象，否则不会计算变换。
由于转换是延迟计算的，因此生命周期相关的行为会被隐式传递下去，而不需要额外的显式调用或依赖项。

如果您认为在ViewModel对象中需要一个Lifecycle对象，那么转换可能是一个更好的解决方案。例如，假设您有一个UI组件，它接受一个地址并返回该地址的邮政编码。
您可以为这个组件实现简单的视图模型，如下面的示例代码所示:

```
class MyViewModel extends ViewModel {
    private final PostalCodeRepository repository;
    public MyViewModel(PostalCodeRepository repository) {
       this.repository = repository;
    }

    private LiveData<String> getPostalCode(String address) {
       // DON'T DO THIS
       return repository.getPostCode(address);
    }
}
```

然后，UI组件需要从以前的LiveData对象注销注册，并在每次调用getPostalCode()时注册到新实例。
此外，如果重新创建UI组件，它将触发对repository.getPostCode()方法的另一个调用，而不是使用前一个调用的结果。

相反，您可以将邮政编码查询实现为地址输入的转换，如下面的示例所示:

```
class MyViewModel extends ViewModel {
    private final PostalCodeRepository repository;
    private final MutableLiveData<String> addressInput = new MutableLiveData();
    public final LiveData<String> postalCode =
            Transformations.switchMap(addressInput, (address) -> {
                return repository.getPostCode(address);
             });

  public MyViewModel(PostalCodeRepository repository) {
      this.repository = repository
  }

  private void setInput(String address) {
      addressInput.setValue(address);
  }
}
```

在本例中，postalCode字段被定义为addressInput的转换。只要您的应用程序有一个与postalCode字段关联的活动观察者，每当addressInput发生更改时，就会重新计算和检索字段的值。

这种机制允许较低级别的应用程序创建按需延迟计算的LiveData对象。ViewModel对象可以很容易地获得对LiveData对象的引用，然后在其上定义转换规则。

### 创建新的转换

在您的应用程序中，有十几个不同的特定转换可能有用，但它们不是默认提供的。要实现自己的转换，可以使用MediatorLiveData类，它侦听其他LiveData对象并处理它们发出的事件。
MediatorLiveData正确地将其状态传播到源LiveData对象。要了解关于此模式的更多信息，请参阅
[Transformations](https://developer.android.google.cn/reference/android/arch/lifecycle/Transformations.html)类的参考文档。


## 合并多个LiveData

MediatorLiveData是一个子类LiveData，允许您合并多个LiveData源。MediatorLiveData 只要任何原始LiveData源对象发生更改，就会触发对象的观察者。

例如，如果LiveDataUI中有一个可以从本地数据库或网络更新的对象，则可以将以下源添加到该 MediatorLiveData对象：

* LiveData与存储在数据库中的数据关联的对象。
* LiveData与从网络访问的数据关联的对象。

您的活动只需要观察MediatorLiveData对象以从两个来源接收更新。