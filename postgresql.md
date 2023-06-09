# Postgresql Basicss

## 连接数据库服务

pg 安装完成后有一个默认用户：postgres。在Linux系统中，这个用户即是系统用户，也是数据库内部超级管理员用户。

#### 本地连接数据库

```shell
$ sudo -u postgres psql
```

此方式表示当系统的当前用户（psql进程的uid）也是一个数据库用户使，psql可以直接连接数据库服务，不需要其他认证。

#### 远程连接数据库

```shell
$ psql -h pg_server_address DB_NAME DB_USER
```

pg_hba.conf配置文件中包含了访问控制规则，默认只有本地loopback地质可以连接数据库服务。

## 使用数据库

#### 查看基本情况

列出所有数据库

```sql
\l
```

列出所有用户

```sql
\du
```

列出当前数据库的模式

```sql
\dn
```

列出当前数据库的表格

```sql
\dt
```



#### 创建数据库

创建数据库

```sql
create database DB_NAME;
```

创建模式

```sql
\c DB_NAME;
create schema SCHEMA_NAME;
```

创建表格

```sql
create table schema_name.table_name (...);
```



#### 创建用户

创建用户

```sql
create user USER_NAME with password 'XXXXX';
```

授权

```sql
alter user USER_NAME set search_path to SCHEMA_NAME;
grant usage on database DB_NAME to USER_NAME; # 默认应该有这个权限
\c DB_NAME; # 必须先切换到对应的数据库下，才能操作对schema的授权
grant usage on schema SCHEMA_NAME to USER_NAME;
grant select,insert,update,delete on all tables in schema SCHEMA_NAME to USER_NAME; #根据实际需求授权
```

