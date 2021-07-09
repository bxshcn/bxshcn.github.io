---
layout: post
title:  "go语言类型系统"
comments: true
categories: 程序语言
tags: golang
---

# go语言类型系统

## 类型的分类
Type denoted by a *type name*, if it has one, or specified using a *type literal*, which composes a type from **existing** types.  即type有两种表示形式，一是用name来表示一种type，另一种是使用literal来表示一种type。

我们也称具有name的type为defined type，区别于没有name的type literal。
> 所有predefined的type都是defined type

由于type literal是根据现有的type name来compose得到的，我们也称其为composite type。也就是说， Composite types—array, struct, pointer, function, interface, slice, map, and channel types—may be constructed using type literals.

> Golang将ArrayType, **StructType**, PointerType, FunctionType, InterfaceType, SliceType, MapType, ChannelType称为type literal。**从词法结构上看，type literal主要是由Golang关键字、符号（identifier，punctuation），加上existing type共同组合而成**，我们根据这个词法结构，就自然的确定了这个type，所以称为type literal ，比如：
> - `*int`，是符号`*`，加上existing type `int`组合而成。
> - `func(int, int, float64) (float64, *[]int)`，是关键字`func`，加上符号`(`以及`)`，再加上existing type `int`, `float64`, `*[]int`组合而成。
> - `map[string]int`，是关键字`map`，加上符号`[`以及`]`，再加上existing type `int`, `string`组合而成。它并不是一种**type name**，没有叫`map[string]int`这个name的type，它只是一种**type literal**
> - `struct { x, y float64 }`，是关键字`struct`，加上符号`{`以及`}`，加上identifier符号`x`和`y`，以及existing defined type `float64`组合而成。
> 
> 注意，type literal是基于现有的type构造的，比如已有`type B1 string`, 那么`type B3 []B1`中定义的B3类型，就是一个type literal。

## 类型声明
有两种类型声明的形式：
1. 定义类型别名。为了代码阅读和编写的方便，用另外一个type name指代一个现有的type（即可以是type name，也可以是type literal），**为现有的type指定别名**。比如 `type nodeList = []*Node`, 注意其中的等号。对编译器来说，别名给出的type name与原type完全一致，所以你不能给别名附加新的methods，因为你不能改变原type。
2. 定义新的类型。基于现有的type（即可以是type name，也可以是type literal），**定义新的type name**。比如`type OperationType string`。注意这种情况下，新type与原type之间没有等号，因为它们是两种完全不同的type。在新的type定义后，我们往往会为这种新的type附加新的methods。

## 底层类型 underlying type
Each type T has an underlying type: If T is one of the predeclared boolean, numeric, or string types, or a type literal, the corresponding underlying type is T itself. Otherwise, T's underlying type is the underlying type of the type to which T refers in its type declaration.

underlying type是一个比较重要的概念，在后面**辨识type identity和两个变量的assignability**时都会涉及。简单说，predefined type name和type literal的underlying type就是其本身，而defined type的underlying type是它所参引类型的underlying type。比如：
```Go
type (
	B1 string
	B2 B1
	B3 []B1
	B4 B3
)
```
1. B1参引的类型为string，string的underlying type为string（其自身）
2. B2参引的类型为B1，B1的underlying type为string（根据1），所以B2的underlying type为string
3. B3参引的类型为[]B1, 这是一个type literal，因此其underlying type就是其自身，为[]B1。注意这种情况下避免了递归解析，B3的underlying type不是[]string
4. B4参引的类型为B3，B3的underlying type为[]B1（根据3），因此B4的underlying type为[]B1

## 类型一致性 type identity
一个defined type总是不同于任何其他任何type。因此**当我们说two types are identical的场景，总是针对两个不同的type literals(涉及array、struct、pointer、function、slice、map、channel)**。两个type literals是否identical，取决于它们是否具有相同的underlying type literal。

