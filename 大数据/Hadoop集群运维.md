# Hadoop集群运维

# 脚本配置

## oracle脚本

```bash
touch oracle.sh

vim oracle.sh

#!/bin/bash

# 设置 Oracle 环境变量
export ORACLE_HOME=/data/u01/app/oracle/product/19/db_1
export ORACLE_SID=cesdb

case $1 in
"start"){
    echo "============== 启动 Oracle 数据库 =============="
    # 启动监听器
    $ORACLE_HOME/bin/lsnrctl start

    # 启动数据库
    $ORACLE_HOME/bin/sqlplus / as sysdba <<EOF
    startup;
    exit;
EOF

    echo "============== Oracle 数据库已启动 =============="
}
;;
"stop"){
    echo "============== 关闭 Oracle 数据库 =============="
    # 关闭数据库
    $ORACLE_HOME/bin/sqlplus / as sysdba <<EOF
    shutdown immediate;
    exit;
EOF

    # 停止监听器
    $ORACLE_HOME/bin/lsnrctl stop

    echo "============== Oracle 数据库已关闭 =============="
}
;;
*)
    echo "Usage: $0 {start|stop}"
    exit 1
;;
esac

chomd -R 777 oracle.sh

# 切换oralce用户
su oracle
# 启动oracle
oracle.sh start
# 关闭oracle
oracle.sh stop
```



## jps脚本

```shell
cd /data/chenli/sh

touch jps.sh
vim jps.sh

#!/bin/bash
echo "==============查询当前所有服务器的jps情况=============="
        for i in cesdb cesdb1 cesdb2
        do
                echo "**************$i当前jps情况***************"
                ssh $i '/usr/jdk/bin/jps'
        done

echo "=======================查询完毕========================"

chmod -R 777 jps.sh

vim /etc/profile

# jps.sh的脚本路径(添加到最后一行)
export PATH=$PATH:/data/chenli/sh

source /etc/profile

# 然后就可以全局使用了
[hadoop@cesdb sh]$ jps.sh
==============查询当前所有服务器的jps情况==============
**************cesdb当前jps情况***************
10113 DataNode
10787 ResourceManager
9893 NameNode
11014 NodeManager
149652 Jps
13933 RunJar
13934 RunJar
**************cesdb1当前jps情况***************
155888 DataNode
33810 Jps
156072 SecondaryNameNode
156361 NodeManager
**************cesdb2当前jps情况***************
136547 NodeManager
136208 DataNode
13707 Jps
=======================查询完毕========================

```

## Hadoop集群(这个脚本有问题，需要修改)

```bash
touch hadoop-all.sh

vim hadoop-all.sh

#!/bin/bash
if [ $# -lt 1 ]
then
 	echo "No Args Input..."
 	exit ;
fi
case $1 in
"start")
 	echo " =================== 启动 hadoop 集群 ==================="
 	echo " --------------- 启动 hdfs ---------------"
 	ssh hadoop@cesdb "/data/u01/app/hadoop/hadoop-3.3.6/sbin/start-dfs.sh"
 	echo " --------------- 启动 yarn ---------------"
 	ssh hadoop@cesdb "/data/u01/app/hadoop/hadoop-3.3.6/sbin/start-yarn.sh"
 	echo " --------------- 启动 historyserver ---------------"
 	ssh hadoop@cesdb2 "/data/u01/app/hadoop/hadoop-3.3.6/sbin/mr-jobhistory-daemon.sh start historyserver"
;;
"stop")
 	echo " =================== 关闭 hadoop 集群 ==================="
    echo " --------------- 关闭 historyserver ---------------"
 	ssh hadoop@cesdb2 "/data/u01/app/hadoop/hadoop-3.3.6/sbin/mr-jobhistory-daemon.sh stop historyserver"
 	echo " --------------- 关闭 yarn ---------------"
 	ssh hadoop@cesdb "/data/u01/app/hadoop/hadoop-3.3.6/sbin/stop-yarn.sh"
 	echo " --------------- 关闭 hdfs ---------------"
 	ssh hadoop@cesdb "/data/u01/app/hadoop/hadoop-3.3.6/sbin/stop-dfs.sh"
;;
*)
 	echo "Input Args Error..."
;;
esac


chmod -R 777 hadoop-all.sh

# 切换Hadoop用户运行
su hadoop

# 启动hadoop集群
hadoop-all.sh start

# 停止hadoop集群
hadoop-all.sh stop

```

