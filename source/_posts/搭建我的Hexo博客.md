---
title: 搭建我的Hexo博客
date: 2024-01-17 13:35:13
description: 记录了我通过hexo搭建博客的过程
tags:
- Hexo
- Blog
- Github
categories:
- Blog
---

# 搭建我的Hexo博客

Hexo是一个快速、简洁高效的博客框架，部署步骤简单，编写支持Markdown

这次记录一下我的博客搭建过程，搭配[Hexo官方文档](https://hexo.io/zh-cn/docs/index.html)食用更佳

## 基础安装


### 基础软件安装

安装Hexo需要安装以下程序
- [Node.js](https://nodejs.org/en)
- [Git](https://git-scm.com/)

#### Linux

##### Arch系

- 安装
    - sudo pacman -S nodejs npm git

##### Debian系

1. 更新本地apt索引
    - `sudo apt-get update`
2. 安装
    - `sudo apt-get install nodejs npm git`

##### Fedora系

- 安装
    - `sudo dnf install nodejs npm git`

##### CentOS

- 安装
    - `sudo yum install nodejs npm git`

#### Windows

Windows可以通过官网进行下载和安装[Git](https://git-scm.com/)，[Node.js](https://nodejs.org/en/download/current)，[npm](https://www.npmjs.com/)

通过以下命令查看是否安装成功

```sh
$ git -v
git version 2.45.0
$ node -v
v21.7.3
$ npm -v
10.5.2
```

### 安装Hexo

然后通过`npm`安装`hexo`，由于在npm的服务器在国外，在国内访问的速度比较慢，可以通过`cnpm`使用[国内的镜像站](https://npmmirror.com/)进行安装

```sh
$ sudo npm install -g cnpm --registry=https://registry.npmmirror.com
```

输入`npm list --global`可以看到`hexo`已经被安装

<img src="Hexo_Installed.png" style="max-width:50%">



## Hexo博客的建立与管理

创建一个文件夹blog用于存储我的博客文件

```sh
$ mkdir ~/Documents/blog
$ cd ~/Documents/blog
```

在文件夹中进行初始化

```sh
$ hexo init
```

等待初始化完成

<img src="Hexo_init.png" style="max-width:100%;">

新建完成，得到这样的目录结构

```
.
├── _config.landscape.yml
├── _config.yml
├── node_modules
├── package.json
├── scaffolds
├── source
│   └── _posts
├── themes
└── yarn.lock
```

博客的源文件会储存在`source`目录内，

编辑`_config.yml`文件，将post_asset_folder:false中的false改为true，该设置使hexo在创建一篇博客同时创建一个文件夹以存储这个博客的图片等有用的文件

## Hexo的构建

初始化Hexo以后，可以直接构建

```sh
$ hexo generate       # 或者hexo g
```

随后会生成一个public目录，里面放置着根据配置生成的博客文件

<img src="Hexo_Generate.png" style="max-width:100%">

## 启动内建服务器

启动hexo内建服务器，然后就可以使用浏览器通过本地IP进行访问了

```sh
$ hexo server         # 或者hexo s
```

<img src="Hexo_Server.png" style="max-width:75%;">

在浏览器中输入`127.0.0.1:4000`即可查看本地博客

<img src="Hexo_Local_Server.png" style="max-width:100%;">

## Hexo部署远程平台

在上述操作中已经把博客部署到本地，现在需要部署到远程平台

### 部署到Github

#### 创建一个Github账号

在[Github官网](https://github.com)上创建一个账号，点击左上角的`Sign up`根据提示进行操作即可（相信你们会的≖‿≖✧）

<img src="Github_Sign_Up.png" style="max-width:100%;">

#### 配置Git

**配置Git本地信息**

这些信息可以是虚拟的，会显示在git commit信息中

```sh
$ git config --global user.email "you@example.com"          # 配置邮箱
$ git config --global user.name "Your Name"                 # 配置kkk名字
```

在 Git 中缓存 GitHub 凭据，这样在推送博客时，不需要输入账号密码

**安装gh-cli**

ArchLinux：

```sh
$ sudo pacman -S github-cli
```

**缓存凭据**

```sh
$ gh auth login
```

<img src="GH_Auth_Login.png" style="max-width:100%;">

复制这串验证码，按`Enter`打开浏览器，粘贴验证码认证即可

<img src="GH_Auth_Web.png" style="max-width:100%;">
<img src="GH_Auth_Success.png" style="max-width:100%;">

#### 创建Github仓库

创建一个仓库，用于部署我们的博客

**注意**：将新仓库的名字设置为`<用户名>.github.io`

比如：你的用户名是Jack

那么创建一个叫做`Jack.github.io`的仓库

<img src="Github_Repo_Name.png" style="max-width:100%;">

点击创建即可

<img src="Create_Repo.png" style="max-width:100%;">

#### 安装推送插件

[hexo-deployer-git](https://github.com/hexojs/hexo-deployer-git)为我们提供了将hexo博客推送到Github的功能

安装命令如下，在博客目录下执行：

```sh
$ cnpm install hexo-deployer-git --save
```

#### 修改主配置文件

编辑博客目录下的_config.yml文件，添加以下内容

```yml [Blogroot]/_config.yml
# You can use this:
deploy:
  type: git
  repo: https://github.com/Jccc-l/Jccc-l.github.io.git
  branch: main
# token: ''
# message: [message]
# name: [git user]
# email: [git email]
# extend_dirs: [extend directory]
# ignore_hidden: false # default is true
# ignore_pattern: regexp  # whatever file that matches the regexp will be ignored when deploying
```

- `repo`：仓库的链接，复制HTTPS或SSH链接  
<img src="Copy_Repo_URL.png" style="max-width:50%;">
- `branch`：博客推送的仓库分支，Github创建的主分支默认是`main`，检查一下是否一致
- `token`：Github生成的token用于访问仓库身份认证，但是之前已经用github-cli缓存了凭据，可以删掉这个选项
- `message`：commit信息，默认：`Site updated: {{ now("yyyy-MM-dd HH:mm:ss") }}`
- `name`和`email`：commit信息中的名字和邮箱，前面用`git config`已经设置了全局的信息，可以删掉
- `ignore_hidden`：在发布博客时是否忽略隐藏文件

还有其他配置选项可以在[插件的仓库](https://github.com/hexojs/hexo-deployer-git)中查询

配置完成后，输入`hexo deploy`或（`hexo d`）进行博客推送

然后访问<用户名>.github.io就可以访问到你的hexo博客网站了

## 完成基本搭建

以后如果需要对博客进行增删改，通过以下步骤进行：

```sh
$ hexo new "Hello World"      # 或者hexo n "Hello World"      创建一篇叫做"Hello World"的博客
$ hexo clean                  # 或者hexo cl                   清除缓存文件
$ hexo generate               # 或者hexo g                    生成博客
$ hexo server                 # 或者hexo s                    启用内建服务器本地预览博客
$ hexo deploy                 # 或者hexo d                    推送博客到Github
```

以后就可以开心地写你的博客了。

