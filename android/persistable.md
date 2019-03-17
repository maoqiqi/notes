* 关于onCreate(@Nullable Bundle savedInstanceState, @Nullable PersistableBundle persistentState)

Android API 21为Activity增加了一个新的`persistableMode`属性，只要将其设置成`persistAcrossReboots`，
activity就有了持久化的能力，另外需要配合一个新的bundle才行，那就是PersistableBundle。

https://blog.csdn.net/li_peipei/article/details/79760635