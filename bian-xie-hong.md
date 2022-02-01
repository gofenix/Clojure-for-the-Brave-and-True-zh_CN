# 编写宏

当我 18 岁时，我在新墨西哥州圣菲的一家酒店找到了一份夜班审计师的工作，每周工作四个晚上，从晚上 11 点到早上 7 点。经过几个月的这种不眠不休的工作，我的情绪有了自己的变化。一天晚上，大约在

凌晨 3 点，我正在看一个信息广告，该产品声称可以恢复男人的头发。当我看到一个曾经秃头的人的故事时，我被真诚的喜悦所淹没。"终于来了！"我的大脑涌动着。"这个人得到了他应得的爱和成功! 多么不可思议的产品，给无望的人以希望！"

从那时起，我发现自己一直在想，我是否能以某种方式重新创造因长期睡眠不足而引起的情感放弃和对生命的欣赏。也许是某种药水--喝上几口，释放我内心的理查德-西蒙斯，但时间不会太长。

![](https://www.braveclojure.com/assets/images/cftbat/writing-macros/simmons-potion.png)

就像药水可以让我暂时改变我的基本性质一样，宏允许你以其他语言无法实现的方式修改 Clojure。有了宏，你可以扩展 Clojure 以适应你的问题空间，建立起语言。

在这一章中，你将彻底研究如何编写宏，从基本的例子开始，逐步提高复杂性。最后，你将戴上你的假想帽，用宏来验证你想象中的在线药水店的客户订单。

在本章结束时，你将了解你用来编写宏的所有工具：引号、语法引号、解引号、解引号拼接（又称皮纳塔工具）和 gensym。你还会了解到对毫无戒心的宏作者来说隐藏着的危险：双重求值、变量捕获和宏感染。

## 宏是必不可少的

在你开始编写宏之前，我想帮助你把它们放在适当的环境中。是的，宏比北极熊的脚趾甲还要酷，但你不应该把宏看成是一些深奥的工具，当你想对你的代码进行额外的花哨处理时，就把它拿出来。事实上，宏允许 Clojure 从一个很小的函数和特殊形式的核心中获得大量的内置功能。以`when`为例。 `when`有这样的一般形式。

```
(when boolean-expression
  expression-1
  expression-2
  expression-3
  ...
  expression-x)
```

你可能认为`when`是一个像`if`一样的特殊形式。那么你猜怎么着？它不是! 在大多数其他语言中，你只能使用特殊的关键字来创建条件表达式，而没有办法创建你自己的条件运算符。然而，`when`实际上是一个宏。

在这个宏扩展中，你可以看到`when`是用`if`和`do`来实现的。

```
(macroexpand '(when boolean-expression
                expression-1
                expression-2
                expression-3))
; => (if boolean-expression
       (do expression-1
           expression-2
           expression-3))
```

这表明宏是 Clojure 开发中不可或缺的一部分--它们甚至被用来提供基本操作。宏并不是为奇特的特殊情况而保留的；你应该把写宏看作是你工具包中的另一个工具。当你学会编写自己的宏时，你会发现它们是如何让你进一步扩展语言，使其适合你的特定问题领域的形状。

## 解剖巨集

巨集定义看起来很像函数定义。它们有一个名称，一个可选的文档字符串，一个参数列表，以及一个主体。主体几乎总是返回一个列表。这是有道理的，因为宏是将数据结构转化为 Clojure 可以求值的形式的一种方式，而 Clojure 使用列表来表示函数调用、特殊形式调用和宏调用。你可以在宏主体中使用任何函数、宏或特殊形式，你调用宏就像调用函数或特殊形式一样。

作为一个例子，这里有我们的老朋友`infix`宏。

```
(defmacro infix
  "Use this macro when you pine for the notation of your childhood"
  [infixed]
  (list (second infixed) (first infixed) (last infixed)))
```

这个宏将一个列表重新排列成正确的 infix 记号顺序。下面是一个例子。

```
(infix (1 + 1))
; => 2
```

函数和宏之间的一个关键区别是，函数参数在传递给函数之前被完全求值，而宏是以未求值的数据形式接收参数。你可以在这个例子中看到这一点。如果你试图单独求值`(1+1)`，你会得到一个异常。然而，因为你在进行一个宏调用，未求值的列表`(1 + 1)`被传递给`infix`。然后宏可以使用`first`、`second`和`last`来重新排列列表，这样 Clojure 就可以求值它。

```
(macroexpand '(infix (1 + 1))
; => (+ 1 1)
```

通过扩展宏，你可以看到`infix`将`(1 + 1)`重新排列成`(+ 1 1)`。很方便!

你也可以在宏定义中使用参数重构，就像你可以使用函数一样。

```
(defmacro infix-2
  [[operand1 op operand2]] (list op operand1 operand2)
  (list op operand1 operand2))
```

解构参数可以让你根据序列参数中的位置简洁地将值与符号绑定。在这里，`infix-2`将一个顺序数据结构作为参数，并按位置进行解构，因此第一个值被命名为`operand1`，第二个值被命名为`op`，第三个值在宏中被命名为`operand2`。

你也可以创建多属性的宏，事实上，基本的布尔运算`and`和`or`都被定义为宏。下面是`and`的源代码。

```
(defmacro and
  "Evaluates exprs one at a time, from left to right. If a form
  returns logical false (nil or false), and returns that value and
  doesn't evaluate any of the other expressions, otherwise it returns
  the value of the last expr. (and) returns true."
  {:added "1.0"}
  ([] true)
  ([x] x)
  ([x & next]
   `(let [and# ~x]
      (if and# (and ~@next) and#))))
```

在这个例子中发生了很多事情，包括符号和`~@`，你很快就会了解到这些。现在重要的是，这里有三个宏体：一个总是返回 "true "的 0-arity 宏体，一个返回操作数的 1-arity 宏体，以及一个递归调用自身的_n_-arity 宏体。这是正确的：宏可以是递归的，它们也可以使用其余的参数（_n_-arity 宏主体中的`& next`），就像函数一样。

现在你对宏的解剖已经很熟悉了，现在是时候把你自己绑在你的奥德修斯式思维的桅杆上，学习写宏体了。

## 为求值建立列表

编写宏就是要为 Clojure 建立一个列表来进行求值，这需要颠覆你的正常思维方式。首先，你经常需要引用表达式，以便在你的最终列表中获得未求值的数据结构（我们稍后会回到这个问题）。更普遍的是，你需要特别注意_符号_和_值_之间的区别。

### 区分符号和值

假设你想创建一个宏，它接收一个表达式，并同时打印和返回其值。(这与`println'不同，`println'总是返回\`nil'。)你希望你的宏能够返回类似这样的列表。

```
(let [result expression]
  (println result)
  result)
```

你的宏的第一个版本可能看起来像这样，使用`list`函数来创建 Clojure 应该求值的列表。

```
 (defmacro my-print-whoopsie
  [expression]
  (list let [result expression]
        (list println result)
        result))
```

然而，如果你尝试这样做，你会得到一个异常\`不能接受一个宏的值。#'clojure.core/let'。这到底是怎么回事？

发生这种情况的原因是，你的宏主体试图获取_符号_ `let`所指的\*值，而你实际想做的是返回`let`符号本身。还有其他的问题：你试图获得`result`的值，这是不绑定的，你试图获得`println`的值，而不是返回其符号。下面是你如何写宏来做你想要的事情。

```
(defmacro my-print
  [expression]
  (list 'let ['result expression]
        (list 'println 'result)
        'result))
```

在这里，你通过在每个符号前加上单引号"''来引出你想作为一个符号使用。这告诉 Clojure_关闭_后面的求值，在这种情况下，防止 Clojure 试图解决这些符号，而只是返回这些符号。使用引号来关闭求值的能力是编写宏的核心，所以让我们给这个主题一个独立的章节。

### 简单的引号

你几乎总是在你的宏中使用引号来获得一个未求值的符号。让我们简单地复习一下引号，然后看看你如何在宏中使用它。

首先，这里是一个没有引号的简单函数调用。

```
(+ 1 2)
; => 3
```

如果我们在开头加上`quote`，它就会返回一个未求值的数据结构。

```
(quote (+ 1 2))
; => (+ 1 2) 
```

这里在返回的列表中，`+`是一个符号。如果我们求值这个加号，就会产生加号函数。

```
+
; => #<core$_PLUS_ clojure.core$_PLUS_@47b36583>
```

而如果我们引用这个加号，它只是产生加号。

```
(quote +)
; => +
```

求值一个未绑定的符号会引发一个异常。

```
sweating-to-the-oldies
; => Unable to resolve symbol: sweating-to-the-oldies in this context
```

但是引用符号会返回一个符号，不管这个符号是否有一个与之相关的值。

```
(quote sweating-to-the-oldies)
; => sweating-to-the-oldies
```

单引号字符是`(quote`x`)`的读者宏。

```
'(+ 1 2)
; => (+ 1 2)

'dr-jekyll-and-richard-simmons
; => dr-jekyll-and-richard-simmons
```

你可以在`when`宏中看到引用的工作。这是`when`的实际源代码。

```
(defmacro when
  "Evaluates test. If logical true, evaluates body in an implicit do."
  {:added "1.0"}
  [test & body]
  (list 'if test (cons 'do body)))
```

注意，宏的定义同时引用了`if`和`do`。这是因为你想让这些符号出现在\`when'返回的最终列表中进行计算。下面是一个返回列表的例子，它可能是这样的。

```
(macroexpand '(when (the-cows-come :home)
                (call me :pappy)
                (slap me :silly)))
; => (if (the-cows-come :home)
       (do (call me :pappy)
           (slap me :silly)))
```

下面是另一个内置宏的源代码的例子，这次是关于`unless`的。

```
(defmacro unless
  "Inverted 'if'"
  [test & branches]
  (conj (reverse branches) test 'if))
```

同样，你必须引用`if`，因为你想让未求值的符号放在结果列表中，就像这样。

```
(macroexpand '(unless (done-been slapped? me)
                      (slap me :silly)
                      (say "I reckon that'll learn me")))
; => (if (done-been slapped? me)
       (say "I reckon that'll learn me")
       (slap me :silly))
```

在许多情况下，在编写宏时，你会使用这样的简单引号，但大多数情况下你会使用更强大的语法引号。

### 语法引用

到目前为止, 你已经看到了通过使用`list`函数来建立列表的宏，以及对列表进行操作的函数，如`first`, `second`, `last`，等等。事实上，你可以这样写宏，直到奶牛回家。但有时，这将导致繁琐和冗长的代码。

语法引号返回未求值的数据结构，与普通引号类似。然而，有两个重要的区别。一个区别是，语法引用将返回_完全合格的_符号（即包括符号的命名空间）。让我们比较一下引号和语法引号。

如果你的代码中不包括名字空间，那么引用就不包括名字空间。

```
'+
; => +
```

写出命名空间，它将被正常的引用所返回。

```
'clojure.core/+
; => clojure.core/+
```

语法引号将总是包括符号的完整命名空间。

```
`+
; => clojure.core/+
```

对一个列表的引用会递归地引用所有的元素。

```
'(+ 1 2)
; => (+ 1 2)
```

语法引用一个列表递归地引用所有的元素。

```
`(+ 1 2)
; => (clojure.core/+ 1 2)
```

语法引号包括名字空间的原因是为了帮助你避免名字的碰撞，这个话题在第 6 章中涉及。

引号和语法引号之间的另一个区别是，后者允许你使用 "tilde"，即"\~"，来\*解除引号的形式。这有点像氪星石：只要超人在氪星石周围，他的能力就会消失。每当在一个语法引号的 Form 中出现 tilde，语法引号返回未求值的、完全命名的 Form 的能力就会消失。这里有一个例子。

```
`(+ 1 ~(inc 1))
; => (clojure.core/+ 1 2)
```

因为它在 tilde 之后，`(inc 1)`被求值而不是被引号。如果没有 unquote，语法引号会返回未求值的形式，并带有完全限定的符号。

```
`(+ 1 (inc 1))
; => (clojure.core/+ 1 (clojure.core/inc 1))
```

如果你熟悉字符串插值，你可以类似地考虑语法引用/非引用的问题。在这两种情况下，你都在创建一种模板，将一些变量放在一个更大的静态结构中。例如，在 Ruby 中，你可以通过连接来创建字符串`"Churn your butter, Jebediah!"`。

```
name = "Jebediah"
"Churn your butter, " + name + "!"
```

或通过内插法。

```
"Churn your butter, #{name}!"
```

就像字符串插值可以使代码更清晰、更简洁一样，语法引号和解引号可以使你更清晰、更简洁地创建列表。比较一下使用`list`函数和使用语法引号。

```
(list '+ 1 (inc 1))
; => (+ 1 2)

`(+ 1 ~(inc 1))
; => (clojure.core/+ 1 2)
```

正如你所看到的，语法引号版本更加简洁。而且，它的视觉形式更接近列表的最终形式，使其更容易理解。

## 在宏中使用语法引语

现在你已经很好地掌握了语法引号的工作原理，来看看`code-critic`宏。你将使用语法引号编写一个更简洁的版本。

```
(defmacro code-critic
  "Phrases are courtesy Hermes Conrad from Futurama"
  [bad good]
  (list 'do
        (list 'println
              "Great squid of Madrid, this is bad code:"
              (list 'quote bad))
        (list 'println
              "Sweet gorilla of Manila, this is good code:"
              (list 'quote good))))

(code-critic (1 + 1) (+ 1 1))
; => Great squid of Madrid, this is bad code: (1 + 1)
; => Sweet gorilla of Manila, this is good code: (+ 1 1)
```

仅仅是看着那些乏味的重复的`list`和单引号，就让我感到害怕。但是如果你用语法引号重写`code-critic`，你就可以使它变得圆滑简洁。

```
(defmacro code-critic
  "Phrases are courtesy Hermes Conrad from Futurama"
  [bad good]
  `(do (println "Great squid of Madrid, this is bad code:"
                (quote ~bad))
       (println "Sweet gorilla of Manila, this is good code:"
                (quote ~good))))
```

在这种情况下，你想引用除符号`good`和`bad`以外的所有内容。在原来的版本中，你必须单独引用每一块，并明确地把它放在一个不方便的列表中，只是为了防止这两个符号被引用。有了语法引号，你只需将整个`do`表达式包裹在一个引号中，并简单地取消你要求值的两个符号的引号。

宏的编写方法介绍到此结束! 亲爱的西萨摩亚和东萨摩亚的神圣蟒蛇，这是很重要的!

总而言之，宏接收未经求值的、任意的数据结构作为参数，并返回 Clojure 求值的数据结构。在定义宏的时候，你可以使用参数重构，就像你可以使用函数和`let`绑定一样。你也可以编写多属性和递归的宏。

大多数情况下，你的宏会返回列表。你可以通过使用`list`函数或使用语法引号来建立要返回的列表。语法引号通常会使代码更清晰、更简洁，因为它可以让你创建一个你想返回的数据结构的模板，更容易进行视觉上的解析。无论你使用语法引号还是普通引号，重要的是在建立你的列表时要清楚地了解符号和它所求值的值之间的区别。如果你想让你的宏返回多种形式供 Clojure 求值，一定要用`do`来包装它们。

## 重构一个宏和取消引号拼接

上一节中的 "code-critic "宏仍然需要一些改进。看看这个重复的地方! 两个 "println "的调用几乎是一样的。让我们把它清理一下。首先，让我们创建一个函数来生成这些\`println'列表。函数比宏更容易思考和使用，所以把宏的内容移到辅助函数中通常是个好主意。

```
(defn criticize-code
  [criticism code]
  `(println ~criticism (quote ~code)))

(defmacro code-critic
  [bad good]
  `(do ~(criticize-code "Cursed bacteria of Liberia, this is bad code:" bad)
       ~(criticize-code "Sweet sacred boa of Western and Eastern Samoa, this is good code:" good)))
```

注意到`criticize-code`函数如何返回一个语法引号的列表。这就是你如何建立起宏将返回的列表。

不过，还有更多的改进空间。这段代码仍然有多个几乎相同的函数调用。在这种情况下，你想对一个值的集合应用同一个函数，使用像`map`这样的 seq 函数是有意义的。

```
(defmacro code-critic
  [bad good]
  `(do ~(map #(apply criticize-code %)
             [["Great squid of Madrid, this is bad code:" bad]
              ["Sweet gorilla of Manila, this is good code:" good]])))
```

这看起来好一点了。你正在 Map 每个批评/代码对，并将 "批评-代码 "函数应用于该对。让我们试着运行这段代码。

```
(code-critic (1 + 1) (+ 1 1))
; => NullPointerException
```

哦，不！这根本就没有用! 发生了什么？问题是，`map`返回一个列表，在这种情况下，它返回一个`println`表达式的列表。我们只想得到每个`println`调用的结果，但是相反，这段代码把两个结果都放在一个列表中，然后试图求值这个列表。

换句话说，当它求值这段代码时，Clojure 会得到类似这样的结果。

```
(do
 ((clojure.core/println " criticism" ' (1 + 1))
  (clojure.core/println "critism" '(+ 1 1)))))
```

然后求值第一个 "println "的调用，给我们提供这个。

```
(do
 (nil
  (clojure.core/println "criticism" '(+ 1 1))))
```

并在求值了第二个\`println'调用后，这样做。

```
(do
 (nil nil))
```

这就是导致异常的原因。`println`求值为`nil`，所以我们最后得到的结果是`(nil nil)`。`nil`是不可调用的，我们得到一个`NullPointerException`。

多么不方便啊 但恰恰相反，无引号拼接正是为了处理这种情况而发明的。取消引号拼接是用`~@`来完成的。如果你只是取消引用一个列表，你会得到这样的结果。

```
`(+ ~(list 1 2 3))
; => (clojure.core/+ (1 2 3))
```

然而，如果你使用 unquote 拼接，你会得到这样的结果。

```
`(+ ~@(list 1 2 3))
; => (clojure.core/+ 1 2 3)
```

Unquote 拼接将一个可排序的数据结构解开，将其内容直接放在包围的语法引号数据结构中。这就像`~@`是一把大锤子，后面的东西是一个皮纳塔，其结果是你曾经参加过的最可怕和最棒的聚会。

总之，如果你在你的代码批评中使用非引号拼接，那么一切都会很顺利。

```
(defmacro code-critic
  [{:keys [good bad]}]
  `(do ~@(map #(apply criticize-code %)
              [["Sweet lion of Zion, this is bad code:" bad]
               ["Great cow of Moscow, this is good code:" good]])))

(code-critic (1 + 1) (+ 1 1))
; => Sweet lion of Zion, this is bad code: (1 + 1)
; => Great cow of Moscow, this is good code: (+ 1 1)
```

呜呼! 你已经成功地将重复的代码提取到一个函数中，并使你的宏代码更加简洁。温尼伯的可爱豚鼠，这是很好的代码!

## 需要注意的事项

宏有一些偷偷摸摸的问题，你应该注意到。在本节中，你将了解到一些宏的陷阱以及如何避免它们。我希望你还没有把自己从你的思想桅杆上解下来。

### 变量捕获

_变量捕获_发生在一个宏引入了一个绑定，而这个绑定对宏的用户来说是未知的，它使一个现有的绑定黯然失色。例如，在下面的代码中，一个宏顽皮地引入了它自己的`let`绑定，这就把代码搞乱了。

```
(def message "Good job!")
(defmacro with-mischief
  [& stuff-to-do]
  (concat (list 'let ['message "Oh, big deal!"])
          stuff-to-do))

(with-mischief
  (println "Here's how I feel about that thing you did: " message))
; => Here's how I feel about that thing you did: Oh, big deal!
```

`println`调用引用了符号`message`，我们认为它与字符串`"好样的！"`绑定。然而，`with-mischief`宏为`message`创建了一个新的绑定。

注意，这个宏没有使用语法引号。这样做会导致一个异常。

```
(def message "Good job!")
(defmacro with-mischief
  [& stuff-to-do]
  `(let [message "Oh, big deal!"]
     ~@stuff-to-do))

(with-mischief
  (println "Here's how I feel about that thing you did: " message))
; Exception: Can't let qualified name: user/message
```

这个异常是为了你自己好：语法引号的设计是为了防止你在宏中意外地捕捉到变量。如果你想在你的宏中引入`let`绑定，你可以使用一个_gensym_。`gensym`函数在每次连续调用时产生唯一的符号。

```
(gensym)
; => G__655

(gensym)
; => G__658
```

你也可以传递一个符号前缀。

```
(gensym 'message)
; => message4760

(gensym 'message)
; => message4763
```

下面是你如何改写`with-mischief`，使之不那么调皮。

```
(defmacro without-mischief
  [& stuff-to-do]
  (let [macro-message (gensym 'message)]
    `(let [~macro-message "Oh, big deal!"]
       ~@stuff-to-do
       (println "I still need to say: " ~macro-message))))

(without-mischief
  (println "Here's how I feel about that thing you did: " message))
; => Here's how I feel about that thing you did:  Good job!
; => I still need to say:  Oh, big deal! 
```

这个例子通过使用`gensym`来创建一个新的、唯一的符号，然后与`macro-message`绑定，避免了变量捕获。在语法引用的`let`表达式中，`macro-message`没有被引用，被解析为 gensym 的符号。这个源码符号与`stuff-to-do`中的任何符号都不同，所以你可以避免变量捕获。因为这是一个常见的模式，你可以使用_自动源码_。自动源码是使用源码的更简洁和方便的方法。

```
`(blarg# blarg#)
(blarg__2869__auto__ blarg__2869__auto__)

`(let [name# "Larry Potter"] name#)
; => (clojure.core/let [name__2872__auto__ "Larry Potter"] name__2872__auto__)
```

在这个例子中，你通过在语法引号列表中的一个符号上附加一个哈希标记（或者_哈希标记_，如果你一定要坚持的话）来创建一个自动源码。Clojure 会自动确保 x`#`的每个实例在同一个语法引号列表中解析为相同的符号，y`#`的每个实例也是如此，以此类推。

`gensym`和 auto-gensym 在编写宏时经常使用，它们允许你避免变量捕获。

### 双重求值

编写宏时要注意的另一个问题是_双重求值_，当一个作为参数传递给宏的表格被求值了不止一次时，就会出现这种情况。请看下面的例子。

```
(defmacro report
  [to-try]
  `(if ~to-try
     (println (quote ~to-try) "was successful:" ~to-try)
     (println (quote ~to-try) "was not successful:" ~to-try)))

;; Thread/sleep takes a number of milliseconds to sleep for
(report (do (Thread/sleep 1000) (+ 1 1)))
```

这段代码是为了测试其参数的真实性。如果参数是真实的，它被认为是成功的；如果是虚假的，它是不成功的。该宏打印出其参数是否成功。在这种情况下，你实际上会睡两秒钟，因为`(Thread/sleep 1000)`被求值了两次：一次在`if`之后，另一次在`println`被调用时。这是因为`(do (Thread/sleep 1000) (+ 1 1))`的代码在整个宏扩展中被重复。这就像你写的一样。

```
(if (do (Thread/sleep 1000) (+ 1 1))
  (println '(do (Thread/sleep 1000) (+ 1 1))
           "was successful:"
           (do (Thread/sleep 1000) (+ 1 1)))

  (println '(do (Thread/sleep 1000) (+ 1 1))
           "was not successful:"
           (do (Thread/sleep 1000) (+ 1 1))))
```

"大问题！"你内心的例子评论家说。好吧，如果你的代码是在银行账户之间转账，这将是一个非常大的问题。以下是你如何避免这个问题的方法。

```
(defmacro report
  [to-try]
  `(let [result# ~to-try]
     (if result#
       (println (quote ~to-try) "was successful:" result#)
       (println (quote ~to-try) "was not successful:" result#))))
```

将 "to-try "放在一个 "let "表达式中，你只需求值一次该代码，并将结果绑定到一个自动标示的符号 "result#"上，现在你可以引用该符号而无需重新求值 "to-try "代码。

### 宏的所有方式

使用宏的一个微妙的缺陷是，你可能最终不得不写越来越多的宏来完成任何事情。这是由于宏的扩展发生在求值之前。

例如，假设你想用`report`宏来`doseq`。而不是多次调用报告。

```
(report (= 1 1))
; => (= 1 1) was successful: true

(report (= 1 2))
; => (= 1 2) was not successful: false
```

让我们进行迭代。

```
(doseq [code ['(= 1 1) '(= 1 2)]]
  (report code))
; => code was successful: (= 1 1)
; => code was successful: (= 1 2)
```

当我们单独传递函数时，报告宏工作正常，但当我们使用`doseq`对多个函数进行`report`迭代时，它是一个毫无价值的失败。下面是其中一个`doseq`迭代的宏扩展的样子。

```
(if
 code
 (clojure.core/println 'code "was successful:" code)
 (clojure.core/println 'code "was not successful:" code))
```

正如你所看到的，`report`在每个迭代中接收未求值的符号`code`；然而，我们希望它在求值时接收任何`code`被绑定的内容。但是`report`在宏扩展时操作，就是不能访问这些值。这就像它有 T.Rex 的手臂，运行时的值永远不在它的掌握之中。

为了解决这种情况，我们可以再写一个宏，像这样。

```
(defmacro doseq-macro
  [macroname & args]
  `(do
     ~@(map (fn [arg] (list macroname arg)) args)))

(doseq-macro report (= 1 1) (= 1 2))
; => (= 1 1) was successful: true
; => (= 1 2) was not successful: false
```

如果你遇到这种情况，请花些时间重新思考你的方法。这很容易使你自己陷入困境，使你无法通过普通的函数调用来完成任何事情。你会被卡住，不得不写更多的宏。宏是非常强大和令人敬畏的，你不应该害怕使用它们。它们把 Clojure 处理数据的设施变成了创造新语言的设施，而这些新语言是根据你的编程问题来设计的。对于某些程序来说，你的代码 90%以上都是宏，这是合适的。尽管它们很棒，但它们也增加了新的组合挑战。它们只是真正的相互组合，所以通过使用它们，你可能会错过 Clojure 中其他类型的组合（函数式、面向对象）。

我们现在已经涵盖了编写宏的所有机制。拍拍你的背吧! 这是一个相当大的交易!

在本章的最后，终于到了戴上你的伪装帽，在本章最开始谈到的网上药水店工作的时候了。

## 为勇敢和真实的人而酿的酒

![](https://www.braveclojure.com/assets/images/cftbat/writing-macros/wizard.png)

在这一章的开头，我透露了一个梦想：找到某种可饮用的东西，一旦摄入，就能暂时让我拥有 80 年代健身大师的力量和气质，把我从抑制和自我意识的牢笼中解放出来。我相信有一天某个地方会有人发明这样的灵丹妙药，所以我们不妨着手建立一个系统来销售这种神话般的药水。让我们把这种假想的混合物称为_勇敢和真实的啤酒_。这个名字是我无缘无故想到的。

在订单纷至沓来之前（双关语！击掌！），我们需要有一些验证的地方。本节向你展示了一种在功能上进行验证的方法，以及如何使用你将编写的名为 "if-valid "的宏更简洁地编写执行验证的代码。这将帮助你了解编写自己的宏的典型情况。如果你只想知道宏的定义，可以跳到["`if-valid`" 第 182 页](https://www.braveclojure.com/writing-macros/#Anchor)。

### 验证函数

为了简单起见，我们只担心验证每个订单的姓名和电子邮件。对于我们的商店，我想我们希望这些订单的细节能像这样表示。

```
(def order-details
  {:name "Mitchard Blimmons"
   :email "mitchard.blimmonsgmail.com"})
```

这个特殊的 Map 有一个无效的电子邮件地址（缺少`@`符号），所以这正是我们的验证代码应该捕捉的订单类型 理想情况下，我们希望编写的代码能产生这样的结果。

```
(validate order-details order-details-validations)
; => {:email ["Your email address doesn't look like an email address."]}
```

也就是说，我们希望能够调用一个函数，\`validate'，其中包含需要验证的数据和如何验证的定义。结果应该是一个 Map，其中每个键对应一个无效的字段，每个值是该字段的一个或多个验证信息的 Vector。下面的两个函数完成了这项工作。

让我们先看看`order-details-validations`。以下是你如何表示验证信息。

```
(def order-details-validations
  {:name
   ["Please enter a name" not-empty]

   :email
   ["Please enter an email address" not-empty

    "Your email address doesn't look like an email address"
    #(or (empty? %) (re-seq #"@" %))]})
```

这是一个 Map，每个键都与错误信息和验证函数对的 Vector 相关。例如，`:name`有一个验证函数，`not-empty`；如果验证失败，你应该得到`"请输入一个名字"`的错误信息。

接下来，我们需要写出`validate'函数。`validate`函数可以分解成两个函数：一个是对单个字段进行验证，另一个是将这些错误信息累积成一个最终的错误信息Map，如`{:email \["你的邮箱地址看起来不像邮箱地址。"]}`。这里有一个叫做`error-messages-for\`的函数，对一个单一的值进行验证。

```
(defn error-messages-for
  "Return a seq of error messages"
  [to-validate message-validator-pairs]
  (map first (filter #(not ((second %) to-validate))
                     (partition 2 message-validator-pairs))))
```

第一个参数，`to-validate`，是你要验证的字段。第二个参数，`message-validator-pairs`，应该是一个有偶数元素的序列。这个序列被分组为`(partition 2 message-validator-pairs)'的对。对中的第一个元素应该是一个错误信息，对中的第二个元素应该是一个函数（就像在`order-details-validations`中安排的对）。`error-messages-for`函数的工作原理是过滤出所有错误信息和验证对，其中验证函数在应用于`to-validate`时返回`true`。然后，它使用`map first\`来获取每对元素的第一个元素，即错误信息。下面是它的操作。

```
(error-messages-for "" ["Please enter a name" not-empty])
; => ("Please enter a name")
```

现在我们需要将这些错误信息积累到一个 Map 中。

下面是完整的`validate`函数，以及我们将其应用于`order-details`和`order-details-validations`时的输出。

```
(defn validate
  "Returns a map with a vector of errors for each key"
  [to-validate validations]
  (reduce (fn [errors validation]
            (let [[fieldname validation-check-groups] validation
                  value (get to-validate fieldname)
                  error-messages (error-messages-for value validation-check-groups)]
              (if (empty? error-messages)
                errors
                (assoc errors fieldname error-messages))))
          {}
          validations))

(validate order-details order-details-validations)
; => {:email ("Your email address doesn't look like an email address")}
```

成功了! 这个函数是通过减少`order-details-validations'并将`order-details'的每个键的错误信息（如果有的话）关联到一个最终的错误信息 Map。

### if-valid

有了我们的验证代码，我们现在可以随心所欲地验证记录了。大多数情况下，验证会像这样。

```
(let [errors (validate order-details order-details-validations)]
  (if (empty? errors)
    (println :success)
    (println :failure errors)))
```

该模式是做以下工作。

1. 验证一条记录并将结果绑定到`errors`。
2. 检查是否有任何错误
3. 3.如果有，做成功的事情，这里`(println :success)`。
4. 否则，做失败的事情，这里`(println :failure errors)`。

我已经在实际生产的网站中使用了这个验证代码。起初，我发现自己不断重复代码的微小变化，这无疑表明我需要引入一个抽象，以隐藏重复的部分：应用`validate`函数，将结果绑定到一些符号，并检查结果是否为空。为了创建这种抽象，你可能会想写一个这样的函数。

```
(defn if-valid
  [record validations success-code failure-code]
  (let [errors (validate record validations)]
    (if (empty? errors)
      success-code
      failure-code)))
```

然而，这不会起作用，因为`success-code`和`failure-code`每次都会被求值。宏会起作用，因为宏允许你控制求值。下面是你如何使用宏的方法。

```
(if-valid order-details order-details-validations errors
 (render :success)
 (render :failure errors))
```

这个宏隐藏了重复的细节，帮助你更简洁地表达你的意图。这就像要求别人给你开瓶器，而不是说："请给我手动装置，用于去除玻璃容器中液体的临时密封剂。" 下面是实施方法。

```
(defmacro if-valid
  "Handle validation more concisely"
  [to-validate validations errors-name & then-else]
  `(let [~errors-name (validate ~to-validate ~validations)]
     (if (empty? ~errors-name)
       ~@then-else)))
```

这个宏需要四个参数。 `to-validate`, `validations`, `errors-name`, 和其余参数`then-else`. 像这样使用`errors-name`是一个新的策略。我们想在`then-else`语句中访问`validate`函数返回的错误。要做到这一点，我们要告诉宏它应该把结果绑定到什么符号上。下面的宏扩展显示了它是如何工作的。

```
 (macroexpand
 '(if-valid order-details order-details-validations my-error-name
            (println :success)
            (println :failure my-error-name)))
(let*
 [my-error-name (user/validate order-details order-details-validations)]
 (if (clojure.core/empty? my-error-name)
  (println :success)
  (println :failure my-error-name)))
```

语法引号抽象了你之前看到的`let/validate/if`模式的一般形式。然后我们使用 unquote 拼接来解压`if`分支，这些分支被打包到`then-else`其余参数中。

这真是太简单了! 说了这么多关于宏的内容，并详细介绍了它们的机制，我打赌你一定以为会有更复杂的东西。对不起，朋友。如果你对你的失望感到难以接受，我知道有一种饮料可以帮助你。

## 总结

在本章中，你学会了如何编写自己的宏。宏的定义与函数非常相似：它们有参数、文件串和主体。它们可以使用参数重构和休息参数，而且可以是递归的。你的宏几乎都会返回列表。你有时会使用`list`和`seq`函数来编写简单的宏，但大多数时候你会使用语法引号，，它让你使用安全模板来编写宏。

当你编写宏时，重要的是要记住符号和值之间的区别：宏在代码被求值之前被展开，因此不能访问求值的结果。双重求值和变量捕获是另外两个微妙的陷阱，但你可以通过明智地使用 "let "表达式和代词来避免它们。

宏是一种有趣的工具，可以让你在编码时少一些拘束。通过让你控制求值，宏给你一定程度的自由和表达，这是其他语言所不允许的。在你的 Clojure 旅程中，你可能会听到有人告诫你不要使用宏，说什么 "宏是邪恶的 "和 "你不应该使用宏"。不要听这些假正经的人的话--至少在开始的时候不要听他们的。走出去，享受美好的时光。这是你学习在哪些情况下适合使用宏的唯一途径。你会从另一个角度知道如何有技巧地、潇洒地使用宏。

## 练习

1.  编写宏`when-valid`，使它的行为与`when`相似。下面是一个调用它的例子。

    ```
    (when-valid order-details order-details-validations
    (println "It's a success!")
    (render :success))
    ```

    当数据有效时，应该求值`println`和`render`形式，如果数据无效，`when-valid`应该返回`nil`。
2. 你看到`and`是作为一个宏实现的。把\`or'作为一个宏来实现。
3.  在第 5 章中，你创建了一系列函数（`c-int`, `c-str`, `c-dex`）来读取一个 RPG 字符的属性。写一个宏，用一个宏调用来定义任意数量的属性检索函数。以下是你如何调用它。

    ```
    ```

(defattrs c-int :intelligence c-str :strength c-dex :dexterity)

```
```
