# MapReduce详细工作流程

job提交流程

## 

InputFormat 切片(片大小等于块大小)

上传配置信息

上传Jar包(集群上在HDFS上,本地在工程所在盘符的根目录)

客户端提交job给yarn的ResourceManager

ResourceManager启动appMaster申请资源(有几片就申请几个)

给的资源叫container(有几个MapTask就会给几个资源)

MapTask调用Mapper类中的map方法

inputFormat的RecorderReader读数据,读到Mapper的map方法

处理,

处理完后map方法把数据写到Shuffle(环形缓冲区)

先分区，算出分区号（几个RedderTask几个分区）

进入到环形缓冲区(在内存中占内存100M)(作用是排序相同分区在一起用**快速排序**按照Key来排)(分区号属于元数据||左边是数据右面是元数据)(数据放百分之80之后排序之后可以反向接着写)

之后数据按照**相同分区开始快速排序**

写到磁盘上

之后写到磁盘的文件归并排序//////  ///////////////////////////////////                       

分区输出mapTask的结果（可以是N个）

起N个ReduceTask把相同分区数据copy到一个ReduceTask(如果内存满了溢出到磁盘)

再次归并排序

分组(相同的Key为一组)

一组一组的数据进入Reducer的reduce方法

OutFormat吧数据写出去 



# Yarn的工作机制

MapReduce程序提交到客户端所在的节点

向ResourceManage申请运行一个Job

ResourceManage告诉客户端所在节点一个地址客户端往HDFS上传资料(jar包,切片,配置信息)

上传完毕后客户端会再次找ResourceManage告诉它资源提交完毕申请运行AppMaster

ResourceManage会把用户的请求初始化成一个Task放到FIFO调度队列(先进来的先执行)

NodeManage领任务创建Container启动Appmaster

把资料拿到本地,申请资源

申请运行MapTask

领取到任务创建Container

AppMaster负责启动MapTask在HDFS读数据(跑map)

在本地出结果

Appmaster再次申请container跑ReduceTask运行Reducer

相同分区的数据会copy过来在把数据写出去写到HDFS

Appmaster向ResourceManage注销 自己 









# inputformat

切片

读取数据

# OutputFormat

把数据写出去

做输出检查



## 切片如何切的

剩余文件大小  /   切片大小 > 1.1

如果不可切,整个文件是一片.

# MinSize和MaxSize的修改

MinSize和MinSize影响了**切片**的大小

## 1.代码中设置

```java
Configuration conf = new Configuration();
conf.setLong("mapreduce.input.fileinputformat.split.minsize", 字节);
conf.setLong("mapreduce.input.fileinputformat.split.maxsize", 字节);
```

## 2.配置文件中设置

```shell
<property>
  <name>mapreduce.input.fileinputformat.split.minsize</name>
  <value>134217728</value> <!-- 设置最小分片大小为128MB -->
</property>
 
<property>
  <name>mapreduce.input.fileinputformat.split.maxsize</name>
  <value>67108864</value> <!-- 设置最大分片大小为64MB -->
</property>
```



# InputFormat(抽象类)的作用

TextInputFormat 这个是默认使用的InputFormat

切片是在**getSplits**里面切的

默认读取数据的是**LineRecordReader**

生成切片

RecordReader用来读数据

# CombineTextInputFormat的修改(小文件切片)

```java
//设置虚拟存储切片最大值
CombineTextInputFormat.setMaxInputSplitSize(job,"字节");
//设置CombineTextInputFormat
job.setInputFormatClass(CombineTextInputFormat.class);
```

# ReduceTasks的数量设置                  

```java
// 默认值是1，手动设置为4
job.setNumReduceTasks(4);
```

# Combiner合并

(1)Combiner是MR程序中Mapper和Reducer之外的一种组件.

(2)Combiner组件的父类就是Reducer.

(3)Combiner和Reducer的区别在于运行的位置

​	Combiner是在每一个MapTask所在的节点运行

