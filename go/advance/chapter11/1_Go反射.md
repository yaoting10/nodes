## 反射

反射（reflection）是在 [Java](http://c.biancheng.net/java/) 出现后迅速流行起来的一种概念，通过反射可以获取丰富的类型信息，并可以利用这些类型信息做非常灵活的工作。

大多数现代的高级语言都以各种形式支持反射功能，反射是把双刃剑，功能强大但代码可读性并不理想，若非必要并不推荐使用反射。

下面我们就来将介绍一下反射在Go语言中的具体体现以及反射的基本使用方法。

**反射的基本概念**

Go语言提供了一种机制在运行时更新和检查变量的值、调用变量的方法和变量支持的内在操作，但是在编译时并不知道这些变量的具体类型，这种机制被称为反射。反射也可以让我们将类型本身作为第一类的值类型处理。

反射是指在程序运行期对程序本身进行访问和修改的能力，程序在编译时变量被转换为内存地址，变量名不会被编译器写入到可执行部分，在运行程序时程序无法获取自身的信息。

支持反射的语言可以在程序编译期将变量的反射信息，如字段名称、类型信息、结构体信息等整合到可执行文件中，并给程序提供接口访问反射信息，这样就可以在程序运行期获取类型的反射信息，并且有能力修改它们。

C/[C++](http://c.biancheng.net/cplus/)语言没有支持反射功能，只能通过 typeid 提供非常弱化的程序运行时类型信息；Java、[C#](http://c.biancheng.net/csharp/) 等语言都支持完整的反射功能；Lua、[JavaScript](http://c.biancheng.net/js/) 类动态语言，由于其本身的语法特性就可以让代码在运行期访问程序自身的值和类型信息，因此不需要反射系统。

Go语言程序的反射系统无法获取到一个可执行文件空间中或者是一个包中的所有类型信息，需要配合使用标准库中对应的词法、语法解析器和抽象语法树（AST）对源码进行扫描后获得这些信息。

Go语言提供了 reflect 包来访问程序的反射信息。

**reflect 包**

Go语言中的反射是由 reflect 包提供支持的，它定义了两个重要的类型 Type 和 Value 任意接口值在反射中都可以理解为由 reflect.Type 和 reflect.Value 两部分组成，并且 reflect 包提供了 reflect.TypeOf 和 reflect.ValueOf 两个函数来获取任意对象的 Value 和 Type。

**反射的类型对象（reflect.Type）**

在Go语言程序中，使用 reflect.TypeOf() 函数可以获得任意值的类型对象（reflect.Type），程序通过类型对象可以访问任意值的类型信息，下面通过示例来理解获取类型对象的过程：

```go
package main
import (
  "fmt" 
  "reflect" 
) 
func main() {
  var a int  
  typeOfA := reflect.TypeOf(a)
  fmt.Println(typeOfA.Name(), typeOfA.Kind()) 
}
```

运行结果如下：

```
int int
```

代码说明如下：

- 第 9 行，定义一个 int 类型的变量。
- 第 10 行，通过 reflect.TypeOf() 取得变量 a 的类型对象 typeOfA，类型为 reflect.Type()。
- 第 11 行中，通过 typeOfA 类型对象的成员函数，可以分别获取到 typeOfA 变量的类型名为 int，种类（Kind）为 int。

**反射的类型（Type）与种类（Kind）**

在使用反射时，需要首先理解类型（Type）和种类（Kind）的区别。编程中，使用最多的是类型，但在反射中，当需要区分一个大品种的类型时，就会用到种类（Kind）。例如需要统一判断类型中的指针时，使用种类（Kind）信息就较为方便。

**1) 反射种类（Kind）的定义**

Go语言程序中的类型（Type）指的是系统原生数据类型，如 int、string、bool、float32 等类型，以及使用 type 关键字定义的类型，这些类型的名称就是其类型本身的名称。例如使用 type A struct{} 定义结构体时，A 就是 struct{} 的类型。

种类（Kind）指的是对象归属的品种，在 reflect 包中有如下定义：

```go
type Kind uint const (   
  Invalid Kind = iota // 非法类型   
  Bool                // 布尔型   
  Int         				// 有符号整型  
  Int8          			// 有符号8位整型   
  Int16      				  // 有符号16位整型  
  Int32    					  // 有符号32位整型  
  Int64								// 有符号64位整型 
  Uint       					// 无符号整型  
  Uint8     				  // 无符号8位整型
  Uint16      				// 无符号16位整型  
  Uint32            	// 无符号32位整型  
  Uint64   					  // 无符号64位整型  
  Uintptr    				  // 指针  
  Float32              // 单精度浮点数
  Float64              // 双精度浮点数 
  Complex64            // 64位复数类型
  Complex128           // 128位复数类型
  Array                // 数组
  Chan                 // 通道
  Func                 // 函数 
  Interface            // 接口
  Map                  // 映射 
  Ptr                  // 指针 
  Slice                // 切片 
  String               // 字符串
  Struct               // 结构体
  UnsafePointer        // 底层指针 
)
```

Map、Slice、Chan 属于引用类型，使用起来类似于指针，但是在种类常量定义中仍然属于独立的种类，不属于 Ptr。type A struct{} 定义的结构体属于 Struct 种类，*A 属于 Ptr。

**2) 从类型对象中获取类型名称和种类**