**type identical是编译器检查所依赖的基本规则，即编译器据此判定某些表达式的逻辑是否合法**，比如
1. 两个接口合并成新的接口时，具有相同名称的两个function是否具有一致签名。参见"A union of method sets contains the (exported and non-exported) methods of each method set exactly once, and methods with the same names **must** have identical signatures."from [Spec](https://golang.google.cn/ref/spec).
2. 二元操作符两侧的变量是否具有一致的类型。参见“For other binary operators, the operand types must be identical unless the operation involves shifts or untyped constants.” from Spec.

## 可赋值性 Assignability
**可赋值性是编译器检查所依赖的另外一项基本规则，即编译器判定什么样的赋值操作是合法的**。

可赋值性可表述为，A value x is assignable to a variable of type T ("x is assignable to T")。主要有两条规则：
1. x's type V and T have identical underlying types and at least one of V or T is not a defined type.比如：
   ```Go
   type eventType int
   var addEvent eventType = 0
   var x int
   x = addEvent  // 报错，因为x和addEvent都是defined type

   type eventPointer *int
   var ep eventPointer = nil
   var x int = 3
   var epp = &x
   ep = epp    // O.K. 虽然ep是defined type，但epp是*int，是literal type，没有明确的类型名，所以可以赋值。
   ```
2. T is an interface type and x implements T.

## 定义新的类型
基于struct组合**多个type name or literal**来定义新的类型可参考后面“方法集和嵌套”的内容，本节主要讨论基于**predefined or literal type**定义新类型的含义。

### 针对predefined的自定义类型——枚举
Golang没有内建的枚举类型，我们往往基于predeclared type来创建枚举类型，比如：
```Go
// 枚举值为integer
type eventType int

func (e eventType) String() string {
	switch e {
	case addEvent:
		return "add"
	case updateEvent:
		return "update"
	case deleteEvent:
		return "delete"
	default:
		return fmt.Sprintf("unknown(%d)", int(e))
	}
}

const (
	addEvent eventType = iota
	updateEvent
	deleteEvent
)
```

再比如：
```Go
// 枚举值为string
type OperationType string

// The constants should be kept in sync with those defined in k8s.io/kubernetes/pkg/admission/interface.go.
const (
	OperationAll OperationType = "*"
	Create       OperationType = "CREATE"
	Update       OperationType = "UPDATE"
	Delete       OperationType = "DELETE"
	Connect      OperationType = "CONNECT"
)
```

### 针对slice和map的自定义类型——抽象并扩展功能
对于slice和map的自定义类型，我们为其增加新的method时，一般不会使用指针作为接收者，因为这两者本身就近似java中的reference类型：我们在方法中对slice和map的参数拷贝对象的修改，会直接影响到原始的对象（slice有一点特殊，见下文说明）。
#### slice
针对slice定义新类型的目的主要有两类，一是添加新的功能语义，二是方便排序（本质上是实现特定的interface），比如：
```Go
// 添加新的功能语义
type Certificates []*KubeadmCert

// AsMap returns the list of certificates as a map, keyed by name.
func (c Certificates) AsMap() CertificateMap {
	certMap := make(map[string]*KubeadmCert)
	for _, cert := range c {
		certMap[cert.Name] = cert
	}

	return certMap
}

// 支持排序
type Person struct {
	Name string
	Age  int
}

func (p Person) String() string {
	return fmt.Sprintf("%s: %d", p.Name, p.Age)
}

// ByAge implements sort.Interface for []Person based on
// the Age field.
type ByAge []Person

func (a ByAge) Len() int           { return len(a) }
func (a ByAge) Swap(i, j int)      { a[i], a[j] = a[j], a[i] }
func (a ByAge) Less(i, j int) bool { return a[i].Age < a[j].Age }

func main() {
	people := []Person{
		{"Bob", 31},
		{"John", 42},
		{"Michael", 17},
		{"Jenny", 26},
	}

	sort.Sort(ByAge(people))
	fmt.Println(people)
}
```
需要特别注意的是，基于slice定义的新类型，我们一般不会提供修改对象本身的方法，比如上面的例子，我们只是根据原始的slice得到一个hash表。如果我们要**修改这个对象本身**，我们就需要考虑是只修改slice对象底层数组，还是要同步考虑修改slice的其他属性（比如长度，容量），如果是后者，就应该使用原始类型，或者使用新类型的指针作为receiver。
- 针对slice对象的操作很简单，比如切片操作`a[m:n]`,追加`s=append(s, n)`，删除使用切片或者`copy`，我们不需要通过自定义函数来修改它。
- slice对象虽然也被当成是reference，但它具有length属性，如果我们通过值的方式传递slice变量，即便我们在函数中通过修改了slice及其底层数组，但原始slice对象的length属性并不会随之变化，因此原始的slice会呈现出一种奇怪的状态，它的内容可能会变化，但长度和容量不会变化。上面例子中的排序Swap方法，虽然也会修改原始的slice对象，但不会修改slice的长度和容量，符合我们的概念认知。举例：
	```Go
	type sType []int

	func (s sType) push(n int) {	// 使用value作为receiver
		s = append(s, n)
	}

	func (s sType) modify(i int, n int) {
		if i > len(s)-1 {
			return
		}
		s[i] = n
	}

	var st sType
	st = make([]int, 0, 10)
	st.push(1)
	fmt.Printf("st = %v\n", st)		// st = [], 因为st的长度为0，不会变化，参见
	st = []int{1, 2, 3, 4, 5}
	st.modify(3, 99)				
	fmt.Printf("st = %v\n", st)		// [1,2,3,99,5], 底层的数组被修改了
	```

#### map
针对map的新类型定义比较简单，你只要在新类型中修改了map对象的内容，那么就一定会影响到原始的map对象。所以新类型一般都直接采用值类型作为receiver。比如：
```Go
type mType map[string]int

func (mt mType) add(s string, n int) {
	mt[s] = n
}

var mt mType
mt = make(map[string]int)
mt.add("a", 1)
fmt.Printf("mt = %v\n", mt)		// map[a:1]
```

再举一例：
```Go
type pluginCount map[string]map[string]int64

func (pc pluginCount) add(state, pluginName string) {
	count, ok := pc[state]
	if !ok {			// 如果state对应的map[string]int64对象不存在，则为了记数（向map[string]int64添加key/value对），必须先构造初始化该对象。如果state对应的map对象已经存在，则直接记数即可
		count = map[string]int64{}
	}
	count[pluginName]++
	pc[state] = count
}
```
原类型为a map of map，我们知道，如果要向map中添加或者修改数据，必须确保map已被初始化，为此这里将这些复杂的操作封装为一个简单的add()函数，方便其他地方对pluginCount对象的操作。

### 定义新的函数类型
如果我们将函数看作控制单元，定义函数类型的目的，就是为了方便管理这些控制单元。常见的是将函数类型对象存放到slice或者map中，作为后续总控操作管理的工具。比如：
```Go
type initFunc func(ctx *cloudcontrollerconfig.CompletedConfig, cloud cloudprovider.Interface, stop <-chan struct{}) (debuggingHandler http.Handler, enabled bool, err error)

func newControllerInitializers() map[string]initFunc {
	controllers := map[string]initFunc{}
	controllers["cloud-node"] = startCloudNodeController
	controllers["cloud-node-lifecycle"] = startCloudNodeLifecycleController
	controllers["service"] = startServiceController
	controllers["route"] = startRouteController
	return controllers
}
```
或者作为结构体的成员，作为结构体类型对象（或者对象相关的其他对象）的控制驱动器，比如：
```Go
type MarshalFunc func(runtime.Object, schema.GroupVersion) ([]byte, error)

// DryRunClientOptions specifies options to pass to NewDryRunClientWithOpts in order to get a dryrun clientset
type DryRunClientOptions struct {
	Writer          io.Writer
	Getter          DryRunGetter
	PrependReactors []core.Reactor
	AppendReactors  []core.Reactor
	MarshalFunc     MarshalFunc
	PrintGETAndLIST bool
}
```

### receiver：pointer vs value

关于新定义类型是使用指针还是使用值作为接收者，Golang官方有一个非常精要的[说明](https://golang.org/doc/faq#methods_on_values_or_pointers)：
```
When defining a method on a type, the receiver behaves exactly as if it were an argument to the method. Whether to define the receiver as a value or as a pointer is the same question, then, as whether a function argument should be a value or a pointer.

First, and most important, does the method need to modify the receiver? If it does, the receiver must be a pointer. (**Slices and maps act as references, so their story is a little more subtle, but for instance to change the length of a slice in a method the receiver must still be a pointer**.) 
 
It is the value receivers in Go that are unusual.
```
也就是说，对于slice和map对象，即便我们要修改它们，它们也不必一定要是指针，因为slice和map看起来就像是reference，但如果我们要修改slice的长度(其真实含义是说修改slice中的数据量)，那么其接收者就可以是指针。上面关于slice新类型的案例可以修改如下：
```Go
type sType []int
 
func (s *sType) push(n int) {	// 使用pointer作为receiver
	*s = append(*s, n)	
}
 
var st sType
st = make([]int, 0, 10)
st.push(1)
fmt.Printf("st = %v\n", st)    //st = [1]
```

## 方法集：类型声明和嵌套
在[spec](https://golang.google.cn/ref/spec)中说，A defined type may have methods associated with it. It does not inherit any methods bound to the given type, but the method set of an interface type or of elements of a composite type remains unchanged. 但Golang中并没有inherit的概念，这句话也引起很多误解和[争论](https://github.com/golang/go/issues/41685)。

除非given type（即新定义类型所参引的类型）**就是struct和interface本身**，否则新定义类型和其所参引的given type的method set没有任何关系。比如：
```Go
// A Mutex is a data type with two methods, Lock and Unlock.
type Mutex struct         { /* Mutex fields */ }
func (m *Mutex) Lock()    { /* Lock implementation */ }
func (m *Mutex) Unlock()  { /* Unlock implementation */ }

// NewMutex has the same composition as Mutex but its method set is empty.
type NewMutex Mutex

// The method set of PtrMutex's underlying type *Mutex remains unchanged,
// but the method set of PtrMutex is empty.
type PtrMutex *Mutex
```
这里`NewMutex`是针对`Mutex`这个named type定义，而`PtrMutex`是针对`*Mutex`这个literal type定义，`Mutex`和`PtrMutex`都不是struct或interface本身，因此`NewMutex`和`PtrMutex`的method set都为空。
> 上面代码注释中说“The method set of PtrMutex's underlying type `*Mutex` remains unchanged”， 是说`*Mutex`的method set与`Mutex`一致。这可归结为下面指针

golang中实现继承的唯二方法只有两种：
1. `type newType struct { embedee }`
2. `type newInterface interface { I }`

另外还有一点是指针与值类型的method set之间的关系，即`*T`类型自动具有`T`类型的method set，但反过来不是。

### 嵌套和提升 embedding and promotion
嵌套就是类型S（struct或interface）的field为一个单一的type T。其中T就叫做embedded type。

promotion机制就是针对嵌套情况下，被嵌套的类型T的method set自动提升为外部类型S的method set的规则。这种机制和继承有类似的语义效果，因而称为golang实现继承的唯一方法。具体规则如下：
- If S contains an embedded field T, the method sets of S and `*S` both include promoted methods with receiver T. The method set of `*S` also includes promoted methods with receiver `*T`.
- If S contains an embedded field `*T`, the method sets of S and `*S` both include promoted methods with receiver T or `*T`.
简单说，就是区分**值嵌套**和**指针嵌套**。只有值嵌套情况下，receiver为`*T`的方法会对类型S隐藏。其他情况下没有任何隐藏！

### 典型嵌套的含义
#### struct嵌套struct值
这种模式不多介绍。

#### struct嵌套struct指针
```Go
type Bitmap struct{
    data [4][4]bool
}

type Renderer struct{
    *Bitmap //Embed by pointer
    on uint8
    off uint8
}

func (r *Renderer)render() {
    for i := range(r.data){
        for j := range(r.data[i]){
            if r.data[i][j] {
                fmt.Printf("%c",r.on)
            } else {
                fmt.Printf("%c",r.off)
            }
        }
        fmt.Printf("\n")
    }
}
```
如果希望**让新定义类型的不同实例共享相同的数据（被嵌套类型的一个实例）**，就可以考虑采用嵌套指针的模式。

#### struct嵌套接口
```Go
type echoer struct{
    io.Reader //嵌套一个接口
}

func (e * echoer) Read(p []byte) (int, error) {  // 实现这个接口
    amount, err := e.Reader.Read(p)    // 实施原接口的行为
    if err == nil {             // 附加额外的打印动作
        fmt.Printf("Overridden read %d bytes:%s\n",amount,p[:amount])
    }
    return amount,err
}

func readUpToN(r io.Reader, n int) {        // 调用接口
    buf := make([]byte,n)
    amount, err := r.Read(buf[:])
    if err == nil {
        fmt.Printf("Read %d bytes:%s\n",amount,buf[:amount])
    }
}

func main(){
    readUpToN(os.Stdin,3)

    var replacement echoer
    replacement.Reader = os.Stdin

    readUpToN(&replacement,3)
}
```
struct嵌套接口后，必须实现该嵌套接口的方法。可以从头开始定义该接口方法的逻辑，但更多情况下是基于该嵌套接口扩展功能（比如上例中的`amount, err := e.Reader.Read(p)`），使得新类型的接口方法具有更丰富的语义。

#### 接口嵌套接口
```Go
type ReadWriter interface {
    Reader
    Writer
}
```
组合既有接口形成更大的接口。不过基于单一职责的考虑，接口的大小（即接口中方法的数量）越小越好。

### 小结
规范中对define type的method type的描述没有实质意义，所以也不是我们关注的点：
1. 如果given type是普通类型（非struct和interface本身），那么新类型不继承原类型的任何method set，既然如此我们自然没有必要这样定义
2. 如果given type的underlying type是interface，新接口具有原接口类型的一样的method set，但我们定义新的interface，肯定是希望这个接口的method set与原接口有所区别，所以这样的定义一样没有意义。

简而言之，**我们只需要关注如下几类新类型定义**：
为函数定义方法
定义枚举
1. 将literal type（尤其是[],map，func等）作为given type，为其增加新的业务语义（比如前面的`Certificates`,以及一般的枚举类型定义），参见定义新的类型
2. 基于struct定义新类型，这个过程中我们用关注的是embedded和相应的promotion机制，参见“典型嵌套的含义”
3. embed interface定义新interface，参见“典型嵌套的含义”之“接口嵌套接口”

## 附录 一个完整的继承示例：
```Go
package main

import "fmt"

type iBase interface {
	say()
}

type base struct {
	value string
}

func (b *base) say() {
	fmt.Println(b.value)
}

type child struct {
	base  //embedding
	style string
}

// overridding
func (c *child) say() {
	c.base.say()
	fmt.Println(c.style)
}

func check(b iBase) {
	b.say()
}

func main() {
	base := base{value: "somevalue"}
	childPtr := &child{
		base:  base,
		style: "somestyle",
	}
	childPtr.say()
	check(childPtr)
}
```


参考：
- [1] [Golang规范](https://golang.google.cn/ref/spec)
- [2] Eric Urban. [Embedding in Go](http://www.hydrogen18.com/blog/golang-embedding.html)
- [3] [golang的继承模型](https://golangbyexample.com/inheritance-go-interface-struct/)
