# 用 core.async 掌握并发进程

有一天，当你走在大街上时，你会惊讶、好奇，并有点厌恶地发现一台热狗自动贩卖机。你的头皮被有罪的好奇心刺痛，你会忍不住掏出三块钱，看看这个装置是否真的能工作。在 "咔嚓 "一声接受了你的钱后，它弹出了一个新鲜的热狗，包括面包和所有的东西。

![](https://www.braveclojure.com/assets/images/cftbat/core-async/hotdog-vending-machine.png)

自动售货机表现出简单的行为：当它收到钱时，它会放出一个热狗，然后为下一次购买做准备。当它的热狗用完时，它就会停止。我们周围的热狗自动售货机以不同的面貌出现，它们是独立的实体，同时对世界上的事件作出反应。你最喜欢的咖啡店的浓缩咖啡机，你小时候喜欢的宠物仓鼠--所有的东西都可以被分解成一组行为，这些行为遵循一般的形式 "当_x_发生时，做_y_"。甚至我们写的程序也只是美化的热狗贩卖机，每一个都是独立的进程，等待着下一个事件的发生，无论是击键、超时，还是套接字上的数据到达。

Clojure 的 core.async 库允许你在一个程序中创建多个独立进程。 本章描述了思考这种编程风格的有用模型，以及你在实际编写代码时需要了解的实际细节。你将学习如何使用通道在由 go 块和`thread`创建的独立进程之间进行通信；了解一些关于 Clojure 如何通过停放和阻塞有效地管理线程；如何使用`alts!!`；以及一种更直接的创建队列的方法。最后，你将学习如何用进程管道来踢回调的屁股。

## 进程的入门

core.async 的核心是\*进程，一个并发运行的逻辑单元，对事件做出反应。进程对应于我们对现实世界的心理模型：实体之间的互动和响应是独立的，没有某种中央控制机制的牵制。你把钱放进机器里，就会有一个热狗出来，所有这些都不需要光照派或老大哥来策划整个事情。这与你迄今为止一直在探索的并发性观点不同，在那里，你定义的任务要么只是控制主线程的扩展（例如，用`pmap`实现数据并行），要么是你没有兴趣与之交流的任务（如用`future`创建的一次性任务）。

把自动售货机看成是一个进程可能很奇怪：自动售货机是名词和事物，而进程是动词和行为。为了获得正确的思维方式，可以尝试将现实世界的物体定义为其事件驱动的行为的总和。当一粒种子被浇水时，它就会发芽；当母亲看着她的新生儿时，她就会感受到爱；而当你观看《_星_战》第一集时，你会充满愤怒和绝望。如果你想变得超级哲学，可以考虑是否有可能将每个事物的本质定义为它所识别的事件的集合，以及它如何做出反应。现实是否只是热狗售卖机的组成？

总之，我说得够多了! 让我们通过创建一些简单的过程，从理论上走向具体。首先，用 "lein new app playsync "创建一个新的 Leiningen 项目，名为_playsync_。然后，打开_project.clj_文件，将 core.async 添加到`:dependencies`向量中，使其内容如下。

```
[[org.clojure/clojure "1.9.0"]
[org.clojure/core.async "0.1.346.0-17112a-alpha"]]
```

注意 自从我写完这篇文章后，core.async 的版本有可能有所进步。关于最新的版本，请查看 core.async 的 GitHub 项目页面。但为了这些练习的目的，请使用这里列出的版本。

接下来，打开_src/playsync/core.clj_，使其看起来像这样。

```
(ns playsync.core
  (:require [clojure.core.async
             :as a
             :refer [>! <! >!! <!! go chan buffer close! thread
                     alts! alts!! timeout]]))
```

现在，当你在 REPL 中打开它时，你将拥有最常用的 core.async 函数供你使用。很好! 在创建像热狗售卖机那样复杂和革命性的东西之前，先创建一个进程，简单地打印它收到的消息。

```
(def echo-chan (chan))
(go (println (<! echo-chan)))
(>!! echo-chan "ketchup")
; => true
; => ketchup
```

在第一行代码中，你用`chan`函数创建了一个名为`echo-chan`的_通道。通道传达_消息\*。你可以把_消息放到一个通道上，也可以把_消息从一个通道上拿下来。进程_等待_放和取的完成--这些是进程所响应的事件。你可以认为进程有两个规则。1）当试图把一个消息放到通道上或从通道上取走一个消息时，等待并不做任何事情，直到放或取成功；2）当放或取成功时，继续执行。

在下一行，你用`go`来创建一个新的进程。`go`表达式中的所有内容都被称为_go 块_，在一个单独的线程上并发运行。Go 块在一个线程池上运行你的进程，该线程池包含的线程数量等于你的机器上的核心数量的两倍，这意味着你的程序不必为每个进程创建一个新的线程。这通常会带来更好的性能，因为你避免了与创建线程有关的开销。

在这个例子中，进程`(println (<! echo-chan))`表达了 "当我从\`echo-chan'那里得到一个消息时，打印它"。该进程被分流到另一个线程，释放了当前线程，使你能够继续与 REPL 交互。

在表达式`(<! echo-chan)`中，`<!`是_take_函数。它监听你给它作为参数的通道，它所属的进程等待，直到另一个进程在该通道上放出一个消息。当`<!`检索到一个值时，该值被返回并执行`println`表达式。

表达式`(>!! echo-chan "ketchup")`将字符串`"ketchup"`放到`echo-chan`上并返回`true`。当你把一个消息放在一个通道上时，该进程会阻塞，直到另一个进程接收该消息。在这种情况下，REPL 进程根本不需要等待，因为已经有一个进程在监听该通道，等待从该通道中获取信息。然而，如果你做了以下事情，你的 REPL 将无限期地阻塞。

```
(>!! (chan) "mustard")
```

你已经创建了一个新的通道，并在上面放了一些东西，但没有进程在监听这个通道。进程不仅仅是等待接收消息，他们也在等待他们放在通道上的消息被接收。

### 缓冲

值得注意的是，前面的练习包含_两个_进程：你用`go`创建的进程和 REPL 进程。这些进程相互之间没有明确的知识，而且它们独立行动。

让我们想象一下，这些过程发生在一个餐厅里。REPL 是番茄酱厨师，当他完成一个批次时，他大声说："番茄酱！" 完全有可能的是，其他员工都在外面欣赏他们有机花园里最新的一批牛至，而厨师只是坐着等待，直到有人来取他的番茄酱。反过来说，"去 "的过程代表了其中一个工作人员，他正在耐心地等待着什么回应。可能是什么都没有发生，他只是无限期地等待，直到餐厅关门。

这种情况似乎有点傻：哪个自尊心强的番茄酱厨师会在制作更多的番茄酱之前，只是坐等别人拿走他最新的一批番茄酱？为了避免这种悲剧的发生，你可以创建缓冲通道。

```
(def echo-buffer (chan 2))
(>!! echo-buffer "ketchup")
; => true
(>!! echo-buffer "ketchup")
; => true
(>!! echo-buffer "ketchup")
; This blocks because the channel buffer is full
```

(小心评估最后一个`(>! ! echo-buffer "ketchup")`，因为它将阻塞你的 REPL。如果你使用的是 Leiningen REPL，ctrl-C 会解除封锁）。

在这种情况下，你已经创建了一个缓冲区大小为 2 的通道。这意味着你可以在通道上放两个值而不需要等待，但放第三个值意味着进程将等待，直到另一个进程从通道上取值。你还可以用`sliding-buffer`创建_滑动_缓冲区，它以先入先出的方式丢弃数值；用`dropping-buffer`创建_丢弃_缓冲区，它以后入先出的方式丢弃数值。这两种缓冲区都不会导致`>!!`阻塞。

通过使用缓冲区，番茄酱大师可以继续制作成批令人垂涎欲滴的番茄酱，而不必等待他的员工把它们带走。如果他使用普通的缓冲器，就像他有一个架子，可以把所有的番茄酱批次放在上面；一旦架子满了，他还得等待空间的打开。如果他用的是滑动缓冲器，当架子上的番茄酱满了，他就会把最旧的一批扔掉，把所有的番茄酱滑下来，然后把新的一批放到空出来的地方。如果是跌落式缓冲器，他就会把最新鲜的一批番茄酱从货架上打下来，然后把新的一批番茄酱放在那个空间里。

缓冲区只是对核心模型的阐述：进程是独立的、并发执行的逻辑单元，对事件作出反应。你可以用 go 块来创建进程，并通过通道来沟通事件。

### 堵塞和停车

你可能已经注意到，take 函数`<!`只使用了一个感叹号，而 put 函数`>!`则使用了两个感叹号。事实上，put 和 take 都有一个感叹号和两个感叹号的种类。什么时候使用哪个？简单的答案是，你可以在 go 块内使用一个感叹号，但你必须在 go 块外使用两个感叹号。

|      | Inside go block | Outside go block |
| ---- | --------------- | ---------------- |
| put  | `>!` or `>!!`   | `>!!`            |
| take | `<!` or `<!!`   | `<!!`            |

这一切都归结为效率问题。因为 go 块使用一个固定大小的线程池，你可以创建 1000 个 go 进程，但只使用少量的线程。

```
(def hi-chan (chan))
(doseq [n (range 1000)])
  (go (>! hi-chan (str "hi " n)))))
```

为了理解 Clojure 是如何做到这一点的，我们需要探索进程如何_等待。等待是使用 core.async 进程的一个关键方面：我们已经确定，put会等待到另一个进程在同一通道上做_take\*，反之亦然。在这个例子中，1,000 个进程在等待另一个进程从 "hi-chan "中提取。

有两种类型的等待。 _停车_和_阻塞_。阻塞是你熟悉的那种等待：一个线程停止执行，直到一个任务完成。通常这发生在你进行某种 I/O 操作的时候。这个线程仍然活着，但不做任何工作，所以如果你想让你的程序继续工作，你必须创建一个新的线程。在第 9 章中，你学到了如何用 "future "来做这件事。

停车释放了线程，这样它就可以继续工作了。假设你有一个线程和两个进程，Process A 和 Process B，Process A 在线程上运行，然后等待放或取。Clojure 将进程 A 移出线程，并将进程 B 移到线程上。如果进程 B 开始等待，而进程 A 的 put 或 take 已经完成，那么 Clojure 将把进程 B 移出线程，把进程 A 放回线程上。停放允许多个进程的指令在一个线程上交错，类似于使用多个线程允许在一个核心上交错的方式。停放的实现并不重要；只需说它只在 go 块内实现，并且只在使用`>!`和`<!`，或_停放 put_和_停放 take_时实现。`>!!`和`<!!`是_停放的放_和_停放的取_。

### 线程

肯定有一些时候你会想使用阻塞而不是停放，比如你的进程要花很长时间才能放或取，在这些场合你应该使用`线程`。

```
(thread (println (<!! echo-chan)))
(>!! echo-chan "mustard")
; => true
; => mustard
```

`thread`的行为几乎与`future`完全一样：它创建一个新的线程并在该线程上执行一个进程。与`future'不同的是，`thread'不是返回一个可以反推的对象，而是返回一个通道。当`thread`的进程停止时，该进程的返回值会被放在`thread`返回的通道上。

```
(let [t (thread "chili") ]
  (<!! t))
; => "chili"
```

在这种情况下，进程不等待任何事件；相反，它立即停止。它的返回值是 "chili"，它被放在与`t绑定的通道上。`我们从`t`中获取，返回\`"chili"。

当你执行一个长期运行的任务时，你应该使用`thread`而不是 go block，原因是你不会堵塞你的线程池。想象一下，你正在运行四个进程，下载巨大的文件，保存它们，然后把文件的路径放在一个通道上。当这些进程在下载文件和保存这些文件时，Clojure 不能停放它们的线程。它只能在最后一步停放线程，即进程将文件的路径放在通道上时。因此，如果你的线程池只有四个线程，所有四个线程都将被用于下载，在其中一个下载完成之前，不允许其他进程运行。

`go`、`thread`、`chan`、`<!`、`<!`、`>!`和`>!`是你用来创建和与进程通信的核心工具。put 和 take 都会使一个进程等待，直到它的补码在给定的通道上被执行。`go`允许你使用 put 和 take 的停车变体，这可以提高性能。如果你在 put 和 take 之前执行长期运行的任务，你应该使用阻塞式变体，以及`thread`。

这应该能满足你的一切需求，让你实现你的心愿，创造一台把钱变成热狗的机器。

## 你一直渴望的热狗机过程

看哪，你的梦想成真了!

```
(defn hot-dog-machine
  []
  (let [in (chan)
        out (chan)]
    (go (<! in)
        (>! out "hot dog"))
    [in out]))
```

这个函数创建了一个`in`通道用于接收钱，一个`out`通道用于发放热狗。然后用`go`创建一个异步进程，等待钱，然后发放热狗。最后，它将`in`和`out`通道作为一个向量返回。

是时候吃热狗了!

```
(let [[in out] (hot-dog-machine)]
  (>!! in "pocket lint")
  (<!! out))
; => "hot dog"
```

在这个片段中，你用 destructuring（在第三章中讲到）和`let`将`in`和`out`通道绑定到`in`和`out`符号。然后你把 "pocket lint "放在 "in "通道上。热狗机器进程等待着一些东西，任何东西，到达`in`通道；一旦`"pocket lint"`到达，热狗机器进程恢复执行，将`"hot dog"`放在`out`通道上。

等一下……这不对。我的意思是，是的，免费的热狗，但是一定会有人因为机器接受小棉絮作为付款而不高兴。不仅如此，这台机器在关闭前只能发放一个热狗。让我们改变热狗机的功能，让你可以指定它有多少个热狗，并且当你给它数字 3 时，它才会发放一个热狗。

```
(defn hot-dog-machine-v2
  [hot-dog-count]
  (let [in (chan)
        out (chan)]
    (go (loop [hc hot-dog-count]
          (if (> hc 0)
            (let [input (<! in)]
             ➊(if (= 3 input)
                (do (>! out "hot dog")
                    (recur (dec hc)))
                (do (>! out "wilted lettuce")
                    (recur hc))))
           ➋(do (close! in)
                (close! out)))))
    [in out]))
```

这里有很多代码，但策略是直接的。新函数`hot-dog-machine-v2`允许你指定`hot-dog-count`。在➊的 go 块内，只有当数字 3（意思是三块钱）被放在\`in'通道上时，它才会派发热狗；否则，它派发枯萎的生菜，这绝对不是热狗。一旦一个进程采取了输出，热狗机进程就会带着更新的热狗数量循环回来，并准备再次接收钱。

当机器进程的热狗用完时，该进程就会在➋处_关闭_通道。当你关闭一个通道时，你就不能再对它执行 put，而且一旦你从一个关闭的通道上取走所有的值，任何后续的取值都将返回 "nil"。

让我们来试试清单 11-1 中的升级版热狗机，把钱和口袋里的棉絮放进去。

```
(let [[in out] (hot-dog-machine-v2 2)]
  (>!! in "pocket lint")
  (println (<!! out))

  (>!! in 3)
  (println (<!! out))

  (>!! in 3)
  (println (<!! out))

  (>!! in 3)
  (<!! out))
; => wilted lettuce
; => hotdog
; => hotdog
; => nil
```

1. 清单 11-1. 与一个健壮的热狗售货机进程交互

首先，我们尝试了 "口袋里的棉絮 "这一招，得到了打蔫的生菜。接下来，我们两次投入 3 美元，两次都得到一个热狗。然后，我们试图再投入 3 美元，但这被忽略了，因为通道已经关闭；数字 3 没有被放在通道上。当我们试图从 "出 "通道取钱时，我们得到的是 "零"，这也是因为该通道是关闭的。你可能会注意到`hot-dog-machine-v2`的几个有趣的细节。首先，它在同一个 go 块中做了一个 put 和一个 take。这并不罕见，这也是创建进程_管道的一种方法：只要让一个进程的_入_通道成为另一个进程的_出\*通道。下面的例子就是这样做的，把一个字符串通过一系列的进程进行转换，直到最后一个进程打印出这个字符串。

```
(let [c1 (chan)
      c2 (chan)
      c3 (chan)]
  (go (>! c2 (clojure.string/upper-case (<! c1))))
  (go (>! c3 (clojure.string/reverse (<! c2))))
  (go (println (<! c3)))
  (>!! c1 "redrum"))
; => MURDER
```

在本章的最后，我将会有更多关于进程管道以及如何使用它们来代替回调的内容。

回到清单 11-1! 另一件需要注意的事情是，热狗机在你处理完它所发放的东西之前，不会接受更多的钱。这允许你建立类似于状态机的行为模型，其中通道操作的完成会触发状态转换。例如，你可以认为自动售货机有两个状态。_准备接收钱_和_发放_物品\*。插入钱和取走物品会触发这两者之间的转换。

## alts

core.async 函数`alts!!`可以让你使用一个操作集合中第一个成功的通道操作的结果。我们在第 198 页的 "延迟 "中用延迟和 Future 做了类似的事情。在那个例子中，我们把一组头像上传到一个头像分享网站，并在第一张照片上传时通知头像所有者。下面是你如何用`alts!!`做同样的事情。

```
(defn upload
  [headshot c]
  (go (Thread/sleep (rand 100))
      (>! c headshot)))

➊ (let [c1 (chan)
      c2 (chan)
      c3 (chan)]
  (upload "serious.jpg" c1)
  (upload "fun.jpg" c2)
  (upload "sassy.jpg" c3)
➋   (let [[headshot channel] (alts!! [c1 c2 c3])]
    (println "Sending headshot notification for" headshot)))
; => Sending headshot notification for sassy.jpg
```

在这里，`upload`函数接收一个头像和一个频道，并创建一个新的进程，该进程将随机睡眠一段时间（模拟上传），然后将头像放到频道上。从➊开始的`let`绑定和`upload`函数调用应该是有意义的：我们创建了三个通道，然后用它们来执行上传。

事情在➋处变得有趣。`alts!!`函数需要一个通道的向量作为其参数。这就好比说，"试着在这些通道上同时做一个阻塞性的拍摄。一旦取值成功，返回一个向量，其第一个元素是取值，第二个元素是获胜的通道"。在这个例子中，与_sassy.jpg_相关的通道首先收到了一个值。如果你想获取它们的值并对它们进行处理，其他通道仍然可用。`alts!!`所做的只是从第一个有值的通道中获取一个值；它并不触及其他通道。

`alts!!`的一个很酷的方面是，你可以给它一个_timeout 通道_，它等待指定的毫秒数，然后关闭。这是一个优雅的机制，可以为并发操作设置一个时间限制。下面是你如何在上传服务中使用它。

```
(let [c1 (chan)]
  (upload "serious.jpg" c1)
  (let [[headshot channel] (alts!! [c1 (timeout 20)])]
    (if headshot
      (println "Sending headshot notification for" headshot)
      (println "Timed out!"))))
; => Timed out!
```

在这种情况下，我们将超时设置为 20 毫秒。因为上传没有在这个时间段完成，我们得到了一个超时消息。

你也可以使用`alts!!`来指定 put 操作。要做到这一点，在你传递给`alts!!`的向量内放置一个向量，就像本例中的➊。

```
(let [c1 (chan)
      c2 (chan)]
  (go (<! c2))
➊   (let [[value channel] (alts!! [c1 [c2 "put!"]])]
    (println value)
    (= channel c2)))
; => true
; => true
```

这里你创建了两个通道，然后创建了一个进程，等待对`c2`进行处理。你提供给`alts!!`的向量告诉它，"尝试对`c1'进行取舍，并尝试将`"put!"`放在`c2'上。如果在`c1`上的取值首先完成，返回其值和通道。如果在`c2`上的投放先完成，如果投放成功，返回`true`，否则返回`false`。" 最后，`value`的结果（是`true`，因为`c2`的通道是开放的）打印出来，显示返回的通道确实是`c2`。

像`<!!`和`>!!`一样，`alts!!`有一个停车的选择，`alts!`，你可以在 go 块中使用它。 `alts!`是一个很好的方法，可以对一组通道中的哪一个进行投入或取出的选择。它仍然执行放和取，所以使用停放或阻塞变量的理由同样适用。

这就涵盖了 core.async 的基础知识! 本章的其余部分解释了协调进程的两种常见模式。

## 队列

在第 202 页的 "滚动你自己的队列 "中，你写了一个宏，让你对 Future 进行排队。进程让你以一种更直接的方式使用类似的技术。假设你想从一个网站上获得一堆随机的报价，并把它们写到一个文件中。你想确保每次只有一个报价被写入文件，这样文本就不会被交错，所以你把你的报价放在一个队列中。下面是完整的代码。

```
(defn append-to-file
  "Write a string to the end of a file"
  [filename s]
  (spit filename s :append true))

(defn format-quote
  "Delineate the beginning and end of a quote because it's convenient"
  [quote]
  (str "=== BEGIN QUOTE ===\n" quote "=== END QUOTE ===\n\n"))

(defn random-quote
  "Retrieve a random quote and format it"
  []
  (format-quote (slurp "http://www.braveclojure.com/random-quote")))

(defn snag-quotes
  [filename num-quotes]
  (let [c (chan)]
    (go (while true (append-to-file filename (<! c))))
    (dotimes [n num-quotes] (go (>! c (random-quote))))))
```

函数`append-to-file`、`format-quote`和`random-quote`有文档说明它们的作用。`snag-quotes`是发生有趣工作的地方。首先，它创建一个通道，在产生报价的进程和消费报价的进程之间共享。然后，它创建了一个使用 "while true "来创建一个无限循环的进程。在循环的每一次迭代中，它等待一个报价到达`c`，然后将其追加到一个文件中。最后，`snag-quotes`创建一个`num-quotes`数量的进程来获取一个引号，然后把它放在`c`上。如果你评估`(snag-quotes "quotes" 2)`并检查你启动 REPL 的目录中的_quotes_文件，它应该有两个引号。

```
=== BEGIN QUOTE ===
Nobody's gonna believe that computers are intelligent until they start
coming in late and lying about it.
=== END QUOTE ===

=== BEGIN QUOTE ===
Give your child mental blocks for Christmas.
=== END QUOTE ===
```

这种排队方式与第 9 章中的例子不同。在那个例子中，每个任务都是按照其创建的顺序来处理的。在这里，每个获取报价的任务是按照它完成的顺序来处理的。在这两种情况下，你都要确保每次只有一个报价被写入文件。

## 用进程管道逃离回调地狱

在没有通道的语言中，你需要用 "回调 "来表达 "当_x_发生时，做_y_"的想法。在像 JavaScript 这样的语言中，回调是一种定义代码的方式，一旦其他代码完成就会异步执行。如果你使用过 JavaScript，你可能已经花了一些时间在_回调地狱_中沉溺。

它被称为回调地狱的原因是，在回调层之间很容易产生不明显的依赖关系。它们最终会共享状态，使得在回调被触发时很难推理整个系统的状态。你可以通过创建一个流程管道来避免这种令人沮丧的结果。这样一来，每个逻辑单元都生活在自己独立的进程中，逻辑单元之间的所有通信都通过明确定义的输入和输出通道进行。

在下面的例子中，我们创建了三个通过通道连接的无限循环进程，将一个进程的_输出_通道作为管道中下一个进程的_输入_通道。

```
(defn upper-caser
  [in］
  (let [out (chan)] (让 [out (chan)])
    (go (while true (>! out (clojure.string/upper-case (<! in))))))
    out))

(defn reverser
  [in］
  (let [out (chan)] (go (while true (>!
    (go (while true (>! out (clojure.string/reverse (<! in))))))
    out))

(defn printer
  [in］
  (go (while true (println (<! in))))))

(def in-chan (chan))
(def upper-caser-out (upper-caser in-chan))
(def reverser-out (reverser upper-caser-out))
(Printer reverser-out)

(>！！in-chan "redrum")
; => MURDER

(>!! in-chan "repaid")
; => DIAPER
```

通过使用这样的流程处理事件，推理整个数据转换系统的各个步骤就更容易了。你可以查看每个步骤并理解它的作用，而不必参考之前可能发生的事情或之后可能发生的事情；每个过程就像一个纯函数一样容易推理。

## 额外资源

Clojure 的 core.async 库在很大程度上受到 Go 的并发模型的启发，它是基于 Tony Hoare 在_Communicating Sequential_ _Processes_中的工作，可在\*[http://www.usingcsp.com/](http://www.usingcsp.com)。\*

Go 的共同创造者 Rob Pike 有一个很好的关于并发的演讲，可在\*[Google I/O 2012 - Go 并发模式 - YouTube](https://www.youtube.com/watch?v=f6kdp27TYZs)\*。

ClojureScript，也被称为浏览器的最佳选择，使用 core.async。不再有回调的地狱! 你可以在\*[https://github.com/clojure/clojurescript](https://github.com/clojure/clojurescript%3C/span%3E)\*了解 ClojureScript 的情况。

最后，在\*[clojure.core.async - core.async 1.2.599-SNAPSHOT API documentation](http://clojure.github.io/core.async/)\*查看 API 文档。

## 总结

在本章中，你了解了 core.async 如何允许你创建并发进程，以响应通道上的 put 和 take 通信事件。你了解了如何使用`go`和`thread`来创建并发进程，通过停放和阻塞来等待通信事件。你还学习了如何通过使一个进程的_出_通道成为另一个进程的_入_通道来创建进程管道，以及这如何使你写的代码比嵌套回调更容易理解。最后，你思考了你是否只是一台花哨的热狗售货机。
