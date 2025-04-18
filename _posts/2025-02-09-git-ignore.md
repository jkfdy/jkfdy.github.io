---
title: 面向新手的Git教程：Git 忽略文件
date: 2025-02-09 11:00:00 +0800
categories: [Git]
tags: [Git, GitHub]
---

上回，我们一起学习了 Git 工作区中文件的状态以及在各 Git 操作下文件状态如何变化。我们知道，工作区中的文件从大方向来说可以分为未跟踪文件和已跟踪文件。新建文件后，Git 并不会替我们决定是否跟踪该文件，我们需要使用`git add`明确告诉 Git 需要跟踪该文件。

但在大多数的情况下，我们不会只用`git add`单个文件，而是通过在项目根目录中执行：

```bash
git add .
```

> `.`代表当前目录，这个命令会把已跟踪的处于非干净状态的文件一次性全部加入暂存区，同时也把未明确告诉 Git 不需要跟踪的未跟踪文件也加入暂存区。

这样是不是方便很多？但同时又有了一个问题，如何明确告诉 Git 不需要跟踪哪些文件呢？答案就是使用`.gitignore`文件。

## 不需要被 Git 跟踪的文件

如何使用`.gitignore`文件我们稍后再说，先说一下哪些文件不需要 git 跟踪。这就有点多了，主要有这么几类：

* 操作系统相关的，比如在 macOS 上，每个文件夹中都会自动生成一个`.DS_Store`隐藏文件，这个文件是不需要让 Git 跟踪的
* 语言相关的，在编译型编程语言的项目里，通常在编译时会生成一些临时文件。比如 Java 中的`.class`文件。还有就是项目最终打包后的文件也不需要被跟踪
* 项目依赖的软件包，像 NodeJS 中的`/node_modules`目录就不需要被跟踪。但这个也不绝对，有的项目也需要 Git 跟踪依赖包
* IDE 相关的，基本上每个 IDE 像 JetBrains、VS Code、Xcode 都会在项目目录中生成一些临时文件，用来保存用户在本项目中在 IDE 中的配置

## 如何使用`.gitignore`

在项目根目录中创建名为`.gitignore`的文件，可以在其中定义哪些文件不需要跟踪的规则，后面称此文件为`忽略文件`。忽略文件的格式规范如下：

* 所有空行或者以`#`开头的行都会被 Git 忽略
* 可以使用标准的 glob 模式匹配，它会递归地应用在整个工作区中
* 匹配模式可以以（`/`）开头防止递归
* 匹配模式可以以（`/`）结尾指定目录
* 要忽略指定模式以外的文件或目录，可以在模式前加上叹号（`!`）取反

这里的 glob 模式匹配就是在命令行中支持的简化了的正则表达式，其匹配规则为：

* 星号（`*`）匹配零个或多个任意字符
* `[abc]`匹配任何一个列在方括号中的字符 （这个例子要么匹配一个 a，要么匹配一个 b，要么匹配一个 c）
* 问号（`?`）只匹配一个任意字符
* 如果在方括号中使用短划线分隔两个字符， 表示所有在这两个字符范围内的都可以匹配（比如`[0-9]`表示匹配所有 0 到 9 的数字）
* 使用两个星号（`**`）表示匹配任意中间目录，比如`a/**/z`可以匹配`a/z`、`a/b/z`或`a/b/c/z`等

这是一个忽略文件示例：

```bash
# 忽略所有的 .a 文件
*.a

# 但跟踪所有的 lib.a，即便你在前面忽略了 .a 文件
!lib.a

# 只忽略当前目录下的 TODO 文件，而不忽略 subdir/TODO
/TODO

# 忽略任何目录下名为 build 的文件夹
build/

# 忽略 doc/notes.txt，但不忽略 doc/server/arch.txt
doc/*.txt

# 忽略 doc/ 目录及其所有子目录下的 .pdf 文件
doc/**/*.pdf
```

## 在线生成忽略文件

忽略文件只需要能看懂即可，没必要自己写，因为现在有很方便的工具网站可以在线生成忽略文件。比如：`gitignore.io`这个网站。

![gitignore.io](/assets/img/posts/2025/gitignore1.png)

打开网页后，可以看到一个输入框和一个创建按钮，在输入框中输入我们的环境后，比如我这里输入了`macOS`、`JetBrains`、`Java`后，点击创建按钮，网页就会跳转到生成的忽略文件，只需要把当前这个文件保存到我们的项目根目录中，然后重命名为`.gitignore`即可。

![gitignore](/assets/img/posts/2025/gitignore2.png)

当然更简单的方式是使用命令行，在项目根目录中执行：

```
curl https://www.toptal.com/developers/gitignore?templates=macos,jetbrains,java -o .gitignore
```

即可把忽略文件下载到本地。

还有一个办法，那就是借助 AI，告诉 AI 你想忽略什么文件，让 AI 帮你生成`.gitignore`文件。

## 后悔药

在初始化`git`仓库后就立刻生成忽略文件是非常好的习惯。如果你没有这样做就很容易把不该跟踪的文件纳入 Git 管理，事后发现提交了一堆没用的文件，这个时候突才想起来创建`.gitignore`文件，就有点亡羊补牢的意思了。记住，如果 Git 已经开始跟踪一个文件，即便你事后在`.gitignore`文件中说明需要忽略此文件，Git 也不会主动帮你取消跟踪此文件的。这种情况应该怎么办呢？

```bash
git rm --cached demo.txt
git rm -r --cached /node_modules
```

如果要取消跟踪的是文件夹，需要添加`-r`选项，表示递归此文件夹下的内容。

一定要记得`--cached`选项，如果没有这个选项，Git 在取消跟踪文件时会同时把工作区域的对应文件删除，但很多情况下，我们并不想删除这些文件。

> 因本人能力有限，错误之处在所难免，如有问题，请在评论区不吝赐教。另因公众平台无法对历史文章重新编辑，可点击查看原文查看最新版本。
