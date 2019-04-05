# 安卓架构组件 - databinding

数据绑定库是一个支持库，允许您使用声明格式(而不是编程方式)将布局中的UI组件绑定到应用程序中的数据源。

## 目录

* [概览](#概览)
* [使用数据绑定库](#使用数据绑定库)
  * [搭建环境](#搭建环境)
  * [布局和绑定表达式](#布局和绑定表达式)
    * [数据绑定布局文件](#数据绑定布局文件)
    * [数据对象](#数据对象)
    * [绑定数据](#绑定数据)
    * [表达语言](#表达语言)
  * [](#)
  * [](#)
  * [](#)
  * [](#)
  * [](#)


## 概览

传统布局通常在活动中定义，其中包含调用UI框架方法的代码。例如，下面的代码调用findViewById()来查找TextView小部件并将其绑定到viewModel变量的userName属性:

```
TextView textView = findViewById(R.id.tv);
textView.setText(viewModel.getUserName());
```

下面的示例展示了如何使用数据绑定库在布局文件中直接向小部件分配文本。这样就不需要调用上面显示的任何Java代码。注意在赋值表达式中使用`@{}`语法:

```
<TextView
    android:text="@{viewModel.userName}" />
```

在布局文件中绑定组件允许您删除活动中的许多UI框架调用，从而使它们更简单、更易于维护。这还可以提高应用程序的性能，并有助于防止内存泄漏和空指针异常。


## 使用数据绑定库

### 搭建环境

数据绑定库提供了灵活性和广泛的兼容性，所以您可以在运行Android 4.0 (API级别14)或更高的设备上使用它。

建议在您的项目中使用最新的Android插件Gradle。但是，1.5.0或更高版本支持数据绑定。

要将应用程序配置为使用数据绑定，请将dataBinding元素添加到构建中。app模块中的gradle文件，如下例所示:

```
android {
...
  dataBinding {
     enabled true
  }
```
}

> 注意：您必须为依赖于使用数据绑定的库的应用程序模块配置数据绑定，即使应用程序模块不直接使用数据绑定也是如此。

### 布局和绑定表达式

表达式语言允许您编写将变量连接到布局中的视图的表达式。数据绑定库自动生成将布局中的视图与数据对象绑定所需的类。库提供了诸如导入、变量以及可以在布局中使用的一些特性。

#### 数据绑定布局文件

数据绑定布局文件略有不同，首先是布局的根标记，然后是数据元素和视图根元素。这个视图元素是非绑定布局文件中的根元素。下面的代码显示了一个布局文件示例:

```
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">
   <data>
       <variable name="user" type="com.example.User"/>
   </data>
   <LinearLayout
       android:orientation="vertical"
       android:layout_width="match_parent"
       android:layout_height="match_parent">
       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.firstName}"/>
       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.lastName}"/>
   </LinearLayout>
</layout>
```

> **注意**：布局表达式应该保持小而简单，因为它们不能进行单元测试，并且只有有限的IDE支持。为了简化布局表达式，您可以使用自定义[绑定适配器](#绑定适配器)。

#### 数据对象

我们现在假设你有一个普通的对象来描述User实体：

```
public class User {
  public final String firstName;
  public final String lastName;
  public User(String firstName, String lastName) {
      this.firstName = firstName;
      this.lastName = lastName;
  }
}
```

这种类型的对象具有永不更改的数据。在应用程序中，数据通常只读取一次，之后就不再更改。也可以使用遵循一组约定的对象，例如在Java中使用访问器方法，如下例所示:

```
public class User {
  private final String firstName;
  private final String lastName;
  public User(String firstName, String lastName) {
      this.firstName = firstName;
      this.lastName = lastName;
  }
  public String getFirstName() {
      return this.firstName;
  }
  public String getLastName() {
      return this.lastName;
  }
}
```

从数据绑定的角度来看，这两个类是等价的。@{user.firstName}用于android:text属性，访问前一个类中的firstName字段，后一个类中的getFirstName()方法。
另外，如果方法存在，它也被解析为firstName()。

#### 绑定数据

数据绑定库为每个布局文件生成一个绑定类。默认情况下，类的名称基于布局文件的名称。例如布局文件名是activity_main.xml，因此生成的相应类是ActivityMainBinding。
该类包含从布局属性(例如，用户变量)到布局视图的所有绑定，并且知道如何为绑定表达式赋值。创建绑定的推荐方法是在扩展布局的同时进行绑定，如下面的示例所示:

```
@Override
protected void onCreate(Bundle savedInstanceState) {
   super.onCreate(savedInstanceState);
   ActivityMainBinding binding = DataBindingUtil.setContentView(this, R.layout.activity_main);
   User user = new User("Test", "User");
   binding.setUser(user);
}
```

在运行时，应用程序在UI中显示Test用户。或者，您可以使用LayoutInflater获得视图，如下面的示例所示:

```
ActivityMainBinding binding = ActivityMainBinding.inflate(getLayoutInflater());
```

如果您在Fragment， ListView或RecyclerView适配器中使用数据绑定，您可能更喜欢使用bindings类或DataBindingUtil类的inflation()方法，如下面的代码示例所示:

```
ListItemBinding binding = ListItemBinding.inflate(layoutInflater, viewGroup, false);
// or
ListItemBinding binding = DataBindingUtil.inflate(layoutInflater, R.layout.list_item, viewGroup, false);
```

#### 表达语言

* 空合并操作符

  空合并操作符(??)在左操作数不为空时选择左操作数。

  ```
  android:text="@{user.displayName ?? user.lastName}"
  ```

  等价于

  ```
  android:text="@{user.displayName != null ? user.displayName : user.lastName}"
  ```

* 属性引用

  表达式可以引用类中的属性，方法如下:

  ```
  android:text="@{user.lastName}"
  ```

* 避免空指针异常

  生成的数据绑定代码自动检查空值并避免空指针异常。例如，在表达式@{user.name}中，如果user为null，则将user.name的默认值赋为null。
  如果您引用user.age，其中age是类型int，那么数据绑定使用默认值0。

* 集合

  为了方便起见，可以使用[]操作符访问常见的集合，如arrays, lists, sparse lists, and maps。

  ```
  <data>
      <import type="java.util.List"/>
      <import type="android.util.SparseArray"/>
      <import type="java.util.Map"/>
      <variable name="list" type="List&lt;String>"/>
      <variable name="sparse" type="SparseArray&lt;String>"/>
      <variable name="map" type="Map&lt;String, String>"/>
      <variable name="index" type="int"/>
      <variable name="key" type="String"/>
  </data>
  …
  android:text="@{list[index]}"
  …
  android:text="@{sparse[index]}"
  …
  android:text="@{map[key]}"
  ```

  > **注意**：要使XML在语法上正确，必须转义<字符。例如:List<String>您必须编写List&lt;String>。

  您还可以使用对象引用映射中的值`object.key`。例如，上面例子中的@{map[key]}可以替换为@{map.key}。

* 字符串文字







* 数据绑定库提供了类和方法来方便地观察数据的变化。当底层数据源发生更改时，您不必担心刷新UI。你可以使你的变量或它们的性质可见。库允许您使对象、字段或集合成为可观察的。

* 数据绑定库生成绑定类，用于访问布局的变量和视图。这个页面向您展示了如何使用和自定义生成的绑定类。

* 对于每个布局表达式，都有一个绑定适配器，用于进行设置相应属性或侦听器所需的框架调用。例如，绑定适配器可以调用setText()方法来设置文本属性，或者调用setOnClickListener()方法来向click事件添加侦听器。
  最常见的绑定适配器，如本页示例中使用的android:text属性的适配器，可供您在android.databinding.adapters包中使用。
  有关公共绑定适配器的列表，请参见适配器。您还可以创建自定义适配器，如下面的示例所示:

  ```
  @BindingAdapter("app:goneUnless")
  public static void goneUnless(View view, Boolean visible) {
      view.visibility = visible ? View.VISIBLE : View.GONE;
  }
  ```

* Android支持库包含架构组件，您可以使用它来设计健壮，可测试和可维护的应用程序。您可以将架构组件与数据绑定库一起使用来进一步简化UI的开发。

* 数据绑定库支持双向数据绑定。用于此类绑定的表示法支持接收对属性的数据更改并同时侦听用户对该属性的更新。