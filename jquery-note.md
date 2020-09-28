# jQuery

jquery 是一个 javascript 库，通过选取 HTML 元素并对其进行操作。

jQuery 在 HTML DOM 标签加载完成后就开始执行（document ready event）；而 javascript 是在 window.onload 开始执行（即html文档和外部资源都加载完毕之后）；

## 安装和引用

从官网下载 jQuery 代码文件，放在合适的位置。在网页中引用 js 文件 即可。

```javascript
<head>
<script src="path/to/jquery.min.js"></script>
</head>
```

## 基本语法

```javascript
$(selector).action()
```

其中

- $ 代表 jquery
- selector 是选取的 HTML 元素
- action 是对元素执行的动作

#### 选择器

```javascript
$(document).ready();
$("*"); // select all elements
$("p").hide(); // select all 'p' tags
$(this).hide(); // select current element
$("p.foo").hide(); // elements 'p' in class foo
$(".foo"); // all elements in class foo
$("#foo").hide(); // element whose id is foo
$("ul li:first"); // select first li tag of first ul tag
$("ul li:first-child"); // select first li tag of every ul tag
$("[href]"); // select elements with href attribute
$("a[target='_blank']"); // select all a tag whose target is _blank
$("a[target!='_blank']"); // select all a tag whose target is not _blank
$(":button"); // select all button tag and input tag which is of type button
$("tr:even"); // all even rows
$("tr:odd"); // all odd rows
```

#### 文档就绪事件

```javascript
$(document).ready(function() {
    // code
});
// or
$(function() {
    // code
});
```

#### jQuery 处理常见事件

```javascript
$(SELECTOR).EVENT(function() {
    // code
});
```

常见事件

- 鼠标事件：click dbclick mouseenter mouseleave hover
- 键盘事件：keypress keydown keyup
- 表单事件：submit change focus blur
- 文档窗口事件：load resize scroll unload

## 特效

#### show/hide/toggle

```javascript
// speed can be slow, fast or millinseconds
// If there are three arguments, the second is a sring describing how to animate (linear/swing)
$(SELECTOR).hide(SPEED, CALLBACK);
$(SELECTOR).show(SPEED, CALLBACK);
$(SELECTOR).toggle();
```

## 操作HTML

几个重要方法：

#### text() 和 html()

text() 设置或者返回元素的文本内容；html() 返回或者设置元素内的内容，包括嵌套在其中的标签。假如有：

```html
<p id="preword">a<b>b</b>b>c</p>p>
```

```javascript
$("#preword").text(); // return abc
$("#preword").html(); // return a <b>b</b> c
```

```javascript
// argument is string
$("#preword").text("new text");

// argument is a callback function
// i: index of this element in the selected element list
// o: old text
$("preword").text(function(i, o) {
    // code
});
```

> html() 拥有相似的调用方式

#### val()

返回表单字段的值

```javascript
$("#name").val();
```

> 当设置value时， val()和text()有相似的调用方式

#### attr() 和 prop()

attr()可能返回undefined；prop()可能放回空字符串

```javascript
$("#home").attr("href"); // return value of href
$("#name").attr("href", "www.foo.bar"); // set one attribution
$("#name").attr({"href": "www.foo.bar", "target": "_self"}); // set multiple values
```

#### append/prepend

在选中元素的内部最后/开头增加元素

#### after/before

在选中元素之后/之前添加元素