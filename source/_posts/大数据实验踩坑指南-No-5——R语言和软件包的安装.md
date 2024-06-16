---
title: 大数据实验踩坑指南_No.5——R语言和软件包的安装
mathjax: false
katex: false
date: 2024-06-16 16:30:37
description: 配置安装R语言和软件包的过程记录
categories:
- Big Data
tags:
- Big Data
- R
- Linux
---

# 大数据实验踩坑指南_No.5——R语言和软件包的安装

## 安装R

安装R的步骤可以在CRAN[^3]上查看，这里不赘述
[^3]: [Ubuntu Packages For R - Brief Instructions](https://cran.r-project.org/bin/linux/ubuntu/)

## 安装依赖

由于在Ubuntu上不支持通过二进制包进行安装R包，需要安装`r-base-dev`软件包来编译R包

```sh
$ sudo apt-get install r-base-dev
```

安装完成后，为了后续能顺利安装好所需的R包，首先需要用`apt`安装以下一些依赖<a id="RDependencies"></a>
- libcurl4-openssl-dev：用于网络请求。
- libxml2-dev：用于处理XML数据。
- libssl-dev：SSL/TLS支持。
- libncurses5-dev：用于文本用户界面。
- libbz2-dev：bzip2压缩支持。
- libreadline-dev：用于读取和编辑命令行输入。
- libsqlite3-dev：SQLite数据库支持。
- libpng-dev：PNG图像支持。
- libjpeg-dev：JPEG图像支持。
- libtiff-dev：TIFF图像支持。
- libpango1.0-dev：文本布局和渲染。
- libharfbuzz-dev：文本形状化。
- libfreetype6-dev：TrueType字体支持。
- liblcms2-dev：色彩管理。
- libpangoft2-dev：Pango和FreeType的接口。
- libudunits2-dev：用于处理单位的库。
- libmysqlclient-dev：MySQL交互支持。
- libmysqlclient21：为MySQL提供运行时支持。

```sh
sudo apt-get install libcurl4-openssl-dev libxml2-dev libssl-dev \
    libncurses5-dev libbz2-dev libreadline-dev libsqlite3-dev libpng-dev \
    libjpeg-dev libtiff-dev libpango1.0-dev libharfbuzz-dev libfreetype6-dev \
    liblcms2-dev libpangoft2-1.0-0 libudunits2-dev libmysqlclient-dev libmysqlclient21
```

> 有些包名包含了版本号，可能随着时间推移，包名会发生改变，可以通过`apt search`命令搜索正确的包名

> 如果由于网络原因安装失败，尝试通过`Software & Updates`修改软件源
>
> <img src="Software&Update.png" style="max-width:50%">
> <img src="set_apt_mirror.png" style="max-width:80%">

由于在Ubuntu上不支持通过二进制包进行安装R包，需要安装以下一些工具
- build-essential：Ubuntu的基础编译工具集，包含了dpkg-dev、g++、gcc、libc6-dev（或libc-dev）、make
- make：用于构建过程
- autoconf、automake、libtool：用于生成和维护自动化的Makefile。
- texinfo：用于创建和维护Texinfo格式文档。
- pkg-config：用于查询库的位置和编译选项。
- cmake：编译系统
这些工具可以通过以下命令安装：

```sh
sudo apt-get install make autoconf automake libtool texinfo pkg-config build-essential cmake
```

安装完成后，为了顺利下载，需要把软件源换为国内的镜像源，输入以下命令写入配置文件，设置`hadoop`用户的R镜像

```sh
$ echo 'options(repos=c(USTC="https://mirrors.ustc.edu.cn/CRAN/"))' >> ~/.Rprofile
```

在终端下运行以下命令启动R

```sh
$ R
```

可以执行下面的命令退出R：

```R
> q()
```

## 安装R包

启动R进入R命令提示符状态，执行以下命令安装所需要的R软件包

> 其中的`c()`表示一个向量，`install.packages()`函数会下载向量里的元素对应的软件包，如果下载单个软件包，可以执行`install.packages("package_name")`
> 
> 普通用户下载R包会下载到用户家目录下的R目录内，第一次下载会创建目录`$HOME/R/x86_64-pc-linux-gnu-library/4.4`
> 
> dependencies选项让R安装软件包时，把它们所依赖的包也安装上
> repos选项如果上面已经配置了，这里可以省略

```R
> install.packages(c('RMySQL','devtools','ggplot2'),repos = c(USTC="https://mirrors.ustc.edu.cn/CRAN/"))
Installing packages into ‘/usr/local/lib/R/site-library’
(as ‘lib’ is unspecified)
Warning in install.packages(c("RMySQL", "ggplot2", "devtools")) :
  'lib = "/usr/local/lib/R/site-library"' is not writable
Would you like to use a personal library instead? (yes/No/cancel) yes
Would you like to create a personal library
‘/home/hadoop/R/x86_64-pc-linux-gnu-library/4.4’
to install packages into? (yes/No/cancel) yes
```

等待编译安装完成

如果安装失败，尝试重新打开终端和R，重新执行安装命令

> ```R
> Warning message:
> In install.packages(c("RMySQL", "devtools", "ggplot2"), repos = c(USTC = "https://mirrors.ustc.edu.cn/CRAN/")) :
>   installation of package ‘RMySQL’ had non-zero exit status
> ```
> 如果出现以上错误，多重复尝试几次安装命令，并确保[上面的依赖](#RDependencies)安装完全


最后在R命令提示符下，再执行如下命令安装`taiyun/recharts`

```R
> devtools::install_github('taiyun/recharts')
```
