---
layout: post
title:  "go语言map实践（译）"
comments: true
categories: 程序语言
tags: golang
---
## [简介](https://blog.golang.org/maps)
计算机科学中最有用的数据结构之一就是hash table。不同hash table的实现各有差异，但基本上它们都提供了快速查找，添加和删除三种功能。Go内建的map type实现了hash table。

## 声明和初始化
map类型定义为：`map[KeyType]ValueType`

而一个map变量的声明则为：`var m map[string]int`

map类型是**reference type**，就像**pointers**或者**slices**一样，因此它的零值是nil，表示未初始化过，而对于一个nil map，它只是在读取的时候像一个empty map，但尝试写的话将会引发panic，这一点要非常注意。

初始化一个map有两种方法：`make(map[KeyType]ValueType)`，或者采用literal composition模式`map[keyType]ValueType{}`

## 操作map
写：`m["route"] = 66`

读：
```Go
i := m["route"]     // 当key"route"不存在时，返回ValueType的零值
i, ok := m["route"] // 2-value赋值，如果key"route"不存在，ok是一个bool类型的值，表示map中是否存在
_, ok := m["route"] // 仅仅判断是否存在
```

删除key/value对：`delete(m, "route")`

获取map的长度：`n := len(m)`

## exploting zero value
从map中读取不存在的key会返回ValueType的零值，这个特性可以简化很多逻辑。

### a map of bool
假设我们先前已经向一个`map[string]bool`类型的变量visited中添加了很多字符串s，则我们可以使用`visited[s]`来判断某个key是否曾被添加到m中：
```Go
if visited[s] {
    doSomething()
}
```

### a map of slice
可以直接向一个nil slice追加对象，所以不论一个slice类型的nil变量是否已被初始化，都可以使用一致的写法：
```Go
var nums []int
nums = append(nums, 1)
```

a map of slice，指map的ValueType是slice类型，由于**对slice的追加操作并不忌讳它是否是零值**，因此我们不必显式的初始化slice就能直接操作它，比如：
```Go
enroll := make(map[string][]string)

enroll["Beckett"] = append(enroll["Beckett"], "algorithm")   // 不论Beckett是否已曾选过课程

fmt.Printf("%v\n", enroll["Beckett"])     // => [algorithm]
```
再举一例：
```Go
    type Person struct {
        Name  string
        Likes []string
    }
    var people []*Person

    likes := make(map[string][]*Person)
    for _, p := range people {
        for _, l := range p.Likes {
            likes[l] = append(likes[l], p)
        }
    }
```
下面打印出喜欢cheese的人：
```Go
    for _, p := range likes["cheese"] {
        fmt.Println(p.Name, "likes cheese.")
    }
```
下面打印出喜欢bacon的人数：
```Go
fmt.Println(len(likes["bacon"]), "people like bacon.")
```
注意range和len都将nil slice视为zero-length的slice，因此这两个案例在无人喜欢cheese或者bacon的情况下都可以work。
> 译注：注意到上面声明people变量，这个slice类型的element是`*Person`，选择指针而不是value作为element，是因为后面有多处都参引到了这些具体的person，不论我们查找喜欢某种食物的人还是统计其个数，我们都没有必要在内存中保留多份Person的拷贝：**我们总是参照现实世界去思考程序逻辑，实体是单独存在的，相关的统计信息则依附于这些实体**。

### a map of map
由于map自身则不具有slice的这种写特性，向一个map中添加（map没有顺序的概念，所以只能说添加，而不是追加）key/value对的前提是map非nil。为了统计某个web页面被某个国家访问的次数，我们可以
```Go
hits := make(map[string]map[string]int)   // 声明
n := hits["/doc/"]["au"]                  // 读取点击次数

func add(m map[string]map[string]int, path, country string) {   // 特别定义的add函数
    mm, ok := m[path]
    if !ok {
        mm = make(map[string]int)
        m[path] = mm
    }
    mm[country]++
}
add(hits, "/doc/", "au")
```
这里特别定义了add函数，因为如果我们要向hits中添加记数的话，就必须确保某个页面对应的记数访问map已被初始化，否则我们是不能直接修改它的。为了规避这个问题，可参考下面

##  key types
map的key必须是comparable的：即key之间可以使用==和!=两类运算。
> 说两个operand是ordered的，即两个operand之间可以支持`<, <=, >, >=`的运算

Golang中comparable的类型几乎包括了所有类型，**除了slice，map，和functions**，这三种类型的变量之间（以及含有这三种类型变量的struct）不能使用==进行比较，所以也不能作为map的key类型。

如前所述，我们可以构造a map of map来处理多维映射问题，但其使用上存在不便。假如我们真正需要的只是一个多维到一维的映射结构，而不是一个严格的层次化映射系统（[a truly hierarchical system of maps](https://stackoverflow.com/questions/44305617/nested-maps-in-golang)），那么就可以使用扁平化的struct作为key。

以web访问记数为例，假如我们关心只是记数，即只需要web页面和国家这个二维空间，到访问记数这个一维空间的映射，那么我们就可以扁平化（flatten）原来的层次化map结构：
```Go
type Key struct {
    Path, Country string
}
hits := make(map[Key]int)

hits[Key{"/", "vn"}]++      // 增加Vietnamese访问/页面的记数
n := hits[Key{"/ref/spec", "ch"}]   // 获取Swiss访问/ref/spec页面的统计记数
```
但**如果我们需要一个严格的层次化映射系统**，我们不仅仅只是要获取(path，country)的访问统计，我们还要区分国家，得到path下不同country的访问次数统计，那么采用a map of map就是应有之义：你必须使用a map of map，甚至，a map of map of map...。

## concurrency
**map是一种并发不安全的类型**。在多个goroutine中并发读写map的行为是不确定的。这种情况下，你必须使用某种同步机制，通常的办法是使用sync.RWMutex，比如：
```Go
// 声明一个含有sync.RWMutex和map的匿名结构体变量，sync.RWMutex用于保护这个map的读写访问
var counter = struct{
    sync.RWMutex
    m map[string]int
}{m: make(map[string]int)}

// 使用读锁来进行读
counter.RLock()
n := counter.m["some_key"]
counter.RUnlock()
fmt.Println("some_key:", n)

// 使用写锁来进行写
counter.Lock()
counter.m["some_key"]++
counter.Unlock()
```

## Iteration order
**map是无序的**。Golang不保证对同一个map的不同迭代（from one iteration to the next）能产生一致的结果输出，如果你希望得到稳定的迭代顺序输出结果，就必须维护一个额外的数据结构来稳定key的顺序，再利用它来给出结果。下面的例子利用一个额外的排序过的slice of keys，来支持按key顺序打印map中的内容：
```Go
import "sort"

var m map[int]string
var keys []int
for k := range m {
    keys = append(keys, k)
}
sort.Ints(keys)
for _, k := range keys {
    fmt.Println("Key:", k, "Value:", m[k])
}
```