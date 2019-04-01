# 使用Travis CI进行持续集成

GitHub项目的免费持续集成平台。(Free continuous integration platform for GitHub projects.)

[![Build Status](https://travis-ci.org/maoqiqi/notes.svg?branch=master)](https://travis-ci.org/maoqiqi/notes)

## 目录

* [概述](*概述)
* [登录](#登录)
* [构建项目](#构建项目)
* [环境变量](#环境变量)
* [自动部署到远程服务器](#自动部署到远程服务器)
* [高大上标志](#高大上标志)


## 概述

Travis CI 提供的是持续集成服务（Continuous Integration，简称 CI）。
它绑定 Github 上面的项目，只要有新的代码，就会自动抓取。然后，提供一个运行环境，执行测试，完成构建，还能部署到服务器。

持续集成指的是只要代码有变更，就自动运行构建和测试，反馈运行结果。确保符合预期以后，再将新代码"集成"到主干。

持续集成的好处在于，每次代码的小幅变更，就能看到运行结果，从而不断累积小的变更，而不是在开发周期结束时，一下子合并一大块代码。


## 登录

打开 https://travis-ci.org ，使用 Github 账户登入 Travis CI。

Travis 会列出 Github 上面你的所有仓库，以及你所属于的组织。此时，选择你需要 Travis 帮你构建的仓库，打开仓库旁边的开关。一旦激活了一个仓库，Travis 会监听这个仓库的所有变化。

todo


## 构建项目

接下来的步骤很清楚，官方也有配图说明：

![travis_ci](travis_ci.png)

### 添加.travis.yml

Travis 要求项目的根目录下面，必须有一个`.travis.yml`文件。这是配置文件，指定了 Travis 的行为。
该文件必须保存在 Github 仓库里面，一旦代码仓库有新的 Commit，Travis 就会去找这个文件，执行里面的命令。

这个文件采用 YAML 格式。下面是一个最简单的 Java 项目的`.travis.yml`文件。

```
language: generic

script: true
```

上面代码中，设置了两个字段。language字段指定了默认运行环境，这里设定 generic 表示通用。script字段指定要运行的脚本，设定 true 表示不执行任何脚本，状态直接设为成功。

Travis 默认提供的运行环境，请参考[官方文档](https://docs.travis-ci.com) 。

### 运行流程

Travis 的运行流程很简单，任何项目都会经过两个阶段。

* install 阶段：安装依赖
* script 阶段：运行脚本

#### install字段

install字段用来指定安装脚本。

```
install: ./install-dependencies.sh
```

如果有多个脚本，可以写成下面的形式。

```
install:
  - command1
  - command2
```

上面代码中，如果command1失败了，整个构建就会停下来，不再往下进行。

如果不需要安装，即跳过安装阶段，就直接设为true。

```
install: true
```

#### script字段

script字段用来指定构建或测试脚本。

```
script: bundle exec thor build
```

如果有多个脚本，可以写成下面的形式。

```
script:
  - command1
  - command2
```

> 注意，script与install不一样，如果command1失败，command2会继续执行。但是，整个构建阶段的状态是失败。

如果command2只有在command1成功后才能执行，就要写成下面这样。

```
script: command1 && command2
```

### 部署

script阶段结束以后，还可以设置通知步骤（notification）和部署步骤（deployment），它们不是必须的。

部署的脚本可以在script阶段执行，也可以使用 Travis 为几十种常见服务提供的快捷部署功能。比如，要部署到 Github Pages，可以写成下面这样。

```
deploy:
  provider: pages
  skip_cleanup: true
  github_token: $GITHUB_TOKEN # Set in travis-ci.org dashboard
  on:
    branch: master
```

其他部署方式，请看[官方文档](#官方文档)。

### 钩子方法

Travis 为上面这些阶段提供了7个钩子。

* before_install：install 阶段之前执行。
* before_script：script 阶段之前执行。
* after_failure：script 阶段失败时执行。
* after_success：script 阶段成功时执行。
* before_deploy：deploy 步骤之前执行。
* after_deploy：deploy 步骤之后执行。
* after_script：script 阶段之后执行。

完整的生命周期，从开始到结束是下面的流程。

```
1. before_install
2. install
3. before_script
4. script
5. after_success or after_failure
6. [OPTIONAL] before_deploy
7. [OPTIONAL] deploy
8. [OPTIONAL] after_deploy
9. after_script
```

### 运行状态

最后，Travis 每次运行，可能会返回四种状态。

* passed：运行成功，所有步骤的退出码都是0。
* cancel：用户取消执行。
* error：before_install、install、before_script有非零退出码，运行会立即停止。
* failed：script有非零状态码，会继续运行。


## 使用技巧

###  环境变量

.travis.yml的env字段可以定义环境变量。

```
env:
  - DB=postgres
  - SH=bash
  - PACKAGE_VERSION="1.0.*"
```

然后，脚本内部就可以使用这些变量了。

有些环境变量（比如用户名和密码）不能公开，这时可以通过 Travis 网站，写在每个仓库的设置页里面，Travis 会自动把它们加入环境变量。
这样一来，脚本内部依然可以使用这些环境变量，但是只有管理员才能看到变量的值。

todo

### 加密信息

如果不放心保密信息明文存在 Travis 的网站，可以使用 Travis 提供的加密功能。

首先，安装 Ruby 的包travis。

```
gem install travis
```

然后，就可以用travis encrypt命令加密信息。

在项目的根目录下，执行下面的命令。

```
travis encrypt key=value
```

上面命令中，key是要加密的变量名，value是要加密的变量值。执行以后，屏幕上会输出如下信息。

```
secure: ".... encrypted data ...."
```

现在，就可以把这一行加入.travis.yml。

```
env:
  global:
    - secure: ".... encrypted data ...."
```

然后，脚本里面就可以使用环境变量$key了，Travis 会在运行时自动对它解密。

travis encrypt命令的--add参数会把输出自动写入`.travis.yml`，省掉了修改env字段的步骤。

```
travis encrypt key=value --add
```

### 加密文件

如果要加密的是文件（比如私钥），Travis 提供了加密文件功能。

安装命令行客户端以后，使用下面的命令登入 Travis CI。

```
travis login

We need your GitHub login to identify you.
This information will not be sent to Travis CI, only to api.github.com.
The password will not be displayed.

Try running with --github-token or --auto if you don't want to enter your password anyway.

Username: xxx@xxx.xxx
Password for xxx@xxx.xxx: ***
Successfully logged in as demo!
```

会要求你输入 GitHub 的账号密码，这个是走 GitHub 的服务。

然后，进入项目的根目录，使用`travis encrypt-file`命令加密那些想要加密的文件。

```

$ travis encrypt-file bacon.txt

encrypting bacon.txt for rkh/travis-encrypt-file-example
storing result as bacon.txt.enc
storing secure env variables for decryption

Please add the following to your build script (before_install stage in your .travis.yml, for instance):

    openssl aes-256-cbc -K $encrypted_0a6446eb3ae3_key -iv $encrypted_0a6446eb3ae3_key -in bacon.txt.enc -out bacon.txt -d

Pro Tip: You can add it automatically by running with --add.

Make sure to add bacon.txt.enc to the git repository.
Make sure not to add bacon.txt to the git repository.
```

上面的代码对文件bacon.txt进行加密，加密后会生成bacon.txt.enc，该文件需要提交到代码库。
此外，该命令还会生成一个环境变量$encrypted_0a6446eb3ae3_key，保存密钥，储存在 Travis CI，文件解密时需要这个环境变量。
你需要把解密所需的openssl命令，写在.travis.yml的before_install字段里面。这些都写在上面的命令行提示里面。

--add参数可以自动把环境变量写入.travis.yml。

```
travis encrypt-file bacon.txt --add

encrypting bacon.txt for rkh/travis-encrypt-file-example
storing result as bacon.txt.enc
storing secure env variables for decryption

Make sure to add bacon.txt.enc to the git repository.
Make sure not to add bacon.txt to the git repository.
Commit all changes to your .travis.yml.
```

这个时候去看一下当前目录下的`.travis.yml`，会多出几行。

```
before_install:
  - openssl aes-256-cbc -K $encrypted_d89376f3278d_key -iv $encrypted_d89376f3278d_iv
  -in id_rsa.enc -out ~\/.ssh/id_rsa -d
```

为保证权限正常，多加一行设置权限的 shell

```
before_install:
  - openssl aes-256-cbc -K $encrypted_d89376f3278d_key -iv $encrypted_d89376f3278d_iv
    -in id_rsa.enc -out ~/.ssh/id_rsa -d
  - chmod 600 ~/.ssh/id_rsa
```

### 自动部署

因为我们需要让 travis 远程登录自己的服务器，所以需要将本地保存着的 SSH 私钥进行加密处理。参考上面的[加密文件](#加密文件)。

如果可以免密登录服务器了，那么写一个部署脚本，在登录时执行该脚本就可以了，一切就是这么顺其自然了。

写部署脚本

```
#!/bin/bash
cd /path/to/your-project
git pull origin master
echo 'travis build done!'
```

在`.travis.yml`配置文件中写下这么两行：

```
after_success:
  - ssh your-user@your-ip "./your-shell-script"
```

### 设置SSH加密密钥

因此，让我们回顾一下我们需要做些什么来安全地准备我们的远程主机和Travis之间的SSH隧道：

1. 生成专用的SSH密钥（更容易隔离和撤销）;
2. 加密私钥使其只能由Travis CI读取（因此我们也可以安全地提交！）;
3. 将公钥复制到远程SSH主机上;
4. 不必要的文件后清理 ;
5. 将修改后的文件转换为Git。

这转化为：

```
ssh-keygen -t rsa -b 4096 -C'build@travis-ci.org'-f ./deploy_rsa
travis encrypt-file deploy_rsa --add
ssh-copy-id -i deploy_rsa.pub <ssh-user> @ <部署主机>
rm -f deploy_rsa deploy_rsa.pub
git add deploy_rsa.enc .travis.yml
```

### 在Travis作业中设置SSH解密

在我们能够制定任何东西之前，我们仍然需要设置一些东西：

1. 第一次连接到主机时阻止SSH交互式提示（这意味着，对于每个构建）;
2. 解密加密的SSH私钥;
3. 将密钥添加到当前ssh-agent以使任何基于SSH的命令与私钥位置无关。

上一次travis encrypt-file调用的结果将生成与我们类似的内容.travis.yml：

```
addons:
  ssh_known_hosts: <deploy-host>

before_deploy:
- openssl aes-256-cbc -K $encrypted_<...>_key -iv $encrypted_<...>_iv -in deploy_rsa.enc -out /tmp/deploy_rsa -d
- eval "$(ssh-agent -s)"
- chmod 600 /tmp/deploy_rsa
- ssh-add /tmp/deploy_rsa
```

### Travis部署脚本

```
deploy:
  provider: script
  skip_cleanup: true
  script: rsync -r --delete-after --quiet $TRAVIS_BUILD_DIR/<dir> <ssh-user>@<deploy-host>:path/to/files
  on:
    branch: master
```


## 高大上标志

如果想在GitHub的项目首页显示一个高大上的build:passing标志，比如像这样：

todo

可以在根目录的README.md中加上一个图片链接：

```
[![Build Status](https://travis-ci.org/maoqiqi/notes.svg?branch=master)](https://travis-ci.org/maoqiqi/notes)
```