​	Reducer是接收全局所有Mapper的输出结果

(4)Combiner的意义就是对每一个MapTask的输出进行局部汇总,以减小网络传输量

(5)combiner能够应用的前提是不能影响最终的业务逻辑,而且Combiner的输出KV应该跟Reducer的输出KV类型要对应起来.

## 自定义Combiner实现步骤

①自定义一个Combiner继承Reducer，重写Reduce方法

```java
public class WordCountCombiner extends Reducer<Text, IntWritable, Text, IntWritable> {

    private IntWritable outV = new IntWritable();

    @Override
    protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {

        int sum = 0;
        for (IntWritable value : values) {
            sum += value.get();
        }
     
        outV.set(sum);
     
        context.write(key,outV);
    }
}
```

②在Job驱动类中设置： 

```java
job.setCombinerClass(WordCountCombiner.class);
```



# 自定义分区

```java
job.setPartitionerClass(TTMyPartitioner.class);
job.setNumReduceTasks(2);
```

ReduceTasks=0表示没有Reduce阶段,输出文件和Map个数一致

# Partitioner分区

key.hashCode()&Integer.MAXVALUE % numReduces

# Hadoop三种调度器各自的特点

FIFO调度器:支持单列对,先进先出,生产环境不会用

容量调度器:支持多列队,优先选择资源占用率低的队列分配资源,按照作业优先级和提交时间顺序分配支援.容量资源分配:本地原则..小中工资会用

公平调度器:支持多列队,保证每个任务公平享有队列资源.资源不够时可以按照缺额分配. 大厂使用

# 添加磁盘

lsblk 看看新家的磁盘是什么名字

sudo fdisk /dev/磁盘名字

输入n,,按三次回车,,,输入w

lsblk查看刚刚添加的分区

sudo mkfs.ext4 /dev/新盘名字

​                临时有效
​				sudo mount /dev/sdb1 /opt/sdb   #注意：只是本次有效
​				永久有效
​				sudo vim /etc/fstab
​				添加如下内容（重启linux并检查lsblk）
​				/dev/sdb1 /opt/sdb(挂载目录)      ext4    defaults        0       0

sudo chown -R atguigu:atguigu /opt/sdb 修改挂载目录的所属组/主

在hdfs-site.xml写如下内容

```shell
<property>
     <name>dfs.datanode.data.dir</name>
     <value>file://${hadoop.tmp.dir}/dfs/data,file:///opt/sdb(挂载目录) </value>
</property>

```

重启集群

# 集群均衡-磁盘均衡

生成均衡计划

**hdfs diskbalancer -plan hadoop102 --thresholdPercentage 1 –v**

执行均衡计划

**hdfs diskbalancer -execute /(生成出来Writing plan to下面的东西)**

查看当前均衡任务的执行情况

**hdfs diskbalancer -query hadoop102**

**Result: PLAN_UNDER_PROGRESS** --表示正在执行

**Result: PLAN_DONE**  --表示执行结束

# 服役新服务器

修改IP地址和主机名称

```shell
vim /etc/sysconfig/network-scripts/ifcfg-ens33

vim /etc/hostname
```

删除Hadoop的历史数据，data和log数据

配置hadoop的ssh无密登录

```shell
ssh-genkey -t rsa 
ssh-copy-id hadoop111
```

改hadoop软件目录下的workers

启动集群

```shell
hdfs --daemon start datanode
yarn --daemon start nodemanager
```

# 服务器间数据均衡

**开启数据均衡命令**

```shell
sbin/start-balancer.sh -threshold 10
```

**停止数据均衡命令**

```shell
sbin/stop-balancer.sh
```

# 黑白名单

hadoop文件目录下创建**whitelist** 和**blacklist**文件

在hdfs-site.xml添加配置文件

