# Android 快速依赖注入框架 dagger

> 作者：March    
> 链接：[Android 快速依赖注入框架 dagger](https://github.com/maoqiqi/blog/blob/master/pages/android_dagger.md)    
> 博客：http://blog.csdn.net/u011810138    
> 邮箱：fengqi.mao.march@gmail.com    
> 著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。    

A fast dependency injector for Android and Java. http://google.github.io/dagger.

## dagger2 介绍

### 为什么要用 dagger2 ？

解耦(DI的特性)，易于测试（DI的特性），高效（不使用反射，google官方说名比Dagger快13%），易混淆（apt方式生成代码，混淆后依然正常使用）。

## Android Studio 配置 dagger2

### 

@Inject

@Module: 为一些三方类库提供注入的对象，常配合 @Provides 使用。

@Component: Component 是一个注入器，就像注射器一样，Component 会把目标类依赖的实例注入到目标类中，来初始化目标类中的依赖。

@Subcomponent: 从名字可以看出，这也是一个注入器，只不过被 @Subcomponent 注解的是被 @Component 注解的下一层级，这就像 Subcomponent 继承于 Component 一样。

@Singleton

@Scope

@Named(@)


## dagger.android

### 核心类

AndroidInjection：Dagger.Android 注入的核心类，主要封装了一些静态方法用来为四大组件和 Fragment 进行注入。

AndroidInjector<T>:注入Android库的类型的接口, T为Android库的基本类型T,比如Activity、Fragment、BroadcastReceive等；

AndroidInjector.Factory<T>：AndroidInjector<T>的工厂类接口

DispatchingAndroidInjector<T>:其为AndroidInjector<T>接口的实现类，将Android核心库的的基本类型T的实例注入Dagger，该操作是由Android核心库的类的实例本身执行，而不是Dagger。


AndroidInjectionModule: 主要提供 Dagger.Android 组件包，它应该被包含在注入 Application 的 Component 注入器的 modules 中。


### 注入实例

由于DispatchingAndroidInjector在运行时由类查找相应的AndroidInjector.Factory，
那么，在基类中，实现HasActivityInjector/HasFragmentInjector接口，
在相应的声明周期(onCreate()或者onAttach())内调用AndroidInjection.inject()方法，注入相应的实例。
所有每个子类都需要做的是绑定相应的@Subcomponent，从而没有必要在每个实例类中调用AndroidInjection.inject()方法。

在dagger.android库中，Dagger提供了一些基本类型，比如DaggerActivity和DaggerFragment。
对于Application,还提供了DaggerApplication，继承其后，
只需重写applicationInjector()方法来返回AndroidInjector&lxtXxApplication>.

Dagger提供的基本类型：

DaggerActivity
DaggerFragment
DaggerService
DaggerIntentService
DaggerBroadcastReceiver
DaggerContentProvider

### 注入Activity实例

### 注入Fragment实例
