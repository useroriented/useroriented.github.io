# rxjs学习笔记

## 常见异步流分析

callback会很容易写出带有副作用的代码，而fp建议我们把函数参数拿出来。函数不过是一个仅仅包含输入和输出的黑盒。而黑盒更方便用于服用和组合，也更方便我们推断程序行为，即使是异步的

event emitter，如果你不在正确的时间监听事件，则会有丢失事件的风险。而且事件系统也有用到回调。还包括可读性问题和很难推理程序逻辑的问题，因为你需要担心异步的问题。

rxjs没有这些：

1. for和while里面很难写异步代码
2. 异步代码里错误处理是很棘手的，而且当你用try/catch之后代码会更错综复杂。还有当你需要实现retry逻辑，即使有其他库帮助你，你也很极难写得出来。
3. 无关的逻辑总是参合在一起。我们需要写出高复用和更模块化的组件，这样我们就需要写出松耦合的代码，同时也方便单元测试。
4. 你的代码越嵌套得深，逻辑越难理解。函数和其他变量和函数混在一起，可读性和复杂性都是个问题
5. 太过依赖闭包。很容易造成副作用。
6. 很难检测事件或者长时操作什么时候会出错，什么时候需要取消。比如一个http请求太长，是脚本问题还是服务器太慢？所以我们需要判断在多久之后能直接取消事件。但是如果你自己实现可取消的机制，也是非常有挑战性而且容易出错的。
7. 一个高质量的响应式设计必须要对用户在ui上的交互进行截流(throttle)，这样系统才不会负载。手动实现这个功能还是非常难做好的。在他们的作用域外调用函数访问数据会对整个系统造成不稳定性
8. 很少有人考虑过JavaScript程序的内存问题。特别是客户端代码。虽然浏览器在底层替我们做了很多事情。然而随着UI变得越来越大且丰富，我们注意到那些正在运行的事件监听器会造成内存泄露，而且增加浏览器需要处理数量从而增加了浏览器负担，
在以前的浏览器上更是如此。如今的JavaScript程序的复杂性和以前的程序已经有天壤之别了。

你当然可以用一些常见的技术，api或者其他库帮你解决这些问题，但是需要一些努力。一个可替代的方案就是es6的Promise

### Promise

promise允许你的函数解耦，增加可读性，甚至可以抛开try/catch，promise用到了continuation-passing style (CPS)。在这种规范下，你的callback调用栈会被隔离成单独的函数，然后作为参数传给每个正在运行的http请求，这样http请求就能
在它需要的时候"continue"传递和转换数据了。

promise的缺点也很明显，不能处理会产生超过一个值得数据源，比如鼠标移动或者文件流。Promise也缺少一个很重要的特性--延迟(debounce)和截流(throttle)，还有失败能retry的功能。然而promise最不可接受的缺点是，因为它是不可变的，所以promise没办法取消。
我们知道http是能被中断的(XmlHttpRequest)，但是通过promise就不行(fetch)。这种限制减少了promise的可用性，也让开发者们开始寻求一些其他框架。

不管事promise也好，事件发射机也好，本质上都是用不同的方式解决同样的问题。两者的使用场景有点区别。promise被用在单个返回值场景，比如http请求。事件发射机被用在多个值场景，比如鼠标点击处理。不是因为他们想这么做，而是他们的实现机制导致他们只适合这些场景。
然后开发者们就必须同时使用他们来达成开发目标，然后就会写出别扭和困惑的代码。

但是我们真正需要是把等待时间(latency)从我们的代码中抽象出来，这样不管多少数据和事件需要处理，我们都能保证我们的程序的可相应性和伸缩性。

