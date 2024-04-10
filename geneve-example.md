# Linux GENEVE tunnel example

## Introduction

GENEVE uses UDP and default port is 6081.

## Example

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

