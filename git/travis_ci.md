# 使用Travis CI进行持续集成

[ ![Build Status](https://travis-ci.org/maoqiqi/notes.svg?branch=master)](https://travis-ci.org/maoqiqi/notes)
[ ![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg) ](http://www.apache.org/licenses/LICENSE-2.0)
[ ![Author](https://img.shields.io/badge/Author-March-orange.svg) ](fengqi.mao.march@gmail.com)

GitHub项目的免费持续集成平台。(Free continuous integration platform for GitHub projects.)


## 目录

* [概述](#概述)
* [登录](#登录)
* [构建项目](#构建项目)
* [部署](#部署)
  * [环境变量](#环境变量)
  * [加密信息](#加密信息)
  * [加密文件](#加密文件)
* [运行流程](#运行流程)
* [运行状态](#运行状态)
* [自动部署](#自动部署)
* [高大上标志](#高大上标志)


## 概述

Travis CI 提供的是持续集成服务（Continuous Integration，简称 CI）。
它绑定 Github 上面的项目，只要有新的代码，就会自动抓取。然后，提供一个运行环境，执行测试，完成构建，还能部署到服务器。

持续集成指的是只要代码有变更，就自动运行构建和测试，反馈运行结果。确保符合预期以后，再将新代码"集成"到主干。

持续集成的好处在于，每次代码的小幅变更，就能看到运行结果，从而不断累积小的变更，而不是在开发周期结束时，一下子合并一大块代码。


## 登录

打开 https://travis-ci.org ，使用 Github 账户登入 Travis CI。

Travis 会列出 Github 上面你的所有仓库，以及你所属于的组织。此时，选择你需要 Travis 帮你构建的仓库，打开仓库旁边的开关。
一旦激活了一个仓库，Travis 会监听这个仓库的所有变化。

![travis_ci_projects](images/travis_ci_projects.png)


## 构建项目

接下来的步骤很清楚，官方也有配图说明：

![travis_ci](images/travis_ci.png)

### 添加.travis.yml

Travis 要求项目的根目录下面，必须有一个`.travis.yml`文件。这是配置文件，指定了 Travis 的行为。
该文件必须保存在 Github 仓库里面，一旦代码仓库有新的 Commit，Travis 就会去找这个文件，执行里面的命令。

这个文件采用 YAML 格式。下面是一个最简单的 Java 项目的`.travis.yml`文件。

```
language: generic

script: true
```

上面代码中，设置了两个字段。language字段指定了默认运行环境，这里设定 generic 表示不设定。script字段指定要运行的脚本，设定 true 表示不执行任何脚本，状态直接设为成功。

Travis 默认提供的运行环境，请参考[官方文档](https://docs.travis-ci.com) 。


## 部署

script阶段结束以后，还可以设置通知步骤（notification）和部署步骤（deployment），它们不是必须的。

部署的脚本可以在script阶段执行，也可以使用 Travis 为几十种常见服务提供的快捷部署功能。比如，要部署到 Github Pages，可以写成下面这样。

```
deploy:
  provider: pages
  skip_cleanup: false
  github_token: "$GITHUB_TOKEN"
  keep_history: true
  on:
    branch: master
```

其他部署方式，请查看[文档](https://github.com/travis-ci/docs-travis-ci-com/tree/master/user/deployment)。

### 环境变量

.travis.yml的env字段可以定义环境变量。

```
env:
  - DB=postgres
  - SH=bash
  - PACKAGE_VERSION="1.0.0"
```

然后，脚本内部就可以使用这些变量了。

有些环境变量（比如用户名和密码）不能公开，这时可以通过 Travis 网站，写在每个仓库的设置页里面，Travis 会自动把它们加入环境变量。
这样一来，脚本内部依然可以使用这些环境变量，但是只有管理员才能看到变量的值。

![encrypt_file](images/travis_ci_encrypt_file.png)

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

这个时候去看一下当前目录下的`.travis.yml`，会多出几行。

```
env:
  global:
    secure: ovOrUJe9M6L2Yc9BoMEYBnoqBwTYnszFkfFYS1Br+F88wOAkRQTQK8LswioKcCCxyJBnw+hfCKphWGFA56Mvl1kLpHRJibRsfRfcF1clWD4A8D+ZhTobjgFu+P9PJHXksoJfd2c6UtMUCtLXQ2fPXrV8whUh70Ol6+6qcp7azdnn8G+wvzgxP0KamVYguKBWtqDnJOduSvly//UBQRiEygvnGnw65oIZ7FvW4ksVNNtZHP1NGEH+w3u5wNVo7jAkHGAJMoBf2l7syQUxNmVe3zYk2k0nJAfPrlIAAWbADDHie/srYVYwDX47LMocfSgC14ZPI7ewGclRrPXI4WTlEnejiBa4bUXIEUYAd3mWqWOUvvSYVgMWtFRv0cIhjg62deF611Ci85+EHZVcq1TmS4OBD+q3zUxDNYPzU2bVHrG0oeBTpuxhrHIAukNZgpUmVlYs5VnejTHxkI3gCOnzxZ/nMW2DELAVMFTo/OjcEbiIjh0RAyA6vuQa85c9gL5l2cuw7E6pK4fJZiuNZqnkVYq37J/SutgDwcoGui8OA6Da5a0ipuRrQ8VG68GylIzg+lakNPfHJjjF0mL4FLWVdBqezZj142dGShbDMZS257R9E7dOnPnWRXV38twivBSuokL8TvAsJ7ayNjTrjuW1WQ+DN5c5pSp1weXklrUKt0g=
```

我们需要让 Travis 远程登录自己的服务器，所以需要将本地保存着的 SSH 私钥进行加密处理。

### 加密文件

如果要加密的是文件（比如私钥），Travis 提供了加密文件功能。

> 生成GitHub对应的SSH公钥并上传到服务器，用于跟仓库通信。可以参考[SSH秘钥](ssh.md)。

安装命令行客户端以后，使用下面的命令登入 Travis CI。

```
travis login
```

会要求你输入 GitHub 的账号密码，这个是走 GitHub 的服务。

然后，进入项目的根目录，使用`travis encrypt-file`命令加密那些想要加密的文件。

```
// 进入项目根目录
// 进行加密
travis encrypt-file id_rsa_travis_ci
```

上面的代码对文件bacon.txt进行加密，加密后会生成id_rsa_travis_ci.enc，该文件需要提交到代码库。
此外，该命令还会生成一个环境变量$encrypted_xxx_key，保存密钥，储存在 Travis CI，文件解密时需要这个环境变量。
你需要把解密所需的openssl命令，写在.travis.yml的before_install字段里面。这些都写在上面的命令行提示里面。

同样--add参数可以自动把环境变量写入.travis.yml。

这个时候去看一下当前目录下的`.travis.yml`，会多出几行。

```
before_install:
  openssl aes-256-cbc -K $encrypted_e69766e2a2e2_key -iv $encrypted_e69766e2a2e2_iv -in id_rsa_travis_ci.enc -out id_rsa_travis_ci -d
```

再次查看我们的 Travis CI 网页，发现多了一些变化。

![encrypt_file](images/travis_ci_encrypt_file.png)


## 运行流程

Travis 的运行流程很简单，任何项目都会经过两个阶段。

* install 阶段：安装依赖。

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

  如果不需要安装，即跳过安装阶段，就直接设为true，`install: true`。

* script 阶段：运行脚本。

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

## 运行状态

最后，Travis 每次运行，可能会返回四种状态。

* passed：运行成功，所有步骤的退出码都是0。
* cancel：用户取消执行。
* error：before_install、install、before_script有非零退出码，运行会立即停止。
* failed：script有非零状态码，会继续运行。


## 自动部署

如果可以免密登录服务器了，那么写一个部署脚本，在登录时执行该脚本就可以了。


## 高大上标志

如果想在GitHub的项目首页显示一个高大上的build:passing标志，比如像这样：

todo

可以在根目录的README.md中加上一个图片链接：

```
[![Build Status](https://travis-ci.org/maoqiqi/notes.svg?branch=master)](https://travis-ci.org/maoqiqi/notes)
```