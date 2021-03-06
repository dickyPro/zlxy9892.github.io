---
layout:     post
title:      "Hadoop 的安装与配置"
subtitle:   " \"Hadoop的使用初体验 (虚拟机中的伪分布模式)\""
date:       2017-01-08 22:00:00
author:     "Nova"
header-img: "img/post/post-bg-22.jpg"
catalog: true
tags:
    - 大数据
    - hadoop
    - 安装配置
---

## 1 引言

hadoop如今已经成为大数据处理中不可缺少的关键技术，在如今大数据爆炸的时代，hadoop给我们处理海量数据提供了强有力的技术支撑。因此，了解hadoop的原理与应用方法是必要的技术知识。

hadoop的基础原理可参考如下的三篇论文：

> 1. [The Google File System, 2003](https://static.googleusercontent.com/media/research.google.com/zh-CN//archive/gfs-sosp2003.pdf)
> 2. [MapReduce: Simplified Data Processing on Large Clusters, 2004](http://static.googleusercontent.com/media/research.google.com/es/us/archive/mapreduce-osdi04.pdf)
> 3. [Bigtable: A Distributed Storage System for Structured Data](https://static.googleusercontent.com/media/research.google.com/zh-CN//archive/bigtable-osdi06.pdf)

## 2 hadoop在WMware虚拟机上的配置

### 2.1 准备工作

#### 2.1.1 软件整理

首先，我们需要先确定好配置hadoop所需的所有软件工具，它们分别是：

- **VMware软件** -- ***VMware® Workstation 10.0.1**
- **Linux操作系统** -- ***CentOS-6.3-x86_64***  （下载地址：[http://ftp.stu.edu.tw/](http://ftp.stu.edu.tw/)）
- **JDK** -- ***Java SE Development Kit 7u40***  （早期jdk版本的下载地址：[http://www.oracle.com/technetwork/java/javase/archive-139210.html](http://www.oracle.com/technetwork/java/javase/archive-139210.html)）
- **Hadoop** -- ***hadoop-1.2.1_x86_64***（下载地址：[http://www-eu.apache.org/dist/hadoop/common/](http://www-eu.apache.org/dist/hadoop/common/)）
- 最好再安装上**Xmanager**，方便后续操作

#### 2.1.2 思路整理

由于hadoop的配置不像其他某些库或者包那样简单，因此，有必要在实际展开配置之前梳理一下整体hadoop的配置思路。

1. 在虚拟机中安装一台CentOS系统；
2. 主机名、ip、hosts、防火墙等基础配置；
3. 配置jdk；
4. 克隆虚拟机，得到三台相同机器，并修改各自的基础配置信息；
5. 配置hadoop；
6. 配置ssh免密码登录；
7. hadoop初始准备工作；
8. 运行hadoop

### 2.2 Hadoop的配置过程

#### 2.2.1 安装CentOS虚拟机

这里不做详细的虚拟机安装介绍，网上可查到很多相关资料。需要注意的有几点：

- 虚拟机的内存设置最好**大于512MB**，否则无法开启GUI，我这里设定的是1024MB
- 虚拟机网络模式设置为**桥接模式**，并设定主机上的 **VMnet8** 的 IP 地址与子网掩码，我这里分别设置为 **`192.168.66.1`** 和 **`255.255.255.0`**

#### 2.2.2 主机名、ip、防火墙的配置

- **主机名** 

当前的主机名可通过 `hostname` 命令进行查看，为了后续多台机器名称的统一，在此笔者将第一台主机设置为 **node1**，以后克隆的主机可依次命名为 **node2, node3, node4 ...** 主机名的配置文件位于 `/etc/sysconfig/network` 文件中，打开该文件，更改为如下内容：

	NETWORKING=yes
	HOSTNAME=node1

同时，可预先设置主机名与IP的映射关系，修改 `/etc/hosts` 文件为如下内容：

	127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
	192.168.66.100   node1
	192.168.66.101   node2
	192.168.66.102   node3
	192.168.66.103   node4
	::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

这里我们预先将其余的几个节点的ip地址设定到hosts文件中，以便后续可直接通过名称访问。

- **IP设置**

我们需要将此台机器的IP设定为静态的，且与我们最开始在主机的VMnet8上设置的IP在同一个网段内，此处笔者设置的为 IP号：**`192.168.66.100`**，子网掩码：**`255.255.255.0`**，网关：**`192.168.66.1`**（该IP为NameNode使用，后续的多个DataNode分别设置为： .101; .102; .103...）。修改方法可直接在系统可视化界面的右上角点击网络设置进行交互式设定，或直接对配置文件：`/etc/sysconfig/network-scripts/ifcfg-eth0` 进行修改，修改内容如下：

	DEVICE="eth0"
	BOOTPROTO=static
	NM_CONTROLLED="yes"
	ONBOOT="yes"
	TYPE="Ethernet"
	UUID="797cd6db-9499-4d45-85e8-b3e841bbcad5"
	IPADDR=192.168.66.100
	PREFIX=24
	GATEWAY=192.168.66.1
	DEFROUTE=yes
	IPV4_FAILURE_FATAL=yes
	IPV6INIT=no
	NAME="System eth0"
	HWADDR=00:0C:29:44:24:87
	LAST_CONNECT=1500978980

- **防火墙**

为了今后各节点之间通信不会受到防火墙的限制，暂且先将防火墙关闭，相关命令如下：

	service iptables status	# 查看当前防火墙的状态
	service iptables stop	# 关闭防火墙
	service iptables start	# 开启防火墙

最后可测试一下从Windows主机上是否能够与Linux虚拟机通信,可直接在主机的控制台输入 `ping 192.168.66.100`，查看结果。


#### 2.2.3 配置JDK

关于jdk的安装配置网上教程很多，这里简单做一下介绍，笔者将下载后的 *jdk-7u40-linux-x64.tar.gz* 文件解压，并放置在/opt/dev/java/目录下(*笔者的开发包目录统一设定为 `/opt/dev/` 目录下).

    // 解压与移动代码：
    tar -zxvf 文件名
    mv 解压后的文件名 jdk
    mv jdk /opt/dev/java/

 接下来进行java环境变量的配置，打开 `/etc/profile` 配置文件，该文件是专门用来管理系统环境变量，配置之后对所有的用户均有效。

	// 在文件末尾追加以下内容：
	#set java environment
	export JAVA_HOME=/opt/dev/java/jdk
	export CLASSPATH=".:$JAVA_HOME/lib:$CLASSPATH"
	export PATH="$JAVA_HOME/bin:$PATH"

保存退出之后，可用 `source /etc/profile` 来重新加载配置文件，但由于该种操作存在弊端，因此可以通过 `logout` 命令重新登录较为稳妥。

最后可在控制台中输入 `java -version` 来检查是否配置成功。配置成功后可看到如下信息：

	java version "1.7.0_40"
	Java(TM) SE Runtime Environment (build 1.7.0_40-b43)
	Java HotSpot(TM) 64-Bit Server VM (build 24.0-b56, mixed mode)

#### 2.2.4 克隆多台主机

该步骤较为简单，直接通过VMware自带的克隆功能进行克隆即可（需选择完全克隆）。此处笔者克隆了另外两台，分别设置名为 node2 和 node3，并设定了相应的IP地址。

#### 2.2.5 配置hadoop

到了最关键的hadoop配置，首先将下载的 *hadoop-1.2.1-bin.tar.gz* 文件进行解压，同样放置在 `/opt/dev/hadoop` 目录下。接下来开始分别对几个关键的配置文件进行修改。

- **配置hadoop-env.sh**

打开hadoop-env.sh文件，找到JAVA\_HOME关键字所在的行，去掉最前面的#号，然后修改成本机的JAVA\_HOME地址：

	export JAVA_HOME=/opt/dev/java/jdk

- **配置core-site.xml**

打开hadoop目录中的conf文件夹，打开其中的core-site.xml文件，在其中的configuration标签中加入以下内容：

```xml
<!-— fs.default.name：用来配置namenode,指定HDFS文件系统的URL，通过该URL我们可以访问文件系统的内容，也可以把localhost换成本机IP地址；如果是完全分布模式，则必须把localhost改为实际namenode机器的IP地址；如果不写端口，则使用默认端口8020。 -->
<property>
	<name>fs.default.name</name> 
	<value>hdfs://node1:9000</value>
</propety>

<!-- hadoop.tmp.dir：Hadoop的默认临时路径，这个最好需要配置一下，如果在新增节点或者其他情况下莫名其妙的DataNode启动不了，就删除此文件中的tmp目录即可。不过如果删除了NameNode机器的此目录，那么就需要重新执行NameNode格式化的命令。该目录必须预先手工创建。-->
<property>
	<name>hadoop.tmp.dir</name>
	<value>/opt/data/hadoop-1.2.1/</value>
</property>
```

- **配置hdfs-site.xml**

打开hdfs-site.xml文件，修改内容如下：

```xml
<!—用来设置文件系统冗余备份数量，因为只有2个节点，所以设置为2，系统默认数量为3-->
<configuration>
	<property>
		<name>dfs.replication</name>
		<value>2</value>
	</property>  
</configuration>
```

- **配置mapred-site.xml**

在该文件中的修改内容如下：

```xml
<configuration>
	<property>
		<name>mapred.job.tracker</name>
		<value>localhost:9001</value>
	</property>
</configuration>
```

#### 2.2.6 配置SSH免密码登录

##### 2.2.6.1 SSH的相关概念

首先需要介绍一下SSH是什么，很多人在初次接触hadoop时，只是简单的按照说明进行了一步步的模仿，但并不懂得其本质的道理，因此造成很多人在配置过程中出现错误，无法继续前进，所以有必要先对SSH的基本概念做一些阐述，在理解的基础上进行配置。

我们知道，在同一个局域网内（相同的网关），两台机器可以相互访问，但是我们通常需要输入用户名和密码才能登陆从A机器登陆到另B机器上，此时，我们A机器的使用者需要知晓B机器的用户名和密码。那么SSH的作用就在于，当A想登陆到B时，如何设定一种协议，使得其无需提供密码就能完成登陆。

SSH（Security Shell）免密码登陆的设计思想是：在A机器上生成一个公钥（在id\_dsa.pub文件中）如果B机器也拥有这个公钥（放置在authorized\_keys文件中）,那么就可以认为A登陆到B是安全的，或者说B机器许可了A机器来登陆。

##### 2.2.6.2 Hadoop中的SSH免密码登陆设置

由于hadoop整个集群系统至少需要NameNode节点可以免密码登陆到所有其他DataNode节点，因此，结合上述概念，我们可以知道，这里我们需要完成的主要配置任务包含如下两点：

1. 让每台机器生成自己的公钥（id\_dsa.pub文件），并实现本地可免密码登陆，即每台机器自己的 authorized\_keys 文件中包含自己的公钥。
2. 让每个DataNode节点拥有NameNode节点的公钥，集每个DataNode的 authorized\_keys 文件中还包含了NameNode的公钥。

具体的配置操作如下：

本地完成SSH免密码登陆的配置:

- 生成公钥：

  `ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa`

- 将公匙添加到authorized_keys文件中：

  `cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys`

此时可测试本地是否可以免密码登陆：

```
ssh localhost	#ssh登陆，若不询问输入密码，则配置成功
exit			#退出ssh登陆
```

同理，将公匙复制到 node2 和 node3 的 authorized_keys 文件中，就可以让 node1 免密码登录到 node2 和 node3:

- 在node1上输入远程复制命令：

  `scp ~/.ssh/id_dsa.pub root@node2:~/`
  `scp ~/.ssh/id_dsa.pub root@node3:~/`

- 分别到 node2 和 node3 机器上，添加 node1 的公匙信息到 authorized_keys 文件中：

  `cat ~/id_dsa.pub >> ~/.ssh/authorized_keys`

此时，可到node1上进行登陆测试。

同理，在node2生成公匙，然后复制到node1和node3的authorized\_keys文件中，这样node2就可以无密码登录node1和node3, node3也可做相同的操作。

以上就是三台虚拟机的SSH免密码登录配置方法，当然，可以根据实际的使用情况来设置，hadoop并不一定非要三台都支持双向的免密码登录。

#### 2.2.7 启动hadoop

最后就是hadoop的启动环节了，具体操作可按如下顺序进行。

##### 2.2.7.1 配置hadoop环境变量

为方便今后的使用，可在环境变量中加入hadoop的bin目录，打开 `/etc/profile` 文件，添加如下内容：

	# set hadoop environment
	export HADOOP_INSTALL=/opt/dev/hadoop/hadoop-1.2.1
	export PATH=${HADOOP_INSTALL}/bin:$PATH

配置完成后，使用logout登出，再重新登录，输入 `hadoop version`，若配置成功，可查看到如下内容：

	[root@node1 ~]# hadoop version
	Hadoop 1.2.1
	Subversion https://svn.apache.org/repos/asf/hadoop/common/branches/branch-1.2 -r 1503152
	Compiled by mattf on Mon Jul 22 15:23:09 PDT 2013
	From source with checksum 6923c86528809c4e7e6f493b6b413a9a
	This command was run using /opt/dev/hadoop/hadoop-1.2.1/hadoop-core-1.2.1.jar

##### 2.2.7.2 格式化NameNode

输入如下命令即可：

```shell
hadoop namenode –format
```

##### 2.2.7.3 启动hadoop进程

输入 `start-all.sh`，若只想先启动分布式文件系统，可输入 `start-dfs.sh`。

##### 2.2.7.4 查看相关信息

我们可以从Windows主机或者虚拟机中的浏览器访问hadoop。

首先需要设置Windows主机上的host文件，目录位置： `C:\Windows\System32\drivers\etc\hosts`，添加虚拟机的三个节点的主机名与IP的映射关系：

	#hadoop-cluster
	192.168.66.100	node1
	192.168.66.101	node2
	192.168.66.102	node3

- 查看NameNode提供的DFS信息：

  [http://node1:50070](http://node1:50070 "http://node1:50070")

- 查看jobtracker信息：

  [http://node1:50030](http://node1:50030 "http://node1:50030")

----------

**----- 至此，Hadoop的基本配置就已基本完成。-----**

