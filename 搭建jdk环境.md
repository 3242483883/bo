# **linux安装JDK教程**



###### （虚拟机、镜像自行解决。）



- ## 配置 yum 源

刚安装的 CentOS 要做的第一件事就是将 yum 源配置为阿里云的镜像，以方便后面安装软件的时候提升响应速度。通过 `curl` 下载阿里云的 yum 源配置文件：



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

```
yum install -y net-tools
```



###### 在敲 `yum makecache`的时候，发现 `tab` 键不能自动补齐命令，这是因为缺少 `bash-completion`工具，正好通过 yum 来安装它。（可以则跳过。）



```
yum -y install bash-completion
```



如果不停的在循环下载插件，可以使用ctrl+c暂停，然后用rm -f /var/run/yum.pid以下命令解决：



```
rm -f /var/run/yum.pid
```





- ## 安装jdk

  

  ###### 下载安装包，并解压到指定位置

  

```
curl -# https://download.java.net/openjdk/jdk8u41/ri/openjdk-8u41-b04-linux-x64-14_jan_2020.tar.gz >> java-se-8u41-ri.tar.gz
```

```
tar -zxf java-se-8u41-ri.tar.gz -C /opt/
```

```
vi /etc/profile
```



###### 在 `/etc/profile` 文件的最后配置 Java的环境变量，追加如下内容：



```
export JAVA_HOME=/opt/java-se-8u41-ri/
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin
```



###### 保存退出后使用 `source` 命令使 `/etc/profile` 文件生效，自此，jdk 安装完成。



```
source /etc/profile
```

```
java -version
```

