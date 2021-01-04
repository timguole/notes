# ARM64 pxe boot

记录使用 cobbler 进行 arm64 服务器 pxe 启动的过程。arm64 又叫做 aarch64，这两个名字是同义词。

### 基本信息

- Linux 发行版为 centos 8 aarch64 架构
- 服务器为华为泰山，使用鲲鹏处理器（aarch64架构）
- cobbler 版本为 3.2.0

### 安装过程

> 首先完成 cobbler 的基本安装配置

导入 centos 8 aarch64 的镜像。

```shell
mount centos8.iso /mnt
cobbler import --name centos8 --arch aarch64 --path /mnt
```

创建一个 system

```shell
cobbler system add --name taishan --profile centos8-aarch64-aarch64 \
	--mac MAC --ip-address IPADDRESS --gateway GATEWAY --hostname taishan \
	--netmask MASK
```

同步 cobbler

```shell
cobbler sync
```



### 尝试 pxe 启动

到目前为止，尝试 pxe 启动会失败。原因是；

- cobbler自动生成的配置文件内容和文件名字都有问题。
- 默认 cobbler 没有提供 aarch64 的 boot loader

首先，把 ISO 中的 EFI boot loader 拷贝到 /var/lib/cobbler/loaders 目录下。然后 执行 cobbler sync，cobbler会在 tftpboot/ 目录下生成配置文件。大致是这个样子：

```shell
/var/lib/tftpboot/
├── boot
├── BOOTAA64.EFI
├── COPYING.syslinux
├── COPYING.yaboot
├── etc
├── grub
│?? ├── aarch64_menu_items.cfg
│?? ├── grub.cfg
│?? ├── images -> ../images
│?? ├── local_efi.cfg
│?? ├── local_legacy.cfg
│?? ├── local_powerpc-ieee1275.cfg
│?? ├── system
│?? │?? ├── A00030F2
│?? │?? └── cc:64:a6:67:70:bf
│?? ├── system_link
│?? │?? └── taishan -> ../system/A00030F2
│?? └── x86_64_menu_items.cfg
├── grubaa64.efi
├── grub.cfg
├── grub-x86_64.efi
├── grub-x86.efi
├── images
│?? ├── centos7.9-x86_64
│?? │?? ├── initrd.img
│?? │?? └── vmlinuz
│?? └── centos8-aarch64-aarch64
│??     ├── initrd.img
│??     └── vmlinuz
├── images2
├── menu.c32
├── ppc
├── pxelinux.0
├── pxelinux.cfg
│?? ├── 01-cc-64-a6-67-70-bf
│?? ├── A00030F2
│?? └── default
├── README
├── s390x
└── yaboot

13 directories, 27 files
```

对比上面的目录结构和 dhcpd 的配置文件，会发现 grubaa64.efi 的路径根本就不对。手工拷贝 grubaa64.efi 到 grub 目录下，尝试 pxe 启动并进行抓包；不断修正错误，就会发现以下内容：

- 服务器从 dhcp 获取到 filename 为 /grub/grubaa64.efi
- 服务器接下来会向tftp 请求  /grub/grubaa64.efi
- 继续请求  /grub/grub.cfg-01-MAC
- 参考 ISO 镜像中的 grub.cfg， /grub/grub.cfg-01-MAC中的 clinux 应该为 linux；cinitrd 应该为 initrd

经过不断修正错误后，最终在 pxe 启动时可以看到类似下面的消息：

```shell
Loading kernel ...
loading initial ramdisk ...
EFI stub: Booting Linux Kernel...
EFI stub: EFI_RNG_PROTOCOL unavailable, no randomness supplied
EFI stub: Using DTR from configuration table
EFI stub: Esiting boot services and installing virtual address map...
```

接下来要耐心等待，因为没有任何消息输出。安装完成，服务器会自动重启。

### 解决 cobbler 带来的问题

经过手工修改 cobbler 自动生成的配置，aarch64 的 pxe 算是成功了。接下来是如何在每次 cobbler sync 时自动改正 cobbler 的错误。

