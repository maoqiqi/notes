# Espresso UI 测试框架

> 作者：March    
> 链接：[Espresso UI 测试框架](https://github.com/maoqiqi/blog/blob/master/pages/android_espresso.md)    
> 博客：http://blog.csdn.net/u011810138    
> 邮箱：fengqi.mao.march@gmail.com    
> 著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。    

Espresso是Google官方提供的Android UI自动化测试的框架。

## Espresso的重要组成部分

1. Espresso：通过onView()和onData()与view交互的进入点，它的api不依赖任何view。
2. ViewMatchers：实现了Matcher<? super View>的集合对象。通过onView()来定位当前的view。
3. ViewActions：具备操作方法（例如点击操作）的集合对象，它里面的操作可以通过ViewInteraction.perform()来实现。
4. ViewAssertions：用它可以断言，查看当前view的状态 ，ViewInteraction.check()会执行它。

## Espresso的使用步骤

- 1.加载View

普通的view用onView()加载视图:

  - R.id.xx是唯一的onView(withId(R.id.main_view))
  - 存在view的id是不唯一：若是通过以上方法会报错，
  com.google.android.apps.common.testing.ui.espresso.AmbiguousViewMatcherException
  解决方式：缩小范围，通过查看所要找到的view属性，采用多个属性来查找

   案例：若是两个edittext中的id相同，text内容不同，现在要获取text内容为"Hello!"：

           //通过id,text属性来确定所要查找的edittextview：
            onView(allOf(withId(R.id.my_view), withText("Hello!")))

AdapterView类型(例如gridview,listview，spinner)中的itemView:要通过onData()加载视图

特殊情况： 为了解决view中R.id问题（可能不存在，不唯一）的问题，获取view视图可以通过自定义(或者已经存在)的ViewMatchers


- 2.执行Action

在view中执行一个操作：
在匹配好指定view（即onView()或者onData()）后，可以通过perform()来执行ViewAction

案例之view的点击：        onView(withId(R.id.main_view)).perform(click())

 案例之view执行多个操作：  onView(...).perform(typeText("Hello"), click());

 案例之ScrollView中的view执行：onView(...).perform(scrollTo(), click());
      理解：执行click()随着sroolrto() ，确保执行操作在其他操作之前

- 3.检查View是否包含某种assertion

当前的view可以通过check()执行Assertions 。
通常通过matches()使用 assertion，matches（）是通过 ViewMatcher来断言当前view的状态

案例1：检查view是否包含“Hello”内容

         onView(...).check(matches(withText("Hello!")));

案例2：若是view没有明确displayed的情况下,不能将assertions放到onView()中，要将检查放到检查代码块中：

   // view的disPlay是未知情况下，assertions（断言）view的内容是否是“Hello!”,以下代码是不好的

   onView(allOf(withId(...), withText("Hello!"))).check(matches(isDisplayed()));

   若是已经明确view已经显示，这代码是好的。

Espresso官方文档：https://developer.android.com/training/testing/espresso/