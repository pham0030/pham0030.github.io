---
layout: post
title:  "Setup Cloudera on CENTOS 7.2"
date:   2018-10-12 21:00:00 +0800
categories: cloudera hadoop centos
---
This post presents guidelines to setup Cloudera on CENTOS 7.2

1. Setup static IP
  * Check name of the ethernet with ```ifconfig```, assume it is to be _eth0_
  
  * Edit or add the following sudo:
  
  ```vi /etc/sysconfig/network-scripts/ifcfg-eth0```
  * Add the following line with correct information from ```ifconfig```:
  
  ```DEVICE=eth0
  BOOTPROTO=static
  IPADDR=192.xxx.x.x
  NETMASK=255.255.255.0
  GATEWAY=192.xx.x.x
  HWADDR=xx:xx:xx:xx:xx
  ONBOOT=yes
  TYPE=Ethernet
  IPV6INIT=no```
