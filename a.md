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
    * [事件处理](#事件处理)
      * [方法引用](#方法引用)
      * [侦听器绑定](#侦听器绑定)
    * [导入变量和包含](#导入变量和包含)
  * [使用可观察的数据对象](#使用可观察的数据对象)
  * [生成的绑定类](#生成的绑定类)
  * [绑定适配器](#绑定适配器)
  * [将布局视图绑定到体系结构组件](#将布局视图绑定到体系结构组件)
  * [双向数据绑定](#双向数据绑定)


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

  > **注意**：要使XML在语法上正确，必须转义`<`字符。例如:List<String>您必须编写List&lt;String>。

  您还可以使用对象引用映射中的值`object.key`。例如，上面例子中的@{map[key]}可以替换为@{map.key}。

* 字符串文字

  您可以使用单引号括起属性值，这允许您在表达式中使用双引号，如以下示例所示：

  ```
  android:text='@{map["firstName"]}'
  ```

  也可以使用双引号来包围属性值。这样做时，字符串文字应该用后引号括起来```：

  ```
  android:text="@{map[`firstName`]}"
  ```

* 资源

  您可以使用以下语法访问表达式中的资源：

  ```
  android:padding="@{large? @dimen/largePadding : @dimen/smallPadding}"
  ```

  格式字符串和复数可以通过提供参数来计算:

  ```
  android:text="@{@string/nameFormat(firstName, lastName)}"
  android:text="@{@plurals/banana(bananaCount)}"
  ```

  当一个复数接受多个参数时，应传递所有参数:

  ```
  Have an orange
  Have %d oranges

  android:text="@{@plurals/orange(orangeCount, orangeCount)}"
  ```

#### 事件处理

数据绑定允许您编写从视图分派的表达式处理事件（例如，onClick()方法）。事件属性名称由侦听器方法的名称确定，但有一些例外。
例如，View.OnClickListener有一个方法onClick()，所以这个事件的属性是`android:onClick`。

对于click事件，有一些特殊的事件处理程序需要使用android:onClick以外的属性来避免冲突。您可以使用以下属性来避免这些类型的冲突:

| Class | Listener setter | Attribute |
|:---|:---|:---|
|SearchView	  |setOnSearchClickListener(View.OnClickListener) |android:onSearchClick|
|ZoomControls |setOnZoomInClickListener(View.OnClickListener) |android:onZoomIn     |
|ZoomControls |setOnZoomOutClickListener(View.OnClickListener)|android:onZoomOut    |

您可以使用以下机制来处理事件:

* 方法引用:在表达式中，可以引用符合侦听器方法签名的方法。当表达式计算为方法引用时，数据绑定将方法引用和所有者对象包装在侦听器中，并在目标视图上设置该侦听器。
  如果表达式的计算结果为null，则数据绑定不会创建侦听器，而是设置一个空侦听器。
* 侦听器绑定:这些是在事件发生时计算的lambda表达式。数据绑定总是创建一个侦听器，并将其设置在视图上。当事件被分派时，侦听器计算lambda表达式。

##### 方法引用

事件可以直接绑定到处理程序方法，类似于`android:onClick`可以分配给活动中的方法的方式。
与View onClick属性相比的一个主要优点是表达式在编译时处理，因此如果该方法不存在或其签名不正确，则会收到编译时错误。

方法引用和侦听器绑定之间的主要区别在于，实际的侦听器实现是在绑定数据时创建的，而不是在触发事件时创建的。如果希望在事件发生时计算表达式，则应该使用侦听器绑定。

要将事件分配给其处理程序，请使用普通绑定表达式，其值为要调用的方法名称。例如，请考虑以下示例布局数据对象：

```
public class MyHandlers {
    public void onClickFriend(View view) { ... }
}
```

绑定表达式可以将视图的click listener分配给onClickFriend()方法，如下所示:

```
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">
   <data>
       <variable name="handlers" type="com.example.MyHandlers"/>
       <variable name="user" type="com.example.User"/>
   </data>
   <LinearLayout
       android:orientation="vertical"
       android:layout_width="match_parent"
       android:layout_height="match_parent">
       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.firstName}"
           android:onClick="@{handlers::onClickFriend}"/>
   </LinearLayout>
</layout>
```

> **注意**：表达式中方法的签名必须与侦听器对象中方法的签名完全匹配。

##### 侦听器绑定

侦听器绑定是在事件发生时运行的绑定表达式。它们与方法引用类似，但它们允许您运行任意数据绑定表达式。此功能适用于Gradle版本2.0及更高版本的Android Gradle插件。

在方法引用中，方法的参数必须与事件侦听器的参数匹配。在侦听器绑定中，只有您的返回值必须与侦听器的预期返回值匹配（除非它期望无效）。
例如，请考虑以下具有该`onSaveClick()`方法的演示者类：

```
public class Presenter {
    public void onSaveClick(Task task){}
}
```

然后，您可以将click事件绑定到onSaveClick()方法，如下所示：

```
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">
    <data>
        <variable name="task" type="com.android.example.Task" />
        <variable name="presenter" type="com.android.example.Presenter" />
    </data>
    <LinearLayout android:layout_width="match_parent" android:layout_height="match_parent">
        <Button android:layout_width="wrap_content" android:layout_height="wrap_content"
        android:onClick="@{() -> presenter.onSaveClick(task)}" />
    </LinearLayout>
</layout>
```

当在表达式中使用回调函数时，数据绑定将自动创建必要的侦听器并为事件注册它。当视图触发事件时，数据绑定计算给定的表达式。
与常规绑定表达式一样，在计算这些侦听器表达式时，仍然会得到null和数据绑定的线程安全性。

在上面的例子中，我们没有定义传递给onClick(view)的视图参数。侦听器绑定为侦听器参数提供了两种选择:您可以忽略方法的所有参数，也可以为它们命名。
如果喜欢为参数命名，可以在表达式中使用它们。例如，上面的表达式可以写成:

```
android:onClick="@{(view) -> presenter.onSaveClick(task)}"
```

或者如果要在表达式中使用该参数，则可以按如下方式工作：

```
public class Presenter {
    public void onSaveClick(View view, Task task){}
}
```

```
android:onClick="@{(theView) -> presenter.onSaveClick(theView, task)}"
```

您可以使用带有多个参数的lambda表达式：

```
public class Presenter {
    public void onCompletedChanged(Task task, boolean completed){}
}
```

```
<CheckBox android:layout_width="wrap_content" android:layout_height="wrap_content"
      android:onCheckedChanged="@{(cb, isChecked) -> presenter.completeChanged(task, isChecked)}" />
```

如果正在侦听的事件返回一个类型不是void的值，则表达式也必须返回相同类型的值。例如，如果要监听长按事件，则表达式应返回布尔值。

```
public class Presenter {
    public boolean onLongClick(View view, Task task) { }
}
```

```
android:onLongClick="@{(theView) -> presenter.onLongClick(theView, task)}"
```

如果由于null对象而无法计算表达式，数据绑定将返回该类型的默认值。例如，引用类型为null, int类型为0,boolean类型为false，等等。

如果需要使用带谓词的表达式(例如三元组)，可以使用`void`作为符号。

```
android:onClick="@{(v) -> v.isVisible() ? doSomething() : void}"
```

**避免复杂的监听**

侦听器表达式非常强大，可以使代码非常容易阅读。另一方面，包含复杂表达式的侦听器会使布局难以阅读和维护。这些表达式应该与将可用数据从UI传递到回调方法一样简单。
您应该在从侦听器表达式调用的回调方法中实现任何业务逻辑。

#### 导入变量和包含

数据绑定库提供了导入、变量和包含等特性。导入使在布局文件中引用类变得容易。变量允许您描述可以在绑定表达式中使用的属性。包括让你在你的应用程序中重用复杂的布局。

##### 导入

导入允许您轻松地引用布局文件中的类，就像在托管代码中一样。数据元素中可以使用零个或多个导入元素。以下代码示例将View类导入布局文件：

```
<data>
    <import type="android.view.View"/>
</data>
```

导入View该类允许您从绑定表达式中引用它。下面的例子展示了如何引用View类的VISIBLE和GONE常量。

```
<TextView
   android:text="@{user.lastName}"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"
   android:visibility="@{user.isAdult ? View.VISIBLE : View.GONE}"/>
```

**输入别名**

当存在类名冲突时，可以将其中一个类重命名为别名。下面的例子将com.example.real.estate包中的View类重命名为Vista:

```
<import type="android.view.View"/>
<import type="com.example.real.estate.View"
        alias="Vista"/>
```

在布局文件中您可以使用Vista来引用com.example.real.estate.View，使用View来引用android.view.View。

**导入其他类**

导入的类型可以用作变量和表达式中的类型引用。下面的例子显示了作为变量类型使用的User和List:

```
<data>
    <import type="com.example.User"/>
    <import type="java.util.List"/>
    <variable name="user" type="User"/>
    <variable name="userList" type="List&lt;User>"/>
</data>
```

还可以使用导入的类型转换表达式的一部分。下面的示例将connection属性转换为User类型:

```
<TextView
   android:text="@{((User)(user.connection)).lastName}"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>
```

当引用表达式中的静态字段和方法时，也可以使用导入的类型。下面的代码导入MyStringUtils类并引用它的capitalize方法:

```
<data>
    <import type="com.example.MyStringUtils"/>
    <variable name="user" type="com.example.User"/>
</data>
…
<TextView
   android:text="@{MyStringUtils.capitalize(user.lastName)}"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>
```

就像托管代码一样，java.lang.*会自动导入。

##### 变量

您可以在数据元素中使用多个变量元素。每个变量元素描述可以在布局上设置的属性，用于布局文件中的绑定表达式。下面的示例声明的user，image和note变量：

```
<data>
    <import type="android.graphics.drawable.Drawable"/>
    <variable name="user" type="com.example.User"/>
    <variable name="image" type="Drawable"/>
    <variable name="note" type="String"/>
</data>
```

变量类型在编译时被检查，所以如果一个变量实现了Observable或者是一个Observable集合，那么它应该反映在类型中。
如果该变量是一个基类或接口，而该基类或接口没有实现可观察的接口，则不观察变量。

当不同配置(例如，横向或纵向)有不同的布局文件时，变量就会组合起来。这些布局文件之间不应该有相互冲突的变量定义。

生成的绑定类为每个描述的变量都有一个setter和getter。这些变量接受默认的托管代码值，直到setter被调用，引用类型为null, int为0，布尔值为false，等等。

将生成一个名为context的特殊变量，以便根据需要在绑定表达式中使用。context的值是根视图的getContext()方法中的上下文对象。上下文变量被具有该名称的显式变量声明覆盖。

##### 包含

通过使用app名称空间和属性中的变量名，可以将变量从包含布局传递到包含布局的绑定中。下面的示例显示了name.xml和contact.xml布局文件中的user变量:

```
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:bind="http://schemas.android.com/apk/res-auto">
   <data>
       <variable name="user" type="com.example.User"/>
   </data>
   <LinearLayout
       android:orientation="vertical"
       android:layout_width="match_parent"
       android:layout_height="match_parent">
       <include layout="@layout/name"
           bind:user="@{user}"/>
       <include layout="@layout/contact"
           bind:user="@{user}"/>
   </LinearLayout>
</layout>
```

数据绑定不支持include作为merge元素的直接子元素。例如，不支持以下布局：

```
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:bind="http://schemas.android.com/apk/res-auto">
   <data>
       <variable name="user" type="com.example.User"/>
   </data>
   <merge><!-- Doesn't work -->
       <include layout="@layout/name"
           bind:user="@{user}"/>
       <include layout="@layout/contact"
           bind:user="@{user}"/>
   </merge>
</layout>
```

### 使用可观察的数据对象

### 生成的绑定类

### 绑定适配器

### 将布局视图绑定到体系结构组件

### 双向数据绑定




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