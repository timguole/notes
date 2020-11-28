# Cobbler 源码安装
## 基本信息
安装过程是在 CentOS 7 上进行的。Cobbler 版本为 3.2.0 。Cobbler 的文档不太完善。源代码包中的dockerfile 包含更准确的信息，可以用于准备构建环境或者时运行时依赖。
## 安装 额外软件包 源
安装 epel 和 ius
```shell
yum makecache fast
yum install -y epel-release
yum install -y https://repo.ius.io/ius-release-el7.rpm \
		https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
yum makecache fast
```
安装编译和开发依赖包

> 这些软件包是开发和构建 rpm 时需要的软件包。

```shell
yum install -y          \
    git                     \
    rsync                   \
    make                    \
    dnf-plugins-core        \
    epel-rpm-macros         \
    openssl                 \
    python-sphinx           \
    python36-coverage       \
    python36-devel          \
    python36-wheel          \
    python36-distro         \
    python36-future         \
    python36-pyflakes       \
    python36-pycodestyle    \
    python36-setuptools     \
    python36-requests       \
    python36-sphinx         \
    rpm-build               \
    which
```

如果需要安装cobbler web，需要安装 mod_ssl

```shell
yum install mod_ssl
```

安装运行时依赖的软件包

```shell
yum install -y          \
    httpd                   \
    python36-mod_wsgi       \
    python36-pymongo        \
    python36-PyYAML         \
    python36-netaddr        \
    python36-simplejson     \
    python36-tornado        \
    python36-django         \
    python36-dns            \
    python36-ldap3          \
    python36-cheetah        \
    createrepo_c            \
    xorriso                 \
    grub2-efi-ia32-modules  \
    grub2-efi-x64-modules   \
    logrotate               \
    syslinux                \
    systemd-sysv            \
    tftp-server             \
    dhcp                    \
    xinetd                  \
    yum-utils               \
    createrepo             \
    debmirror              \
    pykickstart            \
    fence-agents

```

## 构建并安装 cobbler

> make rpms 生成的安装包在rpm-build目录下

```shell
tar -zxf cobbler-3.2.0.tar.gz 
cd cobbler-3.2.0/
make rpms
cd rpm-build
yum localinstall cobbler-3.2.0-1.el7.noarch.rpm
```

如果需要安装 cobbler web，需要安装 cobbler-web 。

```shell
yum localinstall cobbler-web-3.2.0-1.el7.noarch.rpm
```

> 注意：cobbler-web 需要额外的 https 配置

