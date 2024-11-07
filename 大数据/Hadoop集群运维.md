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

## 自动化监控HiveServer2服务

1.脚本

```bash
#!/bin/bash

# 设置Hive的安装路径
HIVE_HOME=/data/u01/app/hive/apache-hive-3.1.3

# 日志目录设置
LOG_DIR="/data/u01/app/hive/apache-hive-3.1.3/logs/hiveserver2_monitor"
mkdir -p $LOG_DIR  # 创建日志目录（如果不存在）
DATE=$(date +%Y-%m-%d)
LOG_FILE="$LOG_DIR/hiveserver2_monitor_$DATE.log"

# 删除两天前的日志文件
find $LOG_DIR -type f -name "hiveserver2_monitor_*.log" -mtime +2 -exec rm {} \;

# 检查hiveserver2是否运行的函数
function is_hiveserver2_running() {
    jps | grep -q "RunJar"
    return $?
}

# 关闭所有RunJar服务
function stop_hiveserver2_services() {
    jps | grep "RunJar" | awk '{print $1}' | xargs kill -9
    echo "$(date '+%Y-%m-%d %H:%M:%S') - Stopped all RunJar services." >> $LOG_FILE
}

# 启动metastore和hiveserver2服务
function start_hiveserver2_services() {
    nohup $HIVE_HOME/bin/hive --service metastore &>> $LOG_FILE &
    nohup $HIVE_HOME/bin/hive --service hiveserver2 &>> $LOG_FILE &
    echo "$(date '+%Y-%m-%d %H:%M:%S') - Started metastore and hiveserver2 services." >> $LOG_FILE
}

# 主监控循环
while true; do
    # 检查hiveserver2服务状态
    if ! is_hiveserver2_running; then
        echo "$(date '+%Y-%m-%d %H:%M:%S') - hiveserver2 is not running, attempting to restart..." >> $LOG_FILE
        stop_hiveserver2_services
        start_hiveserver2_services
    else
        echo "$(date '+%Y-%m-%d %H:%M:%S') - hiveserver2 is running." >> $LOG_FILE
    fi
    sleep 300  # 每5分钟检查一次
done

```

2.运行脚本

```bash
# 添加执行权限
chmod -R 777 monitor_hiveserver2.sh

# 启动脚本，后台执行
nohup ./monitor_hiveserver2.sh &
```

3.查看任务运行情况

```bash
ps -ef | grep monitor_hiveserver2.sh

tail -f nohup.out

# 关闭任务
kill -9 pid
```

# Hadoop

检查当前HDFS的全局副本设置

```bash
hdfs getconf -confKey dfs.replication
```

递归调整目录下文件的副本数

```bash
hdfs dfs -setrep -R 2 /path/to/your/directory

hdfs dfs -setrep -R 2 /user/hive/warehouse/test.db
```

# YARN

## yarn集群界面

![](D:\Github\MyKnowledgeRepository\img\bigdata\yarn\yarn cluster.png)

从截图中可以看出，这是YARN ResourceManager的Web界面，展示了集群的资源状态和节点信息。以下是界面中各个部分的详细解释：

### 1. Cluster Metrics（集群指标）
- **Apps Submitted**：提交到YARN的应用总数。这里显示为`12`，表示共提交了12个应用。
- **Apps Pending**：等待运行的应用数量。显示为`0`，表示没有应用在等待资源。
- **Apps Running**：当前正在运行的应用数量。显示为`0`，表示当前没有应用在运行。
- **Apps Completed**：已完成的应用数量。显示为`12`，表示所有提交的应用已经完成。
- **Containers Running**：当前正在运行的容器数量。显示为`0`，表示没有容器在运行。
- **Used Resources**：当前使用的资源数量。显示为`<memory:0 B, vCores:0>`，表示没有使用内存或vCore资源。
- **Total Resources**：集群中所有节点的总资源。这里显示为`<memory:72 GB, vCores:48>`，表示整个集群有72 GB的内存和48个vCores。
- **Reserved Resources**：为尚未运行的应用预留的资源。显示为`<memory:0 B, vCores:0>`，表示没有预留的资源。
- **Physical Mem Used %**：显示内存的物理使用率，这里为`52%`，说明集群中物理内存的使用量达到了52%。
- **Physical vCores Used %**：显示vCores的物理使用率，这里为`0%`，表示没有使用任何vCores。

### 2. Cluster Nodes Metrics（集群节点指标）
- **Active Nodes**：集群中处于活动状态的节点数量。显示为`3`，表示有3个活跃节点。
- **Decommissioning Nodes**：正在退出集群的节点数量。显示为`0`，表示没有节点在退出。
- **Lost Nodes**：丢失的节点数量。显示为`0`，表示没有丢失的节点。
- **Unhealthy Nodes**：不健康的节点数量。显示为`0`，表示所有节点状态正常。
- **Rebooted Nodes**：重新启动的节点数量。显示为`0`，表示没有节点被重新启动。
- **Shutdown Nodes**：已关闭的节点数量。显示为`0`，表示没有节点被关闭。

