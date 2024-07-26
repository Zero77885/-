# Linux相关

# 1.准备干净的Linux

# 2.修改IP地址

```linux
vi /etc/sysconfig/network-scripts/ifcfg-ens33

修改以下
BOOTPROTO=static
ONBOOT=yes
添加如下内容
IPADDR=192.168.10.102
GATEWAY=192.168.10.2
DNS1=114.114.114.114
DNS2=8.8.8.8
```

# 3.重启network服务

关闭NetworkManager

systemctl stop NetworkManager

systemctl disable NetworkManager



systemctl status network

systemctl restart network

测试一下:ping www.baidu.com

# 4.更换yum源

1.在windows上下载好yum源 ： http://mirrors.aliyun.com/repo/Centos-7.repo (阿里巴巴)

2.通过xftp将下载好的yum源传到linux的/etc/yum.repos.d目录中

3.将原Yum源改名: 

```linux
cd /etc/yum.repos.d/
mv CentOS-Base.repo CentOS-Base.repo.bak
把下载好的源放到yum.repos.d目录里
mv yum源的名字 CentOS-Base.repo
```

4.清空缓存并缓存rpm包列表

yum clean all
yum makecache

# 5.安装插件

yum install -y epel-release net-tools vim
yum install -y  psmisc  nc  rsync  lrzsz  ntp libzstd openssl-static tree iotop git

# 6.关闭防火墙

systemctl stop firewalld
systemctl disable firewalld

# 7.创建用户

​	useradd xxxx

# 8.设置密码123321

​	passwd xxxx

# 9.给xxxx设置可以使用root权限

**visudo**
   在root    ALL=(ALL)       ALL下面添加如下内容
​	atguigu ALL=(ALL)       NOPASSWD:ALL

# 10.创建目录

​	mkdir /opt/module
​	mkdir /opt/software

# 11.修改目录的所属主和所属组为xxxxxx

​	chown xxxx:xxxx module software

# 12.修改主机名:hadoop102 (注意名字后面不要加空格)

​	vi /etc/hostname

# 13.修改hosts文件

## 1.修改linux的hosts文件

​		vi /etc/hosts
​		添加如下内容
192.168.10.100 hadoop100
192.168.10.101 hadoop101
192.168.10.102 hadoop102
192.168.10.103 hadoop103
192.168.10.104 hadoop104
192.168.10.105 hadoop105
192.168.10.106 hadoop106
192.168.10.107 hadoop107
192.168.10.108 hadoop108

等等

## 2.修改windows的hosts文件

如果修改保存时显示没有权限。可以将hosts文件复制到桌面
				修改桌面上的hosts文件再把桌面的hosts文件复制回去即可。查看是否成功）
	C:\Windows\System32\drivers\etc
添加如下内容
192.168.10.100 hadoop100
192.168.10.101 hadoop101
192.168.10.102 hadoop102
192.168.10.103 hadoop103
192.168.10.104 hadoop104
192.168.10.105 hadoop105
192.168.10.106 hadoop106
192.168.10.107 hadoop107
192.168.10.108 hadoop108

# 14.等插件都装完后再重启Linux

# 15.不要在用root登录了用创建的普通用户登录

# 



# 16.在hadoop102安装JDK

**1.**卸载现有JDK

注意：安装JDK前，一定确保提前删除了虚拟机自带的JDK。详细步骤见问文档3.1节中卸载JDK步骤。

**2.**用XShell传输工具将JDK导入到opt目录下面的software文件夹下面

**3.**在Linux系统下的opt目录中查看软件包是否导入成功

 ls /opt/software/

看到如下结果：

jdk-8u212-linux-x64.tar.gz

解压JDK到/opt/module目录下**

tar -zxvf jdk-8u212-linux-x64.tar.gz -C /opt/module/

**4.**配置JDK环境变量

新建/etc/profile.d/my_env.sh文件

sudo vim /etc/profile.d/my_env.sh

添加如下内容保存退出

```shell
export JAVA_HOME=/opt/module/jdk1.8.0_212

export PATH=$PATH:$JAVA_HOME/bin
```

source一下/etc/profile文件，让新的环境变量PATH生效

source /etc/profile

**5.**测试JDK是否安装成功

java -version

如果能看到以下结果，则代表Java安装成功。

java version "1.8.0_212"

注意：重启（如果java -version可以用就不用重启）

sudo reboot

# 17.在hadoop102安装hadoop

