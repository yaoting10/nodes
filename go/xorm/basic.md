xorm是一个简单而强大的ORM库。

　　安装

```go
go get -u github.com/go-xorm/xorm
```

　　驱动支持

```go
Mysql: github.com/go-sql-driver/mysql
MyMysql: github.com/ziutek/mymysql
Postgres: github.com/lib/pq
Tidb: github.com/pingcap/tidb
SQLite: github.com/mattn/go-sqlite3
MsSql: github.com/denisenkom/go-mssqldb
MsSql: github.com/lunny/godbc
Oracle: github.com/mattn/go-oci8 (试验性支持)
```

### 创建orm引擎

　　一个xorm可同时存在orm引擎，一个Orm引擎称为Engine，一个Engine一般只对应一个数据库。Engine通过调用xorm.NewEngine生成，如：

```go
package main
 
import (
    "github.com/go-xorm/xorm"
    _ "github.com/go-sql-driver/mysql"
)

func main()  {
    engine, err := xorm.NewEngine("mysql", "root:passwd@tcp(127.0.0.1:3306)/dabase_name?timeout=3s&parseTime=true&loc=Local&charset=utf8")
    if err !=nil{
        return
    }
    engine.Ping()  // 可以判断是否能连接
     
    defer engine.Close()  // 退出后关闭
}
```

### 定义表结构体

　　表名映射一般有三种方式，且都按优先级高低

- 表名的优先级顺序如下：
  - engine.Table() 指定的临时表名优先级最高
  - TableName() string 其次
  - Mapper 自动映射的表名优先级最后
- 字段名的优先级顺序如下：
  - 结构体tag指定的字段名优先级较高
  - Mapper 自动映射的表名优先级较低　　

　　Column属性定义，首先定义一个结构体如

```go
type User struct {
    Id      int64
    Name string  `xorm:"varchar(25) notnull unique 'usr_name'"`
    Balance float64
    Version int `xorm:"version"` // 乐观锁
}
```

　　具体tag详情如下。且字段名根据不同数据库区分大小写

| Tag                                              | 说明                                                         |
| ------------------------------------------------ | ------------------------------------------------------------ |
| name                                             | 当前field对应的字段的名称，可选，如不写，则自动根据field名字和转换规则命名，如与其它关键字冲突，请使用单引号括起来。 |
| pk                                               | 是否是Primary Key，如果在一个struct中有多个字段都使用了此标记，则这多个字段构成了复合主键，单主键当前支持int32,int,int64,uint32,uint,uint64,string这7种Go的数据类型，复合主键支持这7种Go的数据类型的组合。 |
| 当前支持30多种字段类型，详情参见本文最后一个表格 | 字段类型                                                     |
| autoincr                                         | 是否是自增                                                   |
| [not ]null 或 notnull                            | 是否可以为空                                                 |
| unique或unique(uniquename)                       | 是否是唯一，如不加括号则该字段不允许重复；如加上括号，则括号中为联合唯一索引的名字，此时如果有另外一个或多个字段和本unique的uniquename相同，则这些uniquename相同的字段组成联合唯一索引 |
| index或index(indexname)                          | 是否是索引，如不加括号则该字段自身为索引，如加上括号，则括号中为联合索引的名字，此时如果有另外一个或多个字段和本index的indexname相同，则这些indexname相同的字段组成联合索引 |
| extends                                          | 应用于一个匿名成员结构体或者非匿名成员结构体之上，表示此结构体的所有成员也映射到数据库中，extends可加载无限级 |
| -                                                | 这个Field将不进行字段映射                                    |
| ->                                               | 这个Field将只写入到数据库而不从数据库读取                    |
| <-                                               | 这个Field将只从数据库读取，而不写入到数据库                  |
| created                                          | 这个Field将在Insert时自动赋值为当前时间                      |
| updated                                          | 这个Field将在Insert或Update时自动赋值为当前时间              |
| deleted                                          | 这个Field将在Delete时设置为当前时间，并且当前记录不删除      |
| version                                          | 这个Field将会在insert时默认为1，每次更新自动加1              |
| default 0或default(0)                            | 设置默认值，紧跟的内容如果是Varchar等需要加上单引号          |
| json                                             | 表示内容将先转成Json格式，然后存储到数据库中，数据库中的字段类型可以为Text或者二进制 |

　　需要注意的几点

- 如果field名称为ID，且类型为int64，并且 没有定义tag，则会被xorm视为主键，且拥有自增属性。如果要用其它名字为主键，需对应tag加上 xorm:"pk"

- 

- string类型默认为varchar(255)

- 支持type MyString string 等自定义的field。支持Slice, Map,等field成员。这些成员默认存储为Text类型。并且拥有Json格式来序列化和反序列化。

- 实现Conversion接口的类型或者结构体，将根据接口的转换方式在类型和数据库记录之间进行相互转换。

  ```go
  type Conversion interface {
      FromDB([]byte) error
      ToDB() ([]byte, error)
  }
  ```

  　

