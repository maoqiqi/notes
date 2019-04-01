# 解决 Download https://jcenter.bintray.com 卡住问题

解决 Android Studio 打开项目 Gradle Download https://jcenter.bintray.com 下载慢，超时问题。


## 方法一：修改https为http协议下载

修改项目根目录下build.gradle文件：

```
...
buildscript {
    repositories {
        google()
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.2.1'
    }
}

allprojects {
    repositories {
        google()
        jcenter()
    }
}
...
```

为

```
...
buildscript {
    repositories {
        google()
        jcenter() { url 'http://jcenter.bintray.com/' }
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.2.1'
    }
}

allprojects {
    repositories {
        google()
        jcenter() { url 'http://jcenter.bintray.com/' }
    }
}
...
```

亲测，此方法成功率很低。


## 方法二：切换到国内的Maven镜像仓库

修改项目根目录下build.gradle文件，将 jcenter() 或者 mavenCentral() 替换掉即可。可以用国内的仓库代替：

```
...
buildscript {
    repositories {
        google()
        maven { url 'http://maven.aliyun.com/nexus/content/groups/public/' }
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.2.1'
    }
}

allprojects {
    repositories {
        google()
        maven { url 'http://maven.aliyun.com/nexus/content/groups/public/' }
    }
}
...
```

## 保存关闭文件后，然后重新打开项目。