# Sample rocky 9 kickstart file

This is a sample kickstart file for rocky 9.x on Cobbler. It demonstrates how to configure time zone, network, users, ssh key and auto partition.

```shell
auth  --useshadow
# To disable SELinux, add 'selinux=0'
bootloader --location=mbr --append="selinux=0" --boot-drive=sda --driveorder=sda
# It is important to set the disk name for installation. Here, we just use sda
ignoredisk --only-use=sda
# This command means clear 'all' partitions on disk 'sda'
clearpart --all --initlabel --drives=sda
# Clear the Master Boot Record
zerombr
# The option '--type' is mandatory to autopart
autopart --type=lvm --nohome
# Use text mode install
text
# Firewall configuration
firewall --enabled
# Run the Setup Agent on first boot
firstboot --disable
# System keyboard
keyboard us
# System language
lang en_US
# Use network installation
url --url=$tree
# Reboot after installation
reboot

#Root password
rootpw --iscrypted "hashed password"
# Does this still work?
selinux --disabled
# Do not configure the X Window System
skipx
# System timezone
timezone  Asia/Shanghai
# This shows how to config static ipv4 address
network --device=$mymac --onboot=yes --bootproto=static --ip=$myip --netmask=$mynm --gateway=$mygw --nameserver=$mydns --hostname=$myhn --activate

# This add a user 'foo', and set password for foo, and add foo to group 'wheel'
user --name=foo --groups='wheel' --shell=/bin/bash --iscrypted --password="hashed password"
sshkey --username=foo "ssh key string"

%pre
$SNIPPET('log_ks_pre')
$SNIPPET('autoinstall_start')
# Enable installation monitoring
$SNIPPET('pre_anamon')
%end

%packages
vim
bash-completion
%end

%post --nochroot
$SNIPPET('log_ks_post_nochroot')
%end

%post
$SNIPPET('log_ks_post')

# customization
# force user foo to change password at first login
chage -M 90 foo
chage -d 0 foo
# enable passwordless sudo for foo
echo 'foo ALL=(ALL) NOPASSWD:ALL' > /etc/sudoers.d/foo;

# Enable post-install boot notification
$SNIPPET('post_anamon')
# Start final steps
$SNIPPET('autoinstall_done')
# End final steps
%end

```

