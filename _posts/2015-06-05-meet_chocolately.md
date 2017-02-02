---
layout: post
title: 利用Chocolatey快速在windows下搭建开发环境
category : language-instrument
author: Max
tags : [choco]
---


# 概述

在Linux下，有apt-get/yum 帮助安装应用程序，如今在windows下，出现了Chocolatey（背后使用Nuget）使用户能通过命令行来安装应用程序。

需要说明的是， Chocolatey 只是把官方下载路径封装到了 Chocolatey 中，下载源都是其官方路径，如果原软件是需要 Licence 注册的话，那么 Chocolatey 下载安装好的软件还是需要购买注册。Chocolatey 一般会选用免费 Licence 可用的软件。

# 安装Chocolatey

## 用命令行安装

首先，若想指定安装位置，则需在系统环境变量中新建一个`ChocolateyInstall`变量，指明安装路径（若不存在，需要手动创建）。

然后，以管理员身份打开一个命令行工具，执行下述命令：
```
@powershell -NoProfile -ExecutionPolicy unrestricted -Command "iex ((new-object net.webclient).DownloadString('https://chocolatey.org/install.ps1'))" && SET PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin
```
就可以将Chocolatey安装到指定目录（默认为C:\ProgramData）中，并自动设置环境变量，使得我们可以在任何位置调用Chocolatey命令。

通过`chocolatey /?`或`chocolatey help`查看帮助。

# 搭建开发环境

用Chocolatey安装软件的命令是`choco install`, 短写是 `cinst`，可安装的应用程序，可以参见官方[Package列表](https://chocolatey.org/packages)。

以下是window下开发常用的开发环境应用：

安装 git
```
choco install git.install
```

安装 node
```
choco install nodejs.install
```

安装 vagrant
```
choco install vagrant
```

安装 virtual box
```
choco install virtualbox
```

安装编辑器 notepad++/atom/sublime
```
choco install notepadplusplus.install
choco install Atom
choco install SublimeText3
```

安装 Ruby
```
choco install ruby
```

安装 Python
```
choco install python
```

安装 Java JDK7 或 JDK8
```
choco install jdk7
choco install jdk8
```
