# Go语言笔记

## 安装与验证

Ubuntu终端下执行安装命令

```shell
sudo apt install golang-go
```

验证安装是否正常

``` shell
go version
go env
```

设置代理

```go
go env -w GOPROXY=https://goproxy.cn,direct
```



## go命令环境变量

`GOROOT`：Go语言环境的根目录

`GOPATH`：Go项目目录

## Go项目源文件结构和分类

项目目录结构由`bin`，`pkg`， `src`三个目录构成。其中`bin`和`pkg`由`go`命令自动生成。src为源代码所在目录。

src目录下每个子目录代表一个包。每个子目录下的所有源文件必须属于同一个包。包名与子目录名字可以不一样。程序需要有一个main包（package main），并且main包中要有一个main函数。

包含main函数的文件叫做命令源文件；其他则为库源文件。

> 提示
>
> main包所在目录的名字应该与将要生成的程序名字一致。

## Go模块

#### 创建模块

go语言通过`go mod` 命令和go.mod文件管理模块

```shell
mkdir mymod
cd mymod
go mod mymodule # create a module named mymodule
```

#### 调用模块

```shell
mkdir myapp
cd myapp
go mod myapp
vim main.go # In main.go, import mymodule
```

对于本地未发布的模块，需要修改go.mod文件。添加以下内容

```go
replace mymodule => PATH_TO_MYMODULE_ON_LOCAL_FILESYSTEM
```



## 编译和运行

### go命令的子命令

### `go run`

编译并运行程度，但是生成的文件都是临时的。并且只能用于命令源文件。只能用于命令源文件。

`go run -n` 可以看到具体的执行过程。主要包括以下步骤：

1. 在$WORK下面创建一个临时目录
2. 生成importcfg配置文件
3. 使用compile工具编译代码
4. 生成链接配置文件importcfg.link
5. 链接生成可执行文件
6. 运行可执行文件

### `go build`

如果是main包，则在当前目录生成可执行文件（如果接着go install 命令，则会将次文件移动到bin下）。如果为非main包，则只做语法检查，步生成任何文件。

### `go install`

编译并安装程序到bin目录下，库则安装到pkg目录下。

---

## 数据类型

### 基本数据类型和默认初始化值：

- bool // false
- int, int8, int16, int32, int64 // 0
- uint, uint8, uint16, uint32, uint64, uintptr // 0 or nil
- byte // 0
- rune // 0?
- float32, float64 // 0.0
- complex64, complex128 // 0?
- map[type1]type2 // nil
- []type // nil
- func(type1, type2) (type1, type2) // nil
- var a INTERFACE // nil

### 复合数据类型

#### 字符串

使用双引号或\`

## 变量

### 变量声明

```go
var var_name type
var v1, v2 type

// batch mode
var (
	v1 type1
    v2 type2
    v3 struct {
        member type
    }
)
```

### 变量初始化

```go
var var_name type = initial_value
var v1, v2 type = value1, value2
var var_name = initial_value // compiler infer

// declare and initialize
// 1, must not provide type
// 2, only valid in function body, not for global variables
// 3, must provide initial value
var_name := initial_value
v1, v2 := intial_value // at least one variable is newly declared
```

### 局部变量和全局变量

函数体内的变量，形式参数，返回值都是局部变量。函数体外的变量为全局变量，并且**必须用var声明**。如果包以外的其它包引用全局变量，则变量名的第一个字母要**大写**。

## 常量

常量名字建议全部大写。常量定义的两种形式：

```go
const NAME type = value
const NAME = value
const C1, C2, C3 = v1, v2, v3
const (
	A = value1
    B // same value with A. like enum type in C
    C = value2
)

// iota
const (
	A = iota // in each const group, iota starts from 0
    B // B is iota, so B is 1
    C = 100 // iota still increases by 1, ie, 2
    D // the same value of C. iota is 3
    E = iota // 4
    F // 5
)
```

## 运算符

| 种类       | 符号                    |
| ---------- | ----------------------- |
| 数学运算符 | \+ - * / %              |
| 逻辑运算符 | && \|\| ！ << >>        |
| 关系运算符 | \> >= == != < <=        |
| 位运算符   | & \| ^ &^ (一元^为取反) |

## 类型转换

语法为：`v1 = type(v2)`

```go
var a int
var b int16 = 100

a = int(b)
```

## 关键字

### import

import 引入包

```go
// import a single package
import "fmt"
import "net/http"

// import multiple packages
import (
	"os"
    "strings"
)

// import all functions in a package
// directly into this scope
import . "fmt"

// execute the init function in a package
import _ "fmt"

