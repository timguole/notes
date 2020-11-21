# apt source list 笔记

source list文件所在位置为：

- /etc/apt/sources.list
- /etc/apt/sources.list.d/*.list
- /etc/apt/sources.list.d/*.sources

## 文件格式

### 单行格式（后缀名为 .list）

> 注释以 # 开头

基本语法为：

```shell
TYPE '[' option=value ... ']' uri suite [component1] [component2] [...]
```

其中 TYPE 为 deb 或者 deb-src 。

option的语法为：

```shell
[ option=value ] # no space between option name and value
[ option=value1,value2 ] # option wuth more than one values, use ',' with no space
[ option1=val1 option2=val2,val3 ]
[ option-=val ] # remove a value from defaults
[ option+=val ] # add a val to defaults
```

### DEB822格式

> 以 # 开头的行为注释

基本语法：

```shell
ENTRY1: VALUE1
ENTRY2: VALUE2
ENTRY3: VALUE3
# an empty line stops a stanza

ENTRY1: VALUE1
ENTRY2: VALUE2
ENTRY3: VALUE3
```

## deb 和 deb-src

deb 和 deb-src 以两级形式的debian归档为参照：distribution/component。

distribution 可以是：stable，testing或者一个codename。

component 可以是：main，contrib，non-free。

当 suite 是一个精确的路径时，必须以斜线（/）结尾，并且省略 component。

## Debian archive (package repo)

基本目录结构

```shell
dists/ # sources.list 中的 suite 对应其中的一个子目录，component则是suite中的子目录
indices/
project/
pool/
tools/
```

apt工具会先从dists/<suite-name>中下载Release或者InRelease文件，文件中有不同component的package列表信息。更详细信息参考debian的wiki：https://wiki.debian.org/DebianRepository/Format