Go语言中的类型名称对应的反射获取方法是 reflect.Type 中的 Name() 方法，返回表示类型名称的字符串；类型归属的种类（Kind）使用的是 reflect.Type 中的 Kind() 方法，返回 reflect.Kind 类型的常量。

下面的代码中会对常量和结构体进行类型信息获取。

```go
package main 
import ( 
  "fmt"
  "reflect"
) 
// 定义一个Enum类型
type Enum int 
const (
  Zero Enum = 0 
) 
func main() {
  // 声明一个空结构体
  type cat struct {    } 
  // 获取结构体实例的反射类型对象
  typeOfCat := reflect.TypeOf(cat{})
  // 显示反射类型对象的名称和种类
  fmt.Println(typeOfCat.Name(), typeOfCat.Kind()) 
  // 获取Zero常量的反射类型对象
  typeOfA := reflect.TypeOf(Zero)
  // 显示反射类型对象的名称和种类
  fmt.Println(typeOfA.Name(), typeOfA.Kind())
}
```

运行结果如下：

```
cat struct Enum int
```

代码说明如下：

- 第 17 行，声明结构体类型 cat。
- 第 20 行，将 cat 实例化，并且使用 reflect.TypeOf() 获取被实例化后的 cat 的反射类型对象。
- 第 22 行，输出 cat 的类型名称和种类，类型名称就是 cat，而 cat 属于一种结构体种类，因此种类为 struct。
- 第 24 行，Zero 是一个 Enum 类型的常量。这个 Enum 类型在第 9 行声明，第 12 行声明了常量。如没有常量也不能创建实例，通过 reflect.TypeOf() 直接获取反射类型对象。
- 第 26 行，输出 Zero 对应的类型对象的类型名和种类。

**指针与指针指向的元素**

Go语言程序中对指针获取反射对象时，可以通过 reflect.Elem() 方法获取这个指针指向的元素类型，这个获取过程被称为取元素，等效于对指针类型变量做了一个*操作，代码如下：

```go
package main 
import (
  "fmt"  
  "reflect" 
) 
func main() {
  // 声明一个空结构体  
  type cat struct {    } 
  // 创建cat的实例
  ins := &cat{} 
  // 获取结构体实例的反射类型对象
  typeOfCat := reflect.TypeOf(ins) 
  // 显示反射类型对象的名称和种类 
  fmt.Printf("name:'%v' kind:'%v'\n", typeOfCat.Name(), typeOfCat.Kind()) 
  // 取类型的元素    
  typeOfCat = typeOfCat.Elem()
  // 显示反射类型对象的名称和种类  
  fmt.Printf("element name: '%v', element kind: '%v'\n", typeOfCat.Name(), typeOfCat.Kind()) 
}
```

运行结果如下：

```
name:'' kind:'ptr' element name: 'cat', element kind: 'struct'
```

代码说明如下：

