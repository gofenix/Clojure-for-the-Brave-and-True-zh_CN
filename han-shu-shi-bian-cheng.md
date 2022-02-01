# 函数式编程

到目前为止，你已经专注于熟悉 Clojure 提供的工具：不可变的数据结构、函数、抽象，等等。在这一章中，你将学习如何思考你的编程任务，以最好的方式利用这些工具。你将开始把你的经验整合到一个新的函数式编程思维中。

你将学到的核心概念包括：什么是纯函数，为什么它们很有用；如何使用不可变的数据结构，为什么它们比可变的表亲更有优势；如何将数据和函数分开，给你带来更多的力量和灵活性；以及为什么对一小部分数据抽象进行编程会很强大。一旦你把所有这些知识塞进你的大脑，你就会有一个全新的解决问题的方法

在学习了这些主题之后，你将通过编写一个基于终端的游戏来运用你所学到的一切，这个游戏的灵感来自于美国各地 Cracker Barrel 餐馆中的一种古老而神秘的思维训练装置。Peg Thing!

## 纯函数：是什么和为什么

除了 "println "和 "rand"，到目前为止，你所使用的所有函数都是纯函数。是什么使它们成为纯函数，为什么会有这样的问题？如果一个函数符合两个条件，它就是纯函数。

* 如果给出相同的参数，它总是返回相同的结果。这被称为_引用透明度_，你可以把它添加到你的 5 美元编程术语列表中。
* 它不能引起任何副作用。也就是说，该函数不能做出任何在函数本身之外可以观察到的改变--例如，通过改变一个外部可访问的可改变对象或写到一个文件。

这些特性使你更容易推理你的程序，因为这些函数是完全隔离的，无法影响你系统的其他部分。当你使用它们时，你不必问自己，"我调用这个函数会破坏什么？" 它们也是一致的：你永远不需要搞清楚为什么给一个函数传递相同的参数会导致不同的返回值，因为这永远不会发生。

纯函数和算术一样稳定，没有问题（你最后一次为两个数字相加而烦恼是什么时候？） 它们是巨大的函数小砖块，你可以自信地将其作为你程序的基础。让我们更详细地看看引用透明性和无副作用，看看它们到底是什么，以及它们是如何发挥作用的。

### 纯函数是引用透明的

为了在调用相同参数时返回相同的结果，纯函数只依靠 1）自己的参数和 2）不可变的值来决定其返回值。例如，数学函数是引用透明的。

```clojure
(+ 1 2)
; => 3
```

如果一个函数依赖于一个不可变的值，那么它就是引用透明的。字符串\`", Daniel-san "是不可变的，所以下面的函数也是引用透明的。

```clojure
(defn wisdom
  [words]
  (str words ", Daniel-san"))

(wisdom "Always bathe on Fridays")
; => "Always bathe on Fridays, Daniel-san"
```

相比之下，下面的函数在相同的参数下不会产生相同的结果；因此，它们在指称上是不透明的。任何依赖随机数生成器的函数都不可能是指称透明的。

```clojure
(defn year-end-evaluation
  []
  (if (> (rand) 0.5)
    "You get a raise!"
    "Better luck next year!"))
```

如果你的函数从一个文件中读出，它就不是引用透明的，因为文件的内容可以改变。下面的函数`analyze-file`不是引用透明的，但函数`analysis`是透明的。

```clojure
(defn analyze-file
  [filename]
  (analysis (slurp filename)))

(defn analysis
  [text]
  (str "Character count: " (count text)))
```

当使用一个引用透明的函数时，你永远不必考虑哪些可能的外部条件会影响函数的返回值。如果你的函数在多个地方被使用，或者它被深深地嵌套在一个函数调用链中，这一点就特别重要。在这两种情况下，你可以高枕无忧地知道，外部条件的变化不会导致你的代码中断。

另一种思考方式是，现实在很大程度上是引用透明的。如果你把重力看作一个函数，那么引力就是在两个物体上调用该函数的返回值。因此，当你下次参加编程面试时，你可以通过把面试官桌上的东西打掉来证明你的函数式编程知识（这也证明你知道如何在一个集合上应用一个函数）。

### 纯函数没有副作用

执行副作用就是在一个给定的范围内改变一个名字和它的值之间的关联。下面是一个 JavaScript 的例子。

```clojure
var haplessObject = {
  emotion: "Carefree!"
};

var evilMutator = function(object){
  object.emotion = "So emo :'(";
}

evilMutator(haplessObject);
haplessObject.emotion;
// => "So emo :'("
```

当然，你的程序必须要有一些副作用。它写入磁盘，改变了文件名和磁盘扇区集合之间的关联；它改变了显示器像素的 RGB 值；等等。否则，运行它就没有意义了。

然而，副作用是潜在的有害的，因为它们带来了关于你的代码中的名称所指的不确定性。这就导致了很难追踪为什么以及如何将一个名字与一个值联系起来的情况，这就使程序的调试变得非常困难。当你调用一个没有副作用的函数时，你只需要考虑输入和输出之间的关系。你不必担心其他可能在你的系统中出现的变化。

另一方面，有副作用的函数给你的思想葡萄带来了更多的负担：现在你必须担心当你调用这个函数时，世界是如何受到影响的。不仅如此，每一个依赖于副作用函数的函数都会被这种担忧所感染；它也会成为你在构建程序时需要格外小心和思考的另一个组件。

如果你有使用 Ruby 或 JavaScript 等语言的重要经验，你可能已经遇到了这个问题。当一个对象被传来传去的时候，它的属性不知不觉地发生了变化，而你却不知道为什么。然后你不得不买一台新的电脑，因为你把你的电脑扔到了窗外。如果你读过任何关于面向对象设计的文章，你就会知道，很多文章都是关于管理状态和减少副作用的策略，正是因为这个原因。

由于所有这些原因，在你的代码中寻找限制使用副作用的方法是个好主意。幸运的是，Clojure 通过不遗余力地限制副作用来使你的工作变得更容易--它的所有核心数据结构都是不可改变的。无论你如何努力，你都无法在原地改变它们。然而，如果你不熟悉不可变的数据结构，你可能会觉得你最喜欢的工具被剥夺了。你怎么能\*做没有副作用的事情呢？好吧，这就是下一节要讲的内容! 这段话怎么样，嗯？诶？

## 与不可变的数据结构共处

不可变的数据结构确保你的代码不会有副作用。正如你现在衷心知道的，这是一件好事。但你如何在没有副作用的情况下完成任何事情呢？

### 递归而不是 for/while

如果你曾经在 JavaScript 中写过这样的东西，请举手。

```clojure
var wrestlers = getAlligatorWrestlers();
var totalBites = 0;
var l = wrestlers.length;

