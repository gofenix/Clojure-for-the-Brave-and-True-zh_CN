# Clojure Metaphysics: 原子、Refs、Vars 和拥抱僵尸

三个并发性的小妖精都是从同一个邪恶的坑里生出来的：对可变状态的共享访问。你可以在第九章的引用单元讨论中看到这一点。当两个线程对引用单元进行不协调的更改时，结果是不可预测的。

Rich Hickey 设计 Clojure 是为了专门解决共享访问易变状态所产生的问题。事实上，Clojure 体现了一种非常清晰的状态概念，使其在本质上比大多数流行的编程语言更安全。它是安全的，一直到它的_meta-freakin-physics_。

在本章中，你将了解 Clojure 的底层形而上学，与典型的面向对象（OO）语言的形而上学相比较。学习这种哲学将使你准备好处理 Clojure 剩下的并发工具，_atom_、_ref_和_var_引用类型。(Clojure 还有一个额外的引用类型，_agents_，本书没有涉及。) 这些类型中的每一个都能让你安全地同时执行状态修改操作。你还会学到一些简单的方法，使你的程序更有效率，而不需要引入状态。

形而上学试图用最广泛的术语来回答两个基本问题。

* 那里有什么？
* 它是什么样子的？

为了引出 Clojure 和 OO 语言之间的差异，我将解释两种不同的拥抱僵尸的建模方式。与普通的僵尸不同，拥抱僵尸并不想要吞噬你的大脑。它只想用勺子舀你，也许还想闻闻你的脖子。这使得它的不死、摇晃、腐烂的状态更加悲惨。你怎么能试图杀死只想要爱的东西呢？谁是这里真正的怪物？

## 面向对象的形而上学

OO 形而上学将拥抱僵尸视为存在于世界上的一个对象。这个对象的属性可能会随着时间的推移而改变，但它仍然被当作一个单一的、不变的对象。如果这看起来是一个完全明显的、没有争议的僵尸形而上学的方法，那么你可能没有在哲学入门课上花几个小时来争论一把椅子的存在意味着什么，以及什么真正使它首先成为一把椅子。

棘手的部分是，拥抱的僵尸总是在变化。它的身体慢慢恶化。随着时间的推移，它对拥抱的不灭渴望越来越强烈。在 OO 术语中，我们会说拥抱僵尸是一个具有可改变状态的对象，它的状态是不断波动的。但是不管这个僵尸有多大的变化，我们仍然把它认定为同一个僵尸。下面是你如何在 Ruby 中对抱团僵尸进行建模和交互。

```
class CuddleZombie
  # attr_accessor is just a shorthand way for creating getters and
  # setters for the listed instance variables
  attr_accessor :cuddle_hunger_level, :percent_deteriorated

  def initialize(cuddle_hunger_level = 1, percent_deteriorated = 0)
    self.cuddle_hunger_level = cuddle_hunger_level
    self.percent_deteriorated = percent_deteriorated
  end
end

fred = CuddleZombie.new(2, 3)
fred.cuddle_hunger_level  # => 2
fred.percent_deteriorated # => 3

fred.cuddle_hunger_level = 3
fred.cuddle_hunger_level # => 3
```

1. 10-1. 用 Ruby 建立抱团僵尸行为模型

在这个例子中，你创建了一个抱团僵尸，`fred`，有两个属性。`cuddle_hunger_level`和`percent_deteriorated`。`fred`一开始的`cuddle_hunger_level`是 2，但是你可以把它改成任何你想要的东西，它仍然是好的'Fred，同一个拥抱僵尸。在这种情况下，你把它的\`cuddle\_hunger\_level'改为 3。

