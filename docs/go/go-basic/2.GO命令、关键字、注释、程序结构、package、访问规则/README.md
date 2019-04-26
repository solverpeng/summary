# GO命令

在命令行或终端输入go即可查看所有支持的命令

有如下常用命令

1. go run：直接运行程序
2. go build：测试编译，检查是否有编译错误
3. go fmt：格式化源码
4. go install：编译包文件并编译整个程序
5. go test：运行测试文件
6. go doc：查询文档，godoc -http=:6060，然后在本地访问：localhost:6060，启动本地文档

# GO内置关键字

break/default/func/interface/select/case/defer/go/map/struct/chan/else/goto

/package/switch/const/fallthrought/if/range/type/continue/for/import/return/var

# GO注释

单行注释：//

多行注释：/* */

# GO程序代码结构

```go
//当前程序的包名
package main


// 导入其他包
import "fmt"


// 常量的定义
const PI = 3.14


// 全局变量的定义
var name = "lily"


// 一般类型的生命
type newType int32


// 结构的声明
type gostruct struct {
}


// 接口的声明
type gointerface interface {
}


// main函数
func main() {
   fmt.Println(name)
}
```

1. GO程序是通过package来组织的，只有package名称为main的包可以包含main函数，一个可执行的程序有且仅有一个main包
2. 通过import来导入其他非main包
3. 通过const关键字来进行常量的定义
4. 通过在函数体外部使用 var 关键字来进行全局变量的声明和赋值
5. 通过 type 关键字来进行结构（struct）和接口（interface）声明
6. 通过 func 关键字来进行函数的声明

# package

## 导入方式

```go
import "fmt"
import "io"
import "os"
import "time"
import "strings"

// 也可以使用组的方式替代
import (
    "fmt"
    "io"
    "os"
    "time"
    "strings"
)
```

1. 导入包之后，就可以使用 `<packageName>.<funcName>`进行调用
2. 如果导入包未调用其中函数或类型，就会报出编译错误

## 别名

当使用第三方包时，包名可能非常接近或者相同，因此就可以使用别名来进行区别或调用

```go
//别名的定义
import std "fmt"

// 别名的使用
func main() {
   std.Println()
}
```

还有一种方式是省略调用，将别名定义为【.】，在调用的时候可以省掉函数前边包名和【.】

```go
// 别名定义
import . "fmt"

// 使用省略调用
func main() {
   Println()
}
```

**易混淆，不建议使用省略调用**

# 可见性规则

GO语言中，使用大小写来决定个该常量、变量、类型、接口、结构或者函数是否可以被外部包所调用，根据约定函数名小写表示 private，大写表示 public。



# 组

对声明的多个常量、多个全局变量、多个一般类型（非结构、非接口）进行简写

```go
const (
    PI = 3.14
    const1 = "1"
    const2 = 1
)

var (
    name = 1
    name1 = "1"
    name2 = false
)

type (
    newType int
    type1 float32
    type2 string
    type3 byte
)
```