for(var i=0; i < l; i++){
  totalBites += wrestlers[i].timesBitten;
}
```

或者这样。

```clojure
var allPatients = getArkhamPatients();
var analyzedPatients = [];
var l = allPatients.length;

for(var i=0; i < l; i++){
  if(allPatients[i].analyzed){
    analyzedPatients.push(allPatients[i]);
  }
}
```

注意这两个例子都对循环变量`i`以及循环外的一个变量（第一个例子中的`totalBites`和第二个例子中的`analyzedPatients`）产生了副作用。以这种方式使用副作用--改变\*\*\*内部的变量--是相当无害的。你在创造新的值，而不是改变你从程序中其他地方得到的对象。

![img](https://www.braveclojure.com/assets/images/cftbat/functional-programming/bloodthunder.png)

但是 Clojure 的核心数据结构甚至不允许这些无害的变异。那么，你能做什么呢？首先，忽略一个事实，你可以很容易地使用`map`和`reduce`来完成前面的工作。在这些情况下--对一些集合进行迭代以建立一个结果--替代突变的函数是递归。

让我们看一下第一个例子，建立一个总和。Clojure 没有赋值运算符。如果不创建一个新的作用域，你就无法将一个新的值与一个名字联系起来。

```clojure
(def great-baby-name "Rosanthony")
great-baby-name
; => "Rosanthony"

(let [great-baby-name "Bloodthunder"]
  great-baby-name)
; => "Bloodthunder"

great-baby-name
; => "Rosanthony"
```

在这个例子中，你首先在全局作用域中将 "great-baby-name "与 "Rosanthony "绑定。接下来，你用`let`引入一个新的作用域。在这个作用域中，你将`great-baby-name`绑定到`"Bloodthunder"`。一旦 Clojure 完成了对`let`表达式的求值，你就回到了全局范围，`great-baby-name`再次被求值为`"Rosanthony"`。

Clojure 让你用递归来解决这个明显的限制。下面的例子显示了解决递归问题的一般方法。

```clojure
(defn sum
➊   ([vals] (sum vals 0)) 
  ([vals accumulating-total]
➋      (if (empty? vals)  
       accumulating-total
       (sum (rest vals) (+ (first vals) accumulating-total)))))
```

这个函数需要两个参数，一个要处理的集合（`vals`）和一个累加器（`accumulating-total`），它使用了 arity 重载（在第三章有介绍），在➊为`accumulating-total`提供一个默认值`0`。

像所有的递归解决方案一样，这个函数根据一个基本条件检查它所处理的参数。在这种情况下，我们检查`vals`在➋是否为空。如果是，我们知道我们已经处理了集合中的所有元素，所以我们返回\`累计-总数'。

如果`vals`不是空的，意味着我们还在处理这个序列，所以我们递归调用`sum`，给它传递两个参数：用`(其余的vals)`表示 vals 的\*尾部，用`(+(第一个vals)累加总数)`表示`vals`的第一个元素与累加总数之和。通过这种方式，我们建立了`累积总数`，同时减少`vals`，直到它达到空集合的基本情况。

下面是递归函数调用的情况，如果我们把它每次递归的情况分开，就会是这样的。

```clojure
(sum [39 5 1]) ; single-arity body calls two-arity body
(sum [39 5 1] 0)
(sum [5 1] 39)
(sum [1] 44)
(sum [] 45) ; base case is reached, so return accumulating-total
; => 45
```

对`sum`的每次递归调用都会创建一个新的作用域，其中`vals`和`accumulating-total`被绑定到不同的值上，所有这些都不需要改变最初传递给函数的值或执行任何内部变异。正如你所看到的，你可以在没有突变的情况下顺利完成。

请注意，出于性能的考虑，在进行递归时，你一般应该使用`recur`。原因是 Clojure 不提供尾部调用的优化，这个话题我不会再提了！（请查看这个网址）。(查看这个网址以了解更多信息。[_http://en.wikipedia.org/wiki/Tail\_call_](http://en.wikipedia.org/wiki/Tail\_call)）。所以你可以用`recur`来做这个。

```clojure
(defn sum
  ([vals]
     (sum vals 0))
  ([vals accumulating-total]
     (if (empty? vals)
       accumulating-total
       (recur (rest vals) (+ (first vals) accumulating-total)))))
```

如果你在一个小的集合上进行递归操作，使用`recur`并不重要，但如果你的集合包含数千或数百万个值，你肯定需要使用`recur`，这样你就不会因为堆栈溢出而使程序爆炸。

最后一件事! 你可能会说，"等一下，如果我最终创造了成千上万的中间值怎么办？这不会因为垃圾收集或其他原因导致程序崩溃吗？"

非常好的问题，鹰眼的读者! 答案是否定的。原因是，在幕后，Clojure 的不可变数据结构是使用_结构共享_实现的，这完全超出了本书的范围。这有点像 Git! 如果你想了解更多，请阅读这篇伟大的文章。[_http://hypirion.com/musings/understanding-persistent-vector-pt-1_](http://hypirion.com/musings/understanding-persistent-vector-pt-1)。

### 函数组合而不是属性突变

你可能习惯于使用突变的另一种方式是建立起一个对象的最终状态。在下面的 Ruby 例子中，`GlamourShotCaption`对象使用突变来清理输入，删除尾部的空格并将\`"lol "大写。

```clojure
class GlamourShotCaption
  attr_reader :text
  def initialize(text)
    @text = text
    clean!
  end

  private
  def clean!
    text.trim!
    text.gsub!(/lol/, "LOL")
  end
end