- 第 13 行，创建了 cat 结构体的实例，ins 是一个 *cat 类型的指针变量。
- 第 15 行，对指针变量获取反射类型信息。
- 第 17 行，输出指针变量的类型名称和种类。Go语言的反射中对所有指针变量的种类都是 Ptr，但需要注意的是，指针变量的类型名称是空，不是 *cat。
- 第 19 行，取指针类型的元素类型，也就是 cat 类型。这个操作不可逆，不可以通过一个非指针类型获取它的指针类型。
- 第 21 行，输出指针变量指向元素的类型名称和种类，得到了 cat 的类型名称（cat）和种类（struct）。

**使用反射获取结构体的成员类型**

任意值通过 reflect.TypeOf() 获得反射对象信息后，如果它的类型是结构体，可以通过反射值对象 reflect.Type 的 NumField() 和 Field() 方法获得结构体成员的详细信息。

与成员获取相关的 reflect.Type 的方法如下表所示。

| 方法                                                        | 说明                                                         |
| ----------------------------------------------------------- | ------------------------------------------------------------ |
| Field(i int) StructField                                    | 根据索引返回索引对应的结构体字段的信息，当值不是结构体或索引超界时发生宕机 |
| NumField() int                                              | 返回结构体成员字段数量，当类型不是结构体或索引超界时发生宕机 |
| FieldByName(name string) (StructField, bool)                | 根据给定字符串返回字符串对应的结构体字段的信息，没有找到时 bool 返回 false，当类型不是结构体或索引超界时发生宕机 |
| FieldByIndex(index []int) StructField                       | 多层成员访问时，根据 []int 提供的每个结构体的字段索引，返回字段的信息，没有找到时返回零值。当类型不是结构体或索引超界时发生宕机 |
| FieldByNameFunc(match func(string) bool) (StructField,bool) | 根据匹配函数匹配需要的字段，当值不是结构体或索引超界时发生宕机 |

**1) 结构体字段类型**

reflect.Type 的 Field() 方法返回 StructField 结构，这个结构描述结构体的成员信息，通过这个信息可以获取成员与结构体的关系，如偏移、索引、是否为匿名字段、结构体标签（StructTag）等，而且还可以通过 StructField 的 Type 字段进一步获取结构体成员的类型信息。

StructField 的结构如下：

```go
type StructField struct {
  Name string          // 字段名
  PkgPath string       // 字段路径
  Type      Type       // 字段反射类型对象
  Tag       StructTag  // 字段的结构体标签
  Offset    uintptr    // 字段在结构体中的相对偏移
  Index     []int      // Type.FieldByIndex中的返回的索引值
  Anonymous bool       // 是否为匿名字段
}
```

字段说明如下：

- Name：为字段名称。
- PkgPath：字段在结构体中的路径。
- Type：字段本身的反射类型对象，类型为 reflect.Type，可以进一步获取字段的类型信息。
- Tag：结构体标签，为结构体字段标签的额外信息，可以单独提取。
- Index：FieldByIndex 中的索引顺序。
- Anonymous：表示该字段是否为匿名字段。

**2) 获取成员反射信息**

下面代码中，实例化一个结构体并遍历其结构体成员，再通过 reflect.Type 的 FieldByName() 方法查找结构体中指定名称的字段，直接获取其类型信息。

反射访问结构体成员类型及信息：

```go
package main 
import (
  "fmt"  
  "reflect"
) 
func main() {
  // 声明一个空结构体
  type cat struct { 
    Name string 			
    // 带有结构体tag的字段
    Type int `json:"type" id:"100"`
  }
  // 创建cat的实例
  ins := cat{Name: "mimi", Type: 1}
  // 获取结构体实例的反射类型对象
  typeOfCat := reflect.TypeOf(ins)
  // 遍历结构体所有成员
  for i := 0; i < typeOfCat.NumField(); i++ {
    // 获取每个成员的结构体字段类型
    fieldType := typeOfCat.Field(i) 
    // 输出成员名和tag
    fmt.Printf("name: %v  tag: '%v'\n", fieldType.Name, fieldType.Tag)
  }    
  // 通过字段名, 找到字段类型信息
  if catType, ok := typeOfCat.FieldByName("Type"); ok { 
    // 从tag中取出需要的tag
    fmt.Println(catType.Tag.Get("json"), catType.Tag.Get("id")) 
  } 
}
```

