---
layout: post
title:  "Setup Hadoop Raspberry"
date:   2018-09-29 17:23:12 +0800
categories: hadoop raspberry
---
These are guidelines to setup Hadoop cluster on Raspberry Pi computer.

* We will start our installation in the namenode. Download and install Raspbian Jessy Lite as our OS. At the writing of this guide 2018-09 it is at ```4.9.35-v7+```.

* It is recommended to create a dedicated hadoop user and group accounts to seperate the Hadoop installation from other services. Add the ```hduser``` to ```sudo``` and ```hadoop``` groups.

```console
sudo addgroup hadoop
sudo adduser hduser hadoop
usermod -aG sudo hduser
```

* Create the home installation directory for Hadoop at ```/opt/hadoop/``` and the directory where HDFS (Hadoop Distributed File Systems) will format and place its files at ```/opt/hadoop_tmp/```.

```console
sudo mkdir /opt/hadoop/
sudo mkdir /opt/hadoop_tmp/
```

* Change the permission of the hadoop installation folder and HDFS folder to ```hduser```. Missing this step will prevent the hadoop to start properly.

```console
sudo chown hduser:hadoop /opt/hadoop
sudo chmod 750 /opt/hadoop
sudo chown hduser:hadoop /opt/hadoop_tmp
sudo chmod 750 /opt/hadoop_tmp
```

* It is recommended to install Oracle Java 8. Via ```apt-get``` or manually from [Oracle](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html).

```console
sudo apt-get update
sudo apt-get install oracle-java8-jdk
```

* Setting up ```$JAVA_HOME``` is critical.

```console
echo 'export JAVA_HOME=$(readlink -f /usr/bin/java | sed "s:bin/java::")' >> ~/.bashrc
```

* Verify java installation.

```console
java -version
javac -version
```

* Add environment variables for Hadoop in file ```~/.bashrc```

```console
vi ~/.bashrc
```
* Add the following lines to the bottom of ```~/.bashrc```. If you installed Hadoop in a different path, make sure to change ```HADOOP_HOME```.

