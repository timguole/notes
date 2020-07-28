# Ceph Storage Cluster

## Install

### System Information

The installation includes 6 nodes: 3 monitors and 3 OSDs. Each node has 2 CPUs and 2 GB RAM. OSD node has 3 disks: 1 for OS, two for storage.

Softwares versions are:

- OS: Ubuntu 20.04 LTS
- Ceph: octopus 15.2.1

### Preparation

- Make suire system is up to date
- NTP works properly
- Config hostname

### Install Ceph package

On all nodes, install ceph packages

```shell
sudo apt install ceph
```

### Initialize the first monitor

Create initial ceph cofig file on the first monitor node.

```ini
# /etc/ceph/ceph.conf
[global]
fsid = <uuid> # Use uuidgen tool to create a UUID
mon initial members = <hostname>
mon host = <ip address>
public network = <ip address> # clients connect to this IP
auth cluster required = cephx
auth service required = cephx
auth client required = cephx
osd pool default size = 3
osd pool default min size = 2

```

Create keys for monitor, osd, admin user.

```shell
# Create monitor key
ceph-authtool --create-keyring /tmp/ceph.mon.keyring \
		--gen-key -n mon. --cap mon 'allow *'

# Create ceph.client.admin.keyring
ceph-authtool --create-keyring /etc/ceph/ceph.client.admin.keyring \
		--gen-key -n client.admin \
		--cap mon 'allow *' \
		--cap osd 'allow *' \
		--cap mds 'allow *' \
		--cap mgr 'allow *'

# Create OSD bootstrap keyring
sudo ceph-authtool --create-keyring \
		/var/lib/ceph/bootstrap-osd/ceph.keyring \
		--gen-key -n client.bootstrap-osd \
		--cap mon 'profile bootstrap-osd' \
		--cap mgr 'allow r'

# Import adminn keyring
ceph-authtool /tmp/ceph.mon.keyring \
		--import-keyring /etc/ceph/ceph.client.admin.keyring

# Import osd bootstrap keyring
ceph-authtool /tmp/ceph.mon.keyring \
		--import-keyring /var/lib/ceph/bootstrap-osd/ceph.keyring

# change file owner to ceph
chown ceph:ceph /tmp/ceph.mon.keyring

# create map; substitue values in <*>
monmaptool --create --add <hostname> <host ip> \
		--fsid <fs-uuid> /tmp/monmap

# create monitor directory; substitue <hostname> with real hostname
sudo -u ceph mkdir /var/lib/ceph/mon/ceph-<hostname>

# create the monitor
sudo -u ceph ceph-mon --mkfs -i <hostname> --monmap /tmp/monmap \
		--keyring /tmp/ceph.mon.keyring
```

Config ceph mgr

```shell
sudo -u ceph mkdir /var/lib/ceph/mgr/ceph-<hostname>
sudo -u ceph touch /var/lib/ceph/mgr/ceph-<hostname>/keyring
ceph auth get-or-create mgr.<hostname> mon 'allow profile mgr' \
		osd 'allow *' \
		mds 'allow *' > /var/lib/ceph/mgr/ceph-<hostname>/keyring

```

Start services

```shell
sudo systemctl daemon-reload 
sudo systemctl enable ceph-mon.target
sudo systemctl enable ceph-mon.service
systemctl start ceph-mon.service
ceph mon enable-msgr2
systemctl restart ceph-mon.service
sudo systemctl start ceph-mon@<hostname>.service

sudo systemctl enable ceph-mgr.target
sudo systemctl enable ceph-mgr.service
sudo systemctl start ceph-mgr.service
sudo systemctl start ceph-mgr@<hostname>.service
```

### Copy config files

Copy files under /etc/ceph/ to all other nodes, so that other nodes know where the monitor is, and can use the admin keyring to connect to the first monitor

### Config other monitors

```shell
# create directory for monitor
sudo -u ceph mkdir /var/lib/ceph/mon/ceph-$(hostname)

# get monitor keyring from the first monitor
sudo ceph auth get mon. -o /tmp/ceph.mon.keyring

# get the initial map
sudo ceph mon getmap -o /tmp/ceph.mon.map

# populate the monitor directory
sudo chown ceph:ceph /tmp/ceph.mon.keyring
sudo -u ceph ceph-mon --mkfs -i $(hostname) \
		--monmap /tmp/ceph.mon.map \
		--keyring /tmp/ceph.mon.keyring

# config mgr
sudo -u ceph mkdir /var/lib/ceph/mgr/ceph-$(hostname)
sudo -u ceph touch /var/lib/ceph/mgr/ceph-$(hostname)/keyring
sudo ceph auth get-or-create mgr.$(hostname) \
		mon 'allow profile mgr' \
		osd 'allow *' \
		mds 'allow *' | sudo tee \
		/var/lib/ceph/mgr/ceph-$(hostname)/keyring

```

