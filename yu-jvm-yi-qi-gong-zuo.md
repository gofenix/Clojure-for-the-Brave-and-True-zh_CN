# 与 JVM 一起工作

在每个 Clojurist 的生命中都会有这么一天，她必须从纯函数和不可变数据结构的庇护所冒险进入野蛮的 Java 大陆。这段艰难的旅程是必要的，因为 Clojure 是在 Java 虚拟机（JVM）上托管的，这赋予了它三个基本特性。

赋予它三个基本特征。首先，你运行 Clojure 应用程序的方式与你运行 Java 应用程序的方式相同。第二，你需要使用 Java 对象来实现核心功能，如读取文件和处理日期。第三，Java 有一个庞大的有用库的生态系统，你需要对 Java 有一定的了解才能使用它们。

这样一来，Clojure 就有点像一个乌托邦社区，被放置在一个乌托邦国家的中间。显然，你更愿意与其他乌托邦人互动，但偶尔你也需要与当地人交谈，以便完成工作。

这一章就像一本短语书和 Java 国家的文化介绍之间的交叉。你将了解什么是 JVM，它是如何运行程序的，以及如何为它编译程序。本章还将为你简要介绍常用的 Java 类和方法，并解释如何使用 Clojure 与它们互动。你将学会如何思考和理解 Java，以便将任何 Java 库纳入你的 Clojure 程序中。

