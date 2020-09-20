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
- 函数内的变量是局部变量
- 函数定义之外的变量是全局变量
- 向未声明的变量赋值，该变量将自动成为window的属性
- 函数内的变量如果没有使用var声明，则为全局变量
- html中的全局对象为window

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

### 函数

```javascript
function foobar() {
    // statements
}

function () {
    // statements
}
```

### 字符串

- 使用单引号或者双引号

- 索引从0开始

- 使用反斜线转义

- 内置属性 length 包含字符串的长度

```javascript
var s = "abc";
a[0] = 'b'; // now a is 'bbc'
var l = a.length; // 3

var s = new String("abc") // typeof s is Object
typeof s // Object
```

连接字符串操作

```javascript
var a = 'abc';
var b = 'def';
var c = a + b;

var d = 1;
var e = '2';
d + e; // '12'
d + a; // 1abc
```

数学运算符

JavaScript 拥有和其他语言类似的运算符（包括赋值运算）。

### 比较运算符

JavaScript 拥有和其他语言类似的比较运算符。此外，JavaScript 还有 !== 和 ===（不仅比较值，还要比较类型）

```javascript
var x = 2;
x == 2; // true
x == '2'; // true
x === 2; // true
x === '2'; // false
```

### 逻辑运算符

JavaScript 拥有和其他语言类似的逻辑运算符。

### 条件运算符

```javascript
condition ? expre : expre;
```

### 条件语句

和 C 语言类似，包括 if 和 switch。

### 循环

for 循环的第一种语法和 C 相同。for/in 方式用于遍历对象的属性：

```javascript
var cat = {name: 'tom', age: 12, gender: 'male'};
for (p in cat) { // x is the property name string
    console.log(x + ': ' + cat[x]);
}
```

while 和 do/while 循环与 C 语言类似。

JavaScript 也有 break 和 continue 语句。

### typeof

typeof 操作符用于返回值的类型。

```javascript
typeof "John"                 // 返回 string
typeof 3.14                   // 返回 number
typeof NaN                    // 返回 number
typeof false                  // 返回 boolean
typeof [1,2,3,4]              // 返回 object
typeof {name:'John', age:34}  // 返回 object
typeof new Date()             // 返回 object
typeof function () {}         // 返回 function
typeof myCar                  // 返回 undefined (如果 myCar 没有声明)
typeof null                   // 返回 object
```

对于 object 类型，可以使用 constructor 属性判断具体类型

```javascript
"John".constructor                 // 返回函数 String()  { [native code] }
(3.14).constructor                 // 返回函数 Number()  { [native code] }
false.constructor                  // 返回函数 Boolean() { [native code] }
[1,2,3,4].constructor              // 返回函数 Array()   { [native code] }
{name:'John', age:34}.constructor  // 返回函数 Object()  { [native code] }
new Date().constructor             // 返回函数 Date()    { [native code] }
function () {}.constructor         // 返回函数 Function(){ [native code] }
```

### null

null 是对象类型，表示没有值。

### undefined

是一种类型。没有赋值的变量就是 undefined 类型，值也为 undefined。

### 类型转换

```javascript
String(x) // convert x to string
number(x) // convert x to number
```

### 正则表达式

基本语法： /<regex>/<optional flag>

flags: i, g, m

支持正则表达式的字符串方法有：search, replace, match, split。

查找和替换

```javascript
var s = 'hello, world!';
var i = s.search(/hello/i); // return index of hello in s.
vat s1 = s.replace(/hello/i, "Hallo"); // return the new string
```

测试

```javascript
var p = /ab/;
var i = p.test("habor"); // return true, if pattern is found in string.
```

exec()函数会匹配并返回匹配结果，结果是一个数组，第一个元素为完整的匹配字符串，后面的为捕获的正则表达式中的括号部分。如果不存在匹配，返回null

```javascript
var p = /(t)(h|b)e/i;
var r = p.exec("The best one!"); // r is ['The', 'T', 'h']
```

正则表达式对象

```javascript
var r = new RegExp("pattern", "optional flag");
```

### 处理异常

```javascript
try {
    // some code
} catch (e) {
    // process exception
} finally {
    // statements always executed
}
```

### 抛出异常

throw 抛出的异常会被 catch 捕获。

```javascript
throw "error message";
```

### 变量提升

变量声明会被提升到作用域的顶部，但是变量初始化不会。

### 严格模式

在代码顶部打开严格模式：

```javascript
"use strict" // just a string literal, not a statement
```

严格模式的限制：

- 不允许使用未声明的变量（用 var 声明）
- 不允许删除变量和对象
- 变量不能重名
- 不允许使用 8 进制
- 不能使用转义字符
- 不能对只读属性赋值
- getter 方法获取的属性不能赋值
- 不能删除的属性无法被删除
- 变量名不能是 eval 和 arguments
- 不能使用 eval() 函数创建的变量
- this 不能指向全局对象
- 保留关键字： implements interface let package private protected public static yield

### this

- 在对象方法中，this 绑定到对性
- 在函数中，this 绑定到 全局对象（严格模式下为 undefined）
- 在全局，绑定到全局对象
- 在 DOM 事件中绑定到产生事件的元素

### let

let 关键字让 javascript 有了块作用域的概念。

在代码快内，let 关键字声明的变量作用域为代码块内。代码块就是花括号包围的一段代码，包括 for 循环，if 语句等。

在全局作用域内，let 声明的变量不属于 window；与此相反，var 声明的变量属于 window。

在函数体内，let 和 var 类似。

在同一个作用域内，var 和 let 互相不能重置已经声明的变量。

let 声明的变量不会提升

### const

const 声明的是常量，值不可改变。声明时必须初始化。也是块级作用域。

## 表单验证

form 标签的 onsubmit 属性可以调用一个 javascript 函数，如果函数返回 false，则不提交表单。

在 input 标签设置了约束属性之后，使用 DOM 属性 validity 的属性可以验证数据。



## HTML 事件

事件可以来自浏览器本身或者用户。事件会触发 JavaScript 执行注册的代码。常见事件有：

- onclick
- onload
- onchange
- onmouseover
- onmouseout
- onkeydown

## 函数的绑定

默认，函数会绑定到 window 对象，在函数中 this 代表 window。在元素的各种事件属性中，this 代表元素自己。