## Hive脚本

```bash
touch hive.sh

vim hive.sh

#!/bin/bash

# 定义Hive的安装路径
HIVE_HOME="/data/u01/app/hive/apache-hive-3.1.3/"

# 获取脚本第一个参数（start 或 stop）
case $1 in
"start")
    echo "============== 启动 Hive =============="
    echo "************** 启动 Hive Metastore ***************"
    nohup $HIVE_HOME/bin/hive --service metastore > $HIVE_HOME/logs/hive_metastore.log 2>&1 &
    
    echo "************** 启动 HiveServer2 ***************"
    nohup $HIVE_HOME/bin/hive --service hiveserver2 > $HIVE_HOME/logs/hive_hiveserver2.log 2>&1 &
    
    echo "============== Hive 服务已启动 =============="
    ;;
"stop")
    echo "============== 关闭 Hive =============="
    echo "************** 关闭 Hive 服务 ***************"
    jps | grep -E 'RunJar|HiveMetaStore|HiveServer2' | awk '{print $1}' | xargs -r kill -9
    
    echo "============== Hive 服务已关闭 =============="
    ;;
*)
    echo "Usage: $0 {start|stop}"
    exit 1
esac

chmod -R 777 hive.sh

# 切换hadoop用户
su hadoop
# 启动hive
hive.sh start
# 关闭hive
hive.sh stop
```



## Zookeeper集群脚本

```bash
touch zk-all.sh
vim zk-all.sh

#!/bin/bash

case $1 in
"start"){
    #遍历集群所有机器
	for i in cesdb cesdb1 cesdb2
	do
		#控制台输出日志
		echo =============zookeeper $i 启动====================
		#启动命令
		ssh $i "/data/u01/app/zookeeper/zookeeper-3.8.4/bin/zkServer.sh start"
	done
}
;;
"stop"){
	for i in cesdb cesdb1 cesdb2
	do
		echo =============zookeeper $i 停止====================
		ssh $i "/data/u01/app/zookeeper/zookeeper-3.8.4/bin/zkServer.sh stop"
	done
}
;;
"status"){
	for i in cesdb cesdb1 cesdb2
	do
		echo =============查看 zookeeper $i 状态====================
		ssh $i "/data/u01/app/zookeeper/zookeeper-3.8.4/bin/zkServer.sh status"
	done
}
;;
esac

chmod -R 777 zk-all.sh

#使用root用户
su root
# 启动zookeeper集群
zk-all.sh start

# 停止zookeeper集群
zk-all.sh stop

# 可以查询集群各节点的状态跟角色信息
zk-all.sh status
```

## Nifi集群

```bash
touch nifi-all.sh
vim nifi-all.sh

#!/bin/bash

case $1 in
"start"){
    #遍历集群所有机器
	for i in cesdb cesdb1 cesdb2
	do
		#控制台输出日志
		echo =============nifi $i 启动====================
		#启动命令
		ssh $i "/data/u01/app/nifi/nifi-1.26.0/bin/nifi.sh start"
	done
}
;;
"stop"){
	for i in cesdb cesdb1 cesdb2
	do
		echo =============nifi $i 停止====================
		ssh $i "/data/u01/app/nifi/nifi-1.26.0/bin/nifi.sh stop"
	done
}
;;
"status"){
	for i in cesdb cesdb1 cesdb2
	do
		echo =============查看 nifi $i 状态====================
		ssh $i "/data/u01/app/nifi/nifi-1.26.0/bin/nifi.sh status"
	done
}
;;
esac

chmod -R 777 nifi-all.sh

#使用root用户
su root
# 启动zookeeper集群
nifi-all.sh start

# 停止zookeeper集群
nifi-all.sh stop

# 可以查询集群各节点的状态跟角色信息
nifi-all.sh status
```

