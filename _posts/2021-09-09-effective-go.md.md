---
layout: post
title:  "高效Go编程"
comments: true
categories: 程序语言
tags: golang
---
# [Effective Go](https://golang.google.cn/doc/effective_go.html)
本文是官方effective go的非严格翻译版。

## 介绍
要想写好Go，了解它的特性和idiom（an expression that cannot be understood from the meanings of its separate words but that has a separate meaning of its own）很重要。此外，了解Go编程中的conventions（a custom or a way of acting or doing things that is widely accepted and followed）也很重要，比如命名、格式、程序结构等，这样写出的程序才易于理解。

本文档提供了编写清晰、符合惯例Go代码的技巧。它是对《the language specification》、《Tour of Go》和《How to Write Go Code》的补充，你应该首先阅读这三本书。

## 示例
Go package sources不仅仅是一个核心库，也给出了如何使用这种语言的很多示例。如果你有关于如何处理问题或如何实现某些东西的问题，库中的文档、代码和示例可以提供答案、思路和相关背景。

## 格式化
格式问题是最有争议的，但其实也是最不重要的。人们虽然可以适应不同的格式化风格，但最好不要这么干：如果大家都坚持同一种风格，那么就不用在这个话题上浪费时间。问题是如何在避免一个长长的规范性风格指南的情况下，达成这个目标。

Go使用了一个不同寻常的办法，即让机器来应对格式化的问题。所有标准库中的Go代码都使用了gofmt进行格式化。如果你使用gofmt后发现有什么不对，那么请重新调整你的代码，或者提交一个关于gofmt的bug，不要绕过它。

## 注释
Go提供了C-style /* */ block comments 和 C++-style // line comments。一般情况下我们用行注释。块注释主要是作为package注释出现的，但在表达式中间或禁用大片代码时也很有用。

godoc处理Go源文件，提取其中的文档。top-level declarations之前的注释（注意注释和declaration之间不能有空行）会与声明一起被提取出来，作为该项目的文档。

每个包都应该有一个包注释，即在package clause之前的块注释。对于多文件的包，包注释只需要出现在一个文件中，任何一个文件都可以。

不要用额外的符号来格式化注释，比如使用星号表示的横幅标记。生成的输出文档可能不会以固定宽度的字体呈现，所以也不要依赖代码中注释来对齐间距。记住，注释总是被当作单纯的文本，所有字符（不论是HTML字符还是其他辅助标记，比如_this_）都会被原封不动的提取到文档中。唯一特别的是godoc对缩进的处理：godoc会使用固定宽度的字体并灰底色突出显示缩进的文本，因此适合展现程序片段。fmt包的注释就很好地利用了这一点。

## 命名
命名非常重要，值得花点时间来谈论Go程序中的命名传统。

### package名


尽量简洁（Err on the side of brevity），因为每个使用你的包的人都会输入这个名字。而且不要因为包名短小而先入为主地担心冲突。包名只是导入的默认名称，它不需要在所有源代码中都是唯一的，在极少数冲突的情况下，可以在导入包时设定一个不同的名称。

另一个惯例是包名是其源目录的基名（the package name is the base name of its source directory），src/encoding/base64中的包被导入为 "encoding/base64"，但其名称为就是base64，而不是encoding_base64，也不是encodingBase64。

导入某个包的代码，会使用这个包的名字来限定它的内容，从而避免不同包中的同名公有对象之间的冲突。比如bufio.Reader不同于io.Reader。另外可以使用这种特性来为包中的公有对象选择好的名字，比如once.Do(setup)，比起once.DoOrWaitUntilDone(setup)，once.Do更简洁清晰，所以并非名字越长越好。

### Getters
Go没有提供对getters和setters的原生支持，当然你可以自行提供这些方法，不过注意的是，对于getters，Go的习惯用法是避免在方法名称中使用“Get”，如果你定义的类型中有一个field的名称是小写开头的owner，那么对应的getter方法应该是Owner()，这样调用时就是obj.Owner()，非常清晰。这也是利用了包名限定的特性。

Setter方法习惯上包含Set，比如上面的例子中的设置函数为obj.SetOwner()，意义同样清晰。

### 接口名
习惯上，只有一个方法的接口的命名，是用方法名（动词）加上-er后缀构成，比如Reader, Writer, Formatter, CloseNotifier etc。

核心代码库中已有许多这样的命名，遵从它们以及它们所表达的功能含义会让你受益匪浅（it's productive to honor them and the function names they capture）。比如Read、Write、Close、Flush、String等等都有经典的签名和含义。为了避免混淆，不要给你的方法起这些名字，除非它有相同的签名和含义。反过来说，如果你的类型实现了一个与一个著名类型上的方法具有相同含义的方法，就给它相同的名称和签名；比如你的字符串转换方法应该是String而不是ToString。

> **注释**
> 
> 上面似乎没有提到含有多个方法的接口的命名方法。在java中，总是在名称前加一个字母i来表示接口，而对应的类名则添加-impl后缀，那么go呢？
>
> Go的处理思路更优雅，不同于简单的词法符号标记，Go的命名传统从接口的语义出发，更具面向对象特性：因为接口是行为的集合，那么接口的名称就是具有相关行为的对象。大多数情况下，我们总是使用-er后缀来命名接口，用于表示具备某些行为的抽象对象。对于包含多方法的接口，选择一个能精确描述其目的的名称，比如net.Conn, http.ResponseWriter, io.ReadWriter。再比如：
>```Go
>type Handler interface {
>    ServeHTTP(ResponseWriter, *Request)
>}
>```
> 方法总是动词表示的行为，上面的ServeHTTP准确的表达了其功能，但接口的名称并非总是简答的在方法后面加上-er后缀，而是起了一个更简洁清晰的名称Handler，与此类似的：
> ```Go
> type ByteReader interface {
>    ReadByte() (c byte, err error)
> }
> ```
> 动词ReadByte，其接口名被调整为ByteReader，即符合Go中接口以-er后缀结尾的习惯，又符合英文的语法。
> 
> 显然，上述-er后缀的命名习惯的根基是方法行为、所属对象这两方面的语义，如果通过添加-er的确不合适，那么我们就不必一定要采用-er后缀来命名接口，比如下面的Open方法，我们没有将接口命名为Opener，而是使用了一个抽象名称FileSystem，因为我们都知道而且已经接受FileSystem是一种中间层概念，它有很多种实现比如：
> ```Go
> // net.http
> type FileSystem interface {
>    Open(name string) (File, error)
> }
> ```
> 简而言之，对于接口名称，首先考虑用合适的-er后缀，如果有已约定成俗的中间层名称定义存在（类似上面的FileSystem)，则可以考虑直接借用作为接口名。这些约定成俗的中间层名称定义，往往是系统架构层面的中间层，比如上面的FileSystem，跨网络之间的Conn（net包）等。

### MixedCaps 大小写混合
对于多单词名称，Go的习惯用法是使用MixedCaps或者mixedCaps，而不是使用下划线来区分。

## 分号semicolons
和C一样，Go使用分号来区分语句。但和C不同的是，Go使用词法分析器，依据一个简单的规则来自动插入分号，因此Go的源码中很少出现分号。

规则：如果换行符之前的最后一个token是一个identifier，字面量（比如数字或字符串常量），或如下token之一：
> break continue fallthrough return ++ -- ) }
词法分析器就会在这个token的后面自动加一个分号。简言之，“如果换行符出现在一个可以结束语句的token之后，则在token后插入一个分号”。
> **注释**
> 什么是identifier？就是用来指代程序实体的标记，比如变量、类型（Identifier names program entities such as variables and types.），Go预定义了一些identifiers：
> ```
> Types:
>	bool byte complex64 complex128 error float32 float64
>	int int8 int16 int32 int64 rune string
>	uint uint8 uint16 uint32 uint64 uintptr
>
>Constants:
>	true false iota
>
>Zero value:
>	nil
>
>Functions:
>	append cap close complex copy delete imag len
>	make new panic print println real recover 
> ```

由此造成的后果是，你不能将控制结构的开括号放在一个单独的行中，比如：
```Go
if i < f()  // wrong! 因为词法分析器会在f()后加一个分号，显然这是错误的。
{           // wrong!
    g()
}
```

从惯用法上，Go程序中只有for语句内部，以及在一行中区分多个语句时，才会显示的使用分号。

> **译注**
> 
> 简言之，只要你按管用的模式来编写Go代码，就不必过多考虑这里边的tricks。即便真的有这方面的错误，gofmt以及编译器也会非常明白的提示你。

## 控制结构
Go的控制结果和C类似，但又有很大不同。

Go没有do或while循环，只有一个更通用的for；

