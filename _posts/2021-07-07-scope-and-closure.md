---
layout: post
title:  "语言的作用域和闭包"
comments: true
categories: 程序语言
---
# 词法作用域/静态作用域 vs 动态作用域(lexical scope / static scope vs dynamic scope )

块block和变量的作用域scope是程序语言的基础概念。

我们常说，函数变量的作用域被限定“在函数中”，但这句话到底是什么意思呢？

在lexical scope（也称为lexical scoping，或者static scope或者static scoping）中，“在函数中”指这个函数体的文本定义，即花括号所限定的范围内；而在dynamic scope（也称为dynamic scoping）中，“在函数中”指在这个函数的运行时间内，换句话说，Dynamic scoping means that variable lookups occur in the scope where a function is called, not where it is defined。

比如，函数f调用了一个**单独定义**的函数g，那么在lexical scope下，g不能访问f中定义的local变量，但在dynamic scope下却可行：
```sh
$ x=1
$ function g() { echo $x ; x=2 ; }
$ function f() { local x=3 ; g ; }
$ f # does this print 1, or 3?  KornShell:1   Bash:3
$ echo $x # does this print 1, or 2?  KornShell:2   Bash:1
1
```
上面两条指令会输出什么呢？这取决于程序语言的作用域模型。在lexical scope模型下，因为g是在f外**单独定义**的，因此它无法访问f中的局部变量x，但会访问到全局的变量x，因此函数g对x的读写都是针对全局变量x：g会先输出全局变量x的初始值1，再将其绑定到新值2。而在dynamic scope模型下，因为g是由f调用，在g执行时，f也在执行，因此f中的局部变量x对g而言是可见的，故函数g对x的读写都是针对f中的局部变量x：g会先输出f中局部变量x的值3，再将其值绑定到新值2。最后echo指令输出的是全局变量x，再lexical scope模型下，其值为2，在dynamic scope模型下，其值为1。

实际上，KornShell采用了lexical scope模型，而**Bash采用了dynamic scope**。

go采用了基于block的lexcial scope，我们可以再看一例：
```Go
package main
import "fmt"
var a = 1
func foo(){
    fmt.Println("in foo, a is", a)      // 1
    a = 2       // 这里确认a已被定义，因此直接赋值
}

func main(){
    var a = 0
    fmt.Println("in main, a is", a)     // 0
    foo()
    fmt.Println("after foo() call, a is", a)  // 0
}
// output
in main, a is 0
in foo, a is 1
after foo() call, a is 0
```

在lexical scope模式下，一个name总是根据lexcial上下文确定，这是程序文本的固有属性，且与程序的运行时调用栈无关，因此只需要分析静态的程序文本，就可以确定某个name的具体所指。这也是它也被称为static scope的原因。显然，静态作用域下程序员可以较容易的理清本地所有使用name的所指，这对于构建大规模软件是必要的。

在dynamic scope模式下，一个name总是根据包含它的语句所执行的上下文来确定，这意味着每个name都要根据一个全局的栈来推定：假设在函数中定义一个局部的变量x，当该函数执行时会将其（x及其值）压入某个全局栈，当控制流离开函数时，会从全局栈中弹出x。任何情况下，评估x的值时，总会在该全局栈中查找。注意这个求值动作不可能发生在编译时，因为这个存放绑定变量（变量名和其值）的全局栈只在运行时存在。
> 注意，所谓的全局栈是全局堆中构造的栈，全局堆中的各个栈彼此独立，但每个栈都与特定的函数有关。

## first-class function，lexical scope和闭包closure
实现同时支持first-class function（所谓first-class function，就是说将function视为first-class citizen，即将函数作为一种类型：可以将函数赋值给一个变量，作为参数传递给其他函数，或者作为函数的返回值。）和lexical scope的编程语言并不容易，因为这意味着每个函数值都将附带一个它所依赖的变量字典。

以Go为例，假设有如下定义：
```Go
func intSeq() func() int {
    i := 0
    return func() int {
        i++
        return i
    }
}
func main() {
    nextInt := intSeq()
    fmt.Println(nextInt())
    fmt.Println(nextInt())
    fmt.Println(nextInt())
    
    newInts := intSeq()
    fmt.Println(newInts())
}
// output
1
2
3
1
```
被返回的匿名函数中使用了变量i，根据词法作用域定义，变量i确定的来自于intSeq中定义的本地变量。当我们将intSeq()的返回赋值给nextInt时，这个函数值实际上附带了一个上下文字典，其中有一个变量i，其值初始为0。这意味着上述模式实际上定义了一个带有一个上下文数据字典的函数。

这个带有上下文字典的函数的整体就是closure，换句话说，**为了在lexical scope模式下支持first-class function，人们引入了closure**。 用wiki中的说法“a closure is a record storing a function together with an environment”，closure是一个存有函数和相关上下文环境的记录，closure是对这个上下文变量字典来说的：数据被close在某个隐秘的空间里，且只能使用特定的函数来操作访问。

再举一个例子，这个例子中用到了**自由变量**：
> 所谓自由变量free variable，即不是在*内部函数*中定义的变量，而是在*外部函数*中定义，或者是外部函数的输入参数。这类变量可以被外部指令修改，所以被称为自由变量。
```python
def f(x):
    def g(y):
        return x + y
    return g  # Return a closure.

# Assigning specific closures to variables.
a = f(1)

# Using the closures stored in variables.
assert a(5) == 6

# Using closures without binding them to variables first.
assert f(1)(5) == 6  # f(1) is the closure.
```

这里**自由变量x被用作g的上下文字典的元素**，当我们定义f(1)时，即返回一个将自由变量x初始为1的closure。

可以用一句话来描述closure的技术性构造过程：if **a function with free variable** is first-class, then **returning it** creates a closure。这里有两个关键，一是含有自由变量的内嵌函数，二是返回该函数。

由于closure要求free variable具有和返回函数变量一样的生命周期（因为自由变量就是返回函数的上下文字典，它们是一体的），因此这些自由变量不能在线性的栈中分配，而必须在堆中分配，这也是为什么支持闭包的语言一般都有garbage collection，因为需要使用垃圾回收器来处理堆中属于闭包的数据。

closure与面向对象语言的class的实例有一些类似，只不过class更通用一些：方便定义更多元素的数据字典，以及更多灵活的操作方法。所以有人说，class是带有方法的数据，而closure是带有数据的方法/函数。

参考
- [1] [词法作用域](https://en.wikipedia.org/wiki/Scope_(computer_science)#Lexical_scope
- [2] [闭包](https://en.wikipedia.org/wiki/Closure_(computer_programming))