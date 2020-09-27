# ajax

ajax 可以在不刷新整个页面的情况下，与服务器交换数据，并更新部分页面。

## XMLHttpRequest 对象

#### 创建对象

```javascript
var xhr = new XMLHttpRequest();
```

#### 对象方法

##### open

第一个参数为请求的方法：GET/POST；第二个参数为请求的 URL；第三个参数为异步（true）或者同步(false)。

```javascript
xhr.open("POST", "http://www.foo.bar/", true);
```

##### send

参数为向服务器发送的数据（GET方法不需要参数）

```javascript
xhr.send("data");
```

##### setRequestHeader

添加 HTTP 头信息

```javascript
xhr.setRequestHeader("Header-Name", "Value");
```

当使用 POST 方法时，需要添加如下头部

```javascript
xhr.setRequestHeader("Content-type", "application/x-www-form-urlencoded");
```

#### 对象属性

##### responseText/responseXML

服务器返回数据可以通过这两个属性获取。

##### onreadystatechange

该属性保存用于响应 XMLHttpRequest 对象事件的函数

##### readyState

对象的各种状态

- 0：请求未初始化
- 1：服务器连接已经建立
- 2：请求已经发送
- 3：请求处理中
- 请求处理完成，数据已经就绪

##### status

- 200：OK
- 404：page not found

## 完整示例

```javascript
var xhr = new XMLHttpRequest();
xhr.onreadystatechange = function() {
    if (xhr.readyState == 4 && xhr.status == 200) {
        // process xhr.responseText
    }
};
xhr.setRequestHeader("Content-type", "application/x-www-form-urlencoded");
xhr.open("POST", "url", true);
xhr.send("name=FOO&gender=BAR");
```

