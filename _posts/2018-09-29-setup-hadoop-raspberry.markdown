---
layout: post
title:  "Setup Hadoop Raspberry"
date:   2018-09-29 17:23:12 +0800
categories: hadoop raspberry
---
These are guidelines to setup Hadoop cluster on Raspberry Pi computer.

1. We will start our installation in the namenode. Download and install Raspbian Jessy Lite as our OS. At the writing of this guide 2018-09 it is at ```4.9.35-v7+```.
2. Create the home installation directory for Hadoop at ```/opt/hadoop/``` and the directory where HDFS (Hadoop Distributed File Systems) will format and place its files at ```/opt/hadoop_tmp/```.
```console
sudo mkdir /opt/hadoop/
sudo mkdir /opt/hadoop_tmp/
```
