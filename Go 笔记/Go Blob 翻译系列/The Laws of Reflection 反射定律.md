[TOC]

### Introduction | 介绍

>  Reflection in computing is the ability of a program to examine its own structure, particularly through types; it's a form of metaprogramming. It's also a great source of confusion.

计算反射是程序检查其自身结构的能力，尤其是通过类型。这是元编程的一种形式。这也是造成混乱的重要原因。

> In this article we attempt to clarify things by explaining how reflection works in Go. Each language's reflection model is different (and many languages don't support it at all), but this article is about Go, so for the rest of this article the word "reflection" should be taken to mean "reflection in Go".

在本文中，我们尝试通过解释反射在 Go 中的工作原理来澄清事物。每种语言的反射模型是不同的（许多语言根本不支持它），但是本文是关于 Go 的，因此对于本文的其余部分，应将「反射」一词理解为「Go 中的反射」。

### Types and interfaces | 类型和接口

> Because reflection builds on the type system, let's start with a refresher about types in Go.

因为反射建立在类型系统上，所以让我们从Go语言中的类型开始。

> Go is statically typed. Every variable has a static type, that is, exactly one type known and fixed at compile time: `int`, `float32`, `*MyType`, `[]byte`, and so on. If we declare

Go是静态类型的。每个变量都有一个静态类型，也就是在编译时已知并固定的一种类型：int，float32，* MyType，[] byte等。如果我们声明

```go
type MyInt int

var i int
var j MyInt
```

> then `i` has type `int` and `j` has type `MyInt`. The variables `i` and `j` have distinct static types and, although they have the same underlying type, they cannot be assigned to one another without a conversion.

那么 i 类型为 int，j 的类型为 MyInt。变量 i 和 j 具有不同的静态类型，尽管它们具有相同的基础类型，但是如果不进行转换就无法将它们彼此赋值。

> One important category of type is interface types, which represent fixed sets of methods. An interface variable can store any concrete (non-interface) value as long as that value implements the interface's methods. A well-known pair of examples is `io.Reader` and `io.Writer`, the types `Reader` and `Writer` from the [io package](https://golang.org/pkg/io/):

