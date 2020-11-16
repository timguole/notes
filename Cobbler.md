# Cobbler 笔记

## PXE 安装引导Linux系统的过程

pxe从网络引导，安装Linux的基本步骤如下：

1. 机器加电后从支持pxe的网卡启动，向dhcp服务器申请IP和pxe相关信息
2. dhcp服务根据subnent或者host mac为机器分配ip地址、bootloader文件路径
3. 机器根据bootloader文件的路径向tftp服务获取bootloader文件和相关的配置文件
4. 机器根据bootloader配置文件中指示的内核与initrd路径（一般是url）获取内核
5. bootloader启动内核
6. 内核使用initrd启动完成后，运行系统安装程序。
7. 安装程序可以根据ks或者pressed文件自动完成系统的安装和配置过程

#### 涉及到的服务

dhcpd，tftpd，httpd

### pxe启动步骤的一些细节说明

dhcp服务会根据机器提供的 option arch决定为机器提供哪个bootloader；机器的arch可能是以下情况之一：

- 06：x86 32-bit efi
- 07：amd64 64-bit efi
- legacy模式启动

bootloader路径由dhcp配置中的filename选项提供。

tftp目录中包含pxelinux.0或者其它bootloader。一般在同一级目录下还存在这与bootloader对应的配置文件目录。可能的目录结构是：

```shell
/tftpboot/pxelinux.0
/tftpboot/pxelinux.cfg/
```

机器向tftp请求bootloader配置文件时，文件名可能是：

- 自己的uudi
- ip地址的hex表示（包括完整IP或者部分前缀）
- mac地址（前缀为01-）
- default

以机器ip为192.168.2.91，MAC为88-99-aa-bb-cc-dd为例，机器向tftp请求bootloader配置文件时可能的文件名如下：

```shell
/tftpboot/pxelinux.cfg/b8945908-d6a6-41a9-611d-74a6ab80b83d
 /tftpboot/pxelinux.cfg/01-88-99-aa-bb-cc-dd
 /tftpboot/pxelinux.cfg/C0A8025B
 /tftpboot/pxelinux.cfg/C0A8025
 /tftpboot/pxelinux.cfg/C0A802
 /tftpboot/pxelinux.cfg/C0A80
 /tftpboot/pxelinux.cfg/C0A8
 /tftpboot/pxelinux.cfg/C0A
 /tftpboot/pxelinux.cfg/C0
 /tftpboot/pxelinux.cfg/C
 /tftpboot/pxelinux.cfg/default
```

## Cobbler 基本术语

- distro：对应一个linux发行版镜像；该对象包含发行版的版本，cpu架构等信息；一个distro可以被多个profile关联
- profile：与一个distro关联，并且包含更多安装配置信息；一个profile可以被多个system关联；profile可以用来针对特定的硬件型号或者机器的用途做定制。
- system：代表一个具体的主机，与一个profile关联，包含更详细的配置信息，例如，mac地址，ip地址，主机名等

## Cobbler安装与基本配置

> 操作系统为centos 7。需要提前准备好cobbler用来提供服务的网卡IP等信息

### 准备

关闭防火墙（或者开放相应的端口）

```shell
firewall-cmd --add-port=25151/tcp --permanent
firewall-cmd --add-service=dhcp --permanent 
firewall-cmd --add-service=http --permanent 
firewall-cmd --add-service=https --permanent 
firewall-cmd --add-service=tftp --permanent 
firewall-cmd --add-service=dhcp --permanent
firewall-cmd --reload
```

关闭selinux（/etc/selinux/config）

```shell
SELINUX=disabled
```

安装epel repo

```shell
wget https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
yum localinstall epel-release-latest-7.noarch.rpm 
yum makecache 
```

### 安装

安装cobbler

```shell
yum install cobbler cobbler-web dhcp xinetd debmirror fence-agents pykickstart
```

### 基本配置和启动

配置cobbler（/etc/cobbler/settings）

```yaml
default_password_crypted: "" # generated by openssl passwd -1
server: 127.0.0.1 # ip address
next_server: 127.0.0.1 # tftp server ip address
manage_dhcp: 1
```

修改dhcp模板（/etc/cobbler/dhcp.template）

> 注意：不要修改任何以 $ 开头的字符串。只要修改子网信息即可

```nginx
subnet 192.168.1.0 netmask 255.255.255.0 {
    option routers             192.168.1.1;
    option domain-name-servers 192.168.1.210,192.168.1.211;
    option subnet-mask         255.255.255.0;
    filename                   "/pxelinux.0";
    default-lease-time         21600;
    max-lease-time             43200;
    next-server                $next_server;
}
```

配置tftpd（/etc/xinetd.d/tftp ）

```nginx
 disable = no
```

启动cobbler

```shell
systemctl start xinetd.service
systemctl enable xinetd.service
systemctl start httpd.service
systemctl enable httpd.service
systemctl enable dhcpd.service
systemctl start cobblerd.service
systemctl enable cobblerd.service
systemctl status cobblerd.service
```

检查配置

```shell
cobbler check
```

> 根据输出的提示，进行下载或者配置，并重新检查。可以返回进行此操作

### 第一次同步

```shell
systemctl restart cobblerd
cobbler sync
```

同步成功后的输出类似下面：

```shell
...OMITTED LOTS OF OUTPUT...
running shell triggers from /var/lib/cobbler/triggers/change/*
*** TASK COMPLETE ***
```

### 导入第一个发行版（以centos7为例）

> 准备好要导入的dvd光盘或者iso文件

```shell
mount -t iso9660 -o ro /dev/cdrom /media/
cobbler import --name=centos7 --arch=x86_64 --path=/media
```

查看导入的发行版

```shell
cobbler distro list
cobbler profile list
cobbler distro report --name=centos7-x86_64
```

### 创建一个system

```shell
cobbler system add --name=testvm --profile=centos7-x86_64
```

可以为一个system设置nic，ip，mac，hostname，gateway，netmask等信息。设置mac后，cobbler将会在bootloader的配置文件目录中为该system生成名为01-MAC的配置文件。机器从pxe启动时，将会获取此文件，而不再显示cobbler的profile菜单。

### 自动安装

可以为profile或者system指定一个自定义的ks文件，而不使用默认的ks文件。ks模板中的$SNIPPET将会被实际的snippet文件替换掉，生成最终可用的ks文件（不包含模板标签等信息）。

### 遇到的问题

使用kvm+ipxe测试时，出现类似以下的错误消息：

```shell
dracut-initqueue: /sbin/dmsquash-live-root no space left on /dev/loop0
... /dev/root does not exist...
```

pxe启动的过程中，bootloader根据配置文件加载内核与initrd之后，内核启动并把initrd挂载为rootfs。接下来会使用curl下载squashfs.img。在centos7中这个img大约有500 MB，机器内存如果小于1.5 GB，则会出现内存耗尽，出现上面的消息。squashfs无法下载和挂载，导致安装失败，进入紧急模式。