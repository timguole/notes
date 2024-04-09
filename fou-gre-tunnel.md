# Linux FOU GRE tunnel example
### Introduction
FOU stands for Foo-Over-UDP. FOU is a tunnel encapsulation. It uses UDP and can encapsulates GRE, IPIP etc.

### Example config
This example consists of two Ubuntu hosts:
- Host A: 10.1.1.1/24
- Host B: 10.1.1.2/24
On Host A, create FOU port, GRE device, and add IP address to the GRE device. The FOU port is 55555. The ipproto number is 47(GRE).
```shell
sudo modprobe fou
sudo ip fou add port 55555 ipproto 47
sudo ip link add gre1 type gre ttl 255 remote B-IP local A-IP encap fou encap-sport auto encap-dport 55555
sudo ip addr add 1.1.1.1/30 dev gre1
sudo ip link set gre1 up
# If there is firewall, open the FOU udp port
sudo iptables -D INPUT -p udp --dport 55555 -j ACCEPT
ping 1.1.1.1
ping 1.1.1.2
```

On Host B, create FOU port, GRE device, and add IP address to the GRE device. The FOU port is 55555. The ipproto number is 47(GRE).
```shell
sudo modprobe fou
sudo ip fou add port 55555 ipproto 47
sudo ip link add gre1 type gre ttl 255 remote A-IP local B-IP encap fou encap-sport auto encap-dport 55555
sudo ip addr add 1.1.1.2/30 dev gre1
sudo ip link set gre1 up
# If there is firewall, open the FOU udp port
sudo iptables -D INPUT -p udp --dport 55555 -j ACCEPT
ping 1.1.1.1
ping 1.1.1.2
```

