# oVirt 

## 基本术语

**oVirt Engine**：管理整个oVirt环境的组件集合，可以运行在一个VM中。

**Engine VM**：在self-hosted模式中，运行 Engine 的虚拟机。此虚拟机在部署时由程序自动创建。

**Engine machine**：这个次会在文档中出现，和 Engine VM 是同义词，有时会出现全称 “oVirt Engine machine”。

**oVirt Node**：精简版本的 Enterprise Linux，默认已经安装了需要的软件源；专门用来做Host。

**self-hosted engine node**：运行Engine VM的主机。

**self-hosted engine storage domain**：专属于 Engine VM 的存储域。

**Host**：组成整个虚拟化环境的计算结点。可以是运行标准Enterprise Linux的机器，也可以是运行oVirt Node的机器。一个主机必须属于某个集群（Cluster）

**VDSM**：运行在所有 Host 上，负责与 Engine 通信。

**HA service**：只运行在 self-hosted engine node 上，负责 Engine VM 的高可用。

**Data Center**：一组资源的逻辑集合，包括：主机集群，逻辑网络，存储域等。一个管理界面（Portal）可以管理多个 data center。oVirt 在初始安装时会创建默认的数据中心：Default。

**Cluster**：集群；拥有相同CPU类型和使用相同存储域的主机组成一个集群。一个集群必须属于某个 Data Center。一个虚拟机只会在一个集群内做迁移。集群可以用来运行 VM 或者 Glusterfs Server，但是一个集群只能做一种用途。

**SPM**：存储池管理者；一个赋予某个主机的角色，用于管理存储域。当拥有 SPM角色的主机出问题时，Engine 会保证重新选择一台主机。可以为不同的主机配置不同的 SPM 优先级，以保证 运行关键虚拟机的主机不会承担 SPM 角色，避免虚拟机资源被抢用。

**Data Domain**：用于存储虚拟机磁盘，快照，模板，OVF 文件。一个数据中心可以有多个不同类型的数据域，但是必须都是共享类型，不能时本地类型。一个数据域只能属于一个数据中心。

**ISO Domain**：存储ISO镜像，不同 data center 之间可以共享。只能是 NFS 类型。



## 基本硬件要求

- CPU支持虚拟化
- TODO



## 准备安装所需的信息

### 准备FQDN

需要为以下机器准备FQDN：

- 所有的host节点
- 所有的存储节点
- oVirt Engine VM

### 准备用户密码

需要为以下用户准备密码

- 所有 Host 的 root 密码
- Engine VM的 root 密码
- Portal 的管理员密码，用户默认为 admin

### 子网IP分配

需要为 Host 和 Engine VM 分配管理网段的 IP

## 配置 DNS

oVirt 所有节点都需要一个 FQDN，并且要保证可以反向解析。可以使用 dnsmasq 实现或者 named 。如果没有现成的 DNS 服务，就需要专门准备两台机器作为 DNS 服务器。

安装 dnsmasq

```shell
yum install dnsmasq
systemctl enable --now dnsmasq.service
systemctl status dnsmasq # make suire service started normally
```

dnsmasq 默认使用 /etc/hosts 文件，将准备好的 IP 地址和 FQDN 写入即可。重启服务检测 DNS 服务是否正常

```shell
dig <FQDN> # forward query
dig -x <IP address> # reverse query
```

## 存储配置

oVirt 支持 NFS，iSCSI，FC，Gluster Storage 。如果使用 glusterfs，oVirt 只支持 replica1 和replica3（glusterfs的安装配置在单独的文档中）。

除了基本的 glusterfs 配置之外，oVirt 使用的卷还需要额外设置：

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

Deply Host 并不是虚拟化环境之外的一台独立机器，而是虚拟化环境的第一个节点。安装好 Deploy Host 之后，在其上部署第一个 oVirt Engine VM 。同样，Deploy Host 可以安装标准的 Entetprise Linux，也可以安装 oVirt Node。

#### 选择安装 oVirt Node

1. 下载 oVirt Node ISO 镜像：https://www.ovirt.org/download/node.html
2. 在一台物理机上安装oVirt Node（过程与安装centos类似）

> oVirt Node 基于RHEL，是精简版的RHEL。它只包含用于将机器作为hypervisor的必要软件包以及用于管理目的的cockpit。
>
> Cockpit需要比较新的浏览器才能正常工作。

#### 选择安装 Enterprise Linux

1. 像往常一样安装 CentOS即可（不要关闭防火墙和 SELinux）

2. 安装必要的软件源和模块

   ```shell
   yum install https://resources.ovirt.org/pub/yum-repo/ovirt-release44.rpm
   yum module -y enable javapackages-tools
   yum module -y enable pki-deps
   yum module -y enable postgresql:12
   ```

## oVirt Engine 的安装

由于软件包比较大（超过2GB，并且应该会越来越大），可以提前手工安装 ovirt-engine-appliance。

```shell
yum install ovirt-engine-appliance
```

1. 安装部署工具

   ```shell
   yum install ovirt-hosted-engine-setup tmux
   ```

2. 运行部署命令（部署过程比较耗时，建议在tmux中运行，防止网络中断影响安装过程）

   ```shell
   tmux
   
   # In the tmux session window
   hosted-engine --deploy
   ```


3. 根据提示输入Engine VM配置信息
4. 成功安装之后，登录管理 Portal：https://<engine-fqdn>，查看基本状态。

## 添加新的 Host

首先，按照 “安装 Deploy Host” 的方式为 Host 安装操作系统。然后执行以下步骤：

1. 在管理 Portal 页面上点击 **Compute -> Host**
2. 点击 **New** 按钮
3. 填入 Host 的 FQDN，root 密码等信息
4. 点击 **OK** 按钮，等待添加完成

## 添加 Self-hosted Engine Node

添加新的 Self-hosted Engine Node，可以实现 oVirt Engine 的高可用。步骤与添加普通 Host 基本一样，只是在输入步骤3需要的基本信息后，点击 **Hosted Engine**，选择 **Deploy**，最后点击 **OK**。