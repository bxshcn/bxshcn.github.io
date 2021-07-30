---
layout: post
title:  "go的常量constant（译）"
comments: true
categories: 程序语言
tags: golang
---
# [常量](https://blog.golang.org/constants)
## 介绍
Go是一种静态类型的语言，因此不允许混合运算不同的数字类型，比如你不能将一个float64类型的变量和一个int类型的变量做加法运算，甚至不能将一个int32类型的变量和一个int类型变量做加法运算。但`1e6*time.Second`，或者`math.Exp(1)`，或者`1<<('\t'+2.0)`却是合法的。在Go中，常量不同于变量，它的行为表现非常类似于常规的数字。本文解释了这是什么意思以及为什么会是这样。

## 背景：C语言
在设计Go语言的早期阶段，我们讨论了C及其派生语言因为允许不同数字类型值的混合运算而导致的很多问题。融合不同长度和不同符号特性的整数进行运算的表达式导致了很多莫名其妙的bugs，程序崩溃，以及可移植性问题。尽管一个富有经验的C程序员可能很熟悉下面计算方式，
```C
unsigned int u = 1e9;
long signed int i = -1;
... i + u ...
```
这肯定会得到某个结果，但它却很不直观：它有多大？它的值是多少？它是有符号还是无符号的？

险恶的bug可能就潜伏在此。

C有一组被称为“一般性算术转换”的规则。这些规则微妙而隐晦，且这些年来一直在变化，反过来又导致了更多的bugs。

所以我们在设计Go时决定强制排除不同数字类型之间的混合运算，以避开这块雷区。如果你想基于i和u实施加法运算，你必须明确你想要的结果类型。假设：
```Go
var u uint
var i int
```
你可以写成`uint(i)+u`，或者`i+int(u)`，总之要清晰的表达加法运算的含义及其结果的类型，而不能像C那样写作`i+u`。你甚至不能在int和int32的两个不同类型变量上实施运算，即便int是一个32bit类型也不成。

这种严格性彻底消除了导致程序bugs和失败的一类通常原因。它是Go语言一项非常重要的特性，但也有代价：很多时候需要程序员在代码中加入一些笨拙的数字类型转换以清晰表达它们的含义。

那么常量呢？按照上面的声明，该如何使i=0或者u=0合法呢？0是什么类型？在简单的情况下，我们可以要求写作i=int(0)，但要求对常量进行类型转换有点过分了。

我们很快意识到，这个问题的答案在于，要让数字常量的工作方式不同于它们在其他类C语言中的行为表现。在经历过很多思考和实验后，我们给出了一种我们认为正确的设计方式，即能让程序员不必总是强制转换常量类型，写出类似于`math.Sqrt(2)`的代码，又能满足编译器的强类型要求。

简而言之，**Go的常量在绝大多数时候都能像我们所理解的那样有效**。让我们来看看这是如何实现的。

## 术语
首先来看一个定义。在Go中，`const`是一个引入标量值（比如2，3.14159，或者“scrumptious”）的关键字。这些命名或未命名的值被称为*常量（constants）*。常量也能经由常量组成的表达式所创建，比如`2+3`，或者`2+3i`，或者`math.Pi/2`，或者 `("go"+"pher")`

一些语言没有常量，另外一些则在常量定义或者const关键字的使用上更为宽泛。比如，C和C++中的const是一种类型修饰符，它能为本已复杂的值编入更为微妙的特性。

但在Go中，常量就是一个简单的不可改变的值。从这里开始，我们只谈论Go中的常量。

## 字符串常量
`"Hello, 世界"`是一个string constant，那么它是什么类型呢？最显而易见的答案是string type，但这是错误的。实际上，它是*untyped string constant*，也就是说，它是一个常量文本值，但没有fixed type。它的确是一个string，但并不是golang中具有string type的值，即便我们为其赋予一个name，比如：
```Go
const hello = "Hello, 世界"
```
hello仍然是一个*untyped string constant*, 无类型的常量仅仅是一个没有特定类型的value。注意，类型系统会强制严格的规则检查以避免combine不同类型的值，但无类型这个特性使我们可以更方便的使用常量，比如：
```Go
type MyString string
var m MyString
m = "Hello, 世界"
```
因为`"Hello, 世界"`是untyped string constant，因此将它赋值给a typed variable并不会导致type error，换句话说，不同于有类型的常量`typedHello`，常量`"Hello, 世界"`和`hello`*没有类型*，所以将它们赋值给任意一种和字符串兼容的变量都没有问题。

