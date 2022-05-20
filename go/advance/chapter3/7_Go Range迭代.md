通过前面的学习我们了解到切片其实就是多个相同类型元素的连续集合，既然切片是一个集合，那么我们就可以迭代其中的元素，Go语言有个特殊的关键字 range，它可以配合关键字 for 来迭代切片里的每一个元素，如下所示：

```go
//创建一个整型切片，并赋值 
slice := []int{10, 20, 30, 40} 
// 迭代每一个元素，并显示其值 
for index, value := range slice {
  fmt.Printf("Index: %d Value: %d\n", index, value)
}
```

第 4 行中的 index 和 value 分别用来接收 range 关键字返回的切片中每个元素的索引和值，这里的 index 和 value 不是固定的，读者也可以定义成其它的名字。

关于 for 的详细使用我们将在下一章《[Go语言流程控制](http://c.biancheng.net/golang/flow_control/)》中为大家详细介绍。

上面代码的输出结果为：

```go
Index: 0 Value: 10 Index: 1 Value: 20 Index: 2 Value: 30 Index: 3 Value: 40
```

当迭代切片时，关键字 range 会返回两个值，第一个值是当前迭代到的索引位置，第二个值是该位置对应元素值的一份副本，如下图所示。



图：使用 range 迭代切片会创建每个元素的副本

需要强调的是，range 返回的是每个元素的副本，而不是直接返回对该元素的引用，如下所示。

【示例 1】range 提供了每个元素的副本

```go
// 创建一个整型切片，并赋值 
slice := []int{10, 20, 30, 40}
// 迭代每个元素，并显示值和地址 
for index, value := range slice {
  fmt.Printf("Value: %d Value-Addr: %X ElemAddr: %X\n", value, &value, &slice[index])
}
```

输出结果为：

```
Value: 10 Value-Addr: 10500168 ElemAddr: 1052E100 
Value: 20 Value-Addr: 10500168 ElemAddr: 1052E104 
Value: 30 Value-Addr: 10500168 ElemAddr: 1052E108 
Value: 40 Value-Addr: 10500168 ElemAddr: 1052E10C
```

因为迭代返回的变量是一个在迭代过程中根据切片依次赋值的新变量，所以 value 的地址总是相同的，要想获取每个元素的地址，需要使用切片变量和索引值（例如上面代码中的 &slice[index]）。

如果不需要索引值，也可以使用下划线_来忽略这个值，代码如下所示。

【示例 2】使用空白标识符（下划线）来忽略索引值

```go
// 创建一个整型切片，并赋值 
slice := []int{10, 20, 30, 40}
// 迭代每个元素，并显示其值 
for _, value := range slice {
  fmt.Printf("Value: %d\n", value)
}
```

输出结果为：

```go
Value: 10 Value: 20 Value: 30 Value: 40
```

关键字 range 总是会从切片头部开始迭代。如果想对迭代做更多的控制，则可以使用传统的 for 循环，代码如下所示。

【示例 3】使用传统的 for 循环对切片进行迭代

```go
// 创建一个整型切片，并赋值
slice := []int{10, 20, 30, 40}
// 从第三个元素开始迭代每个元素
for index := 2; index < len(slice); index++ {
  fmt.Printf("Index: %d Value: %d\n", index, slice[index])
}
```

输出结果为：

```go
Index: 2 Value: 30 Index: 3 Value: 40
```

在前面几节的学习中我们了解了两个特殊的内置函数 len() 和 cap()，可以用于处理数组、切片和通道，对于切片，函数 len() 可以返回切片的长度，函数 cap() 可以返回切片的容量，在上面的示例中，使用到了函数 len() 来控制循环迭代的次数。

当然，range 关键字不仅仅可以用来遍历切片，它还可以用来遍历数组、字符串、map 或者通道等，这些我们将在后面的学习中详细介绍。