# 数据绑定库

数据绑定库是一个支持库，允许您使用声明格式(而不是编程方式)将布局中的UI组件绑定到应用程序中的数据源。

## 目录

* [简单示例](#简单示例)
* [数据绑定库](#数据绑定库)


## 简单示例

布局通常在活动中定义，其中包含调用UI框架方法的代码。例如，下面的代码调用findViewById()来查找TextView小部件并将其绑定到viewModel变量的userName属性:

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


## 数据绑定库

* 表达式语言允许您编写将变量连接到布局中的视图的表达式。数据绑定库自动生成将布局中的视图与数据对象绑定所需的类。库提供了诸如导入、变量以及可以在布局中使用的一些特性。

  库的这些特性与现有布局无缝共存。例如，可以在表达式中使用的绑定变量是在一个数据元素中定义的，该数据元素是UI布局的根元素的兄弟元素。
  这两个元素都被包装在一个布局标签中，如下面的例子所示:

  ```
  <layout xmlns:android="http://schemas.android.com/apk/res/android"
          xmlns:app="http://schemas.android.com/apk/res-auto">
      <data>
          <variable
              name="viewModel"
              type="com.app.data.ViewModel" />
      </data>
      <ConstraintLayout... /> <!-- UI layout's root element -->
  </layout>
  ```

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