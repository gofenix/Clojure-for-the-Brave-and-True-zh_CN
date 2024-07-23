# 做事情：Clojure 速成班

是时候学习如何用 Clojure 真正地 _做事_ 了! 该死的! 尽管你无疑已经听说过 Clojure 令人敬畏的并发支持和其他了不起的功能，但 Clojure 最突出的特点是它是一种 Lisp 语言。在本章中，你将探索构成这个 Lisp 核心的元素：语法、函数和数据。它们将共同为你在 Clojure 中表示和解决问题打下坚实的基础。

在打下这个基础之后，你将能够编写一些超级重要的代码。在最后一节中，你将通过创建一个霍比特人的模型，并编写一个函数将其打在一个随机的位置上，从而将一切联系起来。超级! 重要的!

当你阅读本章时，我建议你在 REPL 中输入例子并运行它们。用一种新的语言编程是一种技能，就像约德尔舞或花样游泳一样，你必须通过练习来学习它。 请留意它!

## 语法

Clojure 的语法很简单。像所有的 Lisp 一样，它采用了统一的结构、少量的特殊运算符，以及从藏在麻省理工学院下面的小括号矿井中不断提供的小括号，Lisp 就是在那里诞生的。

### Form

所有的 Clojure 代码都是以统一结构编写的。Clojure 可以识别两种结构。

* 数据结构的字面表示（如数字、字符串、Map 和 Vector）
* 操作

我们使用术语 _form_ 来指代有效的代码。我有时也会用 _表达式_ 来指代 Clojure Form。但不要太纠结于术语。Clojure _求值_每一个 Form，以产生一个值。这些字面意义的表达都是有效的 Form。

```
1
"a string"
["a" "vector" "of" "strings"]
```

当然，你的代码很少包含自由浮动的字符，因为它们本身实际上并不做什么。相反，你会在操作中使用字面符号。操作是你 _做事情_ 的方式。所有操作的 Form 都是： _开括号_ ， _操作符_ ， _操作数 _ ，_闭括号_ 。

```
(operator operand1 operand2 ... operandn)
```

请注意，这里没有逗号。Clojure 使用空格来分隔操作数，它将逗号视为空格。下面是一些操作的例子。

```
(+ 1 2 3)
; => 6

(str "It was the panda " "in the library " "with a dust buster")
; => "It was the panda in the library with a dust buster"
```

