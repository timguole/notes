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

