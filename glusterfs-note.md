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