这些无类型的字符串常量当然都是字符串，因此它们能在任何允许字符串的地方使用，但它们并不是`string`类型。

另外，我们也可以定义typed string constant，比如：
```Go
const typedHello string = "Hello, 世界"
```

## 缺省类型
作为一个Go程序员，你肯定看过许多类似`str := "Hello, 世界"`的声明，也许你正在想，“如果常量是无类型的，`str`又是怎么从这个变量声明中获取到类型的呢？答案是，一个无类型的常量都有一种缺省类型，在需要类型但程序员又没有明确提供的地方，值将被转换成这种未言明的类型，所以`str := "Hello, 世界"`或者`var str = "Hello, 世界"`,将和`var str string = "Hello, 世界"`完全一样。

你可以认为，无类型常量被放置在一个理想化的值空间中，其中的约束比Go的全类型系统空间要少的多。但为了利用它们去做一些事，我们需要将它们赋值给变量，此时变量需要明确其类型，而常量能告诉变量它应该具有的类型。在这个例子中，str成为一个具有`string`类型的值，因为无类型的字符串常量在这个声明中提供了它的缺省类型:`string`

//TODO

总之，一个有类型的常量遵循类型值的所有规则，而一个无类型的常量因为没有Go类型的限制，所以能更灵活进行类型匹配和混合运算。当**需要类型信息但有没有其他类型信息可用**的时候，才会使用该无类型常量对应的缺省类型。

## 缺省类型由语法格式决定
一个无类型常量的缺省类型由其语法格式决定。string是字符串常量唯一的一种隐式类型，而数字常量则有很多种隐式类型。整形常量缺省为int，浮点常量缺省为float64，字符常量缺省为rune（一种int32的别名），虚数常量缺省为complex128。下面是我们为展示缺省类型而反复使用的经典打印语句：
```Go
    fmt.Printf("%T %v\n", 0, 0)         // int 0
    fmt.Printf("%T %v\n", 0.0, 0.0)     // float64 0
    fmt.Printf("%T %v\n", 'x', 'x')     // int32 120
    fmt.Printf("%T %v\n", 0i, 0i)       // complex128 (0+0i)
```
## 无类型布尔常量
我们前面关于*untyped string constants*的规则也完全适用于*untyped boolean constants*，true和false都是untyped boolean constants，它们可以被赋值给any boolean variable。
> 所谓boolean variable，即其类型的underlying type为boolean type的变量。比如`type MyBool bool`定义MyBool类型，则`var mb MyBool`声明的变量mb就是a boolean variable。
> 
> 进一步的，**无类型常量可以被赋值给底层结构一致的类型的变量**。

## 无类型浮点常量
基本上无类型浮点常量与无类型布尔常量类似，唯一有点区别的是Go有float32和float64两种浮点类型，无类型浮点常量对应的缺省浮点类型为float64。

我们可以基于浮点值来介绍overflow以及the range of values的概念。

数值常量（numeric constant）被置于一个任意精度的numeric space中，它们就是普通的number，直到它们被赋值给某个变量：当你将其赋值给一个变量，由于变量总是有特定的类型，因此目标类型变量必须能容纳这个值。我们可以定义一个非常大的常量`const Huge = 1e1000`,Huge就是a number，但我们不能将其赋值给任何变量，甚至不能打印它：`fmt.Println(Huge)`甚至无法通过编译，会报错“constant 1.00000e+1000 overflows float64”。但Huge仍然有用，我们可以将其与其他数值常量进行运算，只要结果能被float64所容纳表示：`fmt.Println(Huge / 1e999)`就会如我们所愿的输出10。

相对来说，浮点常量具有非常高的（远比float64高）的精度，因此相关基于浮点常量的算术往往更精确。比如:
```Go
math.Pi = 3.14159265358979323846264338327950288419716939937510582097494459

fmt.Println(math.Pi)  // 3.141592653589793
```
所以我们可以基于数值常量进行计算并始终保持非常高的精度，直到结果被赋值给某个浮点变量才会截断存储。这也避免了在浮点常量表达式中经常出现的一些典型错误场景，比如计算得到无穷大，soft underflow（下溢），或者得到NaNs。因为除以0的操作被归为一个编译错误，而不是运行时得到无穷大后上溢，而当所有计算的都是number时，根本不存在结果为“not a number”的情况。
> NaN stands for Not A Number and is a common missing data representation.