Enable and start systemd units.

Repeat the above steps on all other monitor nodes.

### Add OSDs

```shell
# get bootstrap-osd keyring
sudo ceph auth get client.bootstrap-osd \
		-o /var/lib/ceph/bootstrap-osd/ceph.keyring

# Run the command for each block device
sudo ceph-volume lvm create --data </dev/diskname>
```

Enable and start systemd units for each OSD daemon

```shell
sudo systemctl daemon-reload
sudo systemctl enable ceph-osd.target

# for each osd daemon
sudo systectl enable ceph-osd@<id>.service
sudo systectl start ceph-osd@<id>.service
```

### Verify ceph status

If everything goes well, `ceph -s` will print something like this.

- all monitors are running
- all managers are running
- all OSDs are running
- id is the fs-uuid
- health is HEALTH_OK
- pgs is active+clean

```shell
ceph -s
  cluster:
    id:     b75cc090-5489-413a-a3b6-b913da25d1eb
    health: HEALTH_OK
 
  services:
    mon: 3 daemons, quorum cephm1,cephm2,cephm3 (age 73m)
    mgr: cephm3(active, since 73m), standbys: cephm1, cephm2
    osd: 6 osds: 6 up (since 72m), 6 in (since 4w)
 
  data:
    pools:   8 pools, 201 pgs
    objects: 202 objects, 5.4 KiB
    usage:   6.2 GiB used, 24 GiB / 30 GiB avail
    pgs:     201 active+clean
 

```

## OSD Basics

Basic commands

```shell
# Create pool
ceph osd pool create <pool> [pg_num] [pgp_num]

# Remove pool
ceph osd pool rm <pool> <repeat-pool> --yes-i-really-really-mean-it

# Pool quota
ceph osd pool set-quota <pool> max_bytes|max_objects <value>
ceph osd pool get-quota <pool>

# List pool
ceph osd pool ls [detail]

# Pool application
ceph osd pool application {enable|disable} <pool> {cephfs|rbd|rgw}
ceph osd pool application get <pool> [<app>] [<key>]
ceph osd pool application set <pool> <app> <key> <value>
ceph osd pool application rm <pool> <app> <key>

```

## RBD Basics

```shell
# Init pool
rbd pool init <pool>
    
# Create user
# create user block; read/write on vms; read-only on images
# save output to /etc/ceph/ceph.client.<username>.keyring
ceph auth get-or-create client.block \
		mon 'profile rbd' \
		osd 'profile rbd pool=vms, profile rbd-read-only pool=images' \
		mgr 'profile rbd pool=images'

# Stats
rbd --id <user> pool stats <pool>

# Create block device image
rbd [--id <id>] create --size <size-in-MB> <pool>/<image>

# List block device
rbd [--id <id>] list <pool>

# List trashed block device
rbd [--id <id>] trash ls <pool>

# Image info
# size, object count, feature, timestamp, id, etc.
rbd [--id <id>] info <pool>/<image>

# Resize image
# extend
rbd [--id <id>] resize --size <size-in-MB> <pool>/<image>
# shrink
rbd [--id <id>] resize --size <size-in-MB> <pool>/<image> --allow-shrink

# Remove image
rbd [--id <id>] rm <pool>/<image>

# Trash image
rbd [--id <id>] trsh mv <pool>/<image>

# Remove from trash
rbd [--id <id>] trsh rm <pool>/<image>

# Restore from trash
rbd [--id <id>] trsh restore <pool>/<image> [--image <new-name>]

# Map/unmap device
sudo rbd [--id <id>] device {map|unmap} <pool>/<image>

# Snapshots creation
rbd [--id <id>] snap create <pool>/<image>@<snap>

# Snapshots listing
rbd [--id <id>] snap ls <pool>/<image>

# Snapshots rollback
rbd [--id <id>] snap roolback <pool>/<image>@<snp>

# Snapshots deletion
rbd [--id <id>] snap rm <pool>/<image>@<snp>

# Snapshots purging
rbd [--id <id>] snap purge <pool>/<image>

# Snapshots protection (for layering)
rbd [--id <id>] snap protect <pool>/<image>@<snp>

# Snapshots clone
rbd [--id <id>] clone <pool>/<image>@<snp> <pool>/<clone>

# Unprotect napshots
rbd [--id <id>] snap unprotect <pool>/<image>@<snp>

# List children
rbd [--id <id>] children <pool>/<image>@<snp>

# Flatten image
rbd [--id <id>] flatten <pool>/<image>
```

## Ceph admin commands

```shell
# cluster quorum status
ceph quorum_status --format json-pretty

# monitor status
ceph mon stat
ceph mon dump

# OSD tree
ceph osd tree

# deal with crash
ceph crash ls
ceph crash info <event-id>

# archive crash warnings
ceph crash archive <event-id>
ceph crash archive-all

# Health detail
ceph health detail
```

