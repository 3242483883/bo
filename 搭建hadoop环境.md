打开虚拟机，登录主控机：

- net-tools 安装

```
yum install -y net-tools
```



如果迟迟循环安装不上，ctrl+c终止，输入命令：

```
rm -f /var/run/yum.pid  #如果可以安装，请跳过这条
```



```
ifconfig    #查看你的虚拟机ip地址，后面从物理机ssh连接它
```



- ## 配置 yum 源

```
curl https://mirrors.aliyun.com/repo/Centos-7.repo >> CentOS-Base-Aliyun.repo
```

```
mv CentOS-Base-Aliyun.repo /etc/yum.repos.d/
```

```
yum clean all
```

```
yum makecache
```



然后回到物理机，打开 Finalshell (以管理员身份打开），以ssh连接到虚拟机。

```
mkdir /etc  #在根目录的路径下创建java文件夹
```



- 安装JDK

  ```
  curl -# https://download.java.net/openjdk/jdk8u41/ri/openjdk-8u41-b04-linux-x64-14_jan_2020.tar.gz >> java-se-8u41-ri.tar.gz
  
  ```

  

- 解压JDK

```
tar -zxf java-se-8u41-ri.tar.gz -C /opt/
```



- 编辑java环境

```
vi /etc/profile
```



`/etc/profile` 文件的最后配置 Java的环境变量，追加如下内容：

```
export JAVA_HOME=/opt/java-se-8u41-ri/
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin
```



保存退出后使用 `source` 命令使 `/etc/profile` 文件生效，自此，jdk 安装完成。

```
source /etc/profile
java -version
```

成功后的样子：

```
openjdk version "1.8.0_41"
OpenJDK Runtime Environment (build 1.8.0_41-b04)
OpenJDK 64-Bit Server VM (build 25.40-b25, mixed mode)
```



- ## 安装 Hadoop （伪分布模式）

1. 下载安装包，并解压到指定位置。

   

   ```
   curl -# https://archive.apache.org/dist/hadoop/common/hadoop-2.7.1/hadoop-2.7.1.tar.gz >> hadoop-2.7.1.tar.gz
   ```

   ```
   tar -zxf hadoop-2.7.1.tar.gz -C /opt/
   ```

   下载比较慢，也可以上传安装包

   

2. 通过 vi 修改 Hadoop 下 `etc/hadoop/hadoop-env.sh` 文件，找到 `JAVA_HOME` 和 `HADOOP_CONF_DIR`。此处的 `JAVA_HOME` 的值需要和 `/etc/profile` 中配置的 `JAVA_HOME` 的值保持一致。修改后结果如下：

   

   ```
   # The only required environment variable is JAVA_HOME.  All others are
   # optional.  When running a distributed configuration it is best to
   # set JAVA_HOME in this file, so that it is correctly defined on
   # remote nodes.
   
   # The java implementation to use.
   export JAVA_HOME=/opt/java-se-8u41-ri/
   
   # The jsvc implementation to use. Jsvc is required to run secure datanodes
   # that bind to privileged ports to provide authentication of data transfer
   # protocol.  Jsvc is not required if SASL is configured for authentication of
   # data transfer protocol using non-privileged ports.
   #export JSVC_HOME=${JSVC_HOME}
   
   export HADOOP_CONF_DIR=/opt/hadoop-2.7.1/etc/hadoop
   
   ```

   配置完成之后，使用 `source` 命令使其生效：

   ```
   source /opt/hadoop-2.7.1/etc/hadoop/hadoop-env.sh
   ```



修改主机名：

```
hostnamectl set-hostname wzg #‘wzg’为你想取的名字，然后重新连接即可刷新
```

创建一个文件夹:

```
#查看当前工作目录是否是/hadoop/hadoop-2.7.1
mkdir hdfs
mkdir hdfs/name
mkdir hdfs/data
```



3.通过 vi 修改 Hadoop 下 `etc/hadoop/core-site.xml` 文件，内容如下：

‘你的主机名’ = 换成你的主机名字

```
<configuration>
    <!-- 指定 NameNode 的主机名和端口号 -->
	<property>
		<name>fs.defaultFS</name>
		<value>你的主机名://h150.c7:9000</value>
	</property>
    <!-- 
	指定 Hadoop 的临时目录
	在默认情况下，该目录会存储 NameNode，DataNode 或其他模块的数据
	-->
	<property>
		<name>hadoop.tmp.dir</name>
		<value>/opt/hadoop-2.7.1/tmp</value>
	</property>
	<!--  缓冲区大小，实际工作中根据服务器性能动态调整 -->
	<property>
		<name>io.file.buffer.size</name>
		<value>4096</value>
	</property>
	<!--
	开启hdfs的垃圾桶机制，删除掉的数据可以从垃圾桶中回收，单位分钟
	-->
	<!--
	<property>
		<name>fs.trash.interval</name>
		<value>10080</value>
	</property>
    -->
</configuration>

```



通过 vi 修改 Hadoop 下 `etc/hadoop/hdfs-site.xml` 文件，内容如下：

```
<configuration>
    <!-- 
	指定 hdfs 保存数据副本的数量（包括自己），默认值是 3。
    如果是伪分布模式，此值应该是 1 
    -->
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
	<!-- 设置HDFS的文件权限-->
	<property>
		<name>dfs.permissions</name>
		<value>true</value>
	</property>
	<!-- 设置一个文件切片的大小：128M -->
    <!--
	<property>
		<name>dfs.blocksize</name>
		<value>134217728</value>
	</property>
	-->
</configuration>

```



etc/hadoop/mapred-site.xml 文件(需要重命名)：

```
	 <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
     </property>
```



通过 vi 修改 Hadoop 下 `etc/hadoop/yarn-site.xml` 文件。

```
<configuration>
    <!-- 指定 Yarn 的 ResourceManager 的主机名 -->
	<property>
		<name>yarn.resourcemanager.hostname</name>
		<value>你的主机名</value>
	</property>
    <!-- 指定 Yarn 的 NodeManager 的获取数据的方式 -->
	<property>
		<name>yarn.nodemanager.aux-services</name>
		<value>mapreduce_shuffle</value>
	</property>
</configuration>

```



通过 vi 修改 Hadoop 下 `etc/hadoop/slaves` 文件。

```
vi /opt/hadoop-2.7.1/etc/hadoop/slaves
```



每个主机名独占一行，因为采用的是伪 分布式模式。

```
192.168.13.130
192.168.13.131
```

修改主控机的host文件，在 /etc/hosts路径下：

```
vi /etc/hosts

#格式如下：ip 主机名
# 192.168.13.132 wzg
# 192.168.13.130 slave1
# 10.70.122.105 slave2

```

让自己免密登录

```
ssh-keygen -t dsa -f ~/.ssh/id_dsa     #生成公钥
cp /root/.ssh/id_dsa.pub /root/.ssh/authorized_keys  #让自身免密登录
```



在`/etc/profile` 文件的最后配置 Hadoop 的环境变量，追加如下内容：

```
export HADOOP_HOME=/opt/hadoop-2.7.1
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```



保存退出后使用 `source` 命令使 `/etc/profile` 文件生效

```
source /etc/profile
```



1. 通过 vi 修改 Hadoop 下 `/etc/hosts` 文件，追加当前主机的IP地址映射主机名。如下所示：

   

   ```
   127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
   ::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
   
   192.168.12.150 主机名
   
   ```

   

执行如下命令对 NameNode 进行格式化

```
hdfs namenode -format
```



执行 `start-dfs.sh` 或 `start-all.sh` 脚本来启动 Hadoop 的组件。启动完成后，使用 `jps` 命令来判断是否启动成功。

```
[root@h150 ~]# start-all.sh 
This script is Deprecated. Instead use start-dfs.sh and start-yarn.sh
Starting namenodes on [h150.c7]
root@h150.c7's password: 
h150.c7: starting namenode, logging to /opt/hadoop-2.7.1/logs/hadoop-root-namenode-h150.c7.out
root@h150.c7's password: 
h150.c7: starting datanode, logging to /opt/hadoop-2.7.1/logs/hadoop-root-datanode-h150.c7.out
Starting secondary namenodes [0.0.0.0]
root@0.0.0.0's password: 
0.0.0.0: starting secondarynamenode, logging to /opt/hadoop-2.7.1/logs/hadoop-root-secondarynamenode-h150.c7.out
starting yarn daemons
starting resourcemanager, logging to /opt/hadoop-2.7.1/logs/yarn-root-resourcemanager-h150.c7.out
root@h150.c7's password: 
h150.c7: starting nodemanager, logging to /opt/hadoop-2.7.1/logs/yarn-root-nodemanager-h150.c7.out
[root@h150 ~]# jps
18208 Jps
17684 SecondaryNameNode
17829 ResourceManager
17526 DataNode
17404 NameNode
18109 NodeManager
[root@h150 ~]# 
```



- 启动Hadoop:

  ```
  bin/hdfs namenode -format   #初始化Hadoop
  sbin/./start-all.sh			#启动服务
  ```

  

- 关闭防火墙（为了能正常访问，后续可以自行配置防火墙）

  

  ```
  systemctl stop firewalld
  ```



在物理浏览器访问目标主机的8088端口验证是否打开了网页，

Hadoop 启动后，其实是可以通过真实机浏览器来访问一个 web 界面的，其端口也可能是 50070，例如访问：http://192.168.12.150:50070/，但是，因为防火墙的缘故，这里是不允许被外部访问的。可以使用 firewall-cmd 来设置需要开放的端口，具体如下所示：

```
[root@h150 ~]# firewall-cmd --add-port=50010/tcp --permanent
success
[root@h150 ~]# firewall-cmd --add-port=50020/tcp --permanent
success
[root@h150 ~]# firewall-cmd --add-port=50070/tcp --permanent
success
[root@h150 ~]# firewall-cmd --add-port=50075/tcp --permanent
success
[root@h150 ~]# firewall-cmd --add-port=50090/tcp --permanent
success
[root@h150 ~]# firewall-cmd --add-port=8030/tcp --permanent
success
[root@h150 ~]# firewall-cmd --add-port=8031/tcp --permanent
success
[root@h150 ~]# firewall-cmd --add-port=8082/tcp --permanent
success
[root@h150 ~]# firewall-cmd --add-port=8088/tcp --permanent
success
[root@h150 ~]# firewall-cmd --add-port=9000/tcp --permanent
success
[root@h150 ~]# firewall-cmd --reload
success
[root@h150 ~]# 

```



- ## 安装 Hadoop （完全分布模式）

```
在伪分布式模式的基础上，搭建 Hadoop 进行完全分布式环境。这里采用三个子节点（包括自己）。
```

1. 首先将虚拟机关机后，对齐进行快照：

2. ![](D:\运维\8.png)

   

1. 依次点击菜单栏的【虚拟机】→【管理】→【克隆】，打开克隆虚拟机向导。
2. 点击下一步后，进入到【克隆源】选择界面，这里选哪个都行。
3. 点击下一步后，进入到【克隆类型】选择界面，这里选择【创建完整克隆】
4. 点击下一步后，进入到为新虚拟机命名的界面，因为要将它作为子节点，这里我将其命名为 Slave01，如下所示：



克隆完成之后，将得到一个全新的虚拟机。因为之前有手动配置 IP，故而，这个新的主机的IP也是 192.168.12.150。为了避免启动集群的时候发生 IP 冲突，这里可以使用 sed 命令手动替换掉它的IP地址。不要将之前的虚拟机开机，把新虚拟机开机后，登录进去，使用如下命令将 IP 地址和主机名进行修改



[root@h150 ~]# sed -i s/'IPADDR="192.168.12.150"'/'IPADDR="192.168.12.151"'/g /etc/sysconfig/network-scripts/ifcfg-ens33
[root@h150 ~]# hostnamectl set-hostname h151.c7
[root@h150 ~]# hostname
h151.c7
[root@h150 ~]# reboot



使用相同的方法，将最初的虚拟机再克隆一份，命名为 Slave02，IP地址为 `192.168.12.152`，主机名为 `h152.c7`，于是就得到了三台主机名和IP地址不同但是配置相同的虚拟机。如下表所示



将三台虚拟机全部开机，使用 vi 修改的 `/etc/hosts` 文件，均如下

127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

192.168.12.150 h150.c7
192.168.12.151 h151.c7
192.168.12.152 h152.c7



1. 将三台虚拟机全部开机，为三台设备相互之间配置免密登录。



[root@h150 ~]# ssh root@h151.c7
[root@h151 ~]# rm -rf ~/.ssh/*
[root@h151 ~]# logout
Connection to h151.c7 closed.
[root@h150 ~]# ssh-copy-id root@h151.c7
[root@h150 ~]# 
[root@h150 ~]# ssh root@h152.c7
[root@h152 ~]# rm -rf ~/.ssh/*
[root@h152 ~]# logout 
Connection to h152.c7 closed.
[root@h150 ~]# ssh-copy-id root@h152.c7
[root@h150 ~]# 
[root@h150 ~]# 
[root@h150 ~]# scp /etc/hosts root@h151.c7:/etc/hosts
hosts                          100%  227   200.9KB/s   00:00 
[root@h150 ~]# scp /etc/hosts root@h152.c7:/etc/hosts
hosts                          100%  227   233.3KB/s   00:00 
[root@h150 ~]# 
[root@h150 ~]# 
[root@h150 ~]# ssh h151.c7
Last login: Tue Apr 26 02:22:18 2022 from h150.c7
[root@h151 ~]# ssh-keygen 
[root@h151 ~]# ssh-copy-id root@h150.c7
[root@h151 ~]# ssh-copy-id root@h151.c7
[root@h151 ~]# ssh-copy-id root@h152.c7
root@h151 ~]# ssh h152.c7
Last login: Tue Apr 26 02:23:44 2022 from h150.c7
[root@h152 ~]# ssh-keygen 
[root@h152 ~]# ssh-copy-id root@h150.c7
[root@h152 ~]# ssh-copy-id root@h151.c7
[root@h152 ~]# ssh-copy-id root@h152.c7
[root@h152 ~]# 
[root@h152 ~]# logout
Connection to h152.c7 closed.
[root@h151 ~]# logout
Connection to h151.c7 closed.
[root@h150 ~]# 



1. 通过 vi 修改 Hadoop 下 `etc/hadoop/slaves` 文件，内容如下：

   192.168....

   192.168...

   192.168...

2. 然后将 Hadoop 的数据、日志和临时文件全部删除后，将配置文件全部分发给子节点，重新格式化 namenode，重启 Hadoop 的所有组件。

3. [root@h150 ~]# rm -rf /opt/hadoop-2.7.1/data/
   [root@h150 ~]# rm -rf /opt/hadoop-2.7.1/logs/*
   [root@h150 ~]# rm -rf /opt/hadoop-2.7.1/tmp/*
   [root@h150 ~]# scp -r /opt/hadoop-2.7.1/ root@h151.c7:/opt/
   [root@h150 ~]# scp -r /opt/hadoop-2.7.1/ root@h152.c7:/opt/
   [root@h150 ~]# hdfs namenode -format
   [root@h150 ~]# start-all.sh

4. 最后打开浏览器，访问 50070 端口的 [datanodes](http://192.168.12.150:50070/dfshealth.html#tab-datanode) 页面，如果出现了三个节点，则说明操作成功！