![](https://www.braveclojure.com/assets/images/cftbat/zombie-metaphysics/cuddle-zombie.png)

你可以看到，这个对象只是一个花哨的引用单元。在多线程环境下，它也会受到同样的非确定性结果的影响。例如，如果两个线程试图用`fred.cuddle_hunger_level = fred.cuddle_hunger_level + 1`这样的方式来增加 Fred 的饥饿度，其中一个增量可能会丢失，就像《三个小妖精》中两个线程向`X`写入的例子一样。参考单元格、相互排斥和矮人狂战士 "中的例子。

即使你只在一个单独的线程上进行读取，程序仍将是非确定性的。例如，假设你正在进行关于抱团僵尸行为的研究。你想记录一个僵尸的饥饿程度，只要它达到 50%的恶化程度，但你想在另一个线程上进行，以提高性能，使用类似清单 10-1 中的代码。

```
if fred.percent_deteriorated >= 50
  Thread.new { database_logger.log(fred.cuddle_hunger_level) }
end
```

1. 这段 Ruby 代码在并发执行时并不安全。

问题是，另一个线程可能在实际写入之前改变`fred`。

例如，图 10-1 显示了两个从上到下执行的线程。在这种情况下，将 5 写入数据库是正确的，但 10 却被写入了。

![](https://www.braveclojure.com/assets/images/cftbat/zombie-metaphysics/fred-read.png)

图 10-1：记录不一致的抱团僵尸数据

这将是很不幸的。当你试图从拥抱僵尸的启示中恢复时，你不希望你的数据是不一致的。然而，没有办法保留一个对象在某一特定时刻的状态。

此外，为了同时改变`cuddle_hunger_level`和`percent_deteriorated`，你必须特别小心。否则，`fred`有可能被视为不一致的状态，因为另一个线程可能会在你打算同时进行的两个变化之间`读取`fred\`对象，像这样。

```
fred.cuddle_hunger_level = fred.cuddle_hunger_level + 1
# At this time, another thread could read fred's attributes and
# "perceive" fred in an inconsistent state unless you use a mutex
fred.percent_deteriorated = fred.percent_deteriorated + 1
```

这是另一个版本的互斥问题。在面向对象编程（OOP）中，你可以用_mutex_来手动解决这个问题，它可以确保在 mutex 的持续时间内，每次只有一个线程可以访问一个资源（在本例中，就是`fred`对象）。

对象永远不稳定的事实并不妨碍我们把它们当作程序的基本构件。事实上，这被认为是 OOP 的一个优势。状态如何变化并不重要；你仍然可以与一个稳定的接口进行交互，一切都会正常工作。这符合我们对世界的直观感觉。一块蜡仍然是同一块蜡，即使它的属性发生了变化：如果我改变了它的颜色，融化了它，然后把它倒在我的敌人的脸上，我仍然会认为它是我开始时的那个蜡对象。

另外，在 OOP 中，对象也会做事。它们相互作用，在程序运行时改变状态。同样，这也符合我们对世界的直观感觉：变化是对象相互作用的结果。一个人的对象推到一个门的对象上，进入一个房子的对象。

## Clojure 形而上学

在 Clojure 形而上学中，我们会说，我们永远不会遇到两次相同的拥抱僵尸。拥抱僵尸并不是一个独立于其变异而存在于世界上的离散事物：它实际上是一连串的_价值_。

术语_值_经常被 Clojurists 使用，其具体含义可能与你的习惯不同。价值是\*原子性的，即它们在一个更大的系统中形成一个单一的不可还原的单位或组成部分；它们是不可分割的、不变的、稳定的实体。数字是价值：数字 15 变异为另一个数字是没有意义的。当你从 15 加减时，你并没有改变 15 这个数字；你只是得到了一个不同的数字。Clojure 的数据结构也是价值，因为它们是不可改变的。当你在一个 Map 上使用`assoc`时，你不会修改原来的 Map；相反，你会派生出一个新的 Map。

所以一个值不会改变，但是你可以对一个值应用一个_过程来产生一个新的值。例如，假设我们从一个值_F1_开始，然后我们把_拥抱僵尸_过程应用到_F1\*，产生值_F2_。然后这个过程又被应用到_F2_的值上，产生_F3_的值，以此类推。

这导致了对_身份_的不同概念。Clojure 形而上学不是像 OO 形而上学那样把身份理解为变化的对象所固有的，而是把身份理解为我们人类强加给由一个过程随时间产生的一连串不变的值的东西。我们使用_名字_来指定身份。名字_Fred_是指一系列单独的状态_F1_、_F2_、_F3_等等的方便方法。从这个角度来看，不存在所谓的可改变的状态。相反，_state_指的是某个时间点上的身份值。

Rich Hickey 用电话号码的比喻来解释状态。 _Alan 的电话号码_已经改变了 10 次，但我们将永远用同一个名字来称呼这些号码，即_Alan 的电话号码_。艾伦五年前的电话号码与今天的电话号码是不同的数值，两者是艾伦电话号码身份的两种状态。

当你考虑到在你的程序中你是在处理关于世界的信息时，这是有意义的。与其说信息发生了变化，不如说你收到了新的信息。周五中午 12 点，"抱抱僵尸 "弗雷德处于 50%的腐烂状态。在下午 1 点，他是 60%的腐烂。这都是你可以处理的事实，引入一个新的事实并不会使以前的事实失效。即使弗雷德的衰变率从 50%增加到 60%，但在下午 12:00 时他处于 50%的衰变状态仍然是事实。

图 10-2 显示了你可以如何将价值、过程、身份和状态可视化。

![](https://www.braveclojure.com/assets/images/cftbat/zombie-metaphysics/fp-metaphysics.png)

图 10-2：价值、过程、身份和状态

这些价值不会相互作用，也不能被改变。它们不能\*做任何事情。只有在以下情况下才会发生变化：a）一个过程产生了一个新的值；b）我们选择将身份与新的值联系起来。

为了处理这种变化，Clojure 使用_参考类型_。参考类型让你在 Clojure 中管理身份。使用它们，你可以命名一个身份并检索其状态。让我们来看看其中最简单的，_原子_。

## 原子

Clojure 的原子引用类型允许你赋予一连串的相关值以身份。下面是你如何创建一个原子。

```
(def fred (atom {:cuddle-hunger-level 0
                 :percent-deteriorated 0}))
```

这将创建一个新的原子，并将其与名称`fred`绑定。这个原子\*引用了`{:cuddle-hunger-level 0 :percent-deteriorated 0}`的值，你可以说这是它的当前状态。

要得到一个原子的当前状态，你要解除对它的引用。下面是 Fred 的当前状态。

```
@fred
; => {:cuddle-hunger-level 0, :percent-deteriorated 0}
```

与期货、延迟和承诺不同，解除对原子（或任何其他引用类型）的引用将永远不会阻塞。当你解除对期货、延迟和承诺的引用时，就像你在说 "我现在需要一个值，我会一直等到我得到它"，所以这个操作会阻塞是合理的。然而，当你解除引用类型的引用时，就像你在说 "给我我现在引用的值"，所以操作不会阻塞是有道理的，因为它不需要等待任何东西。

在清单 10-1 中的 Ruby 例子中，我们看到当你试图在一个单独的线程上记录数据时，对象数据可能会发生变化。当使用原子来管理状态时就不会发生这种危险，因为每个状态都是不可改变的。下面是你如何用`println`来记录一个僵尸的状态。

```
(let [zombie-state @fred]
  (if (>= (:percent-deteriorated zombie-state) 50)
    (future (println (:cuddle-hunger-level zombie-state)))))
```

清单 10-1 中的 Ruby 例子的问题是，它需要两步来读取僵尸的两个属性，而其他线程可能在这两步之间改变这些属性。然而，通过使用原子来引用不可变的数据结构，你只需要执行一次读取，并且返回的数据结构不会被其他线程改变。

要更新原子，使其指向一个新的状态，你可以使用`swap!`。这似乎是矛盾的，因为我说过，原子值是不变的。的确，它们是不变的。但是现在我们正在使用原子的_参考类型_，一个指向原子值的结构。原子值不会改变，但是引用类型可以被更新并被分配一个新的值。

`swap!`接收一个原子和一个函数作为参数。它将函数应用于原子的当前状态以产生一个新的值，然后它更新原子以引用这个新的值。新的值也被返回。下面是你如何将 Fred 的拥抱饥饿度提高 1。

```
(swap! fred
       (fn [current-state]
         (merge-with + current-state {:cuddle-hunger-level 1})))
; => {:cuddle-hunger-level 1, :percent-deteriorated 0}
```

取消引用`fred`将返回新的状态。

```
@fred
; => {:cuddle-hunger-level 1, :percent-deteriorated 0}
```

与 Ruby 不同，`fred`不可能处于不一致的状态，因为你可以同时更新饥饿度和恶化百分比，像这样。

```
(swap! fred
       (fn [current-state]
         (merge-with + current-state {:cuddle-hunger-level 1
                                      :percent-deteriorated 1})))
; => {:cuddle-hunger-level 2, :percent-deteriorated 1}
```

这段代码传递给`swap!`一个只需要一个参数的函数，`current-state`。你也可以传递`swap!`一个需要多个参数的函数。例如，你可以创建一个需要两个参数的函数，一个是僵尸状态，另一个是增加其拥抱饥饿度的数量。

```
(defn increase-cuddle-hunger-level
  [zombie-state increase-by]
  (merge-with + zombie-state {:cuddle-hunger-level increase-by}))
```

让我们在僵尸状态下快速测试一下`increase-cuddle-hunger-level`。

```
(increase-cuddle-hunger-level @fred 10)
; => {:cuddle-hunger-level 12, :percent-deteriorated 1}
```

注意，这段代码实际上并没有更新`fred`，因为我们没有使用`swap!`，我们只是对`increase-cuddle-`hunger`-level`做了一个正常的函数调用，它返回一个结果。

现在用附加参数调用`swap!`，`@fred`将被更新，就像这样。

```
(swap! fred increase-cuddle-hunger-level 10)
; => {:cuddle-hunger-level 12, :percent-deteriorated 1}

@fred
; => {:cuddle-hunger-level 12, :percent-deteriorated 1}
```

或者你可以用 Clojure 的内置函数来表达整个事情。`update-in`函数需要三个参数：一个集合，一个用于识别要更新的值的 Vector，以及一个更新该值的函数。它还可以接受额外的参数，这些参数将被传递给更新函数。下面是几个例子。

```
(update-in {:a {:b 3}} [:a :b] inc)
; => {:a {:b 4}}

(update-in {:a {:b 3}} [:a :b] + 10)
; => {:a {:b 13}}
```

在第一个例子中，你正在更新 Map`{:a {:b 3}}。Clojure使用Vector`\[:a :b]`来遍历嵌套图；`:a`产生嵌套图`{:b 3}`，`:b`产生值`3`。Clojure将`inc`函数应用于`3`，并返回一个替换了`3\`的新 Map。

下面是你如何使用`update-in`函数来改变 Fred 的状态。

```
(swap! fred update-in [:cuddle-hunger-level] + 10)
; => {:cuddle-hunger-level 22, : percent-deteriorated 1}.
```

通过使用原子，你可以保留过去的状态。你可以解除引用一个原子来检索状态 1，然后更新该原子，创建状态 2，并仍然使用状态 1。

```
(let [num (atom 1)
      s1 @num]
  (swap! num inc)
  (println "State 1:" s1)
  (println "Current state:" @num))
; => State 1: 1
; => Current state: 2
```

这段代码创建了一个名为 "num "的原子，检索其状态，更新其状态，然后打印其过去的状态和当前的状态，表明当我说你可以保留过去的状态时，我并不是要欺骗你，因此你可以信任我所有的东西--包括你的真实姓名，我保证只说出你的真实姓名，以拯救你脱离致命的危险。

这一切都很有趣，但如果两个独立的线程调用"（交换！弗雷德增加-拥抱-饥饿等级 1）"会发生什么？是否有可能像清单 10-1 中的 Ruby 例子那样，其中一个增量被丢失？

答案是否定的! `swap!`实现了_比较和设置_的语义，意味着它在内部做了以下工作。

1. 它读取原子的当前状态。
2. 然后将更新函数应用于该状态。
3. 接下来，它检查它在步骤 1 中读取的值是否与原子的当前值相同。
4. 如果是，那么`swap!`就更新原子以引用步骤 2 的结果。
5. 如果不是，那么`swap!`重试，从第 1 步开始再次经历这个过程。

这个过程保证了没有交换会丢失。

关于`swap!`需要注意的一个细节是，原子更新是同步发生的；它们将阻塞其线程。例如，如果你的更新函数由于某种原因调用了`Thread/sleep 1000`，那么当`swap!`完成时，线程将阻塞至少一秒钟。

有时你会想更新一个原子而不检查它的当前值。例如，你可能会开发一种血清，将一个抱枕僵尸的饥饿度和恶化度设置为零。对于这些情况，你可以使用`reset!`函数。

```
(reset! fred {:cuddle-hunger-level 0
              :percent-deteriorated 0})
```

这就涵盖了 atoms 的所有核心功能! 总结一下：原子实现了 Clojure 的状态概念。它们允许你为一系列不可变的值赋予一个身份。它们通过比较和设置语义为引用单元和互斥问题提供了解决方案。它们还允许你处理过去的状态，而不用担心它们会在原地变异。

除了这些核心特性外，原子还与其他引用类型共享两个特性。你可以在原子上附加_watches_和_validators_。现在让我们来看看这些。

## 手表和验证器

观察器允许你超级猥琐地检查你的参考类型的一举一动。验证器允许你有超强的控制力，限制哪些状态是可以允许的。钟表和验证器都是普通的函数。

### 手表

一个_watch_是一个函数，它需要四个参数：一个键，被监视的引用，它的前一个状态，以及它的新状态。你可以为一个引用类型注册任意数量的手表。

比方说，一个僵尸的洗牌速度（以每小时洗牌次数衡量，或称 SPH）取决于其饥饿程度和恶化程度。下面是你的计算方法，用拥抱的饥饿程度乘以它的完整程度。

```
(defn shuffle-speed
  [zombie]
  (* (:cuddle-hunger-level zombie)
     (- 100 (:percent-deteriorated zombie))))
```

我们还可以说，每当僵尸的洗牌速度达到 5000SPH 的危险水平时，你都想得到提醒。否则，你想被告知一切都很好。下面是一个观察函数，你可以用来在 SPH 超过 5000 时打印一个警告信息，否则打印一个一切正常的信息。

```
(defn shuffle-alert
  [key watched old-state new-state]
  (let [sph (shuffle-speed new-state)]
    (if (> sph 5000)
      (do
        (println "Run, you fool!")
        (println "The zombie's SPH is now " sph)
        (println "This message brought to your courtesy of " key))
      (do
        (println "All's well with " key)
        (println "Cuddle hunger: " (:cuddle-hunger-level new-state))
        (println "Percent deteriorated: " (:percent-deteriorated new-state))
        (println "SPH: " sph)))))
```

观察函数有四个参数：一个可以用来报告的键，被观察的原子，原子更新前的状态，以及原子更新后的状态。这个观察函数计算新状态的洗牌速度，如果它过高，就打印一个警告信息，当洗牌速度安全时，就打印一个一切正常的信息，如上所述。在这两组信息中，`key`被用来让你知道信息的来源。

你可以用`add-watch`把这个函数附加到`fred`上。`add-watch`的一般形式是`（add-watch` ref key watch-fn`）`。在这个例子中，我们要重置`fred`的状态，添加`shuffle-alert`的观察函数，然后多次更新`fred`的状态以触发`shuffle-alert`。

```
(reset! fred {:cuddle-hunger-level 22
              :percent-deteriorated 2})
(add-watch fred :fred-shuffle-alert shuffle-alert)
(swap! fred update-in [:percent-deteriorated] + 1)
; => All's well with  :fred-shuffle-alert
; => Cuddle hunger:  22
; => Percent deteriorated:  3
; => SPH:  2134

(swap! fred update-in [:cuddle-hunger-level] + 30)
; => Run, you fool!
; => The zombie's SPH is now 5044
; => This message brought to your courtesy of :fred-shuffle-alert
```

这个观察函数的例子没有使用`watched`或`old-state`，但如果有需要，它们就在那里。现在我们来谈谈验证器。

### 验证器

_验证器_可以让你指定一个引用可以有哪些状态。例如，这里有一个验证器，你可以用来确保一个僵尸的`:%-deteriorated`在 0 到 100 之间。

```
(defn percent-deteriorated-validator
  [{:keys [percent-deteriorated]}]
  (and (>= percent-deteriorated 0)
       (<= percent-deteriorated 100)))
```

正如你所看到的，验证器只需要一个参数。当你给一个引用添加验证器时，该引用被修改，这样，每当它被更新时，它将调用这个验证器，并将更新函数返回的值作为其参数。如果验证器因返回 "false "或抛出一个异常而失败，引用将不会改变以指向新的值。

你可以在创建原子时附加一个验证器。

```
(def bobby
  (atom
   {:cuddle-hunger-level 0 :percent-deteriorated 0}
    :validator percent-deteriorated-validator))
(swap! bobby update-in [:percent-deteriorated] + 200)
; This throws "Invalid reference state"
```

在这个例子中，`percent-deteriorated-validator`返回`false`，原子更新失败。

你可以抛出一个异常，以获得一个更具描述性的错误信息。

```
(defn percent-deteriorated-validator
  [{:keys [percent-deteriorated]}]
  (or (and (>= percent-deteriorated 0)
           (<= percent-deteriorated 100))
      (throw (IllegalStateException. "That's not mathy!"))))
(def bobby
  (atom
   {:cuddle-hunger-level 0 :percent-deteriorated 0}
    :validator percent-deteriorated-validator))
(swap! bobby update-in [:percent-deteriorated] + 200)
; This throws "IllegalStateException: That's not mathy!"
```

相当不错! 现在让我们来看看裁判。

![](https://www.braveclojure.com/assets/images/cftbat/zombie-metaphysics/sock-gnome.png)

原子是管理独立身份状态的理想选择。但有时，我们需要表达一个事件应该同时更新一个以上的身份的状态。 _Refs_是这种情况下的完美工具。

一个典型的例子是记录 sock gnome 交易。我们都知道，袜子侏儒从世界各地的每一个干衣机中取出一只袜子。他们用这些袜子来孵化他们的孩子。作为对这种\*"\*礼物 "的回报，袜子地精保护你的家不被 El Chupacabra 入侵。如果你最近没有被 El Chupacabra 拜访，你要感谢袜子侏儒。

为了建立袜子转移的模型，我们需要表达的是，一个烘干机失去了一只袜子，一个地精同时得到了一只袜子。这一刻，袜子属于烘干机；下一刻，它属于地精。这只袜子不应该同时属于烘干机和侏儒，也不应该同时属于这两个人。

### 为袜子转移建模

你可以用 refs 来模拟这个 sock 传输。Refs 允许你使用事务语义来更新多个身份的状态。这些交易有三个特点。

* 它们是_原子性的_，意味着所有的参考文献都被更新，或者都不被更新。
* 它们是_一致的_，这意味着引用总是显示为有效的状态。一个 sock 总是属于一个 dryer 或一个 gnome，但绝不是两者都属于。
* 它们是_隔离的_，这意味着事务的行为就像它们是连续执行的一样；如果两个线程同时运行改变同一参考信息的事务，一个事务将重试。这类似于原子的比较和设置语义。

你可能认识到这些是数据库事务的 ACID 属性中的_A_、_C_和_I_。你可以认为 Refs 给你提供了与数据库事务相同的并发安全性，只是在内存中的数据。

Clojure 使用\*软件事务性内存（STM）\*来实现这种行为。STM 非常酷，但当你开始使用 Clojure 时，你不需要对它了解太多；你只需要知道如何使用它，这就是本节要告诉你的。

让我们开始转移一些袜子吧! 首先，你需要编码一些袜子和 gnome 的创建技术。下面的代码定义了一些袜子品种，然后定义了几个辅助函数。 `sock-count'将被用来帮助记录每一种袜子有多少只属于地精或烘干机，而`generate-sock-gnome'将创建一个新的、没有袜子的地精。

```
(def sock-varieties
  #{"darned" "argyle" "wool" "horsehair" "mulleted"
    "passive-aggressive" "striped" "polka-dotted"
    "athletic" "business" "power" "invisible" "gollumed"})

(defn sock-count
  [sock-variety count]
  {:variety sock-variety
   :count count})

(defn generate-sock-gnome
  "Create an initial sock gnome state with no socks"
  [name]
  {:name name
   :socks #{}})
```

现在你可以创建你的实际参照物了。侏儒将有 0 只袜子。另一方面，烘干机将有一组由袜子品种集生成的袜子对。下面是我们的参考文献。

```
(def sock-gnome (ref (generate-sock-gnome "Barumpharumph")))
(def dryer (ref {:name "LG 1337"
                 :socks (set (map #(sock-count % 2) sock-varieties))}))
```

你可以像解除对原子的引用一样解除对 ref 的引用。在这个例子中，你的袜子的顺序可能会不同，因为我们使用的是一个无序的集合。

```
(:socks @dryer)
; => #{{:variety "passive-aggressive", :count 2} {:variety "power", :count 2}
       {:variety "athletic", :count 2} {:variety "business", :count 2}
       {:variety "argyle", :count 2} {:variety "horsehair", :count 2}
       {:variety "gollumed", :count 2} {:variety "darned", :count 2}
       {:variety "polka-dotted", :count 2} {:variety "wool", :count 2}
       {:variety "mulleted", :count 2} {:variety "striped", :count 2}
       {:variety "invisible", :count 2}}
```

现在一切都准备好了，可以进行转移了。我们要修改`sock-gnome`参数，以显示它获得了一只袜子，并修改`dryer`参数，以显示它失去了一只袜子。你用`alter'来修改引用，而且你必须在一个事务中使用`alter'。 `dosync`启动一个事务并定义其范围；你把所有的事务操作放在其主体中。这里我们使用这些工具来定义一个\`steal-sock'函数，然后在我们的两个参考文件上调用它。

```
(defn steal-sock
  [gnome dryer]
  (dosync
   (when-let [pair (some #(if (= (:count %) 2) %) (:socks @dryer))]
     (let [updated-count (sock-count (:variety pair) 1)]
       (alter gnome update-in [:socks] conj updated-count)
       (alter dryer update-in [:socks] disj pair)
       (alter dryer update-in [:socks] conj updated-count)))))
(steal-sock sock-gnome dryer)

(:socks @sock-gnome)
; => #{{:variety "passive-aggressive", :count 1}}
```

现在团子有一只被动攻击型的袜子，而烘干机少了一只（你的团子可能偷了一只不同的袜子，因为袜子是以无序的方式存储的）。让我们确保所有被动攻击的袜子都被计算在内。

```
(defn similar-socks
  [target-sock sock-set]
  (filter #(= (:variety %) (:variety target-sock)) sock-set))

(similar-socks (first (:socks @sock-gnome)) (:socks @dryer))
; => ({:variety "passive-aggressive", :count 1})
```

这里有几个细节需要注意：当你`改变`一个引用时，这个改变在当前事务之外并不立即可见。这使得你可以在一个事务中对`dryer`调用`alter`两次，而不用担心`dryer`会在不一致的状态下被读取。同样的，如果你`改变`一个引用，然后在同一个事务中`deref`它，`deref`将返回新的状态。

这里有一个例子来证明这个交易中状态的想法。

```
(def counter (ref 0))
(future
  (dosync
   (alter counter inc)
   (println @counter)
   (Thread/sleep 500)
   (alter counter inc)
   (println @counter)))
(Thread/sleep 250)
(println @counter)
```

这将依次打印出 1、0 和 2。首先，你创建了一个引用，`counter`，用来保存数字 0。然后你用`future`创建一个新的线程来运行一个事务。在事务线程中，你增加计数器并打印它，然后数字 1 被打印出来。同时，主线程等待了 250 毫秒，也打印了计数器的值。然而，主线程上的计数器的值仍然是 0--主线程是在事务之外的，不能访问事务的状态。这就像事务有自己的私有区域，用于尝试对状态的改变，而世界上的其他人在事务完成之前不能知道它们。这在事务代码中得到了进一步说明：在它第一次打印之后，它再次将计数器从 1 增加到 2，并打印出结果 2。

事务只有在结束时才会尝试提交其变更。提交的工作原理类似于原子的比较和设置语义。每个引用都会被检查，看它在你第一次试图改变它之后是否有变化。如果有任何_个引用发生了变化，那么_个引用都不会被更新，事务会被重试。例如，如果事务 A 和事务 B 在同一时间被尝试，并且事件按以下顺序发生，事务 A 将被重试。

1. 事务 A： alter gnome
2. 交易 B: alter gnome
3. 交易 B：改变烘干机
4. 交易 B：改变烘干机
5. 事务 B：提交-成功地更新 gnome 和 dryer
6. 事务 A：改变 dryer
7. 事务 A：改变烘干机
8. 事务 A：提交失败，因为 dryer 和 gnome 已经改变；重试。

这就是你的工作! 安全、简单、并发地协调状态变化。但这还不是全部! Refs 还有一个可疑的长袖子的技巧：`commute`。

### commute

`commute`允许你在一个事务中更新一个 ref 的状态，就像 `alter`一样。然而，它在提交时的行为是完全不同的。下面是\`alter'的行为方式。

1. 在事务之外，读取 Ref 的当前状态。
2. 将当前状态与引用者在事务中开始时的状态进行比较。
3. 如果两者不同，则重试交易。
4. 否则，提交改变后的引用状态。

另一方面，`commute`在提交时的行为是这样的。

1. 在事务之外，读取引用的当前状态。
2. 使用当前状态再次运行`commute`函数。
3. 提交结果。

正如你所看到的，`commute`并不强迫事务重试。这可以帮助提高性能，但重要的是，只有当你确定你的 refs 不可能最终处于无效状态时才使用 `commute'。让我们看看`commute\`的安全和不安全使用的例子。

下面是一个安全使用的例子。`sleep-print-update`函数返回更新的状态，但同时也睡眠了指定的毫秒数，所以我们可以强制事务重叠。它打印了它试图更新的状态，所以我们可以深入了解正在发生的事情。

```
(defn sleep-print-update
  [sleep-time thread-name update-fn]
  (fn [state]
    (Thread/sleep sleep-time)
    (println (str thread-name ": " state))
    (update-fn state)))
(def counter (ref 0))
(future (dosync (commute counter (sleep-print-update 100 "Thread A" inc))))
(future (dosync (commute counter (sleep-print-update 150 "Thread B" inc))))
```

下面是打印的时间线。

```
Thread A: 0 | 100ms
Thread B: 0 | 150ms
Thread A: 0 | 200ms 
Thread B: 1 | 300ms
```

请注意，最后打印的一行是 "线程 B：1"。这意味着`sleep-print-update`在第二次运行时收到`1`作为状态参数。这是有道理的，因为此时线程 A 已经提交了它的结果。如果你在事务运行后解除对`counter`的引用，你会发现其值是`2`。

现在，这里有一个不安全交换的例子。

```
(def receiver-a (ref #{}))
(def receiver-b (ref #{}))
(def giver (ref #{1}))
(do (future (dosync (let [gift (first @giver)]
                      (Thread/sleep 10)
                      (commute receiver-a conj gift)
                      (commute giver disj gift))))
    (future (dosync (let [gift (first @giver)]
                      (Thread/sleep 50)
                      (commute receiver-b conj gift)
                      (commute giver disj gift)))))

@receiver-a
; => #{1}

@receiver-b
; => #{1}

@giver
; => #{}
```

`1`被赋予了`receiver-a`和`receiver-b`，你最终得到了两个`1`的实例，这对你的程序是无效的。这个例子的不同之处在于，应用的函数，基本上是`#(conj % gift)`和`#(disj % gift)`，是由`giver`的状态派生的。一旦`giver`发生变化，派生的函数就会产生一个无效的状态，但是`commute`并不关心产生的状态是无效的，无论如何都会提交结果。这里的教训是，尽管`commute`可以帮助你加快程序的速度，但你必须明智地决定何时使用它。

现在你已经准备好开始安全、理智地使用 Refs 了。引用还有一些细微的差别，我在此不做介绍，但如果你对它们感到好奇，你可以研究`ensure`函数和_write skew_现象。

接下来是本书涉及的最后一种参考文献类型。 _vars_。

## Vars

你已经在第 6 章中了解了一些关于 vars 的知识。简单的说, _vars_是符号和对象之间的关联。你可以用`def`创建新的变量。

尽管 vars 并不像原子和 refs 那样用来管理状态，但它们确实有一些并发的技巧：你可以动态地绑定它们，并且可以改变它们的根。让我们先来看看动态绑定。

### 动态绑定

当我第一次介绍`def`时，我恳请你把它当作定义一个常量。事实证明，vars 比这更灵活：你可以创建一个_动态_的 var，它的绑定可以被改变。动态变量对于创建一个全局名称是非常有用的，它应该在不同的情况下指代不同的值。

#### 创建和绑定动态变量

首先，创建一个动态 var。

```
(def:dynamic *notification-address* "dobby@elf.org")
```

注意这里有两个重要的细节。首先，你用`^:dynamic`向 Clojure 发出信号，表明一个 var 是动态的。第二，var 的名字是由星号括起来的。Lispers 称这些为_earmuffs_，这很可爱。Clojure 要求你将动态变量的名字用耳罩括起来。这有助于向其他程序员发出该变量的_动态性_的信号。

与普通变量不同，你可以通过使用`binding`来临时改变动态变量的值。

```
(binding [*notification-address* "test@elf.org")
  *notification-address*)
; => "test@elf.org"
```

你也可以堆叠绑定（就像你可以用`let`）。

```
(binding [*notification-address* "tester-1@elf.org"]
  (println *notification-address*)
  (binding [*notification-address* "tester-2@elf.org"]
    (println *notification-address*))
  (println *notification-address*))
; => tester-1@elf.org
; => tester-2@elf.org
; => tester-1@elf.org
```

现在你知道了如何动态绑定一个 var，让我们看看一个真实世界的应用。

#### 动态 var 的用途

比方说，你有一个发送通知邮件的函数。在这个例子中，我们将只是返回一个字符串，但假装这个函数真的发送了电子邮件。

```
(defn notify
  [message]
  (str "TO: " *notification-address* "\n"
       "MESSAGE: " message))
(notify "I fell.")
; => "TO: dobby@elf.org\nMESSAGE: I fell."
```

如果你想测试这个函数，而不在每次你的规格运行时向多比发送垃圾邮件，怎么办？这时就需要`binding`来帮忙了。

```
(binding [*notification-address* "test@elf.org"]
  (notify "test!"))
; => "TO: test@elf.org\nMESSAGE: test!"
```

当然，你可以直接定义`notify`来接受一个电子邮件地址作为参数。事实上，这通常是正确的选择。为什么要用动态变量来代替呢？

动态变量最常被用来命名一个或多个函数的目标资源。在这个例子中，你可以把电子邮件地址看作是你写给它的资源。事实上，Clojure 为这个目的提供了大量的内置动态变量。 例如，"_out_"代表打印操作的标准输出。在你的程序中，你可以重新绑定`*out*`，使打印语句写到一个文件中，就像这样。

```
(binding [*out* (clojure.java.io/writer "print-output")]
  (println "A man who carries a cat by the tail learns 
something he can learn in no other way.
-- Mark Twain"))
(slurp "print-output")
; => A man who carries a cat by the tail learns
     something he can learn in no other way.
     -- Mark Twain
```

这比每次调用 "println "都传递一个输出目的地要轻松得多。动态变量是一种指定通用资源的好方法，同时保留了在特殊情况下改变它的灵活性。

动态变量也被用于配置。例如，内置的 var`*print-length*`允许你指定 Clojure 应该打印一个集合中的多少个项目。

```
(println ["Print" "all" "the" "things!"])
; => [Print all the things!]

(binding [*print-length* 1]
  (println ["Print" "just" "one!"]))
; => [Print ...]
```

![](https://www.braveclojure.com/assets/images/cftbat/zombie-metaphysics/troll.png)

最后，可以对已经绑定的动态变量进行`set!`。到目前为止，你所看到的例子允许你将信息_输入到一个函数，而不需要将信息作为参数传入，而`set!`允许你将信息_输出到一个函数，而不需要将其作为参数返回。

例如，假设你是一个心灵感应者，但你的读心能力有点延迟。你只有在了解别人的想法对你有用的时候，才能读懂他们的想法。不过，不要觉得太糟糕，你仍然是一个心灵感应者，这很了不起。总之，假设你想穿过一座由巨魔看守的桥，如果你不回答他的谜语，他就会吃掉你。他的谜语是 "我想的是 1 和 2 之间的哪个数字？" 在巨魔吞噬你的情况下，你至少可以知道巨魔到底在想什么而死。

在这个例子中，你创建了动态 var `*troll-thought*`来传达巨魔的想法，从`troll-riddle`函数中出来。

```
(def ^:dynamic *troll-thought* nil)
(defn troll-riddle
  [your-answer]
  (let [number "man meat"]
➊     (when (thread-bound? #'*troll-thought*)
➋       (set! *troll-thought* number))
    (if (= number your-answer)
      "TROLL: You can cross the bridge!"
      "TROLL: Time to eat you, succulent human!")))

(binding [*troll-thought* nil]
  (println (troll-riddle 2))
  (println "SUCCULENT HUMAN: Oooooh! The answer was" *troll-thought*))

; => TROLL: Time to eat you, succulent human!
; => SUCCULENT HUMAN: Oooooh! The answer was man meat
```

你在➊处使用`thread-bound?`函数来检查 var 是否已经被绑定，如果是，你就`set! *troll-thought*`到➋处的巨魔的思想。

变量在绑定之外返回到它的原始值。

```
*troll-thought*
; => nil
```

注意，你必须将`#'*troll-thought*`（包括`#'），而不是`_troll-thought_`，传递给函数`thread-bound?`。这是因为`thread-bound?\`将 var 本身作为一个参数，而不是它所指向的值。

#### 单线程绑定

关于绑定的最后一点要注意：如果你从一个手动创建的线程中访问一个动态绑定的 var，该 var 将求值为原始值。如果你是 Clojure（和 Java）的新手，这个特性不会立即发生作用；你可以跳过这一节，以后再来讨论它。

具有讽刺意味的是，这种绑定行为使我们无法在 REPL 中轻松创建一个有趣的演示，因为 REPL 绑定了`*out*`。就好像你在 REPL 中运行的所有代码都被隐含地包裹在类似`(`binding\[_out_ repl-printer] your-code`的东西中。如果你创建一个新的线程，`_out_\`就不会被绑定到 REPL 打印机上。

下面的例子使用了一些基本的 Java 互操作。即使它看起来很陌生，下面代码的要点也应该很清楚，你将在第 12 章中准确地了解发生了什么。

这段代码向 REPL 打印输出。

```
(.write *out* "prints to repl")
; => prints to repl
```

下面的代码没有打印输出到 REPL，因为`*out*`没有绑定到 REPL 打印机。

```
(.start (Thread. #(.write *out* "prints to standard out")))
```

你可以通过使用这个愚蠢的代码来解决这个问题。

```
(let [out *out*]
  (.start
   (Thread. #(binding [*out* out]
               (.write *out* "prints to repl from thread")))))
```

或者你可以使用`bound-fn`，它将所有当前的绑定带到新的线程中。

```
(.start (Thread. (bound-fn [] (.write *out* "prints to repl from thread"))))
```

`let`绑定捕获了`*out*`，所以我们可以在子线程中重新绑定它，这是很傻的。重点是，绑定不会被传递到_手动_创建的线程中。然而，它们确实被传递给了期货。这就是所谓的 "绑定传递"。在本章中，我们一直在从期货中打印，没有任何问题，比如说。

关于动态绑定就到此为止。让我们把注意力转向最后一个 var 主题：改变 var 的_根_。

### 改变变量根值

当你创建一个新的 var 时，你提供的初始值是它的_根_。

```
(def power-source "hair")
```

在这个例子中，`"头发"`是`power-source`的根值。Clojure 允许你用函数`alter-var-root`永久地改变这个根值。

```
(alter-var-root #'power-source (fn [_] "7-eleven parking lot"))
power-source
; => "7-eleven parking lot"
```

就像使用`swap!`来更新一个原子或`alter!`来更新一个 ref 一样，你使用`alter-var-root`和一个函数来更新一个 var 的状态。在这种情况下，函数只是返回一个新的字符串，与之前的值没有关系，不像`alter!`的例子，我们使用`inc`来从当前的数字衍生出一个新数字。

你几乎不会想这样做。你尤其不想这样做来执行简单的变量赋值。如果你这样做了，你就会不顾一切地把绑定的变量创建为一个可变的变量，这与 Clojure 的理念相悖；最好是使用你在第 5 章学到的函数式编程技术。

你也可以用`with-redefs`暂时改变一个 var 的根。这与绑定的工作原理类似，只是改变的内容会出现在子线程中。下面是一个例子。

```
(with-redefs [*out* *out*]
        (doto (Thread. #(println "with redefs allows me to show up in the REPL"))
          .start
          .join))
```

`with-redefs`可以用于任何 var，而不仅仅是动态的。因为它有如此深远的影响，你应该只在测试时使用它。例如，你可以用它来重新定义一个从网络调用中返回数据的函数，这样该函数就会返回模拟数据而不需要实际进行网络请求。

现在你知道所有关于 vars 的知识了吧! 尽量不要用它们来伤害你自己或你认识的任何人。

## 使用 pmap 的无状态并发性和并行性

到目前为止，本章的重点是那些旨在减少并发编程中固有风险的工具。你已经了解了共享访问可变状态所带来的危险，以及 Clojure 是如何实现状态的重新概念化，从而帮助你安全地编写并发程序。

但通常情况下，你会想把那些完全独立的任务并发化。没有对易变状态的共享访问；因此，并发运行这些任务没有任何风险，你也不必费心使用我刚才说过的任何工具。

事实证明，Clojure 让你可以很容易地编写代码来实现无状态并发。在这一节中，你将了解到`pmap`，它几乎免费为你提供了并发性能的好处。

`map`是并行化的完美候选者：当你使用它时，你所做的只是通过对现有集合的每个元素应用一个函数，从现有集合中派生出一个新集合。不需要维护状态；每个函数的应用都是完全独立的。Clojure 通过`pmap`使执行并行 Map 变得容易。通过`pmap`，Clojure 在一个单独的线程上处理 Map 函数的每个应用的运行。

为了比较`map`和`pmap`，我们需要大量的例子数据，为了生成这些数据，我们将使用`repeatedly`函数。这个函数接收另一个函数作为参数，并返回一个懒惰序列。懒惰序列的元素是通过调用传递的函数生成的，像这样。

```
(defn always-1
  []
  1)
(take 5 (repeatedly always-1))
; => (1 1 1 1 1)
```

下面是你如何创建一个 0 到 9 之间的随机数的懒人序列。

```
(take 5 (repeatedly (partial rand-int 10)))
; => (1 5 0 3 4)
```

让我们使用`repeatedly`来创建示例数据，该数据由 3000 个随机字符串序列组成，每个字符串长 7000 个字符。我们将比较`map`和`pmap`，用它们在这里创建的`orc-names`序列上运行`clojure.string/lowercase`。

```
(def alphabet-length 26)

;; Vector of chars, A-Z
(def letters (mapv (comp str char (partial + 65)) (range alphabet-length)))

(defn random-string
  "Returns a random string of specified length"
  [length]
  (apply str (take length (repeatedly #(rand-nth letters)))))

(defn random-string-list
  [list-length string-length]
  (doall (take list-length (repeatedly (partial random-string string-length)))))

(def orc-names (random-string-list 3000 7000))
```

因为`map`和`pmap`是懒惰的，我们必须强迫它们实现。但我们不希望将结果打印到 REPL 中，因为那会花费很多时间。`dorun`函数做了我们需要的事情：它实现了序列，但返回`nil`。

```
(time (dorun (map clojure.string/lower-case orc-names)))
; => "Elapsed time: 270.182 msecs"

(time (dorun (pmap clojure.string/lower-case orc-names)))
; => "Elapsed time: 147.562 msecs"
```

用`map`串行执行的时间是`pmap`的 1.8 倍，而你所要做的只是增加一个额外的字母 你的性能可能会更好，这取决于你的计算机有多少个内核；这段代码是在双核机器上运行的。

你可能会想，为什么并行版本所花的时间不正好是串行版本的一半。毕竟，两个核心的时间应该只有单核心的一半，不是吗？原因是，在创建和协调线程的过程中，总是会有一些开销。有时，事实上，这种开销所花费的时间会使每个函数应用的时间相形见绌，`pmap'实际上会比`map'花费更多时间。图 10-3 显示了你如何能直观地看到这一点。

![](https://www.braveclojure.com/assets/images/cftbat/zombie-metaphysics/pmap-small-grain.png)

图 10-3：并行化开销会使任务时间相形见绌，导致性能下降。

如果我们对 20000 个缩写的兽人名字运行一个函数，每个 300 个字符的长度，我们就可以看到这种效果的作用。

```
(def orc-name-abbrevs (random-string-list 20000 300))
(time (dorun (map clojure.string/lower-case orc-name-abbrevs)))
; => "Elapsed time: 78.23 msecs"
(time (dorun (pmap clojure.string/lower-case orc-name-abbrevs)))
; => "Elapsed time: 124.727 msecs"
```

现在`pmap`实际上需要 1.6 倍的\*时间。

解决这个问题的方法是增加_粒度_，或者说每个并行化任务所做的工作量。在这种情况下，任务是对集合中的一个元素应用 Map 函数。粒度不是用任何标准单位来衡量的，但你会说`pmap`的粒度默认是 1。将粒度增加到 2 意味着你将 Map 函数应用于两个元素，而不是一个，所以任务所在的线程正在做更多的工作。图 10-4 显示了增加粒度是如何提高性能的。

![](https://www.braveclojure.com/assets/images/cftbat/zombie-metaphysics/ppmap.png)

图 10-4：可视化的粒度与并行化开销的关系

为了在 Clojure 中实现这一点，你可以通过使用`partition-all`使每个线程对多个元素应用`clojure.string/lower-case`，而不是仅仅一个元素，来增加粒度。 `partition-all`接收一个 seq，并将其分成指定长度的 seq。

```
(def numbers [1 2 3 4 5 6 7 8 9 10])
(partition-all 3 numbers)
; => ((1 2 3) (4 5 6) (7 8 9) (10))
```

现在假设你开始时的代码是这样的。

```
(pmap inc numbers)
```

在这种情况下，颗粒大小是 1，因为每个线程都对一个元素应用了`inc`。

现在假设你把代码改成这样。

```
(pmap (fn [number-group] (doall (map inc number-group))
      (partition-all 3 numbers))
; => ((2 3 4) (5 6 7) (8 9 10) (11))
```

这里有几件事要做。首先，你现在将颗粒大小增加到了三个，因为每个线程现在执行了三个`inc`函数的应用，而不是一个。第二，注意你必须在 Map 函数中调用`doall`。这迫使由`(map inc number-group)`返回的懒惰序列在线程内实现。第三，我们需要取消对结果的分组。下面是我们如何做到这一点。

```
(apply concat
       (pmap (fn [number-group] (doall (map inc number-group)))
             (partition-all 3 numbers)))
```

使用这个技术，我们可以增加 orc 名称低 ase 化的粒度，这样每个线程在 1000 个名称上运行`clojure.string/lower-case`而不是只有一个。

```
(time
 (dorun
  (apply concat
         (pmap (fn [name] (doall (map clojure.string/lower-case name)))
               (partition-all 1000 orc-name-abbrevs)))))
; => "Elapsed time: 44.677 msecs"
```

并行版本再次花费了近一半的时间。为了好玩，我们可以把这个技术概括为一个叫做 "ppmap "的函数，代表_分区的 pmap_。它可以接收一个以上的集合，就像`map`一样。

```
(defn ppmap
  "Partitioned pmap, for grouping map ops together to make parallel
  overhead worthwhile"
  [grain-size f & colls]
  (apply concat
   (apply pmap
          (fn [& pgroups] (doall (apply map f pgroups)))
          (map (partial partition-all grain-size) colls))))
(time (dorun (ppmap 1000 clojure.string/lower-case orc-name-abbrevs)))
; => "Elapsed time: 44.902 msecs"
```

我不知道你怎么想的，但我觉得这东西就是好玩。要想获得更多的乐趣，可以看看 clojure.core.reducers 库（[_http://clojure.org/reducers/_](http://clojure.org/reducers/)）。 这个库提供了 seq 函数的替代实现，如`map`和`reduce`，通常比它们在`clojure.core`中的表亲更快。其代价是它们并不懒惰。总的来说，clojure.core.reducers 库为创建和使用`ppmap`这样的函数提供了一种更精细和可组合的方式。

## 摘要

在本章中，你学到了比大多数人更多的关于安全处理并发任务的知识。你了解了支撑 Clojure 引用类型的形而上学。在 Clojure 的形而上学中，状态是某个时间点上的身份值，而身份是指由某个过程产生的一连串值的一种方便的方式。值是原子性的，就像数字是原子性的一样。它们是不可改变的，这使得它们可以安全地并发工作；你不必担心在你使用它们时其他线程会改变它们。

原子引用类型允许你创建一个身份，你可以使用`swap!`和`reset!`安全地更新引用新值。当你想使用事务语义更新多个身份时，ref 引用类型很方便，你用`alter!`和`commute!`更新它。

此外，你学会了如何通过使用`pmap`和 core.reducers 库进行无状态数据转换来提高性能。呜呼!

## 练习

1. 创建一个初始值为 0 的原子，使用`swap!`将其递增几次，然后取消引用。
2.  创建一个函数，使用期货来并行处理从\*[http://www.braveclojure.com/random-quote](http://www.braveclojure.com/random-quote)\*下载随机报价的任务，使用`(slurp "http://www.braveclojure.com/random-quote")`。期货应该更新一个原子，指的是所有引语的总字数。该函数将把要下载的引语数量作为参数，并返回原子的最终值。请记住，在返回原子的最终值之前，你需要确保所有的期货已经完成。下面是你如何调用它和一个示例结果。

    ```
    (quote-word-count 5)
    ; => {"ochre" 8, "smoothie" 2}
    ```
3. 在一个游戏中创建两个角色的代表。第一个角色有 15 个命中率，总共有 40 个。第二个角色在他的库存中有一个治疗药水。使用参照物和交易来模拟治疗药水的消耗和第一个角色的治疗。
