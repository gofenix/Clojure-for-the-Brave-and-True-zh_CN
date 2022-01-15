# 并发和并行编程的神圣艺术

如果我是一个庄园的主人，而你是我的继承人，我会在你的第 13 个命名日让你坐下来，告诉你："计算的世界正在改变，小姑娘，你必须为多核处理器的新世界做好准备，以免你被它践踏。

"好好听着。近年来，CPU 的时钟速度几乎没有增加，但双核和四核计算机已经变得很普遍。物理定律是残酷而绝对的，它们要求提高时钟速度需要成倍的功率。领域内最好的工程师不太可能很快克服这一限制，如果有的话。因此，你可以预期单台机器上的内核不断增加的趋势将继续下去--作为一个程序员，你将知道如何充分利用现代硬件的期望也是如此。

"在这种新模式下学习编程将是有趣和迷人的，真的。但请注意：它也充满了危险。你必须学习_并发和_并行编程\*，这是一门神圣的艺术，使你的应用结构安全地管理多个同时执行的任务。

"你从对并发和并行概念的概述开始学习这门艺术。然后，你将学习困扰每个从业者的三个小妖精：参考单元、互斥和矮人狂战士。你还将学习三种有助于你的工具：Future、许诺和延迟"。

然后我会用键盘拍拍你的肩膀，示意你可以开始了。

## 并发和并行的概念

并发和并行编程在程序执行的各个层面都涉及到很多混乱的细节，从硬件到操作系统，到编程语言库，再到从你的内心涌出的、落在编辑器中的代码。但在你为这些细节烦恼之前，在这一节中，我将介绍围绕并发和并行的高级概念。

### 管理多个任务与同时执行任务

_并发_指的是在同一时间管理一个以上的任务。 _任务_只是意味着 "需要完成的事情"，它并不意味着任何有关硬件或软件的实现。我们可以用 Lady Gaga 的歌曲《电话》来说明并发性。Gaga 唱道

> I cannot text you with a drink in my hand, eh

这里，她在解释她只能管理一个任务（喝酒）。她断然拒绝了她可以处理一个以上的任务的建议。然而，如果她决定同时处理任务，她会唱歌。

> I will put down this drink to text you, then put my phone away and continue drinking, eh

在这个假设的宇宙中，Lady Gaga 正在处理两个任务：喝酒和发短信。然而，她并没有同时执行这两项任务。相反，她在这两个任务之间进行切换，或者说是_交错_。请注意，在交错过程中，你不必在切换之前完全完成一项任务：Gaga 可以打一个字，放下手机，拿起饮料喝一口，然后换回手机，再打一个字。

_平行性_指的是同时执行一个以上的任务。如果加加夫人平行地执行她的两项任务，她会唱歌。

> I can text you with one hand while I use the other to drink, eh

平行性是并发性的一个子类：在你同时执行多个任务之前，你首先要管理多个任务。

Clojure 有很多功能，可以让你轻松实现并行化。虽然 Lady Gaga 系统是通过在多只手上同时执行任务来实现并行的，但计算机系统一般是通过在多个处理器上同时执行任务来实现并行的。

