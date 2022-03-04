# MySQL安装和复制配置

## 前提条件

- mysql版本为8.0.xx
- 操作系统为centos 7

## MySQL的下载和安装

#### 下载MySQL安装包

> 请从官网获取实际需要的地址

```shell
wget https://cdn.mysql.com/archives/mysql-8.0/mysql-8.0.26-el7-x86_64.tar.gz
md5sum mysql-8.0.26-el7-x86_64.tar.gz
```

#### 新建mysql用户

```shell
useradd -m mysql

# set password
passwd mysql 
```

#### 创建mysql相关目录

> 请以实际情况修改安装和数据路径

```shell
# mysql conf file and data file
mkdir /data2/mysql
mkdir /data2/mysql/{etc,data,tmp,log,sock}
chown mysql:mysql -R /data2/mysql
chmod 750 -R /data2/mysql

# mysql program files
mkdir /usr/local/mysql
chown mysql:mysql /usr/local/mysql
```

#### 解压MySQL安装包

```shell
tar -zxf mysql-8.0.26-el7-x86_64.tar.gz
mv mysql-8.0.26-el7-x86_64/* /usr/local/mysql
chown mysql:mysql -R /usr/local/mysql
```

#### 设置环境变量（mysql用户）

```shell
echo 'export PATH=$PATH:/usr/local/mysql/bin' >> .bashrc;
. .bashrc
```

## 初始化MySQL

> 请以mysql用户进行操作

#### 创建配置文件（/data2/mysql/etc/my.cnf）。填入以下内容

```ini
[mysqld]
basedir=/usr/local/mysql
datadir=/data2/mysql/data
```

#### 初始化MySQL数据目录

> *注意：*
>
> 记录下临时root密码

```shell
cd /usr/local/mysql
bin/mysqld --defaults-file=/data2/mysql/etc/my.cnf --initialize --user=mysql
```



## 启动mysqld服务

#### 配置MySQL

TODO: add log, pid file, port, listen address options in my.cnf

#### 启动服务进程

```shell
mysqld_safe --defaults-file=/data2/mysql/etc/my.cnf &
```

#### 更新root用户密码

```shell
# conntect to mysql server locally
mysql -uroot -p -h127.0.0.1

# update password in mysql shell
alter user identified by 'PASSWORD';
```

#### 执行mysql安全加固脚本

```shell
mysql_secure_installation -h127.0.0.1
```

#### 防火墙和selinux配置

- 开放mysql端口
- 关闭selinux

## 配置MySQL复制

> 复制为GTID半同步模式

#### 停止source和replica服务进程

```shell
mysqladmin -uroot -h127.0.0.1 -p shutdown
```

#### 配置source

```ini
[mysqld]
# replication related settings
server_id=1
plugin-load="rpl_semi_sync_source=semisync_source.so;rpl_semi_sync_replica=semisync_replica.so"
rpl-semi-sync-source-enabled = 1
rpl-semi-sync-replica-enabled = 1
gtid_mode=ON
enforce-gtid-consistency=ON
```

#### 配置replica

```ini
[mysqld]
# replication related settings
server_id=2
plugin-load="rpl_semi_sync_source=semisync_source.so;rpl_semi_sync_replica=semisync_replica.so"
rpl-semi-sync-source-enabled = 1
rpl-semi-sync-replica-enabled = 1
gtid_mode=ON
enforce-gtid-consistency=ON
```

#### 启动source和replica服务

```shell
./mysql-start.sh
```

#### 在source中创建复制用户

```sql
CREATE USER 'repl'@'%' IDENTIFIED BY 'password';
GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%';
```

#### replica获取source的公钥

````shell
mysql -urepl -p -h'source-ip' --get-server-public-key
````

#### replica配置复制source

```sql
CHANGE REPLICATION SOURCE TO
	SOURCE_HOST = 'host ip',
	SOURCE_PORT = 3306,
	SOURCE_USER = 'repl',
	SOURCE_PASSWORD = 'password',
	SOURCE_AUTO_POSITION = 1;
```

#### 启动replica上的复制

```sql
start replica;
```

#### 查看复制状态

```ini
show replica status\G
```

