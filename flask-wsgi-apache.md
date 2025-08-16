# Flask Basics

## Installation

```shell
sudo apt install python3-flask apache2 libapache2-mod-wsgi-py3
```



## Config apache2 wsgi

#### Add useer for wsgi process

```shell
sudo useradd -m -s /usr/bin/nologin wsgi
```



#### Create app dir

```shell
sudo mkdir /var/www/flaskapp
chown wsgi:wsgi -R /var/www/flaskapp
```



#### Create wsgi config file

Create a new config file: `/etc/apache2/site-available/wsgi.conf`

```xml
<VirtualHost *>
    #ServerName example.com

    WSGIDaemonProcess flaskapp user=wsgi group=wsgi processes=1 threads=3 display-name=flaskapp
    WSGIScriptAlias /app /var/www/flaskapp/flaskapp.wsgi
    WSGIProcessGroup flaskapp

    <Directory /var/www/flaskapp>
        Require all granted
    </Directory>
</VirtualHost>
```



#### Disable & enable sites

```shell
sudo a2dissite 000-default.conf
sudo a2ensite wsgi.conf
```



#### Enable wsgi module

```shell
sudo a2enmod wsgi 
```



#### The first testing app

Put the following code into `/var/www/flaskapp/flaskapp.wsgi`

```python
#!/usr/bin/python3

from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello():
    return 'hello world!'
```



#### Restart apache

```shell
sudo systemctl restart apache2.service
```



## Notes

#### The process relationship

```shell
ps -ef | grep apache2
root        3248       1  0 05:30 ?        00:00:00 /usr/sbin/apache2 -k start
www-data    3456    3248  0 05:33 ?        00:00:00 /usr/sbin/apache2 -k start
www-data    3489    3248  0 05:33 ?        00:00:00 /usr/sbin/apache2 -k start
wsgi        3777    3248  0 06:25 ?        00:00:00 /usr/sbin/apache2 -k start
```



#### The wsgi process information

```shell
get_exec_path	['/usr/local/sbin', '/usr/local/bin', '/usr/sbin', '/usr/bin', '/sbin', '/bin', '/snap/bin']
getcwd	/home/wsgi
getegid	1001
geteuid	1001
getgroups	[1001]
getppid	3248
getresgid	(1001, 1001, 1001)
getuid	1001
getcwdb	b'/home/wsgi'
getgid	1001
getloadavg	(0.05, 0.04, 0.01)
getpgrp	3248
getresuid	(1001, 1001, 1001)
getpid	3777
```



## 对flask设计的理解

关于wsgi规范

python定义了wsgi作为web服务器和web应用之间的接口调用规范，大体上是下面这个形式：

```python
def wsgi_app(environ, start_response):

	# process request
    # ...
    start_response(status_code, response_header)
    return response_body # an iterable object
```

当运行web应用时，我们需要在web服务器的配置中指定符合wsgi接口调用规范的callable对象（普通函数，对象）。
在不使用web应用框架的情况下，开发一个web应用的过程不仅包含实现应用逻辑相关的代码，还需要实现wsgi接口调用规范相关的代码。这样的确比较麻烦。web应用框架帮助开发者处理了这些琐碎的细节，并且提供了更多机制以简化web应用开发过程，比如路由机制、全局配置管理、中间件管理。

#### flask做了更多

除了上述提到的web应用框架功能，flask还想方设法的简web应用化代码编写过程。flask引入了几个“全局对象”，比如，flask.current_app、flask.g、flask.request等。这些对象被用来避免在方法调用时的繁琐且冗长的参数传递。web应用代码可以在任何位置引入“全局对象”并使用它们。

#### 多线程让事情变的复杂

> 在下面的叙述中，“全局对象”和“代理对象”是对同一种对象的两种称呼。

如果web服务器运行在单线程模式下，所有客户端请求都需要排队等处理，那事情将会非常简单（只是运行效率会很低）。但是我们必须考虑多线程的情况。在多线程的模式下，多个请求分别被不同的线程同时处理，运行在不同线程中的web应用代码对“全局对象”有不同的理解。不同线程中的代码都认为“全局对象”指向了自己正在处理的请求。flask使用对象代理解决这个问题。“全局对象”
并不是包含特定请求数据的“实际对象”，它们只是在不同线程中代理了对应线程正在处理的请求相关的“实际对象”。当web应用代码访问request或g中的数据时，这些代理对象会从当前线程的TLD或者其他线程私有数据存储位置获取“实际对象”的数据。

flask引入context的概念来表示“实际对象”。每当一个客户端请求被分配给一个线程去处理，flask首先将客户端请求相关的数据包装成一个context，并保存在线程的TLD或者类似位置。web应用代码通过“全局对象”的代理就可以访问到这些数据。
