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

#### 安装客户端

如果其它机器需要连接数据库服务，那么需要在那些机器上安装单独的客户端。

首先，根据需要从官网下载客户端安装包。sqlplus命令行工具需要的安装包如下：

- oracle-instantclient11.2-basic-11.2.0.4.0-1.x86_64.rpm
- oracle-instantclient11.2-sqlplus-11.2.0.4.0-1.x86_64.rpm

安装软件包

```shell
yum localinstall oracle-instantclient11.2-basic-11.2.0.4.0-1.x86_64.rpm \
		oracle-instantclient11.2-sqlplus-11.2.0.4.0-1.x86_64.rpm
```

设置环境变量（在.bashrc中追加以下内容）

```shell
export LD_LIBRARY_PATH=/usr/lib/oracle/11.2/client64/lib/;
export PATH=/usr/lib/oracle/11.2/client64/bin/:$PATH;
```

测试远程连接数据库

> 提示：确保数据库服务所在机器的防火墙打开了 1521/tcp 端口

```shell
sqlplus system/PASSWORD@DB-ADDRESS
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

#### 连接数据库

本地连接

```shell
sqlplus name/password
sqlplus name
sqlplus / as sysdba
```

远程连接

```shell
sqlplus name[/password]@hostname[:port][/XE]
```

> 如果使用 `/nolog` 选项，那么 sqlplus 启动但不连接数据库，直到使用 `connect` 命令。

#### 设置环境变量

本地连接

```shell
source /u01/app/oracle/product/11.2.0/xe/bin/oracle_env.sh
```

远程连接

```shell
source /usr/lib/oracle/xe/app/oracle/product/11.2.0/client/bin/oracle_env.sh
```

#### 管理网络连接

数据库服务和远程客户端均包含 oracle net 组件用于建立网络连接。数据库服务端负责网络连接管理的是 listener 进程，负责监听和接收来自客户端的连接请求。Listener 默认监听的端口为1521和8080。

需要 Listener 的连接类型：

- 远程连接数据库服务
- 远程连接http服务
- 本地连接http服务

查看、停止、启动 listener

```shell
lsnrctl status
lsnrctl stop
lsnrctl start
```

##### 修改 listener 的数据库服务端口号

首先，停止 listener

```shell
lsnrctl stop
```

其次，编辑listenenr 配置文件（/u01/app/oracle/product/11.2.0/xe/network/admin/listener.ora）。找到以下内容，将1521修改为自己需要的端口号。

```shell
LISTENER =
  (DESCRIPTION_LIST =
    (DESCRIPTION =
      (ADDRESS = (PROTOCOL = IPC)(KEY = EXTPROC_FOR_XE))
      (ADDRESS = (PROTOCOL = TCP)(HOST = oracle11gxe)(PORT = 1521))

```

启动 listenenr

```shell
lsnrctl start
```

使用sqlplus连接到数据库服务，并执行以下命令：

> 注意：使用真实的主机名和端口号替换命令中的占位

```SQL
ALTER SYSTEM SET LOCAL_LISTENER = "(ADDRESS=(PROTOCOL=TCP)(HOST=<your hostname>)(PORT=<newport>))";

ALTER SYSTEM REGISTER;
```

最后，检查listener 状态。

##### 修改listener的http端口号

首先，确认listener正在运行。使用system用户连接到数据库，并执行以下命令：

> 注意：使用新端口号替换命令中的nnnn

```sql
EXEC DBMS_XDB.SETHTTPPORT(nnnn);
```

退出sqlplus，并检查listener状态

##### 允许铜鼓http协议远程访问数据库

使用system用户连接数据库并执行命令：

```sql
EXEC DBMS_XDB.SETLISTENERLOCALACCESS(FALSE);
```

#### 内存管理

基本概念

- instance：一个实例由一组后台进程和相关的内存构成（？）
- SGA：由buffer、cache pool、redo log buffer等组成
- PGA：

数据库服务会为每个连接数据库的客户端创建一个进程。每个进程有自己单独的pga，并共享sga。

##### 修改SGA 和 PGA

> 不要轻易修改这些参数。

```sql
ALTER SYSTEM SET pga_aggregate_target = 140 M; # use a real memory size
ALTER SYSTEM SET sga_target = 472 M; # use a real memory size
```

重启数据库服务，以使配置生效。

#### 存储管理

oracle数据库由逻辑上的tablespace和物理上的datafile组成。一个tablespace由一个或者多个datafile组成。除了存储用户数据的文件，还有用于控制和撤销操作的文件。

XE 默认的tablespace有：

- SYSTEM（SYS用户的默认表空间）
- SYSAUX
- TEMP
- UNDO
- USER（除了SYS用户外，其它用户的默认表空间）

#### Flash Recovery Area 相关操作

查看 Flash Recovery Area 的空间大小和使用率：

```sql
SELECT
NAME,
TO_CHAR(SPACE_LIMIT, '999,999,999,999') AS SPACE_LIMIT,
TO_CHAR(SPACE_LIMIT - SPACE_USED + SPACE_RECLAIMABLE, '999,999,999,999')
AS SPACE_AVAILABLE,
ROUND((SPACE_USED - SPACE_RECLAIMABLE)/SPACE_LIMIT * 100, 1)
AS PERCENT_FULL
FROM V$RECOVERY_FILE_DEST;
```

##### 设置 Flash Recovery Area位置

> 以 SYSDBA身份连接数据库
>
> new_path 所指向的目录必须已经存在。
>
> @表示开始执行一个脚本
>
> ？代表oracle home

```sql
ALTER SYSTEM SET DB_RECOVERY_FILE_DEST = 'new_path';

# Run a script to move the logs to new location
# Refer to 2-day dba doc for the content of movelogs.sql
@?/sqlplus/admin/movelogs
```

> **注意：不要手动删除旧的log文件**

##### 修改 Flash Recovery Area 大小

> 使用SYSDBA身份连接数据库
>
> new_size 格式为 nK，nM，nG

```sql
ALTER SYSTEM SET DB_RECOVERY_FILE_DEST_SIZE = new_size;
```



#### 管理用户

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

## 常见问题

#### ORA-00054

登录oracle,查找锁表的会话

```sql
# fin the sessions
select session_id from v$locked_object;

# show details for session XXX
SELECT sid, serial#, username, osuser FROM v$session where sid = XXX;

# kill sesion XXX, YYY is the serial number
ALTER SYSTEM KILL SESSION 'XXX,YYY';
```

