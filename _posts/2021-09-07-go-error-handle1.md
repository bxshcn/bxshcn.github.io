---
layout: post
title:  "go error处理（1）"
comments: true
categories: 程序语言
tags: golang
---

# 简介
本内容包括两篇文章，这是第一篇。

第二篇[go error处理（2）](https://bxshcn.github.io/security/2021/09/08/go-error-handle2.html)

## [errors are value](https://go.dev/blog/errors-are-values)
Rob Pike在15年写的一篇blog，这里简要概括。

在其他语言中，人们习惯采用try/catch块或类似机制来处理errors，与之对应的是Go中的一种更为平实的机制:
```Go
if err != nil {
	return err
}
```
也就是说对可能返回错误的函数结果进行检验。但很多函数都会返回错误，因此频繁的检验会让人感觉很不优雅（原文为feels clumsy）。

Pike指出
> it is clear that these Go programmers miss a fundamental point about errors: Errors are values.
> 
> Values can be programmed, and since errors are values, errors can be programmed.
> 
> Of course a common statement involving an error value is to test whether it is nil, but there are countless other things one can do with an error value, and application of some of those other things can make your program better, eliminating much of the boilerplate that arises if every error is checked with a rote if statement.

这句话的意思是说，Go中的errors并不是我们在java或者其他语言中所传递的那种必须被立刻处理的概念，“wow，好像发生了errors，我们必须立刻马上处理！”，在Go中，我们可以对error类型对象进行编程：即将其视为目标对象的一种属性或状态，在必要的时候（比如处理结束后）才进行处理，其理论依据：
1. errors本身只是一种普通的values
2. 一个代码块往往只会对某个对象进行操作，即便中间出现error，继续执行代码块也只会导致对象的内部结果状态无效，而不会影响其他对象。换句话说，针对这个对象的操作，要么成功，要么失败。
3. 乐观预期：正确操作对象的概率要远大于出错的概率

Pike随后以bufio.Scanner类型为例，解释了**对error编程的基本模式**，其核心思想可概括为如下两点：
1. 为操作对象中增加error类型的域作为状态，记为err
2. 在具体操作时，如果发生错误则更新err状态

比如：
```Go
type errWriter struct {
    w   io.Writer	// 原操作对象
    err error		// 增加err作为对象的状态字段
}
//
func (ew *errWriter) write(buf []byte) {
    if ew.err != nil {
        return
    }
    _, ew.err = ew.w.Write(buf)
}
```
errWriter对象的write方法签名并没有像我们常见的那样返回error，它没有返回值，与之相对应的是更新了对象的err状态。而一旦write发现对象的状态不对，就会让自己成为一个no-op空操作，避免产生负面影响（比如造成错误的系统状态，耗时的等待操作等）。有些情况下即便发生错误，不做检查的鲁莽操作也不会有负面影响，则我们可以干脆不用做这步检查。

与此同时，Pike指出，这种模式有一个缺点，因为我们只在最后检测错误，我们并不知道在错误发生之前处理了多少内容，如果这个信息很重要，那么我们就需要采用一种更细粒度的检测方法（即经典的错误处理模式），但大多数情况下，an all-or-nothing check at the end is sufficient.

上述模式在标准库archive/zip、net/http等包中都有广泛的使用，在bufio.Reader中也有典型的应用：
```Go
type Reader struct {
	buf          []byte
	rd           io.Reader // reader provided by the client
	r, w         int       // buf read and write positions
	err          error
	// ...
}
// --
func (b *Reader) Read(p []byte) (n int, err error) {
	//...
	if b.r == b.w {	// 如果缓冲区中没有数据
		if b.err != nil {		// no-op操作
			return 0, b.readErr()
		}
		// ...
		n, b.err = b.rd.Read(b.buf)
		// ...
	}
	// ...
}
```

下一篇[go error处理（2）](https://bxshcn.github.io/security/2021/09/08/go-error-handle2.html)