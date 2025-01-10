# Linux Policy-based routing on Ubuntu

The below netplan configuration do the following:

- in the main routing table, the default gateway is 10.34.19.1
- add two dns servers for the system
- add a vlan link: vlan1113
- add a routing table '1022' for link vlan1113
- add default gateway, link scope route to routing table '1022' 
- add 15.0.0.0/8, 16.0.0.0/8 to the main routing table

```yaml
network:
  ethernets:
    ens2f1:
      addresses:
        - 10.34.19.13/24
      routes:
        - to: default
          via: 10.34.19.1
      nameservers:
        addresses:
          - 223.5.5.5
          - 223.6.6.6
        search: []
  vlans:
    vlan1113:
      id: 1113
      link: ens2f1
      addresses:
        - 10.22.204.201/24
      routes:
        - to: default
          via: 10.22.204.1
          table: 1022
        - to: 10.22.204.0/24
          scope: link
          from: 10.22.204.201
          table: 1022
        - to: 15.0.0.0/8
          via: 10.22.204.1
        - to: 16.0.0.0/8
          via: 10.22.204.1
      routing-policy:
        - from: 10.22.204.201
          table: 1022
  version: 2

```

