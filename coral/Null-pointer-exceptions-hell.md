# Null pointer exceptions hell

原文链接: [http://dobegin.com/npe-hell](http://dobegin.com/npe-hell/)

## 简介

Null pointer exception (NPE) 在不同的编程语言里都有定义。比如C/C++的Null pointer dereference，java里的NullPointerException，C#和.NET的NullReferenceException，
还有一些脚本语言比如JavaScript的"undefined is not a function"

NPE是最常见的编程bug。因为一些流行的语言存在这些问题，所以用它们写出来程序基本上少不了这些磨人的bug。作为一个软件开发者，你必须每天处理这些NPE，
不然它们就像地雷一样埋进你的代码，静静的等待着你的客户被坑。NPE是一个"十亿美元的错误":
> This has led to innumerable errors, vulnerabilities, and system crashes, which have probably caused a billion dollars of
> pain and damage in the last forty years.
> Tony Hoare from the talk “billion dollar mistake“

## 例子

空值引用(null pointer dereference)的历史甚至比UNIX/C era还早。这些例子会用一些现在最流行的编程语言来写出NPE。

为了方便，我们直接set变量为null或者undefined。真正的编程环境中，NPE通常发生在对一些其他代码进行操作的时候产生，比如第三方库，框架和系统啊等等。
要想出错，只需要在用户输入阶段输入空，数据库字段改成空，以及设备状态为空等等。

### C/C++

参考以下代码:

```c++
QTimer *timer = nullptr;
timer->start(); // crash
```

运行上面的代码会看到如下结果

![null pointer error in c++](https://raw.githubusercontent.com/useroriented/useroriented.github.io/master/images/npe/npe_in_cpp.png)

### Java

```java
String name = null;
name.toLowerCase(); // crash
```

![null pointer error in java](https://raw.githubusercontent.com/useroriented/useroriented.github.io/master/images/npe/npe_in_java_jsp.png)

### C# .NET

```c#
string name = null;
name.ToLower(); // crash
```

![null pointer error in c#](https://raw.githubusercontent.com/useroriented/useroriented.github.io/master/images/npe/npe_in_aspnet.png)

## NPE和动态编程语言

流行的动态脚本语言没有pointer的概念，但是他们有references。空值引用通常会产生同样的结果。而且这些语言通常会有`undefined`的概念。
虽然技术上`undefined`和`null`是不同的，但是实际使用中都会产生NPE。所以脚本语言会有更多的NPE，因为`undefined`和`null`是一家...

![null-loves-undefined](https://raw.githubusercontent.com/useroriented/useroriented.github.io/master/images/npe/null-loves-undefined.png)

### Javascript

```js
var element = null;
element.getAttribute("id");
    // crash

var element = { getAttribute:null }
element.getAttribute("id");
    // crash
```

### Error in browser console:

![npe_in_javascript](https://raw.githubusercontent.com/useroriented/useroriented.github.io/master/images/npe/npe_in_javascript.png)

### php

```php
$name = null;
$name->getMessage(); // crash
```

![npe_in_php_xdebug](https://raw.githubusercontent.com/useroriented/useroriented.github.io/master/images/npe/npe_in_php_xdebug.png)

### Python

```python
name = None
name.lower() # crash
```

可爱的Django网页
![npe_in_django](https://raw.githubusercontent.com/useroriented/useroriented.github.io/master/images/npe/npe_in_django.png)

### Ruby

```ruby
name = nil
name.downcase # crash 
```

![npe_in_rails](https://raw.githubusercontent.com/useroriented/useroriented.github.io/master/images/npe/npe_in_rails.png)

### Perl

```perl
my $person;
print $person->name; # crash 
```

![npe_in_perl](https://raw.githubusercontent.com/useroriented/useroriented.github.io/master/images/npe/npe_in_perl.png)

## 表格

让我们用表格来记录下上面的语言对这种很明显的错误处理的表现：

Example	Compiles*	Crashes
C++	YES	YES
Java	YES	YES
C#	YES	YES
JavaScript	YES	YES
PHP	YES	YES
Python	YES	YES
Ruby	YES	YES
Perl	YES

对动态语言来说，`compiles`意味着语法检查器或者编译器不能在使用的时候检测出错误

## 解决方案

### 手动检查

把空值检查写进编程语言的感觉是这样的:

```js
if (name != null)
    name.toLowerCase();
else
    ... // handle an error case
```

这种特殊的写法显然不是NPE的通用解决方案。因为不可能在程序里到处加上这个。这是程序员的担当，而且对人来说这是一种错误的写法。

### 静态分析

用工具比如lint静态分析也许能找到潜在的NPE。写这种工具是非常麻烦的事情，因为工具必须必编译器或者解释器还聪明，才能从源代码中推断出
错误信息。编程语言允许到处使用null，但是lint会到处报错，修复这些错误让你身心疲惫。静态分析器并不能让你完全杜绝烂代码。


## NonNull 声明

作为补充，一些语言添加了额外的语法(注释或者属性)来告诉编译器哪些值有可能是null哪些不是。Java有`@NotNull`在[various forms](http://stackoverflow.com/questions/4963300/which-notnull-java-annotation-should-i-use),C#有`NotNullAttribute`,
Object-C 有`nonnull`

这些方法或许有用，但是还不是银弹。大多数的代码都没用到这些。你需要花时间去修复和加上这些。还有些情况是一些编程语言就是用null作为默认值的。

## Option type

[Option type](https://www.wikiwand.com/en/Option_type)有很多出名的实现，比如:`Nullable<T>`, `Optional<T>`, `Option<T>` 和 `Maybe<T>`。在不同的语言中，T表示数据类型。这种类型允许你直白的表示值是有可能为空的，如果想要得到非空的值，需要做特殊处理。

option type已经在haskell和F#应用得很好了。但是需要对做转换才能拿到非空值，比如`if-else`。反过来说，那也说明很容易拿到内部值，都不需要检查，也不能被检查出异常。