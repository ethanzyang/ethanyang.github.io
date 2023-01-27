---
title: "（译）反射的法则  | 《 The Go Blog 》"
date: 2019-03-09 19:25:00
slug: "The-Law-of-Reflection-|-The-Go-Blog"
tags: ["golang", "刻意学习Golang"]
categories: [Dev]
draft: false
---

第一次翻译，英文不好，欢迎指正，争取有时间多看 & 翻译一些外文文章。在翻译过程中发现了很多的官方技术博客，讲得都很 nice ！有时间了继续译过来。
# 译者序
原文章是 @Rob Pike Go 发表在 Blog Go 上的一篇官方关于 Golang `reflection`  的阐述 。《The Laws of Reflection》。
Interface  表示接口 不译。
Reflection 表示 反射 不译。
『』「」为附加解释和说明。
ps 的内容非译文内容
> 原文地址：https://blog.golang.org/laws-of-reflection

## 介绍
反射 是程序运行时检查自身结构的一种能力，特别是通过类型；它是元编程的一种形式，同时也是『混乱』的重要来源。

在本文中，我们试图通过解释反射在 Go 中的工作原理来解释。每种语言的反射模型都是不同的（甚至很多语言更本就不支持），但是这篇文章是关于 Go 的，因此本文的『反射』你应该理解为『Go  的反射』。

## type 与 interface
因为反射建立在类型系统上，所以我们先来回顾一下 Go 中的类型。

Go 是静态类型的。每个变量都有一个静态类型，即在编译时就已知并固定的一种类型: int, float32, \*MyType, []byte 等等。我们来看如下代码，加深理解
```go
type MyInt int

var i int
var j MyInt
```
i 是 int，j 是 MyInt。变量 i 跟 j具有不同的静态类型，虽然它们具有相同的基础类型，如果没有类型转换就不能将它们赋值给彼此。

