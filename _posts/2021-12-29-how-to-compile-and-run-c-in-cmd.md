---
title: C语言番外：如何通过命令行编译运行C语言程序？
date: 2021-12-29 20:50:00 +0800
categories: [C语言番外]
tags: [C语言番外]
---

为什么要讲一下如何通过命令行编译、运行 C 语言程序呢？书上都会说一个 C 语言从源代码到可执行文件有两个步骤：

1. 源代码编译成目标文件。
2. 目标文件连接成可执行程序。

但是我们在使用各种 IDE（集成开发环境）时，基本都是在菜单里点击一下运行按钮，程序一下子就运行起来了。而且很多小伙伴会以为想要编译 C 语言源码电脑上就必须安装 IDE。实际上不是这样的，要编译 C 语言源文件，电脑上只需要安装一个编译器即可，而代码可以用任何文本编辑工具进行编写，包括记事本。

> 这里说的编译器可不是 Visual C++ 6.0、Visual Studio、Visual Code，这些统统都是 IDE。编译器比 IDE 小多了。

一个比较常见的 C 语言编译器叫 gcc（GNU Compiler Collection，GNU编译器套件），本文就简单介绍一下使用 gcc 编译 C 语言程序。gcc 在各种操作系统上都能安装，因为大多数读者可能是 Windows 用户，所以这里以 Windows 系统做示例。Windows 版本的 gcc 又叫 MinGW，可以[点此下载安装包](http://sourceforge.net/projects/mingw-w64/files/Toolchains%20targetting%20Win32/Personal%20Builds/mingw-builds/installer/mingw-w64-install.exe/download)。

> 如果你是 Mac 用户，只要你下载安装 Xcode 后，电脑上就有 gcc 了。但是如果你嫌 Xcode 太大的话，可以只下载`Xcode Command Line Tools`即可。但是我还是建议安装完整的 Xcode，因为本人就是一个 Xcode 用户，Xcode 绝对比大多数 IDE 优秀了。同样的，如果你在 Linux 上，只需要安装 Linux 版本的 gcc 即可（因 Linux 发行版本较多，不同版本之间安装软件的方法也有区别，读者可根据自己使用的 Linux 发行版本搜索对应的安装方法）。

安装好 gcc 后，我们就可以在命令行中编译、连接、运行 C 语言程序了，就以最简单的“Hello World”程序为例，代码保存在名为“main.c”的源文件中，这个文件存放在桌面上的C文件夹中。

> 本文示例中的命令在 Windows/Linux/macOS 上通用。

1. `cd`到目标文件夹

```shell
C:\Users\scfhao>cd Desktop/C
```

代码文件不一定要放在这个文件夹，但是在执行编译操作前需要`cd`到对应的文件夹中。

2. 编译 main.c

```shell
gcc -c main.c -o main.o
```

解释一下这个命令：像`-c`这种以`-`开头的叫“选项”，在这个地方`-c`代表让 gcc 执行编译操作（c就是compile的缩写），后面的 main.c 就是`-c`选项的参数，也就是要编译的文件名。`-o`选项中 o 是 out 的缩写，`-o`的参数 main.o 用于指定编译输出文件的文件名。所以执行这个编译命令后，当前文件夹会多出一个文件名为`main.o`的目标文件。

3. 连接生成可执行程序

```shell
gcc main.o -o main.exe
```

这一步就是所谓的连接操作了，同样`-o`选项用于指定输出的可执行文件的文件名。

4. 运行可执行程序

```shell
main.exe
```

直接输入可执行程序的文件名并按下回车键，上一步生成的可执行文件就运行了，控制台就会输出“Hello World”。

通过这篇教程，C 语言程序的编译、连接这个过程有没有变的清晰起来呢？当然这里只是以最简单的 Hello World 为例，在实际项目中，我们的程序不可能只有一个 c 文件，编译时我们需要对每个文件进行单独的编译，然后把众多编译生成的目标文件连接成一个可执行程序。是不是觉得有点复杂了？关注“计科辅导员”，在后续的文章中将会对这种复杂的情况进行讲解。下一期预告：main 函数为什么要`return 0;`？

