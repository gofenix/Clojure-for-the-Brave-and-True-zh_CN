# 构建、运行和 REPL

在本章中，你将预先投入少量时间来熟悉建立和运行 Clojure 程序的快速、傻瓜式方法。让一个真正的程序运行起来感觉很好。达到了这个里程碑，你就可以自由地进行实验，分享你的工作，并向那些仍在使用上个世纪的语言的同事幸灾乐祸。这将有助于保持你的积极性!

你还将学习如何使用\*Read-Eval-Print Loop（REPL）\*在一个正在运行的 Clojure 进程中即时运行代码，这使你能够快速测试你对语言的理解并更有效地学习。

但首先，我将简要地介绍 Clojure。接下来，我将介绍 Leiningen，这是 Clojure 事实上的标准构建工具。在本章结束时，你将知道如何做以下事情。

* 用 Leiningen 创建一个新的 Clojure 项目
* 构建该项目以创建一个可执行的 JAR 文件
* 执行 JAR 文件
* 在 Clojure REPL 中执行代码

## 第一重要的事: 什么是 Clojure

Clojure 是由 Rich Hickey 在一座神话般的火山中铸造的。他使用 Lisp、函数式编程和他自己的一绺史诗般的头发的合金，创造了一种令人愉快而强大的语言。它的 Lisp 遗产使你有能力写出比大多数非 Lisp 语言更有表现力的代码，而它对函数式编程的独特理解将使你作为一个程序员的思维更敏锐。此外，Clojure 为你提供了更好的工具来处理复杂的领域（如并发编程），这些领域在传统上被认为会使开发人员陷入多年的治疗中。

不过，在谈论 Clojure 时，重要的是要牢记 Clojure 语言和 Clojure 编译器之间的区别。Clojure 语言是一种强调函数的 Lisp 方言，其语法和语义与任何实现都无关。编译器是一个可执行的 JAR 文件，_clojure.jar_，它接收用 Clojure 语言编写的代码并将其编译为 Java 虚拟机（JVM）字节码。你会看到_Clojure_被用来指代语言和编译器，如果你不知道它们是独立的东西，就会感到困惑。但现在你意识到了，你就会好起来。

这种区分是必要的，因为与大多数编程语言如 Ruby、Python、C 和其他许多语言不同，Clojure 是一种_托管语言_。Clojure 程序在 JVM 中执行，并依赖 JVM 的核心功能，如线程和垃圾收集。Clojure 还针对 JavaScript 和微软的通用语言运行时（CLR），但本书只关注 JVM 的实现。

稍后我们将更多地探讨 Clojure 和 JVM 之间的关系，但现在你需要了解的主要概念是这些。

* JVM 进程执行 Java 字节码。
* 通常情况下，Java 编译器从 Java 源代码中产生 Java 字节码。
* JAR 文件是 Java 字节码的集合。
* Java 程序通常以 JAR 文件分发。
* Java 程序_clojure.jar_ 读取 Clojure 源代码并产生 Java 字节码。
* 然后，该 Java 字节码由已经运行_clojure.jar_的 JVM 进程执行。

Clojure 持续在发展。截至目前，Clojure 的版本为 1.9.0，开发工作仍在进行中。如果你在遥远的未来读到这本书，并且 Clojure 有更高的版本号，不要担心！这本书涵盖了 Clojure 的所有内容。本书涵盖了 Clojure 的基础知识，这些知识在不同的版本中应该不会改变。没有必要让你的机器人管家把这本书退回给书店。

现在你知道什么是 Clojure 了，让我们来实际构建一个该死的 Clojure 程序吧!

```clojure
lein new app clojure-noob
```

这个命令应该创建一个与此相似的目录结构（如果有一些差异也没关系）。

```clojure
| .gitignore
| doc
| | intro.md
➊ | project.clj
| README.md
➋ | resources
| src
| | clojure_noob
➌ | | | core.clj
➍ | test
| | clojure_noob
| | | core_test.clj
```

