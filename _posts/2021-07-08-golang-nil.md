---
layout: post
title:  "go的空值nil"
comments: true
categories: 程序语言
tags: golang
---
## nil的含义
nil是golang语言内建的，代表pointer, channel, func, interface, map, 或者slice类型的zero value。

## nil is untyped
nil可以指代上述6种类型的zero value，所以**nil本身是untyped**，仅凭nil你并不能得到任何类型信息，因此如果你要将nil赋值给某个变量，你必须先明确该变量的类型，比如：
```Go
var n = nil
```
将导致编译错误(use of untyped nil)，因为编译器并不知道n是什么类型，所以也无法为n分配空间。但下面这样可行：
```Go
var n *Foo = nil
```
虽然这并没什么实质作用，因为单纯的`var n *Foo`也会将n初始化为指向Foo类型的nil。

## nil for interface
interface的底层实现依赖于type T和value V两部分，以空接口`interface{}`为例：
```Go
type eface struct { // 16 bytes
	_type *_type    // T part
	data  unsafe.Pointer  // V part
}
```
接口的值实际上是&lt;T,V&gt;。对于一个接口变量来说，**只有T和V都为nil，这个interface变量的值才等于nil**。
如果将某个变量赋值给接口之前，指定了变量的类型，那么这个接口就不再会是nil，比如：
```Go
package main

type TestStruct struct{}

func NilOrNot(v interface{}) bool {
	return v == nil
}

func main() {
	var s *TestStruct
	fmt.Println(s == nil)      // #=> true
	fmt.Println(NilOrNot(s))   // #=> false 这里将s赋值给参数v时，v记录了s的类型，即其T part被置为*TestStruct
}
```
我们再看一个[例子](https://golang.google.cn/doc/faq#nil_error)
```Go
func returnsError() error {
	var p *MyError = nil
	if bad() {
		p = ErrBad
	}
	return p // Will always return a non-nil error.
}
```
注意error是一个内建的interface，因此当我们返回interface时，如果要明确返回nil，必须直接返回nil，而不是像上面这样返回一个实现了error接口的类型变量（p是&#42;MyError类型的变量，所以它肯定不可能是nil——nil是untype的）：
```Go
func returnsError() error {
	if bad() {
		return ErrBad       // 异常情况下返回实现了error接口的自定义类
	}
	return nil   // 正常情况下直接返回nil
}
```

## 其他类型的nil
### nil for slice
slice在golang的内部表示大概类似于：
```Go
type SliceHeader struct {
	Data uintptr
	Len  int
	Cap  int
}
```
按照nil的定义，当slice的各个字段为0值时，它就是nil，比如：
```Go
func main() {
    var sl []int
    fmt.Printf("%T,%v,%d,%d\n", sl, sl, len(sl), cap(sl))  // []int,[],0,0 
    fmt.Print(sl == nil)  // True
}
```
### nil for map
A nil map behaves like an empty map when reading, but attempts to write to a nil map will cause a runtime panic

为了区分nil map和一个已经初始化的empty map，可以使用一个特别的赋值表达式（我们称其为comma ok），如果键值存在，则ok为untyped boolean值true，否则为false。
```Go
type timeZone map[string]string

func offset(tz string) int {
    if seconds, ok := timeZone[tz]; ok {        // comma ok
        return seconds
    }
    log.Println("unknown time zone:", tz)
    return 0
}
```