### 表结构常用操作

　　获取数据库信息

- DBMetas(): xorm支持获取表结构信息。通过调用engine.DBMetas()获取表，字段，索引信息
- TableInfo(): 根据传入的结构体指针及对应的Tag，提取出模型对应的表结构信息。

　　表操作

- CreateTables(): 创建表engine.CreateTables() 参数为一个或多个空的对应Struct的指针。可用方法有Charset()和StoreEngine()。
- IsTableEmpty(): 判断是否为空。参数和CreateTables()相同。
- IsTableExist():判断是否存在
- DropTables(): 删除表engine.DropTables().参数为一个或多个空的对应Struct的指针或者表的名字。如果为string传入，则只删除对应的表，如果传入的为Struct，则删除表的同时还会删除对应的索引。

　　创建索引和唯一索引

- CreateIndexes: 根据struct中的Tag来创建索引
- CreateUniques: 根据struct中的tag来创建唯一索引

　　同步数据库结构到 mysql中

- Sync

  - 自动检测和创建表，这个检测是根据表的名字

  - 自动检测和新增表中的字段，这个检测是根据字段名

  - 自动检测和创建索引和唯一索引，这个检测是根据索引的一个或多个字段名，而不根据索引名称

    ```go
    err := engine.Sync(new(User), new(Group))
    // 其中User ，Group为要创建的两个表对应的struct
    ```

Sync2, 对Sync进行改进，推荐使用Sync2

- - 自动检测和创建表，这个检测是根据表的名字
  - 自动检测和新增表中的字段，这个检测是根据字段名，同时对表中多余的字段给出警告信息
  - 自动检测，创建和删除索引和唯一索引，这个检测是根据索引的一个或多个字段名，而不根据索引名称。因此这里需要注意，如果在一个有大量数据的表中引入新的索引，数据库可能需要一定的时间来建立索引。
  - 自动转换varchar字段类型到text字段类型，自动警告其它字段类型在模型和数据库之间不一致的情况。
  - 自动警告字段的默认值，是否为空信息在模型和数据库之间不匹配的情况

 　以上信息需要将engine.ShowWarn设置为true才会显示。调用方法

```go
err := engine.Sync2(new(User), new(Group))
```

　　导入导出SQL脚本

　　dump

```go
engine.DumpAll(w io.Writer)
或
engine.DumpAllFile(fpath string)
```

　　Import

```go
engine.Import(r io.Reader)
或者
engine.ImportFile(fpath string)
```

### 插入数据操作（表结构参照上面User表)

　　ORM插入一条数据

```go
user := new(User)
user.Name = "myname"
affected, err := engine.Insert(user)
// INSERT INTO user (name) values (?)
```

　　　插入同一个表的多条数据

```go
users := make([]User, 1)
users[0].Name = "name0"
users[0].ID = "0"
...
affected, err := engine.Insert(&users)
```

　　指针Slice插入多条记录

```go
users := make([]*User, 1)
users[0] = new(User)
users[0].Name = "name0"
users[0].ID = "0"
...
affected, err := engine.Insert(&users)
```

　　不同表的一条记录

```go
user := new(User)
user.Name = "myname"
question := new(Question)
question.Content = "whywhywhwy?"
affected, err := engine.Insert(user, question)
```

　　不同表的多条记录

```go
users := make([]User, 1)
users[0].Name = "name0"
...
questions := make([]Question, 1)
questions[0].Content = "whywhywhwy?"
affected, err := engine.Insert(&users, &questions)
```

　　**使用SQL插入数据**

```go
sql ="insert into config(key,value) values (?, ?)"
res, err := engine.Exec(sql, "OSCHINA", "OSCHINA")
或者
sql_2 := "insert into config(key,value) values (?, ?)"
affected, err := engine.Sql(sql_4, "OSCHINA", "OSCHINA").Execute()
或者
//SqlMap中key为 "sql_i_1" 配置的Sql语句为：insert into config(key,value) values (?, ?)
sql_i_1 := "sql_i_1"
affected, err := engine.SqlMapClient(sql_i_1, "config_1", "1").Execute()
或者
sql_i_3 := "insert.example.stpl"
paramMap_i_t := map[string]interface{}{"key": "config_3", "value": "3"}
affected, err := engine.SqlTemplateClient(sql_i_3, &paramMap_i_t).Execute()
```

### 查询操作

　　**ORM常用查询**

　　注意下面出现的 & +表结构体

　　设置别名

```go
engine.Alias("o").Where("o.name = ?", name).Get(&order)
```

　　条件查找

```go
engine.Where(...).And(...).Get(&order)
```

　　某个字段排序

```go
// 正序
engine.Asc("id").Find(&orders)
//倒序
engine.Asc("id").Desc("time").Find(&orders)
```

　　主键查找