这个项目骨架本身并不特别，也不像 Clojure 那样。它只是 Leiningen 使用的一种惯例。你将使用 Leiningen 来构建和运行 Clojure 应用程序，Leiningen 希望你的应用程序有这种结构。第一个需要注意的文件是位于➊的_project.clj_，它是 Leiningen 的一个配置文件。它可以帮助 Leiningen 回答这样的问题："这个项目有什么依赖？"和 "当这个 Clojure 程序运行时，什么函数应该先运行？" 一般来说，你会把你的源代码保存在_src/\<project\_name>_。在这种情况下，位于➌的_src/clojure\_noob/core.clj_文件就是你要写一段时间的 Clojure 代码的地方。位于➍的_test_目录显然包含了测试，而位于➋的_resources_是你存储图片等资产的地方。

### 运行 Clojure 项目

现在让我们来实际运行这个项目。在你喜欢的编辑器中打开_src/clojure\_noob/core.clj_。你应该看到这个。

```clojure
➊ (ns clojure-noob.core
  (:gen-class))

➋ (defn -main
  "I don't do a whole lot...yet."
  [& args]
➌   (println "Hello, World!"))
```

➊处的行声明了一个命名空间，你现在不需要担心这个问题。➋处的`-main'函数是你的程序的*入口，这个话题将在附录A中介绍。现在，将➌处的`"Hello, World!"`改为`"I'm a little teapot!"`。全行应该是`(println "I'm a little teapot!"))\`。

接下来，在你的终端导航到_clojure\_noob_目录，然后输入。

```clojure
lein run
```

![](https://www.braveclojure.com/assets/images/cftbat/getting-started/teapot.png)

你应该看到输出`"I'm a little teapot!"`恭喜你，小茶壶，你编写并执行了一个程序！"。

当你阅读本书时，你会了解到更多关于程序中实际发生的事情，但现在你需要知道的是，你创建了一个函数，`-main`，当你在命令行执行`lein run`时，这个函数就会运行。

### 构建 Clojure 项目

使用`lein run`对于尝试你的代码是很好的，但是如果你想与没有安装 Leiningen 的人分享你的工作，该怎么办？要做到这一点，你可以创建一个独立的文件，任何安装了 Java 的人（基本上就是所有人）都可以执行。要创建这个文件，请运行以下程序。

```clojure
lein uberjar
```

这个命令创建了_target/uberjar/clojure-noob-0.1.0\*\*-SNAPSHOT-standalone.jar_文件。你可以通过运行这个命令使 Java 执行它。

```clojure
java -jar target/uberjar/clojure-noob-0.1.0-SNAPSHOT-standalone.jar
```

看看这个! 文件_target/uberjar/clojure-noob-0.1.0-SNAPSHOT\*\*-standalone.jar_是你的 Clojure 程序，你可以在几乎任何平台上发布和运行。

现在你已经掌握了构建、运行和分发（非常）基本的 Clojure 程序所需的所有基本细节。在后面的章节中，你会了解到更多的细节，当你运行前面的命令时，Leiningen 在做什么，对 Clojure 与 JVM 的关系以及你如何运行生产代码有了完整的了解。

在我们进入第二章，讨论 Emacs 的神奇和荣耀之前，让我们来看看另一个重要的工具：REPL。

### 使用 REPL

REPL 是一个用于试验代码的工具。它允许你与正在运行的程序进行交互，并快速尝试各种想法。它通过向你提供一个提示，让你输入代码来实现这一目的。然后，它_读取_你的输入，_求值_它，_打印_结果，并_循环_，再次向你提供提示。

这个过程实现了一个快速的反馈循环，这在大多数其他语言中是不可能的。我强烈建议你经常使用它，因为你能够在学习过程中快速检查你对 Clojure 的理解。除此之外，REPL 开发是 Lisp 体验的一个重要部分，如果你不使用它，你就真的错过了。

要启动一个 REPL，请运行以下程序。

```clojure
lein repl
```

输出应该是这样的。

```clojure
nREPL server started on port 28925
REPL-y 0.1.10
Clojure 1.9.0
    Exit: Control+D or (exit) or (quit)
Commands: (user/help)
    Docs: (doc function-name-here)
          (find-doc "part-of-name-here")
  Source: (source function-name-here)
          (user/sourcery function-name-here)
 Javadoc: (javadoc java-object-or-class-here)
Examples from clojuredocs.org: [clojuredocs or cdoc]
          (user/clojuredocs name-here)
          (user/clojuredocs "ns-here" "name-here")
