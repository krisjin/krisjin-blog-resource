title: 为什么需要JVM?
date: 2015-03-30 23:07:47
categories: Java
tags: [java]
---

Java程序可以跨平台，这是你在许多书或文件早就看过的描述，但是跨平台是怎麽一回事？在这之前，你得先了解不能跨平台是怎麽一回事。

其 实对于电脑而言，它只认识一种语言，也就是0101序列所组合而成的指令。当你使用的是C/C++等之类的高阶语言撰写程式时，其实这些语言，都是比较贴 近人类可阅读的文法，精确点来说，大部份就是比较接近英文文法的语言。这是为了方便人类阅读及撰写。电脑其实看不懂C/C++这类语言。<!--more-->

你要有个「翻译员」将你写的C/C++程式，翻译为电脑看得懂的0101序列指令，这个翻译员就是所谓的编译器。

![](/img/compiler.png)


问题在于，每个平台所认识的0101序列并不一样。在Windows上也许某个指令是0101，在Linux下也许是1010，因此不同的平台必须使用不同 的编译器来翻你的程式，而在Windows平台上编译好的程式，也不能直接拿到Linux等其它平台上执行，而必须经过重新编译的动作，让编译器将你的程 式翻译为该平台可以执行的指令。

![](/img/c-compiler.png)

由于每个平台的特性不同，可引用的程式库（Library）不同，也许你的程式还得作修改，才可以在另一个平台上编译执行。这很不方便，这表示如果你在Windows设计了一款游戏，想要卖给Linux的使用者，就得再花费一些功夫。

Java 也是个高阶语言，要让电脑执行你撰写的程式，也是得透过编译器的翻译。不过，Java编译时，并不直接翻译为相依于某平台的0101指令，而是翻译为中介 格式的位元码（byte code）。Java 的原始码副档名为*.java，经过编译器翻译过后，会变成*.class的位元码。如果想要执行这个位元码档桉，目标平台上必须安装有JVM（Java Virtual Machine）。JVM会将位元码翻译为平台相依的语言。

![](/img/java-compiler.png)

不同的平台必须安装该平台相依的JVM。这就好比你讲中文（*.java），Java编译器帮你翻译为英文（*.class）。之后该英文文件，到当地之后，再由当地看得懂英文的人翻译为当地的语言。

所以JVM所担任的职责之一，就是当地的翻译员，将位元码档桉翻译为当时作业系统看得懂的0101序列。不过这不是最重要的，基本上如果只是要翻译员的话，直译器（Interpreter ）就办得到了。

JVM有个很重要的观念就是：「对于Java程式而言，其实它只认识一种作业系统（或说是一种机器），这个系统叫作JVM，而对于JVM而言，位元码档桉就是它的可执行档桉！也就是副档名为.class的档桉。Java程式理想上，并不用理会真正执行于哪个平台之上，它只要知道如何执行于JVM之上就可以了，至于JVM实际上如何与底层平台作沟通，则是JVM自己的事！」这个观念非常的重要，对于往后釐清所谓PATH变数与CLASSPATH变数，有非常大的帮助。