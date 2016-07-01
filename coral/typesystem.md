# 类型系统

## 1 简介

**类型系统**最基本的目的是减少程序运行期异常。当程序运行时有这个特性，我们说这门语言是**type sound.**
在开发阶段，类型系统在语言定义方面提供了概念上的工具来判断代码的合理性。常见的语言定义中通常没有指定该语言的类型系统。同一种语言在不同编译器上有不同的类型系统。
另外很多语言都是type unsound，允许程序崩溃即使它已经通过**typechecker**检查的。理想情况是类型系统应该是所有类型编程语言的一部分。这样类型系统就能基于详细的
规格，让整个语言变成type sound的。

开篇我们介绍了一些术语：typing，execution error和相关概念。我们讨论我们想象的特性和类型系统的好处，回顾下为什么类型系统能成为主流。当我们谈运行时概念的时候，我们通常避免使用单词*type*和*typing*
。比如我们用动态检查替代动态类型，避免使用通用但难懂的术语，比如强类型(strong typing)

在第二小节，我们解释了类型系统的常见概念。

### Execution errors

运行错误最明显的表现就是不期望的软件错误出现，比如错误的指令或者错误的内存引用。更细微的运行错误比如数据错误并且不能立刻发现。这是软件错误，比如除零和空值运算。这些都是类型系统无法检查出来的。这是类型系统缺乏的东西，
如果没有这些，那么软件就不会出错，所以我们必须小心翼翼的定义什么是类型。

### Typed and untyped languages

程序变量在运行期间能用一个范围值。这个范围的顶部叫做这个变量的**type**。能给变量类型的语言叫**typed languages**.

不限制变量范围的的语言叫**untyped languages**:他们没有类型或者只有一个通用的类型来定义所有值。在这些语言中运算符能接受不符合类型的参数:结果可能是任意值,错误,或者无法确定的异常.纯的λ-calculus是无类型语言最典型的例子:
仅有的操作符是函数，所有的值都是函数，所以操作永远不会失败...

类型系统是类型语言不可分割的一部分，被用来跟踪表达式中变量的类型。类型系统被用来判断程序是否是**well bahaved**(后面讲到)

类型语言中，不管语法里是否出现类型，类型系统都是存在的。如果语法中存在类型则这种语言是**explicitly typed**,否则是**implicitly typed**.没有一门主流的语言是纯粹的隐类型。但是ML和Haskell写代码的时候可以省略类型，因为类型系统会自动
给代码加上类型。

### Execution errors and safety

介绍一种区分两种运行错误的方法: 一种是导致计算立即中断，另一种不太容易觉察，但是之后会造成行为异常。前者叫做**trapped errors**,后者叫做**untrapped errors**

untrapped errors最简单的例子就是访问一个非法地址。比如数组下标越界。关于trapped errors的例子有处零和访问非法地址：计算立即中断(在大多数电脑架构中)

如果一段代码不会造成untrapped errors，则这段代码是*安全*的。如果一门语言里所有的代码片段都是安全的，就叫做**safe language**。安全语言可以减少了大量的执行错误，但是有一条可能没注意。无类型语言通过运行时检查保证安全性。类型语言通过静态检查
检查程序中潜在的不安全因素。这个功能就叫做**static checks**

尽管安全性是程序最重要的特性，很少有一门类型语言只关注于减少untrapped errors。类型语言的目标是排除大量的trapped errors和untrapped errors。下一节我们讨论这些问题。

### Execution errors and well-behaved programs

在已存在的语言中，我们也需要设定一个execution errors的子集叫做**forbidden errors**. forbidden errors必须包含所有的untrapped errors且加上trapped errors的子集.
如果一段程序没有造成forbidden errors,那么这段代码就是**good behavior**。这段代码就是安全的。一门语言中的所有代码都是好的行为，那么这门语言是**strongly checked**的。
因此，一门强检查语言有以下特点：

+ 没有untrapped error发生(安全的)
+ 没有属于forbidden error的trapped error发生
+ 其他trapped error可能会发生，但那是程序员需要避免的。

类型语言通过静态检查来保证程序在运行之前不会出错。这种检查过程叫做**typechecking**.执行这个检查过程的算法叫做类型检查器。通过程序检查器检查的程序是**well typed**,否则是**ill typed**。
静态检查语言有:ML,Java和Pascal(Pascal有一些不安全特性)

无类型语言对好的行为和安全性有不同的处理方式--通过充足的运行时检查排除forbidden errors。(比如他们可能会检查所有的数组边界和所有的除法操作，当forbidden errors发生的时候，生成可恢复的异常)。
这种检查过程叫做**dynamic checking**；LISP就是这样一门语言。他的强检查既没有静态检查也没有类型系统

即使是静态检查的语言也通常需要在运行时进行测试以保证安全性。比如数组越界就需要动态的测试。不是说静态检查语言就能检查出所有问题的。

一些语言可以依靠他们的静态类型结构进行复杂的动态测试。比如java的instanceof会通过对象的运行时类型进行判断。这些语言(有点不合理)仍然用静态检查，部分原因是动态类型测试是定义在静态类型系统的基础之上的。

### Lack of safety

在我们的思维中，well behaved 程序是安全的。安全性是一种简单但是比good behavior更重要的特性。类型系统的主要目标是确保语言的安全性和保证程序运行过程中能够排除那些untrapped errors。然而大多数类型系统设计之初就是用来保证good behavior特性
和隐式的安全性。因此类型系统的目的一般都是通过区分well typed 和ill typed 来保证整个程序都处在good behavior中。

但是真实情况是，静态检查语言根本无法保证安全性。因为他们的forbidden errors中没有包含untrapped errors。所以这种语言只能叫**weakly checked**(在这里也可以叫弱类型): 有的不安全行为能检测到，有的却不行...

大多数无类型语言必须保证安全性(eg:lisp)。其他的那些缺乏编译时和运行时检查的语言就显得有点不好用了。注意下图:
![safty](https://raw.githubusercontent.com/useroriented/useroriented.github.io/master/images/2015-6-30/1.png)
