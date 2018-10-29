---
layout: post
title:  "Setup Cloudera on CENTOS 7"
date:   2018-10-12 21:00:00 +0800
categories: cloudera hadoop centos
---
This post presents complementary guidelines to setup Cloudera on CENTOS 7. It is recommended to install Cloudera 5.15.1 on CENTOS 7.3 at the time of this guide for incompatibility free. The steps shown in the official Cloudera is not sufficient, for example the OS SELinux can be enabled and block the installation process. 

### Setup static IP
* Check name of the ethernet with __ifconfig__, assume it is to be __eth0__
* Edit the file ```/etc/sysconfig/network-scripts/ifcfg-eth0``` with the following lines and corresponding information from __ifconfig__:
```console
DEVICE=eth0
BOOTPROTO=static
IPADDR=192.xxx.x.x
NETMASK=255.255.255.0
GATEWAY=192.xx.x.x
HWADDR=xx:xx:xx:xx:xx
ONBOOT=yes
TYPE=Ethernet
IPV6INIT=no
```

### Disable ipv6
* To verify if IPv6 is enabled or not, execute ```ifconfig -a | grep inet6```, it should display nothing if the ipv6 is disabled.
* Edit ```/etc/default/grub``` and add ```ipv6.disable=1``` in line GRUB_CMDLINE_LINUX, e.g.:
```console
GRUB_TIMEOUT=5
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="ipv6.disable=1 crashkernel=auto rhgb quiet"
GRUB_DISABLE_RECOVERY="true"
```
* Regenerate a GRUB configuration file and overwrite existing one:
```console
grub2-mkconfig -o /boot/grub2/grub.cfg
```
* Reboot

### Setting up DNS Server
By setting up a static ip, dns servers might not be automatically determined. Resolve this issue by editing the file ```/etc/resolv.conf``` with specific information such as:
```nameserver 8.8.8.8```

### Disabe SELinux
Add or edit the line ```SELINUX=disabled``` in file ```/etc/selinux/config```.

### Setting Network Time Protocol, NTP
NTP clock sync problem can be solved by commands below:
```console
sudo yum instal ntp
firewall-cmd --add-service=ntp --permanent
firewall-cmd --reload
systemctl start ntpd
systemctl enable ntpd
systemctl status ntpd
```

### Disable Transparent Hugepages
THP need to be disabled with root account:
```console
echo never > /sys/kernel/mm/redhat_transparent_hugepage/enabled
echo never > /sys/kernel/mm/redhat_transparent_hugepage/defrag
```

### Change the swappiness
CentOS swappiness should be set to below 10.
* To check the current swappiness:
```
cat /proc/sys/vm/swappiness
```
* Permanently set the level of swappiness by add the below line to ```/etc/sysctl.conf```
```
vm.swappiness = 10
```

### Following steps shown in the official website
[Cloudera Installation 5.15](https://www.cloudera.com/documentation/enterprise/5-15-x/topics/installation.html)

### Troubleshooting
* During installation, there might be some issues with permission. One way to fix this is by changing permission level for zookeeper/yarn/spark/oozie service via:
```
chmod -R 755 /var/lib/zookeeper
chown -R zookeeper:zookeeper /var/lib/zookeeper
```