## 无类型复数常量
//TODO

## 无类型整数常量
最后是整数，尽管我们有更多类型的整数（不同size，signed或者unsigned，以及其他比如指针uintptr），但它们都遵循同样的规则。

整数常量的不同形式表示对应不同的缺省类型。简单数值常量形式，比如123，0xFF，或者-14对应的缺省整数类型为int，单引号含括的字符形式，比如'a', '世'或者'\r'对应的缺省类型为rune(它实质上是int32的alias)

但没有任何整数常量的形式能缺省对应无符号整形，因此我们只能使用下述模式：
```Go
var u uint = 17
var u = uint(17)
u := uint(17)
```

## 练习：如何表示The largest unsigned int
我们很容易用`const MaxUint32 = 1<<32 - 1`表示最大的uint32的值，那么如何表示最大的unsigned int呢？int或者uint并未指定位数（这个位数即可能是32，也可能是64，这取决于计算机CPU的体系结构），所以这是个问题。

Go的整型数使用2的补码表示，而-1的所有bits都是1，也就是说，-1的内部表示和最大uint的表示一致，所以我们可能会觉得可以用`const MaxUint uint = -1`，但这是非法的，因为-1不能用一个unsigned变量表示，因为-1并不在unsigned值的范围内，即便强制转换也无济于事，`const MaxUint uint = uint(-1)`会得到一样的错误提示。
> 译注：指令代码必须先通过语言系统的强制规则审查后才能转化成硬件指令，仅仅根据硬件层面的兼容来编写代码是不够的，我们必须首先保证在语言层面的兼容，而语言的类型系统是语言整体规则的一个组成部分，故要予以优先考虑。

在运行时将-1转化为unsigned integer是可行的（译注：因为此时已经没有语言类型系统的监管，所有的操作都是指令集的组合，只需符合硬件层面的规则），因此：
```Go
    var u uint
    var v = -1
    u = uint(v)
```
得到的u的确是一个最大的unsigned int，但这是一个变量，而不是我们所理解的一个单纯的常数：我们只是使用它，但永远不会修改它。

回到我们先前的想法，但这次我们不是用-1,而是`^0`, 即任意多位0的位反，但这样也不行，因为在numeric space中，`^0`表示无穷多的1，因此当我们将其赋值给fixed-size的integer类型变量时，会发生overflow：
```Go
const MaxUint uint = ^0 // Error: overflow
```
那么我们怎么才能表示一个最大的unsigned int常量呢？

关键是
1. 避免值out of range of uint，比如负数就不能由uint表示
2. 限制位反操作涉及的bits数目

所以结合上面的分析，可以用
```Go
const MaxUint = ^uint(0)
```
其中uint(0)是最简单的uint值，然后我们将其每位做位反，这就得到了最大的uint常量。如果你理解了上面的整个分析，那么你就理解了Go常量的所有关键知识点。

## 数字
**Go中无类型常量的概念，意味着所有数字常量（无论是整形，浮点型，复数型，甚至字符）都被存放在一个特别的空间。只有在当我们将其带入到变量、赋值以及运算的计算世界时，类型系统才发生效力。但只要我们还在数字常量的世界中，我们就能随意混合任意的常量值进行运算，下面的所有常量都是值1：**
```
1
1.000
1e3-99.0*10-9
'\x01'
'\u0001'
'b' - 'a'
1.0+3i-3.0i
```
因此尽管它们具有不同的隐式缺省类型，但作为无类型常量，它们可以被赋值给任意类型的**整数**变量：
```
    var f float32 = 1
    var i int = 1.000
    var u uint32 = 1e3 - 99.0*10.0 - 9
    var c float64 = '\x01'
    var p uintptr = '\u0001'
    var r complex64 = 'b' - '整数
    var b byte = 1.0 + 3i - 3.0i

    fmt.Println(f, i, u, c, p, r, b)  // 1 1 1 1 1 (1+0i) 1
```
你甚至能像下面这样干
```Go
var f = 'a' * 1.5
fmt.Println(f)   // 145.5
```
当然除了证明上面说的道理，这个例子毫无意义。

