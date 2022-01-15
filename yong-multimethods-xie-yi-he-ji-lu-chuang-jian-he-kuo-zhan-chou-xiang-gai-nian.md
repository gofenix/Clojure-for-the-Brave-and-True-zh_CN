# 用 Multimethods、协议和记录创建和扩展抽象概念

花一分钟思考一下，作为大自然的顶级产品之一：人类是多么伟大。作为一个人类，你可以在社交媒体上闲聊，玩龙与地下城，戴帽子。也许更重要的是，你可以用抽象的概念来思考和交流。

抽象思考的能力确实是人类最好的特征之一。它可以让你规避你的认知极限，将不同的细节捆绑在一起，形成一个整齐的概念包，让你可以在工作记忆中持有。你不需要去想 "可挤压的红球鼻子装饰 "这种笨重的想法，而只需要 "小丑鼻子 "这个概念。

在 Clojure 中，一个_抽象_是一个操作的集合，而_数据类型_实现抽象。例如，seq 抽象由 "first "和 "rest "等操作组成，而向量数据类型是该抽象的实现；它对所有 seq 操作做出响应。像`[:seltzer :water]`这样的特定向量是该数据类型的\*实例。

编程语言越是让你以抽象的方式思考和写作，你的生产力就越高。例如，如果你知道一个数据结构是 seq 抽象的一个实例，你就可以立即调用一个大的知识网，了解哪些函数可以与数据结构一起工作。因此，你会花时间去实际使用这个数据结构，而不是不断地去查找关于它如何工作的文档。同样地，如果你扩展一个数据结构，使其与 seq 抽象一起工作，你就可以在上面使用大量的 seq 函数库。

在第四章中，你了解到 Clojure 是以抽象的方式编写的。这很强大，因为在 Clojure 中，你可以专注于你可以用数据结构实际做的事情，而不用担心实现的细枝末节。本章向你介绍了创建和实现你自己的抽象的世界。你将学习 Multimethods、协议和记录的基础知识。

## 多态性

我们在 Clojure 中实现抽象的主要方式是将一个操作名称与一个以上的算法联系起来。这种技术被称为_多态性_。例如，在列表上执行 "conj "的算法与向量的算法不同，但我们把它们统一在同一个名字下，以表明它们实现了同一个概念，即_向_这个数据结构添加一个元素。

因为 Clojure 的许多数据类型都依赖于 Java 的标准库，所以本章中使用了一点 Java。例如，Clojure 的字符串只是 Java 的字符串，是 Java 类`java.lang.String`的实例。要在 Java 中定义你自己的数据类型，你要使用类。Clojure 提供了额外的类型结构。 _记录_和_类型_。本书只涉及记录。

在我们学习记录之前，让我们看看 Multimethods，这是我们定义多态行为的第一个工具。

### Multimethods

_Multimethods_为你提供了一种直接的、灵活的方法，将多态性引入你的代码中。使用 Multimethods，你可以通过定义一个_调度函数_将一个名字与多个实现联系起来，该函数产生_调度值_，用来决定使用哪个_方法_。调度函数就像餐厅里的主人。主人会问你一些问题，比如 "你有预订吗？"和 "聚会人数？"，然后给你安排相应的座位。同样，当你调用一个 Multimethods 时，调度函数将询问参数，并将它们发送到正确的方法，正如这个例子所显示的。

```
(ns were-creatures)
➊ (defmulti full-moon-behavior (fn [were-creature] (:were-type were-creature)))
➋ (defmethod full-moon-behavior :wolf
  [were-creature]
  (str (:name were-creature) " will howl and murder"))
➌ (defmethod full-moon-behavior :simmons
  [were-creature]
  (str (:name were-creature) " will encourage people and sweat to the oldies"))

(full-moon-behavior {:were-type :wolf
➍                      :name "Rachel from next door"})
; => "Rachel from next door will howl and murder"

(full-moon-behavior {:name "Andy the baker"
➎                      :were-type :simmons})
; => "Andy the baker will encourage people and sweat to the oldies"
```

这个 Multimethods 显示了你如何定义不同种类的狼人生物的满月行为。大家都知道狼人变成了狼，到处嚎叫着杀人。一种不太知名的狼人，即狼-西蒙斯，变成理查德-西蒙斯，烫着头发，到处跑，鼓励人们做最好的自己，为老人们流汗。你不想被这两种生物咬到，否则你就会变成它们。

