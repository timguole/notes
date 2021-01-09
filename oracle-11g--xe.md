# oracle 11g XE 基础

## 安装

#### 前提要求

- centos 7 64-bit（经过测试，可以正常安装）
- 2GB 或者 两倍于内存容量的交换空间
- 可以使用 root 权限执行命令
- unzip
- netstat（非必须）

#### 安装数据库服务

首先，注册用户，从官网下载安装文件：`oracle-xe-11.2.0-1.0.x86_64.rpm.zip`。

解压缩安装包

```shell
unzip oracle-xe-11.2.0-1.0.x86_64.rpm.zip
```

安装软件包

> 安装过程需要 root 权限

```shell
cd Disk1
yum localinstall oracle-xe-11.2.0-1.0.x86_64.rpm
```

配置数据库服务

> 根据提示输入端口号，密码等信息

```shell
/etc/init.d/oracle-xe configure
```

设置环境变量

```shell
echo 'source /u01/app/oracle/product/11.2.0/xe/bin/oracle_env.sh' >> ~/.bashrc

# Copy bash config files for oracle user
cp /etc/skel/.bash* /u01/app/oracle/
echo 'source /u01/app/oracle/product/11.2.0/xe/bin/oracle_env.sh' >> /u01/app/oracle/.bashrc
```

连接数据库服务

> 根据提示，输入之前配置的密码
>
> 断开连接，输入 `exit`
>
> sqlplus 的提示符为：`SQL>`

```shell
# to a local host
sqlplus system

# to a remote oracle server
sqlplus systtem@hostname
```



## 基本操作

#### 启动和停止数据库

启动数据库

> 斜线（/）指示sqlplus 使用操作系统用户认证。用户应该属于 dba 组。

```shell
sqlplus / as sysdba
SQL> startup # type 'startup' after the prompt
```

如果启动成功，输出类似以下内容：

```shell
ORACLE instance started.

Total System Global Area  413372416 bytes
Fixed Size		    2227040 bytes
Variable Size		  297796768 bytes
Database Buffers	  109051904 bytes
Redo Buffers		    4296704 bytes
Database mounted.
Database opened.
```

#### 停止数据库

> 停止数据库之前，最好确认所有用户和应用已经断开连接

```shell
sqlplus / as sysdba
SQL> shutdown immediate
```

命令成功执行后的输出类似以下内容：

```shell
Database closed.
Database dismounted.
ORACLE instance shut down.
```

#### 停止数据库失败后的操作

如果输入`shutdown immediate`后长时间没有反映，可以按 CTLR+C终止操作，然后输入：

```shelll
SQL> shutdown abort
```

终止操作完成后，立即执行恢复操作，步骤如下：

```shell
SQL> startup # oracle will try to recover when starting up
SQL> shutdown immediate
```



创建用户并授权

```sql
create user chris identified by chris123;
grant CREATE SESSION, ALTER SESSION, CREATE DATABASE LINK, -
CREATE MATERIALIZED VIEW, CREATE PROCEDURE, CREATE PUBLIC SYNONYM, -
CREATE ROLE, CREATE SEQUENCE, CREATE SYNONYM, CREATE TABLE, -
CREATE TRIGGER, CREATE TYPE, CREATE VIEW, UNLIMITED TABLESPACE -
to chris;
```

解锁用户并修改密码

```sql
alter user hr account unlock;
alter user hr identified by USER-PASSWORD；
```

