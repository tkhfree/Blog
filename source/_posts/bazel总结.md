---
title: bazel总结
toc: true
thumbnail: 'https://pic.downk.cc/item/5eda03efc2a9a83be5241038.jpg'
comments: true
tags:
  - ONOS
abbrlink: 9598c260
date: 2020-12-25 10:24:13
urlname:
categories:
---

## bazel是用来干啥的

bazel是一个项目的构建工具，像make、maven等，通过编写配置文件等，一键构建整个工程项目目录。

为什么需要构建工具？

我们普通编程的时候，代码量很小，我们可以使用简单的命令行javac、gcc或者在IDE里一键运行，这些编译器可以自动寻找目录下的文件，进行组合，编译成二进制可执行文件。但是如果一些共享库并不在当前目录下，或者需要多种编程语言混合编程的话，并没有一种编译器可以直接编译这种项目。当我们处理多个编程语言或者多个模块时，编程就不是一蹴而就的。我们需要思考代码之间的依赖关系，调整他们在编译器中的位置，如果一旦修改一部分代码，那么依赖他的二进制执行代码都会失效，需要重复整个过程。我们将一些依赖的库，放在诸如lib的文件夹下，时间长了我们很难记住哪些库的作用是什么。

可以用脚本执行构建顺序么？

像t4p4s这种项目，就是使用脚本调整编译器的参数进行构建。但是随着每次更改代码，都需要思考脚本的执行，有一些不需要重复构建的工作通过脚本进行判断也比较困难。

## bazel的文件管理架构

bazel的文件架构有两个：workspace和package

workspace是表示整个项目，也叫repo，项目的根目录下创建WORKSPACE文件定义项目根目录，忽略项目子目录下的WORKSPACE文件

package是项目中的模块，也就是一个个包，包比较随意，可以根据项目需求来定，你想让哪个文件夹下东西成为一个包，就在文件夹目录创建一个BUILD文件，包的管理包括子目录里的东西，不包括子包所包括的内容

一个包路径写全：

```java
@[project_name]//[path/to/package]
```

例如

```java
.../project/
	WORKSPACE
	lib/
		BUILD
		...
	src/
		BUILD
		...
		other1/
			BUILD
		other2/
			...
```

lib、src、other1都是包，写全路径`@project//src/other1`

- workspace 工作空间，每个工作空间中由一个`WORKSPACE`文件，来描述工作空间所使用到的信息。
- package 程序包，每个程序包中包含一个`BUILD`文件，此文件中描述了此工具包的生成构建方式。
- target 目标，生成的目标，每个target又可以作为另外一个规则的输入。
  绝大部分的target属于两种基本类型中的一种，`file`和`rule`。另外，还有一种其他的target类型，package group。但是他们很少见。