if和switch接受一个像for那样的可选初始化语句；比如定义一个临时的本地变量：
> ```Go
> if err := file.Chmod(0664); err != nil {
>     log.Print(err)
>     return err
> }
> ```

break和continue语句接受一个可选的标签来标识要中断或继续的内容；

还有新的控制结构，包括一个类型开关（type switch）和一个多路通信复用器（multiway communications multiplexer）select。语法也略有不同：没有圆括号，控制体的正文必须**始终**用花括号分隔。

### Redeclaration and reassignment
> ```Go
> f, err := os.Open(name)  // 声明了两个变量，f和err
> if err != nil {
>     return err
> }
> d, err := f.Stat()        // 声明了一个新的变量d，重新赋值了变量err
> if err != nil {
>     f.Close()
>     return err
> }
> codeUsing(f, d)
> ```
在上面这个例子中，`d, err := f.Stat()`使用了现存的变量err，并赋给一个新的值。

这是Go从实用主义出发而提供的一种不同寻常的特性，使得在一长串if-else语句中方便的使用同一个err变量。

### for
在array、slice、string、map上的遍历可以使用range语句（range clause），range语句会返回两个值，一个是key或者index，一个是值本身。比如：
> ```Go
> for key, value := range oldMap {
>     newMap[key] = value
> }
> ```
range也可以遍历通道，这个迭代结构会持续顺序取出通道中的元素，直到通道关闭。如果通道为nil，则range语句将**阻塞**。
在string遍历上，range会额外做一些工作，比如会将string拆分成单独的rune字符，如果有错误的字节，则会自动使用rune U+FFFD代替。比如：
> ```Go
> for pos, char := range "日本\x80語" { // \x80 is an illegal UTF-8 encoding
>     fmt.Printf("character %#U starts at byte position %d\n", char, pos)
> }
> // 结果
> character U+65E5 '日' starts at byte position 0
> character U+672C '本' starts at byte position 3
> character U+FFFD '�' starts at byte position 6
> character U+8A9E '語' starts at byte position 7
> ```

最后，Go没有,操作符，并且++和--是语句而不是表达式。
> **注释**
>
> 语句是采用冒号分隔的可独立执行的指令。语句的定义包括
>>  Declaration | LabeledStmt | SimpleStmt |
>>	GoStmt | ReturnStmt | BreakStmt | ContinueStmt | GotoStmt |
>>	FallthroughStmt | Block | IfStmt | SwitchStmt | SelectStmt | ForStmt |
>>	DeferStmt
>
> 其中SimpleStmt则包括：
> 
>>  EmptyStmt | ExpressionStmt | SendStmt | IncDecStmt | Assignment | ShortVarDecl
> 
> for语句的三个组成部分都是表达式，而不是语句 
> 
>> for init; condition; post { }

### switch
分表达式switch和类型两种switch，即分别根据表达式的值，以及类型来切换语句执行流程。

### 表达式switch
```Go
switch x := f(); {  // missing switch expression means "true"
case x < 0: 
    return -x
default: 
    return x
}
```
可以在switch语句中使用一个类似于for和if的初始化语句，用逗号和待求值的表达式分开。如果没有表达式，则switch on true！

case语句的定义为`"case" ExpressionList | "default"`,注意case后可以是一个表达式列表。只要switch的表达式的值与case后表达式列表中的任意表达式值相匹配，则执行对应的语句块。

对于switch，loop，select，不论其组合的执行流程有多深多复杂，break+label语句总是可用来提前终止该label所标记的语句，比如：
```Go
// https://medium.com/golangspec/labels-in-go-4ffd81932339
OuterLoop:
    for i := 0; i < 10; i++ {
        for j := 0; j < 10; j++ {
            fmt.Printf(“i=%v, j=%v\n”, i, j)
            break OuterLoop         // 提前终止OuterLoop所标记的for语句块
        }
    }
> output
i=0, j=0

SwitchStatement:
    switch 1 {
    case 1:
        fmt.Println(1)
        for i := 0; i < 10; i++ {
            break SwitchStatement   // 提前终止SwitchStatement所标记的switch语句块
        }
        fmt.Println(2)
    }
    fmt.Println(3)
> output
1
3
```
通常情况下，在loop语句中包含一个switch进行更进一步的切换控制的模式比较常见，因此一个更好的例子：
```Go
// https://www.geeksforgeeks.org/switch-case-with-break-in-for-loop-in-golang/
forLoop:
    for number := 1; number < 10; number++ {
        fmt.Printf("%d", number)
        switch {
        case number == 1:
            fmt.Println("-- One")
        case number == 2:
            fmt.Println("-- Two")
        case number == 3:
            fmt.Println("-- Three")
        case number == 4:
            fmt.Println("-- Four")
         case number == 5:
            fmt.Println("-- Five")
        case number == 6:
            fmt.Println("-- Six")
        case number > 2:
            fmt.Println("-- Greater than two")
            break forLoop
        case number == 8:
            fmt.Println("-- Eight")
        case number == 9:
            fmt.Println("-- Nine")
        default:
            fmt.Println("-- Number not identified")
        }
    }

> output
1-- One
2-- Two
3-- Three
4-- Four
5-- Five
6-- Six
7-- Greater than two
```
再举一个单纯的在loop语句中包含一个switch进行更进一步的切换控制的例子：
```Go
func Compare(a, b []byte) int {
    for i := 0; i < len(a) && i < len(b); i++ {
        switch {
        case a[i] > b[i]:
            return 1
        case a[i] < b[i]:
            return -1
        }
    }
    switch {
    case len(a) > len(b):
        return 1
    case len(a) < len(b):
        return -1
    }
    return 0
}
```

### type switch
type switch用于判断一个**接口变量**的实际类型

