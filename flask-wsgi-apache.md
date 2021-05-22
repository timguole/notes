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

    WSGIDaemonProcess flaskapp user=wsgi group=wsgi threads=3
    WSGIScriptAlias /app /var/www/flaskapp/flaskapp.wsgi

    <Directory /var/www/flaskapp>
        WSGIProcessGroup flaskapp
        WSGIApplicationGroup %{GLOBAL}
        Order deny,allow
        Allow from all
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

