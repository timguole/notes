# MySQL MHA配置

## 前提条件

 * mysql已经完成GTID方式的半同步复制配置；source和replica上要配置相同的复制用户，互相获取了public key。
 * 主机之间ssh使用公钥登录（用户需要有修改系统ip地址的权限）
 * 操作系统为centos 7

> **注意：**
>
> 新版本的MySQL会要求客户端使用rsa密钥，但是mha并不支持这个功能。所以请将mha使用的用户密码设置为`with mysql_native_password`。
>
> 请参考mysql文档获取具体的修改密码设置的sql语句。

## 安装依赖包和必要工具

#### 安装epel源
```shell
rpm -Uvh http://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
yum makecache
```

#### 安装依赖包和工具
```shell
yum install -y \
	unzip \
	perl-Config-Tiny \
	perl-Log-Dispatch \
	perl-Parallel-ForkManager \
	perl-Time-HiRes \
	perl-Module-Install.noarch \
	perl-DBD-MySQL
```

## MHA下载与安装

#### 下载软件包

```shell
# mha node
wget https://codeload.github.com/yoshinorim/mha4mysql-node/zip/refs/heads/master

# mha manager
wget https://codeload.github.com/yoshinorim/mha4mysql-manager/zip/refs/heads/master
```

#### 配置所有slave为只读

```sql
set global super_read_only=1;
```

#### 安装MHA

> 如果需要将mha安装在非默认目录，请在Makefile.PL后面设置`PREFIX=/real-install-path`；
>
> 并且设置环境变量：
>
> `export PERL5LIB=$PERL5LIB:/real-install-path/share/perl5;`

在所有机器上解压并安装node包（manager会依赖node中的模块）

```perl
perl Makefile.PL
make
make install
```

在manager机器上解压并安装manager包

```perl
perl Makefile.PL
make
make install
```

复制manger源码目录中samples/scripts/下的脚本到mha的bin目录中，其中有关于ip切换的脚本。

#### 配置manager

应用配置文件：/etc/mha-manager/app1.cnf（以实际安装路径为准）

> 请根据实际slave数量，添加相应的`[server X]`

```ini
[server default]
manager_workdir=<path-to-workdir>
manager_log=<path-to-workdir>/manager.log

[server1]
hostname=<mysql-master-ip>
candidate_master=1

[server2]
hostname=<mysql-slave1-ip>
candidate_master=1

[server3]
hostname=<mysql-slave3-ip>
candidate_master=1
```
mha manager配置文件：/etc/mha-manager/mha_default.cnf（以实际安装路径为准）

```ini
[server default]
user=<mysql-user>
password=<mysql-password>
ssh_user=<ssh-user>
repl_user=<repl-user>
repl_password=<repl-password>
master_binlog_dir= <path-to-binlog>
remote_workdir=<path-on-node>
secondary_check_script= masterha_secondary_check -s <slave1-ip> -s <slave2-ip>
ping_interval=3 
master_ip_failover_script= <path-to-master_ip_failover>
# shutdown_script= /script/masterha/power_manager
# report_script= /script/masterha/send_report
# master_ip_online_change_script= /script/masterha/master_ip_online_change
```

>**注意：**
>
>创建上面配置文件中需要的路径，保证mha相关进程有权限读写#### 

#### 修改ip切换脚本

默认情况下，mha自带的master_ip_failover脚本不不能正常运行，需要修改。请将以下代码加入到脚本中的合适位置。修改main函数，在stop分支的eval中调用stop_vip()，在start分支的eval中FIXME_XXX处调用start_vip()

```perl
my $vip = '<vip>/netmask';
my $ssh_start_vip = "/usr/sbin/ip addr add $vip dev <REAL-NIC>";
my $ssh_stop_vip = "/usr/sbin/ip addr del $vip dev <REAL-NIC>";
		
sub start_vip() {
	`ssh $ssh_user\@$new_master_host \" $ssh_start_vip \"`;
}
sub stop_vip() {
	return 0  unless  ($ssh_user);
	`ssh $ssh_user\@$orig_master_host \" $ssh_stop_vip \"`;
}
````

#### 设置VIP

```shell
# run on master node
ip addr add VIP dev NIC
```

#### 检查mha配置

如果输出中包含错误信息，请解决所有存在的问题。

```shell
masterha_check_repl --global_conf=/path/to/mha_default.cnf --conf=/path/to/mha-manager/app1.cnf
```

#### 启动manager进程

```shell
nohup masterha_manager --global_conf=/path/to/mha_default.cnf --conf=/path/to/mha-manager/app1.cnf &
```

#### 检查manger状态

正常情况下，输出中包含正在运行的app，pid，master ip信息。

```shell
masterha_check_status --global_conf=/path/to/mha_default.cnf --conf=/path/to/mha-manager/app1.cnf
```