// short name
import shortname "long/path/package"
```

### if-statement

```go
if bool_expr {
    // statements
} else if bool_expr {
    // statements
} else {
    // statements
}

if statement; bool_expr { //variable in statement is local to if
    //
}
```

### switch-statement

默认每个case后break。也可以在在case中写break。case最后写fallthrough会执行case后紧接着的case。

```go
switch variable {
case v1:
   //
case v2:
    //
default:
    //
}

switch {
case bool_expr1:
    //
case bool_expr2:
    //
default:
    //
}

// multiple values in one case
switch variable {
case v1, v2, v3:
    //
default:
    //
}
// variable is local to this switch
switch init_statement; variable {
case v1:
    //
default:
    //
}
```

### for-statement

golang 有和C语言类似的break和continue

```go
for initial; bool_expr: statement {
    // body
}

for bool_expr {
    // body
}

for {
    // body
}

```

### goto

```go
	goto label_name
	// statements
label_name:
	// statements
```

### defer

- 延迟函数的执行；函数调用会在外层函数return之前执行
- 有多个defer时，先defer的后执行
- 参数在defer语句处就已经传入了函数内部
- 可用于关闭文件之类的清理工作

```go
defer func_name()
```

### type

```go
// myint is differenct from int
type myint int

// aliasing
type myint = int

type myfunc func(T) (T)
```

## 数组

- 数组是一种数值类型，赋值会复制所有数据而不是复制引用
- 可以直接使用Println打印
- 可以直接比较是否相等，不能比较大小

```go
// define an array
var id [SIZE] type

// define and initialize
// n < SIZE
var array = [SIZE]type{v1, v2, ... vn}
var array = [SIZE]type{index0: value, ... indexn: value}

// the size is determined by the number of intial values
var array := [...]type{v, v, v}
var array := [...]type{index0:v, indexn:v}

// length and capacity
len(array)
cap(array)
```

遍历数组

```go
for i := 0; i < len(array); i++ {
    array[i]
}

// for range
for i, v := range array {
    //
}
```

赋值方式

```go
// copy value
// array1 and array2 have their own data copies.
array1 = array2
```

数组比较只有相等和不相等

```go
// equal if length and value are both equal
// a1 and a2 must have the same type (length and type)
array1 == array2
a1 != a2

```

多维数组

```go
var a [SIZE][SIZE]type
var a = [][]type{
    {v1, v2 ...},
    {...},
    ...
}
```

## 切片

- 容量会自动扩展
- 扩容时，容量是翻倍的
- 切片是引用类型的数据
- 切片是指针：`fmt.Printf("%p", slice)`，不需要`&`
- 切片扩容时会改变地质，这时`append`函数返回新地址
- 使用`slice...`展开切片；数组也可以：`array[:]...`
- 如果切片从数组创建，在不扩容的前提下，修改切片，也会修改数组的内容

```go
// array
var array [4]int

// slice, no size
var s []int

var s := make([]int, length, capacity)

// append new value
s = append(s, v1, v2, ...)

// append values of s2 to s1
// ...expands a slice to elements
s1 = append(s1, s2...)

// iteration
for i := 0; i < len(s); i++ {
    //
}

for i, v := range s {
    //
}

// create slice from array
var s = array[start:end]
var s = array[:end]
var s = array[start:]
var s = array[:]
```

深拷贝和浅拷贝

- copy不会导致slice扩容，拷贝的元素个数是len(dest)和len(src)中较小的那个数值

```go
copy(dest, source)
```

## map

- 引用类型

创建map

```go
var m = map[key_type]value_type // nil map, can not store value
var m = make(map[key_type]value_type) // a real map object is created

var m = map[k]v{k1:v1, k2:v2, ...}
```

操作map

```go
m[key] = value
a = m[k]
a, haskey := m[k] // is key does not exist, haskey is false
delete(m, key)
```

遍历map

```go
for k, v := range m {
    //
}
```

## 字符串

```go
var s = "abc"

// get a byte
s[sub]
```

相关包：strings, strconv

## 函数

```go
// If there is only return type (and only one), parenthesis is optional.
// Otherwise, parenthesis is required.
func func_name(param1 type, param2m type) (ret1 type, ret2 type) {
    // body
    return ret1, ret2
}

// variable arguments
func name(param1 type, param2 ... type) (ret1 type, ret2 type) {
    // body
}

func(param type) (ret type) {
    // body
}()

f:= func(param type) type {
    // body
}
r = f(arg)
```

Closure

```go
func count(i int) func() int {
    return func () {
        i++
        return i
    }
}

