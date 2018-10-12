---
layout: post
title:  "Setup Cloudera on CENTOS 7.2"
date:   2018-10-12 21:00:00 +0800
categories: cloudera hadoop centos
---
This post presents guidelines to setup Cloudera on CENTOS 7.2

### Setup static IP
* Check name of the ethernet with __ifconfig__, assume it is to be __eth0__
* Edit or add the following sudo:
```vi /etc/sysconfig/network-scripts/ifcfg-eth0```
* Add the following line with correct information from __ifconfig__:
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
cat /etc/default/grub
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
