# Espresso

> * **作者**：March
> * **链接**：[Espresso UI 测试框架]()
> * **邮箱**：fengqi.mao.march@gmail.com
> * **头条**：https://toutiao.io/u/425956/subjects
> * **简书**：https://www.jianshu.com/u/02f2491c607d
> * **掘金**：https://juejin.im/user/5b484473e51d45199940e2ae
> * **CSDN**：http://blog.csdn.net/u011810138
> * **SegmentFault**：https://segmentfault.com/u/maoqiqi
> * **StackOverFlow**：https://stackoverflow.com/users/8223522
>
> 著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

[Espresso](https://developer.android.google.cn/training/testing/espresso)是Google官方提供的Android UI自动化测试的框架。

## 目录

* [设置](#设置)
  * [设置您的测试环境](#设置您的测试环境)
  * [添加Espresso依赖项](#添加Espresso依赖项)
  * [依赖类库](#依赖类库)
* [使用](#使用)
  * [API组件](#API组件)
  * [得到视图](#得到视图)
  * [对视图执行操作](#对视图执行操作)
  * [断言](#断言)
* [运行测试](#运行测试)
* [适配器视图中的数据加载](#适配器视图中的数据加载)
* [常见的Espresso测试](#常见的Espresso测试)
  * [匹配另一个视图旁边的视图](#匹配另一个视图旁边的视图)
  * [匹配操作栏内的视图](#匹配操作栏内的视图)
  * [断言不显示视图](#断言不显示视图)
  * [断言视图不存在](#断言视图不存在)
  * [断言数据项不在适配器中](#断言数据项不在适配器中)
  * [使用自定义故障处理程序](#使用自定义故障处理程序)
  * [定位非默认窗口](#定位非默认窗口)
  * [在列表视图中匹配页眉或页脚](#在列表视图中匹配页眉或页脚)
* [List](#List)
* [多进程](#多进程)
* [辅助功能检查](#辅助功能检查)
* [Web](#Web)
* [Intent](#Intent)
* [闲置资源](#闲置资源)
* [备忘单](#备忘单)
* [Link](#Link)

## 设置

### 设置您的测试环境

为了避免崩溃，我们强烈建议您关闭用于测试的虚拟或物理设备上的系统动画。在您的设备上，在设置>开发者选项下，禁用以下3个设置:

* 窗口动画比例
* 过渡动画比例
* 动画师持续时间刻度

### 添加Espresso依赖项

要将Espresso依赖项添加到您的项目中，请完成以下步骤:

1. 打开应用程序的`build.gradle`文件。这通常不是顶级`build.gradle`文件而是`app/build.gradle`。
2. 在依赖项中添加以下行：

   ```
   androidTestImplementation "com.android.support.test.espresso:espresso-core:3.0.2"
   androidTestImplementation "com.android.support.test:runner:1.0.2"
   androidTestImplementation "com.android.support.test:rules:1.0.2"
   ```

   在`build.gradle`文件中的`android.defaultConfig`添加以下行：

   ```
   testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
   ```

> 注意：Android Studio新建项目一般都会依赖部分Espresso的库，但是仍然需要自己引入部分库。
例如需要引入`com.android.support.test:rules:1.0.2`库，ActivityTestRule需要导入`rules`库。

示例Gradle构建文件

```
apply plugin: 'com.android.application'

android {
    compileSdkVersion 28

    defaultConfig {
        applicationId "com.android.march.app"
        minSdkVersion 14
        targetSdkVersion 28
        versionCode 1
        versionName "1.0"

        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
    }
}

dependencies {
    androidTestImplementation "com.android.support.test.espresso:espresso-core:3.0.2"
    androidTestImplementation "com.android.support.test:runner:1.0.2"
    androidTestImplementation "com.android.support.test:rules:1.0.2"
}
```

### 依赖类库

* `espresso-core`：包含核心和基本视图匹配器、操作和断言。
* `espresso-intent`：扩展到验证和存根意图的密封测试。
* `espresso-idling-resource`：Espresso与后台作业同步的机制。
* `espresso-remote`：定位Espresso多进程功能位置。
* `espresso-web`：包含支持WebView的资源。
* `espresso-contrib`：外部贡献包括DatePicker, RecyclerView and Drawer动作, accessibility checks, and CountingIdlingResource。


## 使用

Espresso API鼓励测试作者考虑用户在与应用程序交互时可能做什么——定位UI元素并与它们交互。

### API组件

Espresso的主要组成部分包括：

* `Espresso`：与视图交互的入口点(通过onView()和onData())。还公开不一定绑定到任何视图的api，比如pressBack()。
* `ViewMatchers`：实现Matcher<? super View>接口的对象集合。您可以将其中一个或多个传递给onView()方法，以在当前视图层次结构中定位视图。
* `ViewActions`：ViewAction可以传递给ViewInteraction.perform()方法的对象集合，例如click()。
* `ViewAssertions`：可以通过ViewInteraction.check()方法传递ViewAssertion对象集合。大多数情况下，您将使用匹配断言，它使用视图匹配器来断言当前选定视图的状态。

例如：

```
onView(withId(R.id.my_view)).perform(click()).check(matches(isDisplayed()));
```

### 得到视图

在绝大多数情况下，该onView()方法采用hamcrest匹配器，该匹配器在当前视图层次结构中与一个且仅一个视图匹配。
Matchers非常强大，对于那些将它们与Mockito或JUnit一起使用的人来说很熟悉。

```
onView(withId(R.id.my_view))
```

R.id是唯一的，如果不是唯一的，则会报错。

```
java.lang.RuntimeException:
androidx.test.espresso.AmbiguousViewMatcherException
This matcher matches multiple views in the hierarchy: (withId: is <123456789>)
```

这就需要查看视图的各种属性，您可能会发现惟一可标识的属性。

例如：若是两个视图中的id相同，text内容不同，现在要获取text内容为"Hello!"的视图。

```
// 通过id,text属性来确定所要查找的EditText
onView(allOf(withId(R.id.my_view), withText("Hello!")))
```

> 注意：如果目标视图是`AdapterView`(如ListView,GridView,Spinner)，onView()方法可能无法工作。在这些情况下，应该使用onData()。

### 对视图执行操作

当您为目标视图找到合适的匹配器时，可以使用执行方法对其执行`ViewAction`实例。

例如点击视图：`onView(...).perform(click());`

您可以通过一次执行调用执行多个操作：`onView(...).perform(typeText("Hello"), click());`

如果您正在使用的视图位于ScrollView（垂直或水平）内部，请考虑需要先使用scrollTo()显示视图，例如click()和typeText()。
这可确保在继续执行其他操作之前显示视图：`onView(...).perform(scrollTo(), click());`

### 断言

可以使用check()方法应用于当前选中的视图。最常用的断言是`matches()`断言。它使用一个`ViewMatcher`对象来断言当前所选视图的状态。

例如，要检查视图是否包含文本`"Hello!"`：

```
onView(...).check(matches(withText("Hello!")));
```

> 注意:不要将“断言”放入onView()参数中。相反，要清楚地指定要在检查块中检查什么。

如果您想断言“Hello!”是视图的内容，以下被认为是不好的做法：

```
// Don't use assertions like withText inside onView.
onView(allOf(withId(...), withText("Hello!"))).check(matches(isDisplayed()));
```

另一方面，如果您想要断言带有文本“Hello!”的视图是可见的(例如在更改了视图可视性标志之后)，那么代码是可以的。

若是视图没有明确显示的情况下,不能将Assertions放到onView()中，要将Assertions放到检查代码块中。如下：

```
onView(...).check(matches(withText("Hello!")));
```

> 注意：请务必注意断言视图未显示和断言视图不在视图层次结构中存在的区别。


## 运行测试

您可以在Android Studio或命令行中运行测试。

在 Android Studio：

* 打开 Run > Edit Configurations.
* 添加一个新的Android测试配置。
* 选择一个模块。
* 添加一个特定的测试运行器:`android.support.test.runner.AndroidJUnitRunner`
* 运行新创建的配置。

从命令行：

Execute the following Gradle command:

```
./gradlew connectedAndroidTest
```


## 适配器视图中的数据加载

`AdapterView`是一种特殊类型的小部件，可以从适配器动态加载数据。一个最常见的例子AdapterView是ListView。
与LinearLayout之类的静态小部件相反，AdapterView子部件的子集只能加载到当前视图层次结构中。简单 onView()搜索将找不到当前未加载的视图。

Espresso通过提供单独的onData()入口点来处理此问题，该入口点能够首先加载有问题的适配器项目，在对其或其任何子项进行操作之前使其成为焦点。

> 注意:对于最初显示在屏幕上的适配器视图中的项，您可以选择绕过onData()加载操作，因为它们已经加载了。但是，总是使用onData()更安全。

这个简单的测试演示了如何使用onData()。SimpleActivity包含Spinner一些代表咖啡饮料类型的项目。
选择某个项目时，会有一个TextView更改为"One %s a day!"，其中%s表示所选项。

此测试的目标是打开Spinner，选择一个特定的项目，并验证TextView包含该项目。
由于Spinner该类是基于的AdapterView，建议使用onData()而不是onView()匹配项目。

* 打开项目选择：

    ```
    onView(withId(R.id.spinner_simple)).perform(click());
    ```

* 选择一个项目：

    对于项目选择，Spinner创建一个包含其内容的ListView的项目。此视图可能非常长，并且该元素可能不会提供给视图层次结构。
    通过使用onData()我们将所需的元素强制放到视图层次结构中。Spinner字符串中的项目，因此我们希望匹配一个与字符串"Americano"相等的项:

    ```
    onData(allOf(is(instanceOf(String.class)), is("Americano"))).perform(click());
    ```

* 验证文本是否正确：

    ```
    onView(withId(R.id.spinner_text_simple)).check(matches(withText(containsString("Americano"))));
    ```


## 常见的Espresso测试

介绍如何设置各种常见的Espresso测试。

### 匹配另一个视图旁边的视图

布局可以包含某些本身不是唯一的视图。例如，联系人表中的重复呼叫按钮可以具有相同的内容 R.id，包含相同的文本，并且具有与视图层次结构中的其他呼叫按钮相同的属性。

例如，在此活动中，文本为“7”的视图跨多行重复:

![repeated_view](images/repeated_view.png)

通常，非唯一视图将与位于其旁边的某个唯一标签配对，例如呼叫按钮旁边的联系人姓名。在这种情况下，您可以使用hasSibling()匹配器缩小选择范围：

```
onView(allOf(withText("7"), hasSibling(withText("item: 0")))).perform(click());
```

### 匹配操作栏内的视图

`ActionBarTestActivity`有两个不同的操作栏：一个普通操作栏和一个从选项菜单创建的上下文操作栏。
两个操作栏都有一个始终可见的项目和两个仅在溢出菜单中可见的项目。单击某个项目时，它会将TextView更改为所单击项目的内容。

```
public void testClickActionBarItem() {
    // We make sure the contextual action bar is hidden.
    onView(withId(R.id.hide_contextual_action_bar)).perform(click());

    // Click on the icon - we can find it by the r.Id.
    onView(withId(R.id.action_save)).perform(click());

    // Verify that we have really clicked on the icon by checking the TextView content.
    onView(withId(R.id.text_action_bar_result)).check(matches(withText("Save")));
}
```

上下文操作栏的代码看起来相同：

```
public void testClickActionModeItem() {
    // Make sure we show the contextual action bar.
    onView(withId(R.id.show_contextual_action_bar)).perform(click());

    // Click on the icon.
    onView((withId(R.id.action_lock))).perform(click());

    // Verify that we have really clicked on the icon by checking the TextView content.
    onView(withId(R.id.text_action_bar_result)).check(matches(withText("Lock")));
}
```

对于普通的操作栏来说，单击overflow菜单中的项目比较麻烦，因为某些设备有一个硬件overflow菜单按钮，它会打开选​​项菜单中的溢出项，
而有些设备有一个软件overflow菜单按钮，它会打开一个普通的overflow菜单。幸运的是，Espresso为我们处理这个问题。

对于普通的操作栏：

```
public void testActionBarOverflow() {
    // Make sure we hide the contextual action bar.
    onView(withId(R.id.hide_contextual_action_bar)) .perform(click());

    // Open the options menu OR open the overflow menu, depending on whether
    // the device has a hardware or software overflow menu button.
    openActionBarOverflowOrOptionsMenu(ApplicationProvider.getApplicationContext());

    // Click the item.
    onView(withText("World")).perform(click());

    // Verify that we have really clicked on the icon by checking
    // the TextView content.
    onView(withId(R.id.text_action_bar_result)).check(matches(withText("World")));
}
```

具有硬件溢出菜单按钮的操作栏：

```
public void testActionModeOverflow() {
    // Show the contextual action bar.
    onView(withId(R.id.show_contextual_action_bar)).perform(click());

    // Open the overflow menu from contextual action mode.
    openContextualActionModeOverflowMenu();

    // Click on the item.
    onView(withText("Key")).perform(click());

    // Verify that we have really clicked on the icon by
    // checking the TextView content.
    onView(withId(R.id.text_action_bar_result)).check(matches(withText("Key")));
}
```

### 断言不显示视图

在执行一系列操作之后，您肯定想要断言测试中的UI状态。有时，这可能是一个负面情况，例如当某些事情没有发生时。
请记住，您可以使用ViewAssertions.matches()将任何hamcrest视图匹配器转换为ViewAssertion。

在下面的示例中，我们使用isDisplayed()匹配器并使用标准not()匹配器将其反转：

```
import static androidx.test.espresso.Espresso.onView;
import static androidx.test.espresso.assertion.ViewAssertions.matches;
import static androidx.test.espresso.matcher.ViewMatchers.isDisplayed;
import static androidx.test.espresso.matcher.ViewMatchers.withId;
import static org.hamcrest.Matchers.not;

onView(withId(R.id.bottom_left)).check(matches(not(isDisplayed())));
```

如果视图仍然是层次结构的一部分，则上述方法有效。如果不是，你会得到一个NoMatchingViewException，你需要使用ViewAssertions.doesNotExist()。

### 断言视图不存在

如果视图从视图层次结构中消失，这可能在操作转换向另一个活动时发生，您应该使用ViewAssertions.doesNotExist()：

```
import static androidx.test.espresso.Espresso.onView;
import static androidx.test.espresso.assertion.ViewAssertions.doesNotExist;
import static androidx.test.espresso.matcher.ViewMatchers.withId;

onView(withId(R.id.bottom_left)).check(doesNotExist());
```

### 断言数据项不在适配器中

要证明某个特定的数据项不在AdapterView中，您必须以稍微不同的方式进行操作。我们必须找到我们感兴趣的AdapterView并查询它所持有的数据。
我们不需要使用onData()。相反，我们使用onView()来查找AdapterView，然后使用另一个匹配器来处理视图中的数据。

```
private static Matcher<View> withAdaptedData(final Matcher<Object> dataMatcher) {
    return new TypeSafeMatcher<View>() {

        @Override
        public void describeTo(Description description) {
            description.appendText("with class name: ");
            dataMatcher.describeTo(description);
        }

        @Override
        public boolean matchesSafely(View view) {
            if (!(view instanceof AdapterView)) {
                return false;
            }

            @SuppressWarnings("rawtypes")
            Adapter adapter = ((AdapterView) view).getAdapter();
            for (int i = 0; i < adapter.getCount(); i++) {
                if (dataMatcher.matches(adapter.getItem(i))) {
                    return true;
                }
            }

            return false;
        }
    };
}
```

```
@SuppressWarnings("unchecked")
public void testDataItemNotInAdapter() {
    onView(withId(R.id.list)).check(matches(not(withAdaptedData(withItemContent("item: 168")))));
}
```

如果具有ID列表的适配器视图中存在一个等于"item: 168"的项，则我们有一个断言将失败。

### 使用自定义故障处理程序

FailureHandler使用自定义格式替换Espresso中的默认FailureHandler，允许进行其他或不同的错误处理，例如截取屏幕截图或传递额外的调试信息。

该CustomFailureHandlerTest示例演示了如何实现自定义故障处理程序：

```
private static class CustomFailureHandler implements FailureHandler {
    private final FailureHandler delegate;

    public CustomFailureHandler(Context targetContext) {
        delegate = new DefaultFailureHandler(targetContext);
    }

    @Override
    public void handle(Throwable error, Matcher<View> viewMatcher) {
        try {
            delegate.handle(error, viewMatcher);
        } catch (NoMatchingViewException e) {
            throw new MySpecialException(e);
        }
    }
}
```

这个失败处理程序抛出一个MySpecialException而不是一个NoMatchingViewException，并将所有其他失败委托给DefaultFailureHandler。
所述CustomFailureHandler可以在测试的setUp()方法中用Espresso注册:

```
@Override
public void setUp() throws Exception {
    super.setUp();
    getActivity();
    setFailureHandler(new CustomFailureHandler(ApplicationProvider.getApplicationContext()));
}
```

### 定位非默认窗口

Android支持多个窗口。通常，这对用户和应用程序开发人员是透明的，但在某些情况下，多个窗口是可见的，例如当在搜索小部件中的主应用程序窗口上绘制自动完成窗口时。
为了简化操作，默认情况下Espresso使用启发式来猜测您打算与哪个窗口交互。
这种启发式几乎总是足够好;但是，在极少数情况下，您需要指定交互应该针对哪个窗口。你可以通过提供你自己的根窗口匹配器，或`Root`匹配器：

```
onView(withText("South China Sea")).inRoot(withDecorView(not(is(getActivity().getWindow().getDecorView())))).perform(click());
```

与ViewMatchers一样，我们提供了一组预先提供的RootMatchers。当然，您始终可以实现自己的Matcher对象。

### 在列表视图中匹配页眉或页脚

ListViews使用addHeaderView()和 addFooterView()方法添加页眉和页脚。要确保Espresso.onData()知道要匹配的数据对象，
请确保将预设数据对象值作为第二个参数传递给addHeaderView()和addFooterView()。例如：

```
public static final String FOOTER = "FOOTER";
...
View footerView = layoutInflater.inflate(R.layout.list_item, listView, false);
footerView.findViewById<TextView>(R.id.item_content).setText("count:");
footerView.findViewById<TextView>(R.id.item_size).setText(String.valueOf(data.size()));
listView.addFooterView(footerView, FOOTER, true);
```

然后，您可以为页脚编写匹配器：

```
import static org.hamcrest.Matchers.allOf;
import static org.hamcrest.Matchers.instanceOf;
import static org.hamcrest.Matchers.is;

@SuppressWarnings("unchecked")
public static Matcher<Object> isFooter() {
    return allOf(is(instanceOf(String.class)), is(LongListActivity.FOOTER));
}
```

在测试中加载视图很简单:

```
import static androidx.test.espresso.Espresso.onData;
import static androidx.test.espresso.action.ViewActions.click;
import static androidx.test.espresso.sample.LongListMatchers.isFooter;

public void testClickFooter() {
    onData(isFooter()).perform(click());
    // ...
}
```


## List

Espresso提供了针对两种类型列表(适配器视图和回收器视图)滚动到特定项目或对其进行操作的机制。

在处理列表时，特别是那些使用RecyclerView或AdapterView对象创建的列表时，您感兴趣的视图可能甚至不在屏幕上，因为在您滚动时，只会显示少量的子视图并回收它们。
在这种情况下不能不能使用scrollTo()方法，因为它需要一个现有视图。

### 与AdapterView列表项进行交互

不要使用onView()方法，而是使用onData()开始搜索，并提供一个匹配器来匹配支持您希望匹配的视图的数据。
Espresso将完成在适配器对象中查找行并使视图中的项可见的所有工作。

**使用自定义视图匹配器匹配数据**

下面的活动包含一个ListView，它由一个SimpleAdapter支持，该适配器保存Map<String, Object>对象中的每一行的数据。

![list-item](images/list-showing-all-rows.png)

每个映射都有两个条目:一个键“STR”包含一个字符串，例如“item: x”;一个键“LEN”包含一个整数，表示内容的长度。例如:

```
{"STR" : "item: 0", "LEN": 7}
```

单击“item: 50”行的代码如下:

```
onData(allOf(is(instanceOf(Map.class)), hasEntry(equalTo("STR"), is("item: 50")))).perform(click());
```

> 注意，Espresso会根据需要自动滚动列表。

让我们在onData()中分解Matcher<Object>。is(instanceOf(Map.class))方法将搜索范围缩小到AdapterView由Map对象支持的任何项。

在我们的例子中，查询的这个方面匹配列表视图的每一行，但我们想要专门点击一个项目，所以我们进一步缩小搜索范围：

```
hasEntry(equalTo("STR"), is("item: 50"))
```

这个Matcher<String, Object>将匹配任何包含键“STR”和值“item: 50”的条目的映射。
因为查找它的代码很长，我们想在其他位置重用它，让我们为它编写一个自定义 withItemContent匹配器：

```
return new BoundedMatcher<Object, Map>(Map.class) {
    @Override
    public boolean matchesSafely(Map map) {
        return hasEntry(equalTo("STR"), itemTextMatcher).matches(map);
    }

    @Override
    public void describeTo(Description description) {
        description.appendText("with item content: ");
        itemTextMatcher.describeTo(description);
    }
};
```

您使用BoundedMatcher作为基础，因为只匹配Map类型的对象。覆盖matchesSafely()方法，放入前面找到的匹配器，并将其与Matcher<String>匹配，可以将其作为参数传递。
这允许您调用withItemContent(equalTo("foo"))。为了代码简洁，您可以创建另一个matcher，它已经调用equalTo()并接受一个String对象:

```
public static Matcher<Object> withItemContent(String expectedText) {
    checkNotNull(expectedText);
    return withItemContent(equalTo(expectedText));
}
```

现在点击该项目的代码很简单：

```
onData(withItemContent("item: 50")).perform(click());
```

**匹配特定的子视图**

上面的示例在ListView的整个行中间发出单击。但是如果我们想对这一行的特定子元素进行操作呢?例如，我们想要单击该行的第二列。

![list-one](images/list-highlighting-one-col.png)

只需将onChildView()规范添加到DataInteraction的实现中:

```
onData(withItemContent("item: 60")).onChildView(withId(R.id.item_size)).perform(click());
```

### 与RecyclerView列表项进行交互

RecyclerView对象的工作方式与AdapterView对象不同，因此不能使用onData()与它们交互。

要想使用Espresso与RecyclerView视图交互，你可以使用`espresso-contrib`，其中包含RecyclerViewActions可用于滚动到位置或对项执行操作的集合：

* `scrollTo()`：滚动到匹配的视图。
* `scrollToHolder()`：滚动到匹配的ViewHolder。
* `scrollToPosition()`：滚动到特定位置。
* `actionOnHolderItem()`：在匹配的ViewHolder上执行ViewAction。
* `actionOnItem()`：在匹配的视图上执行查看操作。
* `actionOnItemAtPosition()`：在特定位置的视图上执行ViewAction。

以下代码段包含[RecyclerViewSample](https://github.com/googlesamples/android-testing/tree/master/ui/espresso/RecyclerViewSample)示例中的一些示例 ：

```
@Test
public void scrollToItemBelowFold_checkItsText() {
    // First, scroll to the position that needs to be matched and click on it.
    onView(ViewMatchers.withId(R.id.recyclerView))
            .perform(RecyclerViewActions.actionOnItemAtPosition(ITEM_BELOW_THE_FOLD,
            click()));

    // Match the text in an item below the fold and check that it's displayed.
    String itemElementText = activityRule.getActivity().getResources()
            .getString(R.string.item_element_text)
            + String.valueOf(ITEM_BELOW_THE_FOLD);
    onView(withText(itemElementText)).check(matches(isDisplayed()));
}
```

```
@Test
public void itemInMiddleOfList_hasSpecialText() {
    // First, scroll to the view holder using the isInTheMiddle() matcher.
    onView(ViewMatchers.withId(R.id.recyclerView))
            .perform(RecyclerViewActions.scrollToHolder(isInTheMiddle()));

    // Check that the item has the special text.
    String middleElementText =
            activityRule.getActivity().getResources()
            .getString(R.string.middle);
    onView(withText(middleElementText)).check(matches(isDisplayed()));
}
```


## 多进程

随着应用程序的增长，您可能会发现将一些应用程序组件放置在应用程序主进程之外的进程中非常有用。要在这些非默认进程中测试应用程序组件，可以使用多进程Multiprocess Espresso的功能。
这个工具可以在Android 8.0 (API级别26)或更高版本上使用，它允许您无缝地测试应用程序的UI交互，这些交互可以跨越应用程序的进程边界，同时保证Espresso的同步。

使用Multiprocess Espresso时，请记住以下版本控制和范围注意事项：

* 您的应用程序必须针对Android 8.0 (API级别26)或更高。
* 该工具只能测试应用程序包中包含的进程中的应用程序组件。它不能测试外部流程。


要使用Multiprocess Espresso测试应用程序中的流程，请在应用程序的build.gradle文件中添加对espresso-remote工件的引用：

```
dependencies {
    ...
    androidTestImplementation 'androidx.test.espresso:espresso-remote:3.1.0'
}
```

您还需要将以下内容添加到应用程序的androidTest清单中：

* <instrumentation>定义过程的元素。
* 一个<meta-data>元素，表示您要使用Multiprocess Espresso。

以下代码段显示了如何添加这些元素：

```
<manifest ... package="androidx.test.mytestapp.tests">
  <uses-sdk android:targetSdkVersion="27" android:minSdkVersion="14" />
  <instrumentation
    android:name="androidx.test.runner.AndroidJUnitRunner"
    android:targetPackage="androidx.test.mytestapp"
    android:targetProcesses="*">
    <meta-data
      android:name="remoteMethod"
      android:value="androidx.test.espresso.remote.EspressoRemote#remoteInit" />
  </instrumentation>
</manifest>
```

上一个代码段向Android框架表明您希望它测试应用程序包中的每个进程。如果您只想测试应用程序进程的一个子集，则可以在targetProcesses元素中指定以逗号分隔的列表。


```
<instrumentation
    ...
    android:targetProcesses=
            "androidx.test.mytestapp:myFirstAppProcessToTest,
             androidx.test.mytestapp:mySecondAppProcessToTest" ... />
```

> 注意:如果将targetProcesses设置为应用程序包的主进程，Multiprocess Espresso将忽略targetProcesses的值。


## 辅助功能检查

AccessibilityCheck类允许您使用现有的测试代码来测试可访问性问题。当您在测试期间与视图交互时，可访问性测试框架会在继续之前自动运行检查。
只需导入该类并将以下代码添加到使用@Before注释的设置方法中:

```
import androidx.test.espresso.contrib.AccessibilityChecks;

@RunWith(AndroidJUnit4.class)
@LargeTest
public class AccessibilityChecksIntegrationTest {
    @BeforeClass
    public static void enableAccessibilityChecks() {
        AccessibilityChecks.enable();
    }
}
```

这将导致在每次使用ViewActions类中的ViewAction时对给定视图运行可访问性检查。要对层次结构中的所有视图运行这些检查，请使用以下逻辑:

```
AccessibilityChecks.enable().setRunChecksFromRootView(true);
```

首次启用检查时，您可能会遇到一些您可能无法立即处理的问题。您可以通过为要抑制的结果设置匹配器来抑制这些错误。

Matchers AccessibilityCheckResult出现在辅助功能测试框架中的AccessibilityCheckResultUtils中。例如，要抑制ID为R.id.example_view的视图的所有错误:

```
AccessibilityChecks.enable().setSuppressingResultMatcher(matchingViews(withId(R.id.example_view)));
```


## Web

Espresso-Web是使用Android WebView UI组件的入口点。Espresso-Web重用了流行的WebDriver API中的 Atoms 来检查和控制WebView的行为。

### 在你的项目中加入Espresso-Web

在`app/build.gradle`文件的`dependencies`中添加以下行：

```
androidTestImplementation "com.android.support.test.espresso:espresso-web:3.1.0"
```

Espresso-web只兼容`Espresso 2.2`或更高版本以及测试库的0.3或更高版本，因此请确保您也更新这些行：

```
androidTestImplementation "com.android.support.test.espresso:espresso-core:3.0.2"
androidTestImplementation "com.android.support.test:runner:1.0.2"
androidTestImplementation "com.android.support.test:rules:1.0.2"
```

### 常见API

onWebView()方法是在Android上使用Espresso处理WebView时的主要入口点。您可以使用此方法执行Espresso-Web测试，例如：

```
onWebView()
    .withElement(findElement(Locator.ID, "link_2")) // similar to onView(withId(...))
    .perform(webClick()) // Similar to perform(click())

    // Similar to check(matches(...))
    .check(webMatches(getCurrentUrl(), containsString("navigation_2.html")));
```

在本例中，Espresso-Web找到ID为“link_2”的DOM元素，并单击它。然后，该工具验证WebView是否发送包含该"navigation_2.html"字符串的GET请求。

> 注意:在执行测试时，系统使用JavaScript执行所有WebView交互。因此，为了支持JavaScript评估，测试中的WebView必须启用JavaScript。


您可以通过重写ActivityTestRule中的afterActivityLaunched()并调用onWebView().forceJavascriptEnabled()来强制启用JavaScript。
启用JavaScript可能会导致重新加载受测试的WebView。这是必需的，以确保AndroidJUnitRunner加载所有必需的测试基础架构（包括JavaScript交互）：

```
@Rule
public ActivityTestRule<WebViewActivity> activityRule = new ActivityTestRule<WebViewActivity>(WebViewActivity.class, false, false) {
    @Override
    protected void afterActivityLaunched() {
        onWebView().forceJavascriptEnabled();
    }
}
```

### 常见的交互

与Web.WebInteraction对象的常见交互包括以下内容：

* `withElement()`引用WebView中的DOM元素。

  ```
  onWebView().withElement(findElement(Locator.ID, "teacher"));
  ```

* [`withContextualElement()`]

  (/reference/androidx/test/espresso/web/sugar/Web.WebInteraction.html#withContextualElement(androidx.test.espresso.web.model.Atom))
  引用WebView中的一个作用域DOM元素，相对于另一个DOM元素。您应该首先调用withElement()以建立引用Web.WebInteraction对象（DOM元素）。

  ```
  onWebView().withElement(findElement(Locator.ID, "teacher")).withContextualElement(findElement(Locator.ID, "person_name"));
  ```

* [`check()`]

  (/reference/androidx/test/espresso/web/sugar/Web.WebInteraction.html#check(androidx.test.espresso.web.assertion.WebAssertion)
  评估一个条件，确保它解析为true。

  ```
  onWebView()
      .withElement(findElement(Locator.ID, "teacher"))
      .withContextualElement(findElement(Locator.ID, "person_name"))
      .check(webMatches(getText(), containsString("Socrates")));
  ```

* [`perform()`]

  (/reference/androidx/test/espresso/web/sugar/Web.WebInteraction.html#perform(androidx.test.espresso.web.model.Atom))
  在WebView中执行操作，例如单击元素。

  ```
  onWebView().withElement(findElement(Locator.ID, "teacher")).perform(webClick());
  ```

* `reset()`

  将WebView恢复到其初始状态。当先前的操作（如单击）引入导致ElementReference和WindowReference对象不可访问的导航更改时，这是必要的。

  ```
  onWebView().withElement(...).perform(...).reset();
  ```

### 示例

下面的示例测试在WebView中输入文本并选择提交按钮后，同一文本是否出现在同一WebView中的不同元素中：

```
public static final String MACCHIATO = "Macchiato";

@Test
public void typeTextInInput_clickButton_SubmitsForm() {
    // Lazily launch the Activity with a custom start Intent per test.
    activityRule.launchActivity(withWebFormIntent());

    // Selects the WebView in your layout. If you have multiple WebView objects,
    // you can also use a matcher to select a given WebView,
    // onWebView(withId(R.id.web_view)).
    onWebView()
        // Find the input element by ID.
        .withElement(findElement(Locator.ID, "text_input"))

        // Clear previous input and enter new text into the input element.
        .perform(clearElement())
        .perform(DriverAtoms.webKeys(MACCHIATO))

        // Find the "Submit" button and simulate a click using JavaScript.
        .withElement(findElement(Locator.ID, "submitBtn"))
        .perform(webClick())

        // Find the response element by ID, and verify that it contains the
        // entered text.
        .withElement(findElement(Locator.ID, "response"))
        .check(webMatches(getText(), containsString(MACCHIATO)));
}
```


## Intent

Espresso-Intents是Espresso的扩展，它可以验证和存储被测试应用程序发出的意图。这有点像Mockito，但针对Android的意图。

如果您的应用将功能委托给其他应用或平台，您可以使用Espresso-Intents专注于您自己应用的逻辑，同时假设其他应用或平台将正常运行。
使用Espresso-Intents，您可以匹配并验证您的传出意图，甚至可以提供存根响应来代替实际意图响应。

### 在你的项目中加入Espresso-Intents

在`app/build.gradle`文件的`dependencies`中添加以下行：

```
androidTestImplementation 'com.android.support.test.espresso:espresso-intents:3.1.0'
```

Espresso-Intents只兼容`Espresso 2.1+`和Android测试库的0.3+，因此请确保您也更新这些行：

```
androidTestImplementation "com.android.support.test.espresso:espresso-core:3.0.2"
androidTestImplementation "com.android.support.test:runner:1.0.2"
androidTestImplementation "com.android.support.test:rules:1.0.2"
```

### 使用Espresso-Intents

在编写Espresso-Intents测试之前，请设置一个IntentsTestRule。这是ActivityTestRule类的扩展，可以在功能UI测试中轻松使用Espresso-Intents API。
IntentsTestRule在使用@Test注释的每个测试之前初始化Espresso-Intents，并在每次测试运行之后释放Espresso-Intents。

**编写测试规则**

```
@Rule
public IntentsTestRule<MyActivity> intentsTestRule = new IntentsTestRule<>(MyActivity.class);
```

**Match**

Espresso-Intents提供了根据某些匹配条件拦截输出意图的能力，这些匹配条件是使用Hamcrest Matchers定义的。Hamcrest允许您：

* 使用现有的意图匹配器：最简单的选项，几乎总是首选。
* 实现自己的意图匹配器：最灵活的选择。请参阅[Hamcrest教程中](https://code.google.com/archive/p/hamcrest/wikis/Tutorial.wiki)标题为“编写自定义匹配器”的部分。

Espresso-Intents分别提供了意图验证和存根的方法intended() 和intending()方法。两者都以Hamcrest Matcher<Intent>对象作为参数。

下面的代码片段显示了意图验证，它使用现有的意图匹配器来匹配启动浏览器的外向意图:

```
assertThat(intent).hasAction(Intent.ACTION_VIEW);
assertThat(intent).categories().containsExactly(Intent.CATEGORY_BROWSABLE);
assertThat(intent).hasData(Uri.parse("www.google.com"));
assertThat(intent).extras().containsKey("key1");
assertThat(intent).extras().string("key1").isEqualTo("value1");
assertThat(intent).extras().containsKey("key2");
assertThat(intent).extras().string("key2").isEqualTo("value2");
```

**验证**

Espresso-Intents记录所有尝试从被测应用程序启动活动的意式。使用intended()方法(类似于Mockito.verify())，您可以断言已经看到给定的意图。
但是，除非您显式地配置了意图，否则Espresso-Intents不会排除对意图的响应。

```
@Test
public void validateIntentSentToPackage() {
    // User action that results in an external "phone" activity being launched.
    user.clickOnView(system.getView(R.id.callButton));

    // Using a canned RecordedIntentMatcher to validate that an intent resolving
    // to the "phone" activity has been sent.
    intended(toPackage("com.android.phone"));
}
```

**Stubbing**

使用与Mockito.when()类似的intending()方法，可以为使用startActivityForResult()启动的活动提供存根响应。
这对于外部活动特别有用，因为您既不能操作外部活动的用户界面，也不能控制返回给测试活动的ActivityResult。

下面的代码验证了当用户在被测试的应用程序中启动一个“contact”活动时，会显示联系人电话:

1. 构建要在启动特定活动时返回的结果。示例测试截取发送给“contacts”的所有意图，并使用结果代码RESULT_OK用一个有效的ActivityResult列出它们的响应：

  ```
  Intent resultData = new Intent();
  String phoneNumber = "123-345-6789";
  resultData.putExtra("phone", phoneNumber);
  ActivityResult result = new ActivityResult(Activity.RESULT_OK, resultData);
  ```

2. 指示Espresso针对“contacts”意图的所有调用提供存根结果对象:

  ```
  intending(toPackage("com.android.contacts")).respondWith(result);
  ```

3. 验证用于启动活动的操作是否生成预期的存根结果。在本例中，示例测试检查在启动“contacts activity”时是否返回并显示电话号码“123-345-6789”:

  ```
  onView(withId(R.id.pickButton)).perform(click());
  onView(withId(R.id.phoneNumber)).check(matches(withText(phoneNumber)));
  ```

**完整的测试**

```
@Test
public void activityResult_DisplaysContactsPhoneNumber() {
    // Build the result to return when the activity is launched.
    Intent resultData = new Intent();
    String phoneNumber = "123-345-6789";
    resultData.putExtra("phone", phoneNumber);
    ActivityResult result =new ActivityResult(Activity.RESULT_OK, resultData);

    // Set up result stubbing when an intent sent to "contacts" is seen.
    intending(toPackage("com.android.contacts")).respondWith(result);

    // User action that results in "contacts" activity being launched.
    // Launching activity expects phoneNumber to be returned and displayed.
    onView(withId(R.id.pickButton)).perform(click());

    // Assert that the data we set up above is shown.
    onView(withId(R.id.phoneNumber)).check(matches(withText(phoneNumber)));
}
```

## 闲置资源

空闲资源表示异步操作，其结果影响UI测试中的后续操作。通过使用Espresso注册空闲资源，您可以在测试应用程序时更可靠地验证这些异步操作。

### 确定何时需要闲置资源

Espresso提供了一套复杂的同步功能。然而，框架的这种特性只适用于在MessageQueue上发布消息的操作，比如在屏幕上绘制其内容的视图子类。

因为Espresso不知道任何其他异步操作，包括在后台线程上运行的操作，所以在这些情况下，Espresso不能提供同步保证。
为了让Espresso知道你的应用程序的长时间运行，你必须把每一个都注册为一个空闲资源。

如果您在测试应用程序异步工作的结果时没有使用空闲资源，那么您可能会发现必须使用以下糟糕的解决方案之一来提高测试的可靠性:

* **添加对`Thread.sleep()`的调用**。当您在测试中添加人为的延迟时，您的测试套件需要更长的时间来完成执行，并且当您在较慢的设备上执行测试时，您的测试有时可能仍然会失败。
  此外，这些延迟不能很好地扩展，因为您的应用程序在未来的版本中可能需要执行更耗时的异步工作。
* **实现重试包装器**。它使用循环来反复检查应用程序是否仍在执行异步工作，直到超时为止。即使在测试中指定了最大重试计数，每次重新执行都会消耗系统资源，特别是CPU。
* **使用CountDownLatch实例**。它允许一个或多个线程等待，直到在另一个线程中执行的特定数量的操作完成。
  这些对象要求您指定超时长度;否则，您的应用程序可能会被无限期地阻塞。锁存器还增加了代码不必要的复杂性，使维护更加困难。

Espresso允许您从测试中删除这些不可靠的工作区，而是将应用程序的异步工作注册为空闲资源。

### 常见用例

当在测试中执行类似于以下示例的操作时，请考虑使用空闲资源:

* 从Internet或本地数据源加载数据。
* 与数据库和回调建立连接。
* 使用系统服务或IntentService实例管理服务。
* 执行复杂的业务逻辑，例如位图转换。

当这些操作更新测试验证的UI时，注册空闲资源尤其重要。

> 注意：某些第三方库使用需要手动线程管理的线程方案。在这些情况下，添加线程逻辑以实现与直接使用空闲资源所享受的相同的功能好处。

### 空闲资源实现示例

下面的列表描述了几种闲置资源的示例实现，您可以将这些资源集成到您的应用程序中:

* CountingIdlingResource

  维护活动任务的计数器。当计数器为零时，关联的资源被认为是空闲的。这个功能非常类似于信号量。在大多数情况下，这个实现足以在测试期间管理应用程序的异步工作。

* UriIdlingResource

  类似于CountingIdlingResource，但在资源被视为空闲之前，计数器需要在特定时间段内为零。此额外等待时间会考虑连续的网络请求，其中线程中的应用程序可能会在收到对先前请求的响应后立即发出新请求。

* IdlingThreadPoolExecutor

  ThreadPoolExecutor的自定义实现，它跟踪创建的线程池中运行的任务总数。该类使用CountingIdlingResource来维护活动任务的计数器。

* IdlingScheduledThreadPoolExecutor

  ScheduledThreadPoolExecutor的自定义实现。它提供了与IdlingThreadPoolExecutor类相同的功能，但它还可以跟踪为将来安排的计划或计划定期执行的任务。

> 注意:只有当Espresso首次调用该资源的isIdleNow()方法时，与这些空闲资源实现相关联的同步优势才会生效。因此，您必须在需要这些空闲资源之前注册它们。

### 创建自己的空闲资源

在应用程序的测试中使用空闲资源时，可能需要提供自定义资源管理或日志记录。
在这些情况下，上一节中列出的实现可能还不够。如果是这种情况，您可以扩展其中一个空闲资源实现或创建自己的实现。

如果您实现自己的空闲资源功能，请记住以下最佳实践，尤其是第一个：

**在空闲检查之外调用转换到空闲状态**。

在应用程序空闲之后，在isIdleNow()的任何实现之外调用onTransitionToIdle()。这样，Espresso就不需要进行第二次不必要的检查来确定给定的空闲资源是否空闲。

下面的代码片段说明了这一建议:

```
public void isIdle() {
    // DON'T call callback.onTransitionToIdle() here!
}

public void backgroundWorkDone() {
    // Background work finished.
    callback.onTransitionToIdle() // Good. Tells Espresso that the app is idle.

    // Don't do any post-processing work beyond this point. Espresso now
    // considers your app to be idle and moves on to the next test action.
}
```

**在需要之前注册空闲资源**。

只有在Espresso第一次调用该资源的isIdleNow()方法之后，与空闲资源相关的同步优势才会生效。

以下列表显示了此属性的几个示例：

* 如果您在@Before注释的方法中注册了一个空闲资源，则空闲资源将在每个测试的第一行生效。
* 如果在测试中注册了空闲资源，则在下一次基于Espresso的操作期间，空闲资源将生效。即使下一个操作与注册空闲资源的语句在同一测试中，仍会出现此行为。

**完成使用后取消注册闲置资源**。

为了节省系统资源，您应该在不再需要闲置资源时注销它们。例如，如果在用@Before注释的方法中注册一个空闲资源，则最好在用@After注释的相应方法中注销该资源。

**使用空闲注册表注册和取消注册空闲资源**。

通过将此容器用于应用程序的空闲资源，您可以根据需要重复注册和取消注册空闲资源，并且仍然可以观察到一致的行为。

**在空闲资源中仅维护简单的应用程序状态**。

例如，您实现和注册的空闲资源不应包含视图对象的引用。

### 注册空闲资源

Espresso提供了一个容器类，您可以在其中放置应用程序的空闲资源。这个类名为IdlingRegistry，是一个自包含的构件，它为您的应用程序带来了最小的开销。
该类还允许您采取以下步骤来提高应用程序的可维护性:

* 在应用程序的测试中创建对IdlingRegistry所包含的空闲资源的引用，而不是它所包含的空闲资源。
* 维护用于每个构建变体的空闲资源集合的差异。
* 在应用程序的服务中定义空闲资源，而不是在引用这些服务的UI组件中定义。

> 注意:IdlingRegistry类是唯一支持的注册应用程序空闲资源的方法。

### 将空闲资源集成到您的应用中

尽管可以通过几种不同的方式向应用程序添加空闲资源，但有一种方法可以维护应用程序的封装，同时仍允许您指定给定空闲资源所代表的特定操作。

* 推荐的方法

  在向应用程序中添加空闲资源时，我们强烈建议您仅在测试中执行注册和取消注册操作，并将空闲资源逻辑本身放在应用的生产代码中。

  尽管按照这种方法在生产代码中创建了仅测试界面的异常情况，但是您可以将闲置的资源封装到已有的代码中，从而维护应用程序的APK大小和方法计数。

* 替代方法

如果你不希望在你的应用程序的生产代码中有闲置资源的逻辑，那么还有其他几种可行的集成策略:

* 创建构建变体，例如Gradle的 产品风格，并仅在应用程序的调试版本中使用空闲资源。
* 使用像Dagger这样的依赖注入框架将应用程序的空闲资源依赖关系图注入到测试中。如果你使用Dagger 2，注射本身应该来自一个子组件。
* 在应用程序的测试中实现空闲资源，并公开应用程序实现中需要在这些测试中同步的部分。

警告：虽然这个设计决策似乎创建了一个自包含引用空闲资源，但它也打破了除最简单的应用程序之外的所有应用程序的封装。


## 备忘单

[Espresso备忘单](https://developer.android.com/training/testing/espresso/cheat-sheet)是您可以在开发过程中使用的快速参考资料。
此备忘单包含大多数可用的Matcher、ViewAction和ViewAssertion实例。
[离线PDF版本](https://android.github.io/android-test/downloads/espresso-cheat-sheet-2.1.0.pdf)。


## Link

* [Android Testing Templates](https://github.com/googlesamples/android-testing-templates)
* [Android Test Library](https://github.com/android/android-test)
* [Mock 测试](JAVA_MOCK.md)
* [安卓架构组件 Room 持久化类库](JAVA_MOCK.md)