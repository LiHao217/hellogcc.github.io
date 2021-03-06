转换时间： 2021-01-31

# 翻译Clang文档：Choosing the Right Interface for Your Application
Posted on 2013/12/20 by shining


## 为你的程序选择正确的接口



Clang为实现需要一个程序的语法和语义信息的工具提供了基础设施。（（译者注：为了便于理解，再意译一句）如果你想实现一个工具，而这个工具需要获取 一个程序的语法和语义信息，那么恭喜你，Clang可以为你提供一些基础实现。）这个文档将给出使用不同的方法去实现基于Clang的工具的一个简短介 绍，包括它们的优点和缺点。

## LibClang

LibClang是一个稳定的高层次的Clang的C语言接口。如果不确定LibClang是你想要用的接口，只有当你有一个好的理由不用LibClang的时候，这个时候你才可以去用别的接口。

如果遇到如下情况，那么就是典型的使用LibClang的时候：（译者注：Xcode目前没搞清楚，暂时保留）：

-    Xcode
-    Clang Python Bindings

如果遇到下列情况，请使用LibClang:

-    想使用Clang除了C++之外的编程语言的接口
-    需要一个稳定的接口可以向后兼容
-    需要一个十分强大的高层次的抽象，比如：像一个光标一样遍历AST，或者不想去学习Clang的AST的所有的细节问题

如果遇到下列情况，请不要使用LibClang:

-    想完全控制Clang AST


## Clang Plugins



Clang Plugins允许你在AST之上运行一些额外的动作，而这些动作作为编译的一部分。Plugins是运行的时候被编译器加载的动态库，她们很容易合并到你的构建环境中。

如果遇到如下情况，那么就是典型的使用Clang Plugins的时候：

-    你的工程有特殊的lint风格的警告或者错误
-    从一个单独的编译步骤要创建额外的构建神器

如果遇到下列情况，请使用Clang Plugins:

-    需要你的工具去返回任何依赖关系的变化
-    想让你的工具去执行或者跳出一次构建
-    需要完全控制Clang AST

如果遇到下列情况，请不要使用Clang Plugins:

-    想在你的构建环境的外部运行工具
-    想完全控制Clang如何建立起来，包括内存虚文件的映射
-    想去运行你工程中的一部分特殊的文件，而这部分文件和触发重新构建是没有必要联系的

## LibTooling
LibTooling是一个C++接口，它的目标在于实现完全独立的工具，而这些工具就像是被集成到运行Clang工具的服务中了一样。
如果遇到如下情况，那么就是典型的使用LibTooling的时候：

-    一个简易的语法检查器
-    重构工具

如果遇到下列情况，请使用LibTooling:

-    想在独立于构建系统的单个文件或者一系列特殊的文件上运行工具
-    想完全控制Clang AST
-    想和Clang Plugins共享代码

如果遇到下列情况，请不要使用LibTooling:

-    想作为可以被依赖关系改变而触发的构建系统的一部分运行
-    想要一个稳定的接口，不需要因为AST API的改变而改变你的代码
-    想要一个高层次的抽象像光标和在盒子之外完成的代码
-    不想使用C++去实现你的工具

Clang tools是一系列基于LibTooling架构基础之上构建的特殊的开发者工具，它们是Clang工程的一部分。它们的目标是自动化和改善C/C++开发者的核心开发活动。

我们已经在构建或者计划构建的作为Clang工程的一部分例子工具有：

-    语法检查 (clang-check)
-    自动修复编译错误 (clang-fixit)
-    自动代码格式化 (clang-format)
-    新语言标准的新特性的迁移工具（cpp11-migrate，译者注）
-    核心重构工具

（完）
