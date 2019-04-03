# Markdown 语法简单教程

Markdown 是一种方便记忆、书写的纯文本标记语言，用户可以使用这些标记符号以最小的输入代价生成极富表现力的文档。
[官方文档](https://daringfireball.net/projects/markdown/syntax)。


## 目录

* [目录](#目录)
* [区块元素](#区块元素)
    * [标题](#标题)
    * [换行](#换行)
    * [引用](#引用)
    * [列表](#列表)
    * [代码块](#代码块)
    * [分割线](#分割线)
* [区段元素](#区段元素)
    * [链接](#链接)
    * [锚点](#锚点)
    * [强调](#强调)
    * [代码](#代码)
    * [diff语法](#diff语法)
    * [图片](#图片)
* [其它](#其它)
    * [自动链接](#自动链接)
    * [反斜杠](#反斜杠)
* [免费编辑器](#免费编辑器)
    * [Windows平台](#windows平台)
    * [Linux平台](#linux平台)
    * [Mac平台](#mac平台)
    * [在线编辑器](#在线编辑器)
    * [浏览器插件](#浏览器插件)


## 区块元素

### 标题

在Markdown中，你只需要在文本前面加上 # 即可，同理、你还可以增加二级标题、三级标题、四级标题、五级标题和六级标题，总共六级，只需要增加 # 即可，标题字号相应降低。例如：

```
# 一级标题
## 二级标题
### 三级标题
#### 四级标题
##### 五级标题
###### 六级标题
```

### 换行

一个 Markdown 段落是由一个或多个连续的文本行组成，它的前后要有一个以上的空行（空行的定义是显示上看起来像是空的，便会被视为空行。比方说，若某一行只包含空格和制表符，则该行也会被视为空行）。

直接回车不能换行，  
可以在上一行文本后面补两个空格，  
这样下一行的文本就换行了。

### 引用

Markdown 标记区块引用是使用 > 的引用方式。

```
> This is a blockquote with two paragraphs. Lorem ipsum dolor sit amet,
> consectetuer adipiscing elit. Aliquam hendrerit mi posuere lectus.
> Vestibulum enim wisi, viverra nec, fringilla in, laoreet vitae, risus.
>
> Donec sit amet nisl. Aliquam semper ipsum sit amet velit. Suspendisse
> id sem consectetuer libero luctus adipiscing.
```

### 列表

Markdown 支持有序列表和无序列表。

无序列表使用星号、加号或是减号作为列表标记：

```
* Red
* Green
* Blue
```

有序列表则使用数字接着一个英文句点：

```
1. Bird
2. McHale
3. Parish
```

很重要的一点是，你在列表标记上使用的数字并不会影响输出的 HTML 结果。

列表项目标记通常是放在最左边，但是其实也可以缩进，最多 3 个空格，项目标记后面则一定要接着至少一个空格或制表符。

要让列表看起来更漂亮，你可以把内容用固定的缩进整理好：

```
*   Lorem ipsum dolor sit amet, consectetuer adipiscing elit.
    Aliquam hendrerit mi posuere lectus. Vestibulum enim wisi,
    viverra nec, fringilla in, laoreet vitae, risus.
*   Donec sit amet nisl. Aliquam semper ipsum sit amet velit.
```

列表项目可以包含多个段落，每个项目下的段落都必须缩进 4 个空格或是 1 个制表符：

```
1.  This is a list item with two paragraphs. Lorem ipsum dolor
    sit amet, consectetuer adipiscing elit. Aliquam hendrerit
    mi posuere lectus.

    Vestibulum enim wisi, viverra nec, fringilla in, laoreet
    vitae, risus. Donec sit amet nisl. Aliquam semper ipsum
    sit amet velit.

2.  Suspendisse id sem consectetuer libero luctus adipiscing.
```

如果要在列表项目内放进引用，那 > 就需要缩进：

```
*   A list item with a blockquote:

    > This is a blockquote
    > inside a list item.
```

如果要放代码区块的话，该区块就需要缩进两次，也就是 8 个空格或是 2 个制表符：

```
*   一列表项包含一个列表区块：

        <代码写在这>
```

当然，项目列表很可能会不小心产生，像是下面这样的写法：

```
1986. What a great season.
```

换句话说，也就是在行首出现数字-句点-空白，要避免这样的状况，你可以在句点前面加上反斜杠。

```
1986\. What a great season.
```

待办列表: 表示列表是否勾选状态（注意：[ ] 前后都要有空格）。

* [ ] 未完成功能...
* [ ] 未完成功能...
* [x] 已完成功能...
* [x] 已完成功能...
* [x] 已完成功能...

> 在GitHub的**issue**中使用该语法是可以实时点击复选框来勾选或解除勾选的，而无需修改issue原文。

### 表格

在 Markdown 中，可以制作表格，例如：

|表头1|表头2|表头3|
|:-----|:-----:|-----:|
|第一列,左对齐|第二列,居中|第三列,右对齐|
|左对齐|居中|右对齐|

### 代码块

Markdown 中建立代码区块很简单，只要简单地缩进 4 个空格或是 1 个制表符就可以，例如：

```
这是一个普通段落：

    这是一个代码区块。
```

一个代码区块会一直持续到没有缩进的那一行（或是文件结尾）。

在代码区块里面， & 、 < 和 > 会自动转成 HTML 实体，这样的方式让你非常容易使用 Markdown 插入范例用的 HTML 原始码，只需要复制贴上，再加上缩进就可以了，剩下的 Markdown 都会帮你处理。

### 分割线

你可以在一行中用三个以上的星号、减号、底线来建立一个分隔线，行内不能有其他东西。你也可以在星号或是减号中间插入空格。下面每种写法都可以建立分隔线：

```
* * *
***
*****
- - -
---------------------------------------
```


## 区段元素

### 链接

Markdown 支持两种形式的链接语法：行内式和参考式两种形式。

行内式的链接，只要在方块括号后面紧接着圆括号并插入网址链接即可，如果你还想要加上链接的 title 文字，只要在网址后面，用双引号把 title 文字包起来即可，例如：

```
This is [an example](http://example.com/ "Title") inline link.
[This link](http://example.net/) has no title attribute.
```

参考式的链接是在链接文字的括号后面再接上另一个方括号，而在第二个方括号里面要填入用以辨识链接的标记：

```
This is [an example][id] reference-style link.
```

你也可以选择性地在两个方括号中间加上一个空格：

```
This is [an example] [id] reference-style link.
```

接着，在文件的任意处，你可以把这个标记的链接内容定义出来：

```
[id]: http://example.com/  "Optional Title Here"
```

链接内容定义的形式为：

* 方括号（前面可以选择性地加上至多三个空格来缩进），里面输入链接文字
* 接着一个冒号
* 接着一个以上的空格或制表符
* 接着链接的网址
* 选择性地接着 title 内容，可以用单引号、双引号或是括弧包着

下面这三种链接的定义都是相同：

```
[id]: http://example.com/  "Optional Title Here"
[id]: http://example.com/  'Optional Title Here'
[id]: http://example.com/  (Optional Title Here)
```

**注**：有一个已知的问题是 Markdown.pl 1.0.1 会忽略单引号包起来的链接 title。

链接网址也可以用尖括号包起来：

```
[id]: <http://example.com/>  "Optional Title Here"
```

网址定义只有在产生链接的时候用到，并不会直接出现在文件之中。

链接辨别标签可以有字母、数字、空白和标点符号，但是并不区分大小写，因此下面两个链接是一样的：

```
[link text][a]
[link text][A]
```

隐式链接标记功能让你可以省略指定链接标记，这种情形下，链接标记会视为等同于链接文字，要用隐式链接标记只要在链接文字后面加上一个空的方括号，如果你要让 "Google" 链接到 google.com，你可以简化成：

```
[Google][]
```

然后定义链接内容：

```
[Google]: http://google.com/
```

由于链接文字可能包含空白，所以这种简化型的标记内也许包含多个单词：

```
Visit [Daring Fireball][] for more information.
```

然后接着定义链接：

```
[Daring Fireball]: http://daringfireball.net/
```

下面是一个参考式链接的范例：

```
I get 10 times more traffic from [Google] [1] than from
[Yahoo] [2] or [MSN] [3].

  [1]: http://google.com/        "Google"
  [2]: http://search.yahoo.com/  "Yahoo Search"
  [3]: http://search.msn.com/    "MSN Search"
```

### 锚点

其实呢，每一个标题都是一个锚点，和HTML的锚点（#）类似。

```
[目录](#目录)
```

**注**：标题中的英文字母都被转化为小写字母了。

### 强调

|语法|效果|
|----|-----|
|`*斜体1*`|*斜体1*|
|`_斜体2_`| _斜体2_|
|`**粗体1**`|**粗体1**|
|`__粗体2__`|__粗体2__|
|`这是一个 ~~删除线~~`|这是一个 ~~删除线~~|
|`***斜粗体1***`|***斜粗体1***|
|`___斜粗体2___`|___斜粗体2___|
|`***~~斜粗体删除线1~~***`|***~~斜粗体删除线1~~***|
|`~~***斜粗体删除线2***~~`|~~***斜粗体删除线2***~~|

**注**：斜体、粗体、删除线可混合使用。如果你的 \* 、 _ 和 ~ 两边都有空白的话，它们就只会被当成普通的符号。

如果要在文字前后直接插入普通的星号，你可以用反斜线：

```
\*this text is surrounded by literal asterisks\*
```

### 代码

如果要标记一小段行内代码，你可以用反引号把它包起来（\`），例如：

```
Use the `printf()` function.
```
如果要在代码区段内插入反引号，你可以用多个反引号来开启和结束代码区段：

```
``There is a literal backtick (`) here.``
```

代码区段的起始和结束端都可以放入一个空白，起始端后面一个，结束端前面一个，这样你就可以在区段的一开始就插入反引号：

```
A single backtick in a code span: `` ` ``
A backtick-delimited string in a code span: `` `foo` ``
```

在代码区段内， & 、 < 和 > 都会被自动地转成 HTML 实体，这使得插入 HTML 原始码变得很容易，例如：

```
Please don't use any `<blink>` tags.
```

### diff语法

版本控制的系统中都少不了diff的功能，即展示一个文件内容的增加与删除。使用绿色表示新增，红色表示删除。

其语法与代码高亮类似，只是在三个反引号后面写diff， 并且其内容中，以 + 开头表示新增，- 开头表示删除。

```diff
+ 鸟宿池边树，僧敲月下门
- 鸟宿池边树，僧推月下门
```

### 图片

Markdown 使用一种和链接很相似的语法来标记图片，同样也允许两种样式：行内式和参考式。

行内式的图片语法看起来像是：

```
![Alt text](/path/to/img.jpg)
![Alt text](/path/to/img.jpg "Optional title")
```

详细叙述如下：

* 一个惊叹号 !
* 接着一个方括号，里面放上图片的替代文字
* 接着一个普通括号，里面放上图片的网址，最后还可以用引号包住并加上选择性的 'title' 文字。

参考式的图片语法则长得像这样：

```
![Alt text][id]
```

\[id\]是图片参考的名称，图片参考的定义方式则和链接参考一样：

```
[id]: url/to/image  "Optional title attribute"
```

到目前为止， Markdown 还没有办法指定图片的宽高，如果你需要的话，你可以使用普通的 <img> 标签。


## 其它

### 自动链接

Markdown 支持以比较简短的自动链接形式来处理网址和电子邮件信箱，只要是用尖括号包起来， Markdown 就会自动把它转成链接。一般网址的链接文字就和链接地址一样，例如：

```
<http://example.com/>
```

Markdown 会转为：

```
<a href="http://example.com/">http://example.com/</a>
```

### 反斜杠

Markdown 可以利用反斜杠来插入一些在语法中有其它意义的符号，例如：如果你想要用星号加在文字旁边的方式来做出强调效果（但不用 \<em\> 标签），你可以在星号的前面加上反斜杠：

```
\*literal asterisks\*
```

Markdown 支持以下这些符号前面加上反斜杠来帮助插入普通的符号：

```
\  反斜线
`  反引号
*  星号
_  底线
{} 花括号
[] 方括号
() 括弧
#  井字号
+  加号
-  减号
.  英文句点
!  惊叹号
```

## 免费编辑器

### Windows平台

* [MarkdownPad](http://markdownpad.com/)
* [MarkPad](http://code52.org/DownmarkerWPF/)

### Linux平台

* [ReText](http://sourceforge.net/p/retext/home/ReText/)

### Mac平台

* [Mou](http://25.io/mou/)

### 在线编辑器

* [Markable.in](https://markable.in/)
* [Dillinger.io](http://dillinger.io/)
* [Stackdit](https://stackedit.io/)

### 浏览器插件

* [MaDe (Chrome)](https://chrome.google.com/webstore/detail/oknndfeeopgpibecfjljjfanledpbkog)