要运行本章的例子，你需要在电脑上安装 1.6 或更高版本的 Java 开发工具包（JDK）。你可以通过在终端运行`javac -version`来检查。你应该看到类似 "java 1.8.0\_40 "的内容；如果没有，请访问[_http://www.oracle.com/_](http://www.oracle.com/technetwork/java/javase/downloads/index.html)，下载最新的 JDK。

## JVM

开发人员用 JVM 这个词来指代一些不同的东西。你会听到他们说，"Clojure 在_the_ JVM 上运行"，你也会听到，"Clojure 程序在_a_ JVM 中运行"。在第一种情况下，JVM 指的是一个抽象概念--Java 虚拟机的一般模型。在第二种情况下，它指的是一个进程--一个正在运行的程序的实例。我们将专注于 JVM 模型，但当我们谈论运行中的 JVM 进程时，我将指出来。

为了理解 JVM，让我们回头看看普通的计算机是如何工作的。在计算机心脏的深处是它的 CPU，而 CPU 的工作是执行像_加和_无符号乘法这样的操作。你可能听说过程序员将这些指令编码在打卡机上、灯泡里、乌龟壳的神圣缝隙里，或者_什么的，但现在这些操作在汇编语言中用 ADD 和 MUL 这样的记忆符号表示。CPU 架构（X86、ARMv7，等等）决定了哪些操作可以作为该架构的_指令集的一部分。

由于用汇编语言编程并不有趣，人们发明了像 C 和 C++这样的高级语言，将其编译成 CPU 可以理解的指令。大体上说，这个过程是

1. 编译器读取源代码。
2. 编译器输出一个包含机器指令的文件。
3. CPU 执行这些指令。

在图 12-1 中注意到，最终，你必须将程序翻译成 CPU 能够理解的指令，而 CPU 并不关心你用哪种编程语言来产生这些指令。

JVM 类似于计算机，它也需要将代码翻译成低级别的指令，称为_Java 字节码_。然而，作为一个_虚拟_机器，这种翻译是作为软件而不是硬件实现的。运行中的 JVM 通过将字节码实时翻译成主机可以理解的机器代码来执行，这个过程被称为_及时\*\*编译_。

![](https://www.braveclojure.com/assets/images/cftbat/java/compile.png)

图 12-1：C 语言程序如何被翻译成机器码的高级概述

为了让一个程序在 JVM 上运行，它必须被编译成 Java 字节码。通常，当你编译程序时，产生的字节码被保存在一个\*.class_文件中。然后你会把这些文件打包在_Java 归档\*文件（JAR 文件）中。就像 CPU 不关心你用哪种编程语言来生成机器指令一样，JVM 也不关心你如何创建字节码。它不关心你是否使用 Scala、JRuby、Clojure，甚至是 Java 来创建 Java 字节码。一般来说，这个过程就像图 12-2 中所示的那样。

1. Java 编译器读取源代码。
2. 编译器输出字节码，通常是在一个 JAR 文件中。
3. JVM 执行字节码。
4. VM 向 CPU 发送机器指令。

当有人说 Clojure 在 JVM 上运行时，他们的意思之一是 Clojure 程序被编译成 Java 字节码，JVM 进程执行它们。从操作的角度来看，这意味着你对待 Clojure 程序和 Java 程序是一样的。你把它们编译成 JAR 文件，并使用`java`命令运行它们。如果客户需要一个在 JVM 上运行的程序，你可以偷偷地用 Clojure 而不是 Java 来编写，他们不会知道的。从外面看，你无法分辨 Java 和 Clojure 程序之间的区别，就像你无法分辨 C 和 C++程序之间的区别一样。Clojure 可以让你变得富有成效，而且是偷偷摸摸的。

！

图 12-2：Java 程序产生 JVM 字节码，但 JVM 仍然需要产生机器指令，就像 C 语言编译器一样。

## 编写、编译和运行一个 Java 程序

让我们来看看一个真正的 Java 程序是如何工作的。在本节中，你将了解到 Java 所使用的面向对象的范式。然后，你将用 Java 建立一个简单的海盗短语书。这将帮助你对 JVM 感到更加舒适，它将为即将到来的 Java 互操作（编写直接使用 Java 类、对象和方法的 Clojure 代码）一节做好准备，如果有一个恶棍试图在公海上破坏你的战利品，它就会派上用场。为了把所有的信息联系在一起，你将在本章的最后偷看一些 Clojure 的 Java 代码。

### 面向对象的编程在世界最微小的果壳中的应用

Java 是一种面向对象的语言，所以如果你想了解你在 Clojure 编程中使用 Java 库或编写 Java 互操作代码时发生了什么，你就需要了解面向对象编程（OOP）是如何工作的。你也会在 Clojure 文档中发现面向对象的术语，所以学习这些概念很重要。如果你精通 OOP，可以随意跳过本节。对于那些需要两分钟了解的人来说，这里是：OOP 的核心角色是_类_、_对象_和_方法_。

我认为对象是真正的、真正的、可笑的蠢货机器人。它们是那种永远不会引起哲学辩论的机器人，即强迫有知觉的生物进行永久的奴役的伦理。这些机器人只做两件事：他们响应命令和维护数据。在我的想象中，它们通过在小 Hello Kitty 剪贴板上写下东西来做这件事。

想象一下，一个制造这些机器人的工厂。机器人所理解的命令集和它所维护的数据集都是由制造机器人的工厂决定的。在 OOP 术语中，工厂对应于类，androids 对应于对象，而命令对应于方法。例如，你可能有一个`ScaryClown'工厂（类），它生产的androids（对象）响应`makeBalloonArt'命令（方法）。这个安卓机一直跟踪它所拥有的气球的数量，然后在气球的数量发生变化时更新这个数字。它可以用`balloonCount`报告这个数字，用`receiveBalloons`接收任何数量的气球。下面是你如何与代表小丑 Belly Rubs 的 Java 对象进行交互。

```
ScaryClown bellyRubsTheClown = new ScaryClown();
bellyRubsTheClown.balloonCount();
// => 0

bellyRubsTheClown.receiveBalloons(2);
bellyRubsTheClown.balloonCount();
// => 2

bellyRubsTheClown.makeBalloonArt();
// => "Belly Rubs makes a balloon shaped like a clown, because Belly Rubs
// => is trying to scare you and nothing is scarier than clowns."
```

这个例子告诉你如何使用`ScaryClown`类创建一个新的对象`bellyRubsTheClown`。它还向你展示了如何在该对象上调用方法（如`气球计数'、`接收气球'和\`制作气球艺术'），大概是为了让你能吓唬孩子。

你应该知道 OOP 的最后一个方面，或者至少是它在 Java 中的实现方式，就是你也可以向工厂发送命令。在 OOP 术语中，你会说，类也有方法。例如，内置类`Math`有许多类方法，包括`Math.abs`，它返回一个数字的绝对值。

```
Math.abs(-50)
// => 50
```

我希望这些小丑没有给你造成太大的创伤。现在让我们把你的 OOP 知识用在工作上吧!

### Ahoy, World

继续前进，创建一个名为_phrasebook_的新目录。在该目录中，创建一个名为_PiratePhrases.java_的文件，并编写以下内容。

```
public class PiratePhrases
{
    public static void main(String[] args)
    {
        System.out.println("Shiver me timbers!!");
    }
}
```

这个非常简单的程序将在你运行时向你的终端打印 "Shiver me timbers!!!"这句话。(这就是海盗说 "你好，世界！"的方式），当你运行它时，它将打印到你的终端。它由一个类`PiratePhrases`和一个属于该类的静态方法`main`组成。静态方法本质上是类的方法。

在你的终端，用 javac PiratePhrases.java 命令编译`PiratePhrases`源代码。如果你打的字都是正确的，\*\*你的心是纯洁的，你应该看到一个名为_PiratePhrases.class_的文件。

```
$ ls
PiratePhrases.class PiratePhrases.java
```

你刚刚编译了你的第一个 Java 程序，我的朋友! 现在用`java PiratePhrases`运行它。你应该看到这个。

```
Shiver me timbers!!!
```

这里发生的事情是你用 Java 编译器`javac`创建了一个 Java 类文件，_PiratePhrases.class_。这个文件包含了大量的 Java 字节码（好吧，对于这么大的程序，也许只有一个字节）。

当你运行 "java PiratePhrases "时，JVM 首先查看了你的_classpath_，寻找一个名为 "PiratePhrases "的类。classpath 是文件系统的路径列表，JVM 通过搜索来寻找定义类的文件。默认情况下，classpath 包括你运行 java 时所在的目录。试着运行 java -classpath /tmp PiratePhrases，你会得到一个错误，尽管_PiratePhrases.class_就在你的当前目录中。

注意 你可以在你的 classpath 上有多个路径，如果你在 Mac 上或运行 Linux，可以用冒号隔开，如果你在使用 Windows，可以用分号。例如，classpath /tmp:/var/maven:.包括/tmp、/var/maven 和.目录。

在 Java 中，每个文件只允许有一个公有类，而且文件名必须与类名一致。这就是为什么`java`知道要尝试在_PiratePhrases.class_中寻找`PiratePhrases`类的字节码。在`java`找到`PiratePhrases`类的字节码后，它执行了该类的`main`方法。Java 与 C 语言类似，只要你说 "运行某些东西，并使用这个类作为入口点"，它就会一直运行这个类的`main'方法；因此，这个方法必须是`public'，你可以在\`PiratePhrases'的源代码中看到。

在下一节，你将学习如何处理跨越多个文件的程序代码，以及如何使用 Java 库。

## 包和导入

为了了解如何使用多文件程序和 Java 库，我们将编译并运行一个程序。本节对 Clojure 有直接的影响，因为你将使用同样的想法和术语来与 Java 库进行交互。

让我们从几个定义开始。

* **包**与 Clojure 的命名空间类似，包提供了代码组织。包包含类，包名对应于文件系统的目录。如果一个文件中有 "package com.shapemaster "一行，那么目录_com/shapemaster_一定存在于你的 classpath 上。在该目录中会有定义类的文件。
* **import** Java 允许你导入类，这基本上意味着你可以不使用它们的命名空间前缀来引用它们。所以如果你在`com.shapemaster`中有一个名为`Square`的类，你可以在`.java`文件的顶部写上`import``com.shapemaster.Square;`或`import com.shapemaster.*;`，以便在你的代码中使用`Square`而不是`com.shapemaster.Square`。

让我们试试使用`package`和`import`。在这个例子中，你将创建一个名为`pirate_phrases`的包，它有两个类，`问候'和`告别'。 首先，浏览你的_phrasebook_，在该目录下创建另一个目录，_pirate\_phrases_。创建_pirate\_phrases_是必要的，因为 Java 包的名称与文件系统的目录相对应。然后，在_pirate\_phrases_目录下创建_Greetings.java_。

```
➊ package pirate_phrases;

public class Greetings
{
    public static void hello()
    {
        System.out.println("Shiver me timbers!!!");
    }
}
```

在➊，`package pirate_phrases;`表示这个类将是`pirate_phrases`包的一部分。现在在_pirate\_phrases_目录下创建_Farewells.java_。

```
package pirate_phrases;

public class Farewells
{
    public static void goodbye()
    {
        System.out.println("A fair turn of the tide ter ye thar, ye magnificent sea friend!!");
    }
}
```

现在在_phrasebook_目录下创建_PirateConversation.java_。

```
import pirate_phrases.*;

public class PirateConversation
{
    public static void main(String[] args)
    {
        Greetings greetings = new Greetings();
        greetings.hello();

        Farewells farewells = new Farewells();
        farewells.goodbye();
    }
}
```

第一行，`import pirate_phrases.*;`，导入了`pirate_phrases`包中的所有类，其中包含`问候'和`告别'类。

如果你在_phrasebook_目录下运行`javac PirateConversation.java`，接着运行`java PirateConversation`，你应该看到这个。

```
Shiver me timbers!!!
A fair turn of the tide ter ye thar, ye magnificent sea friend!!
```

亲爱的读者，她在那里吹了起来。她确实在吹。

注意，当你编译一个 Java 程序时，Java 会在你的 classpath 中搜索包。试着输入以下内容。

```
cd pirate_phrases
javac ../PirateConversation.java
```

你会得到这个结果。

```
../PirateConversation.java:1: error: package pirate_phrases does not exist
import pirate_phrases.*;
^
```

轰隆隆! Java 编译器刚刚告诉你，让你羞愧地垂下头来，也许还会哭泣一下。

为什么？它认为`pirate_phrases`包不存在。但这很愚蠢，对吗？你是在_pirate\_phrases_目录下！你是在_pirate\_phrases_目录下。

这里发生的情况是，默认的 classpath 只包括当前的目录，在这种情况下是_pirate\_phrases_。 `javac`试图找到_phrasebook/pirate\_phrases/pirate\_phrases_目录，但该目录并不存在。当你在_phrasebook_目录下运行`javac ../PirateConversation.java`时，`javac`试图找到_phrasebook/pirate\_phrases_目录，该目录确实存在。在不改变目录的情况下，尝试运行 javac -classpath ../ ../PirateConversation.java。吓我一跳，居然成功了! 这是因为你手动将 classpath 设置为_pirate\_phrases_的父目录，也就是_phrasebook_。从那里，`javac`可以成功地找到_pirate\_phrases_目录。

综上所述，包组织了代码，并要求有一个匹配的目录结构。导入类可以让你引用它们，而不需要预留整个类的包名。 `javac`和 Java 使用 classpath 查找包。

## JAR 文件

JAR 文件允许你将所有的\*.class_文件捆绑成一个单一的文件。导航到你的_phrasebook\*目录并运行以下程序。

```
jar cvfe conversation.jar PirateConversation PirateConversation.class
pirate_phrases/*.class
java -jar conversation.jar
```

这样就能正确显示海盗对话了。你把所有的类文件捆绑在_conversation.jar_中。使用`e`标志，你还指出`PirateConversation`类是_入口点_。入口点是包含 JAR 整体运行时应该执行的`main'方法的类，`jar'将这些信息存储在 JAR 文件中的_META-INF/MANIFEST.MF_文件中。如果你要阅读该文件，它将包含这一行。

```
Main-Class: PirateConversation
```

顺便说一下，当你执行 JAR 文件时，你不必担心你在哪个目录下，相对于文件而言。你可以换到_pirate\_phrases_目录，然后运行`java -jar .../conversation.jar`，就可以正常工作了。原因是 JAR 文件维护了目录结构。你可以用 jar tf conversation.jar 查看它的内容，它的输出是这样的。

```
META-INF/
meta-inf/manifest.mf
PirateConversation.class
pirate_phrases/Farewells.class
Pirate_phrases/Greetings.class
```

你可以看到，JAR 文件包括_pirate\_phrases_目录。关于 JARs 还有一个有趣的事实：它们实际上只是带有\*.jar\*扩展名的 ZIP 文件。你可以像对待其他 ZIP 文件一样对待它们。

## clojure.jar

现在你已经准备好看看 Clojure 在引擎盖下是如何工作的了! 下载\[1.9.0 稳定版]（[http://repo1.maven.org/maven2/org/clojure/clojure/1.7.0/clojure-1.9.0.zip](http://repo1.maven.org/maven2/org/clojure/clojure/1.7.0/clojure-1.9.0.zip)）并运行它。

```
java -jar clojure-1.7.0.jar
```

你应该看到最舒心的景象，Clojure REPL。它究竟是如何启动的呢？让我们看看 JAR 文件中的_META-INF/MANIFEST.MF_。

```
Manifest-Version: 1.0
Archiver-Version: Plexus Archiver
Created-By: Apache Maven
Built-By: hudson
Build-Jdk: 1.7.0_20
Main-Class: clojure.main
```

看起来，`clojure.main`被指定为入口点。这个类是怎么来的？嗯，看看 GitHub 上的_clojure/main.java_，网址是\*[https://github.com/clojure/clojure/blob/master/src/jvm/clojure/main.java](https://github.com/clojure/clojure/blob/master/src/jvm/clojure/main.java)\*。

```
/**
 *   Copyright (c) Rich Hickey. All rights reserved.
 *   The use and distribution terms for this software are covered by the
 *   Eclipse Public License 1.0 (http://opensource.org/licenses/eclipse-1.0.php)
 *   which can be found in the file epl-v10.html at the root of this distribution.
 *   By using this software in any fashion, you are agreeing to be bound by
 *   the terms of this license.
 *   You must not remove this notice, or any other, from this software.
 **/

package clojure;

import clojure.lang.Symbol;
import clojure.lang.Var;
import clojure.lang.RT;

public class main{

final static private Symbol CLOJURE_MAIN = Symbol.intern("clojure.main");
final static private Var REQUIRE = RT.var("clojure.core", "require");
final static private Var LEGACY_REPL = RT.var("clojure.main", "legacy-repl");
final static private Var LEGACY_SCRIPT = RT.var("clojure.main", "legacy-script");
final static private Var MAIN = RT.var("clojure.main", "main");

public static void legacy_repl(String[] args) {
    REQUIRE.invoke(CLOJURE_MAIN);
    LEGACY_REPL.invoke(RT.seq(args));
}

public static void legacy_script(String[] args) {
    REQUIRE.invoke(CLOJURE_MAIN);
    LEGACY_SCRIPT.invoke(RT.seq(args));
}

public static void main(String[] args) {
    REQUIRE.invoke(CLOJURE_MAIN);
    MAIN.applyTo(RT.seq(args));
}
}
```

正如你所看到的，该文件定义了一个名为`main`的类。它属于 "clojure "包，并定义了一个 "公共静态 "的 "main "方法，JVM 完全乐意将其作为一个入口点。以这种方式来看，Clojure 是一个 JVM 程序，就像其他程序一样。

这并不是一个深入的 Java 教程，但我希望它有助于澄清程序员在谈论 Clojure "在 JVM 上运行 "或成为一种 "托管 "语言时的意思。在下一节中，你将继续探索 JVM 的魅力，学习如何在你的 Clojure 项目中使用额外的 Java 库。

Clojure 应用程序 JARs

你现在知道 Java 是如何运行 Java JARs 的，但它是如何运行捆绑为 JARs 的 Clojure 应用程序的呢？毕竟，Clojure 应用程序没有类，不是吗？

事实证明，你可以通过在命名空间声明中加入`(:gen-class)`指令，让 Clojure 编译器为一个命名空间生成一个类。(你可以在你创建的第一个 Clojure 程序中看到这一点，即第一章的_clojure-noob_。还记得那个程序吗，小茶壶？） 这意味着编译器会产生必要的字节码，使 JVM 把命名空间当作定义了一个 Java 类。

你在程序的_project.clj_文件中，使用`:main`属性，为你的程序设置入口点的命名空间。对于_clojure-noob_，你应该看到`:main ^:skip-aot clojure-noob.core`。当 Leiningen 编译这个文件时，它将添加一个_meta-inf/manifest.mf_文件，该文件包含了生成的 JAR 文件的入口点。

因此，如果你在命名空间中定义了一个`-main`函数，并包括`(:gen-class)`指令，同时在你的_project.clj_文件中设置了`:main`，你的程序在被编译为 JAR 时，将拥有 Java 运行它所需的一切。你可以在你的终端中试用这个方法，浏览你的_clojure-noob_目录并运行这个。

```
lein uberjar
java -jar target/uberjar/clojure-noob-0.1.0-SNAPSHOT-standalone.jar
```

你应该看到打印出来的两条信息。"清洁度仅次于神性 "和 "I'm a little teapot!" 注意，你不需要 Leiningen 来运行这个 JAR 文件；你可以把它发送给朋友和邻居，只要他们安装了 Java，就可以运行它。

## Java Interop

Rich Hickey 对 Clojure 的设计目标之一是创造一种_实用的语言。出于这个原因，Clojure 的设计是为了使你能够轻松地与 Java 类和对象进行交互，这意味着你可以使用 Java 广泛的本地功能和它的巨大生态系统。使用 Java 类、对象和方法的能力被称为_Java interop\*。在本节中，你将学习如何使用 Clojure 的互操作语法，如何导入 Java 包，以及如何使用最常用的 Java 类。

### 互通语法

使用 Clojure 的互操作语法，与 Java 对象和类的交互是很直接的。让我们从对象互操作语法开始。

你可以使用`(.`methodName object)来调用一个对象的方法。例如，因为所有的 Clojure 字符串都是作为 Java 字符串实现的，所以你可以对它们调用 Java 方法。

```
(.toUpperCase "By Bluebeard's bananas!" )
; => "by bluebeard's bananas!"

➊ (.indexOf "Let's synergize our bleeding edges" "y") 
; => 7
```

这些等同于这个 Java。

```
"By Bluebeard's bananas!".toUpperCase()
"Let's synergize our bleeding edges".indexOf("y")
```

注意，Clojure 的语法允许你向 Java 方法传递参数。在这个例子中，在➊，你把参数`"y"`传给了`indexOf`方法。

你也可以调用类上的静态方法和访问类的静态字段。观察一下!

```
➊ (java.lang.Math/abs -3) 
; => 3

➋ java.lang.Math/PI 
; => 3.141592653589793
```

在➊，你调用了`java.lang.Math`类的`abs`静态方法，在➋，你访问了该类的`PI`静态字段。

所有这些例子（除了`java.lang.Math/PI`）都使用了扩展到使用\*dot 特殊形式的宏。一般来说，你不需要使用点的特殊形式，除非你想写自己的宏来与 Java 对象和类交互。尽管如此，下面是每个例子后面的宏扩展。

```
(macroexpand-1 '(.toUpperCase "By Bluebeard's bananas!"))
; => (. "By Bluebeard's bananas!" toUpperCase)

(macroexpand-1 '(.indexOf "Let's synergize our bleeding edges" "y"))
; => (. "Let's synergize our bleeding edges" indexOf "y")

(macroexpand-1 '(Math/abs -3))
; => (. Math abs -3)
```

这是点运算符的一般形式。

```
(. object-expr-or-classname-symbol method-or-member-symbol optional-args*)
```

点运算符还有一些功能，如果你有兴趣进一步探索它，你可以看看 clojure.org 关于 Java 互操作的文档\*[http://clojure.org/java\_interop#Java%20Interop-The%20Dot%20special%20form](http://clojure.org/java\_interop#Java%20Interop-The%20Dot%20special%20form)\*。

创建和变异对象

上一节告诉你如何调用已经存在的对象的方法。本节向你展示如何创建新的对象以及如何与它们进行交互。

你可以通过两种方式创建一个新的对象。`(new ClassName optional-args)`和`(ClassName. optional-args)`。

```
(new String)
; => ""

(String.)
; => ""

(String. "To Davey Jones's Locker with ye hardies")
; => "To Davey Jones's Locker with ye hardies"
```

大多数人使用点的版本，`(ClassName.)`。

要修改一个对象，你要像上一节那样调用其上的方法。为了研究这个问题，让我们使用`java.util.Stack`。这个类代表了一个后进先出（LIFO）的对象堆栈，或者只是_堆栈_。_堆栈_是一种常见的数据结构，它们之所以被称为堆栈，是因为你可以把它们想象成一摞实物，比如说，一摞你刚刚掠夺来的金币。当你向你的堆栈添加一个硬币时，你就把它添加到堆栈的顶部。当你取出一枚金币时，你就把它从上面移走。因此，最后添加的对象就是第一个被移除的对象。

与 Clojure 数据结构不同，Java 堆栈是可变的。你可以向它们添加项目和删除项目，改变对象而不是派生出一个新的值。下面是你如何创建一个堆栈并向其添加一个对象。

```
(java.util.Stack.)
; => []

➊ (let [stack (java.util.Stack.)] 
  (.push stack "Latest episode of Game of Thrones, ho!")
  stack)
; => ["Latest episode of Game of Thrones, ho!"]
```

这里有几个有趣的细节。首先，你需要为`stack`创建一个`let`绑定，就像你在➊看到的那样，并把它作为`let`形式的最后一个表达式。如果你不这样做，整个表达式的值将是字符串`"Game of Thrones, ho!"`，因为那是`push`的返回值。

第二，Clojure 用方括号来打印堆栈，与它用于 Vector 的文本表示法相同，这可能会让你感到困惑，因为它不是一个 Vector。然而，你可以使用 Clojure 的`seq`函数来读取堆栈中的数据结构，比如`first`，。

```
(let [stack (java.util.Stack.)]
  (.push stack "Latest episode of Game of Thrones, ho!")
  (first stack))
; => "Latest episode of Game of Thrones, ho!"
```

但是你不能使用像`conj`和`into`这样的函数来添加元素到栈中。如果你这样做，你会得到一个异常。使用 Clojure 函数读取堆栈是可能的，因为 Clojure 扩展了对`java.util.Stack`的抽象，这个主题你将在第 13 章学习。

Clojure 提供了`doto`宏，它允许你更简洁地在同一个对象上执行多个方法。

```
(doto (java.util.Stack.)
  (.push "Latest episode of Game of Thrones, ho!")
  (.push "Whoops, I meant 'Land, ho!'"))
; => ["Latest episode of Game of Thrones, ho!" "Whoops, I meant 'Land, ho!'"]
```

`doto`宏返回对象，而不是任何方法调用的返回值，它更容易理解。如果你用`macroexpand-1`展开它，你可以看到它的结构与你刚才在前面的例子中看到的`let`表达式相同。

```
(macroexpand-1
 '(doto (java.util.Stack.)
    (.push "Latest episode of Game of Thrones, ho!")
    (.push "Whoops, I meant 'Land, ho!'")))
; => (clojure.core/let
      [G__2876 (java.util.Stack.)]
      (.push G__2876 "Latest episode of Game of Thrones, ho!")
      (.push G__2876 "Whoops, I meant 'Land, ho!'")
      G__2876)
```

很方便!

### 导入

在 Clojure 中，导入的效果和 Java 中的一样：你可以使用类，而不需要打出整个包的前缀。

```
(import java.util.Stack)
(Stack.)
; => []
```

你也可以使用这种一般形式一次导入多个类。

```
(import [package.name1 ClassName1 ClassName2]
        [package.name2 ClassName3 ClassName4])
```

下面是一个例子。

```
(import [java.util Date Stack]
        [java.net Proxy URI])

(Date.)
; => #inst "2016-09-19T20:40:02.733-00:00"
```

但通常情况下，你会在`ns`宏中做所有的导入工作，像这样。

```
(ns pirate.talk
  (:import [java.util Date Stack].
           [java.net Proxy URI])
```

这两种不同的导入类的方法有相同的结果，但通常第二种方法更可取，因为对于阅读你的代码的人来说，在`ns`声明中看到所有涉及命名的代码很方便。

这就是你导入类的方法! 很简单。为了使生活更加简单，Clojure 自动导入了`java.lang`中的类，包括`java.lang.String`和`java.lang.Math`，这就是为什么你能够使用`String`而不用前面的包名。

## 常用的 Java 类

为了完善本章，让我们快速浏览一下你最可能用到的 Java 类。

### 系统类

系统 "类具有有用的类字段和方法，可以与程序运行的环境进行交互。你可以用它来获取环境变量，与标准输入、标准输出和错误输出流进行交互。

最有用的方法和成员是`exit`、`getenv`和`getProperty`。你可能在第 5 章中认识`System/exit`，在那里你用它来退出 Peg Thing 游戏。\`System/exit'可以终止当前程序，你可以把状态代码作为参数传给它。如果你对状态代码不熟悉，我推荐维基百科的 "退出状态 "文章，网址是\*[退出状态-维基百科](http://en.wikipedia.org/wiki/Exit\_status)\*。

`System/getenv`将以 Map 形式返回所有系统的环境变量。

```
(System/getenv)
{"USER" "the-incredible-bulk"
 "JAVA_ARCH" "x86_64" }
```

环境变量的一个常见用途是配置你的程序。

JVM 有自己的属性列表，与计算机的环境变量分开，如果需要读取它们，可以使用`System/getProperty`。

```
➊ (System/getProperty "user.dir")
; => "/Users/dabulk/projects/dabook"

➋ (System/getProperty "java.version")
; => "1.7.0_17"
```

第一个调用➊返回 JVM 启动的目录，第二个调用➋返回 JVM 的版本。

### 日期类

Java 有很好的工具来处理日期问题。我不会对`java.util.Date`类做太多的介绍，因为在线的 API 文档（可在\*[Date (Java Platform SE 7 )](http://docs.oracle.com/javase/7/docs/api/java/util/Date.html)\*)很详尽。作为一个 Clojure 开发者，你应该知道这个`date`类的三个特点。首先，Clojure 允许你使用这样的形式将日期表示为字面意义。

```
#inst "2016-09-19T20:40:02.733-00:00"
```

第二，如果你想自定义如何将日期转换成字符串，或者你想将字符串转换成日期，你需要使用`java.util.DateFormat`类。第三，如果你要做的任务是比较日期或试图在日期上添加分钟、小时或其他时间单位，你应该使用极其有用的 clj-time 库（你可以在\*[GitHub - clj-time/clj-time: 一个用于 Clojure 的日期和时间库，包装了 Joda 时间库。](https://github.com/clj-time/clj-time)\*)。

## 文件和输入/输出

在这一节中，你将了解到 Java 的输入/输出（IO）方法，以及 Clojure 如何简化它。`clojure.java.io`命名空间提供了许多方便的函数来简化 IO（[_clojure.java.io - Clojure v1.10.3 API 文档_](https://clojure.github.io/clojure/clojure.java.io-api.html)）。这很好，因为 Java 的 IO 并不完全是简单的。因为在你的编程生涯中，你可能会在某些时候想要执行 IO，让我们开始把你的思想触角缠绕在它上面。

IO 涉及到资源，无论是文件、套接字、缓冲区，还是其他什么。Java 有独立的类来读取资源的内容，写入其内容，以及与资源的属性进行交互。

例如，`java.io.File`类用于与文件的属性进行交互。

```
(let [file (java.io.File. "/")]
➊   (println (.exists file))  
➋   (println (.canWrite file))
➌   (println (.getPath file))) 
; => true
; => false
; => /
```

![](https://www.braveclojure.com/assets/images/cftbat/java/lion.png)

在其他任务中，你可以用它来检查一个文件是否存在，获得文件的读/写/执行权限，并获得其文件系统路径，你可以在➊、➋和➌分别看到。

在这个能力列表中，明显缺少读和写。要读一个文件，你可以使用`java.io.BufferedReader`类或者`java.io.FileReader`。同样地，你可以使用`java.io.BufferedWriter`或`java.io.FileWriter`类来写。其他类也可用于读写，你选择哪一个取决于你的具体需求。读取器和写入器类的接口都有相同的基本方法集；读取器实现了`读取'、`关闭'等，而写入器实现了`添加'、`写入'、`关闭'和`刷新'。Java 给你提供了各种 IO 工具。一个愤世嫉俗的人可能会说，Java 给你的绳子足以让你上吊，如果你找到这样一个人，我希望你能给他一个拥抱。

不管怎么说，Clojure 使你的读写更容易，因为它包括了统一不同种类资源的读写的函数。例如，`spit`写到一个资源，而`slurp`从一个资源中读出。下面是一个使用它们来写和读一个文件的例子。

```
(spit "/tmp/hercules-todo-list"
"- kill dat lion brov
- chop up what nasty multi-headed snake thing")

(slurp "/tmp/hercules-todo-list")

; => "- kill dat lion brov
      - chop up what nasty multi-headed snake thing"
```

你也可以对代表文件以外的资源的对象使用这些函数。下一个例子使用了一个`StringWriter`，它允许你对一个字符串进行 IO 操作。

```
(let [s (java.io.StringWriter.)]
  (spit s "- capture cerynian hind like for real")
  (.toString s))
; => "- capture cerynian hind like for real"
```

你也可以使用 "slurp "从`StringReader`中读取。

```
(let [s (java.io.StringReader. "- get erymanthian pig what with the tusks")]
  (slurp s))
; => "- get erymanthian pig what with the tusks"
```

此外，你可以对资源使用`读`和`写`方法。使用哪种方法并没有什么区别；`spit`和`slurp`很方便，因为它们只需使用一个代表文件系统路径或 URL 的字符串。

`with-open`宏是另一种便利：它在其主体的末尾隐含地关闭一个资源，确保你不会因为忘记手动关闭资源而意外地占用资源。`reader`函数是一个方便的工具，根据`clojure.java.io`API 文档，"试图将其参数强制到一个开放的`java.io.Reader`"。当你不想使用`slurp`时，这很方便，因为你不想尝试完整地读取一个资源，你也不想弄清楚你需要使用哪个 Java 类。如果你想一行一行地读取一个文件，你可以使用`reader`和`with-open`以及`line-seq`函数。下面是如何打印 Hercules 待办事项清单的第一项的。

```
(with-open [todo-list-rdr (clojure.java.io/reader "/tmp/hercules-todo-list")]
  (println (first (line-seq todo-list-rdr))))
; => - kill dat lion brov
```

这应该足以让你在 Clojure 中开始使用 IO。如果你想做更复杂的任务，一定要看看[`clojure.java.io` docs](https://clojure.github.io/clojure/clojure.java.io-api.html)，`[java.nio.file](https://docs.oracle.com/javase/7/docs/api/java/nio/file/package-summary.html)`包文档，或`[java.io](http://docs.oracle.com/javase/7/docs/api/java/io/package-summary.html)`包文档。

## 资源

* "Java 虚拟机和编译器的解释"。 [_Java 虚拟机和编译器的解释--YouTube_](https://www.youtube.com/watch?v=XjNwyXx2os8)
* clojure.org Java 互操作文档。 [_Clojure - Java Interop_](http://clojure.org/java\_interop)
* 维基百科的 "退出状态 "文章。 [_退出状态 - 维基百科_](http://en.wikipedia.org/wiki/Exit\_status)

## 总结

在本章中，你了解了 Clojure 被托管在 JVM 上的含义。Clojure 程序被编译成 Java 字节码并在 JVM 进程中执行。Clojure 程序也可以访问 Java 库，你可以使用 Clojure 的互操作设施轻松地与它们交互。
