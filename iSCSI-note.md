# iSCSI基本配置

## 基本环境信息

- 操作系统：CentOS 8.2
- 最好是静态 IP 地址
- 除了系统盘，还有额外供 target 使用的存储盘

## 服务器端安装与配置

> **注意**
>
> 如果操作系统使用了 LVM，需要在文件 /etc/lvm/lvm.conf 中将用于 iSCSI target 的磁盘过滤掉，禁止本机 LVM 扫描这些磁盘。

### 安装软件包

安装 target 软件包

```shell
sudo dnf install targetcli
```

启动 target 服务

```shell
sudo systemctl enable --now target
sudo systemctl status target # Make sure target is active and running
```

### 配置 target （以 /dev/sdb为例）

首先，运行 targetcli，进入交互模式

```shell
targetcli
```

输出类似下面的样子

```shell
Copyright 2011-2013 by Datera, Inc and others.
For help on commands, type 'help'.

/> 
```

> 以下所有命令在 targetcli 的交互式界面下执行，直到输入 exit 退出 targetcli。

创建 block backstore

```shell
> /backstores/block create data1 /dev/sdb
```

创建新的 target

```shell
> /iscsi/ create iqn.2020-08.com.example:data1
```

切换到新 target 的 tpg 下面

```shell
> /iscsi/iqn.2020-08.com.example:data1/tpg1/
```

创建 LUN

```shell
> luns/ create /backstores/block/data1
```

创建 ACL

```shell
> acls/ create iqn.2020-08.com.example:init1
```

保存配置并退出

```shell
> / saveconfig
> exit
```

### 配置防火墙

```shell
sudo firewall-cmd --add-service=iscsi-target --permanent
sudo firewall-cmd --reload
```

## 客户端安装与配置

### 安装软件包并配置 initiatorname

```shell
sudo dnf install iscsi-initiator-utils
```

配置 initiatorname（/etc/iscsi/initiatorname.iscsi）

```ini
InitiatorName=iqn.2020-08.com.example:init1
```

### 发现并登录 target

```shell
sudo iscsiadm -m discovery -t sendtargets -p <target-ip>
sudo iscsiadm -m node --login
```

### 配置开机启动

```shell
sudo systemctl enable --now iscsi
sudo systemctl enable --now iscsid
```

### 格式化磁盘并挂在

> **注意**
>
> 在fstab中加入 '_netdev' 选项，例如：
>
> UUID=c1cbfa44-a8a9-4343-a88e-769b63a6e006    /data   ext4    defaults,_netdev  0 0

## 杂项

#### 配置文件的备份和恢复

默认情况下，targetcli 每次退出时自动保存当前配置，并将旧配置备份到 /etc/target/backup/ 目录下。可以使用 targetctl 命令恢复一个旧的配置文件：

```shell
targetctl restore <config-file>
```

