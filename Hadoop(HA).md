# 官方地址：

http://hadoop.apache.org/



# 搭建HA

在/opt下创建ha目录

```linux
sudo mkdir /opt/ha
```

设置所属主和所属组
```linux
sudo chown atguigu:atguigu ha
```

将原hadoop拷贝到/opt/ha下（在hadoop102上即可--后面会分发）
```linux
cp -r /opt/module/hadoop-3.3.4 /opt/ha
```

将/opt/ha下的hadoop里面的data logs 和/tmp/* 全部删除
（在hadoop102上即可--因为hadoop103和hadoop104没有分发呢)

```linux
rm -rf data logs
sudo rm -rf /tmp/*
```

配置环境变量（将指向/opt/module/hadoop-3.3.4 修改成指向/opt/ha/hadoop-3.3.4）
```linux
sudo vim /etc/profile.d/my_env.sh


#HADOOP_HOME
export HADOOP_HOME=/opt/ha/hadoop-3.3.4
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin
```

```shell
修改
#定义HADOOP_HOME变量
export HADOOP_HOME=/opt/ha/hadoop-3.3.4
```

分发环境变量

让hadoop102,hadoop103,hadoop104环境变量生效

分发Hadoop
	xsync /opt/ha/hadoop-3.3.4

# HDFS-HA配置

配置core-site.xml

```shell
<!-- 把多个NameNode的地址组装成一个集群mycluster -->
	  <property>
		<name>fs.defaultFS</name>
		<value>hdfs://mycluster</value>
	  </property>

	  <!-- 指定hadoop运行时产生文件的存储目录 -->
	  <property>
		<name>hadoop.tmp.dir</name>
		<value>/opt/ha/hadoop-3.3.4/data</value>
	  </property>
	<!--   配置ZOOKEEPER的地址  -->
	<property>
		<name>ha.zookeeper.quorum</name>
		<value>hadoop102:2181,hadoop103:2181,hadoop104:2181</value>
	</property>
```

配置hdfs-site.xml

```shell
<!-- NameNode数据存储目录 -->
	  <property>
		<name>dfs.namenode.name.dir</name>
		<value>file://${hadoop.tmp.dir}/name</value>
	  </property>

	  <!-- DataNode数据存储目录 -->
	  <property>
		<name>dfs.datanode.data.dir</name>
		<value>file://${hadoop.tmp.dir}/data</value>
	  </property>

	  <!-- JournalNode数据存储目录 -->
	  <property>
		<name>dfs.journalnode.edits.dir</name>
		<value>${hadoop.tmp.dir}/jn</value>
	  </property>

	  <!-- 完全分布式集群名称 -->
	  <property>
		<name>dfs.nameservices</name>
		<value>mycluster</value>
	  </property>

	  <!-- 集群中NameNode节点都有哪些 -->
	  <property>
		<name>dfs.ha.namenodes.mycluster</name>
		<value>nn1,nn2,nn3</value>
	  </property>

	  <!-- NameNode的RPC通信地址 -->
	  <property>
		<name>dfs.namenode.rpc-address.mycluster.nn1</name>
		<value>hadoop102:8020</value>
	  </property>
	  <property>
		<name>dfs.namenode.rpc-address.mycluster.nn2</name>
		<value>hadoop103:8020</value>
	  </property>
	  <property>
		<name>dfs.namenode.rpc-address.mycluster.nn3</name>
		<value>hadoop104:8020</value>
	  </property>

	  <!-- NameNode的http通信地址 -->
	  <property>
		<name>dfs.namenode.http-address.mycluster.nn1</name>
		<value>hadoop102:9870</value>
	  </property>
	  <property>
		<name>dfs.namenode.http-address.mycluster.nn2</name>
		<value>hadoop103:9870</value>
	  </property>
	  <property>
		<name>dfs.namenode.http-address.mycluster.nn3</name>
		<value>hadoop104:9870</value>
	  </property>

	  <!-- 指定NameNode元数据在JournalNode上的存放位置 -->
	  <property>
		<name>dfs.namenode.shared.edits.dir</name>
	<value>qjournal://hadoop102:8485;hadoop103:8485;hadoop104:8485/mycluster</value>
	  </property>

	  <!-- 访问代理类：client用于确定哪个NameNode为Active -->
	  <property>
		<name>dfs.client.failover.proxy.provider.mycluster</name>
		<value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
	  </property>

	  <!-- 配置隔离机制，即同一时刻只能有一台服务器对外响应 -->
	  <property>
		<name>dfs.ha.fencing.methods</name>
		<value>sshfence</value>
	  </property>

	  <!-- 使用隔离机制时需要ssh秘钥登录-->
	  <property>
		<name>dfs.ha.fencing.ssh.private-key-files</name>
		<value>/home/atguigu/.ssh/id_rsa</value>
	  </property>
	<!-- 启用nn故障自动转移 -->
	<property>
		 <name>dfs.ha.automatic-failover.enabled</name>
		 <value>true</value>
	</property>
```

分发

# 启动

在(hadoop102,hadoop103,hadoop104)启动journalnode
```shell
hdfs --daemon start journalnode
```

在[hadoop102]上，对其进行格式化，并启动NameNode

```shell
hdfs namenode -format
hdfs --daemon start namenode
```

在hadoop103和hadoop104上同步元数据并启动NameNode

```shell
hdfs namenode -bootstrapStandby
hdfs --daemon start namenode
```

启动三台节点上的datanode

```shell
hdfs --daemon start datanode
```

查看页面

关闭所有HDFS服务：
```shell
stop-dfs.sh
```

启动Zookeeper集群：
```shell
zk.sh start
```

初始化HA在Zookeeper中状态：
```shell
hdfs zkfc -formatZK
```

启动HDFS服务（通过jpsall查看是否对）：

```shell
start-dfs.sh
```

验证
将active的那台namenode给停掉查看其它namenode有没有变成active

```shell
hdfs --daemon stop namenode
```

# YARN-HA配置

## 官方文档：

[http://hadoop.apache.org/docs/r3.3.4/hadoop-yarn/hadoop-yarn-site/ResourceManagerHA.html](http://hadoop.apache.org/docs/r2.7.2/hadoop-yarn/hadoop-yarn-site/ResourceManagerHA.html)

配置yarn-site.xml

```shell
	<property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>

    <!-- 启用resourcemanager ha -->
    <property>
        <name>yarn.resourcemanager.ha.enabled</name>
        <value>true</value>
    </property>
 
    <!-- 声明两台resourcemanager的地址 -->
    <property>
        <name>yarn.resourcemanager.cluster-id</name>
        <value>cluster-yarn1</value>
    </property>

    <!--指定resourcemanager的逻辑列表-->
    <property>
        <name>yarn.resourcemanager.ha.rm-ids</name>
        <value>rm1,rm2,rm3</value>
    </property>
<!-- ========== rm1的配置 ========== -->
    <!-- 指定rm1的主机名 -->
    <property>
        <name>yarn.resourcemanager.hostname.rm1</name>
        <value>hadoop102</value>
    </property>

    <!-- 指定rm1的web端地址 -->
    <property>
        <name>yarn.resourcemanager.webapp.address.rm1</name>
        <value>hadoop102:8088</value>
    </property>

    <!-- 指定rm1的内部通信地址 -->
    <property>
        <name>yarn.resourcemanager.address.rm1</name>
        <value>hadoop102:8032</value>
    </property>

    <!-- 指定AM向rm1申请资源的地址 -->
    <property>
        <name>yarn.resourcemanager.scheduler.address.rm1</name>  
        <value>hadoop102:8030</value>
    </property>

    <!-- 指定供NM连接的地址 -->  
    <property>
    <name>yarn.resourcemanager.resource-tracker.address.rm1</name>
        <value>hadoop102:8031</value>
    </property>

<!-- ========== rm2的配置 ========== -->
    <!-- 指定rm2的主机名 -->
    <property>
        <name>yarn.resourcemanager.hostname.rm2</name>
        <value>hadoop103</value>
    </property>
    <property>
        <name>yarn.resourcemanager.webapp.address.rm2</name>
        <value>hadoop103:8088</value>
    </property>
    <property>
        <name>yarn.resourcemanager.address.rm2</name>
        <value>hadoop103:8032</value>
    </property>
    <property>
        <name>yarn.resourcemanager.scheduler.address.rm2</name>
        <value>hadoop103:8030</value>
    </property>

    <property>
<name>yarn.resourcemanager.resource-tracker.address.rm2</name>
        <value>hadoop103:8031</value>
    </property>

<!-- ========== rm3的配置 ========== -->
    <!-- 指定rm1的主机名 -->
    <property>
        <name>yarn.resourcemanager.hostname.rm3</name>
        <value>hadoop104</value>
    </property>
    <!-- 指定rm1的web端地址 -->
    <property>
        <name>yarn.resourcemanager.webapp.address.rm3</name>
        <value>hadoop104:8088</value>
    </property>
    <!-- 指定rm1的内部通信地址 -->
    <property>
        <name>yarn.resourcemanager.address.rm3</name>
        <value>hadoop104:8032</value>
    </property>
    <!-- 指定AM向rm1申请资源的地址 -->
    <property>
        <name>yarn.resourcemanager.scheduler.address.rm3</name>  
        <value>hadoop104:8030</value>
    </property>

    <!-- 指定供NM连接的地址 -->  
    <property>
    <name>yarn.resourcemanager.resource-tracker.address.rm3</name>
        <value>hadoop104:8031</value>
    </property>

    <!-- 指定zookeeper集群的地址 --> 
    <property>
        <name>yarn.resourcemanager.zk-address</name>
        <value>hadoop102:2181,hadoop103:2181,hadoop104:2181</value>
    </property>

    <!-- 启用自动恢复 --> 
    <property>
        <name>yarn.resourcemanager.recovery.enabled</name>
        <value>true</value>
    </property>
 
    <!-- 指定resourcemanager的状态信息存储在zookeeper集群 --> 
    <property>
        <name>yarn.resourcemanager.store.class</name>     
		<value>org.apache.hadoop.yarn.server.resourcemanager.recovery.ZKRMStateStore</value>
</property>

    <!-- 环境变量的继承 -->
    <property>
        <name>yarn.nodemanager.env-whitelist</name>
        <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
    </property>

```

分发文件

在hadoop102中执行：
```linux
start-yarn.sh
```

查看服务状态

```linux
yarn rmadmin -getServiceState rm1
```