best = GlamourShotCaption.new("My boa constrictor is so sassy lol!  ")
best.text
; => "My boa constrictor is so sassy LOL!"
```

在这段代码中，`GlamourShotCaption`类封装了如何清理魅力镜头标题的知识。在创建`GlamourShotCaption`对象时，你将文本分配给一个实例变量，并逐步改变它。

清单 5-1 显示了你如何在 Clojure 中做到这一点。

```clojure
(require '[clojure.string :as s])
(defn clean
  [text]
  (s/replace (s/trim text) #"lol" "LOL"))

(clean "My boa constrictor is so sassy lol!  ")
; => "My boa constrictor is so sassy LOL!"
```

1. 5-1. 使用函数组合来修改一个迷人的镜头标题

![img](https://www.braveclojure.com/assets/images/cftbat/functional-programming/glamour-boa.png)

在第一行，我们使用`require`来访问字符串函数库（我将在第六章讨论这个函数和相关概念）。除此之外，这段代码很简单。不需要变异。`clean`函数的工作方式是将一个不可变的值`text`传递给一个纯函数`s/trim`，该函数返回一个不可变的值（`"我的蟒蛇真时髦 lol!"`；字符串末尾的空格已经被修剪）。然后，该值被传递给纯函数`s/replace`，该函数返回另一个不可变的值（"我的蟒蛇是如此的时髦 LOL！"）。

像这样组合函数--使一个函数的返回值作为参数传递给另一个函数--被称为_函数组合_。事实上，这与之前使用递归的例子并没有什么不同，因为递归不断地将一个函数的结果传递给另一个函数；它只是碰巧是同一个函数。一般来说，函数式编程鼓励你通过组合更简单的函数来建立更复杂的函数。

这种比较也开始揭示了面向对象编程（OOP）的一些限制。在 OOP 中，类的主要目的之一是防止对私有数据进行不必要的修改--这在不可变的数据结构中是没有必要的。你还必须将方法与类紧密结合，从而限制了方法的可重用性。在 Ruby 的例子中，你必须做额外的工作来重复使用`clean!`方法。在 Clojure 中，\`clean'可以对任何字符串起作用。通过 a）将函数和数据解耦，以及 b）根据一组小的抽象进行编程，你最终会得到更多可重用的、可组合的代码。你获得了力量，却没有损失。

除了直接的实际问题之外，你写面向对象的代码和函数式代码的方式之间的差异指向了两种思维方式之间更深层次的差异。编程是为了你自己邪恶的目的而操纵数据（就像你可以说它是_关于_任何东西）。在 OOP 中，你把数据看成是可以体现在一个对象中的东西，你戳戳点点，直到它看起来合适。在这个过程中，你的原始数据会永远丢失，除非你非常小心地保存它。相比之下，在函数式编程中，你认为数据是不变的，你从现有的数据中导出新的数据。在这个过程中，原始数据仍然安全无恙。在前面的 Clojure 例子中，原始标题不会被修改。它是安全的，就像当你把数字加在一起时是安全的一样；当你把 3 加进去时，你不会以某种方式把 4 变成 7。

一旦你有信心使用不可变的数据结构来完成工作，你会感到更加自信，因为你不必担心有什么肮脏的代码会在你宝贵的、可变的变量上沾上油腻的爪子。这将是很好的!

## 用纯函数做的酷事

你可以从现有的函数中派生出新的函数，就像你从现有的数据中派生出新的数据一样。你已经看到了一个函数，`partial`，它可以创建新的函数。本节将向你介绍另外两个函数, `comp`和`memoize`, 它们依赖于引用透明性, 不变性, 或两者兼有.

### comp

像我们在上一节中所做的那样，对纯函数进行组合总是安全的，因为你只需要担心它们的输入/输出关系。组合函数是如此的普遍，以至于 Clojure 提供了一个函数`comp`，用于从任意数量的函数组合中创建一个新的函数。下面是一个简单的例子。

```clojure
((comp inc *) 2 3)
; => 7
```

在这里，你通过组合`inc`和`*`函数来创建一个匿名函数。然后，你立即将这个函数应用于参数`2`和`3`。该函数将数字 2 和 3 相乘，然后将结果递增。使用数学符号，你会说，一般来说，对函数_f_1, _f_2, ... _f_n，创建一个新的函数_g_，使得_g_(_x_1, _x_2, ... _x_n)等于_f_1( _f_2( _f_n(_x_1, _x_2, ... _x_n))。这里需要注意的一个细节是，第一个应用的函数--这里显示的代码中的`*`--可以接受任何数量的参数，而其余的函数必须只能接受一个参数。

下面是一个例子，说明如何使用`comp`来检索角色扮演游戏中的角色属性。

```clojure
(def character
  {:name "Smooches McCutes"
   :attributes {:intelligence 10
                :strength 4
                :dexterity 5}})
(def c-int (comp :intelligence :attributes))
(def c-str (comp :strength :attributes))
(def c-dex (comp :dexterity :attributes))

(c-int character)
; => 10

(c-str character)
; => 4

(c-dex character)
; => 5
```

在这个例子中，你创建了三个函数来帮助你查询一个角色的属性。你可以不使用`comp`，而是为每个属性写成这样的东西。

```clojure
(fn [c] (:strength (:attributes c)))
```

但`comp`更优雅，因为它用更少的代码来表达更多的意思。当你看到 `comp`时，你立即知道所产生的函数的目的是以一种众所周知的方式组合现有的函数。

如果你想组合的一个函数需要接受一个以上的参数，你会怎么做？你把它包在一个匿名函数中。请看下面这个片段，它根据你的角色的智力属性来计算她的法术槽的数量。

```clojure
(defn spell-slots
  [char］
  (int (inc (/ (c-int char) 2))))

(spell-slots character)
; => 6
```

首先，你用智力除以 2，然后加 1，然后用`int`函数来取整。下面是你如何用`comp`做同样的事情。

```clojure
(def spell-slots-comp (comp int inc #(/ % 2) c-int))
```

要除以 2，你所要做的就是把除法包在一个匿名函数中。

Clojure 的`comp`函数可以组成任何数量的函数。为了了解它是如何做到这一点的，这里有一个实现，它只组合了两个函数。

```clojure
(defn two-comp
  [f g].
  (fn [& args]
    (f (apply g args))))
```

我鼓励你求值这段代码，并使用`two-comp`来组合两个函数! 另外，尝试重新实现 Clojure 的`comp`函数，这样你就可以组成任何数量的函数。

### 记忆化

你可以用纯函数做的另一件很酷的事情是对它们进行备忘，这样 Clojure 就会记住某个特定函数调用的结果。你可以这样做，因为正如你前面所学到的，纯函数在指代上是透明的。例如，`+`是指代透明的。你可以把

```clojure
(+ 3 (+ 5 8))
```

替换为

```clojure
(+ 3 13)
```

或

```clojure
16
```

而程序会有相同的行为。

Memoization 让你通过存储传递给函数的参数和函数的返回值来利用引用的透明度。这样一来，以后用相同的参数调用该函数就可以立即返回结果。这对于需要大量时间运行的函数特别有用。例如，在这个没有备忘录的函数中，结果在一秒钟后被返回。

```clojure
(defn sleepy-identity
  "Returns the given value after 1 second"
  [x]
  (Thread/sleep 1000)
  x)
(sleepy-identity "Mr. Fantastico")
; => "Mr. Fantastico" after 1 second

(sleepy-identity "Mr. Fantastico")
; => "Mr. Fantastico" after 1 second
```

然而，如果你用`memoize`创建一个新的、记忆化的`sleepy-identity`版本，只有第一次调用会等待一秒钟；随后的每个函数调用会立即返回。

```clojure
(def memo-sleepy-identity (memoize sleepy-identity))
(memo-sleepy-identity "Mr. Fantastico")
; => "Mr. Fantastico" after 1 second

(memo-sleepy-identity "Mr. Fantastico")
; => "Mr. Fantastico" immediately
```

很酷啊! 从这里开始，`memo-sleepy-identity`在调用\`"Mr.Fantastico "时将不会产生最初的一秒钟的费用。这个实现对于那些计算密集型的函数或者提出网络请求的函数很有用。

## Peg Thing

是时候了! 是时候用你到目前为止所学到的一切来构建 Peg Thing 的终端实现了：不可变的数据结构、懒惰序列、纯函数、递归--一切！这将有助于你理解如何将这些概念和技术结合起来解决更大的问题。这样做将帮助你理解如何结合这些概念和技术来解决更大的问题。最重要的是，你将学会如何对玩家的每一次移动所产生的变化进行建模，而不必像在 OOP 中那样对对象进行变异。

为了构建游戏，你将首先学习游戏的机制以及如何启动和播放程序。然后，我将解释代码的组织。最后，我将对每个函数进行讲解。

你可以在\*[https://www.nostarch.com/clojure/](https://www.nostarch.com/clojure/)\*找到 Peg Thing 的完整代码库。

## 播放

如前所述，Peg Thing 是基于从古至今流传下来的秘密思维磨练工具，现在由 Cracker Barrel 发行。

如果你不熟悉这个游戏，以下是游戏的机制。你开始时有一个三角形的棋盘，上面布满了钉子的孔，其中一个孔缺少一个钉子，如图 5-1 所示。

![img](https://www.braveclojure.com/assets/images/cftbat/functional-programming/peg-thing-starting.png)

图 5-1：Peg Thing 的初始设置

游戏的目的是尽可能多地移除钉子。你通过\*跳过钉子来实现这一目标。在图 5-2 中，图钉 A 跳过图钉 B 进入空洞，你就把图钉 B 从棋盘上移走。

[img](https://www.braveclojure.com/assets/images/cftbat/functional-programming/peg-thing-jumping.png)

图 5-2：跳过一个钉子，把它从棋盘上移走。

要启动 Peg Thing，请下载代码，然后在_pegthing_目录下的终端运行`lein run`。出现一个提示，看起来像这样。

```clojure
Get ready to play Peg Thing!
How many rows? [5]
```

现在你可以输入棋盘的行数，使用`5`作为默认值。如果你想要五行，就按回车键（否则，输入一个数字并按回车键）。然后你会看到这个。

```clojure
Here's your board:
       a0
      b0 c0
    d0 e0 f0
   g0 h0 i0 j0
 k0 l0 m0 n0 o0
Remove which peg? [e]
```

每个字母代表棋盘上的一个位置。数字`0`（应该是蓝色的，但如果不是，也没什么大不了的）表示一个位置被填满。在游戏开始之前，必须有一个钉子是空的，所以提示要求你输入第一个要移除的钉子的位置。默认是中间的钉子，\`e'，但你可以选择一个不同的位置。移走棋子后，你会看到这个。

```clojure
Here's your board:
       a0
      b0 c0
    d0 e- f0
   g0 h0 i0 j0
 k0 l0 m0 n0 o0
Move from where to where? Enter two letters:
```

注意，`e`位置现在有一个破折号，`-`（应该是红色的，但如果不是，也没什么大不了的）。这个破折号表示这个位置是空的。要移动，你要输入你想\*拿起的棋子的位置，然后是你想把它放在的空位置的位置。例如，如果你输入\`le'，你会得到这样的结果。

```clojure
Here's your board:
       a0
      b0 c0
    d0 e0 f0
   g0 h- i0 j0
 k0 l- m0 n0 o0
Move from where to where? Enter two letters:
```

你把棋子从`l`移到`e`，跳过`h`，根据图 5-2 所示的规则移走它的棋子。游戏继续提示你走棋，直到没有棋子可用为止，这时它就会提示你再次下棋。

### 代码组织

该程序必须处理四个主要任务，源代码也相应地组织起来，每个任务的函数都归为一组。

1. 创建一个新的棋盘
2. 返回一个带有棋手行动结果的棋盘
3. 用文字表示棋盘
4. 处理用户互动

关于组织结构还有两点。首先，代码有一个基本的_架构_，或概念性的组织，由两层组成。顶层由处理用户交互的函数组成。这些函数产生了程序的所有副作用，打印出棋盘并为玩家的互动提供提示。这一层的函数使用底层的函数来创建一个新的棋盘，进行移动，并创建一个文本表述，但底层的函数完全不使用顶层的函数。即使是这么小的程序，一个小小的架构也有助于使代码更容易管理。

第二，我尽可能地将任务分解成小的函数，以便每个函数都做一个微小的、可理解的任务。其中一些函数只被另外一个函数使用。我发现这很有帮助，因为它可以让我为每个微小的子任务命名，使我能够更好地表达代码的意图。

但在所有的架构之前，还有这个。

```clojure
(ns pegthing.core
  (require [clojure.set :as set])
  (:gen-class))

(declare successful-move prompt-move game-over query-rows)
```

我将在第六章详细解释这里的函数。如果你好奇这是怎么回事，简短的解释是：`(`require `[`clojure`.set :as set])`允许你轻松使用`clojure.set`命名空间的函数，`(:gen-class)`允许你从命令行运行程序，`(declaration successful-move prompt-move game-over query-rows)`允许函数在被定义之前引用这些名称。如果这还不太明白，相信我很快就会解释。

### 创建棋盘

你用来表示棋盘的数据结构应该使你能够很容易地打印棋盘，检查棋手是否下了一步有效的棋，实际执行一步棋，以及检查游戏是否结束。你可以用很多方式来构造棋盘，以实现这些任务。在这种情况下，你将用一个 Map 来表示棋盘，Map 上有对应于每个棋盘位置的数字键和包含该位置连接信息的值。该 Map 还将包含一个`:rows`键，存储行的总数。图 5-3 显示了一个有每个位置编号的棋盘。

![img](https://www.braveclojure.com/assets/images/cftbat/functional-programming/peg-thing-data-structure.png)

图 5-3：有编号的钉子板

下面是为表示它而建立的数据结构。

```clojure
{1  {:pegged true, :connections {6 3, 4 2}},
 2  {:pegged true, :connections {9 5, 7 4}},
 3  {:pegged true, :connections {10 6, 8 5}},
 4  {:pegged true, :connections {13 8, 11 7, 6 5, 1 2}},
 5  {:pegged true, :connections {14 9, 12 8}},
 6  {:pegged true, :connections {15 10, 13 9, 4 5, 1 3}},
 7  {:pegged true, :connections {9 8, 2 4}},
 8  {:pegged true, :connections {10 9, 3 5}},
 9  {:pegged true, :connections {7 8, 2 5}},
 10 {:pegged true, :connections {8 9, 3 6}},
 11 {:pegged true, :connections {13 12, 4 7}},
 12 {:pegged true, :connections {14 13, 5 8}},
 13 {:pegged true, :connections {15 14, 11 12, 6 9, 4 8}},
 14 {:pegged true, :connections {12 13, 5 9}},
 15 {:pegged true, :connections {13 14, 6 10}},
 :rows 5}
```

你可能会想，为什么在你玩游戏的时候，每个位置都用字母表示，而在这里，位置却用数字表示。使用数字作为内部表示，可以让你在验证和下棋时利用棋盘布局的一些数学特性。另一方面，字母则更适合于显示，因为它们只占用一个字符空间。一些转换函数在[第 120 页的 "渲染和打印棋盘"](https://www.braveclojure.com/functional-programming/#Anchor-19)中有所介绍。

在数据结构中，每个位置都与一个 Map 相关联，其内容是这样的。

```clojure
{:pegged true, :connection {6 3, 4 2}}.
```

pegged "的含义很清楚；它表示该位置是否有一个钉子。\`:connections'就比较隐蔽了。它是一个 Map，每个键标识一个_合法的目的地，每个值代表_将被_跳过的位置。例如，位置 1 的棋子可以_跳到\*位置 6，_越过_位置 3。这可能看起来很落后，但当你看到棋步验证是如何实现的时候，你就会明白其中的道理。

现在你已经看到了代表棋盘的最终 Map 应该是什么样子的，我们可以开始探索在程序中实际建立这个 Map 的函数了。你不会简单地开始随意地分配可变状态来表示每个位置以及它是否被钉住。相反，你将使用嵌套的递归函数调用来逐个建立最终的棋盘位置。这类似于你之前创建魅力镜头标题的方式，通过将参数传递给一连串的函数，从输入中获得新的数据，从而得到最终结果。

本节代码中的前几个表达式是关于三角形数的。三角形数是由前_n_个自然数相加产生的。第一个三角数是 1，第二个是 3（1+2），第三个是 6（1+2+3），以此类推。这些数字很好地与棋盘上每一行末尾的位置数字相一致，这将成为一个非常有用的属性。首先，你定义了函数`tri*`，它可以创建一个三角形数字的懒散序列。

```clojure
(defn tri*
  "Generates 惰性序列uence of triangular numbers"
  ([] (tri* 0 1))
  ([sum n]
     (let [new-sum (+ sum n)]
       (cons new-sum (lazy-seq (tri* new-sum (inc n)))))))
```

快速回顾一下工作原理，调用没有参数的`tri*`会递归调用`(tri* 0 1)`。这将返回一个懒惰列表，其元素是 "new-sum"，在本例中它的值为 "1"。懒惰列表包括一个配方，用于在请求时生成列表的下一个元素，如第四章所述。

下一个表达式调用`tri*`，实际上是创建懒惰序列并将其绑定到`tri`。

```clojure
(def tri (tri*))
```

你可以验证它是否真的有效。

```clojure
(take 5 tri)
; => (1 3 6 10 15)
```

接下来的几个函数对三角形数的序列进行操作。`triangular?`找出它的参数是否在`tri`懒惰序列中。它通过使用`take-while`创建一个三角形数列，其最后一个元素是一个小于或等于参数的三角形数。然后它将最后一个元素与参数进行比较。

```clojure
(defn triangular?
 "Is the number triangular? e.g. 1, 3, 6, 10, 15, etc"
 [n]
 (= n (last (take-while #(>= n %) tri))))
(triangular? 5) 
; => false

(triangular? 6) 
; => true
```

接下来是`row-tri`，它接收一个行号，并给出该行末尾的三角形数字。

```clojure
(defn row-tri
  "The triangular number at the end of row n"
  [n]
  (last (take n tri)))
(row-tri 1) 
; => 1

(row-tri 2) 
; => 3

(row-tri 3) 
; => 6
```

最后，还有`row-num`，它接收一个棋盘位置，并返回它所属的行。

```clojure
(defn row-num
  "Returns row number the position belongs to: pos 1 in row 1,
  positions 2 and 3 in row 2, etc"
  [pos]
  (inc (count (take-while #(> pos %) tri))))
(row-num 1) 
; => 1
(row-num 5) 
; => 3
```

之后是`connect`，它被用来在两个位置之间实际形成相互连接。

```clojure
(defn connect
  "Form a mutual connection between two positions"
  [board max-pos pos neighbor destination]
  (if (<= destination max-pos)
    (reduce (fn [new-board [p1 p2]]
              (assoc-in new-board [p1 :connections p2] neighbor))
            board
            [[pos destination] [destination pos]])
    board))

(connect {} 15 1 2 4)
; => {1 {:connections {4 2}}
      4 {:connections {1 2}}}
```

`connect`做的第一件事是检查目的地是否真的是棋盘上的一个位置，确认它是否小于棋盘的最大位置。例如，如果你有五行，最大位置是 15。然而，当游戏棋盘被创建时，`connect`函数将被调用，参数为`(connect {} 15 7 16 22)`。`connect`开头的`if`语句确保你的程序不允许这种离谱的连接，当你要求它做一些愚蠢的事情时，它只是返回未修改的棋盘。

接下来，`connect`通过`reduce`使用递归，逐步建立起棋盘的最终状态。在这个例子中，你正在减少嵌套 Vector`[[1 4] [4 1]]。这就是允许你返回一个更新的棋盘，`pos'和\`destination'（1 和 4）在它们的连接中都指向对方。

传给`reduce`的匿名函数使用了一个你以前没有见过的函数`assoc-in`。函数`get-in`可以让你在嵌套的 Map 中查找值，而`assoc-in`可以让你在指定的嵌套中返回一个带有给定值的新 Map。下面是几个例子。

```clojure
(assoc-in {} [:cookie :monster :vocals] "Finntroll")
; => {:cookie {:monster {:vocals "Finntroll"}}}

(get-in {:cookie {:monster {:vocals "Finntroll"}}} [:cookie :monster])
; => {:vocals "Finntroll"}

(assoc-in {} [1 :connections 4] 2)
; => {1 {:connections {4 2}}}
```

在这些例子中，你可以看到，新的、嵌套的 Map 被创建，以适应所有提供的键。

现在我们有了一个连接两个位置的方法，但程序首先应该如何选择两个位置来连接呢？这由`connect-right`、`connect-down-left`和`connect-down-right`来处理。

```clojure
(defn connect-right
  [board max-pos pos]
  (let [neighbor (inc pos)
        destination (inc neighbor)]
    (if-not (or (triangular? neighbor) (triangular? pos))
      (connect board max-pos pos neighbor destination)
      board)))

(defn connect-down-left
  [board max-pos pos]
  (let [row (row-num pos)
        neighbor (+ row pos)
        destination (+ 1 row neighbor)]
    (connect board max-pos pos neighbor destination)))

(defn connect-down-right
  [board max-pos pos]
  (let [row (row-num pos)
        neighbor (+ 1 row pos)
        destination (+ 2 row neighbor)]
    (connect board max-pos pos neighbor destination)))
```

这些函数分别取了棋盘的最大位置和一个棋盘的位置，并使用一个小的三角形数学来计算出哪些数字要送入`connect`。例如，`connect-down-left`将试图连接位置 1 和位置 4。如果你想知道为什么没有定义`connect-left`、`connect-up-left`和`connect-up-right`函数，原因是现有的函数实际上涵盖了这些情况。`connect`返回一个建立了相互连接的棋盘；当 4_向右_连接到 6 时，6_向左_连接到 4。 下面是几个例子。

```clojure
(connect-down-left {} 15 1)
; => {1 {:connections {4 2}
      4 {:connections {1 2}}}}

(connect-down-right {} 15 3)
; => {3  {:connections {10 6}}
      10 {:connections {3 6}}}
```

在第一个例子中，`connect`-down-left`接收一个最大位置为15的空棋盘，并返回一个新的棋盘，该棋盘上有1和它下面及左边的位置之间的相互连接。`connect`-down-right`做了类似的事情，返回一个由 3 和它下面及右边的位置之间的相互连接组成的棋盘。

下一个函数，\`add-pos'，很有趣，因为它实际上是在一个_函数_的 Vector 上进行还原，依次应用每个函数来建立结果的棋盘。但它首先更新棋盘，以表示一个棋子在给定的位置上。

```clojure
(defn add-pos
  "Pegs the position and performs connections"
  [board max-pos pos]
  (let [pegged-board (assoc-in board [pos :pegged] true)]
    (reduce (fn [new-board connection-creation-fn]
              (connection-creation-fn new-board max-pos pos))
            pegged-board
            [connect-right connect-down-left connect-down-right])))

(add-pos {} 15 1)
{1 {:connections {6 3, 4 2}, :pegged true}
 4 {:connections {1 2}}
 6 {:connections {1 3}}}
```

就像这个函数首先在`pegged-board`绑定中说："在棋盘的 X 位置添加一个钉子。" 然后，在 "reduce "中，它说："在 X 的位置上采取新钉子的棋盘，并尝试将 X 的位置连接到一个合法的、向右的位置。取该操作产生的棋盘，并尝试将位置 X 连接到一个合法的、向左下方的位置。最后，取该\*操作产生的棋盘，并尝试将位置 X 连接到合法的右下位置。返回所得到的棋盘"。

像这样对函数进行还原是组成函数的另一种方式。为了说明这一点，下面是清单 5-1（[第 103 页](https://www.braveclojure.com/functional-programming/#Anchor-20)）中定义`clean`函数的另一种方式。

```clojure
(defn clean
  [text]
  (reduce (fn [string string-fn] (string-fn string))
          text
          [s/trim #(s/replace % #"lol" "LOL")]))
```

这个对`clean`的重新定义减少了一个函数 Vector，将第一个函数`s/trim`应用于初始字符串，然后将下一个函数，匿名函数`#(s/replace % #"lol" "LOL")`，应用于结果。

对一个函数集合进行缩减并不是一个你经常使用的技术，但它偶尔会很有用，而且它显示了函数式编程的多函数性。

最后一个创建棋盘的函数是`new-board`。

```clojure
(defn new-board
  "Creates a new board with the given number of rows"
  [rows]
  (let [initial-board {:rows rows}
        max-pos (row-tri rows)]
    (reduce (fn [board pos] (add-pos board max-pos pos))
            initial-board
            (range 1 (inc max-pos)))))
```

这段代码首先创建了初始的、空的棋盘，并得到了最大的位置。假设你使用的是 5 行，最大位置将是 15。接下来，该函数使用`(range 1 (inc max-pos))`得到一个从 1 到 15 的数字列表，也就是棋盘的位置。最后，它对位置列表进行还原。缩减的每一次迭代都会调用`(add-pos board max-pos pos)`，正如你前面所看到的，它获取一个现有的棋盘，并返回一个新的棋盘，并添加位置。

### 移动图钉

下一节代码将验证并执行钉子的移动。许多函数（`pegged?`, `remove-peg`, `place-peg`, `move-peg`）都是简单的、不言自明的单行代码。

```clojure
(defn pegged?
  "Does the position have a peg in it?"
  [board pos]
  (get-in board [pos :pegged]))

(defn remove-peg
  "Take the peg at given position out of the board"
  [board pos]
  (assoc-in board [pos :pegged] false))

(defn place-peg
  "Put a peg in the board at given position"
  [board pos]
  (assoc-in board [pos :pegged] true))

(defn move-peg
  "Take peg out of p1 and place it in p2"
  [board p1 p2]
  (place-peg (remove-peg board p1) p2))
```

让我们花点时间来欣赏一下这段代码是多么的整洁。在面向对象的程序中，你通常会在这里进行突变；毕竟，你还能如何改变棋盘？然而，这些都是纯函数，而且它们很好地完成了工作。我还喜欢你不需要类的开销来使用这些小家伙。这样的编程感觉更轻松。

接下来是`valid-moves`。

```clojure
(defn valid-moves
  "Return a map of all valid moves for pos, where the key is the
  destination and the value is the jumped position"
  [board pos]
  (into {}
        (filter (fn [[destination jumped]]
                  (and (not (pegged? board destination))
                       (pegged? board jumped)))
                (get-in board [pos :connections]))))
```

这段代码浏览了给定位置的每个连接，并测试目的地位置是否为空，跳转的位置是否有钉子。为了看到这个动作，你可以创建一个 4 号位置为空的棋盘。

```clojure
(def my-board (assoc-in (new-board 5) [4 :pegged] false))
```

图 5-4 显示了这个棋盘的样子。

![img](https://www.braveclojure.com/assets/images/cftbat/functional-programming/peg-thing-valid-moves.png)

图 5-4：4 号位置为空的钉子板

考虑到这个棋盘，1、6、11 和 13 位有有效的棋步，但其他位置都没有。

```clojure
(valid-moves my-board 1)  ; => {4 2}
(valid-moves my-board 6)  ; => {4 5}
(valid-moves my-board 11) ; => {4 7}
(valid-moves my-board 5)  ; => {}
(valid-moves my-board 8)  ; => {}
```

你可能想知道为什么`valid-moves`会返回一个 Map 而不是一个集合或 Vector。原因是，返回 Map 允许你轻松地查找目标位置，以检查特定的棋步是否有效，这就是`valid-move?`（下一个函数）的作用。

```clojure
(defn valid-move?
  "Return jumped position if the move from p1 to p2 is valid, nil
  otherwise"
  [board p1 p2]
  (get (valid-moves board p1) p2))
  
(valid-move? my-board 8 4) ; => nil
(valid-move? my-board 1 4) ; => 2
```

注意，`valid-move?`从 Map 上查找目标位置，然后返回将被跳过的钉子的位置。这是让`valid-moves`返回 Map 的另一个很好的好处，因为从 Map 中获取的跳跃位置正是我们想要传递给下一个函数`make-move`的东西。当你花时间构建一个丰富的数据结构时，执行有用的操作会更容易。

```clojure
(defn make-move
  "Move peg from p1 to p2, removing jumped peg"
  [board p1 p2]
  (if-let [jumped (valid-move? board p1 p2)]
    (move-peg (remove-peg board jumped) p1 p2)))
```

`if-let`是一种很好的方式，表示 "如果一个表达式求值为一个真实的值，那么就把这个值绑定到一个名字上，就像我在`let`表达式中一样。否则，如果我提供了一个 "else "子句，就执行该 "else "子句；如果我没有提供 "else "子句，就返回 "nil"。在这种情况下，测试表达式是`(valid-move? board p1 p2)`，如果结果是真实的，你要把结果分配给`jumped`这个名字。这将用于调用`move-peg`，它将返回一个新棋盘。你没有提供 "else "子句，所以如果移动无效，整个表达式的返回值为 "nil"。

最后，函数\`can-move? 是用来确定游戏是否结束的，方法是找到第一个有棋步的钉子位置。

```clojure
(defn can-move?
  "Do any of the pegged positions have valid moves?"
  [board]
  (some (comp not-empty (partial valid-moves board))
        (map first (filter #(get (second %) :pegged) board))))
```

这个函数名称末尾的问号表明它是一个_谓词函数_，这个函数是为了在布尔表达式中使用。_谓词_取自谓词逻辑，它关注的是确定一个语句是真还是假。(你已经看到了一些内置的谓词函数，如`empty?`和`every?`。)

`can-move?`的工作原理是通过`(map first (filter #(get (second %) :pegged) board))`获得一个所有挂点的序列。你可以将其进一步分解为`filter'和`map'的函数调用：因为`filter'是一个seq函数，它将`board'，一个 Map，转换成一个两元素 Vector 的 seq（也称为_tuples_），看起来像这样。

```clojure
([1 {:connection {6 3, 4 2}, :pegged true})
 [2 {:connections {9 5, 7 4}, :pegged true}])
```

元组的第一个元素是一个位置号，第二个元素是该位置的信息。`filter`然后将匿名函数`#(get (second %) :pegged)`应用于这些元组中的每一个，过滤掉那些位置信息表明该位置目前没有挂点的元组。最后，结果被传递给`map`，它在每个元组上调用`first`，只从元组中抓取位置号。

当你得到一连串的钉子位置号码后，你对每个位置调用一个谓词函数，找到第一个返回真值的位置。这个谓词函数是用`(`comp not-empty (`partial valid-moves board))`创建的。我们的想法是首先返回一个位置的所有有效棋步的 Map，然后测试该 Map 是否为空。

首先，表达式`(partial valid-moves board)`从`valid-moves`派生出一个匿名函数，第一个参数`board`用`partial`填入（因为你每次调用`valid-moves`时使用的是同一个棋盘）。这个新函数可以接受一个位置，并返回它在当前棋盘上的所有有效棋步的 Map。

第二，你用`comp`将这个函数与`not-empty`组成。这个函数是自述的；如果给定的集合是空的，它返回 "true"，否则返回 "false"。

这段代码最有趣的地方在于，你在使用一个函数链来推导一个新的函数，类似于你使用函数链来推导新的数据。在第三章中，你了解到 Clojure 将函数视为数据，因为函数可以接收函数作为参数并返回。希望这能说明为什么这个函数是有趣和有用的。

### 渲染和打印棋盘

在棋盘表示和打印部分的前几个表达式只是定义常数。

```clojure
(def alpha-start 97)
(def alpha-end 123)
(def letters (map (comp str char) (range alpha-start alpha-end) ))
(def pos-chars 3)
```

绑定 "alpha-start "和 "alpha-end "为字母_a_到_z_设定了数值的开始和结束。我们用这些来建立一个 "字母 "的序列。`char`，当应用于一个整数时，返回该整数对应的字符，`str`将`char`变成一个字符串。`pos-chars`被函数`row-padding`使用，以确定在每一行的开头增加多少间距。接下来的几个定义，`ansi-styles`，`ansi`，和`colorize`向终端输出彩色文本。

函数`render-pos`, `row-positions`, `row-padding`, 和`render-row`创建字符串来表示棋盘。

```clojure
(defn render-pos
  [board pos]
  (str (nth letters (dec pos))
       (if (get-in board [pos :pegged])
         (colorize "0" :blue)
         (colorize "-" :red))))

(defn row-positions
  "Return all positions in the given row"
  [row-num]
  (range (inc (or (row-tri (dec row-num)) 0))
         (inc (row-tri row-num))))

(defn row-padding
  "String of spaces to add to the beginning of a row to center it"
  [row-num rows]
  (let [pad-length (/ (* (- rows row-num) pos-chars) 2)]
    (apply str (take pad-length (repeat " ")))))

(defn render-row
  [board row-num]
  (str (row-padding row-num (:rows board))
       (clojure.string/join " " (map (partial render-pos board) 
                                     (row-positions row-num)))))
```

如果你从下往上看，你可以看到`render-row`调用它上面的每个函数来返回给定行的字符串表示。注意表达式`(map (partial render-pos board) (row-positions row-num))`。这展示了部分函数的一个很好的用例，即多次应用同一函数，并填入一个或多个参数，就像前面展示的`can-move?`函数一样。

还请注意，\`render-pos'使用字母而不是数字来标识每个位置。这在棋盘显示时节省了一点空间，因为它允许每个位置只有一个字符来表示一个五行棋盘。

最后，`print-board`只是用`doseq`遍历每一行的编号，打印该行的字符串表示。

```clojure
(defn print-board
  [board]
  (doseq [row-num (range 1 (inc (:rows board)))]
    (println (render-row board row-num))))
```

当你想对一个集合中的元素进行副作用的操作（比如打印到终端）时，你可以使用`doseq`。紧跟在名字`doseq`后面的 Vector 描述了如何将一个集合中的所有元素逐一绑定到一个名字上，以便你可以对它们进行操作。在这个例子中，你要把数字 1 到 5（假设有五行）分配给`row-num`这个名字，这样你就可以打印每一行。

虽然打印棋盘在技术上属于_交互_，但我想在这里用渲染函数来展示它。当我第一次开始写这个游戏时，`print-board`函数也生成了棋盘的字符串表示。然而，现在`print-board`将所有的渲染工作推迟到纯函数，这使得代码更容易理解，并减少了我们不纯函数的表面积。

### 玩家互动

下一个函数集合处理玩家互动。首先，有`letter->pos`，它将字母（这是玩家显示和识别位置的方式）转换为相应的位置编号。

```clojure
(defn letter->pos
  "Converts a letter string to the corresponding position number"
  [letter]
  (inc (- (int (first letter)) alpha-start)))
```

接下来，辅助函数`get-input`允许你读取和清理玩家的输入。你也可以提供一个默认值，如果玩家没有输入任何东西就按下回车键，就会使用这个值。

```clojure
(defn get-input
  "Waits for user to enter text and hit enter, then cleans the input"
  ([] (get-input nil))
  ([default]
     (let [input (clojure.string/trim (read-line))]
       (if (empty? input)
         default
         (clojure.string/lower-case input)))))
```

下一个函数，`characters-as-strings`，是一个很小的辅助函数，被`prompt-move`用来接收一个字符串并返回一个字母集合，所有非字母的输入都被丢弃。

```clojure
(characters-as-strings "a b")
; => ("a" "b")

(characters-as-strings "a cb")
; => ("a" "c" "b")
```

接下来，`prompt-move`读取玩家的输入，并对其采取行动。

```clojure
(defn prompt-move
  [board]
  (println "\nHere's your board:")
  (print-board board)
  (println "Move from where to where? Enter two letters:")
  (let [input (map letter->pos (characters-as-strings (get-input)))]
    (if-let [new-board (make-move➊ board (first input) (second input))]
      (user-entered-valid-move new-board)
      (user-entered-invalid-move board))))
```

在➊，如果棋手的棋步无效，`make-move`返回`nil`，你用`user-entered-invalid-move`函数来通知她的错误。你将未经修改的棋盘传递给`user-entered-invalid-move`，这样它就可以用棋盘再次提示棋手。下面是函数的定义。

```clojure
(defn user-entered-invalid-move
  "Handles the next step after a user has entered an invalid move"
  [board]
  (println "\n!!! That was an invalid move :(\n")
  (prompt-move board))
```

然而，如果这步棋是有效的，"new-board "将被传递给 "user-entered-valid-move"，如果还有棋步要走，它将控制权交还给 "prompt-move"。

```clojure
(defn user-entered-valid-move
  "Handles the next step after a user has entered a valid move"
  [board]
  (if (can-move? board)
    (prompt-move board)
    (game-over board)))
```

在我们的棋盘创建函数中，我们看到递归是如何使用不可变的数据结构来建立一个值的。同样的事情也发生在这里，只是它涉及两个相互递归的函数和一些用户输入。没有看到可变的属性!

游戏结束后会发生什么？这就是所发生的事情。

```clojure
(defn game-over
  "Announce the game is over and prompt to play again"
  [board]
  (let [remaining-pegs (count (filter :pegged (vals board)))]
    (println "Game over! You had" remaining-pegs "pegs left:")
    (print-board board)
    (println "Play again? y/n [y]")
    (let [input (get-input "y")]
      (if (= "y" input)
        (prompt-rows)
        (do
          (println "Bye!")
          (System/exit 0))))))
```

这里发生的所有事情是，游戏告诉你你的表现，打印出最后的棋盘，并提示你再玩一次。如果你选择_y_，游戏就会调用`prompt-rows`，这就给我们带来了最后一组函数，用来开始新的游戏。

```clojure
(defn prompt-empty-peg
  [board]
  (println "Here's your board:")
  (print-board board)
  (println "Remove which peg? [e]")
  (prompt-move (remove-peg board (letter->pos (get-input "e")))))

(defn prompt-rows
  []
  (println "How many rows? [5]")
  (let [rows (Integer. (get-input 5))
        board (new-board rows)]
    (prompt-empty-peg board)))
```

你使用`prompt-rows`开始游戏，让玩家输入要包括多少行。然后你把控制权交给`prompt-empty-peg`，这样玩家就可以告诉游戏要先移除哪个钉子。从这里开始，程序提示你走棋，直到没有任何棋步。

尽管这个程序的所有副作用都是相对无害的（你所做的只是提示和打印），但像这样将它们封存在自己的函数中是函数式编程的最佳实践。一般来说，如果你能确定哪些函数是透明的、无副作用的，并将这些函数放在自己的函数中，你将从函数式编程中获得更多的好处。这些函数不可能在你的程序的不相关部分引起奇怪的错误。它们更容易在 REPL 中测试和开发，因为它们只依赖于你传递给它们的参数，而不是一些复杂的隐藏状态对象。

## 总结

纯函数在本质上是透明的，并且没有副作用，这使得它们很容易被推理。为了从 Clojure 中获得最大的收益，请尽量减少不纯函数的使用。在一个不可变的世界里，你可以使用递归而不是`for`/`while`循环，使用函数组合而不是连续的突变。纯函数允许强大的技术，如函数组合函数和备忘化。它们也是超级有趣的!

## 练习

发展函数式编程技能的最好方法之一是尝试实现现有的函数。为此，下面的大多数练习都建议你实现一个函数，但不要止步于此；通过 Clojure 小抄（[_http://clojure.org/cheatsheet/_](http://clojure.org/cheatsheet/)），可以挑选更多的函数

1. 你用`(comp :intelligence :attribute)`创建了一个函数来返回一个角色的智力。创建一个新的函数，`attr`，你可以像`(attr :intelligence)`那样调用它，做同样的事情。
2. 实现`comp`函数。
3. 实现`assoc-in`函数。提示：使用`assoc`函数并将其参数定义为`[m [k & ks] v]`。
4. 查阅并使用`update-in`函数。
5. 实现 "update-in"。