type switch同样有可选的初始化语句，另外表达式部分可以声明一个变量，比如：
```Go
// https://tour.golang.org/methods/16
var i interface{}
i = functionOfSomeType()
switch v := i.(type) {  // 这里的i必须是接口类型，i.(type)用到了类型声明的语法结构，
                        // 但使用了type关键字，而不是具体的类型。另外这里使用了
                        // 表达式的声明特性，定义了一个switch语句范围的变量v
case T:
    // here v has type T, 以及i的值
case S:
    // here v has type S, 以及i的值
case V, W:
    // here v has type as i
default:
    // no match; here v has the same type as i, 换句话说，v是i的一个完整拷贝
}
```
上面为了方便说明v的类型，所以使用了不同于变量i的名称。实际上按照Go的习惯用法，我们往往会重用原变量，其效果类似于在每个case中声明一个对应类型的同名新变量（覆盖外层的变量名）。比如：
```Go
var t interface{}
switch t = functionOfSomeType(); t := t.(type) {
default:
    fmt.Printf("unexpected type %T\n", t)     // %T prints whatever type t has
case bool:
    fmt.Printf("boolean %t\n", t)             // t has type bool
case int:
    // ...
}
```
> **注释**
> 
> 隐式（语句）块 [implicit block](https://www.cnblogs.com/wongbingming/archive/2020/05/22/12940041.html)
> 
> 语句块内部声明的名字是无法被该块外部访问。换句话说，块决定了其内部声明变量的作用域。
> 
> 除了使用显示的{}标记语句块，Go还定义了一些隐含块，即虽然没有使用{}标记，但一些特定的结构即定义了一个语句块。Go的隐含语句块包括
> - package block，即一个package自然的形成了一个块的范围
> - file block，即一个文件自然的形成了一个块的范围
> - if，for，switch自身即是一个块
> - switch和select的每个case即成为一个块
> ```Go
> for i := 0; i < 5; i++ {
>    fmt.Println(i)
>}
> // "i" is undefined here
> 
> if j := 0; j >= 0 {
>    fmt.Println(i)
>}
> // "j" is undefined here
> 
> switch m := 2; m * 4 {
>case 8:
>    n := 0
>    fmt.Println(m, n)
>default:
>    // "n" is undefined here
>    fmt.Println(“default”)
>}
>// "m" & "n" is undefined here
> ```

## 函数
### 多返回值
Go语言的多返回值消除了C的一些弊端，比如用一个全局的volatile变量来表示错误码，或者传递一个指针参数作为实际的返回值（即利用函数修改指针指向的对象，相当于返回对应值）这种容易让人混淆的惯用法。

举例来说，下例从要给字节数组中获取一个整形数字：
```Go
func nextInt(b []byte, i int) (int, int) {
    for ; i < len(b) && !isDigit(b[i]); i++ {
    }
    x := 0
    for ; i < len(b) && isDigit(b[i]); i++ {
        x = x*10 + int(b[i]) - '0'
    }
    return x, i
}
```
然后你可以这样调用，以便获取一个字节数组中的所有整形数字：
```Go
for i := 0; i < len(b); {
    x, i = nextInt(b, i)
    fmt.Println(x)
}
```

### 命名的出参 named result parameters
出参能有自己的名字，就像入参一样。命名的出参会被初始化为0值，就像入参一样。

给出参命名并非必要，但这样有一些好处，比如它们本身就是文档，比如上例中的函数原型可以定义成：
```Go
func nextInt(b []byte, pos int) (value, nextPos int) 
```
返回值的含义非常清晰明确。

### Defer
defer某个操作的意思就是说让这个操作恰好在函数返回之前一刻执行。

被defer的函数（某项操作）的参数是在defer被调用时求值，而不是在该函数被调用（执行）时求值。这与go语句类似。另外所有defer的操作的执行顺序遵循LIFO，即是栈的形态，比如：
```Go
for i := 0; i < 5; i++ {
    defer fmt.Printf("%d ", i)
}
> out
4 3 2 1 0
```

注意，从语言的resource management来说，defer的函数资源的作用范围是function-based，而一般标识符资源的生命周期是block-based

## 数据data
### 使用new分配内存
Go有两种内存分配指令：new和make。它们的任务相同，但适用于不同的types。

Go的new不像其他语言中的同名函数，分配内存空间但不会初始化它们，而只会用用零值填满。也就是说new(T)会分配一个type T大小的零值存储区域，并返回该区域的地址（其type为*T）。

因为new返回的内存区域是零值填充的，因此在设计对象的数据结构时，如果其中每个项的零值都具有现实意义，那么new返回的对象本身就具有现实意义，可以直接使用而无需再做进一步的初始化。比如bytes.Buffer的文档声明，Buffer的零值表示一个就绪的空缓存；类似的，sync.Mutex也没有显式的构造器或Init方法，而是用零值表示一个未锁定的互斥信号量。

### Constructors and composite literals
有时候单纯的零值并不能表示确定的现实含义，这就需要使用初始化构造器。比如来自os package中的一个定义：
```Go
func NewFile(fd int, name string) *File {       // 初始化构造器
    if fd < 0 {
        return nil
    }
    f := new(File)
    f.fd = fd           // 域成员特别设定
    f.name = name       // 域成员特别设定
    f.dirinfo = nil
    f.nepipe = 0
    return f
}
```
我们可以使用composite literal来简化上述表达。所谓compostite literal，就是一个用于创建新实例的表达式：表达式的每次求值都会创建一个新实例：
```Go
func NewFile(fd int, name string) *File {
    if fd < 0 {
        return nil
    }
    f := File{fd, name, nil, 0}     // f是一个本地变量，但由于我们返回了它的地址，因此
                                    // 与之相关的存储空间在返回后依然可能被保留:
                                    // 前提是这个返回的地址被赋值给另外一个变量，使得
                                    // 这个存储空间的参引记数>=1
    return &f
}
```

composite literal的域值要么以顺序逐一列出，要么以字典形式按需列出，未列出的域字段将被初始化为对应字段类型的零值，极端情况下，一个不包含任何域字段的composite literal，与使用new来创建某种类型的实例的零值，是完全等同的，比如`new(File)`和`&File{}`。

### 使用make分配内存
内建函数`make(T, args)`和`new(T)`不同，它只是被用来创建slices、maps和channels，返回一个已完成初始化（非零值）的type T的实例（注意区别于new返回的&#42;T）。因为这三种内建的数据类型在使用前必须先初始化，比如单纯的使用new([]int)，将返回一个指向新分配的零值slice结构的指针：
```Go
// Unnecessarily complex
var p *[]int = new([]int)       // allocates slice structure; *p == nil; rarely useful
*p = make([]int, 100, 100)

// Idiomatic: 
v := make([]int, 100)
```

### array
Go的Array与C有三种关键差异：
1. array是值，将一个array赋值给另外一个变量将拷贝其中所有元素
2. 特别的，将array作为参数传递给函数，函数将得到原array的一份拷贝
3. array的size是其类型的一部分，因此type [10]int不同于[20]int

array的这种值属性可能有用，但也可能很昂贵，如果你希望类似C的行为和效率，你可以传递指向array的指针，比如：
```Go
func Sum(a *[3]float64) (sum float64) {
    for _, v := range *a {
        sum += v
    }
    return
}

array := [...]float64{7.0, 8.5, 9.1}
x := Sum(&array)  // Note the explicit address-of operator
```
但这并非Go的惯用法，建议直接使用slice。

### slices
slice通过封装array提供了一个操作序列数据的方便接口。除非需要操作固定维度的场合（比如矩阵运算），Go中的大多数array相关的操作都应使用slices。

因为slice内含指向底层array的指针，因此如果你将一个slice赋值给另外一个slice，则这两个slice都指向同一个array。这样的话，如果一个函数有一个slice作为参数，那么函数中对这个slice的改变（即对slice指向array的改变）也将被调用者可见，其效果与直接传递一个指向array的指针类似。因此Read函数被定义为：
```Go
func (f *File) Read(buf []byte) (n int, err error)
```
如果要从文件读取32个字节到buf中，则`n, err := f.Read(buf[0:32])`

在不超过底层array长度（也就是slcie的容量）的前提下，slice的长度可以任意改变，方法是将赋值给其本身，比如`slice = slice[0:l+len(data)]`

特别需要注意，将slice作为参数传递给函数，在函数中操作这个slice之后，**一定要返回这个slice**，因为：
1. 在Go中一切都是值传递
2. 函数使用一个全新的slice，通过拷贝拿到了原参数的内容，但后续的操作都是针对这个新的slice变量。换句话说，你需要将这个新的slice返回给原调用函数。——一个变通的更清晰的办法是使用指向slice的指针作为参数传递，这样你只需要在函数中更新这个指针即可，减少了拷贝slice的开销。

### 2维slices
有两种方式来分配一个2D slice存储空间，我们假设这个2D的slice是行列构成。如果行是可变长度的，那么我们就单独分配每个行对应的slice，否则就可以一次性分配。其关键思路是减少分配次数，因为不同于单纯的执行CPU指令，分配内存涉及内存读写，是一种比较消耗资源的操作。以分配一个点阵位图的示例如下：
```Go
// 第一种分配方式，循环每次为一行分配内存
// Allocate the top-level slice.
picture := make([][]uint8, YSize) // One row per unit of y.
// Loop over the rows, allocating the slice for each row.
for i := range picture {
	picture[i] = make([]uint8, XSize)
}

// 第二种分配方式，一次性分配内存，然后循环赋值给每行对应的slcie
// Allocate the top-level slice, the same as before.
picture := make([][]uint8, YSize) // One row per unit of y.
// Allocate one large slice to hold all the pixels.
pixels := make([]uint8, XSize*YSize) // Has type []uint8 even though picture is [][]uint8.
// Loop over the rows, slicing each row from the front of the remaining pixels slice.
for i := range picture {
	picture[i], pixels = pixels[:XSize], pixels[XSize:]
}
```

### Maps
maps(key,value)是一种内建数据结构，其中key类型必须支持equality操作，因此**slice不能作为map的key**。

尝试访问不存在的key将返回value类型的0值，因此我们可以使用value type为bool的map来表示Set，比如：
```Go
attended := map[string]bool{    // 使用map[string]bool表示参会人集合。
    "Ann": true,
    "Joe": true,
    ...
}

if attended[person] { // will be false if person is not in the map
    fmt.Println(person, "was at the meeting")
}
```

有时候你需要区分“不存在”和“0值”两种情况，比如记录时区tz.(string)与时差seconds.(int)的map timeZone，timeZone["UTC"]返回0的原因，到底是因为其时差值为0，还是因为"UTC"根本不存在？我们可以使用multiple赋值的形式：
```Go
func offset(tz string) int {
    if seconds, ok := timeZone[tz]; ok {        // comma ok
        return seconds
    }
    log.Println("unknown time zone:", tz)
    return 0
}
```
我们称这种惯用法为“comma ok”。

使用delete内建函数来删除map中的指定key，比如`delete(timeZone, "PDT")`，即便"PDT"不存在也没关系，delete总是确保一致的效果。

### printing
[TODO](https://golang.google.cn/doc/effective_go.html#printing)

### Append
内建函数append示意如下：
```Go
func append(slice []T, elements ...T) []T      // ...T 表示不定长参数列表
```
这里T是一个类型占位符，由于Go不支持泛型操作，**你不能编写一个让调用者来决定T类型的函数，这就是为何append是内建的：因为编译器知道任何变量的类型，有编译器的支持才能实现这种类似泛型函数的功能**。

其他示例：
```Go
// 附加多个元素
x := []int{1,2,3}
x = append(x, 4, 5, 6)

// slice + slice
x := []int{1,2,3}
y := []int{4,5,6}
x = append(x, y...)     // slice... 语法解构slice或者array为单个的元素序列
```

## Initialization
除了单个的对象，对于对象之间，以及不同packages之间的初始化顺序问题，Go都能很好的处理。

### 常量
常量只能是数值、字符、字符串和boolean型，定义常量的表达式（即常量的初始化构造器）必须在编译时由编译器求值，因此`1<<3`是一个常量表达式，而`math.Sin(math.Pi/4)`不是，因为只能在运行时调用`math.Sin`函数。

枚举类型定义中，使用常量的一般形式：
```Go
type ByteSize float64

const (
    _           = iota // ignore first value by assigning to blank identifier
    KB ByteSize = 1 << (10 * iota)
    MB
    GB
    TB
    PB
    EB
    ZB
    YB
)

func (b ByteSize) String() string {
    switch {
    case b >= YB:
        return fmt.Sprintf("%.2fYB", b/YB)
    case b >= ZB:
        return fmt.Sprintf("%.2fZB", b/ZB)
    case b >= EB:
        return fmt.Sprintf("%.2fEB", b/EB)
    case b >= PB:
        return fmt.Sprintf("%.2fPB", b/PB)
    case b >= TB:
        return fmt.Sprintf("%.2fTB", b/TB)
    case b >= GB:
        return fmt.Sprintf("%.2fGB", b/GB)
    case b >= MB:
        return fmt.Sprintf("%.2fMB", b/MB)
    case b >= KB:
        return fmt.Sprintf("%.2fKB", b/KB)
    }
    return fmt.Sprintf("%.2fB", b)
}
```
用户自定义类型提供String()方法，以支持fmt的格式化函数，将其任意值打印出来。

> **注释**
> 
> scalar type: 包括枚举，字符，数值，boolean型统称scalar type。它们代表一个单独的值。C的编译器会自动将这些变量转换为整形保存，因为这些标量的操作与整形值的操作一样。

### 变量variables
变量像常量那样被初始化，但初始化构造器可以是运行时计算的表达式。比如：`var home   = os.Getenv("HOME")`

### init函数
每个源文件都可以定义一个或多个`init()`函数，用于设置程序执行前需要的状态。

`init()`函数和main函数类似，是go的保留函数，不能带参数（niladic），也没有返回值。这些`init()`函数只能被go的编译器自动调用，不可以被程序的其他地方调用。

如果某个包有多个文件，这些文件中又含有`init()`函数，则这些`init()`的执行顺序是：
1. 当前包imported的所有packages被初始化
2. 当前包中声明的全局变量被初始化
3. 将包中的文件按名称的字母升序排列，然后顺序调用这些文件中的init函数
4. 对于同一文件中的多个init函数，从头往后顺序调用

## 方法methods
### 指针和值
正如我们在前面`ByteSize`中看到的，可以为任意命名类型（除了指针和interface类型外）定义methods，方法的receiver不必一定是struct。

定义类型的方法时，如果receiver是指针类型，则只能指针类型对象来调用该方法；如果receiver是值，则可以使用该类型的值或者指针类型对象来调用方法。这是因为，当receiver为指针时，意味着函数可能会修改该receiver，如果通过值的形式调用该方法，则修改可能会被丢弃。但对于receiver为指针的情况，Go的编译器会自动为可寻址的值对象上的方法调用加上`&`操作符，比如：
```Go
// 类型定义
type ByteSlice []byte
// 方法定义，receiver为指针类型
func (p *ByteSlice) Write(data []byte) (n int, err error) {
    slice := *p
    l := len(slice)
	if l+len(data) > cap(slice) {
		newSlice := make([]byte, (l+len(data))*2)
		copy(newSlice, slice)
		slice = newSlice
	}
	slice = slice[0 : l+len(data)]
	copy(slice[l:], data)
    *p = slice
    return len(data), nil
}

// 理论上必须使用*b来调用Write方法
var b ByteSlice
// 由于b是addressable的，因此编译器会自动添加寻址符
b.Write(data)   // 等同于(&b).Write(data)
```
> **补充**
> [addressable](https://golang.google.cn/ref/spec#Address_operators)
> - a variable: &x 
> - pointer indirection: &*x
> - slice indexing operation: &s[1]
> - a field selector of an addressable struct operand: struct中操作数可寻址：&point.X
> - an array indexing operation of an addressable array： array的元素可寻址：&a[0]
> - composite literal：&struct{ X int }{1}
其他可参考[寻址](https://www.ctolib.com/topics-130193.html)

## interfaces and other types
### interfaces
Go的interface提供了一种指定对象行为特性的方法：如果某个东西能做*这个*动作，那么它就能在*这里*使用。

在Go代码中，含有一个或两个方法的接口很常见，这些接口的命名往往也依据这些方法的名称来给出，比如`io.Writer`含有唯一的`Write`方法。

### conversions
如果两种类型的底层类型是完全一致的，那么就可以使用强制类型转换语法：T(x)，将变量x（类型为T')转化为类型T。这种转换并不会创建新的值，它只是让x临时表现为一个T类型。注意在Go中，一个变量同时具有类型和值两种属性。

还有另外一种合法的强制类型转换，比如将一个integer变量强制转化为一个floating point变量，这种情况下会创建新的值。

在Go中，**实施临时的强制类型转换以利用目标类型的不同方法并不罕见，实际上这是一种惯用方法**。比如在实际项目中，我们可能会基于已知类型定义新的类型，以及相关方法，使得其语义更为明确，但基础类型也含有一些通用的方法，我们可以定义新类型并使用我们定义的方法，也可以临时将其强制转化为基础类型，以利用其中的通用方法。

### interface conversions and type assertions
type switches也是一种类型强制转换语法：它将一个interface类型变量在每一种case中转换为对应的类型，比如：
```Go
type Stringer interface {
    String() string
}

var value interface{} // Value provided by caller.
switch str := value.(type) {    // type switch, str是guard子句给出的变量，可以在case中使用
case string:
    return str
case Stringer:
    return str.String()
}
```
上面的type switch是假设有多种类型，我们都要处理应对的情况。但如果我们只关心一种类型呢？换句话说，我们只关心某个接口变量是不是特定的类型，那我们可以直接使用type assertion。其语法为`x.(typeName)`，假设x不是指定的类型，或者没有实现对应的接口，直接赋值`x' := x.(typeName)`会导致panic，解决的办法是使用“comma ok”，比如：
```Go
if str, ok := value.(string); ok {
    fmt.Printf("string value is: %q\n", str)
} else {
    fmt.Printf("value is not a string\n")
}
```
如果type assertion失败，则str仍然是存在，其类型为string，只不过其值为零值，即为空字符串。

### Generality 
如果一种type只是用来实现某个interface，且没有接口之外的其他exported方法，那么就没有必要将该type本身导出。仅仅将interface导出可以清晰的表明，对于某个值对象，我们只关心interface所描述的方法。而且避免了为type的每个接口实现方法提供重复的文档。

这种情况下，这种type的构造器（**注意它是exported的**）应该返回对应的接口类型的值，而非type本身的实例。举例来说，两个hash库的实现crc32.NewIEEE和adler32.New都返回接口类型hash.Hash32，因此即便后续你的程序需要将CRC-32算法替换为Adler-32算法，你只需要修改构造器语句，代码的其他部分无需调整。这也是基于接口的编程体现，只不过将这种思想贯彻到了构造器上！

上述思路在各种不同的流式加密算法实现上也得到应用。一种特定的流式加密算法往往是将多种不同的块式加密算法组合在一起应用的结果，为了分离流式加密算法和块式加密算法，将块式加密算法以接口形式描述，然后流式加密算法只基于块加密接口操作，而无需绑定到具体的块加密算法上。比如：
```Go
// NewCTR returns a Stream that encrypts/decrypts using the given Block in
// counter mode. The length of iv must be the same as the Block's block size.
func NewCTR(block Block, iv []byte) Stream
```
NewCTR适用于任何Block接口和任何Stream的实现。因为它的参数和返回的都是接口，所以它可以使用任意的Block接口实例。另外用其他加密模式替换CTR加密也只需要修改该构造函数，因为它周围的代码只将其视为一个Stream，而不关心具体实现的差异。

### interfaces & methods
你可以向任意的对象类型中添加某个方法，从而满足特定接口。比如http包定义了一个Handler接口，任何对象类型只要实现了这个接口，就能提供HTTP服务：
```Go
type Handler interface {
    ServeHTTP(ResponseWriter, *Request)
}
```
其中ResponseWriter这个接口定义了将Response发送给客户端的方法集，其中还包括了io.Writer接口定义的标准Write方法。Request则是代表客户端请求信息格式化表示的结构体类型。

简单起见，我们假设HTTP请求总是GETs，来看一下handler的设置过程。下面是计算页面访问记数handler的一个实现：
```Go
// Simple counter server.
type Counter int

func (ctr *Counter) ServeHTTP(w http.ResponseWriter, req *http.Request) {
    *ctr++
    fmt.Fprintf(w, "counter = %d\n", *ctr)
}
```
注意这里使用`Fprintf`能将信息附加到`http.ResponseWriter`对象中，因为`http.ResponseWriter`是一个`io.Writer`。

创建好handler后，我们再将其添加到URL树（即所有URL服务页面构成的树状结构）中：
```Go
import "net/http"
...
ctr := new(Counter)
http.Handle("/counter", ctr)
```

如果你的程序有一些内部状态，用于记录页面的访问信息（也就是说，当访问页面时，要将访问请求信息记录到程序的内部表中），你该怎么办呢？将一个通道和请求绑定即可：
```Go
// A channel that sends a notification on each visit.
// (Probably want the channel to be buffered.)
type Chan chan *http.Request

func (ch Chan) ServeHTTP(w http.ResponseWriter, req *http.Request) {
    ch <- req
    fmt.Fprint(w, "notification sent")
}
```
> **译注**
> 
> 使用说明：
> 1. 使用带缓冲的通道，每次请求都会将请求详细信息发送到该通道中
> 2. 启用一个或多个goroutine，从该通道读取信息、解析并记录到数据库或系统日志文件中
> 3. 将上述server注册到指定url的handlers链中。

我们最后再看一个例子，假设我们希望将调用上述server程序的命令行参数输出到/args页面，很容易给出一个打印参数的函数：
```Go
func ArgServer() {
    fmt.Println(os.Args)
}
```
我们怎么把它转换为一个HTTP Server呢？参考前面的例子，**我们将计数器（struct或者int）、通道实现`http.Handler`接口，这些计数器和通道对象就可以作为具体页面的server了：`http.Handle("/page", serverObj)`**。为了让这些计数器和通道具有业务上的语义，我们会在ServerHTTP方法的具体实现中引用它们.

类似的，既然我们要将函数本身作为服务的一部分，只需要为这个函数附加`ServerHTTP`方法，并在该方法中引用这个函数本身。http包提供了标准的实现：
```Go
// The HandlerFunc type is an adapter to allow the use of
// ordinary functions as HTTP handlers.  If f is a function
// with the appropriate signature, HandlerFunc(f) is a
// Handler object that calls f.
// 注意，HandlerFunc(f)是将f强制转换为HandlerFunc对象
type HandlerFunc func(ResponseWriter, *Request)

// ServeHTTP calls f(w, req).
func (f HandlerFunc) ServeHTTP(w ResponseWriter, req *Request) {
    f(w, req)
}
```
HandlerFunc是一个具有ServerHTTP方法的类型，因此这种类型的值对象就能提供HTTP服务。为了将前面的ArgServer转换为一个HTTP Server，我们先修改以便其符合HandlerFunc的签名：
```Go
// Argument server.
func ArgServer(w http.ResponseWriter, req *http.Request) {
    fmt.Fprintln(w, os.Args)
}
```
这样ArgServer就具有和HandlerFunc一样的签名，因此我们能将其强制转换为HandlerFunc并利用其方法，就像我们可以将Sequence强制转换为IntSlice并利用其方法IntSlice.Sort()一样。设定页面的服务程序的代码非常简单：
```Go
http.Handle("/args", http.HandlerFunc(ArgServer))
```
When someone visits the page /args, the handler installed at that page has value `ArgServer` and type `HandlerFunc`. The HTTP server will invoke the method `ServeHTTP` of that type, with `ArgServer` as the receiver, which will in turn call `ArgServer` (via the invocation `f(w, req)` inside `HandlerFunc.ServeHTTP`). The arguments will then be displayed.

## the blank identifier
我们已经在for range循环中看到了空标识符的一种用法，在那里空标识符可以承接我们不在意的任意类型的值。但空标识符还有其他一些用法。
### The blank identifier in multiple assignment 多重赋值中的空标识符
实际上for range中的赋值是多重赋值的一种特殊用法，比如：
```Go
if _, err := os.Stat(path); os.IsNotExist(err) {
	fmt.Printf("%s does not exist\n", path)
}
```
偶尔你可能会看到直接忽略错误处理的代码，这很糟糕，千万不要这么干！
```Go
// Bad! This code will crash if path does not exist.
fi, _ := os.Stat(path)
if fi.IsDir() {
    fmt.Printf("%s is a directory\n", path)
}
```

### Unused imports and variables 未使用的导入包和变量定义
当开发一个程序时，仅仅为了编译通过而不得不删除掉未使用的导入包和变量定义，然后在需要时再添加进来会很麻烦，这时我们可以使用空标识符。比如：
```Go
package main

import (
    "fmt"
    "io"
    "log"
    "os"
)

var _ = fmt.Printf // For debugging; delete when done.
var _ io.Reader    // For debugging; delete when done.

func main() {
    fd, err := os.Open("test.go")
    if err != nil {
        log.Fatal(err)
    }
    // TODO: use fd.
    _ = fd
}
```

### Import for side effect
我们有时候只是想利用导入包的副作用（导入包会引发初始化操作，即执行其中的init()函数），而不是为了调用其中的方法。比如net/http/pprof包的init函数注册了一些HTTP页面的debug处理函数：
```Go
// Index, Cmdline等都是pprof包提供的函数。
// 很多情况下这些函数不必一定要exported，虽然这里都是exported的
func init() {
	http.HandleFunc("/debug/pprof/", Index)
	http.HandleFunc("/debug/pprof/cmdline", Cmdline)
	http.HandleFunc("/debug/pprof/profile", Profile)
	http.HandleFunc("/debug/pprof/symbol", Symbol)
	http.HandleFunc("/debug/pprof/trace", Trace)
}
```
因此你只需要在你的应用服务器中`import _ "net/http/pprof"`，然后就可以让你的web服务器提供对应debug服务：此时客户通过执行`go tool pprof http://localhost:6060/debug/pprof/heap`来查看服务器heap的profile信息。


### Interface checks
大多数接口类型转换（interface conversion）是静态的，因此都在编译时进行检查。

但也有一些interface check发生在运行时。比如`encoding/json`包定义了一个`Marshaler`接口，如果JSON encoder收到一个实现了该接口的值，那么JSON encoder就会直接调用该值对应类型的marshaling方法(`MarshalJSON() ([]byte, error)`)来将其转换为JSON串，而不是使用其自身的标准转换逻辑。encoder中进行此步逻辑判断的语句为：
```Go
if m, ok := val.(json.Marshaler); ok {      // 类型声明：动态类型转换
    // buf, err := m.MarshalJSON()
}
```
如果只是判断某个值是否实现了指定类型，使用type assertion的comma ok模式，配合空标识符忽略type-asserted值：
```Go
if _, ok := val.(json.Marshaler); ok {
    fmt.Printf("value %v of type %T implements json.Marshaler\n", val, val)
}
```
上面的逻辑是说，如果某个值的对应类型实现了特定的接口（方法），我们就利用其接口方法实现来操作，否则就使用标准的模式操作。显然，从应用逻辑上讲，某个值对应的类型是否实现特定接口方法并非必须，因此**我们使用了类型声明这种动态类型转换的策略，而不是静态类型转换，比如`T(interfaceVar)`，这一编译检查机制来保证类型T实现了interface**。假设这种类型声明失败，encoder仍然能够正确工作，只不过无法使用客户化的marshal逻辑。

但既然我们的某种类型实现了该接口的marshaling方法，我们就希望确保实现的有效性。假设RawMessage代表原始的JSON字符串对象，它实现了json.Marshaler接口，则我们可以在代码中额外添加下面的语句，利用编译器实施类型强制转换的静态检查：
```Go
var _ json.Marshaler = (*RawMessage)(nil)
```
如果json.Marshaler接口发生了变化，而当前包的RawMessage类型没有相应调整，则上述语句将触发编译失败。

注意只有当代码中没有静态的强制类型转换语句时，我们才需要使用上面的技巧。实际上这中情况很少出现，毕竟我们抽象接口，就是为了利用接口的抽象特性，基于接口编程，因此代码中往往会有很多相关的强制类型转换发生，特别是在函数调用参数传递时。

## 嵌套 embedding
Go没有提供典型的类继承机制，而是通过在struct和interface中嵌套类型来“借用”已实现的代码。

我们之前已经提到过`io.Reader`和`io.Writer`两个接口，如果我们想定义同时包含`Read`和`Write`的`io.ReadWriter`，我们当然可以在其定义中列出这两个方法，但通过嵌套实现会更简单，也更易于我们理解和接受：
```Go
// ReadWriter is the interface that combines the Reader and Writer interfaces.
type ReadWriter interface {
    Reader
    Writer
}
```
它表示一个ReadWriter既能做Reader做的事，也能做Writer所做的事，它是这两个被嵌套接口的并集。特别注意，**接口只能嵌套接口**，而不能嵌套结构体。（Only interfaces can be embedded within interfaces.）

同样的思想也适用于结构体类型，但影响更为深远。bufio包定义了两种结构体类型：bufio.Reader和bufio.Writer，这两者分别实现了io包的Reader和Writer接口。而且bufio还通过嵌套bufio.Reader和bufio.Writer实现了一个带缓冲的读写器ReaderWriter：
```Go
// ReadWriter stores pointers to a Reader and a Writer.
// It implements io.ReadWriter.
type ReadWriter struct {
    *Reader  // *bufio.Reader
    *Writer  // *bufio.Writer
}
```
嵌套的元素是指向结构体的指针，因此它们只有被初始化为指向有效结构体对象之后才能使用。

嵌套和子类继承有一个关键不同。当我们嵌套一个内部类型时，该内部类型的方法将自动成为外部类型的方法，但当我们通过外部类调用这些方法时，这些方法的接收者其实是内部类，而非外部类。在上例中，调用bufio.ReaderWriter.Read(p)的实际效果，与调用bufio.ReaderWirter.Reader.Read(p)等价。而对于子类继承来说，因为使用了虚表，子类中方法调用的接收者即为其子类型对象本身。（ When we embed a type, the methods of that type become methods of the outer type, but when they are invoked the receiver of the method is the inner type, not the outer one. In our example, when the Read method of a bufio.ReadWriter is invoked, it has exactly the same effect as the forwarding method written out above; ）







## concurrency 并行
### share by communicating 基于通讯的共享
并发编程是一个很大的话题，这里仅介绍一些Go特有的亮点。

由于对共享变量的访问往往涉及一些微妙之处，许多环境下的并发编程非常困难。Go采用通道来传递共享变量，任何时刻都只能有一个goroutine能访问共享变量的值。Go本身的设计确保了数据竞争不可能发生。这种思维方式被简化为一句口号：

Do not communicate by sharing memory; instead, share memory by communicating.

**比如引用计数的一个常规方法是使用mutex来控制并发访问，但更高级的方法是使用通道，这可以更容易地写出清晰、正确的程序。**

channel来源于Hoare的Communication Sequential Processes（CSP）理论，可以简单的理解为unix pipe的一种类型安全的泛化。所谓类型安全，是指通过编译器来保证变量之间的操作合法，针对不合法的操作给出错误提示，比如将一个字符串类型变量赋值给一个整形变量会导致Go抛出编译错误。


### Goroutines
由于现有的术语：线程、coroutine、进程等都无法准确描述实际的含义，Go使用Goroutine来刻画它们。它们的确是一个routine或者说是函数，比如创建一个goroutine的语法为`go func()`。所有这些routine都运行在同一个地址空间。它们非常轻量，只比分配一个函数的栈空间多一点点开销。起初这些栈空间都非常小，因此创建一个goroutine的开销很小，后续如果需要更多的内存，再从堆中分配（或释放）。

goroutine被分配（are multiplexed onto）到多个OS线程，如果某个goroutine因为等待I/O值类的原因被阻塞，其他goroutine会接替其继续运行。这种设计隐藏了线程创建和管理的许多复杂性。

使用go关键字调用一个函数或对象方法，当它们执行结束后，goroutine也会自动结束（这与Unix shell使用&符号在后台执行一个命令很类似）。

### 通道
和map类似，channel也是使用make创建，结果是一个指向底层数据结构的引用值对象。缓冲区大小为0的通道称为unbuffered or synchronous channel。

根据go的内存模型，在一个有缓冲通道上的写动作会在读之前完成，符合我们的直觉。而在无缓冲通道上的读动作将在写动作之前完成，则有悖直觉。但这样做是合理的，因为我们要确保在无缓冲通道上写的数据被读到，故只有读完成后，写动作才能完成，换句话说，写要等待读完成。但在有缓冲通道上，通道本身能容纳数据，因此写不必等待读。

有很多使用channel的惯用法（nice idioms），最简单的情况，我们使用通道来同步：
```Go
c := make(chan int)  // Allocate a channel.
// Start the sort in a goroutine; when it completes, signal on the channel.
go func() {
    list.Sort()
    c <- 1  // Send a signal; value does not matter.
}()
doSomethingForAWhile()
<-c   // Wait for sort to finish; discard sent value.
```
这里假设goroutine是一个更耗时的动作，所以说在主goroutine中要等待sort结束。实际上，这里并没有谁等谁的含义，而是彻底的synchronization，如果sort很快完成，那么`c <- 1`会阻塞导致这个goroutine等待，直到`<-c`完成：无缓冲通道上的读写操作决定了动作的先后顺序。

一个缓冲通道往往被用来当作信号量（），控制任务并行度。每当需要启动一个任务时，就往通道中写一个数据，用于模拟获取一个token，结束一个任务时，就从通道中读取一个数据，模拟释放一个token。通道的大小限定了可写token的述目，即表征了任务的最高并行度：
```Go
var sem = make(chan int, MaxOutstanding)

func handle(r *Request) {
    sem <- 1    // Wait for active queue to drain.
    process(r)  // May take a long time.
    <-sem       // Done; enable next request to run.
}

func Serve(queue chan *Request) {
    for {
        req := <-queue
        go handle(req)  // Don't wait for handle to finish.
    }
}
```
但这里有一个问题，Server启动goroutine的过程并不会受限于sem的大小，当请求来的太快时，Server会消耗掉很多无效的资源：创建的很多goroutine会因为等待sem而无所事事。解决的办法很直接，就是修改Server，使用sem来控制goroutine的创建：
```Go
func Serve(queue chan *Request) {
    for req := range queue {
        sem <- 1
        go func() {
            process(req) // Buggy; see explanation below.
            <-sem
        }()
    }
}
```
但这里仍有一个明显的问题，因为process的参数req来自loop循环，而其值会随着loop的迭代而变化。为区分每个goroutine中使用的req，需要在每一次执行go指令时即对req求值：
```Go
func Server(queue chan *Request) {
    for req := range queue {
        sem <- 1
        go func(req *Request) {
            process(req)
            <-sem
        }(req)      // 执行go指令时即求值
    }
}
```
注意，在go中，用于构造goroutine的function literal是一个非常典型的闭包！它的自由变量是req，它在执行后即脱离于外部函数Server，具有自己独立的生命周期，而其上下文字典元素req也只与该函数实例相关。

除了像上面那样，使用sem来限定并发度，还可以以通道为参数直接启动固定数目的goroutine来处理请求，然后每个goroutine的函数会从同一通道中获取请求并对其进行处理：
```Go
func handle(queue chan *Request) {
    for r := range queue {
        process(r)
    }
}

func Serve(clientRequests chan *Request, quit chan bool) {
    // Start 固定数目的 handlers
    for i := 0; i < MaxOutstanding; i++ {
        go handle(clientRequests)
    }
    <-quit  // Wait to be told to exit.
}
```
This Serve function also accepts a channel on which it will be told to exit; after launching the goroutines it blocks receiving from that channel.如果通知Server关闭，负责处理通道请求的各个goroutine会怎么样呢?？？

> **补充**
> 关于[gorouine](https://golang.google.cn/doc/faq#goroutines)，其核心思想是将可独立执行的函数（也就是coroutines）分配到一组内核线程上执行。当一个coroutine因为某种原因（比如进行系统调用）而阻塞，Go的运行时会自动将同一内核线程上执行的coroutines转移到其他可运行的内核线程上，从而避免它们被阻塞。

### 通道的通道 channels of channels
### parallelization
### 泄露的缓存a leaky buffer


## Errors
Golang的多值返回机制使得我们很容易随同常规返回值提供一个详细错误描述信息。依循惯例，所有具体的error类型都应该实现Go的内建error接口：
```Go
type error interface {
    Error() string
}
```
库的编写者往往会在上述接口之外，提供更为丰富的上下文信息，使得错误提示更为直观清晰。比如，os.Open不仅返回一个&#42;os.File，还会返回一个PathError：
```Go
type PathError struct {
    Op string    // "open", "unlink", etc.
    Path string  // The associated file.
    Err error    // Returned by the system call.
}

func (e *PathError) Error() string {
    return e.Op + " " + e.Path + ": " + e.Err.Error()
}
```
注意
1. 这个客户化的pathError包含了error接口Err，这样就可以在其客户化的error接口实现中，通过引用原系统error，来扩展丰富自定义的信息。如果我们不需要原来的系统error信息，则不必包含该接口类型字段。【aaron：如果直接使用嵌套，则PathError会因为提升机制而自动实现error接口，这可能并非我们所希望的，而且在引用原系统error也需要通过field，因此给定名称Err相对更合适。】
2. 是&#42;PathError实现了error接口。从基于接口的编程角度出发，不论是库函数的返回还是你将来对返回错误值的处理，都针对的是&#42;PathError类型的对象，所以：
```Go
// 返回错误：src/os/file_xxxx.go/openFileNolog()
return nil, &PathError{"open", name, syscall.ENOENT}

// 对返回错误的进一步处理：
for try := 0; try < 2; try++ {
    file, err = os.Create(filename)
    if err == nil {
        return
    }
    if e, ok := err.(*os.PathError); ok && e.Err == syscall.ENOSPC {    // 类型声明的是*os.PathError指针
        deleteTempFiles()  // Recover some space.
        continue
    }
    return
}
```
这样实际的错误输出类似于
```
open /etc/passwx: no such file or directory
```
这远比原始的“no such file or directory”更有用。

### panic
报告错误的一般方法，是随同返回值给出一个error类型的对象，但如果error是不可恢复的呢？这种情况下程序往往不宜再继续执行。

Go提供一个内建函数panic来表示这种情况。它会打印一个任意类型的参数（通常是一个字符串)然后终结当前程序，通常这表明发生了某种不可能出现的情况。比如：
```Go
// A toy implementation of cube root using Newton's method.
// 使用牛顿迭代法求某个数的三次方根
func CubeRoot(x float64) float64 {
    z := x/3   // Arbitrary initial value
    for i := 0; i < 1e6; i++ {
        prevz := z
        z -= (z*z*z-x) / (3*z*z)
        if veryClose(z, prevz) {
            return z
        }
    }
    // A million iterations has not converged; something is wrong.
    panic(fmt.Sprintf("CubeRoot(%g) did not converge", x))
}
```
我们应该避免使用panic，如果问题能被规避或者解决，最好处理后让程序继续运行。

### Recover
当panic时（out of bound的slice或数组索引操作，failing的type assertion都会引发），goroutine会停止当前函数的执行，并开始unwinding当前gorouine的栈，一路运行deffered的函数，如果到达goroutine的栈顶，**整个程序**将终止。但在这个过程中，可以使用内建函数recover重获当前goroutine的控制并恢复正常的执行流程。

[unwinding含义](https://golang.google.cn/ref/spec#Handling_panics)：While executing a function F, an explicit call to panic or a run-time panic terminates the execution of F. Any functions deferred by F are then executed as usual. Next, any deferred functions run by F's caller are run, and so on up to any deferred by the top-level function in the executing goroutine. At that point, the program is terminated and the error condition is reported, including the value of the argument to panic. This termination sequence is called *panicking*.

由于在unwinding goroutine栈的过程中，唯一可执行的代码是那些deferred的函数，因此recover只有在其中定义才有意义。recover会停止unwinding操作并且返回传递给panic的参数。recover的一个应用，是只下宕服务器内的单个错误goroutine，而不是整个服务：
```Go
func server(workChan <-chan *Work) {        // 启用goroutine执行任务
    for work := range workChan {
        go safelyDo(work)
    }
}

func safelyDo(work *Work) {
    defer func() {
        if err := recover(); err != nil {
            log.Println("work failed:", err)
        }
    }()
    do(work)
}
```
recover的过程，假设：
```Go
// 入口函数G()
func G() {
    defer C() 
    defer D()
    do()
}

func C() {
    log.Println("C")
}

func D() {
    err := recover();err != nil {
        log.Println(err)
    }
}

func do() {
    beforPanic()
    panic("oo...")
    afterPanic()
}
```
注意到defer的函数是LIFO，是一个栈的形式。当panic发生后，会先执行defer的D()，这里我们recover程序执行，如果D()正常返回，则panic过程终止，程序流程将丢弃之前G的状态，从afterPanic开始继续执行，直到G的末尾，此后再继续执行defer函数栈中的函数，即C()。
The recover function allows a program to manage behavior of a panicking goroutine. Suppose a function G defers a function D that calls recover and a panic occurs in a function on the same goroutine in which G is executing. When the running of deferred functions reaches D, the return value of D's call to recover will be the value passed to the call of panic. If D returns normally, without starting a new panic, the panicking sequence stops. In that case, the state of functions called between G and the call to panic is discarded, and normal execution resumes. Any functions deferred by G before D are then run and G's execution terminates by returning to its caller.

下面的函数调用函数g(),并保证调用者不会因为g的panic而崩溃，这也是很多library routine的编写策略，确保其中的一些风险操作不至影响库本身的运行：
```Go
func protect(g func()) {
    defer func() {
        log.Println("done")
        if x := recover(); x != nil {
            log.Printf("run time panic: %v", x)
        }
    }()
    log.Println("start")
    g()
}
```

所以panic和recover的配合模式是一种从任意糟糕场景中完美脱身的法宝。我们能借此简化复杂软件的错误处理流程。我们最后来看一个例子：
```Go
// Error is the type of a parse error; it satisfies the error interface.
type Error string
func (e Error) Error() string {
    return string(e)
}

// error is a method of *Regexp that reports parsing errors by
// panicking with an Error.
func (regexp *Regexp) error(err string) {
    panic(Error(err))
}

// Compile returns a parsed representation of the regular expression.
func Compile(str string) (regexp *Regexp, err error) {
    regexp = new(Regexp)
    // doParse will panic if there is a parse error.
    defer func() {
        if e := recover(); e != nil {
            regexp = nil    // Clear return value.
            err = e.(Error) // Will re-panic if not a parse error.
        }
    }()
    return regexp.doParse(str), nil
}
```
这里假设我们有一个正则表达式要解析处理。当正则表达式有误时解析失败，此时需要返回(nil,*error*)。我们当然可以在解析过程中，在发现任意错误时，直接返回错误，但**当实际的解析往往存在两层甚至更多层函数调用（为了结构清晰，我们往往会用函数封装功能，让程序结构简单清晰），整个处理流程就会变得比较复杂**，首先是最里层的函数需要区分返回值和错误值，其次中外层函数需要根据最里层的返回做出区分返回，导致函数存在很多if/else的条件分支语句。

为此，我们可以直接在里层的函数中使用panic，然后使用defer的函数recover，这样函数就能将内部的panic转换成error值，从而屏蔽panic对其客户端（也就是调用该函数的代码）的影响。

在上例中，
1. 我们定义一个客户化的Error类型，专用于表示正则表达式解析过程中的各种错误
2. 然后定义了一个简单的本地error函数，负责调用panic来汇报错误，它会将错误的描述性信息转化为前面客户化的Error类型并panic。比如：
> ```Go
>   if pos == 0 {
>    re.error("'*' illegal at start of expression")
>    }
> ```
3. 然后定义功能函数Compile和实际处理解析的doParse函数。doParse会执行最终的解析动作，并调用error汇报各类错误，我们可以将调用error汇报错误视为返回，doParse的程序逻辑将是一个线性流程。与doParse并行的defer函数会优美的处理好各种错误，并修改命名的返回值。

## A Web Server


## 补充
### "comma ok"的使用场景
1. map，用于区分不在map中，与在map中且其值为0的情况。比如：v, ok := a[x]
2. type assertion，避免错误的类型声明导致的panic。比如：str, ok := value.(string)。如果value不是string，则ok为false，且str为string的0值。当然我们此时一般已不在意str的类型和值。
3. channel，用于区分通道关闭与通道中的内容为0值。比如x, ok = <-ch。

### 命令行交互
首先需要区分几个包的作用
1. io.Reader, io包对操作系统底层基本I/O操作原语进行初步封装，因此一般来说它们都不是并发安全的，平常也不会直接使用它们。
2. os包依赖具体的操作系统实现，其涉及类似于unix，但错误处理机制则是Go-like的，也就是使用error作为错误返回而不是共享的全局错误变量。os包封装了操作系统的差异性，向上层提供一个一致的interface，主要涉及文件、目录、环境、用户权限等操作、对File、Process等关键对象的表示。常用案例：
```Go
// 文件权限操作
if err := os.Chmod("some-filename", 0644); err != nil {
	log.Fatal(err)
}
// 环境变量操作
os.Setenv("NAME", "gopher")
defer os.Unsetenv("Name")
// 文件操作
func op() {
    f, err := os.OpenFile("notes.txt", os.O_RDWR|os.O_CREATE, 0755)
	if err != nil {
		log.Fatal(err)
    }
    defer f.Close()
    // do something else
}
// 定了了标准的输入输出，可直接使用，比如os.Stdin
Stdin  = NewFile(uintptr(syscall.Stdin), "/dev/stdin")
Stdout = NewFile(uintptr(syscall.Stdout), "/dev/stdout")
Stderr = NewFile(uintptr(syscall.Stderr), "/dev/stderr")
```
3. bufio封装了io包中的io.Reader和io.Writer对象，得到带缓冲的Reader和Writer，使用带缓冲的读写对象，可以提供更快的读写速度。当然io包的读写对象并不是说没有缓存，而是说io包没有为底层的系统接口提供缓存，但要知道操作系统底层也会有自己的缓冲，比如os.Stdout也有自己的行缓冲区，只有当行缓冲满，或者遇到换行符才会输出。而os.Stderr则没有缓冲区，只要有数据就会直接输出。所以我们经常在go的并行程序中，因为异常导致输出丢失的情况，就是因为缓冲区中的信息被意外丢弃了。在大数据量传输时，带有缓冲的Reader和Writer对象可以大大提升传输的速度。参考C11标准：When a stream is unbuffered, characters are intended to appear from the source or at the destination as soon as possible. Otherwise characters may be accumulated and transmitted to or from the host environment as a block. When a stream is fully buffered, characters are intended to be transmitted to or from the host environment as a block when a buffer is filled. When a stream is line buffered, characters are intended to be transmitted to or from the host environment as a block when a new-line character is encountered.简而言之，缓冲是用空间换时间，减少同步次数，提升传输速度。常用案例：
```Go
// 使用bufio.NewReader(os.Stdin)利用标准输入生成一个带缓冲的bufio.Reader
reader := bufio.NewReader(os.Stdin)
fmt.Print("Enter text: ")
text, _ := reader.ReadString('\n')      // 读取一行
fmt.Println(text)

// 对一个带缓冲的Reader，可以任意操作这个buffer，从而修改下次从中读取的内容。
func (b *Reader) Discard(n int) (discarded int, err error)  // Discard skips the next n bytes
func (b *Reader) Peek(n int) ([]byte, error)  // returns the next n bytes without advancing the reader
func (b *Reader) Buffered() int   // returns the number of bytes that can be read from the current buffer.
func (b *Reader) Read(p []byte) (n int, err error)  // reads data into p

// 为了方便读取解析以换行符分隔，每行由一些token组成的文本文件，bufio提供了Scanner类型
// Scanner provides a convenient interface for reading data such as a file of newline-delimited lines of text. Successive calls to the Scan method will step through the 'tokens' of a file, skipping the bytes between the tokens. The specification of a token is defined by a split function of type SplitFunc; the default split function breaks the input into lines with line termination stripped. Split functions are defined in this package for scanning a file into lines, bytes, UTF-8-encoded runes, and space-delimited words. The client may instead provide a custom split function.
func main() {
	scanner := bufio.NewScanner(os.Stdin)
	for scanner.Scan() {
		fmt.Println(scanner.Text()) // Println will add back the final '\n'
	}
	if err := scanner.Err(); err != nil {
		fmt.Fprintln(os.Stderr, "reading standard input:", err)
	}
}
```
4. fmt包提供了一组方便的函数scan用于扫描处理io.Reader
```Go
func Fscan(r io.Reader, a ...interface{}) (n int, err error)  // scans text read from r, storing successive space-separated values into successive arguments. Newlines count as space.
func Fscanf(r io.Reader, format string, a ...interface{}) (n int, err error)  // 格式化的版本
func Fscanln(r io.Reader, a ...interface{}) (n int, err error) //similar to Fscan, but stops scanning at a newline
```
5. 其他
```Go
// strings: strings 包implements simple functions to manipulate UTF-8 encoded strings
func NewReader(s string) *Reader  // 从字符串得到一个strings.Reader对象，它实现了io.Reader等接口
```

## 标签，以及它和break/continue共同使用时的一般结构
正常情况下，break用于终止最内层的for、switch和select语句，但配合label使用时，可以用于终止label对应的语句。

Label在配合break时，必须标记在for、switch和select语句上，“If there is a label, it must be that of an enclosing "for", "switch", or "select" statement, and that is the one whose execution **terminates**.”

正常情况下，continue用于跳过当前循环中后面的语句，
Label在配合continue时，必须标记在for语句上，“If there is a label, it must be that of an enclosing "for" statement, and that is the one whose execution **advances**.”

一般来说，如果我们要使用标签并配合break、continue语句，往往最外层是循环，内层是switch或者select，比如：
```Go
Loop:
    for _, ch := range "a b\nc" {
        switch ch {
        case ' ': // skip space
            break
        case '\n': // break at newline
            break Loop
        default:
            fmt.Printf("%c\n", ch)
        }
    }
// output
a
b
```

另外一个示例说明switch语句的执行流程:switch语句的表达式只求值一次，case表达式求值是从左向右，从上到下，一旦找到匹配的表达式，则跳过后边的求值：
```Go
func Foo(n int) int {
    fmt.Println(n)
    return n
}

func main() {
    switch Foo(2) {
    case Foo(1), Foo(2), Foo(3):
        fmt.Println("First case")
        fallthrough
    case Foo(4):
        fmt.Println("Second case")
    }
}
// output
2
1
2
First case
Second case
```

关于select：
```Go
for countdown := 10; countdown > 0; countdown-- {
    fmt.Println(countdown)
    select {
    case <-tick:        // 延迟一秒
        // Do nothing.
    case <-abort:
        fmt.Println("Launch aborted!")
        return
    }
}
```

## method receiver
一、**使用指针作为receiver的方法能修改对象本身，因此pointer receiver相较于value receiver更为常用。**
Methods with pointer receivers can modify the value to which the receiver points (as Scale does here). Since methods often need to modify their receiver, pointer receivers are more common than value receivers.
比如，如果我们将下面Scale方法的receiver改为值（即去除Vertex前的`*`)，它将无法修改v，结果输出将变为5：
```Go
type Vertex struct {
	X, Y float64
}

func (v Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func (v Vertex) Scale(f float64) {
	v.X = v.X * f
	v.Y = v.Y * f
}

func main() {
	v := Vertex{3, 4}
	v.Scale(10)
	fmt.Println(v.Abs())
}
```
进一步的，是否使用指针作为receiver类型主要有两种考虑：
1. If a function has a pointer argument then it will only accept a pointer as an argument
2. If a function has a value argument then it will only accept a value as an argument

二、[调用method](https://golangbyexample.com/pointer-receiver-method-golang/)，基于方便考虑不论method的receiver是采用value还是pointer，都可以使用value或者pointer对象来调用method。Go的编译器会根据需要将指针转换为其指向对象的值，或者将值转化为指向它的指针。
- If a method has a value receiver it supports calling of that method with both value and pointer receiver
- If a method has a pointer receiver then also it supports calling of that method with both value and pointer receiver
  
这与函数不同。等等，为什么我们要说“This is unlike function”，因为method是基于对象定义的行为，是**一种特殊的函数**，其receiver虽然也是method function的一个参数，但它被编译器作为receiver并进行特别处理。

对于普通函数来说，如果其参数是value的，那么就只能接受值对象，如果参数是pointer类型，就只能接受pointer对象。
   - If a function has a pointer argument then it will only accept a pointer as an argument
   - If a function has a value argument then it will only accept a value as an argument
  
三、基于接口的编程。在基于接口编程时，我们会使用接口作为函数的参数或者返回值。不失一般性，传递给接口的对象（不论是参数还是返回值），必须是实现了该接口的类型。需要特别指出的是：The method set of type T consists of all methods declared with receiver type T. The method set of the corresponding pointer type *T is the set of all methods declared with receiver *T or T (that is, it also contains the method set of T). [参见](https://golang.google.cn/ref/spec#Method_sets)。但反过来不成立。示例如下：
```Go
type Person interface {
	Greet()
}

type person struct {
	name string
	age  int
}

func (p person) Greet() {
	fmt.Printf("Hi! My name is %s, I am %d years old\n", p.name, p.age)
}
////
func NewPerson(name string, age int) Person {       // 接口作为返回值
	return &person{         // &person{...}是一个指向person的指针类型，它包含以T类型为receiver的方法
		name: name,
		age:  age,
	}
}
////
func NewPerson1p(name string, age int) *person {
	return &person{name, age}
}

func ftest(p Person) {
	p.Greet()
}

p := NewPerson1p("zhao", 19)
ftest(p)            // 指向person的指针类型，包含以person为receiver的方法，因此也实现了接口Person

// 也用反射来查看
t := reflect.TypeOf(p2)
for i := 0; i < t.NumMethod(); i++ {
	fmt.Println(t.Method(i).Name)
	t.Method(i).Func.Call([]reflect.Value{reflect.ValueOf(p2)})
}
```

