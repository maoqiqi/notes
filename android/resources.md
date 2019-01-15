# 应用程序资源

## 目录

* [提供资源](#提供资源)
  * [分组资源类型](#分组资源类型)
  * [提供备用资源](#提供备用资源)
    * [限定符命名规则](#限定符命名规则)
    * [创建别名资源](#创建别名资源)
  * [利用资源提供最佳设备兼容性](#利用资源提供最佳设备兼容性)
  * [Android如何查找最佳匹配资源](#Android如何查找最佳匹配资源)
* [访问资源](#访问资源)
  * [在代码中访问资源](#在代码中访问资源)
  * [在XML中访问资源](#在XML中访问资源)
  * [引用样式属性](#引用样式属性)
  * [访问原始文件](#访问原始文件)
  * [访问平台资源](#访问平台资源)

## 提供资源

您应该始终外部化应用资源，例如图像和代码中的字符串，这样有利于您单独维护这些资源。
此外，您还应该为特定设备配置提供备用资源，方法是将它们分组到专门命名的资源目录中。
在运行时，Android 会根据当前配置使用适当的资源。
例如，您可能需要根据屏幕尺寸提供不同的 UI 布局，或者根据语言设置提供不同的字符串。

### 分组资源类型

项目 res/ 目录内支持的资源目录:

|目录|资源类型|
|--------|-------|
|animator|用于定义属性动画的 XML 文件。|
|anim    |定义渐变动画的 XML 文件。（属性动画也可以保存在此目录中，但是为了区分这两种类型，属性动画首选 animator 目录。）|
|anim    |用于定义颜色状态列表的 XML 文件。|
|drawable|位图文件、九宫格（可调整大小的位图）、状态列表、形状、动画可绘制对象、其他可绘制对象。|
|mipmap  |适用于不同启动器图标密度的可绘制对象文件。|
|layout  |用于定义用户界面布局的 XML 文件。|
|menu    |用于定义应用菜单（如选项菜单、上下文菜单或子菜单）的 XML 文件。|
|raw     |要以原始形式保存的任意文件。要使用原始 InputStream 打开这些资源，请使用资源 ID（即 R.raw.filename）调用 Resources.openRawResource()。但是，如需访问原始文件名和文件层次结构，则可以考虑将某些资源保存在 assets 目录下（而不是 res/raw）。assets 中的文件没有资源 ID，因此您只能使用 AssetManager 读取这些文件。|
|values  |包含字符串、整型数和颜色等简单值的 XML 文件。|
|xml     |可以在运行时通过调用 Resources.getXML() 读取的任意 XML 文件。各种 XML 配置文件（如可搜索配置）都必须保存在此处。

表格中定义的子目录下的资源是“默认”资源。即，这些资源定义应用的默认设计和内容。但是，采用 Android 技术的不同设备类型可能需要不同类型的资源。
例如，如果设备的屏幕尺寸大于标准屏幕，则应提供不同的布局资源，以充分利用额外的屏幕空间。
或者，如果设备的语言设置不同，则应提供不同的字符串资源，以转换用户界面中的文本。 要为不同的设备配置提供这些不同资源，除了默认资源以外，您还需要提供备用资源。

### 提供备用资源

几乎每个应用都应提供备用资源以支持特定的设备配置。
例如，对于不同的屏幕密度和语言，您应分别包括备用可绘制对象资源和备用字符串资源。
在运行时，Android 会检测当前设备配置并为应用加载合适的资源。

为一组资源指定特定于配置的备用资源：

1.在 res 中创建一个以 <resources_name>-<config_qualifier> 形式命名的新目录。
<resources_name> 是相应默认资源的目录名称。<qualifier> 是指定要使用这些资源的各个配置的名称。
您可以追加多个 <qualifier>，以短划线将其分隔。

2.将相应的备用资源保存在此新目录下。这些资源文件的名称必须与默认资源文件完全一样。

例如，以下是一些默认资源和备用资源：

```
res/
    drawable/
        icon.png
        background.png
    drawable-hdpi/
        icon.png
        background.png
```

hdpi 限定符表示该目录中的资源适用于屏幕密度较高的设备。
其中每个可绘制对象目录中的图像已针对特定的屏幕密度调整大小，但是文件名完全相同。
这样一来，用于引用 icon.png 或 background.png 图像的资源 ID 始终相同，
但是 Android 会通过将设备配置信息与资源目录名称中的限定符进行比较，选择最符合当前设备的各个资源版本。

**注意**：注意：追加多个限定符时，必须按照一定的顺序顺序放置它们。如果限定符的顺序错误，则该资源将被忽略。
查看[android guide 提供备用资源](https://developer.android.google.cn/guide/topics/resources/providing-resources#AlternativeResources)。

#### 限定符命名规则

以下是一些关于使用配置限定符名称的规则：

* 您可以为单组资源指定多个限定符，并使用短划线分隔。例如，drawable-en-rUS-land 适用于横排美国英语设备。
* 这些限定符必须遵循上述链接列出的顺序。例如：错误：drawable-hdpi-port/。正确：drawable-port-hdpi/。
* 不能嵌套备用资源目录。例如，您不能拥有 res/drawable/drawable-en/。
* 值不区分大小写。在处理之前，资源编译器会将目录名称转换为小写，以避免不区分大小写的文件系统出现问题。名称中使用的任何大写字母只是为了便于认读。
* 对于每种限定符类型，仅支持一个值。例如，若要对西班牙语和法语使用相同的可绘制对象文件，则您肯定不能拥有名为 drawable-rES-rFR/ 的目录，而是需要两个包含相应文件的资源目录，如 drawable-rES/ 和 drawable-rFR/。
  然而，实际上您无需将相同的文件都复制到这两个位置。相反，您可以创建指向资源的别名。请参阅下面的[请参阅下面的创建别名资源](创建别名资源)。

将备用资源保存到以这些限定符命名的目录中之后，Android 会根据当前设备配置在应用中自动应用这些资源。
每次请求资源时，Android 都会检查备用资源目录是否包含所请求的资源文件，然后[查找最佳匹配资源](#Android如何查找最佳匹配资源)（下文进行介绍）。
如果没有与特定设备配置匹配的备用资源，则 Android 会使用相应的默认资源（一组用于不含配置限定符的特定资源类型的资源）。

#### 创建别名资源

如果您想将某一资源用于多种设备配置（但是不想作为默认资源提供），则无需将同一资源放入多个备用资源目录中。
相反，您可以（在某些情况下）创建备用资源，充当保存在默认资源目录下的资源的别名。

**注意**：并非所有资源都会提供相应机制让您创建指向其他资源的别名。特别是，xml/ 目录中的动画资源、菜单资源、原始资源以及其他未指定资源均不提供此功能。

例如，假设您有一个应用图标 icon.png，并且需要不同语言区域的独特版本。但是，加拿大英语和加拿大法语这两种语言区域需要使用同一版本。
您可能会认为需要将相同的图像复制到加拿大英语和加拿大法语对应的资源目录中，但事实并非如此。
相反，您可以将用于二者的图像另存为 icon_ca.png（除 icon.png 以外的任何名称），并将其放入默认 res/drawable/ 目录中。
然后，在 res/drawable-en-rCA/ 和 res/drawable-fr-rCA/ 中创建 icon.xml 文件，使用 <bitmap> 元素引用 icon_ca.png 资源。
这样，您只需存储 PNG 文件的一个版本和两个指向该版本的小型 XML 文件。（XML 文件示例如下。）

**可绘制对象**

要创建指向现有可绘制对象的别名，请使用 <bitmap> 元素。例如：

```
<?xml version="1.0" encoding="utf-8"?>
<bitmap xmlns:android="http://schemas.android.com/apk/res/android"
    android:src="@drawable/icon_ca" />
```

如果将此文件另存为 icon.xml（例如，在备用资源目录中，另存为 res/drawable-en-rCA/），则会编译到可作为 R.drawable.icon 引用的资源中，
但实际上它是 R.drawable.icon_ca 资源（保存在 res/drawable/ 中）的别名。

**布局**

要创建指向现有布局的别名，请使用包装在 <merge> 中的 <include> 元素。例如：

```
<?xml version="1.0" encoding="utf-8"?>
<merge>
    <include layout="@layout/main_ltr"/>
</merge>
```

如果将此文件另存为 main.xml，则会编译到可作为 R.layout.main 引用的资源中，但实际上它是 R.layout.main_ltr 资源的别名。

**字符串和其他简单值**

要创建指向现有字符串的别名，只需将所需字符串的资源 ID 用作新字符串的值即可。例如：

```
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="hello">Hello</string>
    <string name="hi">@string/hello</string>
</resources>
```

R.string.hi 资源现在是 R.string.hello 的别名。

其他简单值的原理相同。不再一一细说。

### 利用资源提供最佳设备兼容性

要使应用支持多种设备配置，则务必为应用使用的每种资源类型提供默认资源，这一点非常重要。

例如，如果应用支持多种语言，请始终包含不带语言和区域限定符的 values/ 目录（用于保存字符串）。
相反，如果您将所有字符串放入带有语言和区域限定符的目录中，则在语言设置不支持您的字符串的设备上运行应用时，应用将会崩溃。
但是，只要提供默认 values/ 资源，应用就会正常运行（即使用户不理解该语言，这也总比崩溃要好）。

同样，如果您根据屏幕方向提供不同的布局资源，则应选择一个方向作为默认方向。
例如，不要在 layout-land/ 和 layout-port/ 中分别提供横向和纵向的布局资源，而是保留其中之一作为默认设置，
例如：layout/ 用于横向，layout-port/ 用于纵向。

提供默认资源至关重要，这不仅仅因为应用可能在超出预期的配置上运行，也因为新版 Android 有时会添加旧版本不支持的配置限定符。
若要使用新的资源限定符，又希望维持对旧版 Android 的代码兼容性，
则当旧版 Android 运行应用时，如果不提供默认资源，应用将会崩溃，这是因为它无法使用以新限定符命名的资源。
例如，如果将 minSdkVersion 设置为 4，并使用夜间模式（night 或 notnight，API 级别 8 中新增配置）限定所有可绘制对象资源，
则 API 级别 4 设备无法访问可绘制对象资源，而且会崩溃。
在这种情况下，您可能希望 notnight 成为默认资源，为此，您应排除该限定符，使可绘制对象资源位于 drawable/ 或 drawable-night/ 中。

因此，为了提供最佳设备兼容性，请始终为应用正确运行所必需的资源提供默认资源。然后，使用配置限定符为特定的设备配置创建备用资源。

这条规则有一个例外：如果应用的 minSdkVersion 为 4 或更高版本，则在提供带屏幕密度限定符的备用可绘制对象资源时，不需要默认可绘制对象资源。
即使没有默认可绘制对象资源，Android 也可以从备用屏幕密度中找到最佳匹配项并根据需要缩放位图。
但是，为了在所有类型的设备上提供最佳体验，您应该为所有三种类型的密度提供备用可绘制对象。

### Android如何查找最佳匹配资源

当您请求要为其提供备用资源的资源时，Android 会根据当前的设备配置选择要在运行时使用的备用资源。
为演示 Android 如何选择备用资源，假设以下可绘制对象目录分别包含相同图像的不同版本：

```
drawable/
drawable-en/
drawable-fr-rCA/
drawable-en-port/
drawable-en-notouch-12key/
drawable-port-ldpi/
drawable-port-notouch-12key/
```

同时，假设设备配置如下：

```
语言区域 = en-GB
屏幕方向 = port
屏幕像素密度 = hdpi
触摸屏类型 = notouch
主要文本输入法 = 12key
```

通过将设备配置与可用的备用资源进行比较，Android 从 drawable-en-port 中选择可绘制对象。
具体分析过程参考[android guide 如何查找最佳匹配资源](https://developer.android.google.cn/guide/topics/resources/providing-resources#BestMatch)。


## 访问资源

您在应用中提供资源后（[提供资源](#提供资源)中对此做了阐述），可通过引用其资源 ID 来应用该资源。所有资源 ID 都在您项目的 R 类中定义，后者由 aapt 工具自动生成。

编译应用时，aapt 会生成 R 类，其中包含您的 res/ 目录中所有资源的资源 ID。每个资源类型都有对应的 R 子类（例如，R.drawable 对应于所有可绘制对象资源），
而该类型的每个资源都有对应的静态整型数（例如，R.drawable.icon）。这个整型数就是可用来检索资源的资源 ID。

尽管资源 ID 是在 R 类中指定的，但您应该永远都不需要在其中查找资源 ID。 资源 ID 始终由以下部分组成：

* 资源类型：每个资源都被分到一个“类型”组中，例如 string、drawable 和 layout。
* 资源名称，它是不包括扩展名的文件名；或是 XML android:name 属性中的值，如果资源是简单值的话（例如字符串）。

访问资源的方法有两种：

* **在代码中**：使用来自 R 类的某个子类的静态整型数，例如：

  ```
  R.string.hello
  ```

  `string`是资源类型，`hello`是资源名称。当您提供此格式的资源 ID 时，有许多 Android API 可以访问您的资源。请参阅[在代码中访问资源](#在代码中访问资源)。

* **在XML中**：使用同样与您 R 类中定义的资源 ID 对应的特殊 XML 语法，例如：

  ```
  @string/hello
  ```

  `string`是资源类型，`hello`是资源名称。您可以在 XML 资源中任何应该存在您在资源中所提供值的地方使用此语法。请参阅[在XML中访问资源](#在XML中访问资源)。


### 在代码中访问资源

您可以通过以方法参数的形式传递资源 ID，在代码中使用资源。例如，您可以设置一个 ImageView，以利用 setImageResource() 使用 res/drawable/myimage.png 资源：

```
ImageView imageView = (ImageView) findViewById(R.id.myimageview);
imageView.setImageResource(R.drawable.myimage);
```

您还可以利用 Resources 中的方法检索个别资源，您可以通过 getResources() 获得资源实例。

#### 语法

以下是在代码中引用资源的语法：

```
[<package_name>.]R.<resource_type>.<resource_name>
```

* <package_name> 是资源所在包的名称（如果引用的资源来自您自己的资源包，则不需要）。
* <resource_type> 是资源类型的 R 子类。
* <resource_name> 是不带扩展名的资源文件名，或 XML 元素中的 android:name 属性值（如果资源是简单值）。

#### 用例

有许多方法接受资源 ID 参数，您可以利用 Resources 中的方法检索资源。您可以通过 Context.getResources() 获得 Resources 的实例。

以下是一些在代码中访问资源的示例：

```
// Load a background for the current screen from a drawable resource
getWindow().setBackgroundDrawableResource(R.drawable.my_background_image) ;

// Set the Activity title by getting a string from the Resources object, because
//  this method requires a CharSequence rather than a resource ID
getWindow().setTitle(getResources().getText(R.string.main_title));

// Load a custom layout for the current screen
setContentView(R.layout.main_screen);

// Set a slide in animation by getting an Animation from the Resources object
mFlipper.setInAnimation(AnimationUtils.loadAnimation(this,
        R.anim.hyperspace_in));

// Set the text on a TextView object using a resource ID
TextView msgTextView = (TextView) findViewById(R.id.msg);
msgTextView.setText(R.string.hello_message);
```

> **注意**：切勿手动修改 R.java 文件 — 它是在编译您的项目时由 aapt 工具生成的。您下次编译时所有更改都会被替代。

### 在XML中访问资源

您可以利用对现有资源的引用为某些 XML 属性和元素定义值。创建布局文件时，为给您的小部件提供字符串和图像，您经常要这样做。

例如，如果您为布局添加一个 Button，应该为按钮文本使用字符串资源：

```
<Button
    android:layout_width="fill_parent"
    android:layout_height="wrap_content"
    android:text="@string/submit" />
```

#### 语法

以下是在 XML 资源中引用资源的语法：

```
@[<package_name>:]<resource_type>/<resource_name>
```

* <package_name> 是资源所在包的名称（如果引用的资源来自相同的包，则不需要）
* <resource_type> 是资源类型的 R 子类
* <resource_name> 是不带扩展名的资源文件名，或 XML 元素中的 android:name 属性值（如果资源是简单值）。

#### 用例

在某些情况下，您必须使用资源作为 XML 中的值（例如，对小部件应用可绘制图像），但您也可以在 XML 中任何接受简单值的地方使用资源。例如，如果您具有以下资源文件，其中包括一个颜色资源和一个字符串资源：

```
<?xml version="1.0" encoding="utf-8"?>
<resources>
   <color name="opaque_red">#f00</color>
   <string name="hello">Hello!</string>
</resources>
```

您可以在以下布局文件中使用这些资源来设置文本颜色和文本字符串：

```
<?xml version="1.0" encoding="utf-8"?>
<EditText xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
    android:textColor="@color/opaque_red"
    android:text="@string/hello" />
```

在此情况下，您无需在资源引用中指定包名称，因为资源来自您自己的资源包。 要引用系统资源，您需要加入包名称。 例如：

```
<?xml version="1.0" encoding="utf-8"?>
<EditText xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
    android:textColor="@android:color/secondary_text_dark"
    android:text="@string/hello" />
```

您甚至可以在 XML 中使用资源创建别名。例如，您可以创建一个可绘制对象资源，将其作为另一个可绘制对象资源的别名：

```
<?xml version="1.0" encoding="utf-8"?>
<bitmap xmlns:android="http://schemas.android.com/apk/res/android"
    android:src="@drawable/other_drawable" />
```

这听起来多余，但对使用备用资源可能很有帮助。

### 引用样式属性

您可以通过样式属性资源在当前应用的风格主题中引用某个属性的值。 通过引用样式属性，您可以不采用为 UI 元素提供硬编码值这种方式，而是通过为 UI 元素设置样式，使其匹配当前风格主题提供的标准变型来定制这些元素的外观。引用样式属性的实质作用是，“在当前风格主题中使用此属性定义的样式”。

要引用样式属性，名称语法几乎与普通资源格式完全相同，只不过将 at 符号 (@) 改为问号 (?)，资源类型部分为可选项。 例如：

```
?[<package_name>:][<resource_type>/]<resource_name>
```

例如，您可以通过以下代码引用一个属性，将文本颜色设置为与系统风格主题的“主要”文本颜色匹配：

```
<EditText id="text"
    android:layout_width="fill_parent"
    android:layout_height="wrap_content"
    android:textColor="?android:textColorSecondary"
    android:text="@string/hello_world" />
```

在以上代码中，android:textColor 属性指定当前风格主题中某个样式属性的名称。Android 现在会使用应用于 android:textColorSecondary 样式属性的值作为 android:textColor 在这个小部件中的值。由于系统资源工具知道此环境中肯定存在某个属性资源，因此您无需显式声明类型（类型应为 ?android:attr/textColorSecondary）— 您可以将 attr 类型排除在外。

### 访问原始文件

尽管并不常见，但您的确有可能需要访问原始文件和目录。如果确有需要，则将您的文件保存在 res/ 中不起作用，因为从 res/读取资源的唯一方法是使用资源 ID。您可以改为将资源保存在 assets/ 目录中。

保存在 assets/ 目录中的文件没有资源 ID，因此您无法通过 R 类或在 XML 资源中引用它们。您可以改为采用类似普通文件系统的方式查询 assets/ 目录中的文件，并利用 AssetManager 读取原始数据。

不过，如果只需要读取原始数据（例如视频文件或音频文件）的能力，则可将文件保存在 res/raw/ 目录中，并利用 openRawResource() 读取字节流。

### 访问平台资源

Android 包含许多标准资源，例如样式、风格主题和布局。要访问这些资源，请通过 android 包名称限定您的资源引用。例如，您可以将 Android 提供的布局资源用于 ListAdapter 中的列表项：

```
setListAdapter(new ArrayAdapter<String>(this, android.R.layout.simple_list_item_1, myarray));
```

在上例中，simple_list_item_1 是平台为 ListView 中的项目定义的布局资源。您可以使用它，而不必自行创建列表项布局。