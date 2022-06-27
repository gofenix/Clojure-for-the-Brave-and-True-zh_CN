# 如何使用 Emacs，一个优秀的 Clojure 编辑器

在你掌握 Clojure 的过程中，你的编辑器将是你最亲密的盟友。我强烈建议使用 Emacs，但你当然也可以使用任何你想要的编辑器。如果你不遵循本章中关于 Emacs 的详尽说明，或者你选择使用一个不同的编辑器，那么至少值得投入一些时间来设置你的编辑器，以便与 REPL 一起工作。我推荐的两个在社区中受到好评的替代品是[Cursive](https://cursive-ide.com)和[Nightcode](https://sekao.net/nightcode/)。

我推荐 Emacs 的原因是，它提供了与 Clojure REPL 的紧密集成，这使你可以在写作时立即尝试你的代码。这种紧密的反馈回路在学习 Clojure 和以后编写真正的 Clojure 程序时都很有用。Emacs 也很适合与任何 Lisp 方言一起工作；事实上，Emacs 是用一种叫做 Emacs Lisp（elisp）的 Lisp 方言编写的。

在本章结束时，你的 Emacs 设置将看起来像图 2-1。

![](https://www.braveclojure.com/assets/images/cftbat/basic-emacs/emacs-final.png)

图 2-1: 使用 Clojure 的典型 Emacs 设置：一边是代码，另一边是 REPL。

为了达到这个目的，你将从安装 Emacs 开始，设置一个适合新人的 Emacs 配置。然后你将学习基础知识：如何打开、编辑和保存文件，以及如何使用基本的键绑定与 Emacs 进行交互。最后，你将学习如何实际编辑 Clojure 代码并与 REPL 进行交互。

## 安装

你应该使用 Emacs 的最新主要版本，即 Emacs 24，用于你工作的平台。

* **OS X**从\*[http://emacsformacosx.com](http://emacsformacosx.com)\*安装 vanilla Emacs 作为一个 Mac 应用程序。其他选项，如 Aquamacs，应该是为了使 Emacs 更 "像 Mac"，但从长远来看是有问题的，因为它们的设置与标准 Emacs 有很大的不同，以至于很难使用 Emacs 手册或跟随教程。
* **Ubuntu**按照\*[https://launchpad.net/\~cassou/+archive/emacs](https://launchpad.net/\~cassou/+archive/emacs)\*上的说明。
* **Windows**你可以在\*[Index of /gnu/emacs/windows](http://ftp.gnu.org/gnu/emacs/windows/)_找到一个二进制文件。在你下载并解压最新版本后，你可以在_bin\runemacs.exe\*下运行 Emacs 可执行文件。

安装完 Emacs 后，打开它。你应该看到类似图 2-2 的东西。

![](https://www.braveclojure.com/assets/images/cftbat/basic-emacs/emacs-fresh.png)

图 2-2：当你第一次打开 Emacs 时显示的屏幕

欢迎加入 Emacs 的崇拜者！你让 Richard Stallman 感到骄傲!

## 配置

我创建了一个库，里面有为 Clojure 配置 Emacs 所需的所有文件，可在[https://github.com/flyingmachine/emacs-for-clojure/archive/book1.zip](https://github.com/flyingmachine/emacs-for-clojure/archive/book1.zip)。

注意：这些工具一直在更新，所以如果下面的说明对你不起作用，或者你想使用最新的配置，请阅读[GitHub - flyingmachine/emacs-for-clojure](https://github.com/flyingmachine/emacs-for-clojure/)上的说明。

按以下步骤删除你现有的 Emacs 配置并安装对 Clojure 友好的配置。

1. 关闭 Emacs。
2. 删除_\~/.emacs_或_\~/.emacs.d_，如果它们存在的话。(Windows 用户，你的 Emacs 文件可能在_C:\Users\your\_user\_name\AppData\Roaming\*。因此，举例来说，你可以删除_C:\Users\jason\AppData\Roaming.emacs.d\*）。这是 Emacs 寻找配置文件的地方，删除这些文件和目录将确保你从一个干净的地方开始。
3.
   1. 下载上面列出的 Emacs 配置压缩文件并解压。其内容应该是一个文件夹，_emacs-for-clojure-book1_。运行 mv path/to/emacs-for-clojure-book1 \~/.emacs.d。
4. 打开 Emacs。

当你打开 Emacs 时，你可能会看到大量的活动，因为 Emacs 正在下载一堆有用的软件包。一旦这些活动停止，继续前进，退出 Emacs，然后再打开它。(如果你没有看到任何活动，那也没关系！退出并重新启动 Emacs。退出并重新启动 Emacs 只是为了好玩）。在你这样做之后，你应该看到一个像图 2-3 那样的窗口。

![](https://www.braveclojure.com/assets/images/cftbat/basic-emacs/emacs-configged.png)

图 2-3：Emacs 在安装了你可爱的新配置后的样子

现在我们已经设置好了一切，让我们来学习如何使用 Emacs!

## Emacs 逃逸舱口

在我们进入有趣的东西之前，你需要知道一个重要的 Emacs 键绑定：ctrl-G。这个键绑定可以退出你试图运行的任何 Emacs 命令。所以，如果事情进展不顺利，按住 ctrl，按 G，然后再试一次。它不会关闭 Emacs，也不会使你失去任何工作；它只是取消你当前的行动。

## Emacs 缓冲区

所有的编辑都发生在 Emacs 的\*缓冲区中。当你第一次启动 Emacs 时，一个名为 "_scratch_"的缓冲区被打开。Emacs 总是在窗口的底部显示当前缓冲区的名称，如图 2-4 所示。

![](https://www.braveclojure.com/assets/images/cftbat/basic-emacs/emacs-buffer-name.png)

图 2-4：Emacs 会一直显示当前缓冲区的名称。

默认情况下，`*scratch*`缓冲区处理括号和缩进的方式对 Lisp 开发来说是最理想的，但对编写纯文本却很不方便。让我们创建一个新的缓冲区，这样我们就可以在不发生意外的情况下进行游戏。要创建一个缓冲区，请这样做。

1. 按住 ctrl 键并按下 X 键。
2. 2.松开 ctrl 键。
3. 按 B 键。

我们可以用一个更紧凑的格式来表达同样的序列。 **C-x b**。

执行这个按键序列后，你会在应用程序的底部看到一个提示，如图 2-5 所示。

![](https://www.braveclojure.com/assets/images/cftbat/basic-emacs/emacs-buffer-prompt.png)

图 2-5: 迷你缓冲区是 Emacs 提示你输入的地方。

这个区域被称为_minibuffer_，它是 Emacs 提示你输入的地方。现在它正在提示我们输入一个缓冲区的名称。你可以输入一个已经打开的缓冲区的名称，也可以输入一个新的缓冲区名称。输入 emacs-fun-times，然后按回车键。现在你应该看到一个完全空白的缓冲区，可以直接开始输入。你会发现，按键的工作方式与你所期望的差不多。字符在你输入时出现。上、下、左、右方向键会像你所期望的那样移动你，而回车键会创建一个新行。

你还会注意到，你不会突然长出浓密的 Unix 胡须或穿上 Birkenstocks（除非你一开始就有）。这应该有助于缓解你对使用 Emacs 的任何恐惧感。当你玩够了之后，继续前进，通过输入**C-x k enter**来杀死( kill )当前的缓冲区。(这可能会让人吃惊，但 Emacs 实际上是很暴力的，它直接使用了_ kill _这个词）。

现在你已经杀死了`emacs-fun-times`缓冲区，你应该回到`*scratch*`缓冲区。一般来说，你可以用**C-x b**创建任意多的新缓冲区。你也可以用同样的命令在缓冲区之间快速切换。当你以这种方式创建一个新的缓冲区时，它只存在于内存中，直到你把它保存为一个文件；缓冲区不一定有文件支持，创建一个缓冲区也不一定会创建一个文件。让我们来学习一下关于文件的工作。

## 与文件一起工作

在 Emacs 中打开文件的键位是**C-x C-f**。注意，当你按下 X 和 F 时，你需要按住 ctrl。导航到_\~/.emacs.d/customizations/ui.el_，它可以自定义 Emacs 的外观和你与它的互动方式。Emacs 在一个与文件名相同的新缓冲区中打开文件。让我们转到第 37 行，去掉前面的分号，取消注释。它将看起来像这样。

```clojure
(setq initial-frame-alist '((top . 0) (left . 0) (width . 120) (height . 80))
```

然后改变 "width "和 "height "的值，它们为活动窗口设置_字符_的尺寸。通过改变这些值，你可以设置 Emacs 窗口在每次启动时以某种尺寸打开。一开始可以试试小一点的，比如 80 和 20。

```clojure
(setq initial-frame-alist '((top . 0) (left . 0) (width . 80) (height . 20))
```

现在用下面的键绑定来保存你的文件。 **C-x C-s**。你应该在 Emacs 的底部得到一个信息，如`写了/Users/snuffleupagus/`.emacs.d/customizations/ui.el\`。你也可以尝试使用你在其他应用程序中使用的键绑定来保存你的缓冲区（例如，ctrl-S 或 cmd-S）。你下载的 Emacs 配置应该允许这样做，但如果不允许，也没什么大不了的。

保存文件后，退出 Emacs 并再次启动它。我敢打赌，它非常小! 请看我在图 2-6 中的例子。

![](https://www.braveclojure.com/assets/images/cftbat/basic-emacs/tinemacs.png)

图 2-6：你可以配置 Emacs，使其在每次打开时都设置高度和宽度。

同样的过程要进行几次，直到 Emacs 以你喜欢的尺寸开始。或者再把这几行注释掉就可以了（在这种情况下，Emacs 将以其默认的宽度和高度打开）。如果你完成了对_ui.el_的编辑，你可以用**C-x k**关闭其缓冲区。无论如何，你已经完成了在 Emacs 中保存第一个文件的工作 如果发生了一些疯狂的事情，你可以按照[第 13 页的 "配置"](https://www.braveclojure.com/basic-emacs/#Anchor)中的指示来使 Emacs 重新工作。

如果你想创建一个新的文件，只需使用**C-x C-f**并在迷你缓冲区中输入新文件的路径。一旦你保存了缓冲区，Emacs 就会按照你输入的路径用缓冲区的内容创建一个文件。

让我们来回顾一下。

1. 在 Emacs 中，编辑是在缓冲区\*.\*中进行的。
2. 要切换到一个缓冲区，使用**C-x b**并在 minibuffer\*.\*中输入缓冲区的名称。
3. 要创建一个新的缓冲区，使用**C-x b**并输入一个新的缓冲区名称。
4. 要打开一个文件，使用**C-x C-f**并导航到该文件。
5.
   1. 要将缓冲区保存到文件中，使用**C-x C-s**。
6. 要创建一个新的文件，使用**C-x C-f**并输入新文件的路径。当你保存缓冲区时，Emacs 将在文件系统中创建文件。

## 键绑定和模式

你已经走了很长一段路了! 你现在可以像一个非常基本的编辑器一样使用 Emacs。如果你需要在服务器上使用 Emacs，或者被迫与 Emacs 书呆子配对，这应该能帮助你度过难关。

然而，要想真正有成效，了解一些关于键绑定的\*关键细节对你来说是很有用的（哈哈！）。然后我将介绍 Emacs 模式。之后，我将介绍一些核心术语，并介绍一些超级有用的键绑定。

### Emacs 是一个 Lisp 解释器

术语_键绑定_源于这样一个事实：Emacs 将_键击打_绑定到_命令_上，而这些命令只是 elisp 函数（我将交替使用_命令_和_函数_）。例如，**C-x b**被绑定到函数`switch-to-buffer`。同样地，**C-x C-s**与`save-file`绑定。

但 Emacs 甚至比这更进一步。甚至像**f**和**a**这样简单的按键也被绑定到一个函数上，在这个例子中是 "self-insert-command"，是向你正在编辑的缓冲区添加字符的命令。

从 Emacs 的角度来看，所有的函数都是平等的，你可以重新定义所有的函数，甚至像`save-file`这样的核心函数。你可能不会_想要_重新定义核心函数，但你可以。

你可以重新定义函数，因为就其核心而言，Emacs 只是一个 Lisp 解释器，恰好加载了代码编辑功能。Emacs 的大部分内容都是用 elisp 编写的，所以从 Emacs 的角度来看，`save-file`只是一个函数，就像`switch-to-buffer`和你能运行的几乎所有其他命令一样。不仅如此，你创建的任何函数都被当作内置函数来对待。你甚至可以用 Emacs 来执行 elisp，在它运行时修改 Emacs。

使用强大的编程语言修改 Emacs 的自由是 Emacs 如此灵活的原因，也是为什么像我这样的人对它如此疯狂。是的，它有很多表面上的复杂性，可能需要花时间去学习。但 Emacs 的底层是 Lisp 的优雅简洁，以及随之而来的无限的可修补性。这种可修补性并不局限于创建和重新定义函数。你还可以创建、重新定义和删除键绑定。从概念上讲，按键绑定只是一个查询表中的条目，它将按键与函数联系起来，而这个查询表是完全可修改的。

你也可以使用**M-x**函数名称来运行命令，而不需要特定的键绑定（例如，**M-x** save-buffer）。 _M_代表_meta_，这是一个现代键盘不具备的键，但在 Windows 和 Linux 上被 Map 到 alt，在 Mac 上则是 option。 **M-x**运行`smex`命令，它提示你要运行的另一个命令的名称。

现在你已经了解了键的绑定和功能，你将能够理解什么是模式以及它们是如何工作的。

### 模式

Emacs 的_模式_是一个键绑定和功能的集合，它被打包在一起，帮助你在编辑不同类型的文件时提高工作效率。(模式也可以做一些事情，比如告诉 Emacs 如何做语法高亮，但这是次要的，我不会在这里介绍。)

例如，当你在编辑一个 Clojure 文件时，你会想加载 Clojure 模式。现在我正在写一个 Markdown 文件并使用 Markdown 模式，它有很多专门用于 Markdown 工作的有用的键绑定。在编辑 Clojure 时，最好有一套 Clojure 专用的键绑定，比如**C-c C-k**将当前的缓冲区加载到 REPL 中并进行编译。

模式有两种类型。 _主要_模式和_次要_模式。Markdown 模式和 Clojure 模式是主要模式。主要模式通常在你打开文件时由 Emacs 设置，但你也可以通过运行相关的 Emacs 命令明确地设置模式，例如用**M-x** clojure-mode 或**M-x** major-mode。每次只有一种主要模式是激活的。

主要模式是针对某种文件类型或语言的 Emacs，而次要模式通常提供对各种文件类型都有用的功能。例如，abbrev 模式 "根据预先定义的缩写定义自动展开文本"（根据 Emacs 手册[1.](https://www.braveclojure.com/basic-emacs/#footnote-5680-1)）。你可以同时激活多个次要模式。

你可以在\*模式行中看到哪些模式处于活动状态，如图 2-7 所示。

![](https://www.braveclojure.com/assets/images/cftbat/basic-emacs/emacs-mode-line.png)

图 2-7：模式行显示哪些模式是活动的。

如果你打开一个文件，而 Emacs 没有为它加载一个主要模式，那么这个模式很有可能存在。你只需要下载它的软件包。说到这个 ... .

### 安装软件包

许多模式都是以_包_的形式发布的，这只是存储在包仓库中的 elisp 文件的捆绑。你在本章开始时安装的 Emacs 24，使浏览和安装软件包变得非常容易。 **M-x** package-list-packages 会显示几乎所有可用的软件包；只要确保你先运行**M-x** package-refresh-contents 就能得到最新的列表。你可以用**M-x** package-install 来安装软件包。

你也可以通过加载你自己的 elisp 文件或你在网上找到的文件来定制 Emacs。Emacs 初学者指南》（见\*[http://www.masteringemacs.org/articles/2010/10/04/beginners-guide-to-emacs/](http://www.masteringemacs.org/articles/2010/10/04/beginners-guide-to-emacs/)\*）在文章底部的 "加载新包 "一节中对如何加载自定义文件有很好的描述。

## 核心编辑术语和键绑定

![](https://www.braveclojure.com/assets/images/cftbat/basic-emacs/pirate.png)

如果你只想把 Emacs 当做一个文本编辑器来使用，你可以完全跳过这一节！但你将会错过很多东西。但你将会错过一些好东西。在这一节中，我们将介绍 Emacs 的关键术语；如何选择、剪切、复制和粘贴文本；如何选择、剪切、复制和粘贴文本（看到我做了什么吗？ 哈哈哈！）；以及如何有效地在缓冲区内移动。

要想开始，请在 Emacs 中打开一个新的缓冲区，并将其命名为_jack-handy_。然后输入以下杰克-汉迪的语录。

```clojure
If you were a pirate, you know what would be the one thing that would
really make you mad? Treasure chests with no handles. How the hell are
you supposed to carry it?!

The face of a child can say it all, especially the mouth part of the
face.

To me, boxing is like a ballet, except there's no music, no
choreography, and the dancers hit each other.
```

用这个例子来试验本节中的导航和编辑。

### 点

如果你一直在关注，你应该在你的 Emacs 缓冲区看到一个橘红色的矩形。这就是_游标_，它是_点_的图形表示。点是所有魔法发生的地方：你在点上插入文本，大多数编辑命令都是与点有关的。即使你的光标看起来是在一个字符的上面，但点实际上是位于该字符和前一个字符之间。

例如，把你的光标放在_If you were a pirate_中的_f_上。点就位于_I_和_f_之间。现在，如果你使用**C-k**，从字母_f_开始的所有文字将消失。 **C-k**运行命令`kill-line`，它\*杀了当前行中从点开始的所有文字（我将在后面讲到更多的杀戮）。用\*\*C-/\*\*撤销这一改变。另外，尝试用正常的操作系统的键绑定来撤消；这也应该是有效的。

## 移动

你可以像其他编辑器一样用方向键来移动点，但许多键的绑定可以让你更有效地进行导航，如表 2-1 所示。

1. 表 2-1: 文本导航的键位绑定

| 关键字                 | 描述                                           |
| ------------------- | -------------------------------------------- |
| **C-a**             | 移动到行首。                                       |
| **M-m**             | 移动到该行的第一个非空格字符。                              |
| **C-e**             | 移动到行尾。                                       |
| **C-f**             | 向前移动一个字符。                                    |
| \*_C-b_ \*向后移动一个字符。 |                                              |
| **M-f**             | 向前移动一个字（我经常用这个）。                             |
| **M-b**             | 向后移动一个字（我也经常用这个）。                            |
| **C-s**             | Regex 搜索当前缓冲区内的文本，并移动到它。再按一次**C-s**，移到下一个匹配。 |
| **C-r**             | 与**C-s**相同，但以反向方式搜索。                         |
| **M-<**             | 移到缓冲区的开头。                                    |
| **M->**             | 移动到缓冲区的末端。                                   |
| **M-g g**           | 转到该行。                                        |

来吧，在你的\*\*\*缓冲区里试试这些键的绑定!

### 带区域的选择

在 Emacs 中，我们并不_选择_文本。我们创建_区域_，并通过用**C-spc**（ctrl-spacebar）设置\*标记来实现。然后，当你移动点时，标记和点之间的所有东西都是区域。这与 shift 选择文本的基本目的非常相似。

例如，在你的\*\*\*\*缓冲区里做以下事情。

1. 转到文件的开头。
2. 使用**C-spc**。
3. 使用**M-f**两次。你应该看到一个高亮的区域，包括_If you_。
4.
   1. 按退格键。这将会删除_If you_。

使用标记而不是 Shift 选择文本的一个很酷的事情是，你可以在设置标记后自由使用 Emacs 的所有移动命令。例如，你可以设置一个标记，然后用**C-s**来搜索缓冲区内几百行的一些文本。这样做将创建一个非常大的区域，而你就不必紧张地按住 Shift 键了。

区域还可以让你把一个操作限制在缓冲区的有限区域内。试试这个。

1. 创建一个区域，包括_孩子的脸可以说明一切_。
2.
   1. 使用**M-x**替换字符串，用_head_替换_face_。

这将在当前区域内进行替换，而不是在点之后的整个缓冲区内进行替换，这是默认行为。

### 杀戮和杀戮环

在大多数应用程序中，我们可以_切割_文本，这只是轻微的暴力。我们还可以_复制_和_粘贴_。剪切和复制将选择的内容添加到剪贴板上，而粘贴则将剪贴板上的内容复制到当前的应用程序中。在 Emacs 中，我们采取杀人的方法，_杀_区域，把它们加入到_杀圈_。当你知道你正在浪费数千字节的文本时，你不觉得_勇敢_和_坚强_吗？然后我们可以_yank_，在点上插入最近杀死的文本。我们还可以_复制_文本到杀戮环，而不需要真正杀死它。

为什么要用这些病态的术语呢？嗯，首先，当你听到有人在 Emacs 中谈论杀死东西时，你不会感到害怕。但更重要的是，Emacs 允许你做一些典型的剪切/复制/粘贴剪贴板功能集所不能做的工作。

Emacs 在杀戮环上存储了多个文本块，你可以循环使用它们。这很酷，因为你可以通过循环来找回你很久之前杀死的文本。让我们来看看这个功能的实际应用。

1. 在第一行的_Treasure_这个词上创建一个区域。
2. 2.使用**M-w**，它与 "杀死-循环-保存 "命令绑定。一般来说，**M-w**就像复制一样。它将该区域添加到杀戮环中，而不从你的缓冲区中删除它。
3. 将指针移到最后一行的_choreography_字样上。
4. 使用**M-d**，它与`kill-word`命令绑定。这将把_choreogra\*\*phy_添加到杀戮环中，并将其从你的缓冲区中删除。
5. 使用**C-y**。这将把你刚刚杀死的文字_choreogra_phy\*，插入到点的位置。
6. 使用**M-y**。这将删除_choreography_，并拉出杀戮环上的下一个项目，_Treasure_。

你可以在表 2-2 中看到一些有用的杀戮/拉扯键的绑定。

1. 表 2-2：杀戮和拉扯的键位绑定 文本

| 关键字     | 描述           |
| ------- | ------------ |
| **C-w** | 杀戮区域。        |
| **M-w** | 复制区域到杀戮环。    |
| **C-y** | 绞刑。          |
| **M-y** | 在拉动后循环使用杀伤环。 |
| **M-d** | 杀字。          |
| **C-k** | 杀行。          |

### 编辑和帮助

表 2-3 显示了一些额外的、有用的编辑键绑定，你应该知道如何处理间距和扩展文本。

1. 表 2-3：其他有用的编辑键绑定方式

| 关键字      | 描述                        |
| -------- | ------------------------- |
| \*_Tab_  | 缩进行。                      |
| **C-j**  | 新行和缩进，相当于回车后的 tab。        |
| **M-/**  | 嬉皮士扩展；循环浏览点之前的文本可能的扩展方式。  |
| \*_M-\*_ | 删除点周围的所有空格和制表符。(我经常使用这个。) |

Emacs 也有很好的内置帮助。表 2-4 中显示的两个键绑定将为你提供良好的服务。

1. 表 2-4：内置帮助的键位绑定

| 关键字                                                           | 描述 |
| ------------------------------------------------------------- | -- |
| **C-h k** **键绑定** 说明与该键绑定的功能。为了使其发挥作用，你在输入**C-h k**后实际执行按键序列。 |    |
| **C-h f** \*描述功能。                                             |    |

帮助文本出现在一个新的\*窗口中，这个概念我将在本章后面介绍。现在，你可以通过按**C-x o q**关闭帮助窗口。

## 使用 Emacs 与 Clojure

接下来，我将解释如何使用 Emacs 来有效地开发一个 Clojure 应用程序。你将学习如何启动一个与 Emacs 相连的 REPL 进程，以及如何与 Emacs 窗口一起工作。然后，我将介绍大量有用的键绑定，用于求值表达式、编译文件和执行其他方便的任务。最后，我将向你展示如何处理 Clojure 的错误，并介绍 Paredit 的一些功能，这是一种可选的次要模式，对编写和编辑 Lisp 风格语言的代码很有用。

如果你想开始钻研 Clojure 代码，请务必跳过前面的内容！你可以在以后再回来。你可以稍后再回来。

### 开启你的 REPL

正如你在第 1 章中所学到的，REPL 允许你交互地编写和运行 Clojure 代码。REPL 是一个正在运行的 Clojure 程序，它给你一个提示，然后读取你的输入，求值它，打印结果，并循环返回到提示。在第 1 章中，你在终端窗口用`lein repl`启动了 REPL。在本节中，你将直接在 Clojure 中启动一个 REPL。

为了将 Emacs 连接到 REPL，你将使用 Emacs 软件包 CIDER，可在\*\[GitHub - clojure-emacs/cider: The Clojure Interactive Development Environment that Rocks for Emacs]（[https://github.com/clojure-emacs/cider/](https://github.com/clojure-emacs/cider/)）\*。如果你按照本章前面的配置说明，你应该已经安装了它，但你也可以通过运行**M-x**包-安装，输入 cider，然后按回车键来安装它。

CIDER 允许你在 Emacs 中启动一个 REPL，并为你提供键绑定，使你能更有效地与 REPL 进行交互。现在就去启动一个 REPL 会话吧。使用 Emacs，打开_clojure-noob/\*\*src/clojure\_noob/core.clj_文件，该文件是你在第一章中创建的。接下来，使用**M-x** cider-jack-in。这将启动 REPL 并创建一个新的缓冲区，在那里你可以与它进行交互。经过短暂的等待（应该不到一分钟），你应该看到类似图 2-8 的东西。

![](https://www.braveclojure.com/assets/images/cftbat/basic-emacs/cider-jack-in.png)

图 2-8：运行 M-x cider-jack-in 后你的 Emacs 应该是这样的

现在我们有两个窗口：我们的_core.clj_文件在左边打开，REPL 在右边运行。如果你从来没有见过 Emacs 像这样分成两半，不要担心！我将讲述 Emacs 是如何做到的。我一会儿会讲到 Emacs 是如何分割窗口的。同时，在 REPL 中尝试求值一些代码。键入以下加粗的行。当你按下回车键时，你应该看到打印在 REPL 中的结果，显示在每一行代码的后面。这时不要担心代码，我将在下一章中介绍所有这些功能。

```clojure
(+ 1 2 3 4)
; => 10
(map inc [1 2 3 4])
; => (2 3 4 5)
(reduce + [5 6 100])
; => 111
```

相当漂亮! 你可以像在第一章中使用`lein repl`那样使用这个 REPL。你还可以做更多的事情，但在这之前，我将解释如何在分屏 Emacs 中工作。

\###插曲。Emacs 的窗口和框架

让我们绕道来谈谈 Emacs 是如何处理框架和窗口的，并讨论一些与窗口有关的有用的键绑定方法。如果你已经熟悉了 Emacs 的窗口，请随意跳过这一部分。

Emacs 是在 1802 年左右发明的，所以它使用的术语与你习惯的略有不同。你通常所说的_窗口_，Emacs 称之为_框架_，而框架可以分割成多个_窗口_。分割成多个窗口允许你一次查看多个缓冲区。你在运行`cider-jack-in`时已经看到了这种情况（见图 2-9）。

![](https://www.braveclojure.com/assets/images/cftbat/basic-emacs/emacs-windows.png)

图 2-9：在 Emacs 中，一个框架包含有窗口。

表 2-5 显示了用于处理框架和窗口的几个键的绑定情况。

1. 表 2-5: Emacs 窗口的键位绑定

| 关键字       | 描述                                            |
| --------- | --------------------------------------------- |
| **C-x o** | 将光标切换到另一个窗口。现在试试这个，在你的 Clojure 文件和 REPL 之间切换。 |
| **C-x 1** | 删除所有其他窗口，框架中只留下当前窗口。这不会关闭你的缓冲区，也不会导致你失去任何工作。  |
| **C-x 2** | 分割框架的上方和下方。                                   |
| **C-x 3** | 并排分割框架。                                       |
| **C-x 0** | 删除当前窗口。                                       |

我鼓励你试试 Emacs 的窗口键绑定。例如，把你的光标放在左边的窗口，也就是有 Clojure 文件的那个，然后使用**C-x 1**。另一个窗口应该消失，而你应该只看到 Clojure 代码。然后做以下工作。

* 使用**C-x 3**将窗口再次并排分开。
* 使用**C-x o**来切换到右边的窗口。
* 使用**C-x b** _cider-repl_来切换到右边窗口的 CIDER 缓冲区。

一旦你做了一些实验，设置 Emacs，使它包含两个并排的窗口，左边是 Clojure 代码，右边是 CIDER 缓冲区，就像前面的图片一样。如果你有兴趣了解更多关于窗口和框架的知识，Emacs 手册中有大量的信息：见\*[http://www.gnu.org/software/emacs/manual/html\_node/elisp/Windows.html#Windows](http://www.gnu.org/software/emacs/manual/html\_node/elisp/Windows.html#Windows)\*。

现在你可以浏览 Emacs 窗口了，是时候学习一些 Clojure 开发的键绑定了

\###有用的键绑定的丰富内容

现在你已经准备好学习一些按键绑定，它们将揭示在 Clojure 项目中使用 Emacs 的真正力量。这些命令将使你只需按下几个简单的键就能求值、调整、编译和运行代码。让我们先来看看如何快速求值一个表达式。

在_core.clj_的底部，添加以下内容。

```clojure
(println "Cleanliness is next to godliness")
```

现在使用**C-e**导航到行尾，然后使用**C-x C-e**.文本`Cleanliness is next to godliness`应该出现在 CIDER 缓冲区，如图 2-10 所示。

![](https://www.braveclojure.com/assets/images/cftbat/basic-emacs/cider-eval-last-expression.png)

图 2-10：在 REPL 中从另一个缓冲区即时求值代码

绑定键**C-x C-e**运行`cider-eval-last-expression`命令。顾名思义，该命令将紧接在点之前的表达式发送到 REPL，然后由 REPL 求值该表达式。你也可以试试**C-u C-x C-e**，它打印出点之后的求值结果。

现在让我们试着运行我们在第一章中写的`-main`函数，这样我们就可以让全世界都知道我们是小茶壶。

在_core.clj_的缓冲区中，使用**C-c M-n M-n**。这个键绑定将命名空间设置为你当前文件顶部列出的命名空间，所以右边窗口的提示现在应该是`clojure-noob.core>`。我还没有详细介绍命名空间，但现在只要知道命名空间是一种组织机制，使我们能够避免命名冲突就足够了。接下来，在提示符下输入（-main）。REPL 应该打印出 "I'm a little teapot!"多么令人激动啊

现在让我们创建一个新函数并运行它。在_core.clj_的底部，添加以下内容。

```clojure
(defn train
  []
  (println "Choo choo!")
```

完成后，保存你的文件并使用**C-c C-k**在 REPL 会话中编译你的当前文件。(现在，如果你在 REPL 中运行 `(train)`，它将回显 `Choo choo!`。

当你还在 REPL 中时，试试**C-↑**（ctrl 加向上箭头键）。 **C-↑**和**C-↓**循环浏览你的 REPL 历史，其中包括你要求 REPL 求值的所有 Clojure 表达式。

Mac 用户注意：默认情况下，OS X 将**C-↑**、**C-↓**、**C-←**和**C-→**Map 为任务控制命令。你可以通过打开系统偏好设置，然后进入 Keyboard4Shortcuts4Mission Control 来改变你的 Mac 键绑定。

最后，试试这个。

1. 在 REPL 提示符下输入（-main）。注意没有结尾的括号。
2. 按**C-enter**。

CIDER 应该关闭小括号并求值表达式。这只是 CIDER 为处理这么多小括号而提供的一个很好的小便利。

CIDER 还有一些键的绑定，在你学习 Clojure 的时候非常好。按**C-c C-d C-d**将显示该符号下的文档，这可以大大节省时间。当你看完文档后，按**q**来关闭文档缓冲区。绑定的键\*\*M-.**将导航到点下符号的源代码，而**M-,\*\*将使你回到原来的缓冲区和位置。最后，**C-c C-d C-a**可以让你在函数名和文档中搜索任意的文本。当你不能完全记住一个函数的名字时，这是一个很好的方法来寻找它。

CIDER README（[_GitHub - clojure-emacs/cider: The Clojure Interactive Development Environment that Rocks for Emacs_](https://github.com/clojure-emacs/cider/))有一个全面的键绑定列表，你可以慢慢学习，但现在，表 2-6 和 2-7 包含了我们刚刚经历的键绑定的总结。

1. 表 2-6：Clojure 缓冲区的键绑定情况

| 键值              | 描述                          |
| --------------- | --------------------------- |
| **C-c M-n M-n** | 切换到当前缓冲区的命名空间。              |
| **C-x C-e**     | 求值紧邻点的表达式。                  |
| **C-c C-k**     | 编译当前缓冲区。                    |
| **C-c C-d C-d** | 显示点下符号的文档。                  |
| **M-. 和 M-,**   | 浏览到该点下的符号的源代码，并返回到原来的缓冲区。   |
| **C-c C-d C-a** | Apropros 搜索；在函数名和文档中查找任意文本。 |

1. 表 2-7: CIDER 缓冲区的键绑定方式

| 关键字                      | 描述            |
| ------------------------ | ------------- |
| \*\*C-**↑**, C-\*\***↓** | 循环浏览 REPL 历史。 |
| **C-enter**              | 关闭圆括号并求值。     |

### 如何处理错误

在这一节中，你将写一些有错误的代码，这样你就可以看到 Emacs 是如何反应的，以及你如何从错误中恢复并继续你的快乐之路。你将在 REPL 缓冲区和 _core.clj_ 缓冲区中进行这项工作。让我们从 REPL 开始。在提示符下，输入(map)并按回车。你应该看到类似图 2-11 的东西。

![](https://www.braveclojure.com/assets/images/cftbat/basic-emacs/cider-error.png)

图 2-11：这就是在 REPL 中运行坏代码时发生的情况。

正如你所看到的，在没有参数的情况下调用`map`会使 Clojure 失去理智--它在你的 REPL 缓冲区中显示一个\`ArityException'错误信息，并在你的左边窗口中填满文本，看起来像一个疯子的呓语。这些呓语就是_堆栈跟踪_，它显示了实际抛出异常的函数，以及哪个函数调用了_那个_函数，沿着函数调用的堆栈。

Clojure 的堆栈跟踪在你刚开始的时候可能很难解读，但经过一段时间后，你会学会从其中获得有用的信息。CIDER 通过允许你过滤堆栈痕迹来帮你一把，这可以减少噪音，这样你就可以将异常的原因锁定。`*cider-error*`缓冲区的第 2 行有 Clojure、Java、REPL、Tooling、Duplicates 和 All 等过滤器。你可以点击每个选项来激活该过滤器。你也可以点击每个堆栈跟踪行来跳到相应的源代码。

下面是如何关闭左边窗口中的堆栈跟踪。

1. 使用**C-x o**来切换到窗口。
2. 按**q**关闭堆栈跟踪，回到 CIDER。

如果你想再次查看错误，你可以切换到`*cider-error*`缓冲区。你也可以在尝试编译文件时得到错误信息。要看这个，请到_core.clj_缓冲区，写一些有错误的代码，然后进行编译。

1. 在结尾处添加`(map)`。
2. 使用**C-c C-k**进行编译。

你应该看到一个`*cider-error*`缓冲区，类似于你之前看到的那个。同样，按**q**关闭堆栈跟踪。

### Paredit

在 Clojure 缓冲区中写代码时，你可能已经注意到了一些意外的事情发生。例如，每当你输入一个左括号，一个右括号就会立即被插入。

这要归功于_paredit-mode_，这是一种次要的模式，它将 Lisp 的大量小括号从一种责任变成了一种资产。Paredit 确保所有的小括号、双引号和大括号都是封闭的，从而减轻了你那可恶的负担。

Paredit 还提供了键绑定功能，以轻松浏览和改变所有这些括号所创建的结构。在下一节中，我将介绍最有用的键绑定，但你也可以在\*[https://github.com/georgek/paredit-cheatsheet/blob/master/paredit-cheatsheet.pdf](https://github.com/georgek/paredit-cheatsheet/blob/master/paredit-cheatsheet.pdf)\*（在骗局中，红色管子代表点）查看全面的骗局表。

然而，如果你不习惯，paredit 有时会很烦人。我认为花点时间来学习它是非常值得的，但你可以随时用**M-x** paredit-mode 来禁用它，它可以切换该模式的开启和关闭。

下面的部分向你展示了最有用的键绑定。

#### Wrapping 和 Slurping

_Wrapping_用小括号包围点之后的表达式。 _Slurping_将结束的小括号移到右边包括下一个表达式。例如，假设我们用这个开始。

```clojure
(+ 1 2 3 4)
```

而我们想得到这个结果。

```clojure
(+ 1 (* 2 3) 4)
```

我们可以把`2`包起来，加一个星号，然后再把`3`溜走。首先，放置点，这里表示为一个垂直的管道，`|`。

```clojure
(+ 1 |2 3 4)
```

然后输入**M-(**，与_paredit-wrap-round_绑定，得到这个结果。

```clojure
(+ 1 (|2) 3 4)
```

加上星号和空格。

```clojure
(+ 1 (* |2) 3 4)
```

要在 "3 "上啧啧称奇，请按**C-→**。

```clojure
(+ 1 (* |2 3) 4)
```

这样就可以很容易地增加和扩展括号，而不必浪费宝贵的时间按住方向键来移动点。

#### Barfing

假设在前面的例子中，你不小心吐了四条。要解开它（也被称为_barfing_），将你的光标（`|`）放在括号内的任何地方。

```clojure
(+ 1 (|* 2 3 4))
```

然后使用**C-←**。

```clojure
(+ 1 (|* 2 3) 4)
```

Ta-da! 现在你知道如何随意扩展和收缩括号了。

#### 导航

在用 Lisp 方言写作时，你经常会遇到这样的表达式。

```clojure
(map (comp record first)
     (d/q '[:find ?post
            :in $ ?search
            :where
            [(fulltext $ :post/content ?search)
             [[?post ?content]]]]
          (db/db)
          (:q params))
```

对于这种表达式，快速从一个子表达式跳到下一个子表达式是很有用的。如果你把 point 放在开头的小括号之前，**C-M-f**会把你带到结束的小括号。同样，如果 point 紧跟在闭合小括号之后，**C-M-b**将带你到开头小括号。

表 2-8 总结了你刚刚学到的 Paredit 键的绑定。

1. 表 2-8：Paredit 键的绑定方式

| 关键字                  | 描述                             |
| -------------------- | ------------------------------ |
| **M-x** paredit-mode | 切换 paredit 模式。                 |
| **M-(**              | 括号内点后的表达式(paredit-wrap-round)。 |
| \*\*C-\*\*→          | Slurp;将结束的小括号向右移动，以包括下一个表达式。   |
| \*\*C-\*\*←          | Barf；将括号向左移动，排除最后一个表达式。        |
| \*_C-M-f_,\*_C-M-b_  | 移动到开头/结尾小括号。                   |

## 继续学习

Emacs 是历史最悠久的编辑器之一，它的追随者对它的热情往往接近狂热。一开始使用它可能会很别扭，但坚持下去，你会在一生中得到充分的回报。

每当我打开 Emacs 时，我都会感到受到鼓舞。就像一个工匠进入他的工作室一样，我感到一个可能性的领域在我面前打开。我感到这个环境的舒适，它随着时间的推移已经发展到完全适合我--各种各样的包和键绑定，帮助我日复一日地把想法变成现实。

在你继续你的 Emacs 之旅时，这些资源将帮助你。

* Emacs 手册_提供了优秀、全面的指导。每天早上花点时间看看它吧! 下载 PDF，在旅途中阅读。_[http://www.gnu.org/software/emacs/manual/html\_node/emacs/index.html#Top](http://www.gnu.org/software/emacs/manual/html\_node/emacs/index.html#Top)\*。
* \*《Emacs 参考卡》\*是一个方便的小抄。 [_http://www.ic.unicamp.br/\~helio/disciplinas/MC102/Emacs\_Reference\_Card.pdf_](http://www.ic.unicamp.br/\~helio/disciplinas/MC102/Emacs\_Reference\_Card.pdf)。
* Mickey Petersen 的_Mastering Emacs_是最好的 Emacs 资源之一。从阅读指南开始。 [_阅读指南-掌握 Emacs_](http://www.masteringemacs.org/reading-guide/) 。
* 对于更注重视觉效果的人，我推荐手绘的《如何学习 Emacs》。Emacs 24 或更高版本的初学者指南"，作者 Sacha Chua。 [_http://sachachua.com/blog/wp-content/uploads/2013/05/How-to-Learn-Emacs8.png_](http://sachachua.com/blog/wp-content/uploads/2013/05/How-to-Learn-Emacs8.png)。
* 只要按**C-h t**就可以看到内置的教程。

## 摘要

呜呼! 你已经覆盖了很多地方。你现在知道了 Emacs 作为一个 Lisp 解释器的真正性质。绑定键是执行 elisp 函数的快捷方式，而模式是绑定键和函数的集合。你学会了如何以自己的方式与 Emacs 互动，并掌握了缓冲区、窗口、区域、杀戮和拉动。最后，你学会了如何使用 CIDER 和 paredit 轻松地与 Clojure 工作。

有了这些来之不易的 Emacs 知识，现在是时候开始认真学习 Clojure 了

[1](https://www.braveclojure.com/basic-emacs/#footnote-5680-1-backlink) [_http://www.gnu.org/software/emacs/manual/html\_node/emacs/Minor-Modes.html_](http://www.gnu.org/software/emacs/manual/html\_node/emacs/Minor-Modes.html)。