Hadoop下载地址：[https://archive.apache.org/dist/hadoop/common/hadoop-3.3.4/](https://archive.apache.org/dist/hadoop/common/hadoop-2.7.2/)

**1.**用XShel文件传输工具将hadoop-3.3.4.tar.gz导入到opt目录下面的software文件夹下面

**2.**进入到Hadoop安装包路径下

cd /opt/software/

**3.**解压安装文件到/opt/module下面

 tar -zxvf hadoop-3.3.4.tar.gz -C /opt/module/

**4.**查看是否解压成功

ls /opt/module/hadoop-3.3.4

**5.**将Hadoop添加到环境变量

获取Hadoop安装路径

pwd

**6.**打开/etc/profile.d/my_env.sh文件

sudo vim /etc/profile.d/my_env.sh

在my_env.sh文件末尾添加如下内容：（shift+g）

```shell
export HADOOP_HOME=/opt/module/hadoop-3.3.4

export PATH=$PATH:$HADOOP_HOME/bin

export PATH=$PATH:$HADOOP_HOME/sbin
```

**7.**让修改后的文件生效

source /etc/profile

**8.**测试是否安装成功

hadoop version

**9.**重启（如果Hadoop命令不能用再重启虚拟机）

sudo reboot



# 18.写一个可以互相copy的shell脚本

在/home/用户名/bin 下写 因为它有环境变量

```shell
if [ $# -lt 1 ]
then
    echo Not Enough Arguement!
    exit;
fi

#2. 遍历集群所有机器
for host in hadoop102 hadoop103 hadoop104
do
    echo ====================  $host  ====================
    #3. 遍历所有目录，挨个发送
    for file in $@
    do
        #4. 判断文件是否存在
        if [ -e $file ]
            then
                #5. 获取父目录
                pdir=$(cd -P $(dirname $file); pwd)

                #6. 获取当前文件的名称
                fname=$(basename $file)
                ssh $host "mkdir -p $pdir"
                rsync -av $pdir/$fname $host:$pdir
            else
                echo $file does not exists!
        fi
    done
done

```

修改脚本 xsync 具有执行权限

chmod 777 xsync

# 19.克隆



# 20修改克隆出来虚拟机的Ip和hostname

vi /etc/sysconfig/network-scripts/ifcfg-ens33

vi /etc/hostname

# 21.SSH无密登录配置

**1.**生成公钥和私钥

ssh-keygen -t rsa

**2.**将公钥拷贝到要免密登录的目标机器上

ssh-copy-id hadoop102

# 22.配置核心配置文件

**在 hadoop 路径下的 /etc/hadoop下配置下面五项**

## core-site.xml

```shell
 <!-- 指定NameNode的地址 -->
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://hadoop102:8020</value>
    </property>

    <!-- 指定hadoop数据的存储目录 -->
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/opt/module/hadoop-3.3.4/data</value>
    </property>

    <!-- 配置HDFS网页登录使用的静态用户为atguigu -->
    <property>
        <name>hadoop.http.staticuser.user</name>
        <value>atguigu</value>
    </property>
```



## hdfs-site.xml

```shell
<!-- nn web端访问地址-->
	<property>
        <name>dfs.namenode.http-address</name>
        <value>hadoop102:9870</value>
    </property>
	<!-- 2nn web端访问地址-->
    <property>
        <name>dfs.namenode.secondary.http-address</name>
        <value>hadoop104:9868</value>
    </property>
```

## yarn-site.xml

```shell
 <!-- 指定MR走shuffle -->
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>

    <!-- 指定ResourceManager的地址-->
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>hadoop103</value>
    </property>

    <!-- 环境变量的继承 -->
    <property>
        <name>yarn.nodemanager.env-whitelist</name>
        <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
    </property>
<!-- 开启日志聚集功能 -->
<property>
<name>yarn.log-aggregation-enable</name>
<value>true</value>
</property>

<!-- 设置日志聚集服务器地址 -->
<property>  
<name>yarn.log.server.url</name>  
<value>http://hadoop102:19888/jobhistory/logs</value>
</property>

<!-- 设置日志保留时间为7天 -->
<property>
<name>yarn.log-aggregation.retain-seconds</name>
<value>604800</value>
</property>
	
```



## mapred-site.xml

```shell
<!-- 指定MapReduce程序运行在Yarn上 -->
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
   </property>
   <!-- 历史服务器端地址 -->
<property>
<name>mapreduce.jobhistory.address</name>
<value>hadoop102:10020</value>
</property>

<!-- 历史服务器web端地址 -->
<property>
<name>mapreduce.jobhistory.webapp.address</name>
<value>hadoop102:19888</value>
</property>
```



## workers

```shell
hadoop102
hadoop103
hadoop104
```



# 23.在集群上分发配置好的Hadoop配置文件

xsync /opt/module/hadoop-3.3.4/etc/hadoop/

# 24.第一次启动需要在Namenode节点格式化NameNode

hdfs namenode -format

**注意：格式化NameNode，会产生新的集群id，导致NameNode和DataNode的集群id不一致，集群找不到已往数据。如果集群在运行过程中报错，需要重新格式化NameNode的话，一定要先停止namenode和datanode进程，并且要删除所有机器的data和logs目录，然后再进行格式化。**

# 25.启动/关闭HDFS(在NameNode节点)

脚本myhadoop

```java
#!/bin/bash
#校验参数的个数
if [ $# -eq 0 ];then
	echo 参数的个数不对!!!
	exit
fi

#校验参数的内容
case $1 in
"start")
	#启动HDFS
	ssh hadoop102 start-dfs.sh
	#启动YARN
	ssh hadoop103 start-yarn.sh
    
    ssh hadoop102 mapred --daemon start historyserver
	;;
"stop")
	#关闭HDFS
	ssh hadoop102 stop-dfs.sh
	#关闭YARN
	ssh hadoop103 stop-yarn.sh
    
    ssh hadoop102 mapred --daemon stop historyserver
	;;
*)
	echo 参数的内容不对!!!
	;;
esac
```



启动:start-dfs.sh

关闭:stop-dfs.sh



jpsall脚本

```shell
#!/bin/bash

for host in hadoop102 hadoop103 hadoop104
do
	echo =================$host=====================
	ssh $host jps
done
```





# 26.启动/关闭yarn(在**ResourceManager**节点)

启动:start-yarn.sh

关闭:stop-yarn.sh

# 27.查看hadoop页面

HDFS的页面: http://hadoop102:9870/
YARN的页面: http://hadoop103:8088

看不到页面怎么解决？
		①查看hadoop集群是否启动了还得看是正常
		②查看是否关闭了防火墙-必须关
			systemctl status firewalld
			systemctl stop firewalld
			systemctl disable firewalld
		③将http://hadoop102:9870/地址中的hadoop102替换成192.168.10.102
			如果替换后可以看得到说明windows的hosts有问题---有可能被杀毒软件注释掉了

# 28.添加历史服务器

```shell
查看历史服务
		http://hadoop102:19888
```

# 29.日志的聚集---用来在历史服务器中查看job执行的日志信息（非集群运行的日志信息）



## 1.再次执行job -- 之前的都不会再生效

​		**hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.3.4.jar wordcount /input /output2**
再次查看历史服务器并查看logs
​	http://hadoop102:198888
​	点击对应的job再点击logs

# 30.时间同步

将hadoop102配置成时间服务器

**1.**停止ntpd服务 ： sudo systemctl stop ntpd
**2.**修改配置文件 /etc/ntp.conf
sudo vim /etc/ntp.conf
将#restrict 192.168.1.0 mask 255.255.255.0 nomodify notrap的注释去掉并改成10
restrict 192.168.10.0 mask 255.255.255.0 nomodify notrap

将  

server 0.centos.pool.ntp.org iburst
server 1.centos.pool.ntp.org iburst
server 2.centos.pool.ntp.org iburst
server 3.centos.pool.ntp.org iburst 

全部加上注释-不让去时间服务器同步时间

添加如下内容
```shell
server 127.127.1.0
fudge 127.127.1.0 stratum 10
```

**3.**修改配置文件/etc/sysconfig/ntpd 
sudo vim /etc/sysconfig/ntpd 
添加如下内容

```shell
SYNC_HWCLOCK=yes
```

**4.**启动ntpd服务
sudo systemctl start ntpd
sudo systemctl enable ntpd

---------------------------------------------------------
		如果需要时间服务器去互联网同步时间 ①关闭ntpd服务 ②同步时间sudo ntpdate xxxx.xxx.xxx
					③再次开启ntpd服务
		---------------------------------------------------------

其它节点去hadoop102同步时间

ntpdate hadoop102

**或开启定时任务**
			sudo systemctl start crond
编辑定时任务
			sudo crontab -e
添加如下内容 -- 只是为了演示才每分钟同步一次
			*/1 * * * * sudo ntpdate hadoop102









# 配置windows上的hadoop环境

1.将windows版的hadoop依赖 --- \06-Hadoop\2.资料\03_Windows依赖\hadoop-3.1.0
	拷贝到D:\hadoop中
	
2.添加环境变量
	添加HADOOP_HOME
	将HADOOP_HOME添加到PATH中
	
3.双击hadoop-3.1.0\bin\winutils.exe 如果有错误弹出-需要安装
	资料中的微软运行库 --- 安装后双击还报错可以考虑重启windows
	
4.将hadoop-3.1.0\bin 下的  winutils.exe 和 hadoop.dll 拷贝到C:\Windows\System32