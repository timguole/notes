# Libvirt bridge network

## Temporary network settings

These settings are temporary, a system reboot or network reboot will lose these settings.

### Add a bridge

```shell
ip link add bridge0 type bridge
ip link set bridge0 up
```

### Add an interface to bridge

```shell
ip link set emp0s0 master bridge0
ip link set enp0s0 up
```



## Static network setings on Ubuntu

Ubuntu uses netplan for storing static network settings. The below example will define a bridge and add an interface to the bridge.

```yaml
network:
  version: 2
  ethernets:
    enp0s0:
      dhcp4: no

  bridges:
    bridge0:
      dhcp4: no
      interfaces:
        - enp0s0
```

Use netplan use aplly these settings.

```shell
netplan apply
```



## Define bridge network in libvirt

The network defination (bridge-net.xml)

```xml
<network>
  <name>bridge</name>
  <forward mode='bridge'/>
  <bridge name='bridge0'/>
</network>
```

Define this network

```shell
virsh net-define --file bridge-net.xml
virsh net-list --all
virsh net-start --network bridge
virsh net-autostart --network bridge
```

Now, create a VM and add a NIC connecting to the network 'bridge'.

To set up bridge for VLANs, add vlan sub-link on the physical interface and add the vlan sub-link to a bridge.