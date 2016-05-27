A Survey on Reactive Programming
============================
原文链接: [A Survey on Reactive Programming](http://soft.vub.ac.be/Publications/2012/vub-soft-tr-12-13.pdf)

响应式编程最近变得越来越流行，非常适合事件驱动和命令式编程。它提出了一个抽象的概念来表达基于时间变化的值(time-varying)和自动管理依赖。甚至已经内嵌进了许多语言中，比如：haskell，scheme，js，java，.net等等。这份调查对目前响应式编程的6个特性进行了分类：
representation of time-varying values，evaluation model，lifting operations，multi directionality，glitch avoidance，and support for distribution。调查之后发现，在响应式编程领域里，任然存在公开的挑战。举个例子，multi directionality 只有少量语言支持，并且不会自动跟踪time-varying的依赖。

#### 简介
现在的程序越来越重交互，被程序里的各种事件驱动以及其他外部环境。于是事件驱动在这种持续交互系统中发挥了巨大作用，处理事件，执行任务，更新状态和显示数据。这类程序最响应式的部分就是GUI了，因为它总是需要响应多种事件(比如：鼠标点击，键盘按下，触摸等等)
这种程序很难用传统编程方式写，因为判断和控制外部事件到达顺序是几乎不可能的。当状态改变之后，程序员需要手动修改其他依赖于它的模块。手动修改很复杂而且容易出错。用常见的编程方案(比如：设计模式和事件驱动编程)，这种程序一般都会有一个异步回调的概念(asynchronous callbacks)。不爽的是，合理的使用回调对一些有非常有经验的程序员来说也是很困难的事情，当有大量的代码块操作相同的数据的时候，执行顺序变得更加不可预测。一份最新的关于Adobe desktop applications的分析中指出，事件回调导致了一半的bug。回调管理的困难之处相信大家都很清楚就是回调地狱(s Callback Hell)。出于减少程序员负担的目的，我们需要一种专门的语言抽象来管理事件回调逻辑，就像管理状态一样。通常callback不会有返回值，还需要执行一些副作用代码来修改程序状态。因此，我们需要一种专用的编程思想来处理这些事件回调，来维护这些状态。

响应式编程就是这种解决方案，它非常适合用来开发事件驱动程序。rp提供一种抽象概念把整个程序都当做外部事件的一种响应，然后编程语言自动管理时间流(理论上支持并发)，数据和计算依赖。这么做的好处就是程序员不需要担心事件顺序和计算依赖。因此，rp语言抽象了基于时间的管理，就像gc基于内存的管理一样。这种自动管理数据依赖的特性在电子表格(spreadsheet)程序中得到最好的体现，在终端用户(end-user)编程语言中得到大量的使用

这篇rp范例是基于同步数据流编程范例的基础上的，但是有松散的时间(real-time)约束.它介绍了两个概念：用行为来表达的随时间变化的连续值和事件用事件来表达的离散值。另外，它允许数据流的结构是动态的(i.e. 数据流结构能在运行时随时间改变)和支持高阶数据流。大多数rp的研究都起源于Fran。一款函数式语言在90年代晚期，非常适合用来图形和交互型媒体程序。许多语言的库和拓展里考虑支持rp。

这篇文章提供了rp研究的调查和rp最近的发展。我们对已存在的6种rp特性进行分类，他们分别是： representation of time-varying values, evaluation model, lifting operations, multi-directionality, glitch avoidance, and support for distribution。我们之后将讨论技术和算法在这些方案中的表现。从这些分类中，我们发现一个问题就是这份研究报告仍然需要整理下。另一面，我们发现multi-directionality仅仅被小部分语言支持，并且不会自动追踪数据流依赖。另一个问题是当我们用rp来描述异步和事件驱动程序的时候， glitch avoidance不能保证是否使用了最新技术。随着命令式越来越松散，我们相信rp需要进一步探索进而替代松散的环境。

#### 2. REACTIVE PROGRAMMING

rp是一种基于“随事件变化的连续值”和”改变的传播”的概念的编程范式。它让开发者更专注想做的事，让程序自己决定什么时候去做，从而促进了声明式事件驱动程序开发。在这种范例下，model中的状态自动改变，通过依赖计算高效的传播。下面用一个例子来介绍改变是怎么传播的。
	下面是一个普通的加法运算：
		var1 = 1
		var2 = 2
		var3 = var1 + var2

在传统命令式编程中，变量var3的值是根据var1和var2的和来的，这里是即使var1或者var2分配了一个新的值之后，var3还是3.在响应式编程中，var3的值永远是动态更新的。换句话说，当var1或者var2的值更新之后，var3的值自动重新计算。这是rp最关键的概念。值随着时间改变，当依赖变了之后，重新计算。在rp学术中，变量var3的意思是说它依赖于var1和var2.下图就是这个意思。



![A Survey on Reactive Programming](https://raw.githubusercontent.com/useroriented/useroriented.github.io/master/images/2015-3-28/111.png)








##### 2.1 Distinguishing Features of Reactive Programming Languages
	
rp的两个区别性的特性：行为和事件

行为。在rp文献中，行为指时变值。行为随着时间改变而改变。最简单的例子，行为就是时间自己。大部分rp语言都有原生的行为来表达时间。其他行为当做原始时间的函数则容易理解多了。举个例子，假如一个行为的值是”10time”，可以当做”seconds* 10”

事件。事件指的是改变值的流，和行为会随时间不断改变不同，事件在时间里在离散的点里触发。(e.g. 键盘按下，改变位置).他们可能同时出现，也可能一个呈现出另一个。比如行为，事件是一等值和可组合的。大部分语言提供了原生的运算符来绑定事件和过滤事件流

#### 3. TAXONOMY

这节重点讨论rp的6大特性。

##### 3.1 Basic Abstractions

原始运算符(eg. assignment)和原始值(eg. number)在命令式语言里都是基础抽象，基础抽象在响应式语言中是响应式的。大多数语言都有行为(持续时变值)和事件(基于时间值的流)，我们在2.1里面提到过。这种抽象通常是可组合的且适合rp编程。
		用来描述持续改变值的”行为”提出了一个重要的实现难题。”持续”的本质说明当内部时钟速度增长的时候，实现难度也在增加了。在Fran和Yampa中，持续行为的值会被延迟计算，感谢Haskell的惰性求值。在非惰性求值语言中，行为相对于程序员来说是持续出现的，因为他们在任何时间点都需要一个值，但是需要定期获取。这个特性让Fran和Yampa能提供专用的操作来处理持续改变值，比如integral和derivative，这是其他语言所没有的。这在模型设计领域非常重要。

##### 3.2 Evaluation Model
从程序员的角度来看，改变的传播应该是自动的。这就是rp的本质。改变值应该自动传播到所有依赖去。当事件源产生了一个事件之后，依赖应该接收到这个事件，并且重新计算。在语言层面，设计的时候需要想好谁是事件源。所以，有可能是事件源”推”数据到它的依赖(consumer)，也有可能是依赖从事件源(producer)”拉”数据。正如图二看到的，有两种求值模式：

###### 3.2.1  Pull-based.
在 pull base 模式下，计算需要的值需要从数据源pull。传播被新数据驱动(demand-driven).第一个用这种模式的有Fran。这种模式让计算更加灵活，比如需要新值的时候才去pull。就像haskell的惰性求值。

###### 3.2.2 Push-based. 
在这种模式下，数据源产生新数据，然后push到依赖去。传播是被新数据驱动(data-driven)，而不是需求驱动。通常是调用注册的回调或者方法






![A Survey on Reactive Programming](https://raw.githubusercontent.com/useroriented/useroriented.github.io/master/images/2015-3-28/222.png)








###### push vs pull 
他们都有各自的有点和缺点。比如pull-based在随时间而改变的时间值的rp系统中就非常顺手。另外，惰性语言用pull-based的方式在行为初始化上有好处。他们的自然值都是等需要的时候才计算，没有显式的初始化。在push-based模式中，程序员必须显式的初始化行为来确保值产生了，因为他们使用的是立即求值代码。
	push-based 适合需要立即响应的响应式系统。它还有个问题就是求值会产生glitch，我们会在下一节讨论。整合两种模式可以有push-based模式的高效和低延迟，也可以获得pull-based模式的基于需求的灵活性。整合两种模式已经在Lula系统和Fran最近版本中得到证实

#### 3.3. Glitch Avoidance

Glitch Avoidance是另一个需要被考虑的问题。仅仅在push模式下有可能发生。
var1 = 1
var2 = var1 * 1
var3 = var1 + var2

  这个例子中，var2总是等于var1，var3等于2倍的var1.当var1等于1，var2等于1，var3等于2.如果var1变成2，var2的值应该是2，var3是4.然后在一个简单的rp模型中，把var1改成2有可能让表达式var1+var2在var1*1之前先重新计算。所以var3在某一时刻有可能为3，这是不对的。最后，var1*1重新计算了赋予了var2新值，然后var3也重新计算了，这个时候var3才为4.
	大多数rp语言会整理表达式顺序来避免出现这种错误。确保表达式总是在依赖求值之后求值。一个高效的rp模型中如果值没有改变不应该重新计算，如果新值和之前的值相同也不应该计算。

#### 3.4. Lifting Operations
当rp内嵌到宿主语言里(不管是库还是语言扩展的方式),不管是语言中运算符(eg. + *)还是用户定义的函数或者方法都必须转换成对行为的操作。把普通函数转换成对能操作行为的函数称之为lifting。
	Lifting是一个双重的概念：转换函数类型签名(包括参数类型和返回类型)，注册依赖关系图到程序数据流里面。在下面的定义中，我们假设函数有一个行为参数。
```				
lift : f(T) → f lifted(Behaviour < T >)
```
这里T是某个类型，Behaviour是behaviour类型，并且对应了类型T。lifting把不能接收Behavior类型参数的函数f转换成能接收behaviour类型参数的函数f lifted。

##### 3.4.1 Relation Between Typing and Lifting.
lifting 有一些不同的方法产生。在开始讨论之前，非常有必要了解语言的语义化和函数类型签名之间的相互作用。在静态类型语言中(haskell，java)，函数或者方法不能直接操作行为。通常程序员需要显式的lift操作和方法来确保类型安全(或者函数与方法接收行为参数.eg. 他们的参数通常是静态类型Behaviour)。在有些静态语言中，lift也不是那么必要，因为他们提供了一种叫重载的机制来处理行为问题。

说到动态语言，是一种能把行为传递给函数为不需要显式的提升到安全类型的语言。lifting通常会隐式的发生。不过有时候原生运算符有可能接收到不期望的类型参数。在动态语言中，原生运算符需要被适当的重载到lifted版本(比如，用代码转换技术生成lifted运算符)

##### 3.4.2. Classification of Lifting Strategies.

这小节我们分类了支持lifting语言实现lifting的不同方式。

Implicit lifting. 在这种模式下，普通函数被行为接收后，自动转换成lifed版本。它让rp编程更加通熟易懂，程序员们可以自由的使用事先定义好的运算符来操作行为.

						f(b1) → f lifted(b1)

可以看到函数f接收一个行为b1，然后f自动变成了lifted版本。这种方式适合动态语言。

Explicit lifting. 这种模式下，语言会提供各种连接符把普通函数提升到lifted版本。

				lift(f)(b1) → f lifted(b1)

可以看到函数f被lift连接符显式的提升到lifted版本。这种方式适用于静态语言。rp针对特殊领域，提供了一套丰富的重载单元来操作行为。对于重载，我们在这里仍然把它分类为显示提升。原生运算符可以被重载成lifted版本来解决问题

Manual Lifting. 这种模式下编程语言没有lifting 运算符，程序员需要手动获取当前时变值，然后传给普通函数。

					f(b1) → f(current value(b1))

先获取时变值b1的当前值，然后传给了普通函数f。语言不会提供first-class behaviours，需要手动把值放到一个个容器中，然后硬编码数据流依赖。

#### 3.5. Multidirectionality
另一个rp特性就是传播可以单方向或者多方向。多方向的话，值都是互相传播的。举个例子：F = (C *1.8) + 32 这是一个把华氏度转换成摄氏度的公式。不管是F还是C变了，另一个变量都会跟着改变。

#### 3.6. Support for Distribution

这个特性是关于rp是否支持分布式。这个特性允许你在多个节点在通过计算和数据创建依赖。举个例子：var3 = var1 + var2，这里var，var2，var3有可能在不同节点。一些交互式程序(e.g. web程序，app程序等等)变得越来越松散，这种需求也变得客官起来。然而困难的是，由于松散编程的特性，我们很难保持依赖的连贯性(比如延迟和网络状况)，我们在第五节讨论这个问题所面临的挑战。

#### 4.LANGUAGE SURVEY
我们调查了15种rp语言。大多数语言里fp只是语言里的附属功能。一些rp语言被发展成了frp语言，比如haskell和scheme

##### 4.1. The FRP Siblings
rp语言提供了可组合的抽象(行为和事件).另外，它还提供了原生的连接符来组合事件，提供了switching combinators 支持动态配置数据流和高阶数据流。
一个重要的点是，这些语言有时用有点不同的术语来表示行为和事件。
rp语言里，函数的参数变了之后，返回值也会跟着改变。因为参数变了，函数会自动重新执行。FRP允许程序员用声明式的写法。例如我们需要在屏幕鼠标位置画个圆(draw-circle mouse-x mouse-y)。每当mouse-x或者mouse-y改变的时候，draw-circle自动重新执行，然后更新圆的位置

*************************************中间是各种语言分析，这里略过*************************************

#### 5. OPEN ISSUES AND POSSIBLE SOLUTIONS

从上面的6大分类中我们发现rp仍然有些难题。就像我们在第三节描述的那样，只有很少的语言支持multidirectionality和松散编程，这方面仍然需要继续探索。

##### 5.1 Multidirectionality

常规frp中，数据流都是单向传播的。经过本次调查发现，多向约束可行，即Radul&Sussman 的Propagators 和 Coherence。他们都是rp家族中的一员，但是没有其他兄弟语言的行为和事件源。

在图形和用户交互领域，TBAG 是一门类似于frp的语言。它是面向图形编程的前身。TBAG支持多方向和行为与事件源的概念。然而TBAG并没有一等的行为和事件源。相反，程序员必须显式的区分3D场景中几个对象之间的关系。对这样的对象执行操作时,约束解释器试图满足所有的条件通过对所有相关对象进行转换。如果有断言冲突导致所有的断言失败了，就会有运行时错误抛出来。这些对象对于普通的C++操作是不可变的，他们必须在函数式场合操作，每次返回一个新的对象。之前的对象就被垃圾回收了，新对象就是新的时变行为。在预处理阶段，预处理这些图形和数学操作和生成重载版本。这种方式让TBAG处理一些特殊问题还算不错，这也是为什么Fran会选择haskell这门fp语言来处理frp问题的原因。Fran被用于更广泛的问题领域(eg.更普遍的时间处理)，包括一等行为，事件源和数据流求值策略(单向数据流)

Functional Hybrid Modelling(FHM)作为一种建模语言提出了一个相似的方案：不是直接整合multi-direction到行为和事件源里，而是整合到Yampa的信号函数和switching结构中

TBAG和FHM让时变值之间的关系变得透明起来。这是不同于其他兄弟语言的地方：行为和事件源都是一等值，依赖都是隐式的处理。仍然存在的一个问题是，在传统场景下rp能否使用multi-direction。

##### 5.2. Distributed Reactive Programming

在网络请求和延时执行上，很多松散式的系统都会借助事件这玩意。比如发出了一个异步的ajax请求的订阅发布系统。异步交流解耦了需要交流的部分：他们不需要时刻保持着连通性。但是异步调用不能立刻给调用者返回一个结果，而是在响应之后给调用者一个事件.因此，rp非常适合复杂的松散编程。


#### 6. CONCLUSIONS

rp非常适合开发事件驱动程序。我们整理了rp的6大特性：the basic abstractions for representing time-varying values, evaluation model, lifting operations, multi-directionality, support for distribution, and glitch avoidance.
	我们发现multi-directionality特性没有被大量的支持，即使是一些传统的rp语言。由此可见multi-directionality还有很多路要走。还有就是有越来越多的rp被用来开发交互松散式的程序(比如web程序和p2p手机应用)。蛋疼的是，正如我们在第五节看到的那样，rp的连接符和松散会给我们带来更多的glitches.因此，把rp和distributed programming协调好是未来的一大研究方向