![1.8](https://raw.githubusercontent.com/useroriented/useroriented.github.io/master/images/rxjs-in-action/82D7357F-9A38-4CA7-A964-8ED55963677D.png)

如上图，rxjs能把异步数据流当做一种简单的序列化步骤

我们需要整合事件发射机和promise的特性到一个抽象里，来解耦功能

## rxjs

rxjs是一个可以替代callback和promise的优雅的库。rxjs把所有的事件源都用一种编程模型解决了，不管是读文件，发请求，点击按钮，移动鼠标等等Ï。

### Stream

![1.9](https://raw.githubusercontent.com/useroriented/useroriented.github.io/master/images/rxjs-in-action/ADCC6346-036D-4E19-A314-C768A09906C8.png)

流是一种随时间改变的事件队列。你能订阅它，也能在管道中写函数，当事件触发的时候，你的函数就会被调用。一个比较出名的例子就是Excel表格。

Stream的理念是能接受任意值，在开始深入之前我们抽象一种数据类型，一种容器，叫做**Stream**。它能接受一个值:

```js
Stream(42);Ï
```

这个点上，流还没被使用，什么事也不会发生，除非有subscriber或者observer监听它。这和promise一创建就立即执行有很大不同。stream是lazy的数据类型。
这里42被放到stream中，而且至少会被传播到一个subscriber去。当接收到这个值之后，这条流就结束了。

rxjs算是一种优化过的Subject-Observer pattern。这种模式在很多程序中都有见过，比如一个mvc架构，view始终是监听model的。然而早期的observer模式
有些缺点：对observer不合理的操作会导致内存泄漏。

rxjs的灵感就来自这种模式在异步编程中的Publish-Subscribe系统，不过增加了一些其他特性，比如：信号--能描述stream什么时候完成，懒执行，取消，资源状态和处理结果。

我们甚至能传一串数字到stream中：

```js
Stream(1, 2, 3, 4, 5).subscribe (
   val => {
      console.log(val); //-> prints 1, 2, 3, 4, 5   
   }
);
```

或者一个数组

```js
Stream([1, 2, 3, 4, 5])
  .filter(num => (num % 2) === 0) ❶ 
  .map(num => num * num)  ❶ 
  .subscribe(
      val => {
        console.log(val); //-> prints 4, 16   
      }
);
```

上面的例子中，操作符出现在*Producer*创建之后和*Consumer*创建之前，这就是我们要说的管道。

![pipeline](https://raw.githubusercontent.com/useroriented/useroriented.github.io/master/images/rxjs-in-action/pipeline.png)

线性的冒泡事件(coral: 事件就是值本身),然后通过管道被转换。最终到达各个subscriber。

### Abstracting the notion of time from your programs

讲真，**time is of the essence**。异步流代码处理等待时间是最难的部分。callback和promise都因为自身的限制仍然处理不了这个问题。rxjs把**continuous sequences of events over time**的概念作为语言的一等公民
带给了JavaScript，最终成为了JavaScript的一个真正的事件子系统。也就意味着，rxjs*abstracts over time under the same programming model*，所以你能够写同步线性的改变你的数据。
这是非常亮的，你可以像处理一个数组一样处理一段鼠标事件了。

Stream就像你在现实世界中订阅了月刊杂志一样。我们订阅的是一个被事件分隔的杂志集合：每年有12份杂志，不过每个月收到一份。当我们收到杂志之后我们会产生一些行为(阅读或者扔掉)。
因为我们只在收到杂志之后做出行为，我们称这种行为为响应。

![magazine](https://raw.githubusercontent.com/useroriented/useroriented.github.io/master/images/rxjs-in-action/magazine.png)

rxjs允许你像订阅杂志一样订阅其他东西，比如从硬盘加载文件，处理用户输入，处理类似rss和twitter的实时服务。rxjs允许你用处理同步数据的方式处理异步事件流：

```js
Stream(loadMagazines('/subscriptions/magazines'))
  .filter(magazine => magazine.month === 'July')
  .subscribe(
      magazine => {
        console.log(magazine.title);
        //-> prints Dr. Dobbs "Composing Reactive Animations"   
      }
);
```

### Components of an Rx Stream

Rxjs stream由一些基础组件组成。你也许在其他地方知道了。这里我们正式介绍下他们是：

+ Producers
+ Consumers
+ Data pipeline
+ Time

#### Producers

Producers是我们的数据源，一条流至少需要一个数据源，作为一切的开始。实际上，producer可以从任何可以产生事件的的数据源创建(单个值，数组，鼠标点击，文件流).
在Observer模式中定义producers为Subject。但是在rxjs中，我们称之为Observables，表示可以被observed。

#### Consumers

Producer才是一半，我们还需要一个consumer接收事件和处理事件。当consumer开始监听producer，这条流才会开始push事件

#### DATA PIPELINE

rxjs高级的地方在于我们可以在事件从producer流向customer的途中操作和编辑事件。这就是操作符(Observable方法)所扮演的角色。操作数据意味着我们能让customer拿到期望中的值。
这么做在两个实体之间产生了*separation of concern*和模块性。这种设计原则在大型场景的JavaScript程序中很难实现，但是rxjs可以完成。

#### TIME

rxjs有个潜在的组件叫时间，之后会详细介绍。现在你只需要知道时间不一定是正常速度，如果你需要你可以让他变慢或者变快。幸运的是，如果你决定用rxjs，这就不是事。

这是这些组件具体的样子：

![compoennts](https://raw.githubusercontent.com/useroriented/useroriented.github.io/master/images/rxjs-in-action/components-of-rxjs.png)

你可以用这些写出你自己的响应式设计。但是rxjs不会强迫你只用一种范式。你可以充分组合利用他们创造出更有弹性更好维护的设计。

实际生产中，你可以用oop设计你的系统，也可以用frp驱动你的行为和事件。和oop那样把状态和数据存在变量或者集合里不一样，状态在rp里是暂时的，状态不会保存下来但是会流过订阅他的流。
这让事件处理容易推理和测试了。

### Exploring the advantages of RxJS

这章我们重点介绍如何用rxjs让你从异步代码中解脱出来，他们是：

+ Declarative design
+ Event handling and cancellation
+ Error handling
+ Testability

#### DECLARATIVE DESIGN

当前的异步解决方案都有个共同的问题，很难写出声明式的代码。语言被设计出来解决同步代码，加上异步非线性的本质，导致了代码能在函数中有序排列，却不用在调用时线性调用。因此我们需要跟踪当前
执行状态，然后有了callback和promise。rxjs从fp中给我们带来了简单和声明式设计。你将不再需要创建一个变量来跟踪回调处理，也不用再担心一不小心修改产生了外部状态这种副作用代码。此外，rxjs让我们
很轻松的管理多条数据流，过滤或者转换他们都变得很轻松。通过操作符把程序逻辑串起来，就像一条句子。可组合的特性也让创建高阶操作符变得轻松容易。合并流也让我们能设计出更复杂的行为。

#### EVENT HANDLING AND CANCELLATION

流总是有开始和结束。在流的生命周期中，当订阅者开始监听(subscription)或者停止监听producer的时候，我们需要知道发生了什么。

当Observable停止发送事件，或者已完成，rxjs会自动清理已分配的资源，这在避免事件监听泄露上相对于其他库有巨大的优势。另外，subscription对象是由observable生成的，包含了一个自毁开关。
你可以用它来取消在运行中的操作符和自动清理资源。

#### ERROR HANDLING

之前提到的，用同步编程设计的语言在异步的世界中不那么好使了。其中一种设计try/catch，很难围绕传统的callback机制来实现。开发者必须经常创建和储存错误信息以交给后面的逻辑处理，这种方式不够模块化，
这样反而更容易出错，并且有副作用。聪明的决定就是用rxjs。错误会在Observable里安全的传播到各自的observers中。错误发生时，你的逻辑会立即中断，这非常像try/catch。Promise在处理错误方面和rxjs非常相似。
但是promise不能惰性执行，也不能重新执行已reject的promise。

#### TESTABILITY

测试异步代码是每个程序员都很头疼的事情，尤其是代码里有timeout和interval的时候。在多个timer之间交互变得很难测试.rxjs提供了一种简单，干净和独创的方式对时间以来进行求值，并且通过抽象时间概念缩短了
单元测试时间--schedulers