```shell
<!-- 白名单 -->
<property>
     <name>dfs.hosts</name>
     <value>/opt/module/hadoop-3.3.4/etc/hadoop/whitelist</value>
</property>

<!-- 黑名单 -->
<property>
     <name>dfs.hosts.exclude</name>
     <value>/opt/module/hadoop-3.3.4/etc/hadoop/blacklist</value>
</property>
```

分发配置文件whitelist，hdfs-site.xml

```shell
xsync hdfs-site.xml whitelist
```

第一次添加白名单必须重启集群，不是第一次，只需要刷新NameNode节点即可

刷新NameNode

```shell
hdfs dfsadmin -refreshNodes
```

# HDFS—安全模式

```shell
（1）bin/hdfs dfsadmin -safemode get	（功能描述：查看安全模式状态）
（2）bin/hdfs dfsadmin -safemode enter （功能描述：进入安全模式状态）
（3）bin/hdfs dfsadmin -safemode leave	（功能描述：离开安全模式状态）
（4）bin/hdfs dfsadmin -safemode wait	（功能描述：等待安全模式状态）
```

# Hadoop数据压缩

1）压缩算法对比介绍

| **压缩格式** | **Hadoop****自带？** | **算法** | **文件扩展名** | **是否可切片** | **换成压缩格式后，原来的程序是否需要修改** |
| ------------ | -------------------- | -------- | -------------- | -------------- | ------------------------------------------ |
| **DEFLATE**  | 是，直接使用         | DEFLATE  | .deflate       | 否             | 和文本处理一样，不需要修改                 |
| **Gzip**     | 是，直接使用         | DEFLATE  | .gz            | 否             | 和文本处理一样，不需要修改                 |
| **bzip2**    | 是，直接使用         | bzip2    | .bz2           | 是             | 和文本处理一样，不需要修改                 |
| **LZO**      | 否，需要安装         | LZO      | .lzo           | 是             | 需要建索引，还需要指定输入格式             |
| **Snappy**   | 是，直接使用         | Snappy   | .snappy        | 否             | 和文本处理一样，不需要修改                 |