f = count(5)
f()
f()
```

## 指针

```go
p *[2]int // a pointer to an array of two int
p [2]*int // an array of two pointers to int
var a = 1
p := &a // a pointer to int
*p // pointer dereference

p *[2]int
p[1] // the second element of an array

a [3]*int
*a[1] // value pointed by the second element of a pointer array

```

## 结构体

```go
type Name struct {
    member1 type
    member2type
    ...
}
// 1
var n Name
n.member1 = value
n.member2 = value
...
// 2
n := Name{}
n.member1 = value
...
// 3
n := Name{member1: value, member2: value, ...}
// 4
n := Name{value, value, ...}
// a pointer
n := &Name{...}
// deep copy
n1 :=n2

var n = *Name{value, value}
// both are ok
(*n).member
n.member

// new
var n = new(Name) // a pointer

var n = struct {
    // members
}{
    // initialize
}
```

anonymous filed

```go
type Name struct {
    Type // anonymous
    f1 type
    f2 type
}
// members of Type can be accessed directly by instance of Name

// 1
n := Name{Type{v, v}, v, v}
// 2
n.m1 = v // directl access
n.m2 = v // direct access
n.m3 = v
n.m4 = v
// 3
var n = Name{Type: Type{m1, m2}, m3: v, m4: v}
```

## 方法

- 结构体可以直接访问匿名字段的方法。
- 也可以重写匿名字段方法

```go
// pass value
func (this Type) func_name(param type) {
    // body
}
// pass pointer
func (this *Type) func_name(param type) {
    // body
}
```

## 接口

```go
type Name interface {
    funcname(param type) ret type
    ...
}

type T1 struct {
    //
}

type T2 struct {
    //
}

func (this *T1)funcname(param type) ret type {
    //
}

func (this *T2)funcname(param type) ret type {
    //
}
```

可以创建接口的实例，并把实现赋值给接口实例。接口实例只能访问接口方法。

### 关于value receiver 和pointer receiver

- value receiver可以接受value或者pointer
- pointer receiver 只能接受pointer

因此，当一个方法会改变对象时，应该定义成pointer receiver；当方法不会改变对象时，应该定义成value receiver。

### 空接口

```go
type E interface {
    // nothing
}

var a = map[string]interface{} // value can be any type
```

### 接口嵌套

```go
type A interface {
    a()
}

type B interface {
    A
    b()
}
```

### 接口断言

```go
// i is type of an interface
ins, ok := i.(T)

// here type is the keyword type
switch i.(type) {
    case T1:
    	//
    case T2:
    	//
}
```

## error

- error也是一种数据类型，是一种接口类型
- 通过接口断言获取详细错误信息

```go
// prototype
type error interface {
    Error() string
}
```

创建错误对象

```go
import "errors"
// return apointer to errors.errorString
e := errors.New("Error message")

import "fmt"
// formated string
e := fmt.Errorf("format", v, v)
```

## panic & recover

- 在一个函数内，panic之后的defer语句是不会执行的
- 所有已经defer的函数执行完成后，panic才会传到上一层

```go
panic(value)

// capture panic
// sometimes call recover() in a deferred function
v := recover()
```

## 包

- 使用import引入包
- 使用package定义包名
- 同一个目录下的包名要一致

### init函数

- 包可以定义自己的init函数
- init函数不能有参数和返回值
- init函数会在main函数之前执行，进行一些包的初始化操作
- 一个go文件中可以有多个init函数，按照定义顺序执行
- 同一个包的不同go文件中可以有各自的init函数，按照文件名字字母顺序排序，依次执行
- 不同包中的init函数按照import顺序执行。

### package time

类型Time 包含wall clock和monotonic clock两种时间值，monotonic的值单调递增，不受wall clock被修改的影响。

## goroutine

```go
go func_name()
```

包含 main 的 goroutine 为主协程。协程没有返回值。主协程停止后，其他协程会立即结束。

## channel

默认通道的读写都是阻塞的。对于有缓冲区的通道，当缓冲区满后，仍然会阻塞。

```go
var a chan int // a channel of int; at this point, 'a' is still nil
a = make(chan int) // 'a' is noew a channel

data := <- a // read data from a
a <- data // write data to a

c := make(chan type, LEN) // a buffered channel

// test if channel is closed
// if closed, open is false
d, open := <-c

//close channel
close(c)
```

定向通道

```go
c := make(<- chan int) // read only
c := make(chan <- int) // write only
```

## select

任何一个通道可以读写，则执行该case，否则阻塞（在没有default的情况下）

```go
select {
    case d := <- c1:
    //
    case d := <- c2:
    //
    default:
    //
}
```

