反射是众多编程语言中的一个非常实用的功能，它是一种能够自描述、自控制的应用，Go语言也对反射提供了友好的支持。

Go语言中使用反射可以在编译时不知道类型的情况下更新变量，在运行时查看值、调用方法以及直接对他们的布局进行操作。

由于反射是建立在类型系统（type system）上的，所以我们先来复习一下Go语言中的类型。

**Go语言中的类型**

Go语言是一门静态类型的语言，每个变量都有一个静态类型，类型在编译的时候确定下来。

```go
type MyInt int
var i int
var j MyInt
```

变量 i 的类型是 int，变量 j 的类型是 MyInt，虽然它们有着相同的基本类型，但静态类型却不一样，在没有类型转换的情况下，它们之间无法互相赋值。

接口是一个重要的类型，它意味着一个确定的方法集合，一个接口变量可以存储任何实现了接口的方法的具体值（除了接口本身），例如 io.Reader 和 io.Writer：

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

如果一个类型声明实现了 Reader（或 Writer）方法，那么它便实现了 io.Reader（或 io.Writer），这意味着一个 io.Reader 的变量可以持有任何一个实现了 Read 方法的的类型的值。

```go
var r io.Reader
r = os.Stdin
r = bufio.NewReader(r)
r = new(bytes.Buffer)
// and so on
```

必须要弄清楚的一点是，不管变量 r 中的具体值是什么，r 的类型永远是 io.Reader，由于Go语言是静态类型的，r 的静态类型就是 io.Reader。

在接口类型中有一个极为重要的例子——空接口：

```
interface{}
```

它表示了一个空的方法集，一切值都可以满足它，因为它们都有零值或方法。

有人说Go语言的接口是动态类型，这是错误的，它们都是静态类型，虽然在运行时中，接口变量存储的值也许会变，但接口变量的类型是不会变的。我们必须精确地了解这些，因为反射与接口是密切相关的。

关于接口我们就介绍到这里，下面我们看看Go语言的反射三定律。

**反射第一定律：反射可以将“接口类型变量”转换为“反射类型对象”**

注：这里反射类型指 reflect.Type 和 reflect.Value。

从使用方法上来讲，反射提供了一种机制，允许程序在运行时检查接口变量内部存储的 (value, type) 对。

在最开始，我们先了解下 reflect 包的两种类型 Type 和 Value，这两种类型使访问接口内的数据成为可能，它们对应两个简单的方法，分别是 reflect.TypeOf 和 reflect.ValueOf，分别用来读取接口变量的 reflect.Type 和 reflect.Value 部分。

当然，从 reflect.Value 也很容易获取到 reflect.Type，目前我们先将它们分开。

首先，我们下看 reflect.TypeOf：

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

运行结果如下：

```
type: float64
```

大家可能会疑惑，为什么没看到接口？这段代码看起来只是把一个 float64 类型的变量 x 传递给 reflect.TypeOf 并没有传递接口。其实在 reflect.TypeOf 的函数签名里包含一个空接口：

```go
// TypeOf returns the reflection Type of the value in the interface{}. 
func TypeOf(i interface{}) Type
```

我们调用 reflect.TypeOf(x) 时，x 被存储在一个空接口变量中被传递过去，然后 reflect.TypeOf 对空接口变量进行拆解，恢复其类型信息。

函数 reflect.ValueOf 也会对底层的值进行恢复：

```go
package main 
import (
  "fmt"
  "reflect"
) 
func main() {
  var x float64 = 3.4
  fmt.Println("value:", reflect.ValueOf(x))
}
```

运行结果如下：

```
value: 3.4
```

类型 reflect.Type 和 reflect.Value 都有很多方法，我们可以检查和使用它们，这里我们举几个例子。

类型 reflect.Value 有一个方法 Type()，它会返回一个 reflect.Type 类型的对象。

Type 和 Value 都有一个名为 Kind 的方法，它会返回一个常量，表示底层数据的类型，常见值有：Uint、Float64、Slice 等。

Value 类型也有一些类似于 Int、Float 的方法，用来提取底层的数据：

- Int 方法用来提取 int64
- Float 方法用来提取 float64，示例代码如下：

```go
package main 
import (
  "fmt" 
  "reflect"
)
func main() {
  var x float64 = 3.4
  v := reflect.ValueOf(x) 
  fmt.Println("type:", v.Type())
  fmt.Println("kind is float64:", v.Kind() == reflect.Float64) 
  fmt.Println("value:", v.Float()) 
}
```

运行结果如下：

```
type: float64
kind is float64: true 
value: 3.4
```

还有一些用来修改数据的方法，比如 SetInt、SetFloat。在介绍它们之前，我们要先理解“可修改性”（settability），这一特性会在下面进行详细说明。