类型的一个重要类别是接口类型，它表示固定的方法集。接口变量可以存储任何具体的（非接口）值，只要该值实现接口的方法即可。一对著名的示例是 io.Reader 和 io.Writer，它们是 [io 包](https://golang.org/pkg/io/) 中的 Reader 和 Writer 类型：

```go
// Reader is the interface that wraps the basic Read method.
type Reader interface {
    Read(p []byte) (n int, err error)
}

// Writer is the interface that wraps the basic Write method.
type Writer interface {
    Write(p []byte) (n int, err error)
}
```

> Any type that implements a `Read` (or `Write`) method with this signature is said to implement `io.Reader` (or `io.Writer`). For the purposes of this discussion, that means that a variable of type `io.Reader` can hold any value whose type has a `Read` method:

任何使用此签名实现Read（或Write）方法的类型都称为io.Reader（或io.Writer）。出于讨论的目的，这意味着io.Reader类型的变量可以保存其类型具有Read方法的任何值：

```go
var r io.Reader
r = os.Stdin
r = bufio.NewReader(r)
r = new(bytes.Buffer)
// and so on
```

> It's important to be clear that whatever concrete value `r` may hold, `r`'s type is always `io.Reader`: Go is statically typed and the static type of `r` is `io.Reader`.

重要的是要清楚，无论 r 可能包含什么具体值，r 的类型始终是 io.Reader：Go 是静态类型的，而 r 的静态类型是 io.Reader。

> An extremely important example of an interface type is the empty interface:

接口类型的一个非常重要的示例是空接口：

```go
interface{}
```

> It represents the empty set of methods and is satisfied by any value at all, since any value has zero or more methods.

它表示空方法集，并且由任何值完全满足，因为任何值具有零个或多个方法。

> Some people say that Go's interfaces are dynamically typed, but that is misleading. They are statically typed: a variable of interface type always has the same static type, and even though at run time the value stored in the interface variable may change type, that value will always satisfy the interface.

有人说 Go 的接口是动态类型的，但这会产生误导。它们是静态类型的：接口类型的变量始终具有相同的静态类型，即使在运行时存储在接口变量中的值可能会更改类型，该值也将始终满足接口的要求。

> We need to be precise about all this because reflection and interfaces are closely related.

我们需要确切地认识到这些事情，因为反射和接口密切相关。

### The representation of an interface | 接口的表示

>  Russ Cox has written a [detailed blog post](https://research.swtch.com/2009/12/go-data-structures-interfaces.html) about the representation of interface values in Go. It's not necessary to repeat the full story here, but a simplified summary is in order.

Russ Cox 写了一篇 [详细的博客文章](https://research.swtch.com/2009/12/go-data-structures-interfaces.html)，内容涉及 Go 中接口值的表示。这里没有必要复述完整的内容，但是有必要简单概括一下。

>  A variable of interface type stores a pair: the concrete value assigned to the variable, and that value's type descriptor. To be more precise, the value is the underlying concrete data item that implements the interface and the type describes the full type of that item. For instance, after

接口类型的变量存储一对：分配给该变量的具体值以及该值的类型描述符。更确切地说，该值是实现接口的基础具体数据项，而类型则描述该项的完整类型。例如

```
var r io.Reader
tty, err := os.OpenFile("/dev/tty", os.O_RDWR, 0)
if err != nil {
    return nil, err
}
r = tty
```

> `r` contains, schematically, the (value, type) pair, (`tty`, `*os.File`). Notice that the type `*os.File` implements methods other than `Read`; even though the interface value provides access only to the `Read` method, the value inside carries all the type information about that value. That's why we can do things like this:

r 示意性地包含（值，类型）对（tty，* os.File）。注意，* os.File 类型实现了 Read 以外的方法；即使接口值仅提供对 Read 方法的访问，但内部的值仍包含有关该值的所有类型信息。这就是为什么我们可以做这样的事情：

```
var w io.Writer
w = r.(io.Writer)
```

> The expression in this assignment is a type assertion; what it asserts is that the item inside `r` also implements `io.Writer`, and so we can assign it to `w`. After the assignment, `w` will contain the pair (`tty`, `*os.File`). That's the same pair as was held in `r`. The static type of the interface determines what methods may be invoked with an interface variable, even though the concrete value inside may have a larger set of methods.

此赋值中的表达式是类型断言。它断言 r 也实现了 io.Writer，因此我们可以将其分配给 w。分配后，w 将包含（值，类型）对（tty，* os.File）。这与在 r 中持有的对相同。接口的静态类型决定可以使用接口变量调用哪些方法，即使内部的具体值可能具有更大的方法集。

> Continuing, we can do this:

继续，我们可以这样做：

```
var empty interface{}
empty = w
```

> and our empty interface value `empty` will again contain that same pair, (`tty`, `*os.File`). That's handy: an empty interface can hold any value and contains all the information we could ever need about that value.

并且我们的空接口值 empty 将再次包含同一对（tty，* os.File）。这很方便：一个空接口可以保存任何值，并包含我们可能需要的有关该值的所有信息。

> (We don't need a type assertion here because it's known statically that `w` satisfies the empty interface. In the example where we moved a value from a `Reader` to a `Writer`, we needed to be explicit and use a type assertion because `Writer`'s methods are not a subset of `Reader`'s.)

（我们在这里不需要类型声明，因为静态地知道 w 可以满足空接口。在将值从 Reader 移到 Writer 的示例中，我们需要显式并使用类型声明，因为 Writer 的方法不是 Reader 的子集。）

> One important detail is that the pair inside an interface always has the form (value, concrete type) and cannot have the form (value, interface type). Interfaces do not hold interface values.

一个重要的细节是，接口内的对始终具有形式（值，具体类型），而不能具有形式（值，接口类型）。接口不保存接口值。

> Now we're ready to reflect.

现在我们准备好反射了。

### The first law of reflection | 反射第一个定律

#### Reflection goes from interface value to reflection object. | 反射从接口值变为反射对象

> At the basic level, reflection is just a mechanism to examine the type and value pair stored inside an interface variable. To get started, there are two types we need to know about in [package reflect](https://golang.org/pkg/reflect/): [Type](https://golang.org/pkg/reflect/#Type) and [Value](https://golang.org/pkg/reflect/#Value). Those two types give access to the contents of an interface variable, and two simple functions, called `reflect.TypeOf` and `reflect.ValueOf`, retrieve `reflect.Type` and `reflect.Value` pieces out of an interface value. (Also, from the `reflect.Value` it's easy to get to the `reflect.Type`, but let's keep the `Value` and `Type` concepts separate for now.)

在基本层面上，反射只是一种检查存储在接口变量中的类型和值对的机制。首先，我们需要了解在 [reflect](https://golang.org/pkg/reflect/) 包中的两种类型：[Type](https://golang.org/pkg/reflect/#Type) and [Value](https://golang.org/pkg/reflect/#Value)。这两种类型提供对接口变量内容的访问，还有两个简单的函数，称为 `reflect.TypeOf` 和 `reflect.ValueOf`，从接口值中获取 `reflect.Type` 和 `reflect.Value` 部分。 （此外，从 `reflect.Value` 可以很容易地到达 `reflect.Type`，但是现在让 `Value` 和 `Type` 概念保持分离。）

> Let's start with `TypeOf`:

让我们从 `TypeOf` 开始

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

> This program prints

该程序打印

```
type: float64
```

> You might be wondering where the interface is here, since the program looks like it's passing the `float64` variable `x`, not an interface value, to `reflect.TypeOf`. But it's there; as [godoc reports](https://golang.org/pkg/reflect/#TypeOf), the signature of `reflect.TypeOf` includes an empty interface:

你可能想知道这里的接口是什么，因为该程序看起来像在传递 `float64` 变量 `x` 而不是接口值给 `reflect.TypeOf`。但是在 [godoc](https://golang.org/pkg/reflect/#TypeOf) 中指出，`reflect.TypeOf` 的签名包括一个空接口：

```
// TypeOf returns the reflection Type of the value in the interface{}.
func TypeOf(i interface{}) Type
```

> When we call `reflect.TypeOf(x)`, `x` is first stored in an empty interface, which is then passed as the argument; `reflect.TypeOf` unpacks that empty interface to recover the type information.

当我们调用 `reflect.TypeOf（x）` 时，`x` 首先存储在一个空接口中，然后将其作为参数传递； `Reflection.TypeOf` 解压缩该空接口以恢复类型信息。

> The `reflect.ValueOf` function, of course, recovers the value (from here on we'll elide the boilerplate and focus just on the executable code):

当然，`reflect.ValueOf` 函数可以恢复值（从这里开始，我们将省略样板并只关注可执行代码）：

```go
var x float64 = 3.4
fmt.Println("value:", reflect.ValueOf(x).String())
```

>  prints

打印

```
value: <float64 Value>
```

> (We call the `String` method explicitly because by default the `fmt` package digs into a `reflect.Value` to show the concrete value inside. The `String` method does not.)

（我们明确地调用 `String` 方法，因为默认情况下，`fmt` 包会深入解析 `reflect.Value` 以显示其中的具体值。String 方法不会。）

> Both `reflect.Type` and `reflect.Value` have lots of methods to let us examine and manipulate them. One important example is that `Value` has a `Type` method that returns the `Type` of a `reflect.Value`. Another is that both `Type` and `Value` have a `Kind` method that returns a constant indicating what sort of item is stored: `Uint`, `Float64`, `Slice`, and so on. Also methods on `Value` with names like `Int` and `Float` let us grab values (as `int64` and `float64`) stored inside:

`reflect.Type` 和 `reflect.Value` 都有很多方法可以让我们检查和操作它们。一个重要的例子是 `Value` 具有 Type 方法，该方法返回 `reflect.Value` 的 `Type`。另一个是 `Type` 和 `Value` 都有 `Kind` 方法，该方法返回一个常量，指示存储的项目类型：`Uint`，`Float64`，`Slice` 等。同样，使用诸如 `Int` 和 `Float` 之类的 `Value` 方法，还可以获取存储在其中的值（如 `int64` 和 `float64`）：

```go
var x float64 = 3.4
v := reflect.ValueOf(x)
fmt.Println("type:", v.Type())
fmt.Println("kind is float64:", v.Kind() == reflect.Float64)
fmt.Println("value:", v.Float())
```

> prints

打印

```
type: float64
kind is float64: true
value: 3.4
```

> There are also methods like `SetInt` and `SetFloat` but to use them we need to understand settability, the subject of the third law of reflection, discussed below.

还有诸如 `SetInt` 和 `SetFloat` 之类的方法，但是要使用它们，我们需要了解可沉降性，这是第三反射定律的主题，下面将进行讨论。

> The reflection library has a couple of properties worth singling out. First, to keep the API simple, the "getter" and "setter" methods of `Value` operate on the largest type that can hold the value: `int64` for all the signed integers, for instance. That is, the `Int` method of `Value` returns an `int64` and the `SetInt` value takes an `int64`; it may be necessary to convert to the actual type involved:

反射库具有几个值得一提的属性。首先，为使 API 保持简单，`Value` 的 “getter” 和“setter”方法在可以容纳该值的最大类型上运行：例如，所有有符号整数的 `int64`。也就是说，`Value` 的 `Int` 方法返回一个 `int64`，而 `SetInt` 值采用一个 `int64`；可能需要转换为涉及的实际类型：

```go
var x uint8 = 'x'
v := reflect.ValueOf(x)
fmt.Println("type:", v.Type())                            // uint8.
fmt.Println("kind is uint8: ", v.Kind() == reflect.Uint8) // true.
x = uint8(v.Uint())                                       // v.Uint returns a uint64.
```

> The second property is that the `Kind` of a reflection object describes the underlying type, not the static type. If a reflection object contains a value of a user-defined integer type, as in

第二个属性是反射对象的种类描述基础类型，而不是静态类型。如果反射对象包含用户定义的整数类型的值，例如

```go
type MyInt int
var x MyInt = 7
v := reflect.ValueOf(x)
```

> the `Kind` of `v` is still `reflect.Int`, even though the static type of `x` is `MyInt`, not `int`. In other words, the `Kind` cannot discriminate an int from a `MyInt` even though the `Type` can.

v 的 `Kind` 仍然是 `reflect.Int`，即使 x 的静态类型是 `MyInt`，而不是 `int`。换句话说，即使 `Type` 可以，`Kind` 也无法将 My 和 InIn 区别开。

### The second law of reflection | 反射第二定律

#### Reflection goes from reflection object to interface value. | 反射从反射对象到接口值。

> Like physical reflection, reflection in Go generates its own inverse.

像物理反射一样，Go 中的反射会生成自己的反面。

> Given a `reflect.Value` we can recover an interface value using the `Interface` method; in effect the method packs the type and value information back into an interface representation and returns the result:

给定一个 reflect.Value，我们可以使用 Interface 方法恢复接口值；实际上，该方法将类型和值信息打包回接口表示形式并返回结果：

```go
// Interface returns v's value as an interface{}.
func (v Value) Interface() interface{}
```

> As a consequence we can say

结果，我们可以说

```go
y := v.Interface().(float64) // y will have type float64.
fmt.Println(y)
```

> to print the `float64` value represented by the reflection object `v`.

打印反射对象 v 表示的 float64 值。

> We can do even better, though. The arguments to `fmt.Println`, `fmt.Printf` and so on are all passed as empty interface values, which are then unpacked by the `fmt` package internally just as we have been doing in the previous examples. Therefore all it takes to print the contents of a `reflect.Value` correctly is to pass the result of the `Interface` method to the formatted print routine:

不过，我们可以做得更好。 fmt.Println，fmt.Printf 等的参数都作为空接口值传递，然后像我们在前面的示例中所做的那样，在内部由 fmt 软件包解压缩。因此，正确打印 reflect.Value 的内容所要做的就是将 Interface 方法的结果传递给格式化的打印例程：

```go
fmt.Println(v.Interface())
```

> (Why not `fmt.Println(v)`? Because `v` is a `reflect.Value`; we want the concrete value it holds.) Since our value is a `float64`, we can even use a floating-point format if we want:

（为什么不使用 fmt.Println(v)？因为 v 是 reflect.Value；我们想要它包含的具体值。）由于我们的值是 float64，因此如果需要，我们甚至可以使用浮点格式：

```go
fmt.Printf("value is %7.1e\n", v.Interface())
```

> and get in this case

在这种情况下

```
3.4e+00
```

> Again, there's no need to type-assert the result of `v.Interface()` to `float64`; the empty interface value has the concrete value's type information inside and `Printf` will recover it.

同样，无需将 v.Interface() 的结果类型声明为 float64；空接口值内部具有具体值的类型信息，Printf 将对其进行恢复。

> In short, the `Interface` method is the inverse of the `ValueOf` function, except that its result is always of static type `interface{}`.

简而言之，Interface 方法与 ValueOf 函数相反，但其结果始终是静态类型 interface {}。

> Reiterating: Reflection goes from interface values to reflection objects and back again.

重申：反射从接口值到反射对象，然后再到接口值。

### The third law of reflection | 反射第三定律

### To modify a reflection object, the value must be settable. | 要修改反射对象，该值必须可设置。

> The third law is the most subtle and confusing, but it's easy enough to understand if we start from first principles.

第三定律是最微妙和令人困惑的，但是如果我们从第一条原则开始，就很容易理解。

> Here is some code that does not work, but is worth studying.

这是一些无效的代码，但值得研究。

```go
var x float64 = 3.4
v := reflect.ValueOf(x)
v.SetFloat(7.1) // Error: will panic.
```

> If you run this code, it will panic with the cryptic message

如果您运行此代码，它将会 panic 伴随着一些隐秘信息

```
panic: reflect.Value.SetFloat using unaddressable value
```

> The problem is not that the value `7.1` is not addressable; it's that `v` is not settable. Settability is a property of a reflection `Value`, and not all reflection `Values` have it.

问题不在于值 7.1 是不可寻址的。而是因为 v 不可设置的。可设置性是反射值的属性，并非所有反射值都具有它。

> The `CanSet` method of `Value` reports the settability of a `Value`; in our case,

值的 `CanSet` 方法报告值的可设置性；在上面的例子中

```go
var x float64 = 3.4
v := reflect.ValueOf(x)
fmt.Println("settability of v:", v.CanSet())
```

> prints

打印

```
settability of v: false
```

> It is an error to call a `Set` method on an non-settable `Value`. But what is settability?

在不可设置的值上调用 Set 方法是错误的。但是什么是可设置的呢？

> Settability is a bit like addressability, but stricter. It's the property that a reflection object can modify the actual storage that was used to create the reflection object. Settability is determined by whether the reflection object holds the original item. When we say

可设置性有点像可寻址性，但是更严格。它是反射对象可以修改用于创建反射对象的实际存储的属性。可设置性由反射对象是否保留原始项目确定。当我们说

```go
var x float64 = 3.4
v := reflect.ValueOf(x)
```

> we pass a copy of `x` to `reflect.ValueOf`, so the interface value created as the argument to `reflect.ValueOf` is a copy of `x`, not `x` itself. Thus, if the statement

我们将 x 的副本传递给 reflect.ValueOf，因此，作为 reflect.ValueOf 的参数创建的接口值是 x 的副本，而不是 x 本身。因此，如果声明

```go
v.SetFloat(7.1)
```

> were allowed to succeed, it would not update `x`, even though `v` looks like it was created from `x`. Instead, it would update the copy of `x` stored inside the reflection value and `x` itself would be unaffected. That would be confusing and useless, so it is illegal, and settability is the property used to avoid this issue.

被允许成功，即使 v 看起来是从 x 创建的，它也不会更新 x。相反，它将更新存储在反射值内的 x 的副本，并且 x 本身将不受影响。那将是混乱和无用的，因此是非法的，可设置性是用来避免此问题的属性。

> If this seems bizarre, it's not. It's actually a familiar situation in unusual garb. Think of passing `x` to a function:

如果这看起来很奇怪，但是并不是。实际上，这是很常见的情况。考虑将 x 传递给函数：

```
f(x)
```

> We would not expect `f` to be able to modify `x` because we passed a copy of `x`'s value, not `x` itself. If we want `f` to modify `x` directly we must pass our function the address of `x` (that is, a pointer to `x`):

我们不希望 f 能够修改 x，因为我们传递了 x 值的副本，而不是 x 本身。如果我们想让 f 直接修改 x，则必须将 x 的地址（即指向 x 的指针）传递给函数：

```
f(&x)
```

> This is straightforward and familiar, and reflection works the same way. If we want to modify `x` by reflection, we must give the reflection library a pointer to the value we want to modify.

这是直接且熟悉的，并且反射的工作方式相同。如果要通过反射修改 x，则必须为反射库提供指向要修改的值的指针。

> Let's do that. First we initialize `x` as usual and then create a reflection value that points to it, called `p`.

来做吧。首先，我们像往常一样初始化x，然后创建一个指向它的反射值，称为p。

```go
var x float64 = 3.4
p := reflect.ValueOf(&x) // Note: take the address of x.
fmt.Println("type of p:", p.Type())
fmt.Println("settability of p:", p.CanSet())
```

> The output so far is

到目前为止的输出是

```
type of p: *float64
settability of p: false
```

> The reflection object `p` isn't settable, but it's not `p` we want to set, it's (in effect) `*p`. To get to what `p` points to, we call the `Elem` method of `Value`, which indirects through the pointer, and save the result in a reflection `Value` called `v`:

反射对象 p 是不可设置的，但不是我们要设置的 p，实际上是 *p。为了得到 p 所指向的内容，我们调用 Value 的 Elem 方法，该方法通过指针进行间接操作，并将结果保存在名为 v 的反射 Value 中：

```go
v := p.Elem()
fmt.Println("settability of v:", v.CanSet())
```

> Now `v` is a settable reflection object, as the output demonstrates,

现在 v 是一个可设置的反射对象，如输出所示，

```
settability of v: true
```

> and since it represents `x`, we are finally able to use `v.SetFloat` to modify the value of `x`:

由于它代表 x，我们最终可以使用 v.SetFloat 修改 x 的值：

```go
v.SetFloat(7.1)
fmt.Println(v.Interface())
fmt.Println(x)
```

> The output, as expected, is

预期的输出是

```
7.1
7.1
```

> Reflection can be hard to understand but it's doing exactly what the language does, albeit through reflection `Types` and `Values` that can disguise what's going on. Just keep in mind that reflection Values need the address of something in order to modify what they represent.

反射可能很难理解，但是它确实在做语言的工作，尽管通过反射可以掩盖正在发生的事情的类型和值。请记住，反射值需要某些内容的地址才能修改其表示的内容。

### Structs | 结构体

> In our previous example `v` wasn't a pointer itself, it was just derived from one. A common way for this situation to arise is when using reflection to modify the fields of a structure. As long as we have the address of the structure, we can modify its fields.

在我们前面的示例中，v 本身并不是指针，它只是从一个指针派生的。发生这种情况的常见方法是使用反射修改结构的字段。只要有了结构的地址，就可以修改其字段。

> Here's a simple example that analyzes a struct value, `t`. We create the reflection object with the address of the struct because we'll want to modify it later. Then we set `typeOfT` to its type and iterate over the fields using straightforward method calls (see [package reflect](https://golang.org/pkg/reflect/) for details). Note that we extract the names of the fields from the struct type, but the fields themselves are regular `reflect.Value` objects.

这是一个分析结构值 t 的简单示例。我们使用结构的地址创建反射对象，因为稍后将要对其进行修改。然后，将 typeOfT 设置为其类型，并使用简单的方法调用对字段进行迭代（有关详细信息，请参见包反射）。请注意，我们从结构类型中提取了字段的名称，但是字段本身是常规的 reflect.Value 对象。

```go
type T struct {
    A int
    B string
}
t := T{23, "skidoo"}
s := reflect.ValueOf(&t).Elem()
typeOfT := s.Type()
for i := 0; i < s.NumField(); i++ {
    f := s.Field(i)
    fmt.Printf("%d: %s %s = %v\n", i,
        typeOfT.Field(i).Name, f.Type(), f.Interface())
}
```

> The output of this program is

```
0: A int = 23
1: B string = skidoo
```

> There's one more point about settability introduced in passing here: the field names of `T` are upper case (exported) because only exported fields of a struct are settable.

在此处传递的内容还涉及可设置性的另一点：T 的字段名是大写的（已导出），因为仅可导出结构的导出字段。

> Because `s` contains a settable reflection object, we can modify the fields of the structure.

由于 s 包含一个可设置的反射对象，因此我们可以修改结构的字段。

```go
s.Field(0).SetInt(77)
s.Field(1).SetString("Sunset Strip")
fmt.Println("t is now", t)
```

> And here's the result:

结果如下：

```
t is now {77 Sunset Strip}
```

> If we modified the program so that `s` was created from `t`, not `&t`, the calls to `SetInt` and `SetString` would fail as the fields of `t` would not be settable.

如果我们修改程序，以便从 t 而不是＆t 创建 s，则对 SetInt 和 SetString 的调用将失败，因为 t 的字段不可设置。

### Conclusion | 结论

> Here again are the laws of reflection:
>
> - Reflection goes from interface value to reflection object.
> - Reflection goes from reflection object to interface value.
> - To modify a reflection object, the value must be settable.

再次重申反射的三大定律：

- 反射从接口值变为反射对象。
- 反射从反射对象到接口值。
- 要修改反射对象，该值必须可设置。

> Once you understand these laws reflection in Go becomes much easier to use, although it remains subtle. It's a powerful tool that should be used with care and avoided unless strictly necessary.

一旦理解了这些定律，尽管 Go 中的反射仍然很微妙，但它变得更易于使用。这是一个功能强大的工具，应谨慎使用，除非绝对必要，否则应避免使用。

> There's plenty more to reflection that we haven't covered — sending and receiving on channels, allocating memory, using slices and maps, calling methods and functions — but this post is long enough. We'll cover some of those topics in a later article.

关于反射的很多内容我们还没有涉及到，包括在通道上发送和接收，分配内存，使用切片和映射，调用方法和函数，但是这篇文章足够长。我们将在以后的文章中介绍其中一些主题。

### 原文地址

https://blog.golang.org/laws-of-reflection