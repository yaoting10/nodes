变量在一定程度上能满足函数及代码要求。如果编写一些复杂算法、结构和逻辑，就需要更复杂的类型来实现。这类复杂类型一般情况下具有各种形式的存储和处理数据的功能，将它们称为“容器（container）”。

在很多语言里，容器是以标准库的方式提供，你可以随时查看这些标准库的代码，了解如何创建，删除，维护内存。

本章将以实用为目的，详细介绍数组、切片、映射，以及列表的增加、删除、修改和遍历的使用方法。本章既可以作为教程，也可以作为字典，以方便开发者日常的查询和应用。

**其它语言中的容器**

- C语言没有提供容器封装，开发者需要自己根据性能需求进行封装，或者使用第三方提供的容器。
- C++ 语言的容器通过标准库提供，如 vector 对应数组，list 对应双链表，map 对应映射等。
- C# 语言通过 .NET 框架提供，如 List 对应数组，LinkedList 对应双链表，Dictionary 对应映射。
- Lua 语言的 table 实现了数组和映射的功能，Lua 语言默认没有双链表支持。

\1. [Go语言数组详解](http://c.biancheng.net/view/26.html)

\2. [Go语言多维数组简述](http://c.biancheng.net/view/4117.html)

\3. [Go语言切片详解](http://c.biancheng.net/view/27.html)

\4. [Go语言append()为切片添加元素](http://c.biancheng.net/view/28.html)

\5. [Go语言copy()：切片复制（切片拷贝）](http://c.biancheng.net/view/29.html)

\6. [Go语言从切片中删除元素](http://c.biancheng.net/view/30.html)

\7. [Go语言range关键字：循环迭代切片](http://c.biancheng.net/view/4118.html)

\8. [Go语言多维切片简述](http://c.biancheng.net/view/4119.html)

\9. [Go语言map（Go语言映射）](http://c.biancheng.net/view/31.html)

\10. [Go语言遍历map（访问map中的每一个键值对）](http://c.biancheng.net/view/32.html)

\11. [Go语言map元素的删除和清空](http://c.biancheng.net/view/33.html)

\12. [Go语言sync.Map（在并发环境中使用的map）](http://c.biancheng.net/view/34.html)

\13. [Go语言list（列表）](http://c.biancheng.net/view/35.html)

\14. [Go语言nil：空值/零值](http://c.biancheng.net/view/4776.html)

\15. [Go语言make和new关键字的区别及实现原理](http://c.biancheng.net/view/5722.html)