这些规则的真正关键点就是Go语言的灵活性。灵活性的意思是，即便在同一个表达式中混合浮点和整数变量进行运算是非法的，你依然可以写出：
- `sqrt2 := math.Sqrt(2)`
- `const millisecond = time.Second/1e3`
- `bigBufferWithHeader := make([]byte, 512+1e6)`
  
并得到你所期望的结果。

因为在Go中，数字常量就像你所期望的那样：它们看起来就像数字。


# 补充参考材料（非blog原文内容）
## [immutability of variable？](https://stackoverflow.com/questions/43368604/constant-struct-in-go)
'const' is not meant as a way to prevent variables from mutating or anything like that.

[常量类型：](https://golang.org/ref/spec#Constants)
There are boolean constants, rune constants, integer constants, floating-point constants, complex constants, and string constants. Rune, integer, floating-point, and complex constants are collectively called numeric constants.

**Go中的常量并不是对变量的不可改变的限定含义**。Go只有字符串常量、数字常量和布尔值常量三种类型，就像我们在现实生活中感受到的那样。Go没有struct Constant一说。

## compile-time vs runtime
The value of a constant should be known at compile time.
```Go
func main() {  
    var a = math.Sqrt(4)   //allowed
    const b = math.Sqrt(4) //not allowed, const initializer math.Sqrt(4) is not a constant
}
```
函数表达式求值的严格含义，是指函数的代码指令被装载到内存并执行完成，这是一个运行时概念。编译器在编译时不会执行函数的指令，而只会计算常量的运算。常量运算的结果依然是一个常量。常量的值必须在编译时被确定。

## untyped vs typed
A string constant like "Hello World" does not have any type. so `const hello = "Hello World"`, the constant `hello` doesn't have a type.

Go is a strongly typed language. **All variables require an explicit type.**

那么为什么可以像下面这样，将一个untyped的常量赋值给一个typed的变量呢？
```Go
func main() {  
    const n = "Sam"
    var name = n
    fmt.Printf("type %T value %v", name, name)

}
```
The answer is **untyped constants have a default type associated with them** and they supply it if and only if a line of code demands it.

Is there a way to create a typed constant? The answer is yes.
```Go
const typedHello string = "Hello World"
```
`typedHello`就是一个string类型的常量


An untyped constant has no limits. When it’s used in a context that requires a type, a type will be inferred and a limit applied.比如：
```Go
const big = 10000000000  // Ok, even though it's too big for an int.
const bigger = big * 100 // Still ok.
var i int = big / 100    // No problem: the new result fits in an int.

var j int = big     // Compile time error: "constant 10000000000 overflows int"
var f float64 = big   // Ok 
```

When the type can’t be inferred from the context, an untyped constant is converted to a bool, int, float64, complex128, string or rune depending of the format of the constant. In the case that constant is an integer, it will be converted to **int**.
那么什么情况下我们无法从上下文推断出变量的类型呢？比如
```Go
const n = 9876543210 * 9876543210
fmt.Println(n)
```
这里Println的定义为`fmt.Println(a ...interface{})`，我们无法从a推断出n需要转化的目标类型，所以就会直接根据常量n自身的格式确定其对应的缺省类型（即int），但这个常量的大小超过了int的最大值，所以编译器直接报`constant 97546105778997104100 overflows int`

## 命名习惯
常数的命名习惯与标准的命名习惯一致，也时采用CamelCase方式，比如一个需导出的常量应命名为NorthWest，而不是NORTH_WEST。

## [枚举](https://yourbasic.org/golang/iota/)
Golang没有枚举类型，但一般都采用下面的方式来实现枚举类型
1. 创建一个新的整形类型
2. 使用`iota`列出其值
3. 为该新类型添加一个`String()`方法

比如：
```Go
type Direction int

const (
    North Direction = iota
    East
    South
    West
)

func (d Direction) String() string {
    return [...]string{"North", "East", "South", "West"}[d]
}
```
使用举例：
```Go
var d Direction = North
fmt.Print(d)
switch d {
case North:
    fmt.Println(" goes up.")
case South:
    fmt.Println(" goes down.")
default:
    fmt.Println(" stays put.")
}
// Output: North goes up.
```