反射库提供了很多值得列出来单独讨论的属性，下面就来介绍一下。

首先是介绍下 Value 的 getter 和 setter 方法，为了保证 API 的精简，这两个方法操作的是某一组类型范围最大的那个。比如，处理任何含符号整型数，都使用 int64，也就是说 Value 类型的 Int 方法返回值为 int64 类型，SetInt 方法接收的参数类型也是 int64 类型。实际使用时，可能需要转化为实际的类型：

```go
package main 
import (
  "fmt"    "reflect" ) func main() {    var x uint8 = 'x'    v := reflect.ValueOf(x)    fmt.Println("type:", v.Type())                            // uint8.    fmt.Println("kind is uint8: ", v.Kind() == reflect.Uint8) // true.    x = uint8(v.Uint())                                       // v.Uint returns a uint64. }
```

运行结果如下：

```
type: uint8 kind is uint8: true
```

其次，反射对象的 Kind 方法描述的是基础类型，而不是静态类型。如果一个反射对象包含了用户定义类型的值，如下所示：

```go
type MyInt int
var x MyInt = 7
v := reflect.ValueOf(x)
```

上面的代码中，虽然变量 v 的静态类型是 MyInt，而不是 int，但 Kind 方法仍然会返回 reflect.Int。换句话说 Kind 方法不会像 Type 方法一样区分 MyInt 和 int。

**反射第二定律：反射可以将“反射类型对象”转换为“接口类型变量”**

和物理学中的反射类似，Go语言中的反射也能创造自己反面类型的对象。

根据一个 reflect.Value 类型的变量，我们可以使用 Interface 方法恢复其接口类型的值。事实上，这个方法会把 type 和 value 信息打包并填充到一个接口变量中，然后返回。

其函数声明如下：

```go
// Interface returns v's value as an interface{}. 
func (v Value) Interface() interface{}
```

然后，我们可以通过断言，恢复底层的具体值：

```go
y := v.Interface().(float64) // y will have type float64. 
fmt.Println(y)
```

上面这段代码会打印出一个 float64 类型的值，也就是反射类型变量 v 所代表的值。

事实上，我们可以更好地利用这一特性，标准库中的 fmt.Println 和 fmt.Printf 等函数都接收空接口变量作为参数，fmt 包内部会对接口变量进行拆包，因此 fmt 包的打印函数在打印 reflect.Value 类型变量的数据时，只需要把 Interface 方法的结果传给格式化打印程序：

```go
fmt.Println(v.Interface())
```

为什么不直接使用 fmt.Println(v)？因为 v 的类型是 reflect.Value，我们需要的是它的具体值，由于值的类型是 float64，我们也可以用浮点格式化打印它：

```go
fmt.Printf("value is %7.1e\n", v.Interface())
```

运行结果如下：

```
3.4e+00
```

同样，这次也不需要对 v.Interface() 的结果进行类型断言，空接口值内部包含了具体值的类型信息，Printf 函数会恢复类型信息。

简单来说 Interface 方法和 ValueOf 函数作用恰好相反，唯一一点是，返回值的静态类型是 interface{}。

Go的反射机制可以将“接口类型的变量”转换为“反射类型的对象”，然后再将“反射类型对象”转换过去。

**反射第三定律：如果要修改“反射类型对象”其值必须是“可写的”**

这条定律很微妙，也很容易让人迷惑，但是如果从第一条定律开始看，应该比较容易理解。

下面这段代码虽然不能正常工作，但是非常值得研究：

```go
var x float64 = 3.4
v := reflect.ValueOf(x)
v.SetFloat(7.1) // Error: will panic
```

如果运行这段代码，它会抛出一个奇怪的异常：

```go
panic: reflect: reflect.flag.mustBeAssignable using unaddressable value
```

这里问题不在于值7.1 不能被寻址，而是因为变量 v 是“不可写的”，“可写性”是反射类型变量的一个属性，但不是所有的反射类型变量都拥有这个属性。

我们可以通过 CanSet 方法检查一个 reflect.Value 类型变量的“可写性”，对于上面的例子，可以这样写：

```go
package main 
import (
  "fmt" 
  "reflect"
) 
func main() { 
  var x float64 = 3.4 
  v := reflect.ValueOf(x)
  fmt.Println("settability of v:", v.CanSet())
}
```

运行结果如下：

​                settability of v: false              

对于一个不具有“可写性”的 Value 类型变量，调用 Set 方法会报出错误。

首先我们要弄清楚什么是“可写性”，“可写性”有些类似于寻址能力，但是更严格，它是反射类型变量的一种属性，赋予该变量修改底层存储数据的能力。“可写性”最终是由一个反射对象是否存储了原始值而决定的。

示例代码如下：

```go
var x float64 = 3.4 
v := reflect.ValueOf(x)
```

