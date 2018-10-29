---
layout: post
title:  "Setup Ambari and Hadoop Hortonworks on CENTOS 7.3"
date:   2018-10-29 21:00:00 +0800
categories: ambari hadoop hortonworks centos
---
This post presents complementary guidelines to setup Ambari 2.7.1 and Hadoop Hortonworks on CENTOS 7.3 to the steps shown in the [official documents](https://docs.hortonworks.com/HDPDocuments/Ambari-2.7.1.0/bk_ambari-installation/content/ch_Getting_Ready.html)

### Prerequisite
Run the following commands line by line to prepare the prerequisite environment
```console
sudo yum update -y
echo '*             hard    nofile           65535' | sudo tee --append /etc/security/limits.conf
sudo yum install -y yum-utils device-mapper-persistent-data lvm2 scp curl unzip tar wget vim ntp
sudo umask 0022 && echo 'umask 0022' | sudo tee --append /etc/profile
```

### Disable firewall
```console
systemctl disable firewalld
systemctl stop firewalld
```

### Setting Network Time Protocol, NTP
NTP clock sync problem can be solved by commands below:
```console
systemctl start ntpd
systemctl enable ntpd
systemctl status ntpd
```

### Disabe SELinux
Add or edit the line ```SELINUX=disabled``` in file ```/etc/selinux/config```.

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
### Setting up full hostname
* Run the commands below to set hostname with approriate ip, static IP is essential:
```console
sudo hostnamectl set-hostname master.hadoop
echo '192.168.1.106 master.hadoop' | sudo tee --append /etc/hosts
```
* Add the following lines to ```/etc/sysconfig/network```
```console
NETWORKING=yes
HOSTNAME=master.hadoop
```
* Run the folowing commands to check name consistency
```console
hostname -f
hostname
uname -a
```

### Setup passwordless ssh
This step is important. Make sure every hosts can ssh passwordless to root of other hosts and its own by following the below steps.
* Generate public and private key pairs.
```
ssh-keygen
ssh-copy-id root@master.hadoop
```
* Make sure that ssh passwordless is enabled by testing.
```
ssh root@master.hadoop
```

### Install ambari server and JDBC
```console
sudo wget -nv http://public-repo-1.hortonworks.com/ambari/centos7/2.x/updates/2.7.1.0/ambari.repo -O /etc/yum.repos.d/ambari.repo
sudo yum install ambari-server mysql-connector-java* -y
sudo ln -s /usr/share/java/mysql-connector-java.jar /var/lib/ambari-server/resources/mysql-connector-java.jar
```

### Official guide
Follow the official guides for the remaining steps [here](https://docs.hortonworks.com/HDPDocuments/Ambari/Ambari-2.7.1.0/index.html) and [here](https://docs.hortonworks.com/HDPDocuments/Ambari-2.7.1.0/bk_ambari-installation/content/ch_Deploy_and_Configure_a_HDP_Cluster.html)

### Troubleshooting
* During installation, there might be some issues with permissions and owners. One way to fix this is via commands:
```
chmod -R 755 /var/lib/zookeeper
chown -R zookeeper:zookeeper /var/lib/zookeeper
```
