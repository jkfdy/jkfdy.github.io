---
title: C语言番外：main 函数为什么要`return 0;`？
date: 2021-12-30 22:17:00 +0800
categories: [C语言番外]
tags: [C语言番外]
---

在上一个篇公众号文章中，我们在命令行窗口中分别输入了三条命令来对 C 语言源文件进行编译、连接、运行操作。那我们可以一次输入三个命令吗？因为我不想等一个命令结束后再输入下一个命令，答案当然是可以：

```shell
gcc -c main.c -o main.o && gcc main.o -o main.exe && main.exe
```

如果是在 Linux 或 macOS 等非 Windows 系统上，上面的命令会报错，需要改成下面的格式：

```shell
gcc -c main.c -o main.o && gcc main.o -o main.exe && ./main.exe
```

> 只有最后运行 main.exe 的部分有差异，另外“exe”只是 Windows 系统上可执行程序的后缀名，在 Unix/Linux/macOS 等系统中，文件后缀都是无关紧要的，就算把可执行程序的后缀改成“txt”也照样可以运行，当然非要加“exe”肯定也可以。按照惯例，在这类系统上可执行文件不写后缀名。在这里我们先忽略不同操作系统之间的差异。

可以看到，可以使用“&&”把多个命令进行连接，这样写的作用是如果前一个命令执行成功会接着执行后面的命令，但是如果前面的命令执行失败，后面的命令就不会继续执行了。所以我们上面的整个命令的作用就是先编译 main.c 文件，如果编译成功，接着就连接，如果连接成功，就运行，只要有一个步骤失败，后面的步骤就不需要执行了。

那这里就有一个问题，系统是如何知道前一个步骤是否成功了呢？其实 gcc 也是一个命令行程序，而且同样是用 C 语言编写的，所以 gcc 和我们上课写的 C 语言程序都属于命令行程序。所以 gcc 也有 main 函数，gcc 的 main 函数在最后也会返回一个 int 型返回值。

按照惯例，main 函数的返回值也被叫做退出码（exit code）。顾名思义，就是程序在退出运行的时候返回给操作系统的一个状态码。操作系统就是根据这个退出码来判断这个程序是否执行“成功”的，在 POSIX 系统上退出码的标准为用 0 代表成功，用 1 到 255 之间的其他数字代表失败。

> 这里提到的 POSIX 是操作系统的一个标准，UNIX/Linux 都是遵循这个标准的，Windows 也在朝这个标准靠拢。

那这里就有个问题，1 到 255 之间有这么多数字，该用哪个数字呢？其实 C 语言在`stdlib.h`和`sysexits.h`两个头文件中对此有定义。下面分别是从这两个头文件中的相关代码：

```c
// stdlib.h
#define EXIT_FAILURE 1
#define EXIT_SUCCESS 0
```

```c
// sysexits.h
#define EX_OK  0 /* successful termination */

#define EX__BASE 64 /* base value for error messages */

#define EX_USAGE 64 /* command line usage error */
#define EX_DATAERR 65 /* data format error */
#define EX_NOINPUT 66 /* cannot open input */
#define EX_NOUSER 67 /* addressee unknown */
#define EX_NOHOST 68 /* host name unknown */
#define EX_UNAVAILABLE 69 /* service unavailable */
#define EX_SOFTWARE 70 /* internal software error */
#define EX_OSERR 71 /* system error (e.g., can't fork) */
#define EX_OSFILE 72 /* critical OS file missing */
#define EX_CANTCREAT 73 /* can't create (user) output file */
#define EX_IOERR 74 /* input/output error */
#define EX_TEMPFAIL 75 /* temp failure; user is invited to retry */
#define EX_PROTOCOL 76 /* remote error in protocol */
#define EX_NOPERM 77 /* permission denied */
#define EX_CONFIG 78 /* configuration error */

#define EX__MAX 78 /* maximum listed value */
```

每个数字后面注释中的描述就是这个退出码代表的含义，对于通常的错误，用`EXIT_FAILURE`也就是 1 即可。

另外，main 函数最后不一定必须`return 0`，也可以这样写：

```c
#include <stdio.h>
#include <stdlib.h>

int main() {
    printf("Hello World!\n");
    exit(EXIT_SUCCESS);
}
```

最后，我们在命令行窗口运行一个程序后，如何查看这个程序的退出码是多少呢？


```shell
# Windows
echo %errorlevel%

# Unix/Linux/macOS
echo $?
```

等等，我记得`main`函数的返回值类型还可以是`void`，如果是`void`就是不需要写`return 0;`了，那这种情况下，退出码又会是多少呢？

其实像这位同学这个问题呢，自己写个程序试一下就知道了。我就写了一个 main 函数返回值类型是`void`的程序，在 macOS 上运行了一下，查看到的返回值是 13，但我没在 Windows 上试，有哪位同学留言把 Windows 上对应的值写出来的吗？

好了，这篇公众号我们讲了 main 函数返回值的作用以及使用方法。下期预告：main 函数可以有参数吗？