比较重要的是 interface 类型「接口类型」,它是固定方法的集合。它定义了一组方法。只要该值实现接口的方法，接口变量就可以存储任何具体（非接口）值。一个众所周知的例子是来自 io 包的 Reader 和 Writer 类型：
> ps: 如果没有明白，看这 [interface 类型 与 值](https://learnku.com/articles/25133#5101b8)

```go
// Reader 是 io 包 Read 基础方法的接口
type Reader interface {
    Read(p []byte) (n int, err error)
}

// Writer 是 io 包 Writer 基础方法的接口
type Writer interface {
    Write(p []byte) (n int, err error)
}
```
任何使用此签名实现Read（或Write）方法的类型都被称为实现io.Reader（或io.Writer）。也就是说 io.Reader 类型的变量可以存储任何类型具有 Read 方法的值：
```go
var r io.Reader
r = os.Stdin
r = bufio.NewReader(r)
r = new(bytes.Buffer)
// and so on
```
要明确说明的是无论 r 具体的值是什么，r 的类型总是 io.Reader（Go 是静态类型的，r 的静态类型是 io.Reader）。

另一个非常重要的接口类型列子是 `空 interface`:
```go
interface {}
```
它表示空方法集合，并且完全可以存储任何值，因为任何值都有零个或多个方法。

有人说Go的接口是动态类型的，这是误导。他们是静态类型的：一个接口类型的变量始终具有相同的静态类型，即使在运行时存储在接口变量中的值可能会改变类型，该值也将始终满足接口的「实现该接口」。

我们需要准确地了解所有这些，因为 reflection 和 interface 密切相关。

## interface 的表示
Russ Cox 写了一篇关于 interface 值详细的表示 [detailed blog post](https://research.swtch.com/interfaces) ，这里没必要在这里详细的阐述，但是有一个简单的摘要：
>interface 类型的变量存储是成对的：赋给变量的具体值，以及该值的类型描述符。更确切的说：值是实现 interface 的基础具体数据项，类型描述符描述该项的完整类型

先来看例子
```go
var r io.Reader
tty, err := os.OpenFile("/dev/tty", os.O_RDWR, 0)
if err != nil {
    return nil, err
}
r = tty
```
r 包含了两项（值，类型），（值是 tty，类型是 \* os.File）。 请注意，类型* os.File实现除Read之外的方法; 即使 interface 值仅提供对 Read  （io.Reader 中只有 Read 方法）方法的访问，内部值也包含有关该值的所有类型信息。所以我们还可以这么玩
```go
var w io.Writer
w = r.(io.Writer) // 类型断言
```
此赋值中的表达式是类型断言；它断言的内容是 r 中的项目也实现了 io.Writer，因此我们可以将它分配给 w。 赋值后，w 将包含（tty，* os.File）。还可以这样做：
```go
var empty interface{}
empty = w
```
我们的空 interface 将包含跟上面一样的两项（tty, *os.File）。这样一来，一个空接口可以保存任何值，并包含我们可能需要的有关该值的所有信息。

这里我们不需要`类型断言`，因为静态地知道 w 实现了空 interface。在我们将值从 Reader 分配到 Writer 的示例中，我们需要显式并使用`类型断言`，因为 Writer 的方法不是 Reader 的子集

> 一个重要的细节是 interface 内的对总是以（值，具体类型）的形式存在，而不能是以（值，interface 类型）的形式存在。 interface 不包含 interface 值。

## Relection 从 interface 值到反射对象
本质上讲，反射只是一种检查存储在接口变量中的类型和值对的机制。
在开始之前，这里有 [reflect](https://golang.org/pkg/reflect/) 包的两个类型 Type 和 Value 需要去了解。
这两种类型可以访问 interface 变量的内容，两个简单的函数:
* `reflect.TypeOf` 从 interface 值中检索类型 `reflect.Type`
* `reflect.ValueOf` 从 interface 值中检索值  `reflect.Value`

(另外，从 `reflect.Value` 可以很容易地获得 `reflect.Type` ，但是现在让我们将 Value 和 Type 概念分开)

让我们来看 TypeOf
```go
package main

import (
    "fmt"
    "reflect"
)

func main() {
    var x float64 = 3.4
    fmt.Println("type:", reflect.TypeOf(x))
}
```
程序输出：` type: float64 `

您可能想知道 interface 在哪里，因为程序看起来像是将 float64 变量 x 而不是 interface 值传递给 reflect.TypeOf。 但它确实就在那里; 正如 godoc 报道的那样，reflect.TypeOf 的签名包括一个空接口：
```go
// TypeOf returns the reflection Type of the value in the interface{}.
func TypeOf(i interface{}) Type
```
当我们调用 reflect.TypeOf（x）时，x 首先存储在一个空 interface 中，然后作为参数传递; reflect.TypeOf 函数取出该空 interface 以恢复类型信息。
同理，reflect.ValueOf 函数可以恢复该值。
```go
var x float64 = 3.4
fmt.Println("value:", reflect.ValueOf(x).String())
```
我们明确地调用 String 方法，因为默认情况下，fmt 包会去拿 reflect.Value 以显示其中的具体值。 String 方法没有。

`reflect.Type` 跟 `reflect.Value` 提供了大量方法供我们去检索或者操作他们。值得一提的是：`relect.Value` 有一个获取值类型的方法（返回该值的 `reflect.Type`）
**Kind 方法**
另一个是 Type 和 Value 都有一个 Kind 方法，它返回一个常量，指示存储的项目类型：Uint，Float64，Slice 等等
来看下面的例子
```go
var x float64 = 3.4
v := reflect.ValueOf(x)
fmt.Println("type:", v.Type())
fmt.Println("kind is float64:", v.Kind() == reflect.Float64)
fmt.Println("value:", v.Float())
```
输出
```
type: float64
kind is float64: true
value: 3.4
```
还有像 SetInt 和 SetFloat 这样的方法，我们将在 反射的 第三点钟讨论，

Reflection 库有一些需要单独提出来讲的地方：
* 为了保持 API 简洁，Value 的「getter」和「setter」方法是对可以保存值的最大类型进行操作的。比如所有符号整数都是 int64
```
var x uint8 = 'x'
v := reflect.ValueOf(x)
fmt.Println("type:", v.Type())                            // uint8.
fmt.Println("kind is uint8: ", v.Kind() == reflect.Uint8) // true.
x = uint8(v.Uint())                 // v.Uint returns a uint64.
```
* 反射对象的类型「Kind」描述了基础类型，而不是静态类型。 如果反射对象包含用户定义的整数类型的值，如下看代码就明白了
```go
type MyInt int
var x MyInt = 7
v := reflect.ValueOf(x)
```
v 的 Kind 仍然是 reflect.Int，尽管它的静态变量类型是 MyInt，不是 int；换句话说:即使 Type 可以，Kind 也无法区分 int 和 MyInt。

## 从反射对象到 interface 值
像物理反射一样，Go 中的反射会产生自己的反转。

给定一个 reflect.Value，我们可以使用 Interface 方法恢复 interface 值
```go
// Interface 方法以空 interface 类型返回 v 的值.
func (v Value) Interface() interface{}
```
所以就有如下写法
```go
y := v.Interface().(float64) // y 将有 float64 类型
fmt.Println(y)
```
输出由反射对象 v 表示的 float64 值。

不过，我们可以做得更好。 fmt.Println，fmt.Printf 等的参数都是 空interface 值传递，然后由 fmt 包内部处理，就像我们在前面的示例中所做的那样。 因此，正确打印 reflect.Value 的内容的姿势是将 Interface 方法的结果传递给格式化的打印即可：
```go
fmt.Println(v.Interface())
```
为什么不是 fmt.Println（v）？

因为 v 是一个 reflect.Value;我们想要的是 v 的具体值。

同样的道理，我们也没有必要将 v.Interface（）的结果类型断言为 float64; 空 interface 值内部具有具体值的类型信息，Printf将恢复它。

简而言之，Interface 方法是 ValueOf 函数的反转，除了它的结果总是静态类型 空 interafce (`interface {}`)。

最后重声一边：Reflection『反射』 是从 interface 值到反射对象再返回到 interface 值。

## 要修改反射对象，该值可设置
第三定律是最微妙和最令人困惑的，但是如果我们从第一原则开始就很容易理解

我们先研究一下下面错误的代码
```go
var x float64 = 3.4
v := reflect.ValueOf(x)
v.SetFloat(7.1) // Error: will panic.
```
如果运行会 panic(可以理解为 php throw) 到费解的信息：
```go
panic: reflect.Value.SetFloat using unaddressable value
```
问题不在 7.1 不可寻址，而是 v 是不可设置的。
`可设置性` 是反射值的属性，并非所有反射值都具有它。我们通过 `CanSet` 方法来判断；列如打印 v 的 `可设置性`：
```go
var x float64 = 3.4
v := reflect.ValueOf(x)
fmt.Println("settability of v:", v.CanSet()) // 打印:settability of v: false
```
在不可设置的值上调用 `Set` 方法是错误的，那么怎么样才是可设置的呢？

可设置性 有点像可寻址，但是又比他更严格。
反射对象可以修改被反射对象的实际存储的属性。 可设定性取决于反射对象是否包含原始项。『PS 说了那么多，其实就是要传 x 的地址、而不能传 x 的 copy』

我们将x的副本传递给reflect.ValueOf，因此作为reflect.ValueOf的参数创建的接口值是x的副本，而不是x本身。 因此，如果声明
```go
v.SetFloat(7.1)
```
被允许成功，它不会更新x，即使v看起来像是从x创建的。 相反，它会更新存储在反射值中的x的副本，而x本身将不受影响。 这将是混乱和无用的，因此它是非法的，可设置性是用于避免此问题的属性。

如果这看起来很奇怪，那就不是了。 这实际上是一种不寻常的服装中常见的情况。 考虑将x传递给函数：
```go
f(x)
```
如果我们不想 f() 修改 x 的值,我们就传 x 的 copy,反之我们要传 地址。

只有传地址，给反射库一个指向要修改值得指针，反射库才能够修改值。

我们在看如下代码
```go
var x float64 = 3.4
p := reflect.ValueOf(&x) // Note: 传 x 的地址.
fmt.Println("type of p:", p.Type())
fmt.Println("settability of p:", p.CanSet())
```
打印
```
type of p: *float64
settability of p: false
```
我们看到反射对象 p 不可设置，但它不是我们要设置的 p，它是（实际上）* p 是指向我们要修改值得指针， 为了得到 p 指向的内容，我们调用值的 Elem 方法，它通过指针间接，并将结果保存在一个名为 v的反射值中。

正确的姿势应该是：
```go
v := p.Elem()
fmt.Println("settability of v:", v.CanSet())
```
打印
```
settability of v: true
```
反射可能很难理解，但它正在做语言所做的事；尽管通过反射的 Type 和 Value 可以模拟正在发生的事情，请记住：需要通过传递地址才能修改他们所代表的内容
> Reflection can be hard to understand but it's doing exactly what the language does, albeit through reflection Types and Values that can disguise what's going on.

「ps：感觉这句译得不对，望大佬纠正」

> ps: disguise 是伪装的意思，这里用伪装似乎不合适，我感觉作者要表达的意思是：反射的 Type Value 可以伪装 正在发生的事 = 反射的 Type Value 可以获取程序运行时的状态