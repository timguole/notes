# oVirt 

## 基本术语

**oVirt Engine**：管理整个oVirt环境的组件，可以运行在一个VM中。

**Engine VM**：在self-hosted模式中，运行Engine的虚拟机。此虚拟机在部署时由程序自动创建。

**oVirt Node**：精简版本的Enterprise Linux。专门用来做Host。

**Host**：组成整个虚拟化环境的计算结点。可以是运行标准Enterprise Linux的机器，也可以是运行oVirt Node的机器。

## 基本硬件要求

- CPU支持虚拟化
- 最少2GB内存
- 至少一个千兆网卡
- 至少60GB硬盘

## 准备配置信息

### FQDN

需要为以下机器准备FQDN：

- 所有的host节点
- 所有的存储节点
- oVirt Engine VM

### 用户密码

需要为以下用户准备密码

- host/root
- Engine VM/root
- Portal/admin@internal

### 子网IP分配

需要为Host和Engine VM分配IP

## 配置DNS

oVirt 所有节点都需要一个FQDN，并且要保证可以反向解析。可以dnsmasq实现或者named。如果没有现成的DNS服务，就需要专门准备两台机器作为DNS服务器。

安装dnsmasq

```shell
yum install dnsmasq
systemctl enable --now dnsmasq.service
systemctl status dnsmasq # make suire service started normally
```

dnsmasq默认使用/etc/hosts文件，将准备好的IP地址和FQDN写入即可。重启服务检测dns服务是否正常

```shell
dig <FQDN>
dig -x <IP address>
```

## 存储配置

oVirt支持NFS，iSCSI，FC，Gluster Storage。如果使用glusterfs，oVirt 只支持replica1和replica3（glusterfs的安装配置在单独的文档中）。

除了基本的glusterfs配置之外，oVirt 使用的卷还需要额外设置：

```shell
gluster volume set gv0 cluster.quorum-type auto
gluster volume set gv0 network.ping-timeout 10
gluster volume set gv0 auth.allow \*
gluster volume set gv0 group virt
gluster volume set gv0 storage.owner-uid 36
gluster volume set gv0 storage.owner-gid 36
gluster volume set gv0 server.allow-insecure on
```

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