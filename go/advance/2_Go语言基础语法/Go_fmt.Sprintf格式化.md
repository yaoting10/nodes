# Go fmt.Sprintf 格式化字符串

[![Go 语言基础语法](https://www.runoob.com/images/up.gif) Go 语言基础语法](https://www.runoob.com/go/go-basic-syntax.html)

Go 可以使用 fmt.Sprintf 来格式化字符串，格式如下：

```
fmt.Sprintf(格式化样式, 参数列表…)
```

- **格式化样式：**字符串形式，格式化符号以 **%** 开头， %s 字符串格式，%d 十进制的整数格式。
- **参数列表：**多个参数以逗号分隔，个数必须与格式化样式中的个数一一对应，否则运行时会报错。

## 实例

**package** main


**import** (
  "fmt"
  "io"
  "os"
)


func main() {
  *// go 中格式化字符串并赋值给新串，使用 fmt.Sprintf*
  *// %s 表示字符串*
  **var** stockcode="000987"
  **var** enddate="2020-12-31"
  **var** url="Code=%s&endDate=%s"
  **var** target_url=fmt.Sprintf(url,stockcode,enddate)
  fmt.Println(target_url)

  *// 另外一个实例，%d 表示整型*
  **const** name, age = "Kim", 22
  s := fmt.Sprintf("%s is %d years old.**\n**", name, age)
  io.WriteString(os.Stdout, s) *// 简单起见，忽略一些错误*
}

输出结果为：

```
Code=000987&endDate=2020-12-31
Kim is 22 years old.
```

Go 字符串格式化符号:

| 格  式 | 描  述                                   |
| :----- | :--------------------------------------- |
| %v     | 按值的本来值输出                         |
| %+v    | 在 %v 基础上，对结构体字段名和值进行展开 |
| %#v    | 输出 Go 语言语法格式的值                 |
| %T     | 输出 Go 语言语法格式的类型和值           |
| %%     | 输出 % 本体                              |
| %b     | 整型以二进制方式显示                     |
| %o     | 整型以八进制方式显示                     |
| %d     | 整型以十进制方式显示                     |
| %x     | 整型以十六进制方式显示                   |
| %X     | 整型以十六进制、字母大写方式显示         |
| %U     | Unicode 字符                             |
| %f     | 浮点数                                   |
| %p     | 指针，十六进制方式显示                   |

## 实例

```go
**package** main

**import** (
  "fmt"
  "os"
)

**type** point struct {
  x, y int
}

func main() {

  p := point{1, 2}
  fmt.Printf("%v**\n**", p)

  fmt.Printf("%+v**\n**", p)

  fmt.Printf("%#v**\n**", p)

  fmt.Printf("%T**\n**", p)

  fmt.Printf("%t**\n**", **true**)

  fmt.Printf("%d**\n**", 123)

  fmt.Printf("%b**\n**", 14)

  fmt.Printf("%c**\n**", 33)

  fmt.Printf("%x**\n**", 456)

  fmt.Printf("%f**\n**", 78.9)

  fmt.Printf("%e**\n**", 123400000.0)
  fmt.Printf("%E**\n**", 123400000.0)

  fmt.Printf("%s**\n**", "**\"**string**\"**")

  fmt.Printf("%q**\n**", "**\"**string**\"**")

  fmt.Printf("%x**\n**", "hex this")

  fmt.Printf("%p**\n**", &p)

  fmt.Printf("|%6d|%6d|**\n**", 12, 345)

  fmt.Printf("|%6.2f|%6.2f|**\n**", 1.2, 3.45)

  fmt.Printf("|%-6.2f|%-6.2f|**\n**", 1.2, 3.45)

  fmt.Printf("|%6s|%6s|**\n**", "foo", "b")

  fmt.Printf("|%-6s|%-6s|**\n**", "foo", "b")

  s := fmt.Sprintf("a %s", "string")
  fmt.Println(s)

  fmt.Fprintf(os.Stderr, "an %s**\n**", "error")
}


```

输出结果为：

```go
{1 2}
{x:1 y:2}
main.point{x:1, y:2}
main.point
true
123
1110
!
1c8
78.900000
1.234000e+08
1.234000E+08
"string"
"\"string\""
6865782074686973
0xc0000b4010
|    12|   345|
|  1.20|  3.45|
|1.20  |3.45  |
|   foo|     b|
|foo   |b     |
a string
an error
```