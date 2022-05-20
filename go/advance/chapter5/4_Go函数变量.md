在Go语言中，函数也是一种类型，可以和其他类型一样保存在变量中，下面的代码定义了一个函数变量 f，并将一个函数名为 fire() 的函数赋给函数变量 f，这样调用函数变量 f 时，实际调用的就是 fire() 函数，代码如下：

```go
package main 
import (  
  "fmt"
) 
func fire() {
  fmt.Println("fire") 
} 
func main() {
  var f func()
  f = fire 
  f() 
}
```

代码输出结果：

```
fire
```

代码说明：

- 第 7 行，定义了一个 fire() 函数。
- 第 13 行，将变量 f 声明为 func() 类型，此时 f 就被俗称为“回调函数”，此时 f 的值为 nil。
- 第 15 行，将 fire() 函数作为值，赋给函数变量 f，此时 f 的值为 fire() 函数。
- 第 17 行，使用函数变量 f 进行函数调用，实际调用的是 fire() 函数。