这里我们传递给 reflect.ValueOf 函数的是变量 x 的一个拷贝，而非 x 本身，想象一下如果下面这行代码能够成功执行：

```go
v.SetFloat(7.1)
```

如果这行代码能够成功执行，它不会更新 x，虽然看起来变量 v 是根据 x 创建的，相反它会更新 x 存在于反射对象 v 内部的一个拷贝，而变量 x 本身完全不受影响。这会造成迷惑，并且没有任何意义，所以是不合法的。“可写性”就是为了避免这个问题而设计的。

这看起来很诡异，事实上并非如此，而且类似的情况很常见。考虑下面这行代码：

```
f(x)
```

代码中，我们把变量 x 的一个拷贝传递给函数，因此不期望它会改变 x 的值。如果期望函数 f 能够修改变量 x，我们必须传递 x 的地址（即指向 x 的指针）给函数 f，如下所示：

```
f(&x)
```

反射的工作机制与此相同，如果想通过反射修改变量 x，就要把想要修改的变量的指针传递给反射库。

首先，像通常一样初始化变量 x，然后创建一个指向它的反射对象，命名为 p：

```go
package main 
import (
  "fmt" 
  "reflect"
) 
func main() {
  var x float64 = 3.4
  p := reflect.ValueOf(&x)
  // Note: take the address of x.
  fmt.Println("type of p:", p.Type())
  fmt.Println("settability of p:", p.CanSet()) 
}
```

运行结果如下：

```go
type of p: *float64 settability of p: false
```

反射对象 p 是不可写的，但是我们也不像修改 p，事实上我们要修改的是 *p。为了得到 p 指向的数据，可以调用 Value 类型的 Elem 方法。Elem 方法能够对指针进行“解引用”，然后将结果存储到反射 Value 类型对象 v 中：

```go
package main
import (
  "fmt"
  "reflect"
)
func main() {
  var x float64 = 3.4
  p := reflect.ValueOf(&x) // Note: take the address of x.
  v := p.Elem()
  fmt.Println("settability of v:", v.CanSet())
}
```

运行结果如下：

```go
settability of v: true 
```

由于变量 v 代表 x， 因此我们可以使用 v.SetFloat 修改 x 的值：

```go
package main 
import (
  "fmt"
  "reflect"
)
func main() {
  var x float64 = 3.4
  p := reflect.ValueOf(&x) // Note: take the address of x.
  v := p.Elem()
  v.SetFloat(7.1) 
  fmt.Println(v.Interface()) 
  fmt.Println(x)
}
```

运行结果如下：

```
7.1 7.1
```

反射不太容易理解，reflect.Type 和 reflect.Value 会混淆正在执行的程序，但是它做的事情正是编程语言做的事情。只需要记住：只要反射对象要修改它们表示的对象，就必须获取它们表示的对象的地址。

**结构体**

我们一般使用反射修改结构体的字段，只要有结构体的指针，我们就可以修改它的字段。

下面是一个解析结构体变量 t 的例子，用结构体的地址创建反射变量，再修改它。然后我们对它的类型设置了 typeOfT，并用调用简单的方法迭代字段。

需要注意的是，我们从结构体的类型中提取了字段的名字，但每个字段本身是正常的 reflect.Value 对象。

```go
package main 
import (
  "fmt"
  "reflect" 
) 
func main() {
  type T struct {
    A int 
    B string 
  } 
  t := T{23, "skidoo"}
  s := reflect.ValueOf(&t).Elem()
  typeOfT := s.Type()
  for i := 0; i < s.NumField(); i++ {
    f := s.Field(i)
    fmt.Printf("%d: %s %s = %v\n", i, typeOfT.Field(i).Name, f.Type(), f.Interface())
  }
}
```

运行结果如下：

```go
0: A int = 23 1: B string = skidoo
```

T 字段名之所以大写，是因为结构体中只有可导出的字段是“可设置”的。

因为 s 包含了一个可设置的反射对象，我们可以修改结构体字段：

```go
package main 
import (
  "fmt"
  "reflect"
)
func main() {
  type T struct {
    A int
    B string
  }
  t := T{23, "skidoo"}
  s := reflect.ValueOf(&t).Elem()
  s.Field(0).SetInt(77)
  s.Field(1).SetString("Sunset Strip")
  fmt.Println("t is now", t)
}
```

运行结果如下：

```
t is now {77 Sunset Strip}
```

如果我们修改了程序让 s 由 t（而不是 &t）创建，程序就会在调用 SetInt 和 SetString 的地方失败，因为 t 的字段是不可设置的。

**总结**

反射规则可以总结为如下几条：

- 反射可以将“接口类型变量”转换为“反射类型对象”；
- 反射可以将“反射类型对象”转换为“接口类型变量”；
- 如果要修改“反射类型对象”，其值必须是“可写的”。