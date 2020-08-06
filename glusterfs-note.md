# Glusterfs

## 基础

Glusterfs 提供类似NFS的运程文件系统服务。Glusterfs 可以根据不同需求配置不同类型的卷：分布型 Glusterfs 卷（默认类型），复制 型Glusterfs 卷，分布式复制型 Glusterfs 卷。

### 分布型 Glusterfs 卷（默认类型）

- 同一个文件会保存到同一个brick上
- 同一个卷中的不同文件会根据算法分散到不同brick上

### 复制 型Glusterfs 卷

同一份文件会被复制到不同的brick上。

### 分布式复制型 Glusterfs 卷

根据复制要求，在创建卷时，命令行上相邻的N个brick会组成一个复制组。例如，有6个brick，复制数量为2，那么第一和第二个brick保存同样的数据，第三和第四保存同样的数据；也称为3x2。如果复制数量为3，那么第一，第二和第三个brick为一个复制组，其余的为一个复制组；也被称为2x3。

## 快速安装

首先假设有三台服务器，主机名分别为：server1，server2，server3。三台机器可以使用主机名互相访问。

为了实现让glusterfs存储数据，服务器应该至少有两个块设备：一个用于操作系统，一个用于glusterfs的brick。如果有更多的磁盘，可以创建多个brick。

#### 格式化磁盘

根据磁盘数量，重复步骤1至3，最后使用mount确保文件系统可以正常挂载。

```shell
mkfs.xfs -i size=512 /dev/sdb1
mkdir -p /data/brick1
echo '/dev/sdb1 /data/brick1 xfs defaults 1 2' >> /etc/fstab
mount -a && mount
```

#### 安装glusterfs软件包

```shell
dnf install centos-release-gluster
dnf config-manager --set-enabled PowerTools
dnf install glusterfs-server
systemctl enable --now glusterd

# allow glusterfs traffic
firewall-cmd --add-service=glusterfs --permanent
firewall-cmd --reload
```

*在其它服务器上重复以上操作：格式化磁盘、安装glusterfs软件包*

#### 建立信任池

在server1服务器上执行以下命令

```shell
gluster peer probe server2
gluster peer probe server3
```

再选择一台不同的机器，比如server2，执行以下命令

```shell
gluster peer probe server1
```

#### 检查信任池状态

选择一台服务器，执行以下命令

```shell
gluster peer status
```

输出类似下面的内容

```shell
Number of Peers: 2

Hostname: server2
Uuid: f0e7b138-4874-4bc0-ab91-54f20c7068b4
State: Peer in Cluster (Connected)

Hostname: server3
Uuid: f0e7b138-4532-4bc0-ab91-54f20c701241
State: Peer in Cluster (Connected)
```

#### 创建一个卷

在所有机器上执行以下命令

```shell
mkdir -p /data/brick1/gv0
```

在其中一台机器上执行以下命令

```shell
# create a volume
gluster volume create gv0 replica 3 \
		server1:/data/brick1/gv0 \
		server2:/data/brick1/gv0 \
		server3:/data/brick1/gv0
# output is something like the following:
# volume create: gv0: success: please start the volume to access data

# Then start the volume
gluster volume start gv0

# output is like:
# volume start: gv0: success
```

查看卷的信息

```shell
gluster volume info
```

命令输出类似以下内容

```shell
Volume Name: gv0
Type: Replicate
Volume ID: f25cc3d8-631f-41bd-96e1-3e22a4c6f71f
Status: Started
Snapshot Count: 0
Number of Bricks: 1 x 3 = 3
Transport-type: tcp
Bricks:
Brick1: server1:/data/brick1/gv0
Brick2: server2:/data/brick1/gv0
Brick3: server3:/data/brick1/gv0
Options Reconfigured:
transport.address-family: inet
```

#### 测试卷是否可用

在一台机器上执行如下命令

```shell
mount -t glusterfs server1:/gv0 /mnt
```

使用mount命令查看是否已经成功挂载。

## 其他操作

移除一台服务器

```shell
gluster peer detacch <server-name>
```

