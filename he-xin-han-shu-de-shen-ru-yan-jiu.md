# 核心函数的深入研究

如果你像我一样是焦虑的、以青少年为中心的准肥皂剧\*《吸血鬼日记》的超级粉丝，你一定记得主角埃琳娜开始质疑她苍白的、神秘的暗恋者的行为的那一集。"为什么当我的膝盖被刮伤时，他立刻消失得无影无踪？"和 "为什么当我的手指被划破时，他的脸变成了一个怪异的死亡面具？"等等。

如果你已经开始玩 Clojure 的核心函数，你可能也会问自己类似的问题。"为什么`map'会返回一个列表，而我给它的是一个向量？"和 "为什么`reduce'会把我的 map 当成一个向量列表？"等等。(不过，有了 Clojure，你至少可以免于思考作为一个 17 岁孩子的深刻的存在恐惧，直到永远）。

在这一章中，你将了解到 Clojure 的深邃、黑暗、嗜血、超自然的\*\*\*\*，我的意思是，在这一章中，你将了解到 Clojure 的_编程到抽象的基本概念以及序列和集合的抽象。你还会了解到_疯狂的序列\*。这将为你提供所需的基础，使你能够阅读你以前没有使用过的函数的文档，并理解当你试着使用它们时发生了什么。

![img](https://www.braveclojure.com/assets/images/cftbat/core-functions-in-depth/sparkly.png)

接下来，你将获得更多关于你最需要使用的函数的经验。你将学习如何用函数`map`、`reduce`、`into`、`conj`、`concat`、`some`、`filter`、`take`、`drop`、`sort`、`sort-by`和`identity`来处理列表、向量、Map 和集合。你还将学习如何用`apply`、`partial`和`complement`创建新的函数。所有这些信息将帮助你了解如何以 Clojure 的方式做事，它将为你编写自己的代码以及阅读和学习他人的项目打下坚实的基础。

最后，你将学会如何解析和查询 CSV 中的吸血鬼数据，以确定在你的家乡潜伏着哪些诺斯费拉图。

## 从编程到抽象

为了理解对抽象的编程，让我们把 Clojure 与一种没有考虑到这个原则的语言进行比较。Emacs Lisp（elisp）。在 elisp 中，你可以使用`mapcar`函数来导出一个新的列表，这与你在 Clojure 中使用`map`的方式相似。然而，如果你想在 elisp 中映射一个哈希图（类似于 Clojure 的 map 数据结构），你需要使用`maphash`函数，而在 Clojure 中你仍然可以只使用`map`。换句话说，elisp 使用两个不同的、针对数据结构的函数来实现_map_操作，而 Clojure 只使用一个。你也可以在 Clojure 中对 map 调用`reduce`，而 elisp 并没有提供一个函数来减少散列 map。

原因是 Clojure 在\*序列抽象方面定义了`map`和`reduce`函数，而不是在具体的数据结构方面。只要数据结构响应核心序列操作（函数`first'、`rest'和`cons'，我们稍后会仔细研究），它就能与`map'、\`reduce'以及其他大量的序列函数免费工作。这就是 Clojurists 所说的抽象编程，也是 Clojure 哲学的一个核心原则。

我认为抽象是操作的命名集合。如果你能在一个对象上执行一个抽象的所有操作，那么这个对象就是该抽象的一个实例。我甚至在编程之外也是这样想的。例如，_电池_抽象包括 "将导电介质连接到其阳极和阴极 "的操作，而该操作的输出是_电流_。电池是用锂还是用土豆做的并不重要。只要它对定义_电池_的一系列操作做出反应，它就是一个电池。

同样地，`map`并不关心列表、向量、集合和 Map 是如何实现的。它只关心它是否能对它们进行序列操作。让我们看看`map`是如何在序列抽象中定义的，这样你就能理解一般的抽象编程。

### 把列表、向量、集合和 Map 当作序列对待

如果你把`map`操作独立于任何编程语言，甚至是编程，它的基本行为是用一个函数_ƒ_从现有的序列_x_导出一个新的序列_y_，这样_y_1 = _ƒ_(_x_1), _y_2 = _ƒ_(_x_2), . . _y_n = _ƒ_(_x_n)。图 4-1 说明了你如何将应用于序列的映射可视化。

! [img](https://www.braveclojure.com/assets/images/cftbat/core-functions-in-depth/mapping.png)

图 4-1：映射的可视化

术语_序列_在这里指的是以线性顺序组织的元素集合，而不是无序集合或节点之间没有\*前后关系的图。图 4-2 显示了你如何将一个序列可视化，与上述其他两个集合形成对比。

![img](https://www.braveclojure.com/assets/images/cftbat/core-functions-in-depth/collections.png)

图 4-2：序列和非序列集合

在这个关于映射和序列的描述中，没有提到列表、向量或其他具体的数据结构。Clojure 的设计是让我们尽可能地用这种抽象的术语来思考和编程，它通过用数据结构的抽象来实现函数。在这个例子中，`map'是根据序列抽象来定义的。在对话中，你会说`map`、`reduce\`和其他序列函数_取一个序列_或甚至_取一个_seq\*。事实上，Clojurists 通常使用_seq_而不是_sequence_，使用_seq 函数_和_seq 库_等术语来指代执行顺序操作的函数。无论你使用_sequence_还是_seq_，你都表明有关的数据结构将被视为一个序列，在这种情况下，它实际上是什么最真实的心态并不重要。

如果核心序列函数 "first"、"rest "和 "cons "在一个数据结构上工作，你可以说这个数据结构_实现了_序列的抽象性。列表、向量、集合和 Map 都实现了序列抽象，所以它们都与`map`一起工作，如图所示。

```
(defn titleize
  [topic]
  (str topic " for the Brave and True"))

(map titleize ["Hamsters" "Ragnarok"])
; => ("Hamsters for the Brave and True" "Ragnarok for the Brave and True")

(map titleize '("Empathy" "Decorating"))
; => ("Empathy for the Brave and True" "Decorating for the Brave and True")

(map titleize #{"Elbows" "Soap Carving"})
; => ("Elbows for the Brave and True" "Soap Carving for the Brave and True")

(map #(titleize (second %)) {:uncomfortable-thing "Winking"})
; => ("Winking for the Brave and True")
```

前两个例子表明`map`对向量和列表的工作方式是相同的。第三个例子显示`map`可以与未排序的集合一起工作。在第四个例子中，你必须在匿名函数的参数上调用\`second'，然后再将其标题化，因为参数是一个 map。我将很快解释原因，但首先让我们看看定义序列抽象的三个函数。

### first, rest, and cons

![img](https://www.braveclojure.com/assets/images/cftbat/core-functions-in-depth/hamster.png)

在这一节中，我们将快速迂回到 JavaScript 中，实现一个链表和三个核心函数。`first',`rest', 和`cons'。在这三个核心函数实现之后，我将展示如何用它们来构建`map\`。

重点是要理解 Clojure 中的 seq 抽象和链接列表的具体实现之间的区别。如何实现一个特定的数据结构并不重要：当涉及到在一个数据结构上使用 seq 函数时，Clojure 所问的是 "我可以`first`、`rest`和`cons`吗？" 如果答案是肯定的，你就可以在该数据结构上使用 seq 库。

在一个链接列表中，节点是以线性顺序链接的。下面是你如何在 JavaScript 中创建一个。在这个片段中，`next`是空的，因为它是列表中的最后一个节点。

```
var node3 = {
  value: "last",
  next: null
};
```

在这个片段中，`node2`的`next`指向`node3`，而`node1`的`next`指向`node2`；这就是 "链表 "中的 "链接"。

```
var node2 = {
  value: "middle",
  next: node3
};

var node1 = {
  value: "first",
  next: node2
};
```

从图形上看，你可以如图 4-3 所示表示这个列表。

![img](https://www.braveclojure.com/assets/images/cftbat/core-functions-in-depth/linked-list.png)

图 4-3: 一个链接列表

你可以在一个链表上执行三个核心函数。`first`, `rest`, 和`cons`. `first`返回请求的节点的值，`rest`返回请求的节点之后的剩余值，`cons`在列表的开头添加一个具有给定值的新节点。在这些实现之后，你可以在它们之上实现`map`、`reduce`、`filter`和其他 seq 函数。

下面的代码显示了我们如何用我们的 JavaScript 节点例子实现和使用`first`、`rest`和`cons`，以及如何使用它们来返回特定的节点并导出一个新的列表。请注意，`first`和`rest`的参数被命名为_node_。这可能会让人感到困惑，因为你可能会说："我不是在获取一个_列表_的第一个元素吗？" 好吧，你一次对列表中的元素进行操作，是一个节点一个节点地操作

```
var first = function(node) {
  return node.value;
};

var rest = function(node) {
  return node.next;
};

var cons = function(newValue, node) {
  return {
    value: newValue,
    next: node
  };
};

first(node1);
// => "first"

first(rest(node1));
// => "middle"

first(rest(rest(node1)));
// => "last"

var node0 = cons("new first", node1);
first(node0);
// => "new first"

first(rest(node0));
// => "first"
```

如前所述，你可以用`first`、`rest`和`cons`来实现`map`。

```
var map = function (list, transform) {
  if (list === null) {
    return null;
  } else {
    return cons(transform(first(list)), map(rest(list), transform));
  }
}
```

这个函数转换了 list 的第一个元素，然后在 list 的其余部分再次调用自己，直到到达结尾（一个空值）。让我们看看它的运行情况 在这个例子中，你对以 `node1` 开始的列表进行映射，返回一个新的列表，字符串 `" mapped!"` 被附加到每个节点的值上。然后你用`first`来返回第一个节点的值。

```
first(
  map(node1, function (val) { return val + " mapped!"})
);

// => "first mapped!"
```

这里有件很酷的事：因为`map`是完全用`cons`、`first`和`rest`实现的，你实际上可以把任何数据结构传给它，只要`cons`、`first`和`rest`对该数据结构起作用，它就能工作。

下面是它们对一个数组的作用。

```
var first = function (array) {
  return array[0];
}

var rest = function (array) {
  var sliced = array.slice(1, array.length);
  if (sliced.length == 0) {
    return null;
  } else {
    return sliced;
  }
}

var cons = function (newValue, array) {
  return [newValue].concat(array);
}


var list = ["Transylvania", "Forks, WA"];
map(list, function (val) { return val + " mapped!"})
// => ["Transylvania mapped!", "Forks, WA mapped!"]
```

这个代码片段用 JavaScript 的数组函数定义了`first`、`rest`和`cons`。同时，`map`继续引用名为`first`、`rest`和`cons`的函数，所以现在它在`array`上工作。所以，如果你能实现`first`、`rest`和`cons`，你就能免费得到`map`和前面提到的大量其他函数。

### 通过定向进行抽象

在这一点上，你可能会反对我只是在踢皮球，因为我们仍然面临着像`first`这样的函数如何能够与不同的数据结构一起工作的问题。Clojure 使用两种形式的指示来实现这一目标。在编程中，_indirection_是一个通用术语，指的是一种语言所采用的机制，这样一个名字可以有多种相关的含义。在这个例子中，"first "这个名字有多种数据结构的含义。方向性是使抽象化成为可能的原因。

_多态性_是 Clojure 提供间接性的一种方式。我不想在细节上迷失方向，但基本上，多态函数根据提供的参数类型分配给不同的函数体。(这与多态函数根据你提供的参数数量派发到不同的函数体并无太大区别）。

注意 Clojure 有两种结构来定义多态调度：主机平台的接口结构和平台独立的协议。但在你刚开始的时候，没有必要了解这些东西是如何工作的。我将在第 13 章介绍协议。

当涉及到序列时，Clojure 也通过做一种轻量级的类型转换来创造间接性，产生一种数据结构，与抽象的函数一起工作。每当 Clojure 期望一个序列--例如，当你调用`map`、`first`、`rest`或`cons`时，它就会调用相关数据结构上的`seq`函数，以获得一个允许`first`、`rest`和`cons`的数据结构。

```
(seq '(1 2 3))
; => (1 2 3)

(seq [1 2 3])
; => (1 2 3)

(seq #{1 2 3})
; => (1 2 3)

(seq {:name "Bill Compton" :occupation "Dead mopey guy"})
; => ([:name "Bill Compton"] [:occupation "Dead mopey guy"])
```

这里有两个值得注意的细节。首先，`seq`总是返回一个看起来像列表的值；你会把这个值称为_sequence_或_seq_。第二，Map 的 seq 由两个元素的键值向量组成。这就是为什么`map`把你的 Map 当作向量列表的原因! 你可以在 "Bill Compton "的例子中看到这一点。我想特别指出这个例子，因为它可能是令人惊讶和困惑的。在我刚开始使用 Clojure 的时候就是这样。了解这些底层机制将使你不至于像试图保留人性的男性吸血鬼那样，经常表现出挫折感和普遍的拖沓感。

你可以通过使用`into`将 seq 转换回 Map，将结果粘到一个空的 Map 中（后面你会仔细看`into`）。

```
(into {} (seq {:a 1 :b 2 :c 3})
; => {:a 1, :c 3, :b 2}
```

所以，Clojure 的序列函数在其参数上使用`seq`。序列函数是根据序列抽象定义的，使用`first`、`rest`和`cons`。只要一个数据结构实现了序列抽象，它就可以使用广泛的 seq 库，其中包括诸如`reduce`、`filter`、`distinct`、`group-by`等超级明星函数。

这里的启示是，把注意力集中在我们能对一个数据结构做什么，并尽可能地忽略它的实现，是非常有力的。实现本身并不重要。它们只是达到目的的一种手段。一般来说，抽象编程可以让你在不同的数据结构上使用函数库，不管这些数据结构是如何实现的。

## Seq 函数实例

Clojure 的 seq 库中有很多有用的函数，你会经常用到。现在你已经对 Clojure 的序列抽象有了更深的了解，让我们来详细看看这些函数。如果你是 Lisp 和函数式编程的新手，这些例子将是令人惊讶和愉快的。

### Map

你现在已经看过很多`map`的例子了，但是这一节展示了`map`做了两个新的任务：把多个集合作为参数，以及把一个函数集合作为参数。它还强调了一个常见的`map`模式：使用关键字作为映射函数。

到目前为止，你只看到了`map`在一个集合上操作的例子。在下面的代码中，这个集合是向量`[1 2 3]`。

```
(map inc [1 2 3])
; => (2 3 4)
```

然而，你也可以给`map`多个集合。下面是一个简单的例子来说明这个方法的作用。

```
(map str ["a" "b" "c"] ["A" "B" "C"] )
; => ("aA" "bB" "cC")
```

这就好像`map`做了以下的事情。

```
(list (str "a" "A") (str "b" "B") (str "c" "C"))
```

![img](https://www.braveclojure.com/assets/images/cftbat/core-functions-in-depth/vampire-diary.png)

当你传递给`map`多个集合时，第一个集合的元素（`["a" "b" "c"]`）将作为映射函数（`str`）的第一个参数传递，第二个集合的元素（`["A" "B" "C"`）将作为第二个参数传递，以此类推。只要确保你的映射函数可以接受的参数数量与你传递给`map`的集合数量相等。

下面的例子显示了如果你是一个试图抑制人类消费的吸血鬼，你可以如何使用这种能力。你有两个向量，一个代表人类摄入的升数，另一个代表过去四天的小动物摄入量。unify-diet-data "函数获取人类和动物的单日数据，并将两者统一为一张 Map。

```
(def human-consumption   [8.1 7.3 6.6 5.0])
(def critter-consumption [0.0 0.2 0.3 1.1])
(defn unify-diet-data
  [human critter]
  {:human human
   :critter critter})

(map unify-diet-data human-consumption critter-consumption)
; => ({:human 8.1, :critter 0.0}
      {:human 7.3, :critter 0.2}
      {:human 6.6, :critter 0.3}
      {:human 5.0, :critter 1.1})
```

好样的，把人裁掉了!

你可以用`map`做的另一件有趣的事是把一个函数集合传给它。如果你想对不同的数字集合进行一系列的计算，你可以使用这个方法，就像这样。

```
(def sum #(reduce + %))
(def avg #(/ (sum %) (count %)))
(defn stats
  [numbers]
  (map #(% numbers) [sum count avg]))

(stats [3 4 10])
; => (17 3 17/3)

(stats [80 1 44 13 6])
; => (144 5 144/5)
```

在这个例子中，`stats`函数遍历了一个函数的向量，将每个函数应用于`numbers`。

此外，Clojurists 经常使用`map`从 map 数据结构的集合中检索与一个关键词相关的值。因为关键字可以作为函数使用，你可以简洁地做到这一点。下面是一个例子。

```
(def identities
  [{:alias "Batman" :real "Bruce Wayne"}
   {:alias "Spider-Man" :real "Peter Parker"}
   {:alias "Santa" :real "Your mom"}
   {:alias "Easter Bunny" :real "Your dad"}])

(map :real identities)
; => ("Bruce Wayne" "Peter Parker" "Your mom" "Your dad")
```

(如果你是五岁，那么我深表歉意）。

### reduce

第 3 章展示了`reduce`如何处理序列中的每个元素来建立一个结果。本节展示了其他一些可能不明显的使用方法。

第一种用法是转换一个 Map 的值，产生一个新的 Map，其键值相同，但数值更新。

```
(reduce (fn [new-map [key val]]
          (assoc new-map key (inc val)))
        {}
        {:max 30 :min 10})
; => {:max 31, :min 11}
```

在这个例子中，`reduce`将参数`{:max 30 :min 10}`视为一个向量序列，如`([:max 30] [:min 10])`。然后，它从一个空图（第二个参数）开始，用第一个参数，一个匿名函数来建立它。就像`reduce`这样做。

```
(assoc (assoc {} :max (inc 30))
       :min (inc 10))
```

函数`assoc`需要三个参数：一个 Map，一个键，和一个值。它通过_关联_给定的键和给定的值，从你给它的 Map 中派生出一个新的 Map。例如，`(assoc {:a 1} :b 2)`将返回`{:a 1 :b 2}`。

重组 "的另一个用途是根据键值从 Map 中过滤出来。在下面的例子中，匿名函数检查一个键值对的值是否大于 4，如果不是，那么这个键值对就被过滤掉了。在 Map`{:human 4.1 :critter 3.9}`中，3.9 小于 4，所以`:critter`键和它的 3.9 值被过滤掉了。

```
(reduce (fn [new-map [key val]]
          (if (> val 4)
            (assoc new-map key val)
            new-map))
        {}
        {:human 4.1
         :critter 3.9})
; => {:human 4.1}
```

这里的启示是，`reduce'是一个比最初看起来更灵活的函数。每当你想从一个seqable数据结构中得到一个新的值时，`reduce`通常能够满足你的需要。如果你想做一个真正能让你的头发倒竖的练习，试着用`reduce`实现`map`，然后在本章后面的内容中对`filter`和`some\`做同样的练习。

### take, drop, take-while, and drop-while

`take`和`drop`都接受两个参数：一个数字和一个序列。`take'返回序列的前n个元素，而`drop'返回除去前 n 个元素的序列。

```
(take 3 [1 2 3 4 5 6 7 8 9 10])
; => (1 2 3)

(drop 3 [1 2 3 4 5 6 7 8 9 10])
; => (4 5 6 7 8 9 10)
```

它们的表亲`take-while`和`drop-while`更有趣一些。每一个都需要一个_谓词函数_（一个其返回值被评估为真或假的函数）来决定它何时应该停止取舍。例如，假设你有一个向量，代表你 "食物 "日记中的条目。每个条目都有月份和日期，以及你吃了什么。为了保留空间，我们将只包括几个条目。

```
(def food-journal
  [{:month 1 :day 1 :human 5.3 :critter 2.3}
   {:month 1 :day 2 :human 5.1 :critter 2.0}
   {:month 2 :day 1 :human 4.9 :critter 2.1}
   {:month 2 :day 2 :human 5.0 :critter 2.5}
   {:month 3 :day 1 :human 4.2 :critter 3.3}
   {:month 3 :day 2 :human 4.0 :critter 3.8}
   {:month 4 :day 1 :human 3.7 :critter 3.9}
   {:month 4 :day 2 :human 3.7 :critter 3.6}])
```

使用`take-while`，你可以只检索一月和二月的数据。`take-while`遍历给定的序列（在本例中是`food-journal`），对每个元素应用谓词函数。

这个例子使用匿名函数`#(< (:month %) 3)`来测试日记条目的月份是否超出范围。

```
(take-while #(< (:month %) 3) food-journal)
; => ({:month 1 :day 1 :human 5.3 :critter 2.3}
      {:month 1 :day 2 :human 5.1 :critter 2.0}
      {:month 2 :day 1 :human 4.9 :critter 2.1}
      {:month 2 :day 2 :human 5.0 :critter 2.5})
```

当`take-while`到达第一个 March 条目时，匿名函数返回`false`，而`take-while`返回它在这之前测试的每个元素的序列。

同样的想法也适用于`drop-while`，只是它一直在丢弃元素，直到有一个测试为真。

```
(drop-while #(< (:month %) 3) food-journal)
; => ({:month 3 :day 1 :human 4.2 :critter 3.3}
      {:month 3 :day 2 :human 4.0 :critter 3.8}
      {:month 4 :day 1 :human 3.7 :critter 3.9}
      {:month 4 :day 2 :human 3.7 :critter 3.6})
```

通过同时使用`take-while`和`drop-while`，你可以只获得 2 月和 3 月的数据。

```
(take-while #(< (:month %) 4)
            (drop-while #(< (:month %) 2) food-journal))
; => ({:month 2 :day 1 :human 4.9 :critter 2.1}
      {:month 2 :day 2 :human 5.0 :critter 2.5}
      {:month 3 :day 1 :human 4.2 :critter 3.3}
      {:month 3 :day 2 :human 4.0 :critter 3.8})
```

这个例子使用 "drop-while "去掉 1 月份的条目，然后对结果使用 "take-while "继续取条目，直到到达 4 月份的第一个条目。

### 过滤和一些

使用`filter`来返回一个序列中对一个谓词函数测试为真的所有元素。这里是人类消费少于 5 升的日记条目。

```
(filter #(< (:human %) 5) food-journal)
; => ({:month 2 :day 1 :human 4.9 :critter 2.1}
      {:month 3 :day 1 :human 4.2 :critter 3.3}
      {:month 3 :day 2 :human 4.0 :critter 3.8}
      {:month 4 :day 1 :human 3.7 :critter 3.9}
      {:month 4 :day 2 :human 3.7 :critter 3.6})
```

你可能想知道为什么我们不在前面的 "take-while "和 "drop-while "例子中使用`filter`。事实上，`filter`也可以用于此。这里我们要抓取 1 月和 2 月的数据，就像在`take-while`例子中一样。

```
(filter #(< (:month %) 3) food-journal)
; => ({:month 1 :day 1 :human 5.3 :critter 2.3}
      {:month 1 :day 2 :human 5.1 :critter 2.0}
      {:month 2 :day 1 :human 4.9 :critter 2.1}
      {:month 2 :day 2 :human 5.0 :critter 2.5})
```

这种用法完全没有问题，但是`filter`最终会处理你的所有数据，这并不总是必要的。因为食物日记已经按日期排序，我们知道`take-while`会返回我们想要的数据，而不需要检查任何我们不需要的数据。因此，`take-while`可以更有效率。

通常情况下，你想知道一个集合是否包含对一个谓词函数测试为真的任何值。`some`函数就是这样做的，它返回由一个谓词函数返回的第一个真值（任何不是`false`或`nil`的值）。

```
(some #(> (:critter %) 5) food-journal)
; => nil

(some #(> (:critter %) 3) food-journal)
; => true
```

你没有任何食物日记条目显示你从小动物来源中消耗了超过 5 升的食物，但是你至少有一条显示你消耗了超过 3 升的食物。请注意，第二个例子中的返回值是`true`，而不是产生真值的实际条目。原因是匿名函数`#(> (:critter %) 3)`返回`true`或`false`。下面是你如何返回该条目。

```
(some #(and (> (:critter %) 3) %) food-journal)
; => {:month 3 :day 1 :human 4.2 :critter 3.3}。
```

这里，一个稍有不同的匿名函数使用`and`首先检查条件`(> (:critter %) 3)`是否为真，然后在条件确实为真时返回条目。

### sort and sort-by

你可以用`sort`将元素按升序排序。

```
(sort [3 1 2])
; => (1 2 3)
```

如果你的排序需求更复杂，你可以使用`sort-by`，它允许你将一个函数（有时称为_键函数_）应用于一个序列的元素，并使用它返回的值来决定排序顺序。在下面的例子中，取自\*[http://clojuredocs.org/](http://clojuredocs.org)\*，`count`是关键函数。

```
(sort-by count ["aaa" "c" "bb"] )
; => ("c" "bb" "aaa")
```

如果你使用`sort`进行排序，元素将按字母顺序进行排序，返回`("aaa" "bb" "c")`。相反，结果是`("c" "bb" "aaa")`，因为你是按`count`排序，而`"c "的计数是1，`"bb "是 2，\`"aaa "是 3。

### 协程

最后, `concat`简单地将一个序列的成员附加到另一个序列的末尾:

```
(concat [1 2] [3 4])
; => (1 2 3 4)
```

## Lazy Seqs

正如你之前看到的，`map`首先在你传递给它的集合上调用`seq`。但这并不是故事的全部。许多函数，包括`map`和`filter`，都返回一个_lazy seq_。lazy seq 是一个 seq，它的成员在你试图访问它们时才被计算。计算一个 seq 的成员被称为_实现_seq。将计算推迟到需要的时候，可以使你的程序更有效率，而且它还有一个令人惊讶的好处，就是允许你构建无限的序列。

### 演示懒惰序列的效率

为了看到懒惰序列的作用，假装你是一个现代任务组的成员，其目的是为了识别吸血鬼。你的情报人员告诉你，在你的城市里只有一个活跃的吸血鬼，而且他们已经帮助你把嫌疑人的名单缩小到一百万人。你的老板给了你一份一百万个社会安全号码的名单，并喊道："搞定它，麦克菲斯维奇！"

值得庆幸的是，你拥有一台 Vampmatic 3000 计算机，这是用于识别吸血鬼的最先进的设备。由于这种猎杀吸血鬼的技术的源代码是专有的，我把它存根出来，模拟执行这项任务所需的时间。这里是一个吸血鬼数据库的子集。

```
(def vampire-database
  {0 {:makes-blood-puns? false, :has-pulse? true  :name "McFishwich"}
   1 {:makes-blood-puns? false, :has-pulse? true  :name "McMackson"}
   2 {:makes-blood-puns? true,  :has-pulse? false :name "Damon Salvatore"}
   3 {:makes-blood-puns? true,  :has-pulse? true  :name "Mickey Mouse"}})

(defn vampire-related-details
  [social-security-number]
  (Thread/sleep 1000)
  (get vampire-database social-security-number))

(defn vampire?
  [record]
  (and (:makes-blood-puns? record)
       (not (:has-pulse? record))
       record))

(defn identify-vampire
  [social-security-numbers]
  (first (filter vampire?
                 (map vampire-related-details social-security-numbers))))
```

你有一个函数，`vampire-related-details`，它需要一秒钟从数据库中查找一个条目。接下来，你有一个函数，`vampire?`，如果它通过了吸血鬼测试，就返回一条记录；否则，就返回`false`。最后，`identify-vampire`将社会安全号码映射到数据库记录，然后返回第一条表明有吸血鬼的记录。

为了显示运行这些函数需要多少时间，你可以使用`time`操作。当你使用`time`时，你的代码的行为与你不使用`time`时完全一样，但有一个例外：会打印出一份经过时间的报告。下面是一个例子。

```
(time (vampire-related-details 0))
; => "Elapsed time: 1001.042 msecs"
; => {:name "McFishwich", :makes-blood-puns? false, :has-pulse? true}
```

第一个打印行报告了给定操作所花费的时间--本例是 1,001.042 毫秒。第二行是返回值，在本例中是你的数据库记录。返回值与没有使用`time`的情况下完全相同。

一个不笨的`map`的实现首先要对`social-security-numbers`的每个成员应用`vampire-`related-details`，然后再把结果传给`filter\`。因为你有一百万个嫌疑人，这将需要一百万秒，也就是 12 天，到那时你的一半城市都会死掉！"。当然，如果结果是唯一的吸血鬼是记录中的最后一个嫌疑人，用懒人版本还是会花那么多时间，但至少有一个很好的机会，它不会。

因为`map`是懒惰的，在你试图访问映射的元素之前，它实际上并没有将`吸血鬼相关的细节'应用于社会安全号码。事实上，`map\`几乎立刻就会返回一个值。

第一个打印行报告了给定操作所花费的时间--本例中是 1,001.042 毫秒。第二行是返回值，在这个例子中是你的数据库记录。返回值与没有使用`time`的情况下完全相同。

一个不笨的`map`的实现首先要对`social-security-numbers`的每个成员应用`vampire-`related-details`，然后再把结果传给`filter\`。因为你有一百万个嫌疑人，这将需要一百万秒，也就是 12 天，到那时你的一半城市都会死掉！"。当然，如果结果是唯一的吸血鬼是记录中的最后一个嫌疑人，用懒人版本还是会花那么多时间，但至少有一个很好的机会，它不会。

因为`map`是懒惰的，在你试图访问映射的元素之前，它实际上并没有将`吸血鬼相关的细节'应用于社会安全号码。事实上，`map\`几乎马上就会返回一个值。

```
(time (def mapped-details (map vampire-related-details (range 0 1000000))))
; => "Elapsed time: 0.049 msecs"
; => #'user/mapped-details
```

在这个例子中，`range`返回一个由 0 到 999,999 的整数组成的懒惰序列。然后，`map`返回一个与名称`mapped-details`相关的懒惰序列。因为`map`实际上没有对`range`返回的任何元素应用`vampire-related-details`，整个操作几乎没有花费任何时间，当然，少于 12 天。

你可以认为懒人序列是由两部分组成的：一个关于如何实现序列元素的配方和到目前为止已经实现的元素。当你使用`map`时，它返回的懒人序列不包括任何已实现的元素，但它确实有生成其元素的配方。每当你试图访问一个未实现的元素时，懒人序列将使用它的配方来生成所请求的元素。

在前面的例子中，`mapped-details`是未实现的。一旦你试图访问`mapped-details`的一个成员，它将使用它的配方来生成你所请求的元素，你将产生每秒钟的数据库查询费用。

在这个例子中，`range`返回一个由 0 到 999,999 的整数组成的懒惰序列。然后，`map`返回一个与`mapped-details`名称相关的懒惰序列。因为`map`实际上没有对`range`返回的任何元素应用`vampire-related-details`，整个操作几乎没有花费任何时间，当然，少于 12 天。

你可以认为懒人序列由两部分组成：一个关于如何实现序列元素的配方和到目前为止已经实现的元素。当你使用`map`时，它返回的懒惰序列不包括任何已实现的元素，但它确实有生成其元素的配方。每当你试图访问一个未实现的元素时，懒人序列将使用它的配方来生成所请求的元素。

在前面的例子中，`mapped-details`是未实现的。一旦你试图访问`mapped-details`的一个成员，它将使用它的配方来生成你所请求的元素，你将产生每秒钟的数据库查询费用。

```
(time (first mapped-details))
; => "Elapsed time: 32030.767 msecs"
; => {:name "McFishwich", :makes-blood-puns? false, :has-pulse? true}
```

这个操作花了大约 32 秒。这比一百万秒好得多，但还是比我们预期的多了 31 秒。毕竟，你只是试图访问第一个元素，所以它应该只花一秒钟。

花了 32 秒的原因是 Clojure_chunks_它的懒惰序列，这只是意味着每当 Clojure 要实现一个元素时，它也会预先实现一些下一个元素的实现。在这个例子中，你只想要\`mapped-details'的第一个元素，但 Clojure 继续前进，也准备了后面的 31 个元素。Clojure 这样做是因为它几乎总是能带来更好的性能。

值得庆幸的是，懒惰的 seq 元素只需要实现一次。再次访问`mapped-details`的第一个元素几乎不需要时间。

```
(time (first mapped-details))
; => "Elapsed time: 0.022 msecs"
; => {:name "McFishwich", :makes-blood-puns? false, :has-pulse? true}
```

有了这些新发现的知识，你就可以有效地挖掘吸血鬼数据库，找到带獠牙的罪魁祸首。

```
(time (identify-vampire (range 0 1000000)))
"Elapsed time: 32019.912 msecs"
; => {:name "Damon Salvatore", :makes-blood-puns? true, :has-pulse? false}
```

哦！这就是为什么达蒙会做出那些令人毛骨悚然的双关语的原因。

### 无限序列

lazy seqs 给你的一个很酷、很有用的能力是构建无限序列的能力。到目前为止，你只处理过从向量或列表中生成的懒惰序列，这些序列是终止的。然而，Clojure 自带了一些函数来创建无限序列。创建无限序列的一个简单方法是使用`repeat`，它创建一个序列，其每个成员都是你传递的参数。

```
(concat (take 8 (repeat "na")) ["Batman!"])
; => ("na" "na" "na" "na" "na" "na" "na" "na" "Batman!")
```

在这种情况下，你创建了一个无限的序列，其中每个元素都是字符串 "na"，然后用它来构建一个可能引起或不引起怀旧情绪的序列。

你也可以使用`repeatedly`，它将调用提供的函数来生成序列中的每个元素。

```
(take 3 (repeatedly (fn [] (rand-int 10))))
; => (1 4 0)
```

这里，由`repeatedly`返回的懒惰序列通过调用匿名函数`(fn [] (rand-int 10))`生成每个新元素，该函数返回一个 0 到 9 之间的随机整数。如果你在你的 REPL 中运行这个，你的结果很可能与此不同。

lazy seq 的配方不需要指定一个端点。像`first`和`take`这样的函数实现了懒惰序列，它们没有办法知道序列的下一步是什么，如果序列一直提供元素，那么它们就会一直取走它们。如果你构建你自己的无限序列，你就可以看到这一点。

```
(defn even-numbers
  ([] (even-numbers 0))
  ([n] (cons n (lazy-seq (even-numbers (+ n 2))))))

(take 10 (even-numbers))
; => (0 2 4 6 8 10 12 14 16 18)
```

这个例子有点令人费解，因为它使用了递归。记住`cons`返回一个新的列表，并将一个元素追加到给定的列表中，会有所帮助。

```
(cons 0 '(2 4 6))
; => (0 2 4 6)
```

(顺便说一下，Lisp 程序员在使用`cons`函数时称它为_consing_)。

在 "偶数 "中，你是在对一个懒惰列表进行 consing，其中包括一个关于下一个元素的配方（一个函数）（而不是对一个完全实现的列表进行 consing）。

这就涵盖了懒惰序列! 现在你知道了关于序列抽象的所有知识，我们可以转向集合抽象了。

## 集合抽象

集合的抽象与序列的抽象密切相关。所有 Clojure 的核心数据结构--向量、映射、列表和集合--都参与了这两个抽象。

序列抽象是关于对成员的单独操作，而集合抽象是关于数据结构的整体。例如，集合函数 `count`, `empty?`, 和 `every?` 不是关于任何单独的元素；它们是关于整体的。

```
(empty?[])
; => true

(empty? ["no!"])
; => false
```

实际上，你很少会有意识的说："好的，自己！"。你现在是在和整个集合一起工作。从集合抽象的角度来考虑！" 尽管如此，了解这些作为你所使用的函数和数据结构基础的概念还是很有用的。

现在我们来研究两个常见的集合函数--`into`和`conj`，它们的相似性可能会让人有点困惑。

### into

最重要的集合函数之一是`into`。正如你现在所知，许多 seq 函数返回一个 seq，而不是原始数据结构。你可能想把返回值转换成原始值, `into`让你做到这一点:

```
(map identity {:sunlight-reaction "Glitter!"})
; => ([:sunlight-reaction "Glitter!"])

(into {} (map identity {:sunlight-reaction "Glitter!"}))
; => {:sunlight-reaction "Glitter!"}.
```

在这里，`map`函数在得到一个 map 数据结构后返回一个顺序数据结构，并将 seq 转换回 map。

这也适用于其他数据结构。

```
(map identity [:garlic :sesame-oil :fried-eggs])
; => (:garlic :sesame-oil :fried-eggs)

(into [] (map identity [:garlic :sesame-oil :fried-eggs]))
; => [:garlic :sesame-oil :fried-eggs]
```

这里，在第一行，`map`返回一个序列，我们在第二行使用`into`将结果转换为一个向量。

在下面的例子中，我们从一个有两个相同条目的向量开始，`map`把它转换为一个列表，然后我们用`into`把值粘到一个集合中。

```
(map identity [:garlic-clove :garlic-clove])
; => (:garlic-clove :garlic-clove)

(into #{} (map identity [:garlic-clove :garlic-clove]))
; => #{:garlic-clove}
```

因为集合只包含唯一的值，所以集合中最终只有一个值。

`into`的第一个参数不一定是空的。这里，第一个例子显示了如何使用`into`向 Map 添加元素，第二个例子显示了如何向矢量添加元素。

```
(into {:favorite-emotion "gloomy"} [[:sunlight-reaction "Glitter!"]])
; => {:favorite-emotion "gloomy" :sunlight-reaction "Glitter!"}

(into ["cherry"] '("pine" "spruce"))
; => ["cherry" "pine" "spruce"]
```

当然，两个参数也可以是同一类型。在下一个例子中，两个参数都是 Map，而之前所有的例子都有不同类型的参数。它的工作原理和你所期望的一样，返回一个新的 Map，将第二个 Map 的元素添加到第一个 Map 中。

```
(into {:favorite-animal "kitty"} {:least-favorite-smell "dog"
                                  :relationship-with-teenager "creepy"})
; => {:favorite-animal "kitty"
      :relationship-with-teenager "creepy"
      :least-favorite-smell "dog"}
```

如果`into`在求职面试中被要求描述它的优势，它会说："我很擅长处理两个集合，并将第二个集合中的所有元素添加到第一个集合中。"

### conj

`conj`也是向一个集合添加元素，但它的方式略有不同。

```
(conj [0] [1])
; => [0 [1]]
```

呜呜呜! 看起来它把整个向量`[1]`添加到`[0]`。与`into`比较。

```
(into [0] [1])
; => [0 1]
```

下面是我们如何用`conj`做同样的事情。

```
(conj [0] 1)
; => [0 1]
```

注意，数字 1 是作为标量（单数，非集合）值传递的，而`into`的第二个参数必须是一个集合。

你可以提供尽可能多的元素与`conj`一起添加，你也可以添加到其他集合中，如 map。

```
(conj [0] 1 2 3 4)
; => [0 1 2 3 4]

(conj {:time "midnight"} [:place "ye olde cemetarium"])
; => {:place "ye olde cemetarium" :time "midnight"}
```

`conj`和`into`如此相似，你甚至可以用`into`来定义`conj`。

```
(defn my-conj
  [target & additions]
  (into target additions))

(my-conj [0] 1 2 3)
; => [0 1 2 3]
```

## 功能函数

学习利用 Clojure 的接受函数作为参数和返回函数作为值的能力是非常有趣的，即使它需要一些适应性。

Clojure 的两个函数，`apply`和`partial`，可能看起来特别奇怪，因为它们都接受_和_返回函数。让我们来解开它们的疑惑。

### 应用

`apply` _explodes_一个 seqable 数据结构，所以它可以被传递给一个期望有其余参数的函数。例如，`max`接受任何数量的参数，并返回所有参数中最大的一个。这里是你如何找到最大的数字。

```
(max 0 1 2)
; => 2
```

但如果你想找到一个向量的最大元素，怎么办？你不能只把向量传给`max`。

```
(max [0 1 2])
; => [0 1 2]
```

这不会返回向量中最大的元素，因为`max`返回所有传递给它的参数中最大的，在这种情况下，你只是传递给它一个包含所有你想比较的数字的向量，而不是把数字作为单独的参数传递进去。`apply`是这种情况的完美选择。

```
(apply max [0 1 2])
; => 2
```

通过使用`apply`，就像你调用`(max 0 1 2)`一样。你经常会像这样使用`apply`，对一个集合的元素进行分解，使它们作为单独的参数被传递给一个函数。

还记得我们之前是如何用 "into "来定义 "conj "的吗？那么，我们也可以通过使用`apply`在`conj`的基础上定义`into`。

```
(defn my-into
  [target additions]
  (apply conj target additions))

(my-into [0] [1 2 3])
; => [0 1 2 3]
```

对`my-into`的调用相当于调用`(conj [0] 1 2 3)`。

### 部分

`partial`接收一个函数和任意数量的参数。然后它返回一个新的函数。当你调用返回的函数时，它用你提供的原参数和新参数一起调用原函数。

这里有一个例子。

```
(def add10 (partial + 10))
(add10 3) 
; => 13
(add10 5) 
; => 15

(def add-missing-elements
  (partial conj ["water" "earth" "air"]))

(add-missing-elements "unobtainium" "adamantium")
; => ["water" "earth" "air" "unobtainium" "adamantium"]
```

所以当你调用`add10'时，它会调用原始函数和参数（`+ 10'），并附加你调用`add10'的任何参数。为了帮助澄清`partial'的工作原理，下面是你如何定义它。

```
(defn my-partial
  [partialized-fn & args]
  (fn [& more-args]
    (apply partialized-fn (into args more-args))))

(def add20 (my-partial + 20))
(add20 3) 
; => 23
```

在这个例子中，`add20'的值是由`my-partial'返回的匿名函数。这个匿名函数是这样定义的。

```
(fn [& more-args]
  (apply + (into [20] more-args)))
```

一般来说，当你发现你在许多不同的情况下重复相同的函数和参数组合时，你会想使用 partials。这个玩具例子显示了你如何使用`partial`来专门化一个记录器，创建一个`warn`函数。

```
(defn lousy-logger
  [log-level message]
  (condp = log-level
    :warn (clojure.string/lower-case message)
    :emergency (clojure.string/upper-case message)))

(def warn (partial lousy-logger :warn))

(warn "Red light ahead")
; => "red light ahead"
```

在这里调用`(warning "Red light ahead")`与调用`(lousy-logger :warning "Red light ahead")`是相同的。

### 补充

早些时候，你创建了\`识别吸血鬼'函数，以便在一百万人中找到一个吸血鬼。如果你想创建一个函数来寻找所有的人类呢？也许你想给他们发送感谢卡，因为他们没有成为不死的掠夺者。这里是你可以做的。

```
(defn identify-humans
  [social-security-numbers]
  (filter #(not (vampire? %))
          (map vampire-related-details social-security-numbers)))
```

看看`filter`的第一个参数，`#(not (vampire? %))`。想要得到一个布尔函数的_complement_（否定）是很常见的，所以有一个函数，`complement`，用于此。

```
(def not-vampire? (complement vampire?))
(defn identify-humans
  [social-security-numbers]
  (filter not-vampire?
          (map vampire-related-details social-security-numbers)))
```

下面是你如何实现 "补充 "的方法。

```
(defn my-complement
  [fun]
  (fn [& args]
    (not (apply fun args))))

(def my-pos? (complement neg?))
(my-pos? 1)  
; => true

(my-pos? -1) 
; => false
```

正如你所看到的，`complement'是一个不起眼的函数。它只做一件小事，而且做得很好。`complement`使创建一个`不吸血'的函数变得微不足道，而且任何阅读代码的人都能理解代码的意图。

这不会为你提供数兆字节的数据的 MapReduce 或类似的东西，但它确实证明了高阶函数的力量。它们允许你以一种在某些语言中不可能实现的方式建立起实用函数库。总的来说，这些实用函数使你的生活变得更加轻松。

## A Vampire Data Analysis Program for the FWPD

为了把所有的事情联系起来，让我们为华盛顿州福克斯警察局（FWPD）编写一个复杂的吸血鬼数据分析程序的雏形。

FWPD 有一个花哨的新数据库技术，叫做_CSV_ _（逗号-\*\*分隔的_值）_。你的工作是解析这个最先进的 CSV，并分析它是否有潜在的吸血鬼。我们将通过过滤每个嫌疑人的_闪光指数\*来做到这一点，这是一个由某个少女开发的对嫌疑人的吸血鬼性的 0-10 预测。继续并为你的工具创建一个新的 Leiningen 项目。

```
lein new app fwpd
```

在新的_fwpd_目录下，创建一个名为_suspects.csv_的文件，输入如下内容。

```
Edward Cullen,10
Bella Swan,0
Charlie Swan,0
Jacob Black,3
Carlisle Cullen,6
```

现在是时候通过建立_fwpd/src/fwpd/core.clj_文件来弄脏你的手了。我建议你启动一个新的 REPL 会话，这样你就可以边走边试。在 Emacs 中，你可以通过打开_fwpd/\*\*src/fwpd/core.clj_并运行**M-x** cider-restart 来实现。一旦 REPL 启动，删除_core.clj_的内容，然后加入以下内容。

```
(ns fwpd.core)
(def filename "suspects.csv")
```

第一行建立了命名空间，第二行只是使你创建的 CSV 更容易被引用。你可以通过编译你的文件（Emacs 中的**C-c C-k**）并运行以下程序，在你的 REPL 中做一个快速的理智检查。

```
(slurp filename)
; => "Edward Cullen,10\nBella Swan,0\nCharlie Swan,0\nJacob Black, 3\nCarlisle Cullen, 6"
```

如果`slurp`函数没有返回前面的字符串，试着在_core.clj_打开的情况下重新启动你的 REPL 会话。

接下来，在_core.clj_中添加这个内容。

```
➊ (def vamp-keys [:name :glitter-index])

➋ (defn str->int
  [str]
  (Integer. str))

➌ (def conversions {:name identity
                  :glitter-index str->int})

➍ (defn convert
  [vamp-key value]
  ((get conversions vamp-key) value))
```

最终，你会得到一串看起来像`{:name "Edward Cullen" :glitter-index 10}`的 Map，前面的定义可以帮助你达到目的。首先，`vamp-keys`➊是一个键的向量，你很快会用它来创建吸血鬼 Map。接下来，函数`str->int`➋将一个字符串转换为一个整数。Map`conversions`➌将一个转换函数与每个吸血鬼键相关联。你根本不需要转换名字，所以它的转换函数是`identity`，它只是返回传递给它的参数。熠熠生辉的索引被转换为一个整数，所以它的转换函数是`str->int`。最后，`convert`函数➍接收一个 vamp 键和一个值，并返回转换后的值。下面是一个例子。

```
(convert :glitter-index "3")
; => 3
```

现在把这个添加到你的文件中。

```
(defn parse
  "Convert a CSV into rows of columns"
  [string]
  (map #(clojure.string/split % #",")
       (clojure.string/split string #"\n")))
```

`parse`函数接收一个字符串，首先在换行符上进行分割，创建一个字符串的序列。接下来，它对字符串序列进行映射，在逗号字符上分割每一个字符串。试着在你的 CSV 上运行`parse`。

```
(parse (slurp filename))
; => (["Edward Cullen" "10"] ["Bella Swan" "0"] ["Charlie Swan" "0"]
      ["Jacob Black" "3"] ["Carlisle Cullen" "6"])
```

接下来的代码将向量序列与你的吸血鬼钥匙结合起来，创建 Map。

```
(defn mapify
  "Return a seq of maps like {:name \"Edward Cullen\" :glitter-index 10}"
  [rows]
  (map (fn [unmapped-row]
         (reduce (fn [row-map [vamp-key value]]
                   (assoc row-map vamp-key (convert vamp-key value)))
                 {}
                 (map vector vamp-keys unmapped-row)))
       rows))
```

在这个函数中，`map`通过使用`reduce`将每一行向量如`["Bella Swan" 0]`转化为一个 Map，其方式与上面"`reduce`"中的第一个例子相似。首先，`map`创建一个键值对序列，如`([:name "Bella Swan"] [:glitter-index 0])`。然后，"reduce "通过将一个 vamp 键和一个转换后的 vamp 值关联到 "row-map "来建立一个 Map。下面是第一行的映射。

```
(first (mapify (parse (slurp filename))))
; => {:glitter-index 10, :name "Edward Cullen" }
```

最后，添加这个`glitter-filter`函数。

```
(defn glitter-filter
  [minimum-glitter records]
  (filter #(>= (:glitter-index %) minimum-glitter) records))
```

这需要完全映射的吸血鬼记录，并过滤掉那些`:glitter-index`小于所提供的`minimum-glitter`的记录。

```
(glitter-filter 3 (mapify (parse (slurp filename))))
({:name "Edward Cullen", :glitter-index 10}
 {:name "Jacob Black", :glitter-index 3}
 {:name "Carlisle Cullen", :glitter-index 6})
```

Et voilà! 你现在离实现你的梦想又近了一步，即成为一名猎杀超自然生物的义务警员。你最好去围捕那些粗略的人物!

## 摘要

在本章中，你了解到 Clojure 强调对抽象的编程。序列抽象处理的是对序列中各个元素的操作，而 seq 函数通常将其参数转换为 seq，并返回一个懒惰的 seq。懒惰评估通过将计算推迟到需要时再进行，从而提高性能。你所学到的另一个抽象，即集合抽象，处理的是整个数据结构。最后，你学到的最重要的东西是，你不应该相信那些在阳光下闪光的人。

## 练习

你现在拥有的吸血鬼分析程序已经领先于市场上的任何其他程序几十年了。但你怎样才能使它变得更好呢？我建议尝试以下几点。

1. 把你的闪光过滤器的结果变成一个名字的列表。
2. 写一个函数，\`append'，它将把一个新的嫌疑人追加到你的嫌疑人列表中。
3. 写一个函数，`validate'，它将在你`append'时检查`:name'和`:glitter-index'是否存在。`validate`函数应该接受两个参数：一个类似于`conversions`的验证函数的关键词映射，以及要验证的记录。
4. 编写一个函数，将你的 Map 列表转换为 CSV 字符串。你需要使用`clojure.string/join`函数。

祝你好运，McFishwich!
