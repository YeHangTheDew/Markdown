## Centos8安装Hadoop3.1.2+hive步骤

## 一、主机

采用阿里云ECS主机，五台主机的组成内网。集群内部通信全部用内网地址，主机名称如下。各节点按照相应名称部署，除此之外把secondnamenode部署在NodeManager-DataNode-1，

把hive运行所需的mysql安装在NodeManager-DataNode-2

NodeManager-DataNode-2
 121.40.185.152
 私有ip 192.168.0.37
 NodeManager-DataNode-1
 116.62.161.234
 私有ip 192.168.0.38
 NodeManager-DataNode-Hive
 121.41.10.75
 私有ip 192.168.0.36
 ResourceManager
 47.96.105.93
 私有ip 192.168.0.35
 NameNode
 112.124.30.113

私有ip 192.168.0.34

## 二、Hadoop

#### **1.** **安装jdk8（五台）**

可以用满彬大佬的jdk安装脚本，方便部署其他应用

直接找满彬，获取到了部署所需的服务和组件，安装jdk直接按照脚本执行即可

![img](file:///C:/Users/yechh/AppData/Local/Temp/msohtmlclip1/01/clip_image002.jpg)

把这些安装包，可以统一放到一个目录：如：/home/soft 

安装JDK 

 sh JDK_instanller.sh 1.8.0_181

#### **2.** **配置jdk环境变量（五台）**

.    

vim /etc/profile

 

export JAVA_HOME=/usr/java/jdk1.8.0_181-amd64

export PATH=$PATH:$JAVA_HOME/bin

 

 

.    

**使得配置生效**

```
. Source /etc/profile
```

**查看变量**

```
echo $JAVA_HOME
```

输出 /usr/java/jdk1.8.0_181-amd64

#### **3.****修改hosts文件**

vim /etc/hosts

添加

192.168.0.34  NameNode

192.168.0.37  NodeManager-DataNode-2

192.168.0.35  ResourceManager

192.168.0.36  NodeManager-DataNode-Hive

192.168.0.38  NodeManager-DataNode-1

**注意要添加在阿里云自带的本机主机名前面**例如

192.168.0.34  iZbp1eipwd1cp06xpux92hZ iZbp1eipwd1cp06xpux92hZ

是NameNode的自带的主机名，把上面主机映射放于前面，不然hadoop可能找不到我们配置的主机名。

 

#### **4.** **安装Hadoop**

 

 

创建hadoop用户（五台），会在/home下生成hadoop文件夹，之后hadoop就安装在这里面。

在root用户下vim /etc/sudoers

添加

hadoop ALL=(ALL)    NOPASSWD: ALL

（给hadoop用户赋予sudo权限，之后如果在hadoop用户下一些命令权限不够可以加sudo，所有操作尽量都在hadoop用户下进行）

 

su hadoop

切换hadoop用户（现在开始只在NameNode节点下操作）

**解压：**

把下载好的hadoop-3.1.2.tar.gz上传到NameNode主机的/tmp/，也可以在服务器上直接用wget下载

解压

tar -xzvf /tmp/hadoop-3.1.2.tar.gz

mv hadoop-3.1.2/ /home/hadoop/hadoop3.1.2/hadoop-3.1.2

 

 

```
**在以下三个文件里面**
**etc/hadoop/hadoop-env.sh**
**etc/hadoop/yarn-env.sh**
**etc/hadoop/mapred-env.sh**
 ``**添加JAVA_HOME**
```

.    

.  export JAVA_HOME=/usr/java/jdk1.8.0_181-amd64

.    

**修改etc/hadoop/core-site.xml，把配置改成：**

.  

.  <configuration>

.   <!--指定hdfs中namenode的地址 -->

.      <property>

.          <name>fs.defaultFS</name>

.  

.          <value>hdfs://NameNode:9000</value>

.      </property>

.          <!--指定hadoop运行时产生文件的存储目录 -->

.      <property>

.          <name>hadoop.tmp.dir</name>

.       <value>/home/hadoop/hadoop3.1.2/hadoop-3.1.2/data/tmp</value>

.      </property>

.  

.   <!--hadoop用户代理，hive远程连接用-->

.      <property>

.           <name>hadoop.proxyuser.hadoop.hosts</name>

.           <value>*</value>

.      </property>

.      <property>

.           <name>hadoop.proxyuser.hadoop.groups</name>

.           <value>hadoop</value>

.      </property>

.  

.  

.  </configuration>

.    

**修改etc/hadoop/hdfs-site.xml，把配置改成：**

.    

.  <configuration>

.  <!--指定hdfs中namenode的地址 -->

.  <property>

.     <name>fs.defaultFS</name>

<value>hdfs://NameNode:9000</value>

.  </property>

.  <!--指定hadoop运行时产生文件的存储目录 -->

.  <property>

.      <name>hadoop.tmp.dir</name>

.  <value>/home/hadoop/hadoop3.1.2/hadoop-3.1.2/data/tmp</value>

.  </property>

.  

.  </configuration>

.    

**etc/hadoop/yarn-site.xml****，把配置改成：**

.    

<configuration>

 

.  <!-- Site specific YARN configuration properties -->

.  

.  

.  

.  <!--指定reducer获取数据的方式 -->

.  <property>

.      <name>yarn,nodemanager.aux-services</name>

.      <value>mapreduce_shuffle</value>

.  </property>

.  

.  

.  

.  <!--指定yarn的resourceManager的地址 

.  -->

.  <property>

.      <name>yarn.resourcemanager.hostname</name>

.      <value>ResourceManager</value>

.  </property>

*.*  

.  <!--日志聚集功能使能 -->

.  <property>

.      <name>yarn.log-aggregation-enable</name>

.      <value>true</value>

.  </property>

.  <!--日志保留时间设置为7天 -->

.  <property>

.      <name>yarn.log-aggregation.retain-seconds</name>

.      <value>604800</value>

.  </property>

.  

.  <!--设置虚拟内存大小-->

.  <property>

.    <name>yarn.nodemanager.resource.memory-mb</name>

.    <value>20480</value>

.  </property>

.  <property>

.        <name>yarn.scheduler.minimum-allocation-mb</name>

.         <value>2048</value>

.  </property>

.   <property>

.        <name>yarn.nodemanager.vmem-pmem-ratio</name>

.        <value>2.1</value>

.   </property>

.  <property>

.  <name>yarn.nodemanager.disk-health-checker.max-disk-utilization-per-disk-percentage</name>

.   <value>100.0</value>

.  </property>

.  

.  </configuration>

.    

**etc/hadoop/mapred-site.xml****，内容改为如下：**

.    

.  <configuration>

.  

.   <!-- 指定mapreduce运行在yarn上-->

.    <property>

.      <name>mapreduce.framework.name</name>

.      <value>yarn</value>

.  </property>

.  <property>

.                    

.      <name>yarn.nodemanager.aux-services</name>     

<value>mapreduce_shuffle</value>

.  </property>

 

.  <!--一些其他配置，解决resourcemanager的报错-->

.  <property>

.       <name>yarn.app.mapreduce.am.env</name>

.        <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>

.  </property>

.    <property>

.         <name>mapreduce.map.env</name>

​        <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>

.    </property>

.  <property>

.           <name>mapreduce.reduce.env</name>

.  <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>

.  </property>

.  <!--历史服务器端地址 -->

.  <property>

.      <name>mapreduce.jobhistory.address</name>

.      <value>NameNode:10020</value>

.  </property>

.  

.      <!-- 历史服务器web端地址-->

.  <property>

.     <name>mapreduce.jobhistory.webapp.address</name>

.      <value>NameNode:19888</value>

.  </property>

.  

.  <property>            

<name>mapreduce.jobhistory.done-dir</name>

.     <value>/history/done</value>

.  </property>

.  <property>

​      <name>mapreudce.jobhistory.intermediate.done-dir</name>

.    <value>/history/done/done_intermediate</value>

   </property>

.  </configuration>

 

**etc/hadoop/hdfs-site.xml****，内容改为如下，dfs.namenode.secondary.http-address指定了2nn所在的主机**

.  <configuration>

.  <property>

.     <name>dfs.replication</name>

.     <value>3</value>

.  </property>

.  

.  <property>

.     <name>dfs.namenode.secondary.http-address</name>

.     <value>NodeManager-DataNode-1:50090</value>

.  </property>

.  </configuration>

 

**修改etc/hadoop/workers**

.    

vim etc/hadoop/workers

.    

 

NodeManager-DataNode-2

NodeManager-DataNode-Hive

NodeManager-DataNode-1

 

 

 

**压缩配置好的hadoop文件夹**

```
tar -czvf hadoop.tar.gz /home/hadoop/hadoop3.1.2/hadoop-3.1.2/
```

**拷贝到其余节点：**

scp hadoop.tar.gz [hadoop@192.168.0.35:/](mailto:hadoop@192.168.0.35:/)

scp hadoop.tar.gz hadoop@192.168.0.36:/

scp hadoop.tar.gz hadoop@192.168.0.37:/

scp hadoop.tar.gz hadoop@192.168.0.38:/

```
 
```

**解压删除：**

解压

tar -xzvf /hadoop.tar.gz

mv hadoop-3.1.2/ /home/hadoop/hadoop3.1.2/hadoop-3.1.2

删除

rm –rf hadoop.tar.gz

#### **5.****配置Hadoop环境变量（五台）**

.    

vim /etc/profile

 

\#HADOOP_HOME

export HADOOP_HOME=/home/hadoop/hadoop3.1.2/hadoop-3.1.2

export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin

 

 

 

source /etc/profile

 

 

**只在****NameNode****上添加的环境变量**

 

\#用数据平台的datax同步数据操作hdfs时候，指定以hadoop用户执行

export HADOOP_USER_NAME=hadoop

 

 

**免密码登录自身（五台）**

**为了一劳永逸直接配置所有机器互相以及自身ssh免密登录。**

**格式化HDFS [只有首次部署才可使用]【****谨慎操作，只在NameNode上操作****】**

 

```
/home/hadoop/hadoop3.1.2/hadoop-3.1.2/bin/hdfs namenode -format 
```

开启hdfs，开启日志服务器，在NameNode节点

/home/hadoop/hadoop3.1.2/hadoop-3.1.2/sbin/start-dfs.sh

/home/hadoop/hadoop3.1.2/hadoop-3.1.2/mr-jobhistory-daemon.sh start historyserver

 

开启yarn，在ResourceManager节点上

`/home/hadoop/hadoop3.1.2/`ha`doop-3.1.2/sbin/start-yarn.`sh

 

如果成功的话会在执行jps，各个相应的节点会看到运行的实例

![img](file:///C:/Users/yechh/AppData/Local/Temp/msohtmlclip1/01/clip_image003.png)

![img](file:///C:/Users/yechh/AppData/Local/Temp/msohtmlclip1/01/clip_image004.png)

![img](file:///C:/Users/yechh/AppData/Local/Temp/msohtmlclip1/01/clip_image005.png)

![img](file:///C:/Users/yechh/AppData/Local/Temp/msohtmlclip1/01/clip_image006.png)

![img](file:///C:/Users/yechh/AppData/Local/Temp/msohtmlclip1/01/clip_image007.png)

 

 

**web****地址，从外网访问注意在阿里云控制台开启相应的端口**

 

[http://112.124.30.113:19888](http://112.124.30.113:19888/)  jobhistory

[http://112.124.30.113:9870](http://112.124.30.113:9870/)  hdfs

[http://47.96.105.93:8088](http://47.96.105.93:8088/)   resourcemanager

 

 

**分发脚本：**

**新建一个文件xsync.sh输入**

\#!/bin/bash

 

\#1 获取输入参数个数，如果没有参数，直接退出

 

pcount=$#

if ((pcount==0));then

​     echo no args;

​     exit;

 fi

 

 

 targethosts="

 NodeManager-DataNode-2

 NodeManager-DataNode-1

 NodeManager-DataNode-Hive

 ResourceManager

 "

 

 

 \#2 获取文件名称

 p1=$1

 fname=`basename $p1`

 echo fname=$fname

 

 \#3 获取上级目录到绝对路径

 pdir=`cd -P $(dirname $p1); pwd`

 echo pdir=$pdir

 

 

 \#4 循环

 for node in ${targethosts};do

​     echo -----------------hadoop$host ----------------

​     rsync -av $pdir/$fname hadoop@$node:$pdir

 done

**在NameNode上操作，运行的格式为sh xsync.sh xxx文件 这样每次修改完文件就可以快速分发到其他节点了**

## 三、Hive

基于以上配置把hive部署到NodeManager-DataNode-Hive，hive自带的元数据库只支持单用户登录，需要安装单独安装mysql作为元数据库。mysql5.7安装到NodeManager-DataNode-2

 

**解压对应的hive安装包**

.    

tar -xzvf /home/hive/apache-hive-3.1.1-bin.tar.gz

 

 

**配置hive 进入apache-hive-3.1.1-bin/conf/目录 复制hive-env.sh.template 为 hive-env.sh**

```
cp hive-env.sh.template hive-env.sh
```

**编辑hive-env.sh**

export HADOOP_HOME=/home/hadoop/hadoop3.1.2/hadoop-3.1.2

export HIVE_CONF_DIR=/home/hive/apache-hive-3.1.2-bin/conf

export HIVE_AUX_JARS_PATH=/home/hive/apache-hive-3.1.2-bin/lib

 

 

 

 

**复制hive-default.xml.template 为 hive-site.xml**

```
cp hive-default.xml.template hive-site.xml
 
```

**修改hive-site.xml，可以只保留下面配置删掉其余配置，以免存在什么冲突**

<configuration>

 <property>

​     <name>javax.jdo.option.ConnectionURL</name>

<value>jdbc:mysql://NodeManager-DataNode-2:3306/hive?createDatabaseIfNotExist=true&amp;useSSL=false</value>

<description>

​    JDBC connect string for a JDBC metastore. To use SSL to encrypt/authenticate the connection, provide database-specific SSL flag in the connection URL. For example, jdbc:postgresql://myhost/db?ssl=true for postgres database.

​    </description>

</property>

 

 

 <property>

​       <name>javax.jdo.option.ConnectionDriverName</name>

​       <value>com.mysql.cj.jdbc.Driver</value>

​       <description>Driver class name for a JDBC metastore</description>

</property>

 

 

 <property>

   <name>javax.jdo.option.ConnectionUserName</name>

  <value>hadoop</value>

<description>Username to use against metastore database

</description>

</property>

 

 

<property>

  <name>javax.jdo.option.ConnectionPassword</name>

  <value>123456</value>

<description>password to use against metastore database</description>

</property>

 

</configuration>

.    

 

**复制hive-exec-log4j2.properties.template 为 hive-exec-log4j2.properties**

cp hive-exec-log4j2.properties.template hive-exec-log4j2.properties

 

复制`hive-log4j2.properties.template`为`hive-log4j2.properties`

cp hive-log4j2.properties.template hive-log4j2.properties

 

**下载mysql驱动放入/home/hadoop/apache-hive-3.1.1-bin/lib包中**

**在****NodeManager-DataNode-2****安装mysql5.7版本， 并且设置nodemanager-datanode-hive主机访问以hadoop用户登录密码123456访问**

.    

use mysql;

select host,user from user;

grant all privileges on *.* to hadoop@‘nodemanager-datanode-hive’ identified by "123456";

flush privileges;

select host,user from user;

 

 

 

**在nodemanager-datanode-hive节点**

**初始化（第一次启动）**

```
./schematool -initSchema -dbType mysql
```

**Mysql****数据库中会自动创建hive数据库**

```
 
```

**启动**

.    

./hive

 

**上述配置hive元数据服务metastore是嵌入在hive进程中的，不利于远程访问，可以单独配置metastore服务**

 

**配置metastore，先把metastore设在本地**

**hive-site.xml****添加如下信息**

<!--配置metastore-->

 

<property>

​        <name>hive.metastore.uris</name>

​          <value>thrift://NodeManager-DataNode-Hive:9083</value>

 </property>

 

 

<property>

​    <name>hive.metastore.local</name>

​    <value>true</value>

</property>

 

 

**后台启动metastore服务器**

sudo nohup bin/hive --service metastore 2>&1 >> /tmp/metastore.log & 

**后台启动hiveserver2**

 bin/hive --service hiveserver2 （后台启动有问题，目前还是前台启动）

**（****nohup** **bin/****hive --service hiveserver2 &****）貌似可以**

**在把数据平台整合接入hive的时候发现查出来的表结构中字段名带有表名，在hive-site.xml中加入以下配置后hiveserver2可不显示表名**

<!--不显示表名-->

<property>

*hive.resultset.use.unique.column.names
 false
 *

**显示字段名**-->
 
 hive.cli.print.header
 true
 *

**重启hiveserver2**

**Hive web****控制台**

http://121.41.10.75:10002/