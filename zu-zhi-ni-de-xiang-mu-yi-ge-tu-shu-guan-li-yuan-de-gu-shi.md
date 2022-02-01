# 组织你的项目：一个图书管理员的故事

在我们每个人心中都住着一个叫 Melvil 的图书管理员，一个以组织艺术为乐的奇异生物。日日夜夜，Melvil 都渴望为你的代码库带来秩序。幸运的是，Clojure 提供了一套工具，专门用来帮助这个侏儒与混乱的力量不断斗争。

这些工具通过将相关的函数和数据分组来帮助你组织你的代码。它们还可以防止名称冲突，这样你就不会意外地覆盖别人的代码，反之亦然。在这个充满悬念和神秘的故事中，请和我一起学习如何使用这些工具，并解决一生中的抢劫案吧 在这个传奇故事的最后，你将了解以下内容。

* \`def'是做什么的
* 什么是命名空间以及如何使用它们
* 命名空间和文件系统之间的关系
* 如何使用`refer`、`alias`、`require`、`use`和`ns`。
* 如何使用文件系统来组织 Clojure 项目

我先来介绍一下 Clojure 的组织系统，它的工作原理很像一个库。Melvil 兴奋地颤抖着!

## 你的项目是一个库

现实世界中的图书馆存储对象的集合，如书籍、杂志和 DVD。他们使用寻址系统，所以当你得到一个物体的地址时，你可以导航到物理空间并检索到该物体。

当然，没有人能够直接知道一本书或 DVD 的地址是什么。这就是为什么图书馆要记录一个物体的标题和它的地址之间的联系，并提供工具来搜索这些记录。在计算机之前的旧时代，图书馆提供卡片目录，即装满纸质卡片的柜子，其中包含每本书的标题、作者、"地址"（杜威十进制或国会图书馆编号）和其他信息。

例如，要找到《达芬奇密码》，你可以翻阅书名目录（按书名排序的卡片），直到你找到正确的卡片。在那张卡片上，你会看到地址_813.54_（如果它使用杜威十进制系统），浏览图书馆，找到_达芬奇密码_所在的书架，并参与你一生中的文学和/或仇恨阅读冒险。

在 Clojure 中想象一个类似的设置是很有用的。我认为 Clojure 是将对象（如数据结构和函数）存储在一组巨大的编号架上。没有人能够直接知道一个对象被存储在哪个架子上。相反，我们给 Clojure 一个标识符，它用来检索该对象。

为了使之成功，Clojure 必须维护我们的标识符和货架地址之间的关联。它通过使用_namespaces_来做到这一点。命名空间包含了人类友好的_符号_和书架地址的引用之间的 Map，被称为_vars_，很像卡片目录。

从技术上讲，命名空间是 "clojure.lang.Namespace "类型的对象，你可以与它们互动，就像你可以与 Clojure 数据结构互动一样。例如，你可以用`*ns*`来引用当前的命名空间，你可以用`(ns-name *ns*)`来获得其名称。

```
(ns-name *ns*)
; => user
```

例如，当你启动 REPL 时，你在`user`命名空间中（正如你在这里看到的）。提示符显示当前名称空间，使用`user=>`。

当前名字空间的概念意味着你可以有多个名字空间，事实上 Clojure 允许你创建任意多的名字空间（尽管从技术上讲，你可以创建的名字数量可能有一个上限）。在 Clojure 程序中，你总是_在_个命名空间中。

至于符号，你一直在使用它们，甚至没有意识到。例如，当你写`(map inc [1 2])`时，`map`和`inc`都是符号。符号是 Clojure 中的数据类型，我将在下一章中彻底解释它们。现在，你需要知道的是，当你给 Clojure 一个像`map`这样的符号时，它会在当前命名空间中找到相应的 var，得到一个架子上的地址，并为你从那个架子上检索一个对象--在这里，就是`map`所指的那个函数。如果你想只使用符号本身，而不是它所指的东西，你必须引用它。引述任何 Clojure 的形式告诉 Clojure 不要求值它，而是把它当作数据。接下来的几个例子显示了当你引用一个 Form 时会发生什么。

```
➊ inc
; => #<core$inc clojure.core$inc@30132014>

➋ 'inc
; => inc

➌ (map inc [1 2])
; => (2 3)

➍ '(map inc [1 2])
; => (map inc [1 2])
```

当你在 REPL 中求值 `inc` 在 ➊ 处时，它会打印出 `inc` 所指的函数的文本表述。接下来，你在➋引用`inc`，所以结果是符号`inc`。然后，你在➌处求值一个熟悉的`map`应用程序，得到一个熟悉的结果。之后，你在➍处引用整个列表数据结构，结果是一个未求值的列表，包括`map`符号、`inc`符号和一个 Vector。

现在你知道了 Clojure 的组织系统，让我们来看看如何使用它。

## 用 def 存储对象

Clojure 中用于存储对象的主要工具是`def`。其他的工具，如`defn`，都是使用`def`。下面是一个 def 的应用实例。

```
(def great-books ["East of Eden" "The Glass Bead Game"])
; => #'user/great-books