![](https://pic.downk.cc/item/5fe9b8913ffa7d37b301beb6.jpg)



## bazel的运行架构

bazel自动化构建代码分为三个步骤：

- 加载阶段：load phase
- 分析阶段：analysis phase
- 执行阶段：execution phase

加载阶段是执行BUILD，load等命令类似于c中的预编译，将头文件#includ等拷贝进文件内，和一些宏定义替换。这个阶段就是替换代码，展开代码之类的事。

分析阶段是获得一个完整的BUILD文件，里面是一个个函数调用。在BUILD文件中我们用规则[rules]来描述构建后包的输出。使用包名和规则名称可以唯一的标示这条规则。我们把这两者的结合叫做标签[label]，我们使用标签来描述规则之间的依赖关系。分析阶段就是执行BUILD文件，生成一个依赖关系的无环有向图。

执行阶段进行构建项目。

```java
java_binary(
name = "MyBinary",
srcs = ["MyBinary.java"], deps = [
":mylib", ],
)
java_library(
name = "mylib",
srcs = ["MyLibrary.java", "MyHelper.java"],
visibility = ["//java/com/example/myproduct:__subpackages__"],
deps = [
"//java/com/example/common", "//java/com/example/myproduct/otherlib", "@com_google_common_guava_guava//jar",
], )
```

在Bazel中，BUILD文件定义了targets。上面的两个targets分别是java_binary和java_library. 每个target都对应着bazel能够创建的一种artifact. binary targets生成能够直接运行的二进制文件，library targets生成能够被其它binary或者library所使用的内容。每个target都有一个name（定义它在命令行和其它target中应该如何被引用)，srcs（定义必须被编译的相关源文件以生成对应的target制品），以及deps（定义前置必须先构建或链接的依赖）。依赖关系可以限制在当前package以内（e.g. MyBinary依赖于:mylib），也可以是在同一个源代码层级中的不同package(e.g. mylib依赖于//java/com/example/common)，或者源代码层级之外的第三方artifact(e.g. mylib依赖于"@com_google_common_grava_grava//jar"). 每个源代码层级(source hierarchy)都被称为一个workspace，并由根目录下的一个WORKSPACE文件来标示。

和Ant一样，用户也需要使用bazel的命令行工具来进行构建。为了构建MyBinary这个target，用户需要执行

```java
bazel build :MyBinary
```

在第一次运行上面命令时，Bazel会执行下列工作：

1. 解析当前workspace中每一个BUILD文件，创建各个制品(artifacts)之间的依赖图(graph of dependencies)
2. 使用上面创建的图来决定MyBinary的依赖转换关系(transitive dependencies)，也即是MyBinary所依赖的每个target，及每个target依赖的其它target，以此递归
3. 根据具体定义，按顺序构建或下载每一个依赖。Bazel在这里首先构建没有任何依赖的target，并保持跟踪对于每个target来说还有哪些依赖需要构建。一旦一个target的所有依赖都已经构建好之后，Bazel就开始构建该target. 该过程持续到MyBinary的每一个依赖都被构建完成。
4. 链接所有上一步中所生成的依赖，构建MyBinary来生成最后的可执行二进制文件。

在第二次运行的时候，如果没什么修改，bazel1秒钟之内就会结束--bazel知道每个target都已经是最新的。

## 对工具的依赖

有时候很多构建都依赖于机器上的工具，由于工具版本和安装环境的因素，跨机器构建会有很多问题，如果是多语言编程，会更加困难。

这里牵扯到两个方面：环境依赖和平台针对性

bazel解决第一个问题是把工具作为target依赖的一部分在workspace里配置，bazel构建的时候首先检查工具是否指定版本和是否在指定位置，如果不在则下载。

bazel解决第二个问题时使用工具链，工具链包括一组工具和其他相关属性，定义某个类型target如何在特定平台构建

如何确定一个可靠的外部依赖呢？

如果每次都要从外部第三方下在一些工具和库，如果第三方改变或者挂掉那就不能保证构建一致性。Bazel和其它构建系统通过workspace范围内的清单(manifest)来处理这个问题，这个清单列出了所有外部依赖的加密hash值(cryptographic hash)。

## 处理模块和依赖

**最小化模块可见度**

Bazel和其它构建系统允许每个target都定义自己的可见度(visibility):定义哪些targets可能会依赖自己。Targets可以是public的，这种情况下在workspace中的任何target都可以引用。可以是private，那么只有在同一个BUILD文件中的才能引用。或者也可以只对于一个列表中的targets可见。可见度(Visibility)和dependency是相反的：如果target A需要依赖于target B，那么target B必须是对A可见的。

**内部依赖**

在一个大项目拆分为细分模块后，大部分依赖项都很可能是项目内部的，意即为依赖于另一个在同一个代码仓库中定义和构建的target. 内部依赖和外部依赖的差别在于，他们是直接从源代码构建，而不是在构建过程中从外部作为一个预先构建好的artifact下载。这也意味着对内部构建而言，实际上没有版本的概念--一个target及其所有的内部依赖项都是始终从代码仓库的同一个版本统一构建的。

**外部依赖**

如果一个依赖不是内部的，那么它一定是来自外部。外部依赖指构建和存储在当前构建系统之外的artifacts制品。这样的依赖项被直接从artifact仓库导入(通常通过internet)，并会被直接使用，而不是从代码构建。它和内部依赖最大的一点差异是外部依赖存在版本，并且这些版本独立于项目的源代码存在。