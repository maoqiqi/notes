# GitHub项目徽章的添加和设置

GitHub项目的README.md文件中中可以添加徽章(Badge)对项目进行标记和说明,这些好看的小图标不仅简洁美观,而且还包含了清晰易读的信息。

## 目录

* [徽标简介](#徽标简介)
* [常用徽标添加](#常用徽标添加)
  * [持续集成状态](#持续集成状态)
  * [项目下载量](#项目下载量)
  * [项目版本信息](#项目版本信息)
* [自定义徽标](#自定义徽标)
  * [标题／内容／颜色／链接](#标题／内容／颜色／链接)
  * [附加参数](#附加参数)


## 徽标简介

徽标主要由图片和对应的链接(当然,你可以不填)组成,徽标图片的话一般由左半部分的名称和右半部分的值组成。

GitHub徽标的官方网站是 http://shields.io ,GitHub地址是 https://github.com/badges/shields ,
我们可以在官网预览绝大部分的徽标样式,然后选择自己喜欢的(当然首先需要适用于自己的目标项目)徽标,添加到自己的项目文档中去。

徽标并不是添加的越多越好,合理地选择适合项目的徽标做具有针对性地添加才是理性的做法。


## 常用徽标添加

常用的徽标主要有持续集成状态、项目下载量、项目版本信息、代码测试覆盖率、项目支持平台、项目语言、代码分析等,下面我们就来依次添加这些徽标！

### 持续集成状态

持续集成的话推荐 [Travis CI][1] ,针对开源项目免费,所以你的私有项目无法享受到免费的持续构建服务,不过我们的目的貌似就是给开源项目添加徽标。

同类型的产品还有 [Circle CI][2] ,不过目前跑OSX项目需要额外付费,免费版提供一个Linux项目队列,作为非付费用户在这里不多做评价,大佬们可以自己试下。
可以参考[使用Travis CI进行持续集成](travis_ci.md) 。

其他还有诸如 [Jenkins][3] 和 [CodeShip][4] 等,大家可以在 http://shields.io 的Build这一栏自行翻阅。

徽标图片地址是这个样子的：

```
http://img.shields.io/travis/{GitHub 用户名}/{项目名称}.svg
```

以本项目为例:

```
[![Build Status](https://travis-ci.org/maoqiqi/blog.svg?branch=master)](https://travis-ci.org/maoqiqi/blog)
```

[![Build Status](https://travis-ci.org/maoqiqi/test.svg?branch=master)](https://travis-ci.org/maoqiqi/test)

这里需要指出的是,开源项目的Travis CI也是公开的,包括日志和历史记录在内,都是针对所有人可见的,所以小伙伴们一定不要把密码、私钥等重要信息暴露了。

### 项目下载量

  项目被下载的次数,这个的话各个平台的统计都是独立的。

  ```
  [![Github All Releases](https://img.shields.io/github/downloads/maoqiqi/blog/total.svg)](https://github.com/maoqiqi/blog)
  ```

  [![Github All Releases](https://img.shields.io/github/downloads/maoqiqi/blog/total.svg)](https://github.com/maoqiqi/blog)

### 项目版本信息

  ```
  [![GitHub tag](https://img.shields.io/github/tag/maoqiqi/blog.svg)](https://github.com/maoqiqi/blog)
  ```

  [![GitHub tag](https://img.shields.io/github/tag/maoqiqi/blog.svg)](https://github.com/maoqiqi/blog)

### 代码测试覆盖率

  代码测试覆盖率的话推荐[Code cov][5]。

  [![Code cov](https://img.shields.io/codecov/c/github/maoqiqi/blog.svg)](https://github.com/maoqiqi/blog)

### 项目支持平台
### 项目语言
### 代码分析

[CodeBeat][6]可以计算全局项目评分、GPA、和不同命名空间的等级来帮助您量化技术债务和发现重构机会,你唯一需要做的
就是连接你的Github库,获得反馈就好了。


## 自定义徽标

开源协议类型,如果你的发布工具不提供开源协议类型的徽标的话,可以用自定义徽标的方式实现。

### 标题／内容／颜色／链接

如果以上这些徽标没有满足你的需求,我们还可以定制自己的个性化徽标, http://shields.io 提供了添加自定义徽标的功能,
通过修改如下URL即可获取自定义徽标图片：

```
https://img.shields.io/badge/{徽标标题}-{徽标内容}-{徽标颜色}.svg
```

{徽标标题}：徽标左半部分的文本（短线：--,下划线：__,空格： 或_）

{徽标内容}：徽标右半部分的文本,同上

{徽标颜色}：徽标右半部分背景颜色,可以是red、green、blue等颜色英文单词,也可以直接写十六进制的颜色值,如ff69b4,示例如下：

![p](https://img.shields.io/badge/color-brightgreen-brightgreen.svg)
![p](https://img.shields.io/badge/color-green-green.svg)
![p](https://img.shields.io/badge/color-yellowgreen-yellowgreen.svg)
![p](https://img.shields.io/badge/color-yellow-yellow.svg)
![p](https://img.shields.io/badge/color-orange-orange.svg)
![p](https://img.shields.io/badge/color-red-red.svg)
![p](https://img.shields.io/badge/color-lightgrey-lightgrey.svg)
![p](https://img.shields.io/badge/color-blue-blue.svg)
![p](https://img.shields.io/badge/color-ff69b4-ff69b4.svg)

将其中的{徽标标题}、{徽标内容}、{徽标颜色}分别替换为需要的内容即可,例如我的微博徽标图片地址如下：

```
https://img.shields.io/badge/weibo-@FengqiMao-red.svg
```

再结合我的微博地址`https://weibo.com/u/3738882105`后完整徽标代码和效果如下:

```
[![weibo](https://img.shields.io/badge/weibo-@FengqiMao-red.svg)](https://weibo.com/u/3738882105)
```

[![weibo](https://img.shields.io/badge/weibo-@FengqiMao-red.svg)](https://weibo.com/u/3738882105)


### 附加参数

可以在徽标图片URL后面带上一些参数来控制徽标的样式,这一部分是可选的,不想折腾的话默认的样式就挺好了,可以不看这里的。

使用方法就是在徽标图片URL后面跟上?{参数名}={参数值}

多个参数联用的话就是?{参数名1}={参数值1}&{参数名2}={参数值2}...

* style

  style控制徽标的主体样式,有四种,不设置的话默认是flat的,示例代码和效果如下：

  1.plastic(立体效果)

  ```
  ![](https://img.shields.io/badge/weibo-@FengqiMao-red.svg?style=plastic)
  ```

  ![](https://img.shields.io/badge/weibo-@FengqiMao-red.svg?style=plastic)

  2.flat(扁平化)

  ```
  ![](https://img.shields.io/badge/weibo-@FengqiMao-red.svg?style=flat)
  ```

  ![](https://img.shields.io/badge/weibo-@FengqiMao-red.svg?style=flat)

  3.flat-square(扁平化+去除圆角)

  ```
  ![](https://img.shields.io/badge/weibo-@FengqiMao-red.svg?style=flat-square)
  ```

  ![](https://img.shields.io/badge/weibo-@FengqiMao-red.svg?style=flat-square)

  4.social(社交样式)

  ```
  ![](https://img.shields.io/badge/weibo-@FengqiMao-red.svg?style=social)
  ```

  ![](https://img.shields.io/badge/weibo-@FengqiMao-red.svg?style=social)

* label

  该参数可以用来强制覆盖原有的徽标标题文字,效果如下,原有的weibo字样已经被覆盖了：

  ```
  ![](https://img.shields.io/badge/weibo-@FengqiMao-red.svg?label=微博)
  ```

  ![](https://img.shields.io/badge/weibo-@FengqiMao-red.svg?label=微博)

* logo

  该参数可以用来为徽标添加logo,logo图片会出现在左半部分的徽标标题左边,logo图片高度必须≥14px,logo图片需要先转为base64
  编码然后直接插入到URL中(可以用[http://b64.io/][7]将图片转为base64编码的字符串),格式如下:

  ```
  ?logo={base64 编码后的图片数据}
  ```

  示例代码和效果如下:

  ```
  ![](https://img.shields.io/badge/weibo-@FengqiMao-red.svg?logo=data:image/png;base64,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)
  ```
  ![](https://img.shields.io/badge/weibo-@FengqiMao-red.svg?logo=data:image/png;base64,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)

* logoWidth

  该参数可以设置在上一个参数logo中添加的图标的宽度,设为0的话即为忽略该参数,示例代码和效果如下：

  ```
  ![](https://img.shields.io/badge/weibo-@FengqiMao-red.svg?logoWidth=48&logo=data:image/png;base64,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)
  ```

  ![](https://img.shields.io/badge/weibo-@FengqiMao-red.svg?logoWidth=48&logo=data:image/png;base64,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)

* link

  据说该参数是用来设置点击后跳转的URL(嗯,俗称超链接),官方描述如下：

  Specify what clicking on the left/right of a badge should do (esp. for social badge style)

  不过试了一下好像没啥效果

* colorA

  该参数用来控制徽标左半部分的背景色,只能用十六进制的颜色作为参数,不能直接写red、green、blue之类的,这里将左半部分的背景
  色改为0xabcdef,代码和效果如下：

  ```
  ![](https://img.shields.io/badge/weibo-@FengqiMao-red.svg?colorA=abcdef)
  ```

  ![](https://img.shields.io/badge/weibo-@FengqiMao-red.svg?colorA=abcdef)

* colorB

  该参数用来控制徽标右半部分的背景色,同上,只能用十六进制的颜色作为参数哦,不能直接写red、green、blue之类的,这里将右半部分
  的背景色改为0xabcdef,代码和效果如下：

  ```
  ![](https://img.shields.io/badge/weibo-@FengqiMao-red.svg?colorA=abcdef)
  ```

  ![](https://img.shields.io/badge/weibo-@FengqiMao-red.svg?colorB=abcdef)

* maxAge

  该参数用来设置HTTP缓存时间,以秒为单位。

**注意**

这里需要注意的是,如果你是引用的第三方svg然后添加自己的样式,如果该样式之前已经被第三方添加过,是不一定会覆盖第三方的设置的,
也就是说自己设置的属性不一定会生效。


[1]:https://travis-ci.org/
[2]:https://circleci.com/
[3]:https://jenkins.io/
[4]:https://codeship.com/
[5]:https://codecov.io/
[6]:https://codebeat.co/
[7]:http://b64.io/