great-books
; => ["East of Eden" "The Glass Bead Game"]
```

这段代码告诉 Clojure。

1. 用`great-books`和 var 之间的关联更新当前命名空间的 Map。
2. 找到一个空闲的存储架。
3. 将`[《伊甸园之东》《玻璃珠游戏》]`存放在架子上。
4. 将书架的地址写在 var 上。
5. 5.返回 var（在这个例子中，\`#'user/great-books'）。

这个过程被称为_interning_一个 var。 你可以使用\`ns-interns'与命名空间的符号到内含变量的 Map 进行交互。下面是你如何获得一个内部变量的 Map。

```
(ns-interns *ns*)
; => {great-books #'user/great-books}.
```

你可以使用\`get'函数来获取一个特定的 var。

```
(get (ns-interns *ns*) ' great-books)
; => #'user/great-books
```

通过求值`(`ns-map _ns_)`,`你也可以得到命名空间在给定一个符号时用来查找 var 的完整 Map。`(ns-map *ns*)`给你一个非常大的 Map，我不会在这里打印，但可以试试

`#'user/great-books'是var的*读者形式。 我将在第七章解释更多关于读者形式。现在，只需知道你可以使用`#''来抓取与后面的符号对应的 var；`#'user/great-books'让你在`user'命名空间中使用与符号`great-books'相关的var。我们可以`deref'变量来获得它们所指向的对象。

```
(deref #'user/great-books)
; => ["East of Eden" "The Glass Bead Game"]
```

这就像告诉 Clojure，"从 var 中获取书架号，去那个书架号，抓住上面的东西，然后给我！"

但是通常情况下，你只需要使用符号。

```
great-books
; => ["East of Eden" "The Glass Bead Game"]
```

这就像告诉 Clojure，"检索与 great-books 相关的 var，然后取消那个坏的杰克逊"。

到目前为止还不错，对吗？好吧，请做好准备，因为这个田园诗般的组织天堂即将被颠覆 用同样的符号再次调用`def`。

```
(def great-books ["The Power of Bees" "Journey to Upstairs"])
great-books
; => ["The Power of Bees" "Journey to Upstairs"]
```

![img](https://www.braveclojure.com/assets/images/cftbat/organization/bee-power.png)

var 已经更新了新的 Vector 的地址。这就像你在卡片目录中的卡片上的地址用了白笔，然后写了一个新地址。其结果是，你不能再要求 Clojure 找到第一个 Vector。这被称为_名称碰撞_。混乱! 无政府状态!

你可能在其他编程语言中经历过这种情况。JavaScript 在这方面是臭名昭著的，它也发生在 Ruby 中。这是个问题，因为你可能无意中覆盖了你自己的代码，而且你也不能保证第三方库不会覆盖你的代码。Melvil 惊恐地退缩了! 幸运的是，Clojure 允许你创建任意多的命名空间，这样你就可以避免这些碰撞。

## 创建和切换到命名空间

Clojure 有三种创建命名空间的工具：函数`create-ns`，函数`in-ns`，以及宏`ns`。你将在你的 Clojure 文件中主要使用`ns`宏，但我将推迟几页来解释它，因为它结合了许多工具，而且在我讨论其他工具之后，它更容易理解。

`create-ns`接收一个符号，如果它不存在，就用这个名字创建一个命名空间，并返回这个命名空间。

```
user=> (create-ns 'cheese.taxonomy)
; => #<Namespace cheese.taxonomy>
```

你可以使用返回的名字空间作为函数调用的参数。

```
user=> (ns-name (create-ns 'cheese.taxonomy))
; => cheese-taxonomy
```

在实践中，你可能永远不会在你的代码中使用`create-ns`，因为创建一个命名空间而不移入它并不是非常有用。使用`in-ns`更常见，因为如果命名空间不存在，它会创建命名空间，并\*\*切换到它，如清单 6-1 所示。

```
user=> (in-ns 'cheese.analysis)
; => #<Namespace cheese.analysis>
```

1. 6-1. 使用 in-ns 创建一个命名空间并切换到该空间

注意你的 REPL 提示符现在是`cheese.analysis>`，表明你确实在你刚刚创建的新命名空间中。现在当你使用`def`时，它将在`cheese.analysis`命名空间中存储命名对象。

但是如果你想使用其他命名空间的函数和数据怎么办？要做到这一点，你可以使用一个_完全合格的_符号。一般的形式是 namespace`/`name。

```
cheese.analysis=> (in-ns 'cheese.taxonomy)
cheese.taxonomy=> (def cheddars ["mild" "medium" "strong" "sharp" "extra sharp"])
cheese.taxonomy=> (in-ns 'cheese.analysis)

cheese.analysis=> cheddars
; => Exception: Unable to resolve symbol: cheddars in this context
```

这创建了一个新的命名空间，`cheese.taxonomy`，在该命名空间中定义了`cheddars`，然后切换回`cheese.analysis`命名空间。如果你试图在`cheese.analysis`中引用`cheese.taxonomy`命名空间的`cheddars`，你会得到一个异常，但是使用完全合格的符号可以。

```
cheese.analysis=> cheese.taxonomy/cheddars
; => ["mild" "medium" "strong" "sharp" "extra sharp"]
```

输入这些完全合格的符号很快就会成为一种困扰。 比如说，我是一个极不耐烦的学者，专门研究符号学-au-fromage，或者研究与奶酪有关的符号。

突然间，可能发生的最糟糕的事情发生了！在全世界范围内，神圣的和有可能发生的事情都发生了。在世界各地，神圣的、具有历史意义的奶酪都失踪了。威斯康星州的标准切达干酪：不见了! 图坦卡蒙的大奶酪罐：被偷了! 都灵奶酪：被骗取的奶酪所取代! 这有可能使世界因某种原因而陷入完全的混乱! 自然，作为一个杰出的奶酪研究者，我有责任解开这个谜团。与此同时，我正被光明会、共济会和足部族追捕！因为我是一名学者，所以我必须为他们提供帮助。

因为我是一个学者，我试图用我知道的最好的方式来解决这个谜团--去图书馆研究这个狗屎。我可靠的助手 Clojure 陪着我。当我们从一个名字空间到另一个名字空间忙忙碌碌时，我喊着让 Clojure 把一个又一个东西交给我。

但 Clojure 有点笨，很难弄清楚我指的是什么。在`user`命名空间中，我大声说："`join`! 给我`join'！"--我嘴里的唾沫星子飞了出来。"`RuntimeException: Unable to resolve symbol: join`," Clojure抱怨着回应。"看在布里的份上，把`clojure.string/join\`交给我吧！" 我反驳道，Clojure 尽职尽责地把我要找的函数交给我。

我的声音变得沙哑了。我需要一些方法来告诉 Clojure 要给我什么对象，而不必每次都使用完全合格的符号。

幸运的是，Clojure 提供了 "refer "和 "alias "工具，让我可以更简洁地对它吼叫。

### 引用

`refer`使你能够精细地控制你如何引用其他命名空间的对象。启动一个新的 REPL 会话并尝试以下操作。请记住，在 REPL 中这样玩命名空间是可以的，但你不希望你的 Clojure 文件看起来像这样；正确的文件结构方式在["真正的项目组织 "第 133 页](https://www.braveclojure.com/organization/#Anchor)中涉及。

```
user=> (in-ns 'cheese.taxonomy)
cheese.taxonomy=> (def cheddars ["mild" "medium" "strong" "sharp" "extra sharp"])
cheese.taxonomy=> (def bries ["Wisconsin" "Somerset" "Brie de Meaux" "Brie de Melun"])
cheese.taxonomy=> (in-ns 'cheese.analysis)
cheese.analysis=> (clojure.core/refer 'cheese.taxonomy)
cheese.analysis=> bries
; => ["Wisconsin" "Somerset" "Brie de Meaux" "Brie de Melun"]

cheese.analysis=> cheddars
; => ["mild" "medium" "strong" "sharp" "extra sharp"]
```

这段代码创建了一个 "cheese.taxonomy "命名空间和其中的两个 Vector。 `cheddars`和`bries`。然后它创建并移动到一个新的命名空间，称为`cheese.analysis`。用命名空间的符号调用`refer`可以让你引用相应的命名空间的对象，而不需要使用完全限定的符号。它通过更新当前命名空间的符号/对象 Map 来实现这一目的。你可以看到像这样的新条目。

```
cheese.analysis=> (clojure.core/get (clojure.core/ns-map clojure.core/*ns*) 'bries)
; => #'cheese.taxonomy/bries

cheese.analysis=> (clojure.core/get (clojure.core/ns-map clojure.core/*ns*) 'cheddars)
; => #'cheese.taxonomy/cheddars
```

这就好像 Clojure

1. 在`cheese.taxonomy`命名空间上调用`ns-interns`。
2. 将其与当前命名空间的`ns-map`合并。
3. 将结果作为当前命名空间的新的\`ns-map'。

当你调用`refer`时，你也可以把过滤器`:only`, `:exclude`, 和`:rename`传递给它。正如名字所暗示的，`:only`和`:exclude`限制了哪些符号/变量 Map 被合并到当前命名空间的`ns-map`。 `:rename`允许你使用不同的符号来表示被合并的变量。如果我们将前面的例子修改为使用`:only`，会发生以下情况。

```
cheese.analysis=> (clojure.core/refer 'cheese.taxonomy :only ['bries])
cheese.analysis=> bries
; => ["Wisconsin" "Somerset" "Brie de Meaux" "Brie de Melun"]
cheese.analysis=> cheddars 
; => RuntimeException: 无法解决符号：cheddars
```

下面是`:exclude`的操作。

```
cheese.analysis=> (clojure.core/refer 'cheese.taxonomy :only ['bries])
cheese.analysis=> bries
; => ["Wisconsin" "Somerset" "Brie de Meaux" "Brie de Melun"]
cheese.analysis=> cheddars 
; => RuntimeException: Unable to resolve symbol: cheddars
```

最后，一个`:rename`的例子。

```
cheese.analysis=> (clojure.core/refer 'cheese.taxonomy :rename {'bries 'yummy-bries})
cheese.analysis=> bries
; => RuntimeException: Unable to resolve symbol: bries
cheese.analysis=> yummy-bries
; => ["Wisconsin" "Somerset" "Brie de Meaux" "Brie de Melun"]
```

注意，在这些最后的例子中，我们必须使用`clojure.core`中所有对象的完全合格名称，如`clojure.core/ns-map`和`clojure.core/refer`。我们不需要在`user`命名空间中这样做。这是因为 REPL 在`user`命名空间中自动引用`clojure.core`。当你创建一个新的命名空间时，你可以通过求值`(clojure.core/refer-clojure)`来简化你的生活；这将引用 clojure.core 命名空间，从现在起我将使用它。在例子中你不会看到`clojure.core/refer`，而只会看到`refer`。

另一件需要注意的事情是，你可以完全自由地组织你的函数和数据，跨越命名空间。这让你可以合理地将相关的函数和数据归入同一命名空间。

有时你可能希望一个函数只对同一命名空间内的其他函数有效。Clojure 允许你使用`defn-`来定义_私有_的函数。

```
(in-ns 'cheese.analysis)
;; Notice the dash after "defn"
(defn- private-function
  "Just an example function that does nothing"
  [])
```

如果你试图从其他命名空间调用这个函数或引用它，Clojure 将抛出一个异常。你可以在求值➊和➋的代码时看到这一点。

```
cheese.analysis=> (in-ns 'cheese.taxonomy)
cheese.taxonomy=> (clojure.core/refer-clojure)
➊ cheese.taxonomy=> (cheese.analysis/private-function)
➋ cheese.taxonomy=> (refer 'cheese.analysis :only ['private-function])
```

正如你所看到的，即使你明确地 "引用 "这个函数，你也不能使用其他命名空间的函数，因为你把它变成了私有的。(如果你想狡猾一点，你仍然可以使用神秘的语法\`@#'some/private-var'来访问私有变量，但你很少想这样做)。

### alias

与`refer`相比，`alias`相对简单。它所做的只是让你缩短一个命名空间的名称，以便使用完全合格的符号。

```
cheese.analysis=> (clojure.core/alias 'taxonomy 'cheese.taxonomy)
cheese.analysis=> taxonomy/bries
; => ["Wisconsin" "Somerset" "Brie de Meaux" "Brie de Melun"]
```

这段代码让我们使用来自`cheese.taxonomy`命名空间的调用符号，并使用较短的别名`taxonomy`。

`refer`和`alias`是你引用当前命名空间以外的对象的两个基本工具! 它们是 REPL 开发的好帮手。

然而，你不可能在 REPL 中创建整个程序。在下一节中，我将介绍你需要知道的一切，以组织一个真正的项目，使源代码在文件系统中生存。

## 真正的项目组织

现在我已经介绍了 Clojure 组织系统的构建模块，我将向你展示如何在实际项目中使用它们。我将讨论文件路径和命名空间名称之间的关系，解释如何用`require`和`use`加载文件，并展示如何使用`ns`来设置一个命名空间。

### 文件路径和命名空间名称之间的关系

为了一石二鸟（或者用一颗种子喂养两只鸟，这取决于你是多么的嬉皮士），我将介绍更多关于命名空间的内容，同时我们将通过绘制国际奶酪大盗的抢劫地点来抓捕这个讨厌的大盗。运行以下程序。

```
lein new app the-divine-cheese-code
```

这应该创建一个目录结构，看起来像这样。

```
| .gitignore
| doc
| | intro.md
| project.clj
| README.md
| resources
| src
| | the_divine_cheese_code
| | | core.clj
| test
| | the_divine_cheese_code
| | | core_test.clj
```

现在，打开_src/the\_divine\_cheese\_code/core.clj_。你应该在第一行看到这个。

```
(ns the-divine-cheese-code.core
  (:gen-class))
```

`ns`是在 Clojure 中创建和管理命名空间的主要方式。我很快就会对它进行全面的解释。不过现在，只需知道这一行与我们在清单 6-1 中使用的`in-ns`函数非常相似。如果一个命名空间不存在，它就创建一个命名空间，然后切换到它。我在第 12 章也详细介绍了`(:gen-class)`。

命名空间的名字是`the-divine-cheese-code.core`。在 Clojure 中，命名空间的名称和声明命名空间的文件路径之间有一个一对一的 Map，根据以下约定。

* 当你用`lein`创建一个目录时（就像你在这里做的那样），源代码的根默认为_src_。
* 名称空间中的破折号对应于文件系统中的下划线。所以`the-divine-cheese-code`在文件系统中被 Map 为_the\_divine\_cheese\_code_。
* 命名空间名称中的句号（`.`）前面的成分对应于一个目录。例如，由于`the-divine-cheese-code.core`是命名空间的名称，_the\_divine\_cheese\_code_是一个目录。
* 命名空间的最后一个组成部分对应于扩展名为\*.clj_的文件；`core`被 Map 到_core.clj\*。

你的项目将有一个命名空间，`the-divine-cheese-code.visualization.svg`。现在继续为它创建文件。

```
mkdir src/the_divine_cheese_code/visualization
touch src/the_divine_cheese_code/visualization/svg.clj
```

注意，文件系统的路径遵循这些惯例。有了命名空间和文件系统之间的关系，我们来看看`require`和`use`。

### 要求和使用命名空间

在`the-divine-cheese-code.core`命名空间的代码将使用`the-divine-cheese-code.visualization.svg`命名空间的函数来创建 SVG 标记。为了使用`svg`的函数，`core`将不得不_要求它。但首先，让我们在_svg.clj\*中添加一些代码。让它看起来像这样（你以后会添加更多）。

```
(ns the-divine-cheese-code.visualization.svg)

(defn latlng->point
  "Convert lat/lng map to comma-separated string" 
  [latlng]
  (str (:lat latlng) "," (:lng latlng)))

(defn points
  [locations]
  (clojure.string/join " " (map latlng->point locations)))
```

这定义了两个函数，`latlng->point`和`points`，你将用它们来把一串经纬度坐标转换成一串点。 要使用_core.clj_文件中的这段代码，你必须`require`它。`require`接收一个指定命名空间的符号，并确保该命名空间存在并准备使用；在这种情况下，当你调用`(require 'the-divine-cheese`-code.visualization.svg)`，Clojure读取并求值相应的文件。通过求值该文件，它创建了`the-divine-cheese-code.visualization.svg`命名空间，并在该命名空间中定义了函数`latlng->point`和`points\`。即使文件_svg.clj_在你的项目目录中，Clojure 在运行你的项目时也不会自动求值它；你必须明确告诉 Clojure 你想使用它。

在要求命名空间之后，你可以_参考_它，这样你就不必使用完全合格的名称来引用函数。继续要求`the-divine-cheese-code.visualization.svg`，并添加`heists`序列，使_core.clj_与列表相符。

```
(ns the-divine-cheese-code.core)
;; Ensure that the SVG code is evaluated
(require 'the-divine-cheese-code.visualization.svg)
;; Refer the namespace so that you don't have to use the 
;; fully qualified name to reference svg functions
(refer 'the-divine-cheese-code.visualization.svg)

(def heists [{:location "Cologne, Germany"
              :cheese-name "Archbishop Hildebold's Cheese Pretzel"
              :lat 50.95
              :lng 6.97}
             {:location "Zurich, Switzerland"
              :cheese-name "The Standard Emmental"
              :lat 47.37
              :lng 8.55}
             {:location "Marseille, France"
              :cheese-name "Le Fromage de Cosquer"
              :lat 43.30
              :lng 5.37}
             {:location "Zurich, Switzerland"
              :cheese-name "The Lesser Emmental"
              :lat 47.37
              :lng 8.55}
             {:location "Vatican City"
              :cheese-name "The Cheese of Turin"
              :lat 41.90
              :lng 12.45}])

(defn -main
  [& args]
  (println (points heists)))
```

现在你有一连串的 heist 位置可以使用，你可以使用`visualization.svg`命名空间的函数。`main`函数只是将`points`函数应用于`heists`。如果你用`lein run`运行该项目，你应该看到这个。

```
50.95,6.97 47.37,8.55 43.3,5.37 47.37,8.55 41.9,12.45
```

万岁! 你离抓到那个偷窃发酵乳的人又近了一步! 使用`require`成功加载了`the-divine-cheese-code.visualization.svg`以供使用。

`require`的细节实际上有点复杂，但为了实用，你可以认为`require`是告诉 Clojure 以下内容。

1. 如果你已经用这个符号（`the-divine-cheese-code.visualization.svg`）调用了`require`，则不做任何事情。
2. 否则，使用["文件路径和命名空间名称之间的关系 "第 133 页](https://www.braveclojure.com/organization/#Anchor-3)中描述的规则找到与该符号对应的文件。在这种情况下，Clojure 找到`src/the_divine_cheese_code/visualization/svg.clj`。

读取并求值该文件的内容。Clojure 希望该文件声明一个与它的路径相对应的命名空间（我们的文件就是如此）。

`require`也可以让你在需要一个命名空间时使用`:as`或`alias`来别名它。这样。

```
(require '[the-divine-cheese-code.visualization.svg :as svg] )
```

相当于这样。

```
(require 'the-divine-cheese-code.visualization.svg)
(alias 'svg 'the-divine-cheese-code.visualization.svg)
```

现在你可以使用别名的命名空间了。

```
(svg/points heists)
; => "50.95,6.97 47.37,8.55 43.3,5.37 47.37,8.55 41.9,12.45"
```

Clojure 提供了另一种捷径。函数`use`不需要单独调用`require`和`refer`，而是同时调用。在生产代码中使用`use`是不可取的，但当你在 REPL 中做实验，想快速获得一些函数时，它就很方便。例如，这个。

```
(require 'the-divine-cheese-code.visualization.svg)
(refer 'the-divine-cheese-code.visualization.svg)
```

相当于这样。

```
(use 'the-divine-cheese-code.visualization.svg)
```

你可以用`use`来别名一个命名空间，就像你可以用`require`一样。这样。

```
(require 'the-divine-cheese-code.visualization.svg)
(refer 'the-divine-cheese-code.visualization.svg)
(alias 'svg 'the-divine-cheese-code.visualization.svg)
```

相当于清单 6-2 中的代码，其中也显示了函数调用中使用的别名空间。

```
(use '[the-divine-cheese-code.visualization.svg :as svg])
(= svg/points points)
; => true

(= svg/latlng->point latlng->point)
; => true
```

1. 6-2. 有时，既使用又别名一个命名空间是很方便的。

在这里用`use`别名命名空间似乎是多余的，因为`use`已经引用了命名空间（这让你可以简单地调用`points`而不是`svg/points`）。但在某些情况下，这很方便，因为`use`和`refer`有相同的选项（`:only`, `:exclude`, `:as`, 和`:rename`）。当你跳过引用一个符号时，你可能想用`use`来别名一个命名空间。你可以这样使用。

```
(require 'the-divine-cheese-code.visualization.svg)
(refer 'the-divine-cheese-code.visualization.svg :as :only ['point])
```

或者你可以使用清单 6-3 中的`use`形式（其中还包括如何调用函数的例子）。

```
(use '[the-divine-cheese-code.visualization.svg :as svg :only [points]])
(refer 'the-divine-cheese-code.visualization.svg :as :only ['points])
(= svg/points points)
; => true

;; We can use the alias to reach latlng->point
svg/latlng->point
; This doesn't throw an exception

;; But we can't use the bare name
latlng->point
; This does throw an exception!
```

1. 在你使用一个命名空间后将其别名化，可以让你参考你排除的符号。

如果你在 REPL 中尝试清单 6-3，并且`latlng->point`没有抛出一个异常，这是因为你在清单 6-2 中引用了`latlng->point`。你需要重新启动你的 REPL 会话，使代码表现得如清单 6-3 所示。

这里的启示是，`require`和`use`加载文件，并可选择`alias`或`refer`其命名空间。当你写 Clojure 程序和阅读别人写的代码时，你可能会遇到更多的`require'和`use'的写法，这时，阅读 Clojure 的 API 文档（[_http://clojure.org/libs/_](http://clojure.org/libs/)）来了解发生了什么是有意义的。然而，到目前为止，你所学到的关于`require`和`use`的内容应该能满足你 95.3%的需求。

### ＃＃＃NS 宏

现在是时候看看`ns`宏了。到目前为止所涉及的工具--`in-ns`, `refer`, `alias`, `require`, 和 `use`--最常在你使用 REPL 时使用。在你的源代码文件中，你通常会使用`ns`宏，因为它允许你简洁地使用迄今为止描述的工具，并提供其他有用的功能。在本节中，你将了解一个`ns`调用如何结合`require`、`use`、`in-ns`、`alias`和`refer`。

`ns`做的一个有用的任务是默认引用`clojure.core`命名空间。这就是为什么你可以从`the-divine-cheese-code.core`中调用`println`，而不使用完全限定的名称`clojure.core/println`。

你可以用`:refer-clojure`来控制从`clojure-core`引用的内容，它的选项与`refer`相同。

```
(ns the-divine-cheese-code.core
  (:refer-clojure :exclude [println])
```

如果你在_divine\_cheese\_code.core.clj_的开头调用这个，会破坏你的代码，迫使你在`-main'函数中使用`clojure.core/println'。

在`ns`中，`(:`refer-clojure)\`的形式被称为_reference_。这对你来说可能看起来很奇怪。这个引用是一个函数调用？一个宏？它是什么？你将在第 7 章中了解更多关于底层机器的知识。现在，你只需要了解每个引用如何 Map 到函数调用。例如，前面的代码就相当于这样。

```
(in-ns 'the-divine-cheese-code.core)
(refer 'clojure.core :exclude ['println])
```

在 "ns "中，有六种可能的引用。

* `(:refer-clojure)`.
* "(:require)"。
* `(:use)`
* `(:import)`
* `(:load)`
* `(:gen-class)`。

`(:import)`和`(:gen-class)`将在第 12 章介绍。我将不介绍`(:load)`，因为它很少被使用。

`(:require)`的工作方式很像`require`函数。例如，这样。

```
(ns the-divine-cheese-code.core
  (:require the-divine-cheese-code.visualization.svg))
```

相当于这样。

```
(in-ns 'the-divine-cheese-code.core)
(require 'the-divine-cheese-code.visualization.svg)
```

注意，在 "ns "形式中（与 "in-ns "函数调用不同），你不需要用"''来引用你的符号。在 "ns "中，你从来不需要引用符号。

你也可以`alias`一个你在`ns`内`require`的库，就像你调用函数时一样。这样。

```
(ns the-divine-cheese-code.core
  (:require [the-divine-cheese-code.visualization.svg :as svg])
```

相当于这样。

```
(in-ns 'the-divine-cheese-code.core)
(require ['the-divine-cheese-code.visualization.svg :as 'svg])
```

你可以在一个`(:require)`引用中要求多个库，如下所示。 这样。

```
(ns the-divine-cheese-code.core
  (:require [the-divine-cheese-code.visualization.svg :as svg])
            [clojure.java.browse :as browse]))
```

相当于这样。

```
(in-ns 'the-divine-cheese-code.core)
(require ['the-divine-cheese-code.visualization.svg :as 'svg])
(require ['clojure.java.browse :as 'browse])
```

然而，`(:require)`引用和`require`函数之间的一个区别是，引用也允许你引用名字。这一点。

```
(ns the-divine-cheese-code.core
  (:require [the-divine-cheese-code.visualization.svg :refer [point]))
```

相当于这样。

```
(in-ns 'the-divine-cheese-code.core)
(require 'the-divine-cheese-code.visualization.svg)
(refer 'the-divine-cheese-code.visualization.svg :only ['point])
```

你也可以引用所有的符号（注意`:all`关键字）。

```
(ns the-divine-cheese-code.core
  (:require [the-divine-cheese-code.visualization.svg :refer :all]))
```

这就相当于这样做了。

```
(in-ns 'the-divine-cheese-code.core)
(require 'the-divine-cheese-code.visualization.svg)
(refer 'the-divine-cheese-code.visualization.svg)
```

这是要求代码、别名命名空间和引用符号的首选方式。建议你不要使用`(:use)`，但由于你很可能会遇到它，所以知道它是如何工作的很好。你知道该怎么做。这个。

```
(ns the-divine-cheese-code.core
  (:use clojure.java.browse))
```

这样做。

```
(in-ns 'the-divine-cheese-code.core)
(use 'clojure.java.browse)
```

而这一点。

```
(ns the-divine-cheese-code.core
  (:use [clojure.java browse io])
```

这样做。

```
(in-ns 'the-divine-cheese-code.core)
(use 'clojure.java.browse)
(use 'clojure.java.io)
```

注意，当你在`:use`后面加上一个 Vector 时，它把第一个符号作为_base_，然后用后面的每个符号调用`use`。

哦，我的天哪，就是这样! 现在你可以像专家一样使用`ns`了! 你需要这样做，该死的，因为那个_voleur des fromages_（他们可能在法语中这样说）仍然在肆意妄为。还记得他/她吗？

∮∮抓小偷

我们不能让这个掠夺帕尔马干酪的人带着更多的干酪离开！是时候完成根据坐标画线的工作了。现在是时候根据盗窃案的坐标来完成画线了！这肯定会发现一些问题。这肯定会发现一些问题!

使用每个抢劫案的纬度坐标，你将在一个 SVG 图像中连接这些点。但是，如果你用给定的坐标画线，结果看起来就不对了，原因有二。首先，纬度坐标是由南向北上升的，而 SVG 的 Y 坐标是由上向下上升的。换句话说，你需要翻转坐标，否则绘图就会颠倒过来。

第二，绘图会非常小。为了解决这个问题，你将通过平移和缩放来放大它。这就像把一张看起来像图 6-1a 的图变成图 6-1b。

![](https://www.braveclojure.com/assets/images/cftbat/organization/svg-before.png) ![](https://www.braveclojure.com/assets/images/cftbat/organization/svg-after.png)

图 6-1：通过翻转、平移和缩放纬度坐标来制作一张 SVG 图片。

说实话，这些都是完全随意的，它已经与代码组织没有直接关系了，但是它很有趣，我想你会有一个很好的时间来浏览这些代码的 使你的_svg.clj_文件与清单 6-4 一致。

```
(ns the-divine-cheese-code.visualization.svg
  (:require [clojure.string :as s])
  (:refer-clojure :exclude [min max])

➊ （defn comparator-over-maps
  [比较-fn ks］
  (fn [maps]
➋ (zipmap ks
➌ (map (fn [k] (apply comparison-fn (map k maps)))
                 ks))))

➍ (def min (comparator-over-maps clojure.core/min [:lat :lng])
(def max (comparator-over-maps clojure.core/max [:lat :lng]))
```

1. 6-3. 构建 Map 比较函数

你在➊处定义了`comparator-over-maps`函数。这可能是最棘手的部分，所以请忍受一下。 `comparator-over-maps`是一个返回一个函数的函数。返回的函数使用所提供的比较函数`comparison-fn`对参数`ks`提供的键值进行比较。

你使用`comparator-over-map`来构造`min`和`max`函数➍，你将用它们来寻找我们图形的左上角和右下角。下面是\`min'的操作。

```
(min [{:a 1 :b 3} {:a 5 :b 0}] )
; => {:a 1 :b 0}
```

当你调用`min`时，它调用`zipmap`，它接受两个参数，都是 seq，并返回一个新的 map。第一个序列的元素成为键，第二个序列的元素成为值。

```
(zipmap [:a :b] [1 2])
; => {:a 1 :b 2}。
```

在 ，`zipmap`的第一个参数是`ks`，所以`ks`的元素将是返回 Map 的键。第二个参数是在➌的 Map 调用的结果。那个 Map 调用实际上是在进行比较。

最后，在➍，你使用`comparator-over-maps`来创建比较函数。如果你把图纸看作是刻在一个矩形里，那么`min`是矩形中最接近（0，0）的角，`max`是离它最远的角。

下面是代码的下一部分。

```
 (defn translate-to-00
  [locations]
  (let [mincoords (min locations)]
    (map #(merge-with - % mincoords) locations)))

 (defn scale
  [width height locations]
  (let [maxcoords (max locations)
        ratio {:lat (/ height (:lat maxcoords))
               :lng (/ width (:lng maxcoords))}]
    (map #(merge-with * % ratio) locations)))
```

`translate-to-00`，定义在 ，工作原理是找到我们位置的`min'，然后从每个位置减去这个值。它使用`merge-with\`，其工作原理如下。

```
(merge-with - {:lat 50 :lng 10} {:lat 5 :lng 5})
; => {:lat 45 :lng 5}
```

然后我们定义函数`scale`，它将每个点乘以最大经纬度与所需高度和宽度之间的比率。

下面是_svg.clj_的其余代码。

```
(defn latlng->point
  "Convert lat/lng map to comma-separated string" 
  [latlng]
  (str (:lat latlng) "," (:lng latlng)))

(defn points
  "Given a seq of lat/lng maps, return string of points joined by space"
  [locations]
  (s/join " " (map latlng->point locations)))

(defn line
  [points]
  (str "<polyline points=\"" points "\" />"))

(defn transform
  "Just chains other functions"
  [width height locations]
  (->> locations
       translate-to-00
       (scale width height)))

(defn xml
  "svg 'template', which also flips the coordinate system"
  [width height locations]
  (str "<svg height=\"" height "\" width=\"" width "\">"
       ;; These two <g> tags change the coordinate system so that
       ;; 0,0 is in the lower-left corner, instead of SVG's default
       ;; upper-left corner
       "<g transform=\"translate(0," height ")\">"
       "<g transform=\"rotate(-90)\">"
       (-> (transform width height locations)
           points
           line)
       "</g></g>"
       "</svg>"))
```

这里的函数非常简单明了。它们只是接收`{:lat x :lng y}`Map，并对其进行转换，以便创建一个 SVG。`latlng->point`返回一个字符串，可用于在 SVG 标记中定义一个点。`points`将`lat`/`lng`Map 的序列转换为一个以空格分隔的点的字符串。 `line`返回连接所有给定空间分隔的点字符串的 SVG 标记。 `transform`接收一个位置序列，将它们翻译成从(0, 0)开始的点，并将它们缩放到给定的宽度和高度。最后，`xml`产生标记，用 SVG 显示给定的位置。

有了_svg.clj_的所有代码，现在让_core.clj_看起来像这样。

```
(ns the-divine-cheese-code.core
  (:require [clojure.java.browse :as browse]
            [the-divine-cheese-code.visualization.svg :refer [xml]])
  (:gen-class))

(def heists [{:location "Cologne, Germany"
              :cheese-name "Archbishop Hildebold's Cheese Pretzel"
              :lat 50.95
              :lng 6.97}
             {:location "Zurich, Switzerland"
              :cheese-name "The Standard Emmental"
              :lat 47.37
              :lng 8.55}
             {:location "Marseille, France"
              :cheese-name "Le Fromage de Cosquer"
              :lat 43.30
              :lng 5.37}
             {:location "Zurich, Switzerland"
              :cheese-name "The Lesser Emmental"
              :lat 47.37
              :lng 8.55}
             {:location "Vatican City"
              :cheese-name "The Cheese of Turin"
              :lat 41.90
              :lng 12.45}])

(defn url
  [filename]
  (str "file:///"
       (System/getProperty "user.dir")
       "/"
       filename))

(defn template
  [contents]
  (str "<style>polyline { fill:none; stroke:#5881d8; stroke-width:3}</style>"
       contents))

(defn -main
  [& args]
  (let [filename "map.html"]
    (->> heists
         (xml 50 100)
         template
         (spit filename))
    (browse/browse-url (url filename))))
```

这里没有太复杂的事情发生。在 "main "中，你使用 "xml "和 "template "函数建立绘图，用 "spit "将绘图写入一个文件，然后用 "browse/browse-url "打开它。你现在应该试试! 运行`lein run`，你会看到类似图 6-2 的东西。

图 6-2: 抢劫模式的最终 SVG!

等一下 ……这看起来很像 ……这看起来很像一个 lambda。Clojure 的标志是一个 lambda . ……哦，我的天啊! Clojure，一直以来都是你!

## 总结

在本章中你学到了很多东西。在这一点上，你应该拥有所有你需要的工具来开始组织你的项目。你现在知道命名空间组织了符号和 vars 之间的 Map，vars 是对 Clojure 对象（数据结构、函数等）的引用。 `def`存储一个对象，并用符号和指向该对象的 var 之间的 Map 来更新当前命名空间。你可以用`defn-`创建私有函数。

Clojure 允许你用`create-ns`创建命名空间，但通常使用`in-ns`更有用，它也会切换到命名空间。你可能只在 REPL 中使用这些函数。当你在 REPL 中时，你总是_在_当前命名空间中。当你在文件中而不是在 REPL 中定义名字空间时，你应该使用 `ns` 宏，名字空间和它在文件系统中的路径之间是一对一的关系。

你可以通过使用完全限定的名称来引用其他命名空间中的对象，如`cheese.taxonomy/cheddars`。 `refer`可以让你使用其他命名空间的名字，而不需要完全限定它们，`alias`可以让你在写出完全限定的名字时，使用一个更短的名字来命名空间。

`require`和`use`确保一个名字空间的存在并准备好被使用，并且可以选择让你`refer`和`alias`相应的名字空间。你应该使用`ns`在你的源文件中调用`require`和`use`。\*[Clojure ns syntax cheat-sheet - GitHub](https://gist.github.com/ghoseb/287710/)\*是使用`ns`的所有变化的一个很好的参考。

最后，也是最重要的一点，做一个俗气的人并不容易。

![](https://www.braveclojure.com/assets/images/cftbat/organization/cheese.png)
