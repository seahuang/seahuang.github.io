
# How to install hadoop 2.8.0

作者：海啸 

版本历史：

| ver | date | description |
| ----- |:-----:| -----:|
| 1.0 | 2017-06-06 | 创建 |


## 1. 目的
> 本文档描述了如何在集群环境下安装和配置Hadoop 2.8.0。

备注：
```
# 表示root用户
$ 表示hadoop用户
```

## 2. 先决条件
### 2.1 集群清单

| ip  | hostname | 备注 |
| --- |:-- -----:| -----------:|
| ip1 | hadoop1  | master(主机) |
| ip2 | hadoop2  | slave1(从机) |
| ip3 | hadoop3  | slave2(从机) |

### 2.2 系统要求
+ Centos 7
+ Java 1.7.0_51
+ 下载 hadoop 2.8.0

```
#cd ~
#wget http://www.apache.org/dyn/closer.cgi/hadoop/common/hadoop-2.8.0/hadoop-2.8.0.tar.gz
```

## 3. 主机设置（集群）
### 3.1 创建用户（hadoop）
```
#useradd -d /home/hadoop -s /bin/bash hadoop
#passwd hadoop
hadoop
```

### 3.2 解压缩
```
#cd ~
#tar xvfz hadoop-2.8.0.tar.gz
#mv ~/hadoop-2.8.0 /usr/local
#mv /usr/local/hadoop-2.8.0 /usr/local/hadoop
#chown -R hadoop:hadoop /usr/local/hadoop
```

### 3.3 修改主机名
```
#hostnamectl set-hostname 对应主机名
#hostname
```

### 3.4 配置/etc/hosts
/etc/hosts文件增加如下
```
#vi /etc/hosts
ip1 hadoop1
ip2 hadoop2
ip3 hadoop3
```

### 3.5 设置免密登录
需要保证主机可以免密登录从机。
+ 主机
```
$ssh-keygen
...
$cat ~/.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC6DmGHdfuvb92InJpkvteFF08H48m4edBxUNh8sMeZBseEE8gUDipAE7VH5lcqCNrV69P6ZB45HreTRbwaK3xjzGrt87lR7q3nPTIME7tfgOJUJJDvZFSulKthQlNB7izqqM+hfP2c61hEY2ildTaJhesNro2O6S9diJlKd6kXkcjp/zBuQSBSvQJhNBcKhjvaSJOLgwXMEmihrDbHTir/qBxCnENWSXqfNJgkH9k6Y16fmU9RUYX7/O01GsKD1hkNiIApNql/vM5Ef17rKry0gDQspVib88Ey+uV6hQnAwAhnG7QQ5eJRjNa5tNxxreH4I9YoDyPJcPfFqKBO/VgD hadoop@localhost.localdomain
```

+ 从机
```
$ssh-keygen
...
$cat ~/.ssh/id_rsa.pub
ssh-rsa ... hadoop@localhost.localdomain
$echo 主机id_rsa.pub >> ~/.ssh/authorized_keys
```

+ 验证(主机)
```
$sftp hadoop@hadoop2
...
$sftp hadoop@hadoop3
...

```

### 3.6 设置环境变量
+ 设置java环境
```
#vi /etc/profile.d/java.sh
```
```
# java initialization script (sh)
JAVA_HOME=/usr/java/jdk1.7.0_51
CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
PATH=$JAVA_HOME/bin:$PATH

export JAVA_HOME CLASSPATH PATH

```

+ 设置hadoop环境
```
#vi /etc/profile.d/hadoop.sh
```
```
# hadoop initialization script (sh)
HADOOP_HOME=/usr/local/hadoop
HADOOP_PREFIX=$HADOOP_HOME

LD_LIBRARY_PATH=${HADOOP_HOME}/lib/native/:$LD_LIBRARY_PATH
PATH=$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$PATH

export HADOOP_HOME HADOOP_PREFIX PATH LD_LIBRARY_PATH

```