![img](https://www.braveclojure.com/assets/images/cftbat/do-things/panda.png)

在第一个操作中，运算符`+`将操作数`1`、`2`和`3`相加。在第二个操作中，运算符`str`将三个字符串连接起来，形成一个新的字符串。这两种 Form 都是有效的。这里有一个不是 Form 的东西，因为它没有一个结束的小括号。

```
(+
```

Clojure 的结构统一性可能与你所习惯的不同。在其他语言中，不同的操作可能有不同的结构，这取决于操作符和操作数。例如，JavaScript 采用的是 中缀符号、点运算符和小括号的大杂烩。

```
1 + 2 + 3
"It was the panda ".concat("in the library ", "with a dust buster")
```

相比之下，Clojure 的结构是非常简单和一致的。无论你使用哪种运算符，或对哪种数据进行操作，其结构都是一样的。

### 控制流

让我们来看看三个基本的控制流操作符。`if`, `do`, 和`when`。在本书中，你会遇到更多的操作，但这些操作可以让你开始。

#### if

这是一个 `if` 表达式的一般结构。

```
(if boolean-form
  then-form
  optional-else-form)
```

boolean-form 只是一个求值为真或假的 Form。你会在下一节中了解到逻辑真和逻辑假。下面是几个`if`的例子。

```
(if true
  "By Zeus's hammer!"
  "By Aquaman's trident!")
; => "By Zeus's hammer!"

(if false
  "By Zeus's hammer!"
  "By Aquaman's trident!")
; => "By Aquaman's trident!"
```

第一个例子返回 "By Zeus's hammer!"，因为其布尔 Form 求值为 `true`，是一个真值；第二个例子返回 "By Aquaman's trident!"，因为其布尔 Form "false"，求值为一个假值。

你也可以省略`else`分支。如果你这样做，并且布尔表达式是假的，Clojure 会返回`nil`，就像这样。

```
(if false
  "By Odin's Elbow!")
; => nil
```

注意`if`使用操作数位置将操作数与`then`和`else`分支联系起来：第一个操作数是`then`分支，第二个操作数是（可选）`else`分支。因此，每个分支只能有一种 Form。这与大多数语言不同。例如，你可以在 Ruby 中这样写。

```
if true
  doer.do_thing(1)
  doer.do_thing(2)
else
  other_doer.do_thing(1)
  other_doer.do_thing(2)
end
```

为了绕过这个明显的限制，你可以使用`do`操作符。

#### do

`do`操作符可以让你在括号中 _包裹_ 起多个 Form，并运行其中的每一个。在你的 REPL 中尝试以下操作。

```
(if true
  (do (println "Success!")
      "By Zeus's hammer!")
  (do (println "Failure!")
      "By Aquaman's trident!"))
; => Success!
; => "By Zeus's hammer!"
```

这个操作符让你在`if`表达式的每个分支中做多件事情。在这种情况下，会发生两件事。`Success!`被打印在 REPL 中，`By Zeus's hammer!`被作为整个`if`表达式的值返回。

#### when

`when`操作符就像`if`和`do`的组合，但没有`else`分支。下面是一个例子。

```
(when true
  (println "Success!")
  "abra cadabra")
; => Success!
; => "abra cadabra"
```

如果你想在某个条件为真时做多件事，而你总是想在条件为假时返回`nil`，请使用`when`。

#### nil, true, false, Truthiness, Equality, 和 Boolean 表达式

Clojure 有`true`和`false`。`nil`在 Clojure 中用来表示 _没有值_ 。你可以用`nil?`函数来检查一个值是否为`nil`。

```
(nil? 1)
; => false

(nil? nil)
; => true
```

`nil`和`false`都是用来表示逻辑上的虚假性，而所有其他的值都是逻辑上的真实性。 _Truthy_ 和 _falsey_ 指的是在布尔表达式中如何处理一个值，比如传递给`if`的第一个表达式。

```
(if "bears eat beets"
  "bears beets Battlestar Galactica")
; => "bears beets Battlestar Galactica"

(if nil
  "This won't be the result because nil is falsey"
  "nil is falsey")
; => "nil is falsey"
```

在第一个例子中，字符串`bears eat beets`被认为是`true`，所以`if`表达式求值为 bears beets Battlestar Galactica\`。第二个例子显示一个 nil 是假的。

Clojure 的等于运算符是`=`。

```
(= 1 1)
; => true

(= nil nil)
; => true

(= 1 2)
; => false
```

其他一些语言要求你在比较不同类型的值时使用不同的运算符。例如，你可能不得不使用某种专门为字符串制作的特殊字符串等于运算符。但在使用 Clojure 的内置数据结构时，你不需要像这样奇怪或繁琐的东西来测试等于。

Clojure 使用布尔运算符`or`和`and`。`or`返回第一个真值或最后一个值。`and`返回第一个 false 的值，如果没有 false 的值，则返回最后一个 true 的值。让我们先看一下`or`。

```
(or false nil :large_I_mean_venti :why_cant_I_just_say_large)
; => :large_I_mean_venti

(or (= 0 1) (= "yes" "no"))
; => false

(or nil)
; => nil
```

在第一个例子中，返回值是`:large_I_mean_venti`，因为它是第一个真值。第二个例子没有真值，所以`or`返回最后一个值，即`false`。在最后一个例子中，同样没有真值存在，`or`返回最后一个值，即`nil`。现在我们来看看`and`。

```
(and :free_wifi :hot_coffee)
; => :hot_coffee

(and :feelin_super_cool nil false)
; => nil
```

在第一个例子中，`and`返回最后一个真值，`:hot_coffee`。在第二个例子中, `and`返回`nil`, 这是第一个 false 的值.

### 用 def 命名

在 Clojure 中, 你可以使用`def`将一个名字与一个值绑定起来:

```
(def failed-protagonist-names
  ["Larry Potter" "Doreen the Explorer" "The Incredible Bulk"])

failed-protagonist-names
; => ["Larry Potter" "Doreen the Explorer" "The Incredible Bulk"]
```

![img](https://www.braveclojure.com/assets/images/cftbat/do-things/larry-potter.png)

在这个例子中，你把名字`failed-protagonist-names`绑定到一个包含三个字符串的 Vector（你将在["Vector "第 45 页](https://www.braveclojure.com/do-things/#Anchor-3)中了解 Vector）。

请注意，我使用的是 "绑定"一词，而在其他语言中，你会说你是在给一个 _变量_ 赋值。那些其他语言通常鼓励你对同一个变量进行多次赋值。

例如，在 Ruby 中，你可以对一个变量进行多次赋值。

```
severity = :mild
error_message = "OH GOD! IT'S A DISASTER! WE'RE "
if severity == :mild
  error_message = error_message + "MILDLY INCONVENIENCED!"
else
  error_message = error_message + "DOOOOOOOMED!"
end
```

你可能想在 Clojure 中做类似的事情。

```
(def severity :mild)
(def error-message "OH GOD! IT'S A DISASTER! WE'RE ")
(if (= severity :mild)
  (def error-message (str error-message "MILDLY INCONVENIENCED!"))
  (def error-message (str error-message "DOOOOOOOMED!")))
```

然而，像这样改变与名字相关的值会使你更难理解你的程序的行为，因为更难知道哪个值是与名字相关的，或者为什么这个值可能已经改变了。Clojure 有一套处理变化的工具，你会在第 10 章中了解到。随着你对 Clojure 的学习，你会发现你很少需要改变一个名字/值的关联。下面是你写前面代码的一种方式。

```
(defn error-message
  [severity]
  (str "OH GOD! IT'S A DISASTER! WE'RE "
       (if (= severity :mild)
         "MILDLY INCONVENIENCED!"
         "DOOOOOOOMED!")))

(error-message :mild)
; => "OH GOD! IT'S A DISASTER! WE'RE MILDLY INCONVENIENCED!"
```

这里，你创建了一个函数，`error-message`，它接受一个参数，`severity`，并使用它来决定返回哪个字符串。然后你用`:mild`作为严重程度来调用这个函数。你将在["函数 "第 48 页](https://www.braveclojure.com/do-things/#Anchor-4)中学习所有关于创建函数的知识；与此同时，你应该把`def`当作定义常量。在接下来的几章中，你将学习如何通过接受函数式编程范式来处理这个明显的限制。

## 数据结构

Clojure 带有少量的数据结构，你在大多数时候都会用到。如果你来自面向对象的背景，你会惊讶于你可以用这里介绍的看似基本的类型做很多事情。

Clojure 的所有数据结构都是不可改变的，这意味着你不能在原地改变它们。例如，在 Ruby 中，你可以做以下事情来重新分配索引为 0 的失败主角的名字。

```
failed_protagonist_names = [
  "Larry Potter",
  "Doreen the Explorer",
  "The Incredible Bulk"
]
failed_protagonist_names[0] = "Gary Potter"

failed_protagonist_names
# => [
#   "Gary Potter",
#   "Doreen the Explorer",
#   "The Incredible Bulk"
# ]
```

Clojure 没有与之对应的东西。你会在第 10 章中了解到更多关于 Clojure 这样实现的原因，但现在只学习如何做事情，而不考虑所有的哲学问题，这很有趣。不多说了，让我们来看看 Clojure 中的数字。

### Number

Clojure 有相当复杂的 Number 支持。我不会花太多时间纠缠于无聊的技术细节（比如强制和传染），因为那会妨碍 _做事情_ 。如果你对这些枯燥的细节感兴趣，请查看\*[http://clojure.org/data\_structures#Data%20Structures-Numbers](http://clojure.org/data\_structures#Data%20Structures-Numbers)\*的文档。可以说，Clojure 会很高兴地处理你扔给它的所有东西。

在此期间，我们将使用整数和浮点数。我们还将使用分数，Clojure 可以直接表示这些分数。下面分别是一个整数、一个浮点数和一个分数。

```
93
1.2
1/5
```

### 字符串

字符串代表文本。这个名字来自于古代腓尼基人，他们在一次涉及纱线的事故后，有一天发明了字母表。下面是一些字符串字面的例子。

```
"Lord Voldemort"
"\"He who must not be named\""
"\"Great cow of Moscow!\" - Hermes Conrad"
```

![img](https://www.braveclojure.com/assets/images/cftbat/do-things/wookie.png)

注意，Clojure 只允许用双引号来划分字符串。例如，'Lord Voldemort' 就不是一个有效的字符串。还要注意，Clojure 没有字符串插值。它只允许通过`str`函数进行拼接。

```
(def name "Chewbacca")
(str "\"Uggllglglglglglll\" - " name)
; => "Uggllglglglglglll" - Chewbacca
```

### Map

Map 类似于其他语言中的字典或哈希值。它们是一种将一些值与另一些值联系起来的方式。Clojure 中的两种 Map 是哈希 Map 和排序 Map。我将只介绍更基本的哈希 Map。让我们来看看 Map 字面的一些例子。这里有一个空 Map。

```
{}
```

在这个例子中，`:first-name`和`:last-name`是关键字（我将在下一节介绍这些）。

```
{:first-name "Charlie"
 :last-name "McFishwich"}
```

这里我们把`"string-key"`和`+`函数联系起来。

```
{"string-key" +}
```

Map 可以被嵌套。

```
{:name {:first "John" :middle "Jacob" :last "Jingleheimerschmidt"}}.
```

注意，Map 的值可以是任何类型--字符串、数字、Map、Vector，甚至函数。Clojure 并不关心这个问题。

除了使用 map 字面，你还可以使用`hash-map`函数来创建一个 map。

```
(hash-map :a 1 :b 2)
; => {:a 1 :b 2}.
```

你可以用`get`函数在 Map 中查询数值。

```
(get {:a 0 :b 1} :b)
; => 1

(get {:a 0 :b {:c "ho hum"}} :b)
; => {:c "ho hum"}
```

在这两个例子中，我们向`get`询问给定 Map 中`:b`键的值--在第一个例子中，它返回`1`，而在第二个例子中，它返回嵌套 Map`{:c "ho hum"}`。

如果没有找到你的键，`get`将返回`nil`，或者你可以给它一个默认值，例如`"unicorns？"`。

```
(get {:a 0 :b 1} :c)
; => nil

(get {:a 0 :b 1} :c "unicorns?")
; => "unicorns?"
```

`get-in`函数可以让你在嵌套 Map 中查询数值。

```
(get-in {:a 0 :b {:c "ho hum"}} [:b :c])
; => "ho hum"
```

另一种在 Map 中查询数值的方法是把 Map 当作一个以键为参数的函数。

```
({:name "The Human Coffeepot"} :name)
; => "The Human Coffeepot"
```

你可以用 Map 做的另一件很酷的事情是把 Keywords 作为函数来查询它们的值，这就引出了下一个主题，Keywords。

### Keywords

了解 Clojure 关键字的最好方法是看它们是如何被使用的。正如你在上一节中所看到的，它们主要是作为 Map 中的键来使用。下面是一些 Keywords 的例子。

```
:a
:rumplestiltsken
:34
:_?
```

Keywords 可以作为函数使用，在数据结构中查找相应的值。例如，你可以在一个 Map 中查找`:a`。

```
(:a {:a 1 :b 2 :c 3})
; => 1
```

这相当于。

```
(get {:a 1 :b 2 :c 3} :a)
; => 1
```

你可以提供一个默认值，和`get`一样。

```
(:d {:a 1 :b 2 :c 3} "No gnome knows homes like Noah knows")
; => "No gnome knows homes like Noah knows"
```

使用关键字作为一个函数是令人愉快的简洁，Real Clojurists 一直在这样做。你也应该这样做!

### Vector

Vector 类似于数组, 它是一个以 0 为索引的 Set。例如, 下面是一个 Vector 的字面意思:

```
[3 2 1]
```

这里我们要返回一个 Vector 的第 0 个元素。

```
(get [3 2 1] 0)
; => 3
```

下面是另一个按索引获取的例子。

```
(get ["a" {:name "Pugsley Winterbottom"} "c"] 1)
; => {:name "Pugsley Winterbottom"}
```

你可以看到，Vector 元素可以是任何类型，而且你可以混合类型。还注意到我们使用的`get`函数与我们在 Map 中查找数值时使用的相同。

你可以用`vector`函数来创建 Vector。

```
(vector "creepy" "full" "moon")
; => ["creepy" "full" "moon"]
```

你可以使用`conj`函数来添加额外的元素到 Vector 中。元素被添加到 Vector 的 _尾部_ 。

```
(conj [1 2 3] 4)
; => [1 2 3 4]
```

Vector 不是存储序列的唯一方法；Clojure 还有 _列表_ 。

### 列表

列表 与 Vector 类似，它们都是数值的线性 Set。但也有一些区别。例如，你不能用`get`检索列表元素。要写一个列表的字面意思, 只需将元素插入括号内, 并在开头使用单引号:

```
'(1 2 3 4)
; => (1 2 3 4)
```

注意，当 REPL 打印出列表时，它不包括单引号。我们将在后面的第 7 章中再来讨论为什么会这样。如果你想从一个列表中检索一个元素，你可以使用 `nth` 函数。

```
(nth '(:a :b :c) 0)
; => :a

(nth '(:a :b :c) 2)
; => :c
```

我在本书中没有详细介绍性能，因为我认为只有你熟悉一种语言之后再关注它才是有用的。然而，知道使用`nth`从列表中检索一个元素比使用`get`从 Vector 中检索一个元素要慢一些是很好的。这是因为 Clojure 必须遍历一个列表中的所有 _n_ 个元素才能到达 _n_ 个，而通过索引访问一个 Vector 元素最多只需要几跳。

列表值可以有任何类型，你可以用`list`函数创建列表。

```
(list 1 "two" {3 4})
; => (1 "二" {3 4})
```

元素被添加到一个列表的 _开头_ 。

```
(conj '(1 2 3) 4)
; => (4 1 2 3)
```

什么时候应该使用列表，什么时候应该使用 Vector？一个好的经验法则是，如果你需要很容易地把项目添加到一个序列的开头，或者你正在写一个宏，你应该使用一个列表。否则，你应该使用 Vector。随着你学习的深入，你会对何时使用哪种方法有很好的感觉。

### Set

Set 是唯一值的集合。Clojure 有两种类型的 Set：哈希 Set 和排序 Set。我将专注于哈希 Set，因为它们更经常被使用。下面是一个哈希 Set 的文字符号。

```
#{"kurt vonnegut" 20 :icicle}.
```

你也可以用`hash-set`来创建一个 Set:

```
(hash-set 1 1 2 2)
; => #{1 2}
```

注意，一个值的多个实例在 Set 中成为一个唯一的值，所以我们只剩下一个`1`和一个`2`。如果你试图将一个值添加到一个已经包含该值的 Set 中（比如下面代码中的`:b`），它仍然只有一个该值。

```
( conj #{:a :b} :b)
; => #{:a :b}
```

你也可以通过使用`set`函数从现有的 Vector 和列表中创建 Set。

```
(set [3 3 3 4 4])
; => #{3 4}
```

你可以使用`contains?`函数来检查 Set 的成员资格，通过使用`get`，或通过使用关键字作为函数，以 Set 为参数。`contains?`返回`true`或`false`，而`get`和关键字查找将返回存在的值，如果不存在，则返回`nil`。

下面是你如何使用`contains?`。

```
(contains? #{:a :b} :a)
; => true

(contains? #{:a :b} 3)
; => false

(contains? #{nil} nil)
; => true
```

下面是你如何使用关键字。

```
(:a #{:a :b})
; => :a
```

这里是你如何使用`get`的方法。

```
(get #{:a :b} :a)
; => :a

(get #{:a nil} nil)
; => nil

(get #{:a :b} "kurt vonnegut")
; => nil
```

注意，使用`get`来测试一个 Set 是否包含`nil`，将总是返回`nil`，这令人困惑。当你专门测试 Set 成员时，`contains?`可能是更好的选择。

### 简单性

你可能已经注意到，到目前为止，对数据结构的处理并不包括对如何创建新类型或类的描述。原因是 Clojure 对简单性的强调鼓励你首先去接触内置的数据结构。

如果你来自面向对象的背景，你可能会认为这种方法很奇怪而且落后。然而，你会发现，你的数据不一定非要和一个类紧密地捆绑在一起，才是有用和可理解的。这里有一个被 Clojurists 喜爱的寓言故事，暗示了 Clojure 的哲学。

> 让 100 个函数操作一个数据结构比让 10 个函数操作 10 个数据结构要好。 -Alan Perlis

在接下来的章节中，你会了解到更多关于 Clojure 哲学的这个方面。现在，请留意你通过坚持使用基本数据结构来获得代码重用性的方法。

我们的 Clojure 数据结构入门课程到此结束。现在，是时候深入到函数中去，学习如何使用这些数据结构了

## 函数

人们为 Lisp 疯狂的原因之一是，这些语言可以让你建立起行为复杂的程序，但主要的构件--函数--却是如此简单。本节通过解释以下内容，让你开始了解 Lisp 函数的美丽和优雅。

* 调用函数
* 函数与宏和特殊 Form 有什么不同
* 定义函数
* 匿名函数
* 返回函数

### 调用函数

现在你已经看到了许多函数调用的例子。

```
(+ 1 2 3 4)
(* 1 2 3 4)
(first [1 2 3 4])
```

请记住，所有的 Clojure 操作都有相同的语法：开括号、操作符、操作数、闭括号。 _函数调用_ 只是操作的另一个术语，其中运算符是一个函数或一个 _函数表达式_ （一个返回函数的表达式）。

这可以让你写出一些相当有趣的代码。下面是一个函数表达式，它返回`+`（加法）函数。

```
(or + -)
; => #<core$_PLUS_ clojure.core$_PLUS_@76dace31>
```

该返回值是加法函数的字符串表示。因为`or`的返回值是第一个真值，而这里的加法函数是真值，所以返回的是加法函数。你也可以在另一个表达式中使用这个表达式作为运算符。

```
((or + -) 1 2 3)
; => 6
```

因为`(or + -)`返回`+`，这个表达式被求值为`1`、`2`和`3`之和，返回`6`。

下面是几个有效的函数调用，它们都返回`6`。

```
((and (= 1 1) +) 1 2 3)
; => 6

((first [+ 0]) 1 2 3)
; => 6
```

在第一个例子中，`and`的返回值是第一个假值或最后一个真值。在这个例子中，`+`被返回，因为它是最后一个真值，然后被应用于参数`1 2 3`，返回`6`。在第二个例子中，`first`的返回值是一个序列中的第一个元素，在这个例子中是`+`。

然而，这些都不是有效的函数调用，因为数字和字符串都不是函数。

```
(1 2 3 4)
("test" 1 2 3)
```

如果你在 REPL 中运行这些，你会得到这样的结果。

```
ClassCastException java.lang.String cannot be cast to clojure.lang.IFn
user/eval728 (NO_SOURCE_FILE:1)
```

当你继续使用 Clojure 时，你可能会多次看到这个错误： _cannot be cast to clojure.lang.IFn_ 。只是意味着你试图将某个东西作为一个函数使用，而它并不是。

函数的灵活性并没有随着函数表达式的出现而结束! 在语法上，函数可以接受任何表达式作为参数--包括 _其他函数_ 。可以接受一个函数作为参数或返回一个函数的函数被称为 _高阶函数_ 。具有高阶函数的编程语言被称为支持 _函数一等公民_ ，因为你可以像对待数字和 Vector 等更熟悉的数据类型一样，将函数作为值来处理。

以`map`函数（不要与 map 数据结构混淆）为例。`map`通过对一个集合的每个成员应用一个函数来创建一个新的列表。这里，`inc`函数将一个数字增加 1。

```
(inc 1.1)
; => 2.1

(map inc [0 1 2 3])
; => (1 2 3 4)
```

(注意`map`并不返回一个 Vector，尽管我们提供了一个 Vector 作为参数。你将在第四章中了解原因。现在，请相信这是好的，也是预期的）。

Clojure 对一等公民函数的支持使你能够建立比没有一等公民函数的语言更强大的抽象概念。那些不熟悉这种编程方式的人认为函数允许你对数据实例进行泛化操作。例如，`+`函数对任何特定数字的加法进行了抽象。

相比之下，Clojure（以及所有 Lisp）允许你创建泛化进程的函数。`map`允许你通过在任何集合上应用一个函数--任何函数--来概括转换一个集合的过程。

你需要知道的关于函数调用的最后一个细节是，Clojure 在将所有函数参数传递给函数之前，会递归地求值这些参数。下面是 Clojure 如何求值一个参数也是函数调用的函数调用。

```
(+ (inc 199) (/ 100 (- 7 2)))
(+ 200 (/ 100 (- 7 2))) ; evaluated "(inc 199)"
(+ 200 (/ 100 5)) ; evaluated (- 7 2)
(+ 200 20) ; evaluated (/ 100 5)
220 ; final evaluation
```

函数调用启动了求值过程，在应用`+`函数之前，所有的子 Form 都被求值了。

### 函数调用、宏调用和特殊 Form

在上一节中，你了解到函数调用是以函数表达式为操作符的表达式。另外两种表达式是 _宏调用_ 和 _特殊 Form_ 。你已经看到了几种特殊 Form：`def` 和`if`表达式。

你将在第 7 章中学习关于宏调用和特殊 Form 的所有知识。现在，使特殊 Form "特殊"的主要特征是，与函数调用不同，它们不求值所有的操作数。

以 "if "为例。这是它的一般结构。

```
(if boolean-form
  then-form
  optional-else-form)
```

现在想象一下你有一个这样的`if`语句。

```
(if good-mood
  (tweet walking-on-sunshine-lyrics)
  (tweet mopey-country-song-lyrics))
```

显然，在这样的`if`表达中，我们希望 Clojure 只求值两个分支中的一个。如果 Clojure 同时求值两个`tweet`函数调用，你的 Twitter 粉丝们最终会非常困惑。

另一个区别于特殊 Form 的特征是，你不能把它们作为函数的参数。一般来说，特殊 Form 实现了 Clojure 的核心功能，只是不能用函数实现。Clojure 只有少量的特殊 Form，而如此丰富的语言是用如此小的一组构建块来实现的，这是很令人惊讶的。

宏与特殊 Form 类似，它们对操作数的求值与函数调用不同，而且它们也不能作为参数传递给函数。但这段弯路已经走得够长了；现在是学习如何定义函数的时候了!

### 定义函数

函数的定义由五个主要部分组成。

* `defn`
* 函数名
* 描述该函数的 docstring(可选)
* 括号中列出的参数
* 函数体

下面是一个函数定义的例子和函数的调用示例。

```
➊ (defn too-enthusiastic
➋   "Return a cheer that might be a bit too enthusiastic"
➌   [name]
➍   (str "OH. MY. GOD! " name " YOU ARE MOST DEFINITELY LIKE THE BEST "
  "MAN SLASH WOMAN EVER I LOVE YOU AND WE SHOULD RUN AWAY SOMEWHERE"))

(too-enthusiastic "Zelda")
; => "OH. MY. GOD! Zelda YOU ARE MOST DEFINITELY LIKE THE BEST MAN SLASH WOMAN EVER I LOVE YOU AND WE SHOULD RUN AWAY SOMEWHERE"
```

在➊处，`too-enthusiastic`是函数的名称，在➋处有一个描述性的 docstring。参数 "name "在➌处给出，函数体在➍处接受参数，并做了它所描述的事情--返回一个可能有点过于热情的欢呼。

让我们更深入地了解 docstring、参数和函数体。

#### docstring

docstring\*是一种描述和记录你的代码的有用方法。你可以在 REPL 中用 `(doc`fn-name`)`查看一个函数的 docstring，例如 `(doc map)`。如果你使用一个工具为你的代码生成文档，那么 docstring 也会发挥作用。

#### 参数和 Arity

Clojure 函数可以用零个或多个参数来定义。你传递给函数的值被称为 _arguments_ ，参数可以是任何类型。参数的数量就是函数的特性。下面是一些具有不同性质的函数定义。

```
(defn no-params
  []
  "I take no parameters!")
(defn one-param
  [x]
  (str "I take one parameter: " x))
(defn two-params
  [x y]
  (str "Two parameters! That's nothing! Pah! I will smoosh them "
  "together to spite you! " x y))
```

在这些例子中，`no-params`是一个 0-arity 函数，`one-param`是 1-arity，`two-params`是 2-arity。

函数也支持 _参数重载_。这意味着你可以定义一个函数，使不同的函数体根据不同的参数来运行。下面是一个多义性函数定义的一般方式。请注意，每个数位定义都被括在括号里，并且有一个参数列表。

```
(defn multi-arity
  ;; 3-arity arguments and body
  ([first-arg second-arg third-arg]
     (do-things first-arg second-arg third-arg))
  ;; 2-arity arguments and body
  ([first-arg second-arg]
     (do-things first-arg second-arg))
  ;; 1-arity arguments and body
  ([first-arg]
     (do-things first-arg)))
```

函数参数重载是为参数提供默认值的一种方法。在下面的例子中，`"karate"`是`chop-type`参数的默认参数。

```
(defn x-chop
  "Describe the kind of chop you're inflicting on someone"
  ([name chop-type]
     (str "I " chop-type " chop " name "! Take that!"))
  ([name]
     (x-chop name "karate")))
```

如果你用两个参数调用`x-chop`，该函数的工作原理和它不是一个多义性函数时一样。

```
(x-chop "Kanye West" "slap")
; => "I slap chop Kanye West! Take that!"
```

![img](https://www.braveclojure.com/assets/images/cftbat/do-things/kanye.png)

如果你调用`x-chop`时只有一个参数，`x-chop`实际上会在提供第二个参数`karate`时调用自己。

```
(x-chop "Kanye East")
; => "I karate chop Kanye East! Take that!"
```

像这样用函数本身来定义一个函数，可能显得不寻常。如果是这样，那就好了! 你正在学习一种新的方法来做事!

你也可以让每种函数做一些完全不相关的事情。

```
(defn weird-arity
  ([]
     "Destiny dressed you this morning, my friend, and now Fear is
     trying to pull off your pants. If you give up, if you give in,
     you're gonna end up naked with Fear just standing there laughing
     at your dangling unmentionables! - the Tick")
  ([number]
     (inc number)))
```

0-arity 主体返回一个明智的引号，1-arity 主体增加一个数字。最有可能的是，你不会想写一个这样的函数，因为有两个完全不相关的函数体会让人困惑。

Clojure 还允许你通过包括一个 _可变参数_ 来定义函数，就像 "把这些参数的其余部分放在一个列表中，名称如下"。可变参数用安培号（`&`）表示，如➊所示。

![img](https://www.braveclojure.com/assets/images/cftbat/do-things/old-man.png)

```
(defn codger-communication
  [whippersnapper]
  (str "Get off my lawn, " whippersnapper "!!!"))

(defn codger
➊   [& whippersnappers]
  (map codger-communication whippersnappers))

(codger "Billy" "Anne-Marie" "The Incredible Bulk")
; => ("Get off my lawn, Billy!!!"
      "Get off my lawn, Anne-Marie!!!"
      "Get off my lawn, The Incredible Bulk!!!")
```

正如你所看到的，当你为变量性质的函数提供参数时，参数被当作一个列表来处理。你可以把可变参数和普通参数混在一起，但可变参数必须放在最后。

```
(defn favorite-things
  [name & things]
  (str "Hi, " name ", here are my favorite things: "
       (clojure.string/join ", " things)))

(favorite-things "Doreen" "gum" "shoes" "kara-te")
; => "Hi, Doreen, here are my favorite things: gum, shoes, kara-te"
```

最后，Clojure 有一种更复杂的定义参数的方法，叫做 _解构_ ，这值得有自己的小节。

#### 解构

解构的基本思想是，它可以让你在一个集合中简洁地将名字与值绑定。让我们看看一个基本的例子。

```
;; Return the first element of a collection
(defn my-first
  [[first-thing]] ; Notice that first-thing is within a vector
  first-thing)

(my-first ["oven" "bike" "war-axe"])
; => "oven"
```

这里，`my-first`函数将符号`first-thing`与作为参数传入的 Vector 中的第一个元素联系起来。你告诉`my-first`这样做，就是把符号`first-thing`放在一个 Vector 中。

Vector 就像一个巨大的牌子，对 Clojure 说："嘿！这个函数将收到一个列表或 Vector 作为参数。为了让我的生活更轻松，请帮我拆开参数的结构，并将有意义的名字与参数的不同部分联系起来！" 当对一个 Vector 或列表进行解构时，你可以随意命名你想要的元素，也可以使用其他参数。

```
(defn chooser
  [[first-choice second-choice & unimportant-choices]]
  (println (str "Your first choice is: " first-choice))
  (println (str "Your second choice is: " second-choice))
  (println (str "We're ignoring the rest of your choices. "
                "Here they are in case you need to cry over them: "
                (clojure.string/join ", " unimportant-choices))))

(chooser ["Marmalade", "Handsome Jack", "Pigpen", "Aquaman"])
; => Your first choice is: Marmalade
; => Your second choice is: Handsome Jack
; => We're ignoring the rest of your choices. Here they are in case \
     you need to cry over them: Pigpen, Aquaman
```

这里，其余的参数`unimportant-choices`处理用户在第一和第二选择之后的任何数量的额外选择。

你也可以对 Map 进行解构。就像你告诉 Clojure 通过提供一个 Vector 作为参数来对 Vector 或列表解构一样，你也可以通过提供一个 Map 作为参数来对 Map 进行解构。

```
(defn announce-treasure-location
➊   [{lat :lat lng :lng}]
  (println (str "Treasure lat: " lat))
  (println (str "Treasure lng: " lng)))

(announce-treasure-location {:lat 28.22 :lng 81.33})
; => Treasure lat: 28.22
; => Treasure lng: 81.33
```

让我们更详细地看看➊的那一行。这就像告诉 Clojure，"哟！Clojure! 为我做一件事，把`lat`这个名字与键`:lat`对应的值联系起来。对`lng`和`:lng`做同样的事情，好吗？"

我们经常想直接把关键词从 Map 中分离出来，所以有一个更短的语法。这和前面的例子有相同的结果。

```
(defn announce-treasure-location
  [{:keys [lat lng]}] 。
  (println (str "Treasure lat: " lat))
  (println (str "Treasure lng: " lng)))
```

你可以通过使用`:as`关键字保留对原始 Map 参数的访问。在下面的例子中，原始 Map 是用`treasure-location`来访问的。

```
(defn receive-treasure-location
  [{:keys [lat lng] :as treasure-location}]
  (println (str "Treasure lat: " lat))
  (println (str "Treasure lng: " lng))

  ;; One would assume that this would put in new coordinates for your ship
  (steer-ship! treasure-location))
```

一般来说，你可以把解构看作是指示 Clojure 如何将名字与列表、Map、集合或 Vector 中的值联系起来。现在，我们来看看函数中真正起作用的部分：函数体!

#### 函数体

函数主体可以包含任何形式 的 Form。Clojure 会自动返回最后求值的 Form。这个函数体只包含三种 Form，当你调用这个函数时，它会吐出最后一种 Form，`"joe"`。

```
(defn illustrative-function
  []
  (+ 1 304)
  30
  "joe")

(exstrative-function)
; => "joe"
```

下面是另一个函数体，它使用一个`if`表达式。

```
(defn number-comment
  [x]
  (if (> x 6)
    "Oh my gosh! What a big number!"
    "That number's OK, I guess"))

(number-comment 5)
; => "That number's OK, I guess"

(number-comment 7)
; => "Oh my gosh! What a big number!"
```

#### 所有函数都是平等的

最后说明一下：Clojure 没有特权函数。`+`只是一个函数，`-`只是一个函数，而`inc`和`map`也只是函数。它们并不比你自己定义的函数好。所以，不要让他们给你任何口实!

更重要的是，这个事实有助于证明 Clojure 的底层简单性。在某种程度上，Clojure 是非常愚蠢的。当你进行函数调用时，Clojure 只是说，"`map`？当然，不管怎样! 我只是应用这个并继续前进"。它并不关心这个函数是什么，或者它来自哪里；它对所有的函数都一视同仁。在它的核心，Clojure 并不关心加法、乘法或 Map 的问题。它只关心函数的应用。

当你继续用 Clojure 编程时，你会发现这种简单性是很理想的。你不必为处理不同的函数而担心特殊的规则或语法。它们的工作原理都是一样的!

### 匿名函数

在 Clojure 中，函数不需要有名字。事实上，你会一直使用 _匿名函数_ 。多么神秘啊! 你可以通过两种方式创建匿名函数。第一种是使用`fn`Form。

```
(fn [param-list]
  function body)
```

看起来很像`defn`，不是吗？让我们试一试几个例子。

```
(map (fn [name] (str "Hi, " name))
     ["Darth Vader" "Mr. Magoo"])
; => ("Hi, Darth Vader" "Hi, Mr. Magoo")

((fn [x] (* x 3)) 8)
; => 24
```

你可以用处理`defn`的方式来处理`fn`，这几乎是相同的。参数列表和函数体的工作原理完全相同。你可以使用参数解构，可变参数，等等。你甚至可以将你的匿名函数与一个名字联系起来，这不应该是一个惊喜（如果这确实是一个惊喜，那么 ... ... 惊喜！）。

```
(def my-special-multiplier (fn [x] (* x 3))
(my-special-multiplier 12)
; => 36
```

Clojure 还提供了另一种更紧凑的方式来创建匿名函数。下面是一个匿名函数的样子。

```
#(* % 3)
```

哇，这看起来很奇怪。来吧，应用这个看起来很奇怪的函数。

```
(#(* % 3) 8)
; => 24
```

下面是一个将匿名函数作为参数传递给 map 的例子。

```
(map #(str "Hi, " %)
     ["Darth Vader" "Mr. Magoo"])
; => ("Hi, Darth Vader" "Hi, Mr. Magoo")
```

这种看起来很奇怪的匿名函数的编写方式是由一个叫做 _reader macros_ 的函数实现的。你会在第 7 章中了解到这些。现在，只学习如何使用这些匿名函数就可以了。

你可以看到，这种语法肯定更紧凑，但也有点奇怪。让我们把它分解一下。这种匿名函数看起来很像函数调用，只是它前面有一个哈希标记，`#`。

```
;; Function call
(* 8 3)

;; Anonymous function
#(* % 3)
```

这种相似性使你能更快地看到应用这个匿名函数时将发生什么。"哦，"你可以对自己说，"这是要把它的参数乘以 3"。

现在你可能已经猜到了，百分号`%`，表示传递给函数的参数。如果你的匿名函数需要多个参数，你可以像这样区分它们。`%1`, `%2`, `%3`, 以此类推。`%`相当于`%1`。

```
(#(str %1 " and " %2) "cornbread" "butter beans")
; => "cornbread and butter beans"
```

你也可以用`%&`传递其余参数。

```
(#(identity %&) 1 "blarg" :yip)
; => (1 "blarg" :yip)
```

在这种情况下，你将 identity 函数应用于其余参数。identity 会返回给定的参数，而不改变它。可变参数是以列表 Form 存储的，所以应用 identity 函数会返回所有参数的列表。

如果你需要写一个简单的匿名函数，使用这种风格是最好的，因为它在视觉上很紧凑。另一方面，如果你要写一个更长、更复杂的函数，它很容易变得不可读。如果是这种情况，请使用`fn`。

### 返回函数

现在你已经看到，函数可以返回其他函数。返回的函数是 _closures_ ，这意味着它们可以访问函数创建时的作用域内的所有变量。下面是一个标准的例子。

```
(defn inc-maker
  "Create a custom incrementor"
  [inc-by]
  #(+ % inc-by))

(def inc3 (inc-maker 3))

(inc3 7)
; => 10
```

这里，`inc-by`在作用域内，所以即使返回的函数在`inc-maker`之外使用，也可以访问它。

## 把这一切放到一起

![img](https://www.braveclojure.com/assets/images/cftbat/do-things/model-hobbit.png)

好了! 是时候把你新发现的知识用于一个崇高的目的了：打倒霍比特人! 要打一个霍比特人，你首先要建立它的身体部位模型。每个身体部位都将包括其相对大小，以表明该部位被击中的可能性有多大。为了避免重复，霍比特人的模型将只包括 _左脚_ ， _左耳_ 的条目，以此类推。因此，你需要一个函数来完全对称该模型，创建 _右脚_ ， _右耳_ ，等等。最后，你将创建一个函数，迭代身体各部分，并随机选择击中的部分。在这一过程中，你将了解到一些新的 Clojure 工具。`let`表达式，循环，和正则表达式。有趣!

### 夏尔的下一个顶级模型

对于我们的霍比特人模型，我们将避开霍比特人的特征，如活泼和调皮，只关注霍比特人的小身板。下面是霍比特人的模型。

```
(def asym-hobbit-body-parts [{:name "head" :size 3}
                             {:name "left-eye" :size 1}
                             {:name "left-ear" :size 1}
                             {:name "mouth" :size 1}
                             {:name "nose" :size 1}
                             {:name "neck" :size 2}
                             {:name "left-shoulder" :size 3}
                             {:name "left-upper-arm" :size 3}
                             {:name "chest" :size 10}
                             {:name "back" :size 10}
                             {:name "left-forearm" :size 3}
                             {:name "abdomen" :size 6}
                             {:name "left-kidney" :size 1}
                             {:name "left-hand" :size 2}
                             {:name "left-knee" :size 2}
                             {:name "left-thigh" :size 4}
                             {:name "left-lower-leg" :size 3}
                             {:name "left-achilles" :size 1}
                             {:name "left-foot" :size 2}])
```

这是一个包含 Map 的 Vector。每个 Map 都有身体部位的名称和身体部位的相对大小。(我知道只有动漫人物的眼睛是头部的三分之一大小，但就这样吧，好吗？)

明显缺少的是霍比特人的右侧。让我们来解决这个问题。清单 3-1 是到目前为止你看到的最复杂的代码，它引入了一些新的想法。但是不要担心，因为我们将详细地研究它。

```
(defn matching-part
  [part]
  {:name (clojure.string/replace (:name part) #"^left-" "right-")
   :size (:size part)})

(defn symmetrize-body-parts
  "Expects a seq of maps that have a :name and :size"
  [asym-body-parts]
  (loop [remaining-asym-parts asym-body-parts
         final-body-parts []]
    (if (empty? remaining-asym-parts)
      final-body-parts
      (let [[part & remaining] remaining-asym-parts]
        (recur remaining
               (into final-body-parts
                     (set [part (matching-part part)])))))))
```

1. 3-1. 匹配-部分和对称-身体-部分的函数

当我们对`asym-hobbit-body-parts`调用函数`symmetriz-body-parts`时，我们得到一个完全对称的霍比特人。

```
(symmetrize-body-parts asym-hobbit-body-parts)
; => [{:name "head", :size 3}
      {:name "left-eye", :size 1}
      {:name "right-eye", :size 1}
      {:name "left-ear", :size 1}
      {:name "right-ear", :size 1}
      {:name "mouth", :size 1}
      {:name "nose", :size 1}
      {:name "neck", :size 2}
      {:name "left-shoulder", :size 3}
      {:name "right-shoulder", :size 3}
      {:name "left-upper-arm", :size 3}
      {:name "right-upper-arm", :size 3}
      {:name "chest", :size 10}
      {:name "back", :size 10}
      {:name "left-forearm", :size 3}
      {:name "right-forearm", :size 3}
      {:name "abdomen", :size 6}
      {:name "left-kidney", :size 1}
      {:name "right-kidney", :size 1}
      {:name "left-hand", :size 2}
      {:name "right-hand", :size 2}
      {:name "left-knee", :size 2}
      {:name "right-knee", :size 2}
      {:name "left-thigh", :size 4}
      {:name "right-thigh", :size 4}
      {:name "left-lower-leg", :size 3}
      {:name "right-lower-leg", :size 3}
      {:name "left-achilles", :size 1}
      {:name "right-achilles", :size 1}
      {:name "left-foot", :size 2}
      {:name "right-foot", :size 2}]
```

让我们来分析一下这段代码!

### let

在清单 3-1 的大量疯狂中，你可以看到结构`(let ...)`的 Form。让我们通过一个例子来建立对`let`的理解，当我们熟悉了所有的部分后，再来检查程序中的完整例子。

`let`将名字与值绑定。你可以认为`let`是_let_ _it_ _be_的缩写，这也是披头士乐队关于编程的一首优美的歌曲。这里有一个例子。

```
(let [x 3]
  x)
; => 3

(def dalmatian-list
  ["Pongo" "Perdita" "Puppy 1" "Puppy 2"])
(let [dalmatians (take 2 dalmatian-list)]
  dalmatians)
; => ("Pongo" "Perdita")
```

在第一个例子中，你将名字`x`与值`3`绑定。在第二个例子中，你把名字`dalmatians`绑定到表达式`(取2`dalmatian`-list)`的结果，也就是列表`("Pongo" "Perdita")`。`let`还引入了一个新的作用域。

```
(def x 0)
(let [x 1] x)
; => 1
```

这里，你首先使用`def`将名字`x`绑定到值`0`上。然后，`let`创建了一个新的作用域，在这个作用域中，名字`x`被绑定到值`1`上。我认为作用域取决于上下文。例如，在 "请清理这些烟头 "这句话中， _烟头_ 的含义是不同的，这取决于你是在产科病房工作还是在香烟制造商大会的监管人员工作。在这个代码片段中，你在说："我希望`x`在全局上下文中是`0`，但在这个`let`表达式的上下文中，它应该是`1`。"

你可以在你的`let`绑定中引用现有的绑定。

```
(def x 0)
(let [x (inc x)] x)
; => 1
```

在这个例子中，`(inc x)`中的`x`是指由`(def x 0)`创建的绑定。结果是`1`，然后在`let`创建的新作用域中与名称`x`绑定。在`let`Form 的作用域内，`x`指的是`1`，而不是`0`。

你也可以在`let`中使用可变参数，就像你在函数中一样。

```
(let [[Pongo & dalmatians] dalmatian-list] [Pongo dalmatians]
  [Pongo dalmatians])
; => ["Pongo" ("Perdita" "Puppy 1" "Puppy 2") ]
```

注意，`let`Form 的值是其主体中最后被求值的 Form。`let`Form 遵循所有在["调用函数"第 48 页](https://www.braveclojure.com/do-things/#Anchor)中介绍的析构规则。在这个例子中，`[pongo & dalmatians]`解构了`dalmatian-list`，将字符串`"Pongo "绑定到名称`pongo`上，将其余的dalmatians列表绑定到`dalmatians`上。Vector`\[pongo dalmatians]`是`let`的最后一个表达式，所以它是`let\`Form 的值。

`let`Form 有两个主要用途。首先，它们通过允许你对事物进行命名。其次，它们允许你只求值一个表达式，并重复使用其结果。当你需要重复使用一个昂贵的函数调用的结果时，这一点特别重要，比如网络 API 调用。当表达式有副作用时，这也很重要。

让我们再看一下我们的对称函数中的`let`Form，这样我们就能明白到底发生了什么。

```
(let [[part & remaining] remaining-asym-parts]
  (recur remaining
         (in into final-body-parts
               (set [part (matching-part part part)]))))    
```

这段代码告诉 Clojure，"创建一个新的作用域。在它里面，将`part`与`remaining-asym-parts`的第一个元素相关联。将`remaining`与`remaining-asym-parts`中的其他元素联系起来"。

至于`let`表达式的主体，你将在下一节中了解到`recur`的含义。函数调用

```
(into final-body-parts
  (set [part (matching-part part)]))
```

首先告诉 Clojure, "使用`set`函数创建一个由`part`和它的匹配部分组成的集合。然后使用函数`into`将该集合的元素添加到 Vector`final-body-parts`中"。你在这里创建一个集合，以确保你向`final-body-parts`添加唯一的元素，因为`part`和`(matching-part part)`有时是同一个东西，正如你将在接下来的正则表达式部分看到的。下面是一个简化的例子。

```
(into [] (set [:a :a]))
; => [:a]
```

首先，`(set [:a :a])`返回集合`#{:a}`，因为集合不包含重复的元素。然后`(into [] #{:a})`返回 Vector`[:a]`。

回到`let`：注意`part`在`let`的主体中被多次使用。如果我们使用原来的表达式，而不是使用`part`和`remaining`的名字，那将是一个混乱的局面! 下面是一个例子。

```
(recur (rest remaining-asym-parts)
       (in into final-body-parts
             (set [(first remaining-asym-parts) (matching-part (first remaining-asym-parts)) ]))
```

所以，`let`是一种方便的方法，可以为值引入本地名称，这有助于简化代码。

### 循环

在我们的`symmetrize-body-parts`函数中，我们使用了`loop`，它提供了另一种在 Clojure 中进行递归的方法。让我们看看一个简单的例子。

```clojure
(loop [iteration 0]
  (println (str "Iteration " iteration))
  (if (> iteration 3)
    (println "Goodbye!")
    (recur (inc iteration))))
; => Iteration 0
; => Iteration 1
; => Iteration 2
; => Iteration 3
; => Iteration 4
; => Goodbye!
```

第一行，`loop [iteration 0]`，开始了循环并引入了一个初始值的绑定。在循环的第一次传递中，`iteration`的值为 0.接下来，它打印一个短消息。然后，它检查`iteration`的值。如果该值大于 3，那么是时候说再见了。否则，我们就 "重来"。这就好比`loop`创建了一个匿名函数，其参数名为`iteration`，而`recur`允许你从其内部调用该函数，传递参数`(inc iteration)`。

事实上，你可以通过使用一个普通的函数定义来完成同样的事情。

```clojure
(defn recursive-printer
  ([]
     (recursive-printer 0))
  ([iteration]
     (println iteration)
     (if (> iteration 3)
       (println "Goodbye!")
       (recursive-printer (inc iteration)))))
(recursive-printer)
; => Iteration 0
; => Iteration 1
; => Iteration 2
; => Iteration 3
; => Iteration 4
; => Goodbye!
```

但正如你所看到的，这是个比较啰嗦的方法。而且，`loop`有更好的性能。在我们的对称化函数中，我们将使用`loop`遍历不对称的身体部位列表中的每个元素。

### 正则表达式

_正则表达式_是对文本进行模式匹配的工具。正则表达式的文字符号是将表达式放在哈希标记后的引号中。

```
#"regular-expression"
```

在清单 3-1 中的函数`matching-part`中，`clojure.string/replace`使用正则表达式`#"^left-"`来匹配以`"left-"`开头的字符串，以便用`"right-"`替换`"left-"`。卡特，`^`，是正则表达式发出的信号，即只有当文本`"left-"`位于字符串的开头时，它才会匹配，这就确保了像`"cleft-chin"`这样的字符串不会匹配。你可以用`re-find`来测试，它检查一个字符串是否与正则表达式描述的模式相匹配，如果不匹配，则返回匹配的文本或`nil`。

```clojure
(re-find #"^left-" "left-eye")
; => "left-"

(re-find #"^left-" "cleft-chin")
; => nil

(re-find #"^left-" "wongleblart")
; => nil
```

下面是几个`matching-part`的例子，使用一个重词将`"left-"`替换为`"right-"`。

```clojure
(defn matching-part
  [part]
  {:name (clojure.string/replace (:name part) #"^left-" "right-")
   :size (:size part)})
(matching-part {:name "left-eye" :size 1})
; => {:name "right-eye" :size 1}]

(matching-part {:name "head" :size 3})
; => {:name "head" :size 3}]
```

请注意，名称 "head" 是原样返回的。

### 对称器

现在让我们回到完整的对称器，对其进行更详细的分析。

```clojure
(def asym-hobbit-body-parts [{:name "head" :size 3}
                             {:name "left-eye" :size 1}
                             {:name "left-ear" :size 1}
                             {:name "mouth" :size 1}
                             {:name "nose" :size 1}
                             {:name "neck" :size 2}
                             {:name "left-shoulder" :size 3}
                             {:name "left-upper-arm" :size 3}
                             {:name "chest" :size 10}
                             {:name "back" :size 10}
                             {:name "left-forearm" :size 3}
                             {:name "abdomen" :size 6}
                             {:name "left-kidney" :size 1}
                             {:name "left-hand" :size 2}
                             {:name "left-knee" :size 2}
                             {:name "left-thigh" :size 4}
                             {:name "left-lower-leg" :size 3}
                             {:name "left-achilles" :size 1}
                             {:name "left-foot" :size 2}])


(defn matching-part
  [part]
  {:name (clojure.string/replace (:name part) #"^left-" "right-")
   :size (:size part)})

➊ (defn symmetrize-body-parts
  "Expects a seq of maps that have a :name and :size"
  [asym-body-parts]
➋   (loop [remaining-asym-parts asym-body-parts 
         final-body-parts []]
➌     (if (empty? remaining-asym-parts) 
      final-body-parts
➍       (let [[part & remaining] remaining-asym-parts] 
➎         (recur remaining 
               (into final-body-parts
                     (set [part (matching-part part)])))))))
```

`symmetriz-body-parts`函数（从➊开始）采用了函数式编程中常见的一般策略。给定一个序列（在本例中，是一个身体部位及其尺寸的 Vector），该函数连续地将该序列分割成 _head_ 和 _tail_ 。然后，它处理头部，将其添加到某个结果中，并使用递归来继续处理尾部的过程。

我们在➋处开始循环处理主体部分。序列的尾部将被绑定到`remaining-asym-parts`。最初，它被绑定到传递给函数的完整序列：`asym-body-parts`。我们还创建了一个结果序列，`final-body-parts`；它的初始值是一个空 Vector。

如果`remaining-asym-parts`在➌处是空的，这意味着我们已经处理了整个序列，可以返回结果，`final-body-parts`。否则，在➍，我们将列表分成`head`，`part`，和`tail`，`remaining`。

在➎处，我们用`remaining`进行循环，这个列表在循环的每一次迭代中都会缩短一个元素，还有`(in)`表达式，它建立了对称的身体部分的 Vector。

如果你是这种编程的新手，这段代码可能需要一些时间来解决。请坚持下去! 一旦你理解了正在发生的事情，你会觉得自己像个百万富翁!

### 用 reduce 来编写更好的对称器

处理序列的每个元素并返回一个结果的模式非常普遍，以至于有一个内置的函数叫做`reduce`。下面是一个简单的例子。

```clojure
;; sum with reduce
(reduce + [1 2 3 4])
; => 10
```

这就像告诉 Clojure 这样做。

```clojure
(+ (+ (+ 1 2) 3) 4)
```

`reduce`函数按照以下步骤工作。

1. 将给定的函数应用于一个序列的前两个元素。这就是`(+ 1 2)`的由来。
2. 将给定的函数应用于结果和序列的下一个元素。在本例中，步骤 1 的结果是`3`，序列的下一个元素也是`3`。所以最后的结果是`(+3 3)`。
3. 对序列中剩下的每个元素重复第 2 步。

`reduce`也需要一个可选的初始值。这里的初始值是`15`。

```clojure
(reduce + 15 [1 2 3 4])
```

如果你提供了一个初始值，`reduce`就会开始对初始值和序列的第一个元素应用给定的函数，而不是序列的前两个元素。

需要注意的一个细节是，在这些例子中，`reduce`接收一个元素的集合，`[1 2 3 4]`，并返回一个单一的数字。虽然程序员经常这样使用`reduce`，但你也可以使用`reduce`来返回一个比你开始时更大的集合，就像我们在`symmetrize`-body-parts`中尝试做的那样。`reduce`抽象了 "处理一个集合并建立一个结果 "的任务，它对返回的结果类型是不确定的。为了进一步了解`reduce\`的工作原理，这里有一种方法可以实现它。

```clojure
(defn my-reduce
  ([f initial coll]
   (loop [result initial
          remaining coll]
     (if (empty? remaining)
       result
       (recur (f result (first remaining)) (rest remaining)))))
  ([f [head & tail]]
   (my-reduce f head tail)))
```

我们可以重新实现我们的对称器，如下所示。

```clojure
(defn better-symmetrize-body-parts
  "Expects a seq of maps that have a :name and :size"
  [asym-body-parts]
  (reduce (fn [final-body-parts part]
            (into final-body-parts (set [part (matching-part part)])))
          []
          asym-body-parts))
```

真棒! 使用`reduce`的一个显而易见的好处是，你写的代码总体上更少。你传递给`reduce`的匿名函数只专注于处理一个元素和建立一个结果。原因是`reduce`处理了底层的机制，即跟踪哪些元素已经被处理，并决定是否返回一个最终结果或递归。

使用`reduce`也更有表现力。如果你的代码的读者遇到 "loop"，如果不阅读所有的代码，他们将不能确定这个循环到底在做什么。但是如果他们看到`reduce`，他们会立即知道代码的目的是处理一个集合的元素以建立一个结果。

最后，通过将 "reduce "过程抽象为一个以另一个函数为参数的函数，你的程序变得更有可塑性。例如，你可以将`reduce`函数作为一个参数传递给其他函数。你还可以创建一个更通用的 "对称体-部件 "版本，例如 "扩展体-部件"。除了身体部位的列表外，它还可以接受一个 _扩展器_ 函数，并让你的模型不仅仅是霍比特人。例如，你可以有一个蜘蛛扩展器，可以增加眼睛和腿的数量。我会让你自己来写，因为我是邪恶的。

### 霍比特人的暴力

我的话，这真是为勇敢和真实的人准备的 Clojure! 为了给你的工作画上句号，这里有一个函数可以确定霍比特人的哪个部分被击中。

```clojure
(defn hit
  [asym-body-parts]
  (let [sym-parts (➊better-symmetrize-body-parts asym-body-parts)
        ➋body-part-size-sum (reduce + (map :size sym-parts))
        target (rand body-part-size-sum)]
    ➌(loop [[part & remaining] sym-parts
           accumulated-size (:size part)]
      (if (> accumulated-size target)
        part
        (recur remaining (+ accumulated-size (:size (first remaining))))))))
```

`hit`的工作原理是取一个不对称的身体部位的 Vector，在➊处将其对称，然后在➋处将各部位的大小相加。一旦我们将这些尺寸相加，就好像从 1 到`body-part-size-sum`的每个数字都对应于一个身体部位；1 可能对应于左眼，而 2、3、4 可能对应于头部。这使得当你击中一个身体部位时（通过在这个范围内选择一个随机数字），特定身体部位被击中的可能性将取决于身体部位的大小。

![img](https://www.braveclojure.com/assets/images/cftbat/do-things/hobbit-hit-line.png)

图 3-1：身体部位与数字的范围相对应，如果目标在这个范围内，就会被击中。

最后，这些数字中的一个被随机选择，然后我们在➌处使用`loop`来寻找并返回与该数字对应的身体部位。循环是通过跟踪我们已经检查过的部分的累计大小，并检查累计大小是否大于目标值来实现的。我把这个过程想象成用一排编号的槽来排列身体部位。在我排完一个身体部位后，我问自己："我已经达到目标了吗？" 如果我达到了，这意味着我刚刚排好的身体部位就是被击中的那个部位。否则，我就继续排查这些部位。

例如，假设你的零件清单是 _头_ 、 _左眼_ 和 _左手_ ，如图 3-1。在取完第一个部分，即头部后，累计大小为 3。当累计大小超过目标时，身体部分就被击中，所以如果目标小于 3，那么头部就被击中了。否则，你取下下一个部分，即左眼，并将累积大小增加到 4，如果目标大于或等于 3 且小于 4，则产生一个命中。

下面是一些`hit`函数的运行样本。

```clojure
(hit asym-hobbit-body-parts)
; => {:name "right-upper-arm", :size 3}

(hit asym-hobbit-body-parts)
; => {:name "chest", :size 10}

(hit asym-hobbit-body-parts)
; => {:name "left-eye", :size 1}
```

哦，我的上帝，那个可怜的霍比特人！你这个怪物！

## 总结

本章让你对如何在 Clojure 中进行 _操作_ 有了一个龙卷风摧毁停车场式的了解。你现在知道了如何用字符串、数字、Map、关键字、Vector、列表和 Set 来表示信息，以及如何用`def`和`let`来命名这些表达式。你已经了解了函数的灵活性以及如何创建你自己的函数。此外，你还了解了 Clojure 的简单哲学，包括其统一的语法和强调在原始数据类型上使用大型函数库。

第 4 章将带你详细探究 Clojure 的核心函数，第 5 章解释了函数式编程的思维模式。本章向你展示了如何编写 Clojure 代码--接下来的两章将向你展示如何更好的编写 Clojure。

在这一点上，我建议你开始写代码，我的每一根纤维都是这样。没有比这更好的方法来巩固你的 Clojure 知识了。Clojure Cheat Sheet（[_http://clojure.org/api/cheatsheet_](http://clojure.org/api/cheatsheet)）是一个很好的参考资料，它列出了所有在本章中涉及的数据结构上操作的内置函数。

下面的练习会让你的大脑非常兴奋。如果你想更多地测试你的新技能，可以尝试一些 _[Project Euler](http://www.projecteuler.net)_ 挑战。你还可以看看 _4Clojure（[http://www.4clojure.com/problems/](http://www.4clojure.com/problems/)_），这是一套在线的 Clojure 问题，旨在测试你的知识。来随便写点什么吧!

## 练习

这些练习是为了测试你的 Clojure 知识和学习更多的 Clojure 函数，是一种有趣的方式。前三个可以只用本章介绍的信息来完成，但后三个需要你使用到目前为止还没有涉及的函数。如果你真的很想写更多的代码并探索 Clojure 的标准库，那么就去解决后三个问题。如果你觉得这些练习太难了，可以在读完第 4 章和第 5 章后再来看看，你会发现它们要容易得多。

1. 使用`str`, `vector`, `list`, `hash-map`, 和`hash-set`函数。
2. 编写一个函数，接收一个数字，并向其添加 100。
3.  写一个 `dec-maker` 函数，其工作原理与函数 `inc-maker`完全相同，除了用减法。

  期望的结果：
    ```clojure
    (def dec9 (dec-maker 9))
    (dec9 10)
    ; => 1
    ```
4.  写一个函数 `mapset` ,其工作原理与函数 `map`完全相同，除了返回值是一个集合。

    ```clojure
    (mapset inc [1 1 2 2])
    ; => #{2 3}
    ```
5. 创建一个类似于`symmetriz-body-parts`的函数，只是它必须与具有径向对称性的奇怪的太空外星人一起工作。他们没有两只眼睛、胳膊、腿等等，而是有五只。
6. 创建一个函数，将`symmetriz-body-parts`和你在练习 5 中创建的函数通用化。这个新的函数应该接受一个身体部位的集合，以及要增加的匹配身体部位的数量。如果你对 Lisp 语言和函数式编程是完全陌生的，那么如何做到这一点可能并不明显。如果你被卡住了，只需转到下一章，以后再重温这个问题。
