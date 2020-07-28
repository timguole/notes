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

