# tcpdump basics

## 基本命令格式和选项

```shell
tcpdump [ -AbdDefhHIJKlLnNOpqStuUvxX# ] [ -B buffer_size ]
       [ -c count ]
       [ -C file_size ] [ -G rotate_seconds ] [ -F file ]
       [ -i interface ] [ -j tstamp_type ] [ -m module ] [ -M secret ]
       [ --number ] [ -Q in|out|inout ]
       [ -r file ] [ -V file ] [ -s snaplen ] [ -T type ] [ -w file ]
       [ -W filecount ]
       [ -E spi@ipaddr algo:secret,...  ]
       [ -y datalinktype ] [ -z postrotate-command ] [ -Z user ]
       [ --time-stamp-precision=tstamp_precision ]
       [ --immediate-mode ] [ --version ]
       [ expression ]

```

#### 选项说明

- -C：设置数据包文件大小；达到限制后，重新打开一个新文件
- -c：设置截获数据包的数量；达到数量后自动退出
- -B/--buffer-size=：设置数据包缓冲区大小
- -d/-dd/-ddd：将编译的匹配规则打印出来
- -D/--list-interfaces：列出所有网卡（网卡序号或名字可用于 -i 选项的参数）
- -e：打印数据包的链路层头部
- -f：将非本地IP地址以数字形式打印
- -F：制定匹配规则文件，命令行上的规则将被忽略
- -G：设置rotate数据包文件的秒数（需要时间戳参数）
- -i/--interface=：设置截获数据包的网卡（参考 -D 选项）
- -I/--monitor-mode：适用于无限网卡的监控模式
- --immediate-mode：不使用缓冲区；当没有使用-w选项时，此为默认设置
- -j/--time-stamp-type=：设置时间戳类型（参考pcap-tstamp(7)）
- -J：列出时间戳类型
- --time-stamp-precision=：设置时间戳精度，micro/nano
- -K/--dont-verify-checksums：不检查IP，TCP，UDP数据包的校验值；如果硬件也不校验，发出的TCP数据包校验值都会被标记为bad
- -l：打开行缓冲功能；这时输出以行为单位即时输出到标准输出
- -L/--list-data-link-types：列出网卡支持的数据链路类型
- -m：加载mib文件，该选项可以重复多次
- -M：设置用于检验TCP-MD5摘要的密码
- -n：不解析IP和端口号，直接以数字形式输出
- -N：不打印主机明的域名后缀
- -#/--number：打印数据包序号
- -O/--no-optimize：关闭匹配规则的编译优化
- -p/--no-promiscuous-mode：使网卡不要运行在混杂模式
- -Q/--direction=：设置捕获哪个方向的数据包，in/out/inout
- -q：快速或者安静模式；输出较少的协议信息
- -r：从文件中读取数据包，而非从网卡读取
- -S/--absolute-tcp-sequence-numbers：输出绝对tcp序列号
- -s/--snapshot-length=：设置从每个数据包抓取的数据长度
- -T：强制将匹配规则捕获的数据包解析为指定的类型
- -t/-tt/-ttt/-tttt/-ttttt：各种时间戳格式
- -u：打印未解码的nfs句柄
- -U/--packet-buffered：以数据包为单位进行缓冲输出
- -v/-vv/-vvv：设置verbose级别
- -V：从指定的文件中读取文件名字列表
- -w：将数据包写入到文件中
- -W：与-C/-G选项一起使用
- -x：十六进制打印数据包（头部和数据），不包括链路层头部
- -xx：十六进制打印数据包，包括链路层头部
- -X：以十六进制和ascii 打印数据包，不包括链路层头部
- -XX：以十六进制和ascii 打印数据包，包括链路层头部
- -y/--link-type=：设置数据链路类型
- -z：与-C/-G一起使用，设置rotate之后要执行的命令
- -Z/--relinquish-privileges=：在打开设备文件后，设置要切换的非root用户

#### 匹配规则

当不提供匹配规则时，所有数据包都会被捕获。匹配规则的语法可以参考pcap-filter(7) 。

```shell
Filter-Expression := Primitive ...
Primitive := [Quilifier ...] Id
Qualifier := Type | Dir | Proto
Type := host | net | port | portrange
Dir := src | dst | src or dst | src and dst | ra | ta | addr1 | addr2 | addr3 | addr4
Proto := ether | fddi | tr | wlan | ip | ip6 | arp | rarp | decnet | tcp | udp
Id := String | Number
```

- host 是默认的 type
- src or dst 是默认的方向
- 未指定协议，默认是 type 支持的所有协议

#### 输出格式