## 4. hadoop设置（主机）
### 4.1 修改hadoop-env.sh
> $vi $HADOOP_HOME/etc/hadoop/hadoop-env.sh
```
JAVA_HOME=/usr/java/jdk1.7.0_51
```

### 4.2 配置slaves
> $vi $HADOOP_HOME/etc/hadoop/slaves
```
hadoop1
hadoop2
hadoop3
```

### 4.3 配置core-site.xml
> $vi $HADOOP_HOME/etc/hadoop/core-site.xml
```
<configuration>
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://hadoop1:9000</value>
  </property>
  <property>
    <name>io.file.buffer.size</name>
    <value>131072</value>
  </property>
  <property>
    <name>hadoop.tmp.dir</name>
    <value>file:/usr/local/hadoop/tmp</value>
  </property>
  <property>
    <name>hadoop.proxyuser.hadoop.hosts</name>
    <value>*</value>
  </property>
  <property>
    <name>hadoop.proxyuser.hadoop.groups</name>
    <value>*</value>
  </property>
</configuration>
```

### 4.4 配置hdfs-site.xml
> $vi $HADOOP_HOME/etc/hadoop/hdfs-site.xml
```
<configuration>
  <property>
    <name>dfs.namenode.name.dir</name>
    <value>file:/usr/local/hadoop/hdfs/name</value>
  </property>
  <property>
    <name>dfs.datanode.data.dir</name>
    <value>file:/usr/local/hadoop/hdfs/data</value>
  </property>
  <property>
    <name>dfs.replication</name>
    <value>3</value>
  </property>
  <property>
    <name>dfs.namenode.secondary.http-address</name>
    <value>hadoop1:9001</value>
  </property>
  <property>
    <name>dfs.webhdfs.enabled</name>
    <value>true</value>
  </property>
</configuration>
```

### 4.5 配置mapred-site.xml
> $vi $HADOOP_HOME/etc/hadoop/mapred-site.xml
```
<configuration>
  <property> 
    <name>mapreduce.framework.name</name> 
    <value>yarn</value> 
  </property>
  <property>
    <name>mapreduce.jobhistory.address</name>
    <value>hadoop1:10020</value>
  </property>
  <property>
    <name>mapreduce.jobhistory.webapp.address</name>
    <value>hadoop1:19888</value>
  </property>
</configuration>
```

### 4.6 配置yarn-site.xml
> $vi $HADOOP_HOME/etc/hadoop/yarn-site.xml
```
<configuration>
  <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
  </property>
  <property>
    <name>yarn.nodemanager.auxservices.mapreduce.shuffle.class</name>
    <value>org.apache.hadoop.mapred.ShuffleHandler</value>
  </property>
  <property>
    <name>yarn.resourcemanager.address</name>
    <value>hadoop1:8032</value>
  </property>
  <property>
    <name>yarn.resourcemanager.scheduler.address</name>
    <value>hadoop1:8030</value>
  </property>
  <property>
    <name>yarn.resourcemanager.resource-tracker.address</name>
    <value>hadoop1:8031</value>
  </property>
  <property>
    <name>yarn.resourcemanager.admin.address</name>
    <value>hadoop1:8033</value>
  </property>
  <property>
    <name>yarn.resourcemanager.webapp.address</name>
    <value>hadoop1:8088</value>
  </property>
  <property>
    <name>yarn.nodemanager.resource.memory-mb</name>
    <value>6078</value>
  </property>
</configuration>
```

### 4.7 配置从机
```
$scp -r $HADOOP_HOME/etc/hadoop hadoop@hadoop2:/usr/local/hadoop/etc/
$scp -r $HADOOP_HOME/etc/hadoop hadoop@hadoop3:/usr/local/hadoop/etc/
```

## 5. hadoop运维（主机）

### 5.1 格式化namenode（只需一次）
```
$hdfs namenode -format
```
注意：只需一次，下次启动不再需要格式化。
结果倒数第五行出现如下提示则为成功
```
...
Exiting with status 0 
...
```

