# Linux GENEVE tunnel example

## Introduction

GENEVE uses UDP and default port is 6081.

## Example 1: point to point IP tunnel

There are two hosts in this example. If there is firewall, open udp port 6081.

- Host A: 10.1.1.1/24

- Host B: 10.1.1.2/24

  On host A

  ```shell
  ip link add geneve0 type geneve id 12 remote 10.1.1.2
  ip addr add 1.1.1.1/30 dev geneve0
  ip link set geneve0 up
  ```

  

On host B

```shell
ip link add geneve0 type geneve id 12 remote 10.1.1.1
ip addr add 1.1.1.2/30 dev geneve0
ip link set geneve0 up
```

Now test the tunnel

```shell
# on host A
ping 1.1.1.2

# on host B
ping 1.1.1.1
```

## Example 2: point to point Ethernet tunnel

There are four hosts in this example. Host A and host B are in the same LAN. Host C and host D are in the same LAN. We create a GENEVE tunnel between host B and host C, so that host A and host D can communicate directly via layer 2.

| Host A | eth0: 10.1.1.1/24            | N/A               |
| ------ | ---------------------------- | ----------------- |
| Host B | eth0: no IP address, promisc | eth1: 10.2.2.1/24 |
| Host C | eth0: no IP address, promisc | eth1: 10.2.2.2/24 |
| Host D | eth0: 10.1.1.2/24            | N/A               |

```shell
            LAN               GE   NEVE Tunnel                 LAN
             v                         v                        v
HostA:eth0-------eth0:HostB:eth1=============eth1:HostC:eth0--------eth0:HostD
```

On host B

```shell
ip link add br0 type bridge
ip link set br0 up
ip addr del x.x.x.x/x dev eth0 # If eth0 has IP address, delete the IP address.
ip link set eth0 promisc on
ip link set eth0 up
ip link set eth0 master br0
ip link add geneve0 type geneve id 12 remote 10.2.2.2
ip link set geneve0 master br0
ip link set geneve0 up
```

On host C

```shell
ip link add br0 type bridge
ip link set br0 up
ip addr del x.x.x.x/x dev eth0 # If eth0 has IP address, delete the IP address.
ip link set eth0 promisc on
ip link set eth0 up
ip link set eth0 master br0
ip link add geneve0 type geneve id 12 remote 10.2.2.1
ip link set geneve0 master br0
ip link set geneve0 up
```

Test  the tunnel on host A

```shell
ping 10.1.1.2
```

Test  the tunnel on host D

```shell
ping 10.1.1.1
```

