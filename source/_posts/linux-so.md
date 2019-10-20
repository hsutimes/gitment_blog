---
title: linux_so
tags:
  - linux
  - c++
  - c
originContent: ''
categories:
  - 文章
toc: false
date: 2019-10-14 20:46:29
---

# Linux下的静态库、动态库和动态加载库

## Linux库类型

Linux下可以创建两种类型的库:

1. 静态库(.a): 在链接期间被应用程序直接链接进可执行文件

2. 动态链接库(.so): 动态库还分为两种用法: a) 应用程序运行期间链接动态库，但是在编译期间声明动态库的存在，也就是说这种动态库必须在编译时对编译器可见，但编译器却不将此种库编译进可执行文件; b) 在运行期间，动态加载和卸载的库，使用动态加载方法加载。这种库的形式跟动态链接没有本质区别，区别是在调用时，是由用户程序决定何时链接的，而不是由系统链接器自动链接

## 命名约定

库需要以`lib`作为开头，而在指定链接命令行参数时，却无需包含开头和扩展名，例如：
```shell
gcc src-file.c -lm -lpthread
```
这个例子中，链接了`libmath.a`和`libpthread.a`

## 静态库(.a)
生成静态库的方法如下：

- 编译object文件。例如：cc -Wall -c ctest1.c ctest2.c，该命令会生成ctest1.o和ctest2.o(其中-Wall表示编译时输出警告)。

- 创建库文件。例如：ar -cvq libctest.a ctest1.o ctest2.o。该命令会得到一个libctest.a文件

- 可以通过ar -t查看.a文件中包含哪些.o。所以，实际上ar就是一个打包命令，类似tar

- 构建符号表。ranlib libctest.a用于为.a创建符号表。有些ar命令实际上已经集成了ranlib的功能

`.a`文件与windows下的`.lib`是相同的概念。

## 动态库(.so)
生成动态库的方法如下：

- 编译object文件时使用-fPIC选项：

```shell
gcc -Wall -fPIC -c *.c
```

这个选项的目的是让编译器生成地址无关(position independent)的代码，这是因为，动态库是在运行期间链接的，变量和函数的偏移量是事先不知道的，需要链接以后根据offset进行地址重定向。

- 使用-shared链接
```shell
gcc -shared -Wl,-soname,libctest.so.1 -o libctest.so.1.0 *.o
```

`-shared`选项是让动态库得以在运行期间被动态链接;`-Wl,options`是设置传递给`ld(链接器)`的参数，在上面的例子中，当链接器在链接`.o`时会执行`ld -soname ibctest.so.1`

- 创建软链

上面的命令将最终输出一个动态库libctest.so.1.0，而出于习惯，会创建两个软链:
```shell
mv libctest.so.1.0 /opt/lib
ln -sf /opt/lib/libctest.so.1.0 /opt/lib/libctest.so.1
ln -sf /opt/lib/libctest.so.1.0 /opt/lib/libctest.so
```

`libctest.so`用于在编译期间使用`-lctest`让编译器找到动态库，而`libctest.so.1`用于在运行期间链接

```shell
gcc -Wall -I/path/to/include-files -L/path/to/libraries prog.c -lctest -o prog
```shell

### 查看依赖
使用ldd命令来查看程序对动态库的依赖。例如：

```shell
ldd prog

libctest.so.1 => /opt/lib/libctest.so.1 (0x00002aaaaaaac000)
libc.so.6 => /lib64/tls/libc.so.6 (0x0000003aa4e00000)
/lib64/ld-linux-x86-64.so.2 (0x0000003aa4c00000)
```

### obj文件
obj文件的格式和组成可能是系统差异性的一大体现，比如windows下的PE、linux和一些unix下的elf、macos的mach-o、aix下的xcoff。

查看obj文件的符号表信息，可以通过nm objdump readelf等方法。

### 运行期间查找动态库

运行期间，系统需要知道到哪里去查找动态库，这是通过/etc/ld.so.conf配置的。ldconfig用于配置运行时动态库查找路径，实际是更新/etc/ld.so.cache。另外一些环境变量也可以影响查找：(Linux/Solaris: LD_LIBRARY_PATH, SGI: LD_LIBRARYN32_PATH, AIX: LIBPATH, Mac OS X: DYLD_LIBRARY_PATH, HP-UX: SHLIB_PATH)

## 动态加载和卸载的库
需要应用程序希望设计成插件化的架构，这就需要可以动态加载和卸载库的机制。与动态链接不同的是，动态加载的意思是，编译期间可以对动态库的存在一无所知，而是在运行期间通过用户程序尝试加载进来的。

通过`dlfcn.h`中的`dlopen`、`dlsym`和`dlclose`等函数实现此种功能。

另外，使用到dlfcn机制的可执行文件需要使用-rdynamic选项，它将指示连接器把所有符号（而不仅仅只是程序已使用到的外部符号，但不包括静态符号，比如被static修饰的函数）都添加到动态符号表（即.dynsym表）里。

## GNU Libtool
如今许多软件的编译都采用libtool工具，libtool是一个编译链接包装工具，实际只是一个脚本，用libtool编译和链接会产生类似.la的文件，.la这种文件其实是个文本文件，指向.a文件，并声明一些版本信息。


[传送门](https://segmentfault.com/a/1190000005988462)