```console
export HADOOP_HOME=/opt/hadoop
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin 
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib/native"
```
* The namenode is our master node. The slave nodes are refered as datanodes. We will install Hadoop on our namenode and perform some configurations, then, we will work on the datanodes. Since this is the basic configuration, this and the following steps will be done only in the namenode. Then we will copy this basic configuration to the remaining nodes, and then we will perform specific customizations. In the releases page [https://hadoop.apache.org/releases.html](https://hadoop.apache.org/releases.html), select the version of hadoop you want to install. At the time of this guide, `hadoop-2.6.5` was used. It is recommended to compile the binary file following the instructions from apache to get the best performance and secure installation of hadoop. The following commands will download a Hadoop package and uncompress it.

```console
cd /opt/hadoop/
sudo wget http://www-us.apache.org/dist/hadoop/common/hadoop-2.6.5/hadoop-2.6.5.tar.gz
sudo tar xvf hadoop-2.6.5.tar.gz
mv hadoop-2.6.5 hadoop
```

* Setting up connectivity between the Raspberry pis. All the Raspberry Pis should be configured with *static IP* in the network. Follow this [guide](/raspberry/config/2018/09/28/configure-raspberry-network.html) to setup the *static IP* for your Pis. To make our job easier, we will use an alias for each node from now on. Make a list of all the IP addressess and add them to the ```/etc/hosts``` file in every Raspberry. In our case of 5 nodes, it looks like this. These setting should be copied to all of the nodes in the clusters.

```console
9.42.157.175    master
9.42.157.182    slave-01
9.42.157.186    slave-02
9.42.157.187    slave-03
9.42.157.194    slave-04
```

* We need all the Raspberies to be able to comunicate between each other without prompting for passwords. For that we will generate SSH keys in all of them and replicate the keys to all the Raspberies.

```console
ssh-keygen -t rsa -b 4096 -C your_email@example.com
ssh-copy-id hduser@master
ssh-copy-id hduser@slave-01
ssh-copy-id hduser@slave-02
ssh-copy-id hduser@slave-03
ssh-copy-id hduser@slave-04
```
* Make sure to repeat the ```ssh-keygen``` and ```sss-copy-id``` on all the nodes so that they can communicate back and forth. Take your time to verify that all the nodes can connect to each other via ```ssh```. Your public key will be at ```~/.ssh/id_rsa.pub``` this will be copy over to other nodes at ```~/.ssh/authorized_keys```. Sometime you will encounter the warning about ```ECDSA host key```, clean the cached key for the nodes on the local machine with the following.

```console
ssh-keygen -R [IP of the node]
```

* Add the namenode (master) and datanode (slaves) configuration. This step should only be done in the namenode and will be copied over to other nodes with ```rsync```. In the ```$HADOOP_HOME/hadoop/etc/hadoop/master``` add the hostname or alias of the namenode. So in our case this fill will contain one line.

```console
master
```

* In the ```$HADOOP_HOME/hadoop/etc/hadoop/slaves``` add our datanode (slaves), one alias per line.

```console
slave-01
slave-02
slave-03
slave-04
```

* A couple of directories need to be created in our HDFS directory. In the namenode. go to the HDFS directory ```/opt/hadoop_tmp/ in our case``` and create a directory named *hdfs*, then, inside that directory create another directory named *namenode*.

```console
sudo mkdir /opt/hadoop_tmp/hdfs
sudo mkdir /opt/hadoop_tmp/hdfs/namenode
```

* Next, we will go to every slave node, and do the same, except that the newest directory will be named *datanode*:

```console
sudo mkdir /opt/hadoop_tmp/hdfs
sudo mkdir /opt/hadoop_tmp/hdfs/datanode
```

* Edit the configuration files in ```$HADOOP_HOME/hadoop/etc/hadoop/```. There are *core-site.xml*, *hdfs-site.xml*, *yarn-site.xml* and *mapred-site.template.xml* needed to be configured.

* *core-site.xml*.

```console
<configuration>
    <property>
        <name>fs.default.name</name>
        <value>hdfs://master:9000/</value>
    </property>
    <property>
        <name>fs.default.FS</name>
        <value>hdfs://master:9000/</value>
    </property>
</configuration>
```

* *hdfs-site.xml*.

```console
<configuration>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>/opt/hadoop_tmp/hdfs/datanode</value>
        <final>true</final>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>/opt/hadoop_tmp/hdfs/namenode</value>
        <final>true</final>
    </property>
    <property>
        <name>dfs.namenode.http-address</name>
        <value>master:50070</value>
    </property>
    <property>
        <name>dfs.replication</name>
        <value>11</value>
    </property>
</configuration>
```

* *yarn-site.xml*.

```console
<configuration>
    <property>
        <name>yarn.resourcemanager.resource-tracker.address</name>
        <value>master:8025</value>
    </property>
    <property>
        <name>yarn.resourcemanager.scheduler.address</name>
        <value>master:8035</value>
    </property>
    <property>
        <name>yarn.resourcemanager.address</name>
        <value>master:8050</value>
    </property>
</configuration>
```

* *mapred-site.template.xml*.

```console
<configuration>
    <property>
        <name>mapreduce.job.tracker</name>
        <value>master:5431</value>
    </property>
    <property>
        <name>mapred.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```

* Check if the ports are in use with one of the following commands.
If it is used by other programs, you need to reconfigure to different ports.

```console
sudo lsof -i -P -n | grep LISTEN
netstat -tulpn | grep LISTEN
sudo nmap -sT -O localhost
nc -vz {host} {port}
telnet {host} {post}
```

* The previous steps for the Hadoop installation wer done only in the master node. Now we will take those files and copy them to the remaining Raspberries. On the master node, run the following commands in order to copy the basic setup to the slaves (you can also do this with scp).

```console
rsync -avxP <$HADOOP_HOME> <hadoop_user>@<slave-hostname>:<$HADOOP_HOME>
```

In our case it would be
```console
rsync -avxP /opt/hadoop/ hduser@slave-01:/opt/hadoop/
rsync -avxP /opt/hadoop/ hduser@slave-02:/opt/hadoop/
rsync -avxP /opt/hadoop/ hduser@slave-03:/opt/hadoop/
rsync -avxP /opt/hadoop/ hduser@slave-04:/opt/hadoop/
rsync -avxP /opt/hadoop/ hduser@slave-05:/opt/hadoop/
```

* Format the namenode and start the service. Go to the namenode in $HADOOP_HOME/bin/ execute.

```console
./hdfs namenode -format
```

* That will format the master as a proper namenode. Finally, start the services. Previously, Hadoop used the start-all.sh script for this, but it has been deprecated and the recommended method now is to use the start-<service>.sh scripts individually. In each node, go to $HADOOP_HOME/sbin/ and execute the following:

```console
./start-yarn.sh
./start-dfs.sh
```

* Verify the clusters with this command. This should shown number of Live datanodes and their status.

```console
/opt/hadoop/hadoop/bin/hdfs dfsadmin -report
```