### 5.2 启动hadoop集群
```
$start-all.sh
```
或
```
$start-dfs.sh
$start-yarn.sh
```

### 5.3 查看节点
```
$hdfs dfsadmin -report
Configured Capacity: 160982630400 (149.93 GB)
Present Capacity: 140953784320 (131.27 GB)
DFS Remaining: 140952612864 (131.27 GB)
DFS Used: 1171456 (1.12 MB)
DFS Used%: 0.00%
Under replicated blocks: 0
Blocks with corrupt replicas: 0
Missing blocks: 0
Missing blocks (with replication factor 1): 0
Pending deletion blocks: 0

-------------------------------------------------
Live datanodes (3):

Name: ip2:50010 (hadoop2)
Hostname: hadoop2
Decommission Status : Normal
Configured Capacity: 53660876800 (49.98 GB)
DFS Used: 389120 (380 KB)
Non DFS Used: 6674685952 (6.22 GB)
DFS Remaining: 46985801728 (43.76 GB)
DFS Used%: 0.00%
DFS Remaining%: 87.56%
Configured Cache Capacity: 0 (0 B)
Cache Used: 0 (0 B)
Cache Remaining: 0 (0 B)
Cache Used%: 100.00%
Cache Remaining%: 0.00%
Xceivers: 1
Last contact: Fri Jun 09 14:13:14 CST 2017


Name: ip3:50010 (hadoop3)
Hostname: hadoop3
Decommission Status : Normal
Configured Capacity: 53660876800 (49.98 GB)
DFS Used: 389120 (380 KB)
Non DFS Used: 6679486464 (6.22 GB)
DFS Remaining: 46981001216 (43.75 GB)
DFS Used%: 0.00%
DFS Remaining%: 87.55%
Configured Cache Capacity: 0 (0 B)
Cache Used: 0 (0 B)
Cache Remaining: 0 (0 B)
Cache Used%: 100.00%
Cache Remaining%: 0.00%
Xceivers: 1
Last contact: Fri Jun 09 14:13:12 CST 2017


Name: ip1:50010 (hadoop1)
Hostname: hadoop1
Decommission Status : Normal
Configured Capacity: 53660876800 (49.98 GB)
DFS Used: 393216 (384 KB)
Non DFS Used: 6674673664 (6.22 GB)
DFS Remaining: 46985809920 (43.76 GB)
DFS Used%: 0.00%
DFS Remaining%: 87.56%
Configured Cache Capacity: 0 (0 B)
Cache Used: 0 (0 B)
Cache Remaining: 0 (0 B)
Cache Used%: 100.00%
Cache Remaining%: 0.00%
Xceivers: 1
Last contact: Fri Jun 09 14:13:14 CST 2017

```

### 5.4 启动history服务
```
$mr-jobhistory-daemon.sh start historyserver
```
不然在面板中不能打开history链接。

### 5.5 查看进程
```
$jps
4504 ResourceManager
4066 DataNode
4761 NodeManager
5068 JobHistoryServer
4357 SecondaryNameNode
3833 NameNode
5127 Jps
```

### 5.6 停止hadoop集群
```
$stop-all.sh
```
或
```
$stop-dfs.sh
$stop-yarn.sh
```

### 5.7 WEB服务
| 服务                   	 | URL              		   | 备注	|
|----------------------------|:---------------------------:|-------:|
|NameNode                    | http://ip1:50070/ |    	|
|ResourceManager             | http://ip1:8088/  |      	|
|MapReduce JobHistory Server | http://ip1:19888/ |      	|


## 6. hadoop使用（主机）

### 6.1 查看hdfs文件目录
```
$hadoop fs -ls /tmp
```

### 6.2 创建hdfs文件目录
```
$hadoop fs -mkdir /tmp
```

### 6.3 向hdfs中上传文件
```
$vi ~/hadoop-test-data.txt
...
$hadoop fs -put ~/hadoop-test-data.txt /tmp/hadoop-test-data.txt
```