### 3. Scheduler Metrics（调度器指标）
- **Scheduler Type**：调度器的类型。这里显示为`Capacity Scheduler`，表示使用的是容量调度器。
- **Scheduling Resource Type**：调度资源的类型。显示为`[memory-mb (unit=Mi), vcores]`，说明调度器按照内存（以MiB为单位）和vCores来分配资源。
- **Minimum Allocation**：调度器允许的最小资源分配。显示为`<memory:1024, vCores:2>`，表示每个容器分配的最小内存为1024MB，最小vCores为2。
- **Maximum Allocation**：调度器允许的最大资源分配。显示为`<memory:8192, vCores:4>`，表示每个容器分配的最大内存为8192MB，最大vCores为4。
- **Maximum Cluster Application Priority**：显示为`0`，通常在不设置任务优先级时为0。
- **Scheduler Busy %**：调度器的繁忙程度。显示为`0%`，表示调度器当前没有繁忙状态，系统空闲。

### 4. Nodes of the Cluster（集群节点信息）
此部分详细列出了集群中的每个节点的资源使用情况。

#### 表头信息
- **Node Labels**：节点的标签，通常用于分类节点资源分配。这里显示为`/default-rack`，表示所有节点在默认机架中。
- **Rack**：节点所在的机架。所有节点都在`/default-rack`机架下。
- **Node State**：节点状态。所有节点显示为`RUNNING`，表示节点正在运行，状态正常。
- **Node Address**：节点的网络地址和端口号，用于内部通信。三个节点的地址分别为`cesdb1:38680`、`cesdb2:44380`和`cesdb:32361`。
- **Node HTTP Address**：节点的HTTP地址，用于通过Web界面访问节点的状态信息。各节点的HTTP地址分别为`cesdb1:8042`、`cesdb2:8042`和`cesdb:8042`。
- **Last Health-update**：节点的最近健康状态更新的时间。
- **Health-report**：节点的健康状态报告。这里未显示具体信息，表示节点正常。
- **Containers**：节点上当前正在运行的容器数量。所有节点显示为`0`，表示没有容器在运行。
- **Allocation Tags**：分配标签，用于对资源进行分类和分配。这里为空。
- **Mem Used**：节点当前使用的内存量。所有节点显示为`0 B`，表示没有内存使用。
- **Mem Avail**：节点剩余的内存量。所有节点都显示为`24 GB`，即每个节点的可用内存为24 GB。
- **Phys Mem Used %**：节点物理内存的使用百分比。分别为`49%`、`39%`和`68%`，表示每个节点当前物理内存的使用情况。
- **VCores Used**：节点当前使用的vCores数量。所有节点显示为`0`，表示没有使用vCores。
- **VCores Avail**：节点的可用vCores数量。每个节点的可用vCores均为`16`。
- **Phys VCores Used %**：节点物理vCores的使用百分比。分别为`1%`、`1%`和`2%`，表示当前物理CPU的使用情况。
- **Version**：YARN版本。这里为`3.3.6`。

### 5. 总结
- **集群资源状态**：集群配置有72GB内存和48个vCores，但当前没有运行的任务或容器。
- **节点资源状态**：每个节点的内存和vCores配置为24GB和16个vCores。虽然节点显示有一些物理内存和vCores的使用，但并没有分配到任何容器。
- **调度器配置**：容量调度器的最小和最大分配设置生效，确保容器资源在`1024 MB`到`8192 MB`之间，vCores在`2`到`4`之间。

这表明当前YARN集群已正常启动，资源配置正确，但当前没有运行任务，因此没有资源占用。

# 报错

## 1.java.lang.OutOfMemoryError: GC overhead limit exceeded

调整集群资源

### 并发

是的，这样配置后，你的集群可以更高效地利用资源，支持同时运行多个任务。通过合理设置每个容器的CPU核心和内存分配，YARN会自动调度资源，使得集群能够并发执行多个任务。

**并发任务的优势**

- 每台机器分配的16个CPU核心和足够的内存允许YARN在单台机器上运行多个Mapper、Reducer或其他容器任务。
- 多个任务可以同时运行，不会因为资源不足而互相阻塞，从而提升集群的吞吐量。

**如何控制并发任务数量**

配置完成后，集群中可以同时运行的任务数量受以下因素影响：
1. **NodeManager的资源总量**：`yarn.nodemanager.resource.memory-mb`和`yarn.nodemanager.resource.cpu-vcores`定义了每台机器的资源上限。
2. **单个容器的资源分配**：`mapreduce.map.cpu.vcores`、`mapreduce.reduce.cpu.vcores`、`mapreduce.map.memory.mb`和`mapreduce.reduce.memory.mb`决定了每个MapReduce任务所需的资源量。
3. **YARN调度器**：YARN会依据配置的资源限制，在任务之间动态分配资源，确保所有节点上的资源得到最优利用。

**举例计算**

假设你设置了以下参数：
- 每台机器分配了 **16个CPU核心** 和 **24GB内存**。
- 每个Mapper或Reducer任务分配 **2个核心** 和 **4GB内存**。

在这种配置下，每台机器最多可以并发执行大约 **8个任务**（24GB ÷ 4GB = 6，16个核心 ÷ 2个核心 = 8，以资源较少的计算），最终并发数量还会依据任务的资源需求在上下浮动。