**为了支持多种压缩/****解压缩算法，Hadoop****引入了编码/****解码器**

| **压缩格式** | **对应的编码/解码器**                      |
| ------------ | ------------------------------------------ |
| **DEFLATE**  | org.apache.hadoop.io.compress.DefaultCodec |
| **gzip**     | org.apache.hadoop.io.compress.GzipCodec    |
| **bzip2**    | org.apache.hadoop.io.compress.BZip2Codec   |
| **LZO**      | com.hadoop.compression.lzo.LzopCodec       |
| **Snappy**   | org.apache.hadoop.io.compress.SnappyCodec  |

**要在Hadoop****中启用压缩，可以配置如下参数**

| **参数**                                                     |                   **默认值**                   | **阶段**    | **建议**                                      |
| ------------------------------------------------------------ | :--------------------------------------------: | ----------- | --------------------------------------------- |
| **io.compression.codecs (在core-site.xml中配置)**            | 无，这个需要在命令行输入hadoop checknative查看 | 输入压缩    | Hadoop使用文件扩展名判断是否支持某种编解码器  |
| **（在mapred-site.xml中配置）**                              |                     false                      | mapper输出  | 这个参数设为true启用压缩                      |
| **mapreduce.map.output.compress.codec（在mapred-site.xml中配置）** |   org.apache.hadoop.io.compress.DefaultCodec   | mapper输出  | 企业多使用LZO或Snappy编解码器在此阶段压缩数据 |
| **mapreduce.output.fileoutputformat.compress（在mapred-site.xml中配置）** |                     false                      | reducer输出 | 这个参数设为true启用压缩                      |
| **mapreduce.output.fileoutputformat.compress.codec（在mapred-site.xml中配置）** |   org.apache.hadoop.io.compress.DefaultCodec   | reducer输出 | 使用标准工具或者编解码器，如gzip和bzip2       |

## Map输出端采用压缩

```java
// 开启map端输出压缩
		conf.setBoolean("mapreduce.map.output.compress", true);

// 设置map端输出压缩方式
conf.setClass("mapreduce.map.output.compress.codec", BZip2Codec.class,CompressionCodec.class);
```

## Reduce输出端采用压缩

```java
// 设置reduce端输出压缩开启
		FileOutputFormat.setCompressOutput(job, true);

// 设置压缩的方式
 FileOutputFormat.setOutputCompressorClass(job, BZip2Codec.class); 

```

# 小文件存档

需要启动YARN进程

```shell
start-yarn.sh
```

归档文件

把/user/atguigu/input目录里面的所有文件归档成一个叫input.har的归档文件，并把归档后文件存储到/user/atguigu/output路径下。

```shell
hadoop archive -archiveName input.har -p  /user/atguigu/input   /user/atguigu/output
```

查看归档

```shell
hadoop fs -ls /user/atguigu/output/input.har
hadoop fs -ls har:///user/atguigu/output/input.har
```

解归档文件

```shell
hadoop fs -cp har:/// user/atguigu/output/input.har/*    /user/atguigu
```

# 回收站

启用回收站

修改core-site.xml，配置垃圾回收时间为1分钟。

```shell
<property>
    <name>fs.trash.interval</name>
	<value>1</value>
</property>
```

查看回收站

回收站目录在hdfs集群中的路径：/user/atguigu/.Trash/….

通过程序删除的文件不会经过回收站，需要调用moveToTrash()才进入回收站

```shell
//因为本地的客户端拿不到集群的配置信息 所以需要自己手动设置一下回收站
Configuration conf = new Configuration();
conf.set(“fs.trash.interval”,”1”);
conf.set(“fs.trash.checkpoint.interval”,”1”);
//创建一个回收站对象
Trash trash = new Trash(conf);
trash.moveToTrash(path);
```

通过网页上直接删除的文件也不会走回收站。

只有在命令行利用hadoop fs -rm命令删除的文件才会走回收站。

恢复回收站数据

```shell
hadoop fs -mv
/user/atguigu/.Trash/Current/user/atguigu/input    /user/atguigu/input
```

# Yarn生产环境核心参数配置案例

修改yarn-site.xml配置参数如下：

```shell
<!-- 选择调度器，默认容量 -->
<property>
	<description>The class to use as the resource scheduler.</description>
	<name>yarn.resourcemanager.scheduler.class</name>
	<value>org.apache.hadoop.yarn.server.resourcemanager.scheduler.capacity.CapacityScheduler</value>
</property>

<!-- ResourceManager处理调度器请求的线程数量,默认50；如果提交的任务数大于50，可以增加该值，但是不能超过3台 * 4线程 = 12线程（去除其他应用程序实际不能超过8） -->
<property>
	<description>Number of threads to handle scheduler interface.</description>
	<name>yarn.resourcemanager.scheduler.client.thread-count</name>
	<value>8</value>
</property>


<!--
是否将虚拟核数当作CPU核数，默认是false，采用物理CPU核数 
-->
<property>
	<description>Flag to determine if logical processors(such as
	hyperthreads) should be counted as cores. Only applicable on Linux
	when yarn.nodemanager.resource.cpu-vcores is set to -1 and
	yarn.nodemanager.resource.detect-hardware-capabilities is true.
	</description>
	<name>yarn.nodemanager.resource.count-logical-processors-as-cores</name>
	<value>false</value>
</property>

<!-- 是否让yarn自动检测硬件进行配置，默认是false，如果该节点有很多其他应用程序，建议手动配置。如果该节点没有其他应用程序，可以采用自动 -->
<property>
	<description>Enable auto-detection of node capabilities such as
	memory and CPU.
	</description>
	<name>yarn.nodemanager.resource.detect-hardware-capabilities</name>
	<value>false</value>
</property>


<!--
Core转成Vcore的个数（虚拟核数和物理核数乘数，默认是1.0） 
hadoop中的vcore不是真正的core，通常vcore的个数设置为逻辑cpu个数的1~5倍。
-->
<property>
	<description>Multiplier to determine how to convert phyiscal cores to vcores. This value is used if 
yarn.nodemanager.resource.cpu-vcores is set to -1(which implies auto-calculate vcores) and
yarn.nodemanager.resource.detect-hardware-capabilities is set to true. The	number of vcores will be calculated as	number of CPUs * multiplier.
	</description>
	<name>yarn.nodemanager.resource.pcores-vcores-multiplier</name>
	<value>1.0</value>
</property>

<!-- NodeManager使用内存数，默认8G，修改为4G内存 -->
<property>
	<description>Amount of physical memory, in MB, that can be allocated 
	for containers. If set to -1 and
	yarn.nodemanager.resource.detect-hardware-capabilities is true, it is
	automatically calculated(in case of Windows and Linux).
	In other cases, the default is 8192MB.
	</description>
	<name>yarn.nodemanager.resource.memory-mb</name>
	<value>4096</value>
</property>

<!-- nodemanager的CPU核数，不按照硬件环境自动设定时默认是8个，修改为4个 -->
<property>
	<description>Number of vcores that can be allocated
	for containers. This is used by the RM scheduler when allocating
	resources for containers. This is not used to limit the number of
	CPUs used by YARN containers. If it is set to -1 and
	yarn.nodemanager.resource.detect-hardware-capabilities is true, it is
	automatically determined from the hardware in case of Windows and Linux.
	In other cases, number of vcores is 8 by default.</description>
	<name>yarn.nodemanager.resource.cpu-vcores</name>
	<value>4</value>
</property>

<!-- 容器最小内存，默认1G -->
<property>
	<description>The minimum allocation for every container request at the RM	in MBs. Memory requests lower than this will be set to the value of this	property. Additionally, a node manager that is configured to have less memory	than this value will be shut down by the resource manager.
	</description>
	<name>yarn.scheduler.minimum-allocation-mb</name>
	<value>1024</value>
</property>

<!-- 容器最大内存，默认8G，修改为2G -->
<property>
	<description>The maximum allocation for every container request at the RM	in MBs. Memory requests higher than this will throw an	InvalidResourceRequestException.
	</description>
	<name>yarn.scheduler.maximum-allocation-mb</name>
	<value>2048</value>
</property>

<!-- 容器最小CPU核数，默认1个 -->
<property>
	<description>The minimum allocation for every container request at the RM	in terms of virtual CPU cores. Requests lower than this will be set to the	value of this property. Additionally, a node manager that is configured to	have fewer virtual cores than this value will be shut down by the resource	manager.
	</description>
	<name>yarn.scheduler.minimum-allocation-vcores</name>
	<value>1</value>
</property>

<!-- 容器最大CPU核数，默认4个，修改为2个 -->
<property>
	<description>The maximum allocation for every container request at the RM	in terms of virtual CPU cores. Requests higher than this will throw an
	InvalidResourceRequestException.</description>
	<name>yarn.scheduler.maximum-allocation-vcores</name>
	<value>2</value>
</property>

<!-- 虚拟内存检查(限制)，默认打开，修改为关闭 -->
<property>
	<description>Whether virtual memory limits will be enforced for
	containers.</description>
	<name>yarn.nodemanager.vmem-check-enabled</name>
	<value>false</value>
</property>

<!-- 虚拟内存和物理内存设置比例,默认2.1 -->
<property>
	<description>Ratio between virtual memory to physical memory when	setting memory limits for containers. Container allocations are	expressed in terms of physical memory, and virtual memory usage	is allowed to exceed this allocation by this ratio.
	</description>
	<name>yarn.nodemanager.vmem-pmem-ratio</name>
	<value>2.1</value>
</property>
```

分发配置

重启集群

```shell
sbin/stop-yarn.sh
sbin/start-yarn.sh
```

# 配置多队列的容量调度器

在capacity-scheduler.xml中配置如下

```shell
<!-- 指定多队列，增加hive队列 -->
<property>
    <name>yarn.scheduler.capacity.root.queues</name>
    <value>default,hive</value>
    <description>
      The queues at the this level (root is the root queue).
    </description>
</property>

<!-- 降低default队列资源额定容量为40%，默认100% -->
<property>
    <name>yarn.scheduler.capacity.root.default.capacity</name>
    <value>40</value>
</property>

<!-- 降低default队列资源最大容量为60%，默认100% -->
<property>
    <name>yarn.scheduler.capacity.root.default.maximum-capacity</name>
    <value>60</value>
</property>
（2）为新加队列添加必要属性
<!-- 指定hive队列的资源额定容量 -->
<property>
    <name>yarn.scheduler.capacity.root.hive.capacity</name>
    <value>60</value>
</property>

<!-- 用户最多可以使用队列多少资源，1表示所有 -->
<property>
    <name>yarn.scheduler.capacity.root.hive.user-limit-factor</name>
    <value>1</value>
</property>

<!-- 指定hive队列的资源最大容量 -->
<property>
    <name>yarn.scheduler.capacity.root.hive.maximum-capacity</name>
    <value>80</value>
</property>

<!-- 启动hive队列 -->
<property>
    <name>yarn.scheduler.capacity.root.hive.state</name>
    <value>RUNNING</value>
</property>

<!-- 哪些用户有权向队列提交作业 -->
<property>
    <name>yarn.scheduler.capacity.root.hive.acl_submit_applications</name>
    <value>*</value>
</property>

<!-- 哪些用户有权操作队列，管理员权限（查看/杀死） -->
<property>
    <name>yarn.scheduler.capacity.root.hive.acl_administer_queue</name>
    <value>*</value>
</property>

<!-- 哪些用户有权配置提交任务优先级 -->
<property>
    <name>yarn.scheduler.capacity.root.hive.acl_application_max_priority</name>
    <value>*</value>
</property>

<!-- 任务的超时时间设置：yarn application -appId appId -updateLifetime Timeout
参考资料：https://blog.cloudera.com/enforcing-application-lifetime-slas-yarn/ -->

<!-- 如果application指定了超时时间，则提交到该队列的application能够指定的最大超时时间不能超过该值。 
-->
<property>
    <name>yarn.scheduler.capacity.root.hive.maximum-application-lifetime</name>
    <value>-1</value>
</property>

<!-- 如果application没指定超时时间，则用default-application-lifetime作为默认值 -->
<property>
    <name>yarn.scheduler.capacity.root.hive.default-application-lifetime</name>
    <value>-1</value>
</property>
```

重启Yarn或者执行yarn rmadmin -refreshQueues刷新队列，就可以看到两条队列

# 向xxx队列提交任务

## hadoop jar的方式

```shell
hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.3.4.jar wordcount -D mapreduce.job.queuename=xxx /input /output
```

## 打jar包的方式

默认的任务提交都是提交到default队列的。如果希望向其他队列提交任务，需要在Driver中声明：

```java
Configuration conf = new Configuration();
conf.set("mapreduce.job.queuename","xxx");
```



# FIFO、Capacity Scheduler（容量调度器）和Fair Sceduler（公平调度器）各自特点及使用场景

FIFO、Capacity Scheduler（容量调度器）和Fair Sceduler（公平调度器）。

Apache默认的资源调度器是容量调度器。

CDH默认的资源调度器是公平调度器。

**区别**

FIFO调度器：支持单队列 、先进先出，生产环境不会用。

容量调度器：支持多队列。队列资源分配，优先选择资源占用率最低的队列分配资源；作业资源分配，按照作业的优先级和提交时间顺序分配资源；容器资源分配，本地原则（同一节点/同一机架/不同节点不同机架）。

公平调度器：支持多队列，保证每个任务公平享有队列资源。资源不够时可以按照缺额分配。
