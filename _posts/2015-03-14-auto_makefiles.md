---
layout: post
title: Auto Makefiles
category : language-instrument
author: Max
tags : [make, makefile]
---


# 自动生成Makefile流程

```
1)在工程根目录下新建Makefile.am文件，并在每个包含源代码的子目录下也新建makefile.am文件
2)运行autoscan命令,生成configure.scan文件
3)将configure.scan 文件重命名为configure.ac（configure.in），并修改configure.ac文件
4)运行autoheader，生成文件config.h.in
     (configure.in里有宏AC_CONFIG_HEADER()时用)
5)运行aclocal命令得到aclocal.m4文件
6)运行libtoolize，生成一些支持文件(文件夹m4下）和ltmain.sh
     (需要用libtool生成共享库用)
7)运行autoconf命令得到configure文件
8)运行automake命令
     使用"automake -a"或"automake --add-missing"，会自动将config.guess、config.sub、
  install.sh、missing、depcomp等文件补齐。
9)运行./confiugre脚本
```

  需注意，automake -a命令补全的文件为符号链接，若要将本源代码发布，则还需要将这些链接替换为其所指的实际文件。

***

# **工具简介**
所必须的软件：autoconf/automake/m4/perl/libtool（其中libtool非必须）。

##autoconf

autoconf是一个用于生成可以自动地配置软件源码包，用以适应多种UNIX类系统的shell脚本工具，它可以适应各种类UNIX系统的需要。其中autoconf需要用到 m4，便于生成脚本。autoconf产生的配置脚本在运行时独立于autoconf，也就是说使用这些脚本的用户不需要安装autoconf。

autoconf生成的配置脚本通常名称是configure，得到这个文件，通常需要以下的依赖文件：

* configure.in文件：生成configure的必需文件，需要手动编写。

* aclocal.m4和acsite.m4文件：在编写了除autoconf提供的测试外的其他测试补充的时候，才会用到这两个文件，也需要手动编写。

* acconfig.h文件：如果使用了含有#define指令的头文件，则需要自行编写该文件，一般都需要使用，这个时候会生成另外一个config.h.in文件，这个文件需要和软件包一同发布。

## automake

automake是一个从Makefile.am文件自动生成Makefile.in的工具。为了生成Makefile.in，automake还需用到perl，由于automake创建的发布完全遵循GNU标准，所以在创建中不需要perl。

典型的automake输入文件是一系列简单的宏定义。处理所有相关的文件并创建Makefile.in文件。在一个项目的每个目录中通常仅包含一个Makefile.am。

目前automake支持三种目录层次：平坦模式（flat）、混合模式（shallow）和深层模式（deep）:

1. flat指的是所有文件都位于同一个目录中。

   就是所有源文件、头文件以及其他库文件都位于当前目录中，且没有子目录。Termutils就是这一类。

2. shallow指的是主要的源代码都储存在顶层目录，其他各个部分则储存在子目录中。

   就是主要源文件在当前目录中，而其它一些实现各部分功能的源文件位于各自不同的目录。automake本身就是这一类。

3. deep指的是所有源代码都被储存在子目录中；顶层目录主要包含配置信息。

   就是所有源文件及自己写的头文件位于当前目录的一个子目录中，而当前目录里没有任何源文件。 GNU cpio和GNU tar就是这一类。

## libtool

libtool是一款方便生成各种程序库的工具。

##autoscan

autoscan可以用来为软件包创建configure.in文件。autoscan在以命令行参数中指定的目录为根（如果未给定参数，则以当前目录为根）的目录树中检查源文件。为软件包创建一个configure.scan文件，该文件就是configure.in的前身。

##autoheader

能够产生供configure脚本使用的C #define语句模板文件。

##autom4te

对文件执行 GNU M4。

##autoreconf

如果有多个autoconf产生的配置文件，autoreconf可以保存一些相似的工作，它通过重复运行autoconf（以及在合适的地方运行autoheader）以重新产生autoconf配置脚本和配置头模板，这些文件保存在以当前目录为根的目录树中。

##autoupdate

该程序的作用是转换configure.in，从而使用新的宏名。


***

# **生成 Makefile示例**

## 工程结构示例

   假设工程目录project下文件结构如下所示：
````
|-<project>
　　　|-<src>
　　　　　　|-add.h
　　　　　　|-sub.h
　　　|-main.c
```

main.c
```
#include <stdio.h>
#include "src/add.h"
#include "src/sub.h"

