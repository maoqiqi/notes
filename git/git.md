# Git 简单教程

> 本文链接：[GitHub项目徽章的添加和设置](https://github.com/maoqiqi/blog/blob/master/pages/git/git.md)    
> 著作权归作者所有.商业转载请联系作者获得授权,非商业转载请注明出处.    


## 安装

### 在 Linux 上安装

### 在 Mac 上安装

### 在 Windows 上安装

## 初次运行 Git 前的配置

一般在新的系统上，我们都需要先配置下自己的 Git 工作环境。配置工作只需一次，以后升级时还会沿用现在的配置。
当然，如果需要，你随时可以用相同的命令修改已有的配置。

Git 提供了一个叫做 git config 的工具，专门用来配置或读取相应的工作环境变量。
而正是由这些环境变量，决定了 Git 在各个环节的具体工作方式和行为。这些变量可以存放在以下三个不同的地方：

- /etc/gitconfig 文件：系统中对所有用户都普遍适用的配置。若使用 git config 时用--system 选项，读写的就是这个文件。

- ~/.gitconfig 文件：用户目录下的配置文件只适用于该用户。若使用 git config 时用--global 选项，读写的就是这个文件。

- 当前项目的 git 目录中的配置文件（也就是工作目录中的 .git/config 文件）：这里的配置仅仅针对当前项目有效。
每一个级别的配置都会覆盖上层的相同配置，所以.git/config 里的配置会覆盖/etc/gitconfig 中的同名变量。

在 Windows 系统上，Git 会找寻用户主目录下的 .gitconfig 文件。
主目录即 $HOME 变量指定的目录，一般都是C:\Documents and Settings\$USER。
此外，Git 还会尝试找寻/etc/gitconfig 文件，只不过看当初 Git 装在什么目录，就以此作为根目录来定位。

### 用户信息

```
git config --global user.name maoqiqi
git config --global user.email fengqi.mao.march@gmail.com
```

如果用了 --global 选项，那么更改的配置文件就是位于你用户主目录下的那个，以后你所有的项目都会默认使用这里配置的用户信息。
如果要在某个特定的项目中使用其他名字或者电邮，只要去掉--global 选项重新配置即可，新的设定保存在当前项目的.git/config 文件里。

### 查看配置信息

要检查已有的配置信息，可以使用`git config --list`命令


## 基本使用

### 检出仓库

执行如下命令以创建一个本地仓库的克隆版本：

```
git clone /path/to/repository
```

如果是远端服务器上的仓库，你的命令会是这个样子：
```
git clone https://github.com/maoqiqi/GitTest.git
```

使用Git下载指定分支命令

```
git clone -b feature_1 https://github.com/maoqiqi/GitTest.git
```

### 创建新仓库

创建新文件夹，打开，然后执行

```
git init
```

### 添加与提交

你可以计划改动（把它们添加到缓存区），使用如下命令：

```
git add <filename>
git add *
```

这是 git 基本工作流程的第一步；使用如下命令以实际提交改动：

```
git commit -m "代码提交信息"
```

现在，你的改动已经提交到了 HEAD，但是还没到你的远端仓库。

### 推送改动

你的改动现在已经在本地仓库的 HEAD 中了。执行如下命令以将这些改动提交到远端仓库：

```
git push origin master
```

可以把 master 换成你想要推送的任何分支。

如果你还没有克隆现有仓库，并欲将你的仓库连接到某个远程服务器，你可以使用如下命令添加：

```
git remote add origin <server>
```

如此你就能够将你的改动推送到所添加的服务器上去了。

### 一个简单的完整流程

```
mkdir 文件夹名
cd 文件夹名
git init
git remote add origin https://github.com/maoqiqi/GitTest.git
git add .
git commit -m "Initial commit"
git push -u origin master
```



## 分支操作

1.查看分支

列出所有本地分支：

> git branch

列出所有远程分支：

> git branch -r

列出所有本地分支和远程分支：

> git branch -a

2.新建一个分支,但依然停留在当前分支：

> git branch [branch-name]

例如：创建名称为dev的分支：

> git branch dev

3.切换到指定分支

> git checkout [branch-name]

例如：切换到dev分支上：

> git checkout dev

4.新建一个分支,并切换到该分支：

> git checkout -b [branch]

例如：创建名称为dev的分支并切换到该分支上：

> git checkout -b dev

**注意**：在git中创建一个空分支

有时候我们需要在Git里面创建一个空分支，该分支不继承任何提交，没有父节点，完全是一个干净的分支，例如我们需要在某个分支里存放项目文档。

使用传统的git checkout命令创建的分支是有父节点的，意味着新branch包含了历史提交，所以我们无法直接使用该命令。

使用 git checkout 的 --orphan 参数:

> git checkout --orphan doc

该命令会创建一个名为doc的分支，并且该分支下有前一个分支下的所有文件。

我们不想提交任何内容，所以我们需要把当前内容全部删除，用git命令：

> git rm -rf .

5. 删除分支

> git branch -d [branch-name]

例如：删除本地dev分支

> git branch -d dev

6. 将本地分支推送到远程服务器

除非你将分支推送到远端仓库，不然该分支就是不为他人所见的：

> git push origin [branch-name](:[branch-name])

例如：将本地分支dev推送到远程服务器

> git push origin dev:dev

第一个[branch-name]：本地仓库名称,第二个[branch-name]：远程仓库名,可以不一样。

这样远程仓库中也就创建了一个dev分支。

7. 删除远程分支

> git push origin --delete <branchName>

或者,可以使用这种语法,推送一个空分支到远程分支,其实就相当于删除远程分支：

> git push origin :<branchName>

**重命名远程分支**

在git中重命名远程分支，其实就是先删除远程分支，然后重命名本地分支，再重新提交一个远程分支。

例如，把远程分支dev重命名为develop，操作如下：

```
// 删除远程分支
git push origin --delete dev
// 重命名本地分支
git branch -m dev develop
// 推送本地分支
git push origin develop
```




## git push 详解

命令格式如下：

> git push <远程主机名> <本地分支名>:<远程分支名>

例如：

> git push origin master:master

命令中的本地分支是指将要被推送到远端的分支，而远程分支是指推送的目标分支，即将本地分支合并到远程分支。

如果省略远程分支名，则表示将本地分支推送与之存在”追踪关系”的远程分支(通常两者同名)，如果该远程分支不存在，则会被新建。

> git push origin master

上面命令表示，将本地的master分支推送到origin主机的master分支。如果后者不存在，则会被新建。origin是一个远程厂库地址。

---

如果省略本地分支名，则表示删除指定的远程分支，因为这等同于推送一个空的本地分支到远程分支，这条命令是删除远程master分支。

```
git push origin :master
# 等同于
git push origin --delete master
```

上面命令表示删除origin主机的master分支。

---

如果当前分支与远程分支之间存在追踪关系（即分支名相同），则本地分支和远程分支都可以省略。

> git push origin

上面命令表示，将当前分支推送到origin主机的对应分支。

---

如果当前分支只有一个追踪分支，那么主机名都可以省略。

git push

---

> git push -u origin master

上面命令将本地的master分支推送到origin主机，同时指定origin为默认主机，后面就可以不加任何参数使用git push了。




## git pull 详解

git pull命令用于从另一个存储库或本地分支获取并集成(整合)。
git pull命令的作用是：取回远程主机某个分支的更新，再与本地的指定分支合并。命令格式如下：

> git pull <远程主机> <远程分支>:<本地分支>

将远程存储库中的更改合并到当前分支中。在默认模式下，git pull是git fetch后跟git merge FETCH_HEAD的缩写。

更准确地说，git pull使用给定的参数运行git fetch，并调用git merge将检索到的分支头合并到当前分支中。
使用--rebase，它运行git rebase而不是git merge。

例如：拉取origin/next分支,与本地的master分支合并

> git pull origin next:master

如果远程分支(next)要与当前分支合并，则冒号后面的部分可以省略。上面命令可以简写为：

> git pull origin next

如果当前分支与远程分支存在追踪关系，git pull就可以省略远程分支名。

> git pull origin

如果当前分支只有一个追踪分支，连远程主机名都可以省略。

> git pull

上面命令表示，拉取origin/next分支,与本地的master分支合并。实质上，这等同于先做git fetch，再执行git merge。

```
git fetch origin
git merge origin/next
```

**git pull 和 git fetch 的区别**

- git fetch：相当于是从远程获取最新版本到本地,不会自动合并。
- git pull：相当于是从远程获取最新版本并merge到本地

git fetch 更新远程仓库的方式如下：

```
// 切换到master分支
git checkout master
// 从远程的origin的master分支下载最新的版本到origin/master分支上
git fetch origin master
// 比较本地的master分支和origin/master分支的差别
git log -p master..origin/master
// 合并origin/master分支到master分支
git merge origin/master
```

上述的另一种清晰的实现：

```
git checkout master
// 在本地新建一个temp分支，并将远程origin仓库的master分支代码下载到本地temp分支
git fetch origin master:tmp
// 比较本地代码与刚刚从远程下载下来的代码的区别
git diff tmp
// 合并temp分支到本地的master分支
git merge tmp
// 如果不想保留本地temp分支,可以用这步删除
git branch -d temp
```



在合并改动之前，也可以使用如下命令查看：

```
git diff <source_branch> <target_branch>
```

在dev分支开发代码
git checkout dev  # 切换到dev分支进行开发
# 开发代码之后，我们有两个选择
# 第一个：如果功能开发完成了，可以合并主分支
git checkout master  # 切换到主分支
git merge dev  # 把dev分支的更改和master合并
git push  # 提交主分支代码远程
git checkout dev  # 切换到dev远程分支
git push  # 提交dev分支到远程



## 标签(tag)操作

1.列出所有tag

> git tag

2.打轻量标签

> git tag [tag name]

3.附注标签

> git tag -a [tag name] -m [message]

例如：打v1.0标签

> git tag -a v1.0 -m 'v1.0 release'

4.后期打标签

> git tag -a [tag name] [version]

例如：可以执行如下命令以创建一个叫做 1.0.0 的标签

> git tag 1.0.0 1b2e1d63ff

1b2e1d63ff 是你想要标记的提交 ID 的前 10 位字符。

你也可以用该提交 ID 的少一些的前几位，只要它是唯一的。

5.查看tag信息

> git show [tag]

6.删除本地tag

> git tag -d [tag]

例如：删除本地v1.0标签

> git tag -d v1.0

7.提交指定tag

> git push [remote] [tag]

例如：将v1.0标签推送到远程服务器上

> git push origin v1.0

8.提交所有tag

> git push [remote] --tags

9.删除远程tag

> git push origin --delete tag <tag name>




## 替换本地改动

假如你做错事（自然，这是不可能的），你可以使用如下命令替换掉本地改动：

```
git checkout -- <filename>
```

此命令会使用 HEAD 中的最新内容替换掉你的工作目录中的文件。
已添加到缓存区的改动，以及新文件，都不受影响。

假如你想要丢弃你所有的本地改动与提交，可以到服务器上获取最新的版本并将你本地主分支指向到它：

```
git fetch origin
git reset --hard origin/master
```

## 有用的贴士

内建的图形化 git：

```
gitk
```

彩色的 git 输出：

```、、
git config color.ui true
```

显示历史记录时，只显示一行注释信息：

```
git config format.pretty oneline
```

交互地添加文件至缓存区：

```
git add -i
```
## 链接与资源

### 图形化界面

- [GitX (L) (OSX, open source)](http://gitx.laullon.com/)
- [Tower (OSX)](https://www.git-tower.com/mac/)
- [Source Tree (OSX, free)](https://www.sourcetreeapp.com/)
- [GitHub for Mac (OSX, free)](https://desktop.github.com/)

### 指南与手册

Git 社区参考书(https://book.git-scm.com/)

http://marklodato.github.io/visual-git-guide/index-zh-cn.html




```
march:GoProject march$ git clone https://github.com/maoqiqi/aa.git
march:GoProject march$ cd aa
march:aa march$ git checkout develop
march:aa march$ git reset --hard 89a22fe54278745d5123ff5a8caa0e176db1d9fc
march:aa march$ git push -f origin develop

march:aa march$ git add .
march:aa march$ git commit -m "kong"
march:aa march$ git push origin develop

march:aa march$ git checkout master

march:aa march$ git merge develop

march:aa march$ git add .
march:aa march$ git commit -m "kong ok"
march:aa march$ git push
```