### 6.4 hdfs移除文件命令
```
$hadoop fs -rm -r /tmp/hadoop-test-data.txt
```

### 6.5 WordCount测试
```
$cd $HADOOP_HOME
$hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.8.0.jar wordcount /tmp/hadoop-test-data.txt output
17/06/09 14:39:07 INFO client.RMProxy: Connecting to ResourceManager at hadoop1/ip1:8032
17/06/09 14:39:09 INFO input.FileInputFormat: Total input files to process : 1
17/06/09 14:39:09 INFO mapreduce.JobSubmitter: number of splits:1
17/06/09 14:39:09 INFO mapreduce.JobSubmitter: Submitting tokens for job: job_1496917561417_0002
17/06/09 14:39:10 INFO impl.YarnClientImpl: Submitted application application_1496917561417_0002
17/06/09 14:39:10 INFO mapreduce.Job: The url to track the job: http://hadoop1:8088/proxy/application_1496917561417_0002/
17/06/09 14:39:10 INFO mapreduce.Job: Running job: job_1496917561417_0002
17/06/09 14:39:22 INFO mapreduce.Job: Job job_1496917561417_0002 running in uber mode : false
17/06/09 14:39:22 INFO mapreduce.Job:  map 0% reduce 0%
17/06/09 14:39:32 INFO mapreduce.Job:  map 100% reduce 0%
17/06/09 14:39:41 INFO mapreduce.Job:  map 100% reduce 100%
17/06/09 14:39:41 INFO mapreduce.Job: Job job_1496917561417_0002 completed successfully
17/06/09 14:39:42 INFO mapreduce.Job: Counters: 49
	File System Counters
		FILE: Number of bytes read=2718
		FILE: Number of bytes written=278585
		FILE: Number of read operations=0
		FILE: Number of large read operations=0
		FILE: Number of write operations=0
		HDFS: Number of bytes read=2300
		HDFS: Number of bytes written=1949
		HDFS: Number of read operations=6
		HDFS: Number of large read operations=0
		HDFS: Number of write operations=2
	Job Counters
		Launched map tasks=1
		Launched reduce tasks=1
		Data-local map tasks=1
		Total time spent by all maps in occupied slots (ms)=7619
		Total time spent by all reduces in occupied slots (ms)=6308
		Total time spent by all map tasks (ms)=7619
		Total time spent by all reduce tasks (ms)=6308
		Total vcore-milliseconds taken by all map tasks=7619
		Total vcore-milliseconds taken by all reduce tasks=6308
		Total megabyte-milliseconds taken by all map tasks=7801856
		Total megabyte-milliseconds taken by all reduce tasks=6459392
	Map-Reduce Framework
		Map input records=32
		Map output records=310
		Map output bytes=3416
		Map output materialized bytes=2718
		Input split bytes=109
		Combine input records=310
		Combine output records=191
		Reduce input groups=191
		Reduce shuffle bytes=2718
		Reduce input records=191
		Reduce output records=191
		Spilled Records=382
		Shuffled Maps =1
		Failed Shuffles=0
		Merged Map outputs=1
		GC time elapsed (ms)=115
		CPU time spent (ms)=1700
		Physical memory (bytes) snapshot=459010048
		Virtual memory (bytes) snapshot=1842786304
		Total committed heap usage (bytes)=290979840
	Shuffle Errors
		BAD_ID=0
		CONNECTION=0
		IO_ERROR=0
		WRONG_LENGTH=0
		WRONG_MAP=0
		WRONG_REDUCE=0
	File Input Format Counters
		Bytes Read=2191
	File Output Format Counters
		Bytes Written=1949
$hadoop fs -ls /user/hadoop/output
Found 2 items
-rw-r--r--   3 hadoop supergroup          0 2017-06-09 14:39 /user/hadoop/output/_SUCCESS
-rw-r--r--   3 hadoop supergroup       1949 2017-06-09 14:39 /user/hadoop/output/part-r-00000
```
