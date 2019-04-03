# 配置方法数超过 64K 的应用

随着 Android 平台的持续成长，Android 应用的大小也在增加。当您的应用及其引用的库达到特定大小时，您会遇到构建错误，指明您的应用已达到 Android 应用构建架构的极限。
[Enable multidex for apps with over 64K methods](https://developer.android.google.cn/studio/build/multidex.html)。


## 目录

* [概述](#概述)
* [规避64K限制](#规避64K限制)
* [配置您的应用进行Dalvik可执行文件分包](#配置您的应用进行Dalvik可执行文件分包)
* [Dalvik可执行文件分包支持库的局限性](#Dalvik可执行文件分包支持库的局限性)
* [声明主DEX文件中需要的类](#声明主DEX文件中需要的类)
  * [multiDexKeepProguard属性](#multiDexKeepProguard属性)
  * [multiDexKeepProguard属性](#multiDexKeepProguard属性)

## 概述

Android 应用 (APK) 文件包含 Dalvik Executable (DEX) 文件形式的可执行字节码文件，其中包含用来运行您的应用的已编译代码。
Dalvik Executable 规范将可在单个 DEX 文件内可引用的方法总数限制在 65,536，其中包括 Android 框架方法、库方法以及您自己代码中的方法。
在计算机科学领域内，术语千（简称 K）表示 1024（或 2^10）。由于 65,536 等于 64 X 1024，因此这一限制也称为“64K 引用限制”。

* Android 5.0 之前版本的 Dalvik 可执行文件分包支持

  Android 5.0（API 级别 21）之前的平台版本使用 Dalvik 运行时来执行应用代码。默认情况下，Dalvik 限制应用的每个 APK 只能使用单个 classes.dex 字节码文件。
  要想绕过这一限制，您可以使用 Dalvik 可执行文件分包支持库，它会成为您的应用主要 DEX 文件的一部分，然后管理对其他 DEX 文件及其所包含代码的访问。

* Android 5.0 及更高版本的 Dalvik 可执行文件分包支持

  Android 5.0（API 级别 21）及更高版本使用名为 ART 的运行时，后者原生支持从 APK 文件加载多个 DEX 文件。
  ART 在应用安装时执行预编译，扫描 classesN.dex 文件，并将它们编译成单个 .oat 文件，供 Android 设备执行。
  因此，如果您的 minSdkVersion 为 21 或更高值，则不需要 Dalvik 可执行文件分包支持库。


## 规避64K限制

在将您的应用配置为支持使用 64K 或更多方法引用之前，您应该采取措施减少应用代码调用的引用总数，包括由您的应用代码或包含的库定义的方法。
下列策略可帮助您避免达到 DEX 引用限制：

* 检查您的应用的直接和传递依赖项。确保您在应用中使用任何庞大依赖库所带来的好处大于为应用添加大量代码所带来的弊端。一种常见的反面模式是，仅仅为了使用几个实用方法就在应用中加入非常庞大的库。减少您的应用代码依赖项往往能够帮助您规避 dex 引用限制。
* 通过 ProGuard 移除未使用的代码。为您的版本构建启用代码压缩以运行 ProGuard。启用压缩可确保您交付的 APK 不含有未使用的代码。

使用这些技巧使您不必在应用中启用 Dalvik 可执行文件分包，同时还会减小 APK 的总体大小。


## 配置您的应用进行Dalvik可执行文件分包

将您的应用项目设置为使用 Dalvik 可执行文件分包配置需要对您的应用项目进行以下修改，具体取决于应用支持的最低 Android 版本。

如果您的 minSdkVersion 设置为 21 或更高值，您只需在模块级 build.gradle 文件中将 multiDexEnabled 设置为 true，如此处所示：

```
android {
    defaultConfig {
        ...
        minSdkVersion 21
        targetSdkVersion 28
        multiDexEnabled true
    }
    ...
}
```

但是，如果您的 minSdkVersion 设置为 20 或更低值，则您必须按如下方式使用 Dalvik 可执行文件分包支持库：

* 修改模块级 build.gradle 文件以启用 Dalvik 可执行文件分包，并将 Dalvik 可执行文件分包库添加为依赖项，如此处所示：

  ```
  android {
      defaultConfig {
          ...
          minSdkVersion 15
          targetSdkVersion 28
          multiDexEnabled true
      }
      ...
  }

  dependencies {
    compile 'com.android.support:multidex:1.0.3'
  }
  ```

* 根据是否要替换 Application 类，执行以下操作之一：

  * 如果您没有替换 Application 类，请编辑清单文件，按如下方式设置 <application> 标记中的 android:name：

    ```
    <?xml version="1.0" encoding="utf-8"?>
    <manifest xmlns:android="http://schemas.android.com/apk/res/android"
        package="com.example.myapp">
        <application
                android:name="android.support.multidex.MultiDexApplication" >
            ...
        </application>
    </manifest>
    ```

  * 如果您替换了 Application 类，请按如下方式对其进行更改以扩展 MultiDexApplication（如果可能）：

    ```
    public class MyApplication extends MultiDexApplication { ... }
    ```

  * 或者，如果您替换了 Application 类，但无法更改基本类，则可以改为替换 attachBaseContext() 方法并调用 MultiDex.install(this) 来启用 Dalvik 可执行文件分包：

    ```
    public class MyApplication extends SomeOtherApplication {
      @Override
      protected void attachBaseContext(Context base) {
         super.attachBaseContext(base);
         MultiDex.install(this);
      }
    }
    ```

构建应用后，Android 构建工具会根据需要构建主 DEX 文件 (classes.dex) 和辅助 DEX 文件（classes2.dex 和 classes3.dex 等）。
然后，构建系统会将所有 DEX 文件打包到您的 APK 中。

运行时，Dalvik 可执行文件分包 API 使用特殊的类加载器来搜索适用于您的方法的所有 DEX 文件（而不是仅在主 classes.dex 文件中搜索）。


## Dalvik可执行文件分包支持库的局限性

Dalvik 可执行文件分包支持库具有一些已知的局限性，将其纳入您的应用构建配置之中时，您应该注意这些局限性并进行针对性的测试：

* 启动期间在设备数据分区中安装 DEX 文件的过程相当复杂，如果辅助 DEX 文件较大，可能会导致应用无响应 (ANR) 错误。
  在此情况下，您应该通过 ProGuard 应用代码压缩以尽量减小 DEX 文件的大小，并移除未使用的那部分代码。
* 由于存在 Dalvik linearAlloc 错误（问题 22586），使用 Dalvik 可执行文件分包的应用可能无法在运行的平台版本早于 Android 4.0（API 级别 14）的设备上启动。
  如果您的目标 API 级别低于 14，请务必针对这些版本的平台进行测试，因为您的应用可能会在启动时或加载特定类群时出现问题。代码压缩可以减少甚至有可能消除这些潜在问题。
* 由于存在 Dalvik linearAlloc 限制（问题 78035），因此，如果使用 Dalvik 可执行文件分包配置的应用发出非常庞大的内存分配请求，则可能会在运行期间发生崩溃。
  尽管 Android 4.0（API 级别 14）提高了分配限制，但在 Android 5.0（API 级别 21）之前的 Android 版本上，应用仍有可能遭遇这一限制。


## 声明主DEX文件中需要的类

为 Dalvik 可执行文件分包构建每个 DEX 文件时，构建工具会执行复杂的决策制定来确定主要 DEX 文件中需要的类，以便应用能够成功启动。
如果启动期间需要的任何类未在主 DEX 文件中提供，那么您的应用将崩溃并出现错误 java.lang.NoClassDefFoundError。

该情况不应出现在直接从应用代码访问的代码上，因为构建工具能识别这些代码路径，但可能在代码路径可见性较低（如使用的库具有复杂的依赖项）时出现。
例如，如果代码使用自检机制或从原生代码调用 Java 方法，那么这些类可能不会被识别为主 DEX 文件中的必需项。

因此，如果您收到 java.lang.NoClassDefFoundError，则必须使用构建类型中的 multiDexKeepFile 或 multiDexKeepProguard 属性声明它们，
以手动将这些其他类指定为主 DEX 文件中的必需项。如果类在 multiDexKeepFile 或 multiDexKeepProguard 文件中匹配，则该类会添加至主 DEX 文件。

### multiDexKeepFile属性

您在 multiDexKeepFile 中指定的文件应该每行包含一个类，并且采用 com/example/MyClass.class 的格式。
例如，您可以创建一个名为 multidex-config.txt 的文件，如下所示：

```
com/example/MyClass.class
com/example/MyOtherClass.class
```

然后，您可以按以下方式针对构建类型声明该文件：

```
android {
    buildTypes {
        release {
            multiDexKeepFile file 'multidex-config.txt'
            ...
        }
    }
}
```

请记住，Gradle 会读取相对于 build.gradle 文件的路径，因此如果 multidex-config.txt 与 build.gradle 文件在同一目录中，以上示例将有效。

### multiDexKeepProguard属性

multiDexKeepProguard 文件使用与 Proguard 相同的格式，并且支持整个 Proguard 语法。

您在 multiDexKeepProguard 中指定的文件应该在任何有效的 ProGuard 语法中包含 -keep 选项。
例如，-keep com.example.MyClass.class。您可以创建一个名为 multidex-config.pro 的文件，如下所示：

```
-keep class com.example.MyClass
-keep class com.example.MyClassToo
```

如果您想要指定包中的所有类，文件将如下所示：

```
-keep class com.example.** { *; } // All classes in the com.example package
```

然后，您可以按以下方式针对构建类型声明该文件：

```
android {
    buildTypes {
        release {
            multiDexKeepProguard 'multidex-config.pro'
            ...
        }
    }
}
```