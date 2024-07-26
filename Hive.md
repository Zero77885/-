# Hive官网地址

http://hive.apache.org/

**文档查看地址**：https://cwiki.apache.org/confluence/display/Hive/GettingStarted

**下载地址：**http://archive.apache.org/dist/hive/

**github地址：**https://github.com/apache/hive

# Hive安装部署

把apache-hive-3.1.3-bin.tar.gz上传到Linux的/opt/software目录下

解压apache-hive-3.1.3-bin.tar.gz到/opt/module/目录下面

```linux
tar -zxvf /opt/software/apache-hive-3.1.3-bin.tar.gz -C /opt/module/
```

修改apache-hive-3.1.3-bin.tar.gz的名称为hive

```linux
mv /opt/module/apache-hive-3.1.3-bin/ /opt/module/hive
```

修改/etc/profile.d/my_env.sh，添加环境变量

```linux
sudo vim /etc/profile.d/my_env.sh
```

```shell
#HIVE_HOME
export HIVE_HOME=/opt/module/hive
export PATH=$PATH:$HIVE_HOME/bin
```

生效环境变量

```linux
source /etc/profile.d/my_env.sh
```

初始化元数据库（默认是derby数据库）

```shell
#hive根目录下
schematool -dbType derby -initSchema
```

# 使用Hive

#hive根目录下输入hive

# MySQL安装

上传MySQL安装包以及MySQL驱动jar包

解压MySQL安装包

```linux
mkdir mysql_lib
tar -xvf mysql-5.7.28-1.el7.x86_64.rpm-bundle.tar -C mysql_lib/
```

卸载系统自带的mariadb

```linux
sudo rpm -qa | grep mariadb | xargs sudo rpm -e --nodeps
```

安装MySQL依赖

```linux
cd mysql_lib

sudo rpm -ivh mysql-community-common-5.7.28-1.el7.x86_64.rpm

sudo rpm -ivh mysql-community-libs-5.7.28-1.el7.x86_64.rpm

sudo rpm -ivh mysql-community-libs-compat-5.7.28-1.el7.x86_64.rpm
```

安装mysql-client

```linux
sudo rpm -ivh mysql-community-client-5.7.28-1.el7.x86_64.rpm
```

安装mysql-server

```linux
sudo rpm -ivh mysql-community-server-5.7.28-1.el7.x86_64.rpm
```

**注意：若出现以下错误**

```shell
warning: 05_mysql-community-server-5.7.16-1.el7.x86_64.rpm: Header V3 DSA/SHA1 Signature, key ID 5072e1f5: NOKEY
error: Failed dependencies:
libaio.so.1()(64bit) is needed by mysql-community-server-5.7.16-1.el7.x86_64

**解决办法****：**

sudo yum -y install libaio
```

## 启动MySQL

```linux
sudo systemctl start mysqld
```

查看MySQL密码

```linux
sudo cat /var/log/mysqld.log | grep password
```

## 配置MySQL

用刚刚查到的密码进入MySQL（如果报错，给密码加单引号）

```linux
mysql -uroot -p'password'
```

更改MySQL密码策略

```linux
set global validate_password_policy=0;
set global validate_password_length=4;
```

设置简单好记的密码

```linux
set password=password("123456");
```

进入MySQL库

```linux
use mysql
```

查询user表

```linux
select user, host from user;
```

修改user表，把Host表内容修改为%

```linux
update user set host="%" where user="root";
```

刷新

```linux
flush privileges;
```

退出

```linux
quit;
```

## 卸载MySQL

若因为安装失败或者其他原因，MySQL需要卸载重装，可参考以下内容

**清空原有数据**

通过/etc/my.cnf查看MySQL数据的存储位置

```linux
sudo cat /etc/my.cnf
```

去往/var/lib/mysql路径需要root权限

```linux
mysql]$ su - root

cd /var/lib/mysql

rm -rf *  （注意敲击命令的位置）
```

### 卸载MySQL相关包

查看安装过的MySQL相关包

```linux
sudo rpm -qa | grep -i -E mysql
```

一键卸载命令

```linux
rpm -qa | grep -i -E mysql\|mariadb | xargs -n1 sudo rpm -e --nodeps
```

