# 用 Leiningen 构建和开发

用任何语言编写软件都需要生成_工件_，即可执行文件或库包，用于部署或共享。它还涉及到管理依赖工件，也称为_依赖_，以确保它们被加载到你正在构建的项目中。Clojurists 中最流行的管理工件的工具是 Leiningen，本附录将告诉你如何使用它。你还将学习如何使用 Leiningen 来完全增强你的开发经验，使用_插件_。

## Artifact Ecosystem

因为 Clojure 托管在 Java 虚拟机（JVM）上，所以 Clojure 的工件是以 JAR 文件的形式分发的（在第 12 章有介绍）。Java 地已经有一个处理 JAR 文件的完整的工件生态系统，Clojure 也使用它。_神器生态系统_并不是一个官方的编程术语；我用它来指代用于识别和分发神器的一套工具、资源和惯例。Java 的生态系统是围绕着 Maven 构建工具发展起来的，由于 Clojure 使用这个生态系统，你会经常看到对 Maven 的引用。Maven 是一个巨大的工具，可以执行各种古怪的项目管理任务。值得庆幸的是，你不需要获得 Maven 学的博士学位就能成为一名有效的 Clojurist。你需要知道的唯一特征是，Maven 规定了一种识别 Clojure 项目所遵守的工件的模式，它还规定了如何在 Maven _仓库_中托管这些工件，Maven \*仓库只是存储工件以供分发的服务器。

### Identification

Maven 工件需要一个_组 ID_，一个_工件 ID_，以及一个_版本_。你可以在_project.clj_文件中为你的项目指定这些。以下是你在第一章创建的`clojure-noob`项目的_project.clj_第一行的内容。

```
(defproject clojure-noob "0.1.0-SNAPSHOT"
```

`clojure-noob`是你项目的组 ID 和工件 ID，`"0.1.0-SNAPSHOT"`是其版本。一般来说，版本是永久性的；如果你将一个版本为 0.1.0 的工件部署到存储库，你不能对该工件进行修改并使用相同的版本号进行部署。您需要改变版本号。(许多程序员喜欢 Semantic Versioning 系统，您可以在\*[Semantic Versioning 2.0.0 | Semantic Versioning](http://semver.org).\*中阅读到这一系统。） 如果你想表明该版本是一个正在进行的工作，并且你计划不断地更新它，你可以在你的版本号后面加上`-SNAPSHOT`。

如果你想让你的组 ID 与你的工件 ID 不同，你可以用斜线将两者分开，像这样。

```
(defproject group-id/artifact-id "0.1.0-SNAPSHOT"
```

通常，开发者会使用他们的公司名称或 GitHub 用户名作为组的 ID。

### 依赖

你的_project.clj_文件还包括一行看起来像这样的内容，它列出了你项目的依赖。

```
  :dependencies [[org.clojure/clojure "1.9.0"]]
```

如果你想使用一个库，使用与你命名项目时相同的命名模式将其添加到这个依赖 Vector 中。例如，如果你想轻松地处理日期和时间，你可以添加 clj-time 库，像这样。

```
  :dependencies [[org.clojure/clojure "1.9.0"]
            [clj-time "0.9.0"]]
```

下次你启动你的项目时，无论是通过运行它还是通过启动 REPL，Leiningen 都会自动下载 clj-time 并使其在你的项目中可用。

Clojure 社区创造了大量有用的库，寻找它们的好地方是\*[http://www.clojure-toolbox.com](http://www.clojure-toolbox.com)\*的 Clojure 工具箱，它根据项目的目的进行分类。几乎每一个 Clojure 库都在其 README 的顶部提供了它的标识符，使你很容易找出如何把它添加到你的 Leiningen 依赖项中。

有时你可能想使用一个 Java 库，但标识符并不那么容易获得。例如，如果你想添加 Apache Commons Email，你必须在网上搜索，直到你找到一个包含这样内容的网页。

```
<dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-email</artifactId>
            <version>1.3.3</version>
</dependency>
```

该 XML 是 Java 项目沟通其 Maven 标识符的方式。要把它添加到 Clojure 项目中，你需要修改`:dependencies`Vector，使其看起来像这样。

```
  :dependencies [[org.clojure/clojure "1.9.0"]
            [clj-time "0.9.0"]
            [org.apache.commons/commons-email "1.3.3"]]
```

主要的 Clojure 库是 Clojars（[_Clojars_](https://clojars.org)），主要的 Java 库是 The Central Repository（[_Maven Central Repository Search_](http://search.maven.org)），人们通常只把它称为_Central_，就像旧金山居民把旧金山称为_the city_一样。你可以使用这些网站来寻找库和它们的标识符。

要把你自己的项目部署到 Clojars，你所要做的就是在那里创建一个账户，然后在你的项目中运行`lein deploy clojars`。该任务会生成 Maven 工件所需的一切，包括 POM 文件（我就不多说了）和 JAR 文件，以便储存在仓库中。然后将它们上传到 Clojars。

### 插件

Leiningen 让你使用_插件_，这是一些在你写代码时能帮助你的库。例如，Eastwood 插件是一个 Clojure 检查工具；它可以识别写得不好的代码。你通常要在\*$HOME/.lein/profiles.clj_文件中指定你的插件。要添加 Eastwood，你要把_profiles.clj\*改成这样。

```
{:user {:plugins [[jonase/eastwood "0.2.1"]] ] 。}}
```

这就为你的所有项目启用了一个`eastwood'Leiningen任务，你可以在项目的根目录下用`lein eastwood'运行。

Leiningen 的 GitHub 项目页面有关于如何使用配置文件和插件的优秀文档，它包括一个方便的插件列表。

## 总结

本附录着重介绍了项目管理中那些重要但又难以了解的方面，比如什么是 Maven 以及 Clojure 与它的关系。它向你展示了如何使用 Leiningen 来命名你的项目，指定依赖关系，并部署到 Clojars。Leiningen 为软件开发任务提供了很多功能，但并不涉及实际编写代码。如果你想了解更多，请在网上查看 Leiningen 教程\*\[leiningen/TUTORIAL.md at stable - technomancy/leiningen - GitHub]（[https://github.com/technomancy/leiningen/blob/stable/doc/TUTORIAL.md](https://github.com/technomancy/leiningen/blob/stable/doc/TUTORIAL.md)）\*.