将并行性与_分布式区分开来是很重要的。分布式计算是并行计算的一个特殊版本，处理器在不同的计算机中，任务通过网络分布到计算机上。这就像 Lady Gaga 问 Beyoncé，"请在我喝酒时给这家伙发短信"。尽管你可以借助库在 Clojure 中进行分布式编程，但本书只涉及并行编程，在这里我用_parallel_只指同居的处理器。如果你对分布式编程感兴趣，可以去看看 Kyle Kingsbury 的_Call Me Maybe_系列，网址是_[https://aphyr.com/](https://aphyr.com)\*。

### 阻塞和异步任务

并发编程的主要用例之一是用于_阻塞_操作。阻塞实际上是指等待一个操作的完成。你最常听到的是与 I/O 操作有关的，比如读取文件或等待 HTTP 请求的完成。让我们用 Lady Gaga 并发的例子来研究这个问题。

如果 Lady Gaga 给她的对话者发短信，然后拿着手机站在那里，盯着屏幕等待回应，而不喝水，那么你会说_读下一条短信_操作是阻塞的，这些任务是\*同步执行的。

相反，如果她把手机收起来，这样她就可以喝酒了，直到手机发出哔哔声或振动来提醒她，那么_阅读下一条短信_任务就不是阻塞的，你会说她是在\*异步地处理这个任务。

### 并发编程和并行编程

并发编程和并行编程指的是将一个任务分解成可以并行执行的子任务的技术，以及管理程序同时执行多个任务时产生的风险。在本章的其余部分，我将交替使用这两个术语，因为两者的风险几乎是一样的。

为了更好地理解这些风险以及 Clojure 如何帮助你避免这些风险，让我们来看看 Clojure 中是如何实现并发和并行的。

## Clojure 实现。JVM 线程

我一直在抽象地使用\*任务这个词，指的是一系列相关的操作，而不考虑计算机可能如何实现任务的概念。例如，发短信就是一个由一系列相关操作组成的任务，它与往你脸上倒饮料的操作完全不同。

在 Clojure 中，你可以把你正常的、_串行的代码看作是任务的序列。你可以通过把任务放在 JVM 的_线程\*上来表示任务可以并发执行。

### 什么是线程

我很高兴你问这个问题! 一个线程是一个子程序。一个程序可以有很多线程，每个线程执行自己的指令集，同时享受对程序状态的共享访问。

！

线程管理功能可以存在于计算机的多个层面。例如，操作系统内核通常提供系统调用来创建和管理线程。JVM 提供了自己的独立于平台的线程管理功能，由于 Clojure 程序在 JVM 中运行，所以它们使用 JVM 线程。你将在第 12 章中了解更多关于 JVM 的信息。

你可以把线程看作是一个实际的、物理的线段，它把一连串的指令串起来。在我看来，这些指令是棉花糖，因为棉花糖很好吃。处理器按顺序执行这些指令。我把这想象成一条鳄鱼在吃这些指令，因为鳄鱼喜欢吃棉花糖（这是事实！）。因此，执行一个程序看起来就像一堆棉花糖串在一条线上，一条鳄鱼沿着这条线逐一吃掉。图 9-1 显示了单核处理器执行单线程程序的这个模型。

![](https://www.braveclojure.com/assets/images/cftbat/concurrency/single-thread.png)

图 9-1：单核处理器执行一个单线程的程序

一个线程可以\*产生一个新的线程来并发地执行任务。在单处理器系统中，处理器在线程之间来回切换（交织）。这里就引入了潜在的并发性问题。尽管处理器按顺序执行每个线程的指令，但它不保证何时在线程之间来回切换。

图 9-2 显示了两个线程，A 和 B，以及它们的指令如何执行的时间线。我对线程 B 的指令做了阴影处理，以帮助区分它们与线程 A 的指令。

![](https://www.braveclojure.com/assets/images/cftbat/concurrency/two-threads-one-processor.png)

单核处理器执行两个线程

请注意，这只是一种可能的指令执行顺序。例如，处理器也可以按照 A1、A2、A3、B1、A4、B2、B3 的顺序执行指令。这使程序变得_不确定_。你不能事先知道结果是什么，因为你无法知道执行顺序，不同的执行顺序会产生不同的结果。

这个例子显示了通过交织在单个处理器上的并发执行，而多核系统为每个核分配一个线程，允许计算机同时执行一个以上的线程。每个核心按顺序执行其线程的指令，如图 9-3 所示。

![](https://www.braveclojure.com/assets/images/cftbat/concurrency/two-threads-two-processors.png)

两个线程，两个处理器。

与单核上的交织一样，整体执行顺序没有保证，所以程序是不确定的。当你在程序中加入第二个线程时，它就变得不确定了，这使得你的程序有可能成为三种问题的牺牲品。

### 三个小妖精。参考单元、互斥和矮人狂战士

并发编程中有三个核心挑战，也被称为 "三个并发妖精"。要知道为什么这些是可怕的，想象一下图 9-3 中的程序包括表 9-1 中的假指令。

1. 非确定结果的程序的指令

| ID | Instruction       |
| -- | ----------------- |
| A1 | WRITE `X = 0`     |
| A2 | READ `X`          |
| A3 | WRITE `X = X + 1` |
| B1 | READ `X`          |
| B2 | WRITE `X = X + 1` |

如果处理器遵循 A1, A2, A3, B1, B2 的顺序，那么`X`的值将是`2`，正如你所期望的。但是如果它遵循 A1, A2, B1, A3, B2 的顺序，`X'的值将是`1'，正如你在图 9-4 中看到的那样。

![](https://www.braveclojure.com/assets/images/cftbat/concurrency/reference-cell.png)

两个线程与一个引用单元进行交互

我们把这称为_参考单元_问题（第一个并发性妖精）。 当两个线程可以对同一个位置进行读写时，就会出现引用单元问题，而该位置的值取决于读写的顺序。

第二个并发妖精是_互斥_。想象一下，两个线程，每个人都试图向一个文件写一个咒语。如果没有任何方法可以要求对文件进行独占性的写入访问，那么这个咒语最终会变成乱码，因为写入指令会被交错使用。考虑一下以下两个咒语。

> By the power invested in me\
> by the state of California,\
> I now pronounce you man and wife Thunder, lightning, wind, and rain,\
> a delicious sandwich, I summon again

如果你把这些写到一个没有相互排斥的文件中，你可能会得到这样的结果。

> By the power invested in me\
> by Thunder, lightning, wind, and rain,\
> the state of California,\
> I now pronounce you a delicious man sandwich, and wife\
> I summon again

第三个并发妖精就是我所说的_矮人狂战士_问题（又称_死锁_）。想象一下，四个狂暴者围坐在一张粗糙的圆形木桌旁，互相安慰。"我知道我对我的孩子很疏远，但我就是不知道如何与他们沟通，"一个人咆哮道。其余的人啜饮着咖啡，有意无意地点头，他们的眼角处都有护理纹。

现在，每个人都知道，矮人狂战士结束舒适的咖啡聚会的仪式是拿起他们的 "安慰棒"（双刃战斧），互相抓挠对方的背部。每对矮人之间放一把战斧，如图 9-5 所示。

他们的仪式是这样进行的。

1. 拿起_左边_的战斧，如果有的话。
2. 拿起_右边_战斧，如果有的话。
3. 用你的 "安慰棒 "大力挥舞来安慰你的邻居。
4. 释放两把战斧。
5. 重复。

![](https://www.braveclojure.com/assets/images/cftbat/concurrency/deadlock.png)

矮人狂热者在舒适的咖啡聚会中

按照这个仪式，所有的矮人狂战士完全有可能拿起他们左边的安慰棒，然后无限期地阻挡，同时等待他们右边的安慰棒出现，导致僵局。(顺便说一下，如果你想进一步研究这种现象，它通常被称为_吃饭的哲学家问题_，但这是一个更无聊的场景）。本书没有详细讨论死锁，但了解这个概念和术语是很好的。

并发编程有它的小妖精，但有了正确的工具，它是可控的，甚至是有趣的。让我们开始看一下正确的工具。

## Futures, Delays, and Promises

Future、延迟和 Promise 是用于并发编程的简单、轻便的工具。在这一节中，你将学习每个工具的工作原理，以及如何一起使用它们来抵御引用单元的并发妖精和互斥的并发妖精。你会发现，虽然简单，但这些工具对满足你的并发需求有很大帮助。

它们通过给予你比串行代码更多的灵活性来做到这一点。当你写串行代码时，你把这三个事件绑定在一起。

* 任务定义
* 任务执行
* 要求任务的结果

作为一个例子，看一下这个假设的代码，它定义了一个简单的 API 调用任务。

```
(web-api/get :dwarven-beard-waxes)
```

一旦 Clojure 遇到这个任务定义，它就会执行它。它也需要_现在_的结果，阻塞直到 API 调用完成。学习并发编程的一部分是学会识别何时不需要这些时间上的联接。Future、延迟和 Promise 允许你把任务定义、任务执行和要求结果分开。继续前进!

### Future

在 Clojure 中，你可以使用_futures_来定义一个任务，并把它放在另一个线程上，而不要求立即得到结果。你可以用`future`宏来创建一个未来。在 REPL 中试试这个。

```
(future (Thread/sleep 4000)
        (println "I'll print after 4 seconds"))
(println "I'll print immediately")
```

`Thread/sleep`告诉当前线程在指定的毫秒数内坐着什么都不做。通常情况下，如果你在你的 REPL 中评估了`Thread/sleep`，你就不能评估任何其他语句，直到 REPL 完成睡眠；执行你的 REPL 的线程将被阻塞。然而，`future`创建了一个新的线程，并将你传递给它的每个表达式放在新的线程上，包括`Thread/sleep`，允许 REPL 的线程继续运行，不受阻塞。

你可以使用 future 在一个单独的线程上运行任务，然后忘记它们，但你经常想使用任务的结果。`future`函数返回一个引用值，你可以用它来请求结果。参考值就像干洗店给你的票据：在任何时候你都可以用它来请求你的干净衣服，但是如果你的衣服还没有洗干净，你就必须等待。类似地，你可以使用参考值来请求一个未来的结果，但是如果未来还没有完成计算结果，你就必须等待。

请求一个未来的结果被称为_dereferencing_未来，你可以用`deref`函数或`@`读者宏来做。一个 future 的结果值是其主体中最后一个被评估的表达式的值。一个 future 的主体只执行一次，它的值被缓存起来。试试下面的方法。

```
(let [result (future (println "this prints once")
                     (+ 1 1))]
  (println "deref: " (deref result))
  (println "@: " @result))
; => "this prints once"
; => deref: 2
; => @: 2
```

请注意，"this prints once "这个字符串确实只打印了一次，尽管你对 future 进行了两次推断。这表明 future 的主体只运行了一次，结果`2`被缓存了。

如果未来程序还没有完成运行，解指未来程序就会阻塞，就像这样。

```
(let [result (future (Thread/sleep 3000)
                     (+ 1 1))]
  (println "The result is: " @result)
  (println "It will be at least 3 seconds before I print"))
; => The result is: 2
; => It will be at least 3 seconds before I print
```

有时你想为一个未来的等待时间设置一个时间限制。要做到这一点，你可以给`deref`一个等待的毫秒数，以及当`deref`超时时要返回的值。

```
(deref (future (Thread/sleep 1000) 0) 10 5)
; => 5
```

这段代码告诉`deref`，如果 future 在 10 毫秒内没有返回一个值，则返回值`5`。

最后，你可以使用`realized?`来询问一个 future，看看它是否已经运行完毕。

```
(realized? (future (Thread/sleep 1000)))
; => false

(let [f (future)]
  @f
  (realized? f))
; => true
```

Future 是一种简单的方法，可以在你的程序中撒上一些并发性。

就其本身而言，它们让你有能力将任务转移到其他线程上，这可以使你的程序更有效率。它们还可以让你的程序表现得更加灵活，让你控制何时需要一个任务的结果。

当你解除对一个 future 的引用时，你表明_现在_需要这个结果，并且在获得这个结果之前应该停止计算。你会看到这如何帮助你处理相互排斥的问题。另外，你也可以忽略这个结果。例如，你可以使用 Future 来异步写入一个日志文件，在这种情况下，你不需要解除对 Future 的引用来获得任何返回值。

Future 给你带来的灵活性是非常酷的。Clojure 还允许你用延迟和 Promise 来独立处理任务定义和要求结果。

### 延迟

_延迟_允许你定义一个任务，而不需要立即执行它或要求得到结果。你可以使用`delay`创建一个延迟。

```
(def jackson-5-delay
  (delay (let [message "Just call my name and I'll be there"]
           (println "First deref:" message)
           message)))
```

在这个例子中，没有任何东西被打印出来，因为我们还没有要求对`let`形式进行评估。你可以评估延迟，并通过解构它或使用`force`来获得其结果。 `force`的行为与`deref`相同，但它更清楚地表达了你正在使一个任务开始，而不是等待一个任务完成。

```
(force jackson-5-delay)
; => First deref: Just call my name and I'll be there
; => "Just call my name and I'll be there"
```

像 Future 一样，延迟只运行一次，其结果被缓存。后续的取消引用将返回 Jackson 5 的信息，而不打印任何东西。

```
@jackson-5-delay
; => "Just call my name and I'll be there"
```

你可以使用延迟的一种方式是在一组相关的 Future 中的一个 Future 第一次完成时启动一个语句。例如，假设你的应用程序将一组头像上传到一个头像分享网站，并在第一张头像完成后立即通知所有者，如下所示。

```
(def gimli-headshots ["serious.jpg" "fun.jpg" "playful.jpg"])
(defn email-user
  [email-address]
  (println "Sending headshot notification to" email-address))
(defn upload-document
  "Needs to be implemented"
  [headshot]
  true)
(let [notify (delay ➊(email-user "and-my-axe@gmail.com"))]
  (doseq [headshot gimli-headshots]
    (future (upload-document headshot)
            ➋(force notify))))
```

在这个例子中，你定义了一个要上传头像的向量（`gimli-headshots`）和两个函数（`email-user`和`upload-document`）来假装执行这两个操作。然后你用`let`将`notify`绑定到一个延迟。延迟的主体，`(email-user "and-my-axe@gmail.com")`➊，在创建延迟的时候并没有被评估。相反，当由`doseq`表单创建的 Future 之一第一次评估`(force notify)`➋时，它就被评估了。即使`(force notify)`将被评估三次，延迟主体只被评估一次。Gimli 会很高兴知道第一张头像什么时候可用，这样他就可以开始调整它并分享它。他也会感谢不被垃圾邮件，而你也会感谢不面对他的矮人之怒。

这种技术可以帮助你避免相互排斥的并发妖精--确保每次只有一个线程可以访问特定资源的问题。在这个例子中，延迟守护着电子邮件服务器资源。因为延迟的主体被保证只发射一次，所以你可以确定你永远不会遇到两个线程发送相同邮件的情况。当然，没有线程能够再次使用延迟来发送邮件。对于大多数情况来说，这可能是一个过于激烈的约束，但在像这个例子这样的情况下，它是完美的。

### Promise

_Promise_允许你表达你期望的结果，而不需要定义应该产生结果的任务或该任务应该何时运行。你用`promise'创建Promise，用`deliver'向他们传递一个结果。你通过取消引用来获得结果。

```
(def my-promise (promise))
(deliver my-promise (+ 1 2))
@my-promise
; => 3
```

这里，你创建了一个 promise，然后向它传递一个值。最后，你通过解除对 Promise 的引用来获得该值。解除引用是你表达你期望一个结果的方式，如果你试图在没有首先传递一个值的情况下解除引用\`my-promise'，程序将阻塞，直到一个 Promise 被传递，就像 Future 和延迟一样。你只能向一个 Promise 传递一次结果。

Promise 的一个用途是在一个数据集合中找到第一个满意的元素。例如，假设你正在收集成分以使你的鹦鹉听起来像詹姆斯-厄尔-琼斯。因为詹姆斯-厄尔-琼斯的声音是世界上最顺畅的，所以其中一种成分是顺畅度达到 97 以上的优质牦牛油。你的预算是 100 美元一磅。

你是一个现代神奇鸟类学艺术的实践者，因此，与其繁琐地浏览每个牦牛油零售网站，不如创建一个脚本，给你提供第一个符合你需求的牦牛油的 URL。

下面的代码定义了一些牦牛油产品，创建了一个函数来模拟 API 调用，并创建了另一个函数来测试产品是否满意。

```
(def yak-butter-international
  {:store "Yak Butter International"
    :price 90
    :smoothness 90})
(def butter-than-nothing
  {:store "Butter Than Nothing"
   :price 150
   :smoothness 83})
;; This is the butter that meets our requirements
(def baby-got-yak
  {:store "Baby Got Yak"
   :price 94
   :smoothness 99})

(defn mock-api-call
  [result]
  (Thread/sleep 1000)
  result)

(defn satisfactory?
  "If the butter meets our criteria, return the butter, else return false"
  [butter]
  (and (<= (:price butter) 100)
       (>= (:smoothness butter) 97)
       butter))
```

该 API 调用在返回结果前等待一秒钟，以模拟执行实际调用的时间。

为了说明同步检查网站需要多长时间，我们将使用`some`对集合中的每个元素应用`satisfactory?`函数，并返回第一个真实的结果，如果没有，则返回 nil。当你同步检查每个站点时，每个站点可能需要超过一秒钟的时间来获得结果，正如下面的代码所示。

```
(time (some (comp satisfactory? mock-api-call)
            [yak-butter-international butter-than-nothing baby-got-yak]))
; => "Elapsed time: 3002.132 msecs"
; => {:store "Baby Got Yak", :smoothness 99, :price 94}
```

这里我用`comp`来组合函数，我用`time`来打印评估一个表单的时间。你可以使用 promise 和 futures 来在一个单独的线程上执行每个检查。如果你的计算机有多个核心，这可以把时间减少到一秒钟左右。

```
(time
 (let [butter-promise (promise)]
   (doseq [butter [yak-butter-international butter-than-nothing baby-got-yak]]
     (future (if-let [satisfactory-butter (satisfactory? (mock-api-call butter))]
               (deliver butter-promise satisfactory-butter))))
   (println "And the winner is:" @butter-promise)))
; => "Elapsed time: 1002.652 msecs"
; => And the winner is: {:store Baby Got Yak, :smoothness 99, :price 94}
```

在这个例子中，你首先创建了一个 Promise，`butter-promise`，然后创建了三个访问该 Promise 的 Future。每个 Future 的任务是评估一个牦牛黄油网站，如果该网站令人满意，则向 Promise 提供该网站的数据。最后，你解除对`butter-promise`的引用，导致程序阻塞，直到网站数据被交付。这需要一秒钟而不是三秒钟，因为网站的评估是平行进行的。通过将对结果的要求与结果的实际计算方式脱钩，你可以并行地进行多个计算，并节省一些时间。

你可以把这看作是一种保护自己不受参考单元格并发性妖精影响的方法。因为 Promise 只能被写入一次，你可以防止非确定性读写产生的那种不一致的状态。

你可能想知道，如果牦牛油都不满意会怎么样。如果发生这种情况，解除引用将永远阻塞，并绑住线程。为了避免这种情况，你可以加入一个超时。

```
(让 [p (promise)]
  (deref p 100 "timed out")
```

这将创建一个 Promise，`p'，并试图解除对它的引用。数字100告诉`deref`等待100毫秒，如果届时没有可用的值，就使用超时值，`"timed out"\`。

我应该提到的最后一个细节是，你也可以使用 Promise 来注册回调，实现与你在 JavaScript 中可能习惯的相同功能。JavaScript 的回调是一种定义代码的方式，一旦其他代码完成，就应该异步执行。下面是如何在 Clojure 中做到这一点。

```
(let [ferengi-wisdom-promise (promise)]
  (future (println "Here's some Ferengi wisdom:" @ferengi-wisdom-promise))
  (Thread/sleep 100)
  (deliver ferengi-wisdom-promise "Whisper your way to success."))
; => Here's some Ferengi wisdom: Whisper your way to success.
```

这个例子创建了一个立即开始执行的未来。然而，未来的线程是阻塞的，因为它在等待一个值被传递给`ferengi-wisdom-promise`。100 毫秒后，你交付了值，未来中的\`println'语句开始运行。

Future、延迟和 Promise 是在你的应用程序中管理并发性的伟大而简单的方法。在下一节中，我们将看到一个更有趣的方法来控制你的并发应用程序。

### 滚动你自己的队列

到目前为止，你已经看到了一些简单的方法来结合 Future、延迟和 Promise，使你的并发程序更加安全。在这一节中，你将使用一个宏来以一种稍微复杂的方式结合 Future 和 Promise。你可能不一定会用到这段代码，但它会更多地展示这些适度的工具的力量。这个宏需要你在头脑中同时持有运行时逻辑和宏扩展逻辑，以了解正在发生的事情；如果你被卡住了，就跳过前面。

三个并发妖精的一个共同特点是，它们都涉及到任务以不协调的方式并发地访问一个共享资源--变量、打印机、矮人战斧。如果你想确保每次只有一个任务会访问一个资源，你可以把任务的资源访问部分放在一个序列执行的队列中。这有点像做蛋糕：你和一个朋友可以分别取回原料（鸡蛋、面粉、蝾螈的眼睛，等等），但有些步骤你必须连续执行。你必须在把面糊放进烤箱之前准备好它。图 9-6 说明了这个策略。

![](https://www.braveclojure.com/assets/images/cftbat/concurrency/enqueue.png)

将任务分为串行部分和并发部分，可以让你安全地使你的代码更有效率。

为了实现排队宏，你要向英国人致敬，因为他们发明了队列。你将使用一个队列来确保英国人习惯的问候语 "Ello, gov'na! Pip Pip! Cheerio!"的正确顺序进行传递。这个演示将涉及到大量的 "睡眠"，所以这里有一个宏来更简洁地完成这个任务。

```
(defmacro wait
  "Sleep `timeout` seconds before evaluating body"
  [timeout & body]
  `(do (Thread/sleep ~timeout) ~@body))
```

这段代码所做的就是接受你给它的任何形式，并在它们之前插入对`Thread/sleep`的调用，所有这些都被`do`包裹起来。

清单 9-1 中的代码将任务分成了并发部分和序列化部分。

```
(let [saying3 (promise)]
  (future (deliver saying3 (wait 100 "Cheerio!")))
  @(let [saying2 (promise)]
     (future (deliver saying2 (wait 400 "Pip pip!")))
➊      @(let [saying1 (promise)]
        (future (deliver saying1 (wait 200 "'Ello, gov'na!")))
        (println @saying1)
        saying1)
     (println @saying2)
     saying2)
  (println @saying3)
  saying3)
```

1. 9-1. 一个 enqueue 宏调用的扩展

整体策略是为每个任务（在本例中，打印问候语的一部分）创建一个 Promise，以创建一个相应的未来，将并发计算的值交付给该 Promise。这确保了在任何一个 Promise 被解除引用之前，所有的未来都被创建，并且确保序列化的部分以正确的顺序执行。首先打印`saying1'的值-`"'Ello, gov'na!"-然后是`saying2'的值，最后是`saying3'。在`let`块中返回`saying1`，并在➊处解除对`let`块的引用，可以确保在代码继续对`saying2`做任何事情之前，你已经完全完成了`saying1`，而且这种模式在`saying2`和`saying3`上重复。

解除对`let`块的引用似乎很傻，但这样做可以让你用一个宏来抽象这段代码。你肯定想使用宏，因为像前面的例子那样写出的代码会让你发疯（英国人会这么说）。理想情况下，这个宏的工作方式如清单 9-2 所示。

```
(-> (enqueue ➊saying ➋(wait 200 "'Ello, gov'na!") ➌(println @saying))
   ➍(enqueue saying (wait 400 "Pip pip!") (println @saying))
    (enqueue saying (wait 100 "Cheerio!") (println @saying)))
```

1. 这就是你使用 enqueue 的方法。

该宏让你命名被创建的 Promise➊，定义如何获取价值以交付该 Promise➋，并定义如何处理该 Promise➌。这个宏也可以把另一个`enqueue`宏调用作为它的第一个参数，这样你就可以把它变成线程➍。清单 9-3 显示了你如何定义`enqueue`宏。定义完`enqueue`后，清单 9-2 中的代码将扩展为清单 9-1 中的代码，其中包含所有嵌套的`let`表达式。

```
(defmacro enqueue
➊   ([q concurrent-promise-name concurrent serialized]
➋    `(let [~concurrent-promise-name (promise)]
      (future (deliver ~concurrent-promise-name ~concurrent))
➌       (deref ~q)
      ~serialized
      ~concurrent-promise-name))
➍   ([concurrent-promise-name concurrent serialized]
   `(enqueue (future) ~concurrent-promise-name ~concurrent ~serialized)))
```

1.enqueue 的实现

首先注意这个宏有两个 arities，以便提供一个缺省值。第一个 arity ➊是真正的工作所在。它有参数`q`，而第二个 arity 则没有。第二个 arity ➍调用第一个 arity，为`q`提供了`(future)`的值；你将在一分钟内看到原因。在➋中，宏返回一个表单，该表单创建了一个 Promise，在一个 future 中传递它的值，取消对提供给`q`的任何表单的引用，评估序列化代码，最后返回 Promise。`q`通常是一个嵌套的`let`表达式，由另一个对`enqueue`的调用返回，如清单 9-2 所示。如果没有为`q`提供值，宏会提供一个未来，这样在➌的`deref`就不会引起异常。

现在我们已经写好了`enqueue`宏，让我们试试它是否能减少执行时间

```
(time @(-> (enqueue saying (wait 200 "'Ello, gov'na!") (println @saying))
           (enqueue saying (wait 400 "Pip pip!") (println @saying))
           (enqueue saying (wait 100 "Cheerio!") (println @saying))))
; => 'Ello, gov'na!
; => Pip pip!
; => Cheerio!
; => "Elapsed time: 401.635 msecs"
```

天哪! 问候语是按照正确的顺序传递的，你可以通过耗时看到，睡眠的 "工作 "是同时进行的。

## 总结

对于像你这样的程序员来说，学习并发和并行编程技术很重要，这样你就可以设计出在现代硬件上高效运行的程序。并发是指一个程序能够执行一个以上的任务，在 Clojure 中，你可以通过将任务放在不同的线程上来实现。当计算机有一个以上的 CPU 时，程序就会并行执行，这样就可以在同一时间执行一个以上的线程。

并发编程指的是用于管理三种并发风险的技术：引用单元、互斥和死锁。Clojure 为你提供了三个基本工具，帮助你减轻这些风险：Future、延迟和 Promise。每个工具都可以让你把定义任务、执行任务和要求任务结果这三个事件解耦。Future 让你定义一个任务并立即执行它，允许你稍后或永远不要求结果。 Future 也会缓存其结果。延迟（Delay）让你定义一个稍后才执行的任务，并且延迟的结果会被缓存起来。许诺让你表达你需要一个结果，而不需要知道产生该结果的任务。你只能向一个 Promise 传递一次值。

在下一章中，你将探索并发编程的哲学层面，并学习更复杂的工具来管理风险。

## 练习

1. 编写一个函数，将一个字符串作为参数，使用`slurp`函数在 Bing 和 Google 上搜索它。你的函数应该返回搜索到的第一个页面的 HTML。
2. 更新你的函数，使其接受第二个参数，包括要使用的搜索引擎。
3. 创建一个新的函数，将搜索词和搜索引擎作为参数，并从每个搜索引擎的第一页搜索结果中返回一个 URL 的向量。