int main(void)
{
    printf("%d\n",add(sub(100, 5), 1));
    return 0;
}
```

add.h
```
int add(int x, int y)
{
    return x + y;
}
```

sub.h
```
int sub(int x, int y)
{
    return x - y;
}
```

## Makefile.am

项目根目录下的makefile.am如下所示：
```
AUTOMAKE_OPTIONS = foreign               ##忽视ChangeLog、AUTHOR、NEWS、README、INSTALL、COPYING文件
SUBDIRS = src                            ##处理本目录前需要递归处理的子目录
bin_PROGRAMS = main                      ##生成可执行文件main
ACLOCAL_AMFLAGS = -I m4                  ##使用m4
main_SOURCES = main.c                    ##可执行文件main依赖的源文件
main_LDADD = src/libadd.la src/libsub.la ##可执行文件main连接时需要的库文件
```

每个包含源文件的子目录下都需要一个makefile.am。
本例中src/Makefile.am如下所示：
```
AUTOMAKE_OPTIONS = foreign
lib_LTLIBRARIES = libadd.la libsub.la  ##生成共享库libadd.la
libadd_la_SOURCES = add.h              ##共享库libadd.la依赖的源文件
libsub_la_SOURCES = sub.h
```

对于可执行文件和静态库类型，如果只想编译，不想安装到系统中，可以用noinst_PROGRAMS代替bin_PROGRAMS，noinst_LIBRARIES代替lib_LIBRARIES。

一般来说，如果不显式地进行声明，默认的几个全局路径如下：

* 安装路径前缀：$(prefix) = /usr/local

* 目标文件安装路径：bindir = $(prefix)/bin

* 库文件安装路径：libdir = $(prefix)/lib

* 数据文件安装路径：datadir = $(prefix)/share

* 系统配置安装路径：sysconfdir = $(prefix)/etc


## configure.ac

当我们利用autoscan工具生成confiugre.scan文件时，我们需要将confiugre.scan重命名为confiugre.ac文件。confiugre.ac调用一系列autoconf宏来测试程序需要的或用到的特性是否存在，以及这些特性的功能。

本例中，修改前的confiugre.scan如下：
````
#                                               -*- Autoconf -*-
# Process this file with autoconf to produce a configure script.

AC_PREREQ([2.63])
AC_INIT([FULL-PACKAGE-NAME], [VERSION], [BUG-REPORT-ADDRESS])
AC_CONFIG_SRCDIR([main.c])
AC_CONFIG_HEADERS([config.h])

# Checks for programs.
AC_PROG_CC

# Checks for libraries.

# Checks for header files.

# Checks for typedefs, structures, and compiler characteristics.

# Checks for library functions.

AC_CONFIG_FILES([Makefile
                 src/Makefile])
AC_OUTPUT
```

上示命令有两种，一种是以AC开头，表示是由autoconf提供，另一种是以AM开头，代表由automake提供。每个configure.scan文件都是以AC_INIT开头，以AC_OUTPUT结束。我们不难从文件中看出confiugre.ac文件的一般布局：
```
AC_INIT
 测试程序
 测试函数库
 测试头文件
 测试类型定义
 测试结构
 测试编译器特性
 测试库函数
 测试系统调用
AC_OUTPUT
```

修改后的结果如下：
```
#                                               -*- Autoconf -*-
# Process this file with autoconf to produce a configure script.

AC_PREREQ([2.63])
AC_INIT(test, 0.1, [])
AM_INIT_AUTOMAKE(test, 0.1)
AC_CONFIG_SRCDIR([main.c])
AC_CONFIG_HEADERS([config.h])
AC_CONFIG_MACRO_DIR([m4])

# Checks for programs.
AC_PROG_CC
AC_PROG_LIBTOOL

# Checks for libraries.

# Checks for header files.

# Checks for typedefs, structures, and compiler characteristics.

# Checks for library functions.

CFLAGS=""
CXXFLAGS=""

AC_ARG_WITH(extra_macros, [  --with-extra_macros=MACRO extra macro defination], [with_extra_macros="$withval"], [with_extra_macros="no"])
if test "x$with_extra_macros" != "xno"; then
    CFLAGS="$CFLAGS ${with_extra_macros}"
    CXXFLAGS="$CXXFLAGS ${with_extra_macros}"
fi

AC_ARG_WITH(extra_header_path, [  --with-extra_header_path=DIR extra include directory], [with_extra_header_path="$withval"], [with_extra_header_path="no"])
if test "x$with_extra_header_path" != "xno"; then
    CPPFLAGS="$CPPFLAGS ${with_extra_header_path}"
fi

AC_ARG_ENABLE(debug, [--enable-debug Enable debugging options (bugreports and developers only)], enable_debug="$enableval", enable_debug="no")
if test "x$enable_debug" = "xyes"; then
    CFLAGS="$CFLAGS -g -O0"
    CXXFLAGS="$CXXFLAGS -g -O0"
else
    CFLAGS="$CFLAGS -O3 -Wall -fmessage-length=0"
    CXXFLAGS="$CXXFLAGS -O3 -Wall -fmessage-length=0"
fi

AC_CONFIG_FILES([Makefile
                 src/Makefile])
AC_OUTPUT
```
对比可以发现，本例作了如下修改：

1. 修改AC_INIT宏，为其设置了参数；

2. 添加AM_INIT_AUTOMAKE宏；

3. 添加AC_CONFIG_MACRO_DIR([m4])宏，为了使用m4；

4. 添加AC_PROG_LIBTOOL宏，测试libtool工具；

5. 添加自定义参数宏AC_ARG_WITH和AC_ARG_ENABLE

    ./configure的自定义参数有两种，一种是开关式(--enable-XXX或--disable-XXX)，另一种是开放式，即后面要填入一串字符(--with-XXX=yyyy)参数。用宏AC_ARG_WITH如上定义。上述自定义参数代码中，第一个参数是参数名，第二个是说明(执行"./configure --help"后所显示出来的内容)，最后一个参数是默认值。一般来说默认值和用户提示应该是互斥的，即默认值是no的话，应提示用户用enable进行修改，反之亦然。