![](https://www.braveclojure.com/assets/images/cftbat/multimethods-records-protocols/weresimmons.png)

我们在➊处创建 Multimethods。这告诉 Clojure，"嘿，创建一个名为`full-moon-behavior'的新Multimethods。每当有人调用`full-moon-behavior`时，在参数上运行调度函数`(fn \[were-creature] (:were-type were-creature))\`。使用该函数的结果，也就是调度值，来决定使用哪个具体方法！"

接下来，我们定义了两个方法，一个是当调度函数返回的值是➋的`:wolf`时，另一个是当它是➌的`:simmons`时。方法定义看起来很像函数定义，但主要的区别是，方法名称后面紧跟着_dispatch 值_。 `:wolf`和`:simmons`都是_dispatch 值_。这与调度\*值不同，后者是调度函数的返回值。完整的调度序列是这样的。

1. 形式`(full-moon-behavior {:wer-type :wolf :name "Rachel from next door"})`被评估。
2. 运行`full-moon-behavior`的调度函数，返回`:wolf`作为调度值。
3. Clojure 将调度值`:wolf`与为`full-moon-behavior`定义的所有方法的调度值相比较。这些调度值是`:wolf`和`:simmons`。
4. 因为调度值`:wolf`等于调度值`:wolf`，所以`:wolf`的算法运行。

不要让术语把你绊倒! 主要的想法是，调度函数返回一些值，这个值被用来决定使用哪个方法定义。

回到我们的例子! 接下来我们调用该方法两次。在➍处，调度函数返回值":wolf"，并使用相应的方法，通知你 "隔壁的 Rachel 将嚎叫并杀人"。在➏，该函数的行为类似，只是`:simmons`是调度值。

你可以定义一个以`nil`为调度值的方法。

```
(defmethod full-moon-behavior nil
  [were-creature]
  (str (:name were-creature) " will stay at home and eat ice cream"))

(full-moon-behavior {:were-type nil
                     :name "Martin the nurse"})
; => "Martin the nurse will stay at home and eat ice cream"
```

当你这次调用`full-moon-behavior`时，你给它的参数`:wer-type`是`nil`，所以对应于`nil`的方法被评估，你被告知\`"护士 Martin 将呆在家里吃冰淇淋"。

你也可以通过指定`:default`作为调度值，定义一个默认方法，在没有其他方法匹配的情况下使用。在这个例子中，给出的参数的`:were-type`与之前定义的方法都不匹配，所以使用了默认方法。

```
(defmethod full-moon-behavior :default
  [were-creature]
  (str (:name were-creature) " will stay up all night fantasy footballing"))

(full-moon-behavior {:were-type :office-worker
                     :name "Jimmy from sales"})
; => "Jimmy from sales will stay up all night fantasy footballing"
```

Multimethods 的一个很酷的地方是，你可以随时添加新的方法。如果你发布了一个包括`wer-creatures`命名空间的库，其他人可以继续扩展 Multimethods 来处理新的派发值。这个例子显示，你创建了自己的随机命名空间并包括了`wer-creatures`命名空间，然后为`full-moon-behavior`Multimethods 定义了另一个方法。

```
(ns random-namespace
  (:require [were-creatures]))
(defmethod were-creatures/full-moon-behavior :bill-murray
  [were-creature]
  (str (:name were-creature) " will be the most likeable celebrity"))
(were-creatures/full-moon-behavior {:name "Laura the intern" 
                                    :were-type :bill-murray})
; => "Laura the intern will be the most likeable celebrity"
```

你的调度函数可以使用它的任何或所有参数返回任意的值。下一个例子定义了一个 Multimethods，它接收两个参数，并返回一个包含每个参数类型的向量。它还定义了该方法的一个实现，当每个参数都是字符串时，该方法将被调用。

```
(ns user)
(defmulti types (fn [x y] [(class x) (class y)]))
(defmethod types [java.lang.String java.lang.String]
  [x y]
  "Two strings!")

(types "String 1" "String 2")
; => "Two strings!"
```

顺便说一下，这就是为什么它们被称为_multi_methods：它们允许对多个参数进行调度。我没有经常使用这个功能，但我可以看到它被用于角色扮演游戏中，根据法师的主要魔法学校和他的魔法专长来编写方法。无论如何，有它而不需要它总比需要它而没有它好。

注意 Multimethods 也允许分层调度。Clojure 可以让你建立自定义的层次结构，我不会介绍这些，但你可以通过阅读[http://clojure.org/multimethods/](http://clojure.org/multimethods/) 的文档来了解它们。

### 协议

在大约 93.58%的情况下，你会希望根据参数的类型来调度方法。例如，`count`需要对向量使用不同的方法，而不是对 map 或 list 使用不同的方法。尽管可以用 Multimethods 进行类型调度，但_协议_是为类型调度而优化的。它们比 Multimethods 更有效，而且 Clojure 让你很容易简洁地指定协议的实现。

Multimethods 只是一个多态的操作，而协议是一个_集合_的一个或多个多态操作。协议操作被称为方法，就像 Multimethods 操作一样。与 Multimethods 不同的是，Multimethods 对调度函数返回的任意值进行调度，而协议方法是根据第一个参数的类型进行调度，如本例所示。

```
(ns data-psychology)
➊(defprotocol ➋Psychodynamics
  ➌"Plumb the inner depths of your data types"
  ➍(thoughts [x] "The data type's innermost thoughts")
  ➎(feelings-about [x] [x y] "Feelings about self or other"))
```

首先，在➊有`defprotocol`。这需要一个名字，`Psychodynamics` ➋，和一个可选的文件串，`"探究你的数据类型的内部深度"`➌。接下来是方法签名。一个_方法签名_由一个名称、一个参数说明和一个可选的文档串组成。第一个方法签名被命名为`thoughts`➍，只能接受一个参数。第二个名为`feelings-about`➎，可以接受一个或两个参数。协议有一个限制：方法不能有其余参数。所以像下面这样的行是不允许的。

```
(feels-about [x] [x & others])
```

通过定义一个协议，你在定义一个抽象，但你还没有定义如何实现这个抽象。这就像你为行为保留了名字（在这个例子中，你保留了`思想`和`感觉-关于`），但你还没有定义具体的行为。如果你要评估`(thoughts "blorb")`，你会得到一个异常，内容是："没有为 java.lang.String 类找到方法的实现：protocol: data-psychology/psychodynamics 的 thoughts。" 协议是根据第一个参数的类型分配的，所以当你调用`(thoughts "blorb")`时，Clojure 试图为字符串查找`thoughts`方法的实现，但失败了。

你可以通过_扩展_字符串数据类型来_实现_`Psychodynamics`协议来解决这一遗憾。

```
➊ (extend-type java.lang.String
➋   Psychodynamics
➌   (thoughts [x] (str x " thinks, 'Truly, the character defines the data type'")
➍   (feelings-about
    ([x] (str x " is longing for a simpler way of life"))
    ([x y] (str x " is envious of " y "'s simpler way of life"))))

(thoughts "blorb")
➎ ; => "blorb thinks, 'Truly, the character defines the data type'"

(feelings-about "schmorb")
; => "schmorb is longing for a simpler way of life"

(feelings-about "schmorb" 2)
; => "schmorb is envious of 2's simpler way of life"
```

`extend-type`后面是你想扩展的类或类型的名称和你想让它支持的协议--在这个例子中，你在➊处指定了类`java.lang.String`和你想让它支持的协议`Psychodynamics`，在➋。之后，你在➌为 "thoughts "方法和➍为 "feelings-about "方法提供一个实现。如果你要扩展一个类型来实现一个协议，你必须实现协议中的每一个方法，否则 Clojure 会抛出一个异常。在这种情况下，你不能只实现`思想'或只实现`感觉'；你必须同时实现这两种方法。

注意，这些方法的实现不像 Multimethods 那样以`defmethod`开头。事实上，它们看起来类似于函数定义，只是没有`defn'。要定义一个方法的实现，你要写一个以方法名称开头的表格，像`thoughts'，然后提供一个参数向量和方法的主体。这些方法也允许重载，就像函数一样，你定义多重性的方法实现与多重性的函数类似。你可以在➍的 "feelings-about "实现中看到这一点。

在你扩展了`java.lang.String`类型以实现`Psychodynamics`协议后，Clojure 知道如何调度调用`(thoughts "blorb")`，你会在➎得到字符串\`"blorb thinks, 'Truly, the character defines the data type'"。

如果你想提供一个默认的实现，就像你对 multimethods 所做的那样呢？要做到这一点，你可以扩展`java.lang.Object`。这样做是因为 Java（也就是 Clojure）中的每个类型都是`java.lang.Object`的后代。如果这不是很有意义（也许是因为你不熟悉面向对象的编程），不要担心，只要知道它是有效的。下面是你如何使用这个技术为`Psychodynamics`协议提供一个默认实现。

```
(extend-type java.lang.Object
  Psychodynamics
  (thoughts [x] "Maybe the Internet is just a vector for toxoplasmosis")
  (feelings-about
    ([x] "meh")
    ([x y] (str "meh about " y))))

(thoughts 3)
; => "Maybe the Internet is just a vector for toxoplasmosis"

(feelings-about 3)
; => "meh"

(feelings-about 3 "blorb")
; => "meh about blorb"
```

因为我们还没有为数字定义一个`心理动力学'的实现，Clojure将对`思想'和`感觉-关于'的调用分派给为`java.lang.Object'定义的实现。

你可以使用`extend-protocol'来代替多次调用`extend-type'来扩展多个类型，它可以让你一次为多个类型定义协议实现。下面是你如何定义前面的协议实现。

```
(extend-protocol Psychodynamics
  java.lang.String
  (thoughts [x] "Truly, the character defines the data type")
  (feelings-about
    ([x] "longing for a simpler way of life")
    ([x y] (str "envious of " y "'s simpler way of life")))

  java.lang.Object
  (thoughts [x] "Maybe the Internet is just a vector for toxoplasmosis")
  (feelings-about
    ([x] "meh")
    ([x y] (str "meh about " y))))
```

你可能会发现这个技术比使用`extend-type`更方便。然后，你也可能不觉得。`extend-type`让你感觉如何？`extend-protocol`怎么样？来坐在这个沙发上，告诉我这一切。

值得注意的是，一个协议的方法 "属于 "它们所定义的命名空间。在这些例子中，"心理动力学 "方法的完全限定名称是 "数据-心理学/想法 "和 "数据-心理学/感觉-关于"。如果你有面向对象的背景，这可能看起来很奇怪，因为方法属于 OOP 中的数据类型。但不要吓坏了! 这只是 Clojure 赋予抽象优先权的另一种方式。这个事实的一个后果是，如果你想让两个不同的协议包括具有相同名称的方法，你需要把协议放在不同的命名空间中。

## 记录

Clojure 允许你创建_records_，它是自定义的、类似 Map 的数据类型。它们类似于 Map，因为它们将键和值联系起来，你可以像使用 Map 一样查询它们的值，而且它们像 Map 一样是不可改变的。它们的不同之处在于，你为记录指定\*字段。字段是数据的槽；使用它们就像指定一个数据结构应该有哪些键。记录也与 Map 不同，你可以扩展它们来实现协议。

要创建一个记录，你可以使用`defrecord`来指定它的名字和字段。

```
(ns were-records)
(defrecord WereWolf [name title])
```

这个记录的名字是`WereWolf`，它的两个字段是`name`和`title`。你可以通过三种方式创建这个记录的实例。

```
➊ (WereWolf. "David" "London Tourist")
; => #were_records.WereWolf{:name "David", :title "London Tourist"}.

➋ (->WereWolf "Jacob" "Lead Shirt Discarder")
; => #were_records.WereWolf{:name "Jacob", :title "Lead Shirt Discarder"}。

➌ (map->WereWolf {:name "Lucian" :title "CEO of Melodrama"})
; => #were_records.WereWolf{:name "Lucian", :title "CEO of Melodrama"}.
```

在➊，我们以创建 Java 对象的方式创建一个实例，使用类实例化的互操作调用。(_Interop_指的是在 Clojure 中与本地 Java 结构交互的能力)。请注意，参数必须遵循与字段定义相同的顺序。这样做的原因是，记录实际上是被掩盖的 Java 类。

➋的实例看起来与➊的实例几乎相同，但关键的区别在于`->WereWolf`是一个函数。当你创建一条记录时，工厂函数`->`RecordName 和`map->`RecordName 会自动创建。在➌，`map->WereWolf`接收一个 map 作为参数，其关键字与记录类型的字段相对应，并返回一个记录。

如果你想使用其他命名空间的记录类型，你必须导入它，就像你在第 12 章中对 Java 类所做的那样。请注意将命名空间中的所有破折号替换为下划线。这个简单的例子显示了如何在另一个命名空间导入`WereWolf`记录类型。

```
(ns monster-mash
  (:import [were_records WereWolf])
(WereWolf. "David" "London Tourist")
; => #were_records.WereWolf{:name "David", :title "London Tourist"}
```

注意，`were_records`有一个下划线，而不是破折号。

你可以用查询 Map 值的方式查询记录值，也可以使用 Java 字段访问互操作。

```
(def jacob (->WereWolf "Jacob" "Lead Shirt Discarder")
➊ (.name jacob) 
; => "Jacob"

➋ (:name jacob) 
; => "Jacob"

➌ (get jacob :name) 
;=> "Jacob"
```

第一个例子`(.name jacob)`在➊，使用了 Java 互操作，➋和➌的例子访问`:name`的方式与使用 map 相同。

当测试相等时，Clojure 将检查所有字段是否相等，以及两个比较体是否具有相同的类型。

```
➊ (= jacob (->WereWolf "Jacob" "Lead Shirt Discarder"))
; => true

➋ (= jacob (WereWolf. "David" "London Tourist"))
; => false

➌ (= jacob {:name "Jacob" :title "Lead Shirt Discarder"})
; => false
```

➊处的测试返回`true`，因为`jacob'和新创建的记录是同一类型，并且它们的字段是相等的。➋处的测试返回 "false"，因为字段不相等。最后在➌处的测试返回 "false"，因为两个比较对象的类型不一样。`jacob'是一个\`WereWolf'记录，而另一个参数是一个 Map。

任何你能在 Map 上使用的函数，你也能在记录上使用。

```
(assoc jacob :title "Lead Third Wheel")
; => #were_records.WereWolf{:name "Jacob", :title "Lead Third Wheel"}。
```

然而，如果你`dissoc`一个字段，结果的类型将是一个普通的'Clojure map；它将不会有与原始记录相同的数据类型。

```
(dissoc jacob :title)
; => {:name "Jacob"} <- that's not a were_records.WereWolf
```

这至少有两个原因：第一，访问 Map 值比访问记录值要慢，所以如果你要建立一个高性能的程序，就要注意了。第二，当你创建一个新的记录类型时，你可以扩展它来实现一个协议，类似于你之前使用`extend-type`扩展一个类型。如果你`dissoc`一个记录，然后试图在结果上调用一个协议方法，记录的协议方法就不会被调用。

下面是你在定义记录时如何扩展一个协议。

```
➊ (defprotocol WereCreature
➋   (full-moon-behavior [x]))

➌ (defrecord WereWolf [name title]
  WereCreature
  (full-moon-behavior [x]
    (str name " will howl and murder")))

(full-moon-behavior (map->WereWolf {:name "Lucian" :title "CEO of Melodrama"}))
; => "Lucian will howl and murder"
```

我们创建了一个新的协议，`WereCreature` ➊，有一个方法，`full-moon-behavior` ➋。在➌，`defrecord`为`WereWolf`实现了`WereCreature`。在`full-moon-behavior`实现中最有趣的部分是你可以访问`name`。你还可以访问`title'和任何其他可能为你的记录定义的字段。你也可以使用`extend-type`和`extend-protocol\`来扩展记录。

你什么时候应该使用记录，什么时候应该使用 Map？一般来说，如果你发现自己在创建 Map 时反复使用相同的字段，你应该考虑使用记录。这告诉你，这组数据代表了你的应用程序领域的信息，如果你提供一个基于你试图建模的概念的名称，你的代码将更好地传达其目的。不仅如此，记录访问比 Map 访问更有表现力，所以你的程序会变得更有效率一些。最后，如果你想使用协议，你就需要创建一个记录。

## 进一步研究

Clojure 提供了其他的工具来处理抽象和数据类型。这些工具，我认为是高级的，包括`deftype`，`reify`，和`proxy`。如果你有兴趣了解更多，请查看\*[http://clojure.org/datatypes/](http://clojure.org/reference/datatypes/)\*上关于数据类型的文档。

## 总结

Clojure 的设计原则之一就是要写到抽象。在本章中，你学到了如何使用 Multimethods 和原型来定义你自己的抽象概念。这些结构提供了多态性，允许同一个操作根据它的参数有不同的表现。你还学会了如何用`defrecord`创建和使用自己的关联数据类型，以及如何扩展记录来实现协议。

当我刚开始学习 Clojure 时，我对使用 Multimethods、协议和记录感到很害羞。然而，它们在 Clojure 库中经常被使用，所以了解它们的工作原理是很好的。一旦你掌握了它们，它们会帮助你写出更干净的代码。

## 练习

1. 扩展`full-moon-behavior`Multimethods，为你自己的 were-creature 类型添加行为。
2. 创建一个`WereSimmons`记录类型，然后扩展`WereCreature`协议。
3. 创建你自己的协议，然后使用`extend-type`和`extend-protocol`来扩展它。
4. 创建一个角色扮演游戏，使用多重调度来实现行为。
