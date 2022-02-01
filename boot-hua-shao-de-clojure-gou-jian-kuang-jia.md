# Boot，花哨的 Clojure 构建框架

Boot 是 Leiningen 的替代品，提供同样的功能。Leiningen 更受欢迎（截至 2015 年夏天），但我个人喜欢用 Boot 工作，因为它更容易扩展。本附录解释了 Boot 的基本概念，并指导你编写你的第一个 Boot 任务。如果你对使用 Boot 构建项目感兴趣，请查看它的 GitHub README（[_GitHub - boot-clj/boot: Build tooling for Clojure._](https://github.com/boot-clj/boot/)）和它的 wiki（[Home - boot-clj/boot Wiki - GitHub](https://github.com/boot-clj/boot/wiki/)_）_。

注意 截至本文写作时，Boot 对 Windows 的支持有限。Boot 团队欢迎大家的贡献!

## Boot 的抽象

Boot 由 Micha Niskin 和 Alan Dipert 创建，是对 Clojure 工具领域的一个有趣而强大的补充。从表面上看，它是构建 Clojure 应用程序和从命令行运行 Clojure 任务的一种便捷方式。深入研究一下，你会发现 Boot 就像 Git 和 Unix 的爱情结晶，它提供的抽象使你在操作系统和应用程序的交叉点上编写代码时更加愉快。

Unix 提供了我们都很熟悉的抽象，以至于我们认为它们是理所当然的。(偶尔带你的电脑去吃一顿好的餐厅会死吗？) 进程抽象让你把程序推理成独立的逻辑单元，可以通过 STDIN 和 STDOUT 文件描述符轻松地组成一个流处理管道。这些抽象使某些类型的操作，如文本处理，变得非常直接。

同样，Boot 也提供了一些抽象，使得独立的操作很容易被组合成构建工具最终要做的那种复杂、协调的操作，比如将 ClojureScript 转换为 JavaScript。 Boot 的任务抽象让你可以轻松地定义逻辑单元，通过_文件集合_进行通信。文件集合抽象可以跟踪不断变化的构建环境，并提供一个定义明确、可靠的任务协调方法。

这就是很多高层次的描述，希望能吸引你的注意力。但是，如果我带着一板一眼的隐喻离开你，那就太丢人了。哦，不，亲爱的读者，这只是开胃菜而已。在本附录的其余部分，你将学习如何建立自己的 Boot 任务。在这一过程中，你会发现，构建工具实际上是有概念基础的。

## 任务

像 make、rake、grunt 和其他以前的构建工具一样，Boot 让你定义任务。 _任务_是命名的操作，接受由某个中间程序（make、rake、Boot）调度的命令行选项。

Boot 提供了调度程序_boot_和一个 Clojure 库，使你可以很容易地用`deftask`宏来定义命名的操作及其命令行选项。为了看看所有的大惊小怪，让我们来创建你的第一个任务。通常情况下，编程教程鼓励你写代码来打印 "Hello World"，但我希望我的例子能有真实的效用，所以你的任务是打印 "我的裤子着火了！" 这个信息客观上更有用。首先，安装 Boot；然后创建一个名为_boot-walkthrough_的新目录，导航到该目录，创建一个名为\*build.boot\*\*的文件，\*然后这样写。

```
(deftask fire
  "Prints 'My pants are on fire!'"
  []
  (println "My pants are on fire!"))
```

现在用`boot fire`从命令行运行这个任务；你应该看到你写的信息被打印到终端。这个任务展示了三个任务组件中的两个：任务被命名为（`fire`），并且由 boot 调度。这真是太酷了。你基本上已经创建了一个 Clojure shell 脚本，独立的 Clojure 代码，你可以轻松地从命令行运行。不需要_project.clj_，不需要目录结构，也不需要命名空间!

让我们扩展一下这个例子，演示一下你如何编写命令行选项。

```
(deftask fire
  "Announces that something is on fire"
  [t thing     THING str  "The thing that's on fire"
   p pluralize       bool "Whether to pluralize"]
  (let [verb (if pluralize "are" "is")]
    (println "My" thing verb "on fire!")))
```

试着像这样运行该任务。

```
boot fire -t heart
# => My heart is on fire!

boot fire -t logs -p
# => My logs are on fire!
```

在第一种情况下，要么你是新近恋爱，要么你需要赶到急诊室。在第二个例子中，你是一个童子军，尴尬地表达了你对达到功绩勋章要求的兴奋。在这两种情况下，你都能轻松地指定任务的选项。

这次对`fire`任务的改进引入了两个命令行选项，`thing`和`pluralize`。这两个选项都是用\*域特定语言（DSL）\*定义的。DSL 是他们自己的主题，但简单地说，这个术语指的是微型语言，你可以在一个大的程序中使用，为狭义的领域（如定义选项）编写紧凑的、富有表现力的代码。

在选项`thing`中，`t`指定其短名称，`thing`指定其长名称。 `THING`有点复杂，我稍后会讲到它。 `str`指定了选项的类型，Boot 用它来验证参数并进行转换。 `"着火的东西 "是该选项的文档。你可以用`boot task-name -h\`在终端查看一个任务的文档。

```
boot fire -h
# Announces that something is on fire
#
# Options:
#   -h, --help         Print this help info.
#   -t, --thing THING  Set the thing that's on fire to THING.
#   -p, --pluralize    Whether to pluralize
```

相当棒的! Boot 使编写要从命令行调用的代码变得非常容易。

现在，让我们看看`THING`。`THING`是一个_optarg_，它表示这个选项需要一个参数。当你定义一个选项时，你不需要包括 optarg（注意`pluralize`选项没有 optarg）。optarg 不必与选项的全名相对应；你可以用`BILLY_JOEL'或其他你想要的东西来代替`THING'，任务也会照常进行。你也可以使用 optarg 来指定复杂的选项。([访问_https://github.com/boot-clj/boot/wiki/Task-Options-DSL#complex-options_](https://github.com/boot-clj/boot/wiki/Task-Options-DSL#complex-options)了解 Boot 关于这个问题的文档。) 基本上，复杂选项允许你指定选项参数应被视为 Map、集合、Vector，甚至是嵌套集合。这是很强大的。

Boot 为你提供了用 Clojure 构建命令行界面所需的所有工具。而你才刚刚开始学习它!

## The REPL

Boot 有许多有用的内置任务，包括一个 REPL 任务。运行 `boot repl` 来启动这个小家伙。Boot 的 REPL 与 Leiningen 的类似，它负责加载你的项目代码，这样你就可以随意玩耍。你可能认为这不适用于你所写的项目，因为你只写了任务，但实际上你可以在 REPL 中运行任务（我省略了`boot.user=>`提示）。你可以用一个字符串指定选项。

```
(fire "-t" "NBA Jam guy")
; My NBA Jam guy is on fire!
; => nil
```

注意，选项的值就在选项的后面。

你也可以用关键字来指定一个选项。

```
(fire :thing "NBA Jam guy")
; My NBA Jam guy is on fire!
; => nil
```

你也可以结合选项。

```
(fire "-p" "-t" "NBA Jam guys")
; My NBA Jam guys are on fire!
; => nil

(fire :pluralize true :thing "NBA Jam guys")
; My NBA Jam guys are on fire!
; => nil
```

当然，你也可以在 REPL 中使用`deftask`，毕竟这只是 Clojure。我们的收获是，Boot 可以让你把任务作为 Clojure 函数进行交互，因为它们就是这样的。

## 组成和协调

如果到目前为止你所看到的就是 Boot 所能提供的一切，那它将是一个非常棒的工具，但它与其他构建工具没有什么不同。让 Boot 与众不同的一个特点是，它可以让你编排任务。为了便于比较，这里有一个 Rake 调用的例子（Rake 是主要的 Ruby 构建工具）。

```
rake db:create d{:tag :a, :attrs {:href "db:seed"}, :content ["b:migra"]}te db:seed
```

这段代码将创建一个数据库，在其上运行迁移，并在 Rails 项目中运行时向其填充种子数据。然而，值得注意的是，Rake 并没有提供任何方法让这些任务之间相互通信。指定多个任务只是为了方便，让你不必运行`rake db:create; rake db:migrate; rake db:seed`。如果你想在任务 B 中访问任务 A 的结果，构建工具并不能帮助你；你必须自己管理这种协调。通常，你要做的是把任务 A 的结果塞进文件系统中的一个特殊位置，然后确保任务 B 读取这个特殊位置。这看起来就像用易变的全局变量进行编程，而且它也是很脆弱的。

### Handler 和中间件

Boot 通过将任务视为_中间\*\*件工厂_来解决这个任务通信问题。如果你熟悉 Ring，Boot 的任务工作起来非常相似，所以请随意跳到["任务是中间件工厂 "第 287 页](https://www.braveclojure.com/appendix-b/#Anchor)。如果你对中间件的概念不熟悉，请允许我解释一下! _中间件_指的是程序员遵守的一套_公约，这样他们就可以灵活地创建特定领域的功能管道。这是相当密集的，所以让我们解除密集。我将在本节中讨论_灵活的部分，并在["文件集合 "第 288 页](https://www.braveclojure.com/appendix-b/#Anchor-12)中介绍_特定领域的_。

为了理解中间件方法与普通函数组合的不同之处，这里有一个组合日常函数的例子。

```
(def strinc (comp str inc))
(strinc 3)
; => "4"
```

这个函数组合并没有什么有趣的地方。事实上，这个函数组合是如此的不起眼，以至于我作为一个作家，要对它说些什么都很费劲。有两个函数，各自做自己的事情，现在它们被组成了一个。Whoop-dee-doo!

中间件为函数组合引入了一个额外的步骤，使你在定义函数管道时有更大的灵活性。假设在前面的例子中，你想对任意的数字返回 "我不喜欢这个数字 X"，而对其他的东西返回一个字符串化的数字。以下是你如何做到这一点的。

```
(defn whiney-str
  [rejects]
  {:pre [(set? rejects)]}
  (fn [x]
    (if (rejects x)
      (str "I don't like " x)
      (str x))))

(def whiney-strinc (comp (whiney-str #{2}) inc))
(whiney-strinc 1)
; => "I don't like 2"
```

现在让我们再进一步。如果你想决定是否首先调用`inc`呢？清单 B-1 显示了你如何做到这一点。

```
(defn whiney-middleware
  [next-handler rejects]
  {:pre [(set? rejects)]}
  (fn [x]
➊     (if (= x 1)
        "I'm not going to bother doing anything to that"
        (let [y (next-handler x)]
          (if (rejects y)
            (str "I don't like " y)
            (str y))))))

(def whiney-strinc (whiney-middleware inc #{2}))
(whiney-strinc 1)
; => "I'm not going to bother doing anything to that"
```

1. B-1. 函数组合的中间件方法让你引入选择权

在这里，你不是用`comp`来创建你的函数管道，而是将管道中的下一个函数作为第一个参数传递给中间件函数。在这种情况下，你将`inc`作为第一个参数传递给`whiney-middleware`作为`next-handler`。 `whiney-middleware`然后返回一个匿名函数，该函数关闭了`inc`并有能力选择是否调用它。你可以在➊看到这个选择。

我们说，一个中间件把一个 Handler 作为它的第一个参数，并返回一个 Handler。在这个例子中，`whiney-middleware`将一个 Handler 作为它的第一个参数，`inc`，它返回另一个 Handler，即匿名函数，`x`是它唯一的参数。中间件也可以接受额外的参数，如`rejects`，作为配置。其结果是，中间件返回的 Handler 可以表现得更加灵活（由于配置），而且它对函数管道有更多的控制（因为它可以选择是否调用下一个 Handler）。

### 任务是中间件工厂

Boot 通过将中间件的配置与 Handler 的创建分开，将这种使函数组合更加灵活的模式向前推进了一步。首先，你创建一个接受_n_配置参数的函数。这就是_中间件工厂_，它返回一个中间件函数。中间件函数希望得到一个参数，即下一个 Handler，并返回一个 Handler，就像前面的例子中一样。下面是一个发牢骚的中间件工厂。

```
(defn whiney-middleware-factory
  [rejects]
  {:pre [(set? rejects)]}
  (fn [handler]
    (fn [x]
      (if (= x 1)
        "I'm not going to bother doing anything to that"
        (let [y (handler x)]
          (if (rejects y)
            (str "I don't like " y " :'(")
            (str y)))))))

(def whiney-strinc ((whiney-middleware-factory #{3}) inc))
```

正如你所看到的，这段代码与清单 B-1 几乎相同。变化在于，最上面的函数，`whiney-middleware-factory`，现在只接受一个参数，`rejects`。它返回一个匿名函数，即中间件，它希望得到一个参数，即 Handler。其余的代码都是一样的。

在 Boot 中，任务可以充当中间件工厂。为了说明这一点，让我们把`fire`任务分成两个任务：`what`和`fire`（见清单 B-2）。 `what`让你指定一个对象以及它是否是复数，而`fire`则宣布它着火了。这是伟大的模块化软件工程，因为它允许你添加其他任务，如`gnomes`，宣布一个东西被地精占领了，这在客观上同样有用。(作为一个练习，尝试创建 gnome 任务。它应该和`what`任务组成，就像`fire`一样）。

```
(deftask what
  "Specify a thing"
  [t thing     THING str  "An object"
   p pluralize       bool "Whether to pluralize"]
  (fn middleware [next-handler]
➊     (fn handler [fileset]
      (next-handler (merge fileset {:thing thing :pluralize pluralize})))))

(deftask fire
  "Announce a thing is on fire"
  []
  (fn middleware [next-handler]
➋     (fn handler [fileset]
      (let [verb (if (:pluralize fileset) "are" "is")]
        (println "My" (:thing fileset) verb "on fire!")
        fileset))))
```

1. 宣布某物着火的可组合 Boot 任务的完整代码

以下是你如何在命令行上运行它。

```
boot what -t "pants" -p - fire
```

下面是在 REPL 中的运行方式。

```
(boot (what :thing "pants" :pluralize true) (fire))
```

等一下，那个`boot'的调用是怎么回事？在➊和➋的`fileset`又是怎么回事？用Micha的话说，"`boot`宏负责设置和清理（创建初始文件集合，停止由任务启动的服务器，诸如此类的事情）。任务是函数，所以你可以直接调用它们，但如果它们使用了文件集合，就会失败，除非你通过`boot\`宏调用它们。" 让我们仔细看看文件集合的情况。

## 文件集合

前面我提到，中间件是用来创建_域特定的_函数管道。这意味着每个 Handler 都期望接收特定领域的数据并返回特定领域的数据。以 Ring 为例，每个 Handler 都希望收到一个代表 HTTP 请求的请求 Map，它可能看起来像这样。

```
{:server-port 80
 :request-method :get
 :scheme :http}
```

每个 Handler 可以选择以某种方式修改这个请求 Map，然后再传递给下一个 Handler，例如，添加一个`:params`键，其中包含所有查询字符串和 POST 参数的漂亮 Clojure Map。环形 Handler 返回一个_响应 Map_，由`:status'、`:headers'和\`:body'三个键组成，每个 Handler 可以再次以某种方式转换这些数据，然后再返回给其父 Handler。

在 Boot 中，每个 Handler 接收并返回一个_fileset_。文件集合的抽象让你把文件系统上的文件当作不可更改的数据，这对构建工具来说是一项伟大的创新，因为构建项目是以文件为中心的。例如，你的项目可能需要在文件系统上放置临时的、中间的文件。通常，在大多数构建工具中，这些文件被放置在一些特别命名的地方，比如，_project/target/tmp_。这样做的问题是，_project/target/tmp_实际上是一个全局变量，其他任务可能会意外地把它搞乱。

Boot 的文件集合抽象通过在文件系统上增加一层间接性来解决这个问题。比方说，任务 A 创建了文件 X，并告诉文件集合来存储它。在幕后，文件集合将该文件存储在一个匿名的临时目录中。然后，该文件集合被传递给任务 B，任务 B 修改了文件 X 并要求文件集合存储结果。在幕后，一个新的文件，文件 Y，被创建和存储，但文件 X 仍然没有被触动。在任务 B 中，一个更新的文件集合被返回。这相当于用 Map 做 "assoc-in"。任务 A 仍然可以访问原始文件集合和它引用的文件。

在清单 B-2 中的`what'和`fire'任务中，你甚至都没有使用这些很酷的文件管理功能。尽管如此，当 Boot 组成任务时，它希望 Handler 能接收并返回 fileset 记录。因此，为了跨任务传达你的数据，你偷偷地用`(merge fileset {:thing thing :pluralize pluralize})`把它加到文件集合记录中。

虽然这涵盖了中间件工厂的基本概念，但你还需要学习更多的东西来充分利用文件集合的优势。在 fileset wiki（[_Filesets - boot-clj/boot Wiki - GitHub_](https://github.com/boot-clj/boot/wiki/Filesets)）中，对使用 filesets 的机制都有解释。同时，我希望这些信息能给你一个很好的概念性概述!

## 接下来的步骤

本附录的重点是解释 Boot 背后的概念。不过，Boot 还有一堆其他的功能，比如`set-env!`和`task-options!`，当你真正使用它的时候，会让你的编程生活更轻松。它提供了惊人的神奇功能，比如提供 classpath 隔离，这样你就可以用一个 JVM 运行多个项目，并让你在无需重启 REPL 的情况下向项目添加新的依赖项。如果 Boot 让你心痒难耐，请查看它的 README，了解更多关于实际使用的信息。另外，它的 wiki 提供了一流的文档。