clojure-noob.core=>
```

最后一行，`clojure-noob.core=>`，告诉你，你在`clojure -noob.core`命名空间中。你将在后面学习命名空间，但现在注意到命名空间基本上与你的_src/**clojure\_noob**/core.clj_文件的名称一致。另外，注意到 REPL 显示的版本是_Clojure 1.9.0_，但如前所述，无论你使用哪个版本，一切都可以正常工作。

该提示还表明你的代码在 REPL 中被加载，你可以执行被定义的函数。现在只有一个函数，`-main`，被定义了。现在就去执行它吧。

```clojure
clojure-noob.core=> (-main)
I'm a little teapot!
nil
```

干得好! 你刚刚使用 REPL 求值了一个函数调用。试试几个更基本的 Clojure 函数。

```clojure
clojure-noob.core=> (+ 1 2 3 4)
10
clojure-noob.core=> (* 1 2 3 4)
24
clojure-noob.core=> (first [1 2 3 4])
1
```

真棒! 你加了一些数字，乘了一些数字，并从一个 Vector 中取出了第一个元素。你还第一次接触到了奇怪的 Lisp 语法! 所有的 Lisp，包括 Clojure，都采用_前缀符号_，这意味着运算符在表达式中总是排在第一位。如果你不确定这意味着什么，不要担心。你很快就会了解到 Clojure 的所有语法。

从概念上讲，REPL 类似于 SSH。就像你可以使用 SSH 与远程服务器交互一样，Clojure REPL 允许你与正在运行的 Clojure 进程交互。这项功能可以非常强大，因为你甚至可以将 REPL 附加到一个实时生产的应用程序，并在它运行时修改你的程序。不过现在，你将使用 REPL 来建立你对 Clojure 语法和语义的了解。

还有一点要注意：今后，本书将介绍没有 REPL 提示的代码，但请尝试一下这些代码! 下面是一个例子。

```clojure
(do (println "no prompt here!")
   (+ 1 3))
; => no prompt here!
; => 4
```

当你看到这样的代码片段时，以`；=>`开头的行表示正在运行的代码的输出。在这种情况下，应该打印出`这里没有提示'的文字，代码的返回值是`4'。

## Clojure 编辑器

到此为止，你应该已经具备了开始学习 Clojure 语言所需的基本知识，而不需要对编辑器或集成开发环境（IDE）大费周章。但如果你确实想要一个关于强大编辑器的好教程，第 2 章涉及 Emacs，这是 Clojurists 中最受欢迎的编辑器。你绝对不需要在 Clojure 开发中使用 Emacs，但 Emacs 提供了与 Clojure REPL 的紧密集成，非常适合编写 Lisp 代码。然而，最重要的是，你要使用适合你的东西。

如果 Emacs 不是你的那杯茶，这里有一些为 Clojure 开发设置其他文本编辑器和 IDE 的资源。

* 这个 YouTube 视频将告诉你如何为 Clojure 开发设置 Sublime Text 2。  [_- YouTube_](http://www.youtube.com/watch?v=wBl0rYXQdGg/)。
* Vim 有很好的工具用于 Clojure 开发。这篇文章是一个很好的起点。 [_Writing Clojure With Vim In 2013 - mybuddymichael.com_](http://mybuddymichael.com/writings/writing-clojure-with-vim-in-2013.html)。
* Counterclockwise 是一个强烈推荐的 Eclipse 插件。[_GoogleCodeHome - ccw-ide/ccw Wiki - GitHub_](https://github.com/laurentpetit/ccw/wiki/GoogleCodeHome)。
* Cursive Clojure 是推荐给那些使用 IntelliJ 的 IDE： [_https://cursiveclojure.com/_](https://cursiveclojure.com)。
* Nightcode 是一个用 Clojure 编写的简单、免费的 IDE。 [_GitHub - oakes/Nightcode: An IDE for Clojure_](https://github.com/oakes/Nightcode/)。

## 总结

我真为你感到骄傲，小茶壶。你已经运行了你的第一个 Clojure 程序! 不仅如此，你还熟悉了 REPL，这是开发 Clojure 软件的最重要工具之一。太神奇了! 这让我想起了我个人英雄之一的《Long Live》中的不朽名句。

> You held your head like a hero\
> On a history book page\
> It was the end of a decade\
> But the start of an age\
> —Taylor Swift

好样的!