代码输出如下：

```
name: Name tag: ''
name: Type tag: 'json:"type" id:"100"'
type 100
```

代码说明如下：

- 第 10 行，声明了带有两个成员的 cat 结构体。
- 第 13 行，Type 是 cat 的一个成员，这个成员类型后面带有一个以 ` 开始和结尾的字符串。这个字符串在Go语言中被称为 Tag（标签）。一般用于给字段添加自定义信息，方便其他模块根据信息进行不同功能的处理。
- 第 16 行，创建 cat 实例，并对两个字段赋值。结构体标签属于类型信息，无须且不能赋值。
- 第 18 行，获取实例的反射类型对象。
- 第 20 行，使用 reflect.Type 类型的 NumField() 方法获得一个结构体类型共有多少个字段。如果类型不是结构体，将会触发宕机错误。
- 第 22 行，reflect.Type 中的 Field() 方法和 NumField 一般都是配对使用，用来实现结构体成员的遍历操作。
- 第 24 行，使用 reflect.Type 的 Field() 方法返回的结构不再是 reflect.Type 而是 StructField 结构体。
- 第 27 行，使用 reflect.Type 的 FieldByName() 根据字段名查找结构体字段信息，catType 表示返回的结构体字段信息，类型为 StructField，ok 表示是否找到结构体字段的信息。
- 第 29 行中，使用 StructField 中 Tag 的 Get() 方法，根据 Tag 中的名字进行信息获取。

**结构体标签（Struct Tag）**

通过 reflect.Type 获取结构体成员信息 reflect.StructField 结构中的 Tag 被称为结构体标签（StructTag）。结构体标签是对结构体字段的额外信息标签。

JSON、BSON 等格式进行序列化及对象关系映射（Object Relational Mapping，简称 ORM）系统都会用到结构体标签，这些系统使用标签设定字段在处理时应该具备的特殊属性和可能发生的行为。这些信息都是静态的，无须实例化结构体，可以通过反射获取到。

**1) 结构体标签的格式**

Tag 在结构体字段后方书写的格式如下：

```go
`key1:"value1" key2:"value2"`
```

结构体标签由一个或多个键值对组成；键与值使用冒号分隔，值用双引号括起来；键值对之间使用一个空格分隔。

**2) 从结构体标签中获取值**

StructTag 拥有一些方法，可以进行 Tag 信息的解析和提取，如下所示：

- func (tag StructTag) Get(key string) string：根据 Tag 中的键获取对应的值，例如`key1:"value1" key2:"value2"`的 Tag 中，可以传入“key1”获得“value1”。
- func (tag StructTag) Lookup(key string) (value string, ok bool)：根据 Tag 中的键，查询值是否存在。

**3) 结构体标签格式错误导致的问题**

编写 Tag 时，必须严格遵守键值对的规则。结构体标签的解析代码的容错能力很差，一旦格式写错，编译和运行时都不会提示任何错误，示例代码如下：

```go
package main 
import (
  "fmt"
  "reflect"
) 
func main() {
  type cat struct {
    Name string
    Type int `json: "type" id:"100"` //fixme 需要注意json:  后面不能加空格，否则无法识别
  } 
  typeOfCat := reflect.TypeOf(cat{})
  if catType, ok := typeOfCat.FieldByName("Type"); ok {
    fmt.Println(catType.Tag.Get("json"))
  }
}
```

运行上面的代码会输出一个空字符串，并不会输出期望的 type。

代码第 11 行中，在 json: 和 "type" 之间增加了一个空格，这种写法没有遵守结构体标签的规则，因此无法通过 Tag.Get 获取到正确的 json 对应的值。这个错误在开发中非常容易被疏忽，造成难以察觉的错误。所以将第 12 行代码修改为下面的样子，则可以正常打印。

```go
type cat struct { 
  Name string
  Type int `json:"type" id:"100"`
}
```

运行结果如下：

```
type
```