## ds集群

```bash
touch ds-all.sh

vim ds-all.sh

#!/bin/bash

# 设置环境变量
export DS_HOME=/data/u01/app/dolphinscheduler/apache-dolphinscheduler-3.2.2-bin

# 检查输入参数
if [ "$#" -ne 1 ]; then
    echo "Usage: $0 {start|stop}"
    exit 1
fi

ACTION=$1

if [ "$ACTION" == "start" ]; then
    echo "Starting DolphinScheduler components..."

    # 启动 Master（cesdb 和 cesdb1）
    ssh dolphinscheduler@cesdb "cd $DS_HOME && bash ./bin/dolphinscheduler-daemon.sh start master-server"
    ssh dolphinscheduler@cesdb1 "cd $DS_HOME && bash ./bin/dolphinscheduler-daemon.sh start master-server"

    # 启动 Worker（cesdb、cesdb1 和 cesdb2）
    ssh dolphinscheduler@cesdb "cd $DS_HOME && bash ./bin/dolphinscheduler-daemon.sh start worker-server"
    ssh dolphinscheduler@cesdb1 "cd $DS_HOME && bash ./bin/dolphinscheduler-daemon.sh start worker-server"
    ssh dolphinscheduler@cesdb2 "cd $DS_HOME && bash ./bin/dolphinscheduler-daemon.sh start worker-server"

    # 启动 Api（cesdb）
    ssh dolphinscheduler@cesdb "cd $DS_HOME && bash ./bin/dolphinscheduler-daemon.sh start api-server"

    # 启动 Alert（cesdb2）
    ssh dolphinscheduler@cesdb2 "cd $DS_HOME && bash ./bin/dolphinscheduler-daemon.sh start alert-server"

    echo "DolphinScheduler components have been started."

elif [ "$ACTION" == "stop" ]; then
    echo "Stopping DolphinScheduler components..."

    # 停止 Master（cesdb 和 cesdb1）
    ssh dolphinscheduler@cesdb "cd $DS_HOME && bash ./bin/dolphinscheduler-daemon.sh stop master-server"
    ssh dolphinscheduler@cesdb1 "cd $DS_HOME && bash ./bin/dolphinscheduler-daemon.sh stop master-server"

    # 停止 Worker（cesdb、cesdb1 和 cesdb2）
    ssh dolphinscheduler@cesdb "cd $DS_HOME && bash ./bin/dolphinscheduler-daemon.sh stop worker-server"
    ssh dolphinscheduler@cesdb1 "cd $DS_HOME && bash ./bin/dolphinscheduler-daemon.sh stop worker-server"
    ssh dolphinscheduler@cesdb2 "cd $DS_HOME && bash ./bin/dolphinscheduler-daemon.sh stop worker-server"

    # 停止 Api（cesdb）
    ssh dolphinscheduler@cesdb "cd $DS_HOME && bash ./bin/dolphinscheduler-daemon.sh stop api-server"

    # 停止 Alert（cesdb2）
    ssh dolphinscheduler@cesdb2 "cd $DS_HOME && bash ./bin/dolphinscheduler-daemon.sh stop alert-server"

    echo "DolphinScheduler components have been stopped."

else
    echo "Invalid action: $ACTION"
    echo "Usage: $0 {start|stop}"
    exit 1
fi

chmod -R 777 ds-all.sh

# 启动集群
ds-all.sh start
# 关闭集群
ds-all.sh stop
```



