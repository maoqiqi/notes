# 监听整个App所有Activity和Fragment
  
> 本文链接：[监听整个App所有Activity和Fragment](https://github.com/maoqiqi/blog/blob/master/pages/android/android_global_monitor.md)    
> 著作权归作者所有.商业转载请联系作者获得授权,非商业转载请注明出处.    

通过非继承ActivityFragment来实现以前需要封装进 BaseActivity BaseFragment,通过非继承来实现的一些公共逻辑,以及监听整个
App所有Activity以及Fragment的生命周期(包括三方库),并可向其生命周期内插入代码.

## 为什么提倡少封装BaseActivity,少用继承

BaseActivity封装多了,除了不好管理外,还有最重要的一点就是,Java只能单继承.当如果你的Activity需要使用到某个三方库,那个三
方库必须让你继承于它的Activity但是你又需要你自己封装的 BaseActivity 的某些特性,这时你怎么办?你不能改三方库的Activity
所以你只有改你的BaseActivity让它去继承三方库的Activity,但是当改了BaseActivity后,发现有很多继承于BaseActivity的
Activity并不需要这个三方库,却被迫继承了它的Activity,这时你又要重新封装一个 BaseActivity.

当这种情况发生的多了,你的 BaseActivity 也越来越多,这样就恶性循环了.所以我不仅提倡App开发者少封装 BaseActivity 少用继
承,也提倡 三方库 开发者少封装 BaseActivity 少用继承,为什么呢?因为当App开发者的Activity需要使用到两个三方库,两个三方库
都需要继承它的 Activity,这时你让App开发者怎么办?

**思考**

我们为什么一定要封装 BaseActivity 通过继承来实现一些Activity的公共逻辑,而不能将公共逻辑封装到其他类里面?

答案很简单,因为我们必须使用到Activity的一些生命周期,在对应的生命周期里执行对应的逻辑,这个就是我们不能通过封装其他类来实现
的原因,找到了问题关键,那我们就从生命周期上下手来解决问题.

## ActivityLifecycleCallbacks

ActivityLifecycleCallbacks 是 Application 中声明的一个内部接口,我们来看看它的结构

```java
public interface ActivityLifecycleCallbacks {
    void onActivityCreated(Activity activity, Bundle savedInstanceState);
    void onActivityStarted(Activity activity);
    void onActivityResumed(Activity activity);
    void onActivityPaused(Activity activity);
    void onActivityStopped(Activity activity);
    void onActivitySaveInstanceState(Activity activity, Bundle outState);
    void onActivityDestroyed(Activity activity);
}
```

这个接口有什么用呢?

Application 提供有一个 registerActivityLifecycleCallbacks() 的方法,需要传入的参数就是这个 ActivityLifecycle
Callbacks 接口,作用和你猜的没错,就是在你调用这个方法传入这个接口实现类后,系统会在每个Activity执行完对应的生命周期后都调
用这个实现类中对应的方法,请记住是每个!

```java
public class MyApplication extends Application {

    private final String TAG = "MyApplication";

    @Override
    public void onCreate() {
        super.onCreate();
        registerActivityLifecycleCallbacks(new ActivityLifecycleCallbacks() {
            @Override
            public void onActivityCreated(Activity activity, Bundle savedInstanceState) {
                Log.d(TAG, "onActivityCreated" + activity.getClass().getName());
            }

            @Override
            public void onActivityStarted(Activity activity) {
                Log.d(TAG, "onActivityStarted" + activity.getClass().getName());
            }

            @Override
            public void onActivityResumed(Activity activity) {
                Log.d(TAG, "onActivityResumed" + activity.getClass().getName());
            }

            @Override
            public void onActivityPaused(Activity activity) {
                Log.d(TAG, "onActivityPaused" + activity.getClass().getName());
            }

            @Override
            public void onActivityStopped(Activity activity) {
                Log.d(TAG, "onActivityStopped" + activity.getClass().getName());
            }

            @Override
            public void onActivitySaveInstanceState(Activity activity, Bundle outState) {
                Log.d(TAG, "onActivitySaveInstanceState" + activity.getClass().getName());
            }

            @Override
            public void onActivityDestroyed(Activity activity) {
                Log.d(TAG, "onActivityDestroyed" + activity.getClass().getName());
            }
        });
    }
}
```

## 总结

值得注意的是 ActivityLifecycleCallbacks 可以注册多个,可以针对不同情况添加各种 ActivityLifecycleCallbacks,按照需
要进行组合从而达到不同的需求,和Okhttp的Interceptor类似.

现在整个App的所有Activity和Fragment只要生命周期调用都会被拦截,所以就可以动态的向所有Activity和Fragment的对应生命周期
插入任意代码.

## About Me

* **邮箱**：fengqi.mao.march@gmail.com
* **微博**：https://weibo.com/u/3738882105
* **简书**：https://www.jianshu.com/u/02f2491c607d
* **掘金**：https://juejin.im/user/5b484473e51d45199940e2ae
* **思否**：https://segmentfault.com/u/maoqiqi
* **CSDN**：http://blog.csdn.net/u011810138
* **GitHub**：https://github.com/maoqiqi
