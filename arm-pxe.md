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

efi模式启动时，不同系统使用的bootloader不同。cobbler支持针对profile设置filename，以此实现不同发行版使用各自的bootloader文件。

```shell
mkdir -p /var/lib/cobbler/loaders/grub/c8aa64

# then copy grubaa64.efi from iso to grub/c8aa64

# find shim rpm file and extract the shimaa64.efi
rpm2cpio THE-SHIM-RPM.rpm | cpio -dimv

# copy the extracted shim*.efi fiels to grub/c8aa64

# edit the cobbler profile
cobbler profile edit --name centos8-aarch64 --filename grub/c8aa64/shimaa64.efi
```

创建一个cobbler post sync trigger用来处理bootloader配置文件

```shell
#!/bin/bash

cd /var/lib/tftpboot/grub;
for s in system/*; do
        mac=$(basename $s);
        c="grub.cfg-01-$(echo $mac | tr ':' '-')";
        ln -sf $s $c;

        # some grub versions have bug.
        #The required cfg file name has a trailing dash '-'
        c="grub.cfg-01-$(echo $mac | tr ':' '-')-";
        ln -sf $s $c;
done

for loader in /var/lib/cobbler/loaders/grub/*; do
        if [ -d "${loader}" ]; then
                cp -r ${loader} /var/lib/tftpboot/grub/

                (
                        cd /var/lib/tftpboot/grub/$(basename ${loader});

                        for p in ../grub.cfg-01-*; do
                                ln -sf $p $(basename $p);
                        done
                )
        fi
done
```

### 尝试 pxe 启动

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

目前，aarch64版本的系统似乎仍然不稳定，bootloader获取的cfg文件名可能有错误。可以使用tcpdump抓包确认tftp过程中请求的具体文件名字。

如果出现以下画面，说明内核与initrd都下载到了机器上。

```shell
Loading kernel ...
loading initial ramdisk ...
EFI stub: Booting Linux Kernel...
EFI stub: EFI_RNG_PROTOCOL unavailable, no randomness supplied
EFI stub: Using DTR from configuration table
EFI stub: Exiting boot services and installing virtual address map...
```

接下来要耐心等待，因为没有任何消息输出。安装完成，服务器会自动重启。
