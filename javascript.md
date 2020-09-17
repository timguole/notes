# JavaScript

## 如何在 html 中使用 JavaScript

html 提供了 <script></script> 标签来嵌入 JavaScript 代码。这个标签可以出现在 head 和 body 元素中。此外，该标签还有 src 属性可以用来指定 JavaScript 源代码文件（后缀名为 .js）。该标签也有一个 type 属性用来指定脚本语言的类型，但是在现代的浏览器上已经没有必要了。

## 基本语法

### 关键字和常量

JavaScript 的关键字有：

```javascript
break case catch class const continue debugger default delete do else export extends finally for function if import in instanceof new return switch this throw try typeof var void while with
```

预定义的常量：

```javascript
true false
```

### 注释

```javascript
// double slash marks the start of comment
/* multiple-line
comment
*/
```

### 语句

JavaScript 语句以分号分隔。

### 变量

- 变量以 var 声明
- 变量名字（标识符）可以以字母、下划线、$ 开头
- 变量声明后是未定义的
- 重复声明变量不是错误，也不影响变量的值

声明变量：

```javascript
var name;
var age, gender;
```

声明并初始化：

```javascript
var name = "Tom";
var gender = 'male';
var age = 12;
var height = 1.71; // float number
```

### 基本数据类型

数字、字符串（双引号或者单引号）、布尔值、null、undefined、symbol。

JavaScript 的变量具有动态类型。

### 引用数据类型

数组、对象、函数

#### 数组

```javascript
var a = new Array();
a[0] = 'abc';
a[1] = 'cdf';

var a = new Array('a', 'b', c );

var a = ['a', 'b'];
```

#### 对象

```javascript
var o = { name: "tom",
         gender: "male",
         age: 12,
        run: function() {
            return "hahaha";
        }
};

// Access object properties
o.name
// or
o["name"]

// access object method without () will return the definition of that method
o.run(); // return hahaha
o.run; // return the method definition: function() {return "hahaha"; }
```

### 声明时确定变量类型

```javascript
var a = new Array;
var s = new String;
var n = new Num;
var b = new Boolean;
var o = new Object;
```