```go
var user User
engine.Id(1).Get(&user)
// SELECT * FROM user Where id = 1
 
// 或者复合主键
engine.Id(core.PK{1, "name"}).Get(&user)
// SELECT * FROM user Where id =1 AND name= 'name'
```

　　select， in, cols

```go
engine.Select("a.*, (select name from b limit 1) as name").Find(&beans)
engine.Select("a.*, (select name from b limit 1) as name").Get(&bean)
 
 
// in
engine.In("cloumn", 1, 2, 3).Find()
engine.In("column", []int{1, 2, 3}).Find()
 
// cols
engine.Cols("age", "name").Get(&usr)
// SELECT age, name FROM user limit 1
engine.Cols("age", "name").Find(&users)
// SELECT age, name FROM user
engine.Cols("age", "name").Update(&user)
// UPDATE user SET age=? AND name=?
```

　　GET方法，查询到的数据会赋给结构体

```go
has, err := engine.Get(&user)
// SELECT * FROM user LIMIT 1
has, err := engine.Where("name = ?", name).Desc("id").Get(&user)
// SELECT * FROM user WHERE name = ? ORDER BY id DESC LIMIT 1
var name string
has, err := engine.Where("id = ?", id).Cols("name").Get(&name)
// SELECT name FROM user WHERE id = ?
var id int64
has, err := engine.Where("name = ?", name).Cols("id").Get(&id)
// SELECT id FROM user WHERE name = ?
var valuesMap = make(map[string]string)
has, err := engine.Where("id = ?", id).Get(&valuesMap)
// SELECT * FROM user WHERE id = ?
var valuesSlice = make([]interface{}, len(cols))
has, err := engine.Where("id = ?", id).Cols(cols...).Get(&valuesSlice)
// SELECT col1, col2, col3 FROM user WHERE id = ?
或者
user := new(User)
has, err := engine.Where("name=?", "xlw").Get(user)
```

　子查询

```go
var student []Student
err = db.Table("student").Select("id ,name").Where("id in (?)", db.Table("studentinfo").Select("id").Where("status = ?", 2).QueryExpr()).Find(&student)
//SELECT id ,name FROM `student` WHERE (id in (SELECT id FROM `studentinfo` WHERE (status = 2)))
```

　　SQL操作返回格式json or xml

```go
var users []User
results,err := engine.Where("id=?", 6).Search(&users).Xml() //返回查询结果的xml字符串
results,err := engine.Where("id=?", 6).Search(&users).Json() //返回查询结果的json字符串
```

### 更新操作

　　update方法

```go
user := new(User)
user.Name = "myname"
affected, err := engine.Id(id).Update(user)
```

　　指定更新值

```go
affected, err := engine.Id(id).Cols("age").Update(&user)
// 或
affected, err := engine.Table(new(User)).Id(id).Update(map[string]interface{}{"age":0})
```

　　更新时间，可以在字段名后添加 update如下

```go
type User struct {
    Id int64
    Name string
    UpdatedAt time.Time `xorm:"updated"`
}
```

### 删除操作

　　delete方法

```go
user := new(User)
affected, err := engine.Id(id).Delete(user)
 
//Delete的返回值第一个参数为删除的记录数，第二个参数为错误。
```

　　xorm还提供了软删除，如下设置

```go
type User struct {
    Id int64
    Name string
    DeletedAt time.Time `xorm:"deleted"`
}
```

　　如果设置软删除，那么永久删除或者获取使用Unscoped

```go
var user User
engine.Id(1).Unscoped().Get(&user)
// 此时将可以获得记录
engine.Id(1).Unscoped().Delete(&user)
// 此时将可以真正的删除记录
```

### 创建数据库组

　　xorm提供了可以连接多个数据库。如下

```go
package main
 
import (
    "github.com/go-xorm/xorm"
    _ "github.com/go-sql-driver/mysql"
 
)
 
func main()  {
    conns := []string{
        "mysql", "root:passwd@tcp(127.0.0.1:3306)/dabase_name?timeout=3s&parseTime=true&loc=Local&charset=utf8",
        "mysql", "root:passwd@tcp(127.0.0.1:3306)/dabase_name?timeout=3s&parseTime=true&loc=Local&charset=utf8",
        "mysql", "root:passwd@tcp(127.0.0.1:3306)/dabase_name?timeout=3s&parseTime=true&loc=Local&charset=utf8",
    }
    engine, err := xorm.NewEngineGroup("mysql", conns)
    if err !=nil{
        return
    }
    engine.Ping()  // 可以判断是否能连接
 
    defer engine.Close()  // 退出后关闭
}
```

### 连接池　　

　　engine内部支持连接池接口和对应的函数。

- 如果需要设置连接池的空闲数大小，可以使用engine.SetMaxIdleConns()来实现。
- 如果需要设置最大打开连接数，则可以使用engine.SetMaxOpenConns()来实现。