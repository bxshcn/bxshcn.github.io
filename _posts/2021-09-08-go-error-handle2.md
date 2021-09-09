---
layout: post
title:  "go error处理（2）"
comments: true
categories: 程序语言
tags: golang
---

## [Defer, Panic, and Recover](https://go.dev/blog/defer-panic-and-recover)
Andrew Gerrand在Go的早期写的一篇blog，这里简要概括，并辅之以另外一篇文章作为补充。

### defer statement
defer是一个statement，类似于go statement。比如：
```Go
defer resp.Body.Close()
```

defer后面跟一个函数，其实质是将该函数压入当前函数（即defer statement所属的函数，也即surrounding function）相关的一个后进先出的栈，并在surrounding function**返回后**逐一弹出执行（原文：A defer statement pushes a function call onto a list. The list of saved calls is executed after the surrounding function returns）。

基于这一原理，我们可以得出三个简单的应用规则：
1. A deferred function’s arguments are evaluated when the defer statement is evaluated.
```Go
func a() {
    i := 0
    defer fmt.Println(i)
    i++
    return
}
// result: 0
```
2. Deferred function calls are executed in Last In First Out order after the surrounding function returns.
```Go
func b() {
    for i := 0; i < 4; i++ {
        defer fmt.Print(i)
    }
}
// result: 3210
```
3. Deferred functions may read and assign to the returning function’s named return values.Deffered函数可以访问surrounding函数的命名返回值
```Go
func c() (i int) {
    defer func() { i++ }()
    return 1
}
// 这个函数实质上返回的是2
```

结合panic和recover，defer语句可以修改中间的异常错误状态，给上层返回一个具有明确意义的error类型对象。实际上，The convention in the Go libraries is that **even when a package uses panic internally, its external API still presents explicit error return values**.

### built-in function: panic and recover
函数签名：
```Go
func panic(interface{})
func recover() interface{}
```

golang specification清晰的描述了什么是panicking：
> While executing a function F, an explicit call to panic or a run-time panic terminates the execution of F. Any functions deferred by F are then executed as usual. Next, any deferred functions run by F's caller are run, and so on up to any deferred by the top-level function in the executing goroutine. At that point, the program is terminated and the error condition is reported, including the value of the argument to panic. This termination sequence is called panicking.

简单说，panic的调用会导致F终止执行，F中defer的函数列表被依次执行，然后F的caller函数（假设为Fp）被终止执行，Fp中defer的函数列表被依次执行，然后F的caller的caller函数（假设为Fpp）被终止执行，Fpp中defer的函数列表被依次执行……这样直到顶层goroutine函数被终止执行，goroutine终止，此后：
1. 最初传递给panic的参数值被给出
2. 错误堆栈信息被抛出

所以panicking的过程中，只有各层defer函数会被执行，如果我们要处理这个panic，就必须在某个层面的defer语句中调用recover()，recover会获取到先前panic的值（如果没有panic但是调用了recover就会返回nil），然后当前层函数会被视为正常终止，更上一层的函数会根据函数调用正常执行的流程继续。

示例：
```Go
package main

import "fmt"

func main() {
    f()
    fmt.Println("Returned normally from f.")
}

func f() {
    defer func() {
        if r := recover(); r != nil {	// 在f的defer语句中调用recover()
            fmt.Println("Recovered in f", r)
        }
    }()
    fmt.Println("Calling g.")
    g(0)
    fmt.Println("Returned normally from g.")
}

func g(i int) {
    if i > 3 {		// i等于4时panic，此后递归调用的g会逐层返回
        fmt.Println("Panicking!")
        panic(fmt.Sprintf("%v", i))
    }
    defer fmt.Println("Defer in g", i)
    fmt.Println("Printing in g", i)
    g(i + 1)		// 递归调用g(int)
}
/* result:
Calling g.
Printing in g 0
Printing in g 1
Printing in g 2
Printing in g 3   	// 递归调用
Panicking!		// i==4时，panic
Defer in g 3		// 逐层返回调用defer语句
Defer in g 2
Defer in g 1
Defer in g 0
Recovered in f 4	// 在最上层f函数中被recover，并获取到调用panic时的参数值4
Returned normally from f.	// f正常返回
*/

/* if not recover in f, that's to say we remove the defer statement, the result will be:
Calling g.
Printing in g 0
Printing in g 1
Printing in g 2
Printing in g 3
Panicking!
Defer in g 3
Defer in g 2
Defer in g 1
Defer in g 0
panic: 4

goroutine 1 [running]:
[stack trace omitted]
*/
```

