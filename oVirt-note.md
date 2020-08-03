# oVirt 

## 基本硬件要求

- CPU支持虚拟化
- 最少2GB内存
- 至少一个千兆网卡
- 至少60GB硬盘

## 准备配置信息

### FQDN

TODO

### 用户密码

需要为以下用户准备密码

- host/root
- Engine VM/root
- Portal/admin@internal

### 子网IP分配

需要为Host和Engine VM分配IP

## 配置DNS

oVirt 机器需要一个FQDN，并且DNS服务器上要有正向和反向解析。

### named服务的安装和配置

TODO

## 存储配置

oVirt支持NFS，iSCSI，FC，Gluster Storage。

#### Gluster Storage配置

oVirt 只支持replica1和replica3。

TODO

## 安装Deploy Host

1. 下载oVirt Node ISO镜像：https://www.ovirt.org/download/node.html
2. 在一台物理机上安装oVirt Node（过程与安装centos类似）

> oVirt Node 基于RHEL，是精简版的RHEL。它只包含用于将机器作为hypervisor的必要软件包以及用于管理目的的cockpit。
>
> Cockpit需要比较新的浏览器才能正常工作。

## oVirt Engine的安装

可以提前手工安装ovirt-engine-appliance

```shell
yum install ovirt-engine-appliance
```

1. 安装部署工具

   ```shell
   yum install ovirt-hosted-engine-setup tmux
   ```

2. 运行部署脚本（部署过程比较耗时，建议在tmux中运行）

   ```shell
   tmux
   
   # In the tmux session window
   hosted-engine --deploy
   ```

   3. 根据提示输入Engine VM配置信息

   > **注意**
   >
   > Engine VM 与物理机在同一个网段