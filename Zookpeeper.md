# Zookpeeper官网首页：

https://zookeeper.apache.org/

# 安装

安装JDK

拷贝apache-zookeeper-3.7.1-bin.tar.gz安装包到Linux系统下

解压到指定目录

```linux
tar -zxvf apache-zookeeper-3.7.1-bin.tar.gz -C /opt/module/
```

将conf这个路径下的zoo_sample.cfg修改为zoo.cfg；

打开zoo.cfg文件，修改dataDir路径,并在路径下创建dataDir文件夹

zkData目录下创建一个myid的文件

在文件中添加与server对应的编号（注意：上下不要有空行，左右不要有空格）

拷贝配置好的zookeeper到其他机器上,并修改Myid的内容

打开zoo.cfg文件,增加如下内容

```shell
#######################cluster##########################
server.2=hadoop102:2888:3888
server.3=hadoop103:2888:3888
server.4=hadoop104:2888:3888
```

同步zoo.cfg配置文件

分别启动ZooKeeper

设置环境变量

# 启动停止脚本

```shell
#!/bin/bash

case $1 in
"start"){
	for i in hadoop102 hadoop103 hadoop104
	do
        echo ---------- zookeeper $i 启动 ------------
		ssh $i "/opt/module/zookeeper-3.7.1/bin/zkServer.sh start"
	done
};;
"stop"){
	for i in hadoop102 hadoop103 hadoop104
	do
        echo ---------- zookeeper $i 停止 ------------    
		ssh $i "/opt/module/zookeeper-3.7.1/bin/zkServer.sh stop"
	done
};;
"status"){
	for i in hadoop102 hadoop103 hadoop104
	do
        echo ---------- zookeeper $i 状态 ------------    
		ssh $i "/opt/module/zookeeper-3.7.1/bin/zkServer.sh status"
	done
};;
esac
```



# 启动ZooKeeper

zkServer.sh start

# 查看状态

zkServer.sh status

# 启动客户端

zkCli.sh

# 退出客户端

quit

# 停止ZooKeeper

zkServer.sh stop

# 客户端命令行操作

## 命令行语法

| **命令基本语法** | **功能描述**                                                 |
| ---------------- | ------------------------------------------------------------ |
| **help**         | 显示所有操作命令                                             |
| **l****s path**  | 使用 ls 命令来查看当前znode的子节点 [可监听]  -w 监听子节点变化  -s  附加次级信息 |
| **create**       | 普通创建  -s 含有序列  -e 临时（重启或者超时消失）           |
| **get  path**    | 获得节点的值 [可监听]  -w 监听节点内容变化  -s  附加次级信息 |
| **set**          | 设置节点的具体值                                             |
| **stat**         | 查看节点状态                                                 |
| **delete**       | 删除节点                                                     |
| **deleteall**    | 递归删除节点                                                 |

## 显示所有操作命令

help

# leader选举机制

## 第一次启动

(1)服务器1启动,发起第一次选举.服务器1投自己一票.此时服务器1的票数是一票,不够半数以上,选举无法完成,服务器一状态保持为Looking;

(2)服务器2启动,在发起一次选举,服务器1和服务器2分别投自己一票并交换选票信息:此时服务器1发现服务器2的**myid**比自己的大,更改投票为服务器2,此时服务器1为0票,服务器2为2票,超过半数以上的结果,选举完成,服务器2当选leader,服务器2的状态改为Leading.服务器一的状态改为Following.

(3)服务器3启动,发起一次选举,此时服务器1,2已经不是Looking状态,不会更改选票结果.

## 非第一次启动

(1)当ZooKeeper集群中的一台服务器出现以下两种情况之一时,会开始进入Leader选举:

- 服务器初始化启动
- 服务器运行期间无法和Leader保持连接.

(2)当一台机器进入Leader选举流程时,当前集群也可能会处于以下两种状态:

- 集群中本就已经存在一个Leader.

对于第一种已经存在Leader的情况,机器试图去选举Leader时,会被告知当前服务器的Leader信息,对于该机器来说,仅仅需要和Leader机器建立连接,并进行状态同步即可.

- 集群中确实不存在Leader

那么就会重新重新选举

## 选举Leader规则

Epoch(每个Leader任期的代号)大的直接胜出

Epoch相同,事务ID(ZXID)大的胜出

事务ID相同,服务器ID(SID和myid一致)大的胜出