For a real-world example of panic and recover, see the json package from the Go standard library. It encodes an interface with a set of recursive functions. If an error occurs when traversing the value, panic is called to unwind the stack to the top-level function call, which recovers from the panic and returns an appropriate error value (see the ‘error’ and ‘marshal’ methods of the encodeState type in [encode.go](https://golang.org/src/pkg/encoding/json/encode.go)):
```Go
func (e *encodeState) marshal(v interface{}, opts encOpts) (err error) {
	defer func() {
		if r := recover(); r != nil {		// 在顶层marshal函数的defer语句中recover，并返回常规的error类型对象
			if je, ok := r.(jsonError); ok {
				err = je.error
			} else {
				panic(r)	// 非json包内部的panic直接抛出，留给更上层解决。
			}
		}
	}()
	e.reflectValue(reflect.ValueOf(v), opts)
	return nil
}

// error aborts the encoding by panicking with err wrapped in jsonError.
func (e *encodeState) error(err error) {   
	panic(jsonError{err})
}

// 在各种类型序列化时，如果出现错误则调用本error函数panic，比如：
func textMarshalerEncoder(e *encodeState, v reflect.Value, opts encOpts) {
	// ...
	b, err := m.MarshalText()
	if err != nil {
		e.error(&MarshalerError{v.Type(), err, "MarshalText"})  // panic
	}
	e.stringBytes(b, opts.escapeHTML)
}
```

### [综合起来看](https://eli.thegreenplace.net/2018/on-the-uses-and-misuses-of-panics-in-go/)
Go的error handling，with a combination of **explicit error values** and **an exception-like panic mechanism**.

我们先来看看Rob Pike在提议panic/recover机制时说的一段话：
> This is exactly the kind of thing the proposal tries to avoid. Panic and recover are not an exception mechanism as usually defined because the usual approach, which ties exceptions to a control structure, encourages fine-grained exception handling that makes code unreadable in practice. There really is a difference between an error and what we call a panic, and we want that difference to matter. Consider Java, in which opening a file can throw an exception. In my experience few things are less exceptional than failing to open a file, and requiring me to write inside-out code to handle such a quotidian operation feels like a Procrustean imposition.

Pike认为，execption机制容易被当成一种额外的控制结构，怂恿人们使用细粒度的exception处理机制，使得代码佶屈难懂。以java中打开一个文件为例，很少有比打开文件失败更为异常的其他情况，也就是说，这些情况不那么严重，但在java中，包括打开文件失败之外的其他情况都是异常被抛出，都需要程序员针对处理，但打开文件是我们日常的常规操作，要针对每一种异常情况都编写应对处理办法非常繁琐，容易混淆程序员编写代码时的思路。

Go的解决办法是将各中微小的错误细节通过panic/recover屏蔽在open函数中，而只将打开文件失败的严重错误通过error类型暴露：
```Go
// fs package
type FS interface {
    Open(name string) (File, error)
}

// os package
func Open(name string) (*File, error)
```

程序员只需要考虑打开文件时error是否为nil即可，从而能更好的专注于其业务领域逻辑。

正如Pike所言：
>  instead ties the handling to a function - a dying function - and thereby, deliberately, makes it harder to use. We want you think of panics as, well, panics! They are rare events that very few functions should ever need to think about. If you want to protect your code, one or two recover calls should do it for the whole program. If you're already worrying about discriminating different kinds of panics, you've lost sight of the ball.

panic就是panic，因为其稀缺性，一般情况下都不需要考虑它。如果你真的希望保护你的代码，在代码的上层使用一两个recover调用就足够了。如果你一直担心和并设法区分不同的panics，那么你就错了。

另外需要注意的是，recover只能用在defer语句中，而在defer中，你只能做一些清理工作，并至多修改函数的返回值。因为go的convention是将error作为返回值，因此这个模式相当于将内部的panic转换成error类型对象（通过修改函数的error类型返回值），这也是go的一个非常重要的coding guideline - keep panics within package boundaries. It's a good practice for packages not to panic in their public interfaces. Rather, the public-facing functions and methods should recover from panics internally and translate them to error messages. This makes panics friendlier to use, though high-availability servers will still likely want to have outer recovers installed just in case.

**将错误限定在边界（比如一个package，或者一个函数，甚至一个代码块）之内，在边界之内使用panic来简化错误处理，这也是Go的一种错误处理模式.**
```Go
func (s *ss) scanInt(verb rune, bitSize int) int64 {
  if verb == 'c' {
    return s.scanRune(bitSize)
  }
  s.SkipSpace()		// 可能panic
  s.notEOF()		// 可能panic
  base, digits := s.getBase(verb)	// 可能panic
  // ... other code
}
```

scanInt函数并不处理这些错误，而是交给更上层的调用者处理，并将其转换为用户接口层面的error：
```Go
func errorHandler(errp *error) {
	if e := recover(); e != nil {
		if se, ok := e.(scanError); ok { // catch local error
			*errp = se.err
		} else if eof, ok := e.(error); ok && eof == io.EOF { // out of input
			*errp = eof
		} else {
			panic(e)
		}
	}
}

// doScan does the real work for scanning without a format string.
// doScan是用户接口层面直接调用的函数。
func (s *ss) doScan(a []interface{}) (numProcessed int, err error) {
	defer errorHandler(&err)	// 捕获并recover，将其转换为error类型对象err
	for _, arg := range a {
		s.scanOne('v', arg)	// 会调用scanInt函数，可能会panic
		numProcessed++
	}
	// ...
}
```

另外一种错误处理模式是在前一篇[go error处理（1）](https://bxshcn.github.io/程序语言/2021/09/07/go-error-handle1.html)中提到的：封装对象，辅之以error类型对象作为中间错误状态保存，只在系列操作的最后检查该错误状态并完成错误应对处理。前提是这些系列操作只针对被封装的对象，而不影响其他对象。

当然，最为**经典的错误处理模式**就是将error作为函数返回值，然后实时的检测上一步的函数调用是否发生错误，并进行相应处理。

## 总结
go error handle有三种模式：
1. real-time check模式，即经典模式。函数返回含有error类型对象，即时检测函数的执行结果。
2. all-or-nothing模式，即对error编程模式。封装对象，为其增加error类型的状态，针对该对象的操作会实时更新该对象的error状态，但只在最后一步操作检测结果。
3. lazy模式，即defer-panic-recover模式。应用defer、panic和recover机制，在一定边界内，只将error通过panic抛出，并在边界上recover，同时将最严重的错误转换为error类型对象返回。