# 配置元数据到MySQL

新建Hive元数据库

```linux
#登录MySQL

mysql -u root -p 123456

#创建Hive元数据库
mysql> create database metastore;
mysql> quit;
```

将MySQL的JDBC驱动拷贝到Hive的lib目录下。

```linux
cp /opt/software/mysql-connector-java-5.1.37.jar $HIVE_HOME/lib
```

在$HIVE_HOME/conf目录下新建hive-site.xml文件

```linux
vim $HIVE_HOME/conf/hive-site.xml
```

添加如下内容：

```shell
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>
    <!-- jdbc连接的URL -->
    <property>
        <name>javax.jdo.option.ConnectionURL</name>
        <value>jdbc:mysql://hadoop102:3306/metastore?useSSL=false</value>
    </property>
    
    <!-- jdbc连接的Driver-->
    <property>
        <name>javax.jdo.option.ConnectionDriverName</name>
        <value>com.mysql.jdbc.Driver</value>
    </property>
    
	<!-- jdbc连接的username-->
    <property>
        <name>javax.jdo.option.ConnectionUserName</name>
        <value>root</value>
    </property>

    <!-- jdbc连接的password -->
    <property>
        <name>javax.jdo.option.ConnectionPassword</name>
        <value>123456</value>
    </property>

    <!-- Hive默认在HDFS的工作目录 -->
    <property>
        <name>hive.metastore.warehouse.dir</name>
        <value>/user/hive/warehouse</value>
    </property>
</configuration>
```

初始化Hive元数据库（修改为采用MySQL存储元数据）

```linux
bin/schematool -dbType mysql -initSchema -verbose
```

# Hive服务部署

##  hiveserver2服务

Hive的hiveserver2服务的作用是提供jdbc/odbc接口，为用户提供远程访问Hive数据的功能，例如用户期望在个人电脑中访问远程服务中的Hive数据，就需要用到Hiveserver2。

### Hadoop端配置

修改配置文件core-site.xml，然后记得分发三台机器

```shell
cd $HADOOP_HOME/etc/hadoop
vim core-site.xml
```

增加如下配置：

```shell
<!--配置所有节点的atguigu用户都可作为代理用户-->
<property>
    <name>hadoop.proxyuser.atguigu.hosts</name>
    <value>*</value>
</property>

<!--配置atguigu用户能够代理的用户组为任意组-->
<property>
    <name>hadoop.proxyuser.atguigu.groups</name>
    <value>*</value>
</property>

<!--配置atguigu用户能够代理的用户为任意用户-->
<property>
    <name>hadoop.proxyuser.atguigu.users</name>
    <value>*</value>
</property>
```

### Hive端配置

在hive-site.xml文件中添加如下配置信息

```linux
vim hive-site.xml
```

```shell
<!-- 指定hiveserver2连接的host -->
<property>
	<name>hive.server2.thrift.bind.host</name>
	<value>hadoop102</value>
</property>

<!-- 指定hiveserver2连接的端口号 -->
<property>
	<name>hive.server2.thrift.port</name>
	<value>10000</value>
</property>
```

### 启动hiveserver2命令

```linux
hive --service hiveserver2
```

### 使用命令行客户端beeline进行远程访问

```linux
beeline -u jdbc:hive2://hadoop102:10000 -n atguigu
```

## metastore服务

Hive的metastore服务的作用是为Hive CLI或者Hiveserver2提供元数据访问接口。

metastore有两种运行模式，分别为嵌入式模式和独立服务模式。下面分别对两种模式进行说明：

### 嵌入式模式

嵌入式模式下，只需保证Hiveserver2和每个Hive CLI的配置文件hive-site.xml中包含连接元数据库所需要的以下参数即可：

```shell
 <!-- jdbc连接的URL -->
    <property>
        <name>javax.jdo.option.ConnectionURL</name>
        <value>jdbc:mysql://hadoop102:3306/metastore?useSSL=false</value>
    </property>
    
    <!-- jdbc连接的Driver-->
    <property>
        <name>javax.jdo.option.ConnectionDriverName</name>
        <value>com.mysql.jdbc.Driver</value>
    </property>
    
	<!-- jdbc连接的username-->
    <property>
        <name>javax.jdo.option.ConnectionUserName</name>
        <value>root</value>
    </property>

    <!-- jdbc连接的password -->
    <property>
        <name>javax.jdo.option.ConnectionPassword</name>
        <value>123456</value>
    </property>
```

### 独立服务模式

独立服务模式需做以下配置：

首先，保证metastore服务的配置文件hive-site.xml中包含连接元数据库所需的以下参数：

```shell
 <!-- jdbc连接的URL -->
    <property>
        <name>javax.jdo.option.ConnectionURL</name>
        <value>jdbc:mysql://hadoop102:3306/metastore?useSSL=false</value>
    </property>
    
    <!-- jdbc连接的Driver-->
    <property>
        <name>javax.jdo.option.ConnectionDriverName</name>
        <value>com.mysql.jdbc.Driver</value>
    </property>
    
	<!-- jdbc连接的username-->
    <property>
        <name>javax.jdo.option.ConnectionUserName</name>
        <value>root</value>
    </property>

    <!-- jdbc连接的password -->
    <property>
    <name>javax.jdo.option.ConnectionPassword</name>
        <value>123456</value>
    </property>
```

其次，保证Hiveserver2和每个Hive CLI的配置文件hive-site.xml中包含访问metastore服务所需的以下参数：

```shell
<!-- 指定metastore服务的地址 -->
<property>
	<name>hive.metastore.uris</name>
	<value>thrift://hadoop102:9083</value>
</property>
```

**注意：**主机名需要改为metastore服务所在节点，端口号无需修改，metastore服务的默认端口就是9083。

### metastore服务的启动命令：

```linux
hive --service metastore
```

# Hive服务启动脚本

```shell
#!/bin/bash

HIVE_LOG_DIR=$HIVE_HOME/logs
if [ ! -d $HIVE_LOG_DIR ]
then
	mkdir -p $HIVE_LOG_DIR
fi

#检查进程是否运行正常，参数1为进程名，参数2为进程端口
function check_process()
{
    pid=$(ps -ef 2>/dev/null | grep -v grep | grep -i $1 | awk '{print $2}')
    ppid=$(netstat -nltp 2>/dev/null | grep $2 | awk '{print $7}' | cut -d '/' -f 1)
    echo $pid
    [[ "$pid" =~ "$ppid" ]] && [ "$ppid" ] && return 0 || return 1
}

function hive_start()
{
    metapid=$(check_process HiveMetastore 9083)
    cmd="nohup hive --service metastore >$HIVE_LOG_DIR/metastore.log 2>&1 &"
    [ -z "$metapid" ] && eval $cmd || echo "Metastroe服务已启动"
    server2pid=$(check_process HiveServer2 10000)
    cmd="nohup hive --service hiveserver2 >$HIVE_LOG_DIR/hiveServer2.log 2>&1 &"
    [ -z "$server2pid" ] && eval $cmd || echo "HiveServer2服务已启动"
}

function hive_stop()
{
metapid=$(check_process HiveMetastore 9083)
    [ "$metapid" ] && kill $metapid || echo "Metastore服务未启动"
    server2pid=$(check_process HiveServer2 10000)
    [ "$server2pid" ] && kill $server2pid || echo "HiveServer2服务未启动"
}

case $1 in
"start")
    hive_start
    ;;
"stop")
    hive_stop
    ;;
"restart")
    hive_stop
    sleep 2
    hive_start
    ;;
"status")
    check_process HiveMetastore 9083 >/dev/null && echo "Metastore服务运行正常" || echo "Metastore服务运行异常"
    check_process HiveServer2 10000 >/dev/null && echo "HiveServer2服务运行正常" || echo "HiveServer2服务运行异常"
    ;;
*)
    echo Invalid Args!
    echo 'Usage: '$(basename $0)' start|stop|restart|status'
    ;;
esac
```

# 外部表和管理表有什么区别

删除管理表时会将元数据和HDFS上的数据全部删除掉。

删除外部表时只会删除元数据。

truncate table时只能清空管理表。清空外部表时会报错。









