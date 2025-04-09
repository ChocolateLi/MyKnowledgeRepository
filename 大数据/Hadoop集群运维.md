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

优化后代码

```bash
#!/bin/bash

# 设置Hive的安装路径
HIVE_HOME=/data/u01/app/hive/apache-hive-3.1.3

# 日志目录设置
LOG_DIR="/data/u01/app/hive/apache-hive-3.1.3/logs/hiveserver2_monitor"
mkdir -p $LOG_DIR  # 创建日志目录（如果不存在）
DATE=$(date +%Y-%m-%d)
LOG_FILE="$LOG_DIR/hiveserver2_monitor_$DATE.log"

# 将所有输出重定向到日志文件，以减少文件描述符使用
exec >> $LOG_FILE 2>&1

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
    echo "$(date '+%Y-%m-%d %H:%M:%S') - Stopped all RunJar services."
}

# 启动metastore和hiveserver2服务
function start_hiveserver2_services() {
    $HIVE_HOME/bin/hive --service metastore &
    $HIVE_HOME/bin/hive --service hiveserver2 &
    echo "$(date '+%Y-%m-%d %H:%M:%S') - Started metastore and hiveserver2 services."
}

# 主监控循环
while true; do
    # 检查hiveserver2服务状态
    if ! is_hiveserver2_running; then
        echo "$(date '+%Y-%m-%d %H:%M:%S') - hiveserver2 is not running, attempting to restart..."
        stop_hiveserver2_services
        start_hiveserver2_services
    else
        echo "$(date '+%Y-%m-%d %H:%M:%S') - hiveserver2 is running."
    fi
    sleep 300  # 每5分钟检查一次
done

```

再优化代码

```bash
#!/bin/bash

# 设置Hive的安装路径
HIVE_HOME=/data/u01/app/hive/apache-hive-3.1.3

# 日志目录设置
LOG_DIR="/data/u01/app/hive/apache-hive-3.1.3/logs/hiveserver2_monitor"
mkdir -p $LOG_DIR  # 创建日志目录（如果不存在）
DATE=$(date +%Y-%m-%d)
LOG_FILE="$LOG_DIR/hiveserver2_monitor_$DATE.log"

# 将所有输出重定向到日志文件，以减少文件描述符使用
exec >> $LOG_FILE 2>&1

# 删除两天前的日志文件
find $LOG_DIR -type f -name "hiveserver2_monitor_*.log" -mtime +2 -exec rm {} \;

# HiveServer2 端口
HIVESERVER2_PORT=10000
HIVESERVER2_STATUS_PORT=10002

# 检查hiveserver2服务是否运行
function is_hiveserver2_running() {
    # 使用netstat检查端口是否被占用
    netstat -an | grep -q ":$HIVESERVER2_PORT .*LISTEN"
    return $?
}

# 发送简单查询以检测服务是否阻塞
function is_hiveserver2_responsive() {
    $HIVE_HOME/bin/beeline -u "jdbc:hive2://localhost:$HIVESERVER2_PORT" -e "show databases;" > /dev/null 2>&1
    return $?
}

# 关闭所有HiveServer2和MetaStore服务
function stop_hiveserver2_services() {
    jps | grep -E "HiveServer2|RunJar" | awk '{print $1}' | xargs kill -9
    echo "$(date '+%Y-%m-%d %H:%M:%S') - Stopped HiveServer2 and MetaStore services."
}

# 启动metastore和hiveserver2服务
function start_hiveserver2_services() {
    $HIVE_HOME/bin/hive --service metastore &
    $HIVE_HOME/bin/hive --service hiveserver2 &
    echo "$(date '+%Y-%m-%d %H:%M:%S') - Started metastore and hiveserver2 services."
}

# 主监控循环
while true; do
    # 检查hiveserver2服务状态
    if ! is_hiveserver2_running || ! is_hiveserver2_responsive; then
        echo "$(date '+%Y-%m-%d %H:%M:%S') - hiveserver2 is not running or not responsive, attempting to restart..."
        stop_hiveserver2_services
        start_hiveserver2_services
    else
        echo "$(date '+%Y-%m-%d %H:%M:%S') - hiveserver2 is running and responsive."
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

## 每天备份hive元数据

脚本

```bash
#!/bin/bash

# 配置环境变量
export ORACLE_HOME=/data/u01/app/oracle/product/19/db_1
export PATH=$ORACLE_HOME/bin:$PATH
export ORACLE_SID=cesdb  # 本地 Oracle 数据库 SID

# 配置变量
BACKUP_DIR="/data/u01/backup/hive_backup/hive_metadata"  # 本地备份目录
LOG_DIR="/data/u01/backup/hive_backup/logs"  # 日志目录
REMOTE_HOSTS=("cesdb1" "cesdb2")  # 远程主机列表
REMOTE_BACKUP_DIR="/data/u01/backup/hive_backup/hive_metadata"  # 远程备份目录
DB_USER="hive_user"  # 数据库用户名
DB_PASSWORD="hive_0753"  # 数据库密码
# DB_CONNECTION="10.201.100.75:1521:cesdb"  # 数据库连接字符串
DATE=$(date +"%Y%m%d")
LOG_FILE="${LOG_DIR}/backup_log_${DATE}.log"
BACKUP_FILE="hive_metadata_${DATE}.dmp"  # 备份文件名

# 确保备份和日志目录存在
mkdir -p "$BACKUP_DIR"
mkdir -p "$LOG_DIR"

# 备份Hive元数据
echo "[$(date +"%Y-%m-%d %H:%M:%S")] 开始备份Hive元数据..." | tee -a "$LOG_FILE"
expdp $DB_USER/$DB_PASSWORD schemas=HIVE_USER directory=DATA_PUMP_DIR dumpfile=$BACKUP_FILE logfile=expdp_hive_metadata_${DATE}.log

# 检查备份文件是否生成成功
if [ -f "/data/u01/app/oracle/admin/cesdb/dpdump/$BACKUP_FILE" ]; then
    # 移动备份文件到备份目录
    mv /data/u01/app/oracle/admin/cesdb/dpdump/$BACKUP_FILE "$BACKUP_DIR/"
    mv /data/u01/app/oracle/admin/cesdb/dpdump/expdp_hive_metadata_${DATE}.log "$LOG_DIR/"
    echo "[$(date +"%Y-%m-%d %H:%M:%S")] 备份完成：$BACKUP_DIR/$BACKUP_FILE" | tee -a "$LOG_FILE"
else
    echo "[$(date +"%Y-%m-%d %H:%M:%S")] 备份失败，未生成备份文件" | tee -a "$LOG_FILE"
    exit 1  # 退出脚本，避免传输不存在的文件
fi

# 将备份文件传输到远程服务器并覆盖
for host in "${REMOTE_HOSTS[@]}"; do
    echo "[$(date +"%Y-%m-%d %H:%M:%S")] 正在将备份文件传输到 $host 并覆盖..." | tee -a "$LOG_FILE"
    scp "$BACKUP_DIR/$BACKUP_FILE" "$host:$REMOTE_BACKUP_DIR/$BACKUP_FILE" && \
    echo "[$(date +"%Y-%m-%d %H:%M:%S")] 传输到 $host 成功" | tee -a "$LOG_FILE" || \
    echo "[$(date +"%Y-%m-%d %H:%M:%S")] 传输到 $host 失败" | tee -a "$LOG_FILE"
done

# 删除三天前的备份和日志
find "$BACKUP_DIR" -type f -name "hive_metadata_*.dmp" -mtime +3 -exec rm {} \;
find "$LOG_DIR" -type f -name "backup_log_*.log" -mtime +3 -exec rm {} \;
echo "[$(date +"%Y-%m-%d %H:%M:%S")] 已删除三天前的旧备份和日志" | tee -a "$LOG_FILE"

echo "[$(date +"%Y-%m-%d %H:%M:%S")] 全部任务完成" | tee -a "$LOG_FILE"


```

添加权限

```bash
# 添加执行权限
chmod -R 777 backup_hive_metadata.sh
```

配置定时任务

```bash
crontab -e
0 0 * * * /path/to/backup_hive_metadata.sh >> /data/u01/backup/hive_backup/logs/cron_backup_log.log 2>&1
```

## shell +  cron方式监控Metastore和HiveServer2端口

1.查看端口是否监听

netstat可以检查端口是否被占用。

lsof是一个查看打开文件和端口使用情况的工具。

```bash
[hadoop@cesdb bin]$ netstat -tuln | grep 10000
tcp6       0      0 :::10000                :::*                    LISTEN     
[hadoop@cesdb bin]$ netstat -tuln | grep 9083
tcp6       0      0 :::9083                 :::*                    LISTEN
[hadoop@cesdb conf]$ lsof -i:10000
COMMAND    PID   USER   FD   TYPE    DEVICE SIZE/OFF NODE NAME
java    110500 hadoop  649u  IPv6 119579645      0t0  TCP cesdb:ndmp->199.168.66.197:54920 (ESTABLISHED)
java    110500 hadoop  678u  IPv6 119528279      0t0  TCP *:ndmp (LISTEN)
java    110500 hadoop  693u  IPv6 119579472      0t0  TCP cesdb:ndmp->199.168.66.201:65189 (ESTABLISHED)
java    110500 hadoop  696u  IPv6 119579488      0t0  TCP cesdb:ndmp->199.168.66.201:65196 (ESTABLISHED)
java    110500 hadoop  703u  IPv6 119579496      0t0  TCP cesdb:ndmp->199.168.66.201:65197 (ESTABLISHED)
java    110500 hadoop  704u  IPv6 119579648      0t0  TCP cesdb:ndmp->199.168.66.197:54921 (ESTABLISHED)
java    110500 hadoop  706u  IPv6 119579498      0t0  TCP cesdb:ndmp->199.168.66.201:65198 (ESTABLISHED)
java    110500 hadoop  732u  IPv6 119610371      0t0  TCP cesdb:ndmp->199.168.66.197:54922 (ESTABLISHED)
java    110500 hadoop  735u  IPv6 119610374      0t0  TCP cesdb:ndmp->199.168.66.197:54923 (ESTABLISHED)
java    110500 hadoop  740u  IPv6 119610377      0t0  TCP cesdb:ndmp->199.168.66.197:54924 (ESTABLISHED)
java    110500 hadoop  743u  IPv6 119610380      0t0  TCP cesdb:ndmp->199.168.66.197:54925 (ESTABLISHED)
java    110500 hadoop  746u  IPv6 119610383      0t0  TCP cesdb:ndmp->199.168.66.197:54926 (ESTABLISHED)
java    110500 hadoop  751u  IPv6 119610386      0t0  TCP cesdb:ndmp->199.168.66.197:54927 (ESTABLISHED)
java    110500 hadoop  755u  IPv6 119610389      0t0  TCP cesdb:ndmp->199.168.66.197:54928 (ESTABLISHED)
java    110500 hadoop  820u  IPv6 119858124      0t0  TCP cesdb:ndmp->199.168.66.168:65484 (ESTABLISHED)
java    110500 hadoop  821u  IPv6 119858172      0t0  TCP cesdb:ndmp->199.168.66.168:65487 (ESTABLISHED)
java    110500 hadoop  837u  IPv6 119875979      0t0  TCP cesdb:ndmp->199.168.66.197:55010 (ESTABLISHED)
java    110500 hadoop  900u  IPv6 120201026      0t0  TCP cesdb:ndmp->199.168.66.168:49346 (ESTABLISHED)
[hadoop@cesdb conf]$ lsof -i:9083
COMMAND    PID   USER   FD   TYPE    DEVICE SIZE/OFF NODE NAME
java    110499 hadoop  670u  IPv6 119590967      0t0  TCP *:emc-pp-mgmtsvc (LISTEN)
[hadoop@cesdb conf]$ 

```

2.监控shell脚本

```bash
#!/bin/bash

# 配置
LOG_DIR="/data/u01/app/hive/apache-hive-3.1.3/logs/hiveserver2_monitor"  # 日志目录
TODAY=$(date '+%Y-%m-%d')
YESTERDAY=$(date -d "yesterday" '+%Y-%m-%d')

METASTORE_PORT=9083
METASTORE_HOST=127.0.0.1
HIVESERVER2_PORT=10000
HIVESERVER2_HOST=127.0.0.1

# 创建日志目录（如果不存在）
mkdir -p "$LOG_DIR"

# 日志文件
LOG_FILE="$LOG_DIR/$TODAY.log"

# 检查服务函数
check_service() {
    local HOST=$1
    local PORT=$2
    local SERVICE_NAME=$3
    if nc -z "$HOST" "$PORT"; then
        echo "$(date '+%Y-%m-%d %H:%M:%S') - $SERVICE_NAME is running." >> "$LOG_FILE"
    else
        echo "$(date '+%Y-%m-%d %H:%M:%S') - $SERVICE_NAME is down. Restarting..." >> "$LOG_FILE"
        # 重启服务命令（根据实际环境修改）
        if [ "$SERVICE_NAME" == "Metastore" ]; then
            nohup $HIVE_HOME/bin/hive --service metastore >> "$LOG_FILE" 2>&1 &
        elif [ "$SERVICE_NAME" == "HiveServer2" ]; then
            nohup $HIVE_HOME/bin/hive --service hiveserver2 >> "$LOG_FILE" 2>&1 &
        fi
        echo "$(date '+%Y-%m-%d %H:%M:%S') - $SERVICE_NAME restart command executed." >> "$LOG_FILE"
    fi
}

# 检查 Metastore 和 HiveServer2
check_service "$METASTORE_HOST" "$METASTORE_PORT" "Metastore"
check_service "$HIVESERVER2_HOST" "$HIVESERVER2_PORT" "HiveServer2"

# 删除昨天的日志文件
if [ -f "$LOG_DIR/$YESTERDAY.log" ]; then
    rm -f "$LOG_DIR/$YESTERDAY.log"
    echo "$(date '+%Y-%m-%d %H:%M:%S') - Deleted $LOG_DIR/$YESTERDAY.log" >> "$LOG_FILE"
fi
```

3.配置cron定时任务

```bash
# 每15分钟检查一次
*/15 * * * * /data/u01/app/hive/apache-hive-3.1.3/bin/check_hive_services.sh >> /data/u01/app/hive/apache-hive-3.1.3/logs/hiveserver2_monitor 2>&1

```

## 集群文件分发脚本

```bash
vim sync_script.sh

#!/bin/bash

# 获取输出参数，如果没有参数则直接返回
pcount=$#
if [ $pcount -eq 0 ]; then
    echo "No parameter provided!"
    exit 1
fi

# 获取传输文件名
p1=$1
filename=$(basename "$p1")
echo "Load file $p1 success!"

# 获取文件的绝对路径
pdir=$(cd -P "$(dirname "$p1")" && pwd)
echo "File path is $pdir"

# 获取当前用户（如果想使用 root 用户权限拷贝文件，在命令后加入 -root 参数即可）
user=$2
case "$user" in
"-root")
    user="root"
    ;;
"")
    user=$(whoami)
    ;;
*)
    echo "Illegal parameter $user"
    exit 1
    ;;
esac

echo "Using user: $user"

# 拷贝文件到从机（这里的目标机器是 cesdb1 到 cesdb4）
for host in cesdb1 cesdb2 cesdb3 cesdb4; do
    echo "================ Current host is $host ================="
    scp "$pdir/$filename" "$user@$host:$pdir"
    if [ $? -eq 0 ]; then
        echo "File successfully copied to $host:$pdir"
    else
        echo "Failed to copy file to $host"
    fi
done

echo "Complete!"

```

添加权限

```bash
chmod -R 777 sync_script.sh
```

使用，执行脚本

```bash
./sync_script.sh /data/u01/app/hadoop_config.xml

# 如果用root用户执行
./sync_script.sh /data/u01/app/hadoop_config.xml -root
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

## Yarn队列

![](D:\Github\MyKnowledgeRepository\img\bigdata\yarn\yarn队列.png)

该界面是 YARN (Yet Another Resource Negotiator) 中资源管理器的应用程序队列页面，显示了 `root.default` 队列的配置信息和资源使用情况。以下是对主要部分的解释：

1. **Queue State (队列状态)**:  
   队列的当前状态是 `RUNNING`，表示该队列处于活动状态，可以接受新任务。

2. **Used Capacity (已使用容量)**:  
   `Used Capacity: <memory:0, vCores:0> (0.0%)` 表示当前没有使用内存或虚拟 CPU 核心，使用率为 0%。这意味着该队列没有正在运行的任务。

3. **Configured Capacity (配置容量)** 和 **Configured Max Capacity (配置最大容量)**:  
   配置的容量为 `<memory:0, vCores:0>`，并且最大容量为 `unlimited`，这表明该队列没有被设置固定容量限制。

4. **Effective Capacity (有效容量)** 和 **Effective Max Capacity (有效最大容量)**:  
   显示该队列的有效容量，即该队列可以使用的最大资源量。在此例中为 `<memory:73728, vCores:48>`，即 73728 MB 内存和 48 个 vCores。

5. **Absolute Used Capacity (绝对已使用容量)**:  
   为 `0.0%`，表示绝对资源使用率为 0%。

6. **Configured Max Application Master Limit (配置的最大应用程序主限制)**:  
   为 `1.0`，意味着每个用户在该队列中只能有一个应用程序主进程。

7. **Max Application Master Resources (最大应用程序主资源)**:  
   每个应用程序主进程最多可以使用 `memory:8192, vCores:1`，即 8192 MB 内存和 1 个 vCore。

8. **Num Schedulable Applications (可调度应用程序数)** 和 **Num Non-Schedulable Applications (不可调度应用程序数)**:  
   当前该队列中没有可以调度的应用程序或不可调度的应用程序。

9. **Max Applications (最大应用程序数)** 和 **Max Applications Per User (每用户最大应用程序数)**:  
   队列的最大应用程序数为 10000，每个用户的最大应用程序数也为 10000。

10. **Ordering Policy (排序策略)**:  
    使用 `FifoOrderingPolicy`，即先进先出排序。

11. **Preemption (抢占)** 和 **Intra-queue Preemption (队列内抢占)**:  
    抢占功能被禁用（disabled），表示不会通过抢占资源来调度其他任务。

12. **Default Node Label Expression (默认节点标签表达式)**:  
    为 `<DEFAULT_PARTITION>`，表明该队列的默认标签，用于控制任务在哪些节点上运行。

13. **Default Application Priority (默认应用程序优先级)**:  
    默认优先级为 0，表示任务在调度时没有特殊优先级。

总体来说，该界面展示了 `root.default` 队列的配置信息和资源使用情况，目前没有任务正在运行，队列处于空闲状态，未占用任何资源。



## Yarn Running任务

![](D:\Github\MyKnowledgeRepository\img\bigdata\yarn\yarn running任务.png)

在YARN（Yet Another Resource Negotiator）的界面上，您看到的是某个应用程序（Application Attempt）正在运行的详细信息。这是一个典型的YARN应用程序监控界面，显示了资源分配和执行情况。

从界面上可以观察到以下几点：

1. **Application Attempt State**: 应用程序的当前状态为“RUNNING”，表示这个任务正在执行中。

2. **Total Allocated Containers**: 系统为这个应用程序分配了6个容器（Containers）。这些容器是YARN用于并行处理的基本执行单元。每个容器代表一个独立的进程或线程，可以在集群中的不同节点上执行。

3. **Num Node Local Containers / Rack Local Containers / Off Switch Containers**:
   - **Node Local Containers**表示容器位于相同节点上，减少数据传输的开销（此处显示为1个）。
   - **Rack Local Containers**表示容器位于相同机架上，但不同节点上（此处显示为0）。
   - **Off Switch Containers**表示容器位于不同机架上，可能需要跨机架传输数据（此处显示为2个）。

4. **Container ID 和 Node**: 在下方的表格中列出了每个容器的ID和对应的节点地址，说明这些容器被分配到不同节点上执行。YARN通过分配多个容器来支持任务的并行处理，因此确实是多个容器并行执行。

**总结**

是YARN的设计目标之一就是支持并行计算。每个容器可以看作是一个独立的计算单元，多个容器可以并行执行不同的任务，从而加快整个应用的执行时间。在这个界面中，您可以看到6个容器分布在不同节点上，支持应用程序的并行执行。

![](D:\Github\MyKnowledgeRepository\img\bigdata\yarn\yarn running任务1.png)

该界面是YARN的应用程序资源请求和分配情况的详细界面，具体解释如下：

### Application Attempt Headroom
- **Application Attempt Headroom** 表示此应用程序在资源上限内还能请求的剩余资源量。
- 本次尝试的剩余资源上限为 **2048 MB 内存** 和 **30 个 vCores**。

### Total Allocated Containers
- **Total Allocated Containers: 74** 表示当前已经为该应用程序分配了 **74 个容器**。
- 这些容器被分配在不同的节点上，用于并行执行任务。

### Container Locality 分配情况
界面下方的表格展示了容器的分配情况，分为 **Node Local Request** 和 **Off Switch Request**，没有 **Rack Local Request**，说明该应用程序优先在本地节点和机架之外的节点上执行。
- **Num Node Local Containers (satisfied by)**: 表示满足 **Node Local 请求**的容器数量，目前分配了 **67 个 Node Local 请求**。
- **Num Rack Local Containers (satisfied by)**: 没有分配到任何 **Rack Local** 请求。
- **Num Off Switch Containers (satisfied by)**: 通过 **Off Switch** 方式分配了 **7 个容器**，这通常指容器被分配到与数据所在机架不同的地方，增加了跨机架传输的开销。

### Total Outstanding Resource Requests
此部分表格详细展示了当前应用的 **资源请求** 状况：
- **Priority**: 请求的优先级，这里显示所有请求的优先级为20。
- **AllocationRequestId**: 资源分配请求ID，此处统一显示为 **-1**，表示没有特定ID。
- **ResourceName**: 显示资源位置的名称：
  - **cesdb1、cesdb2、/default-rack、cesdb** 等为集群中的节点或机架。
  - **"*”** 表示可以分配到任意节点。
- **Capability**: 每个容器请求的资源规格，这里每个容器请求 **4096 MB 内存和1个vCore**。
- **NumContainers**: 当前请求的容器数量，每行表示不同资源位置的请求数量。例如，`cesdb1`请求了**30个容器**，`cesdb2`请求了**24个容器**，`default-rack`请求了**87个容器**等。
- **RelaxLocality**: 允许资源调度松弛本地性。**true** 表示请求可以放宽位置要求，可以分配到非本地节点。
- **NodeLabelExpression**: 目前显示为空（N/A），表示没有特定的节点标签需求。
- **ExecutionType**: 执行类型，显示为 **GUARANTEED**，表示这些请求的资源分配是保证执行的，YARN会优先满足这类请求。
- **AllocationTags / PlacementConstraint**: 分配标签和位置约束，当前显示为空（N/A），表示没有特殊约束。

### 总结
- 该界面显示了一个运行中的应用程序资源使用情况。应用当前已分配了74个容器，其中67个容器是Node Local本地分配，7个为Off Switch跨机架分配。
- 资源请求表格显示应用正在持续请求多个容器来处理任务，并且请求的容器配置均为4096 MB内存和1个vCore。
- RelaxLocality为true表示应用可以接受非本地容器分配，这样有助于在资源紧张时加快容器分配，提高资源利用率。

# Hive

## 1.设置会话时间

**hive-site.xml**

```bash
<!-- 每隔30分钟检查不活跃的会话 -->
<property>
    <name>hive.server2.session.check.interval</name>
    <value>1800000</value> <!-- 以毫秒为单位，1800000毫秒即30分钟 -->
    <description>检查不活跃会话的时间间隔（毫秒）。设置为1800000表示每30分钟检查一次。</description>
</property>

<!-- 会话的空闲超时时间为1小时 -->
<property>
    <name>hive.server2.idle.session.timeout</name>
    <value>3600000</value> <!-- 以毫秒为单位，3600000毫秒即1小时 -->
    <description>不活跃的会话超过该时间将被关闭（毫秒）。设置为3600000表示会话空闲超过1小时自动关闭。</description>
</property>

```

## 2.查看连接等

```bash
# 查看哪些主机连接了10000端口
netstat -anp | grep 10000

# 查看这台主机的连接数pid等
ss -tnp | grep '199.168.66.201'
ss -tnp | grep ':10000' | wc -l

# 查看日志
less hive.log.2025-03-19

# 查看日志 error  
grep "error" hive.log.2025-03-19
grep "SocketException" hive.log.2025-03-19

grep -i "error" /var/log/hive/hiveserver2.log
grep -i "timeout" /var/log/hive/hiveserver2.log
grep -i "disconnect" /var/log/hive/hiveserver2.log
```

## 3.设置hive的JVM参数

```bash
cd /data/u01/app/hive/apache-hive-3.1.3/conf

vim hive-env.sh
# 设置Hive服务的内存配置
export HIVE_HEAPSIZE=8192  # HiveServer2的堆内存：4GB
export HADOOP_HEAPSIZE=8192  # Hive使用的Hadoop客户端内存：4GB
export HADOOP_OPTS="-XX:+UseG1GC -XX:InitiatingHeapOccupancyPercent=35"  # 启用G1GC

# 重启hive服务

# 验证服务是否生效
[hadoop@cesdb conf]$ ps -ef | grep -E 'HiveMetaStore|HiveServer2' | grep -v grep
hadoop   221218      1  1 15:39 pts/0    00:00:38 /usr/jdk/bin/java -Dproc_jar -XX:+UseG1GC -XX:InitiatingHeapOccupancyPercent=35 -Dproc_metastore -Dlog4j2.formatMsgNoLookups=true -Dlog4j.configurationFile=hive-log4j2.properties -Djava.util.logging.config.file=/data/u01/app/hive/apache-hive-3.1.3/conf/parquet-logging.properties -Dyarn.log.dir=/data/u01/app/hadoop/hadoop-3.3.6/logs -Dyarn.log.file=hadoop.log -Dyarn.home.dir=/data/u01/app/hadoop/hadoop-3.3.6 -Dyarn.root.logger=INFO,console -Djava.library.path=/data/u01/app/hadoop/hadoop-3.3.6/lib/native -Xmx8192m -Dhadoop.log.dir=/data/u01/app/hadoop/hadoop-3.3.6/logs -Dhadoop.log.file=hadoop.log -Dhadoop.home.dir=/data/u01/app/hadoop/hadoop-3.3.6 -Dhadoop.id.str=hadoop -Dhadoop.root.logger=INFO,console -Dhadoop.policy.file=hadoop-policy.xml -Dhadoop.security.logger=INFO,NullAppender org.apache.hadoop.util.RunJar /data/u01/app/hive/apache-hive-3.1.3/lib/hive-metastore-3.1.3.jar org.apache.hadoop.hive.metastore.HiveMetaStore
hadoop   221219      1  4 15:39 pts/0    00:01:20 /usr/jdk/bin/java -Dproc_jar -XX:+UseG1GC -XX:InitiatingHeapOccupancyPercent=35 -Dproc_hiveserver2 -Dlog4j2.formatMsgNoLookups=true -Dlog4j.configurationFile=hive-log4j2.properties -Djava.util.logging.config.file=/data/u01/app/hive/apache-hive-3.1.3/conf/parquet-logging.properties -Djline.terminal=jline.UnsupportedTerminal -Dyarn.log.dir=/data/u01/app/hadoop/hadoop-3.3.6/logs -Dyarn.log.file=hadoop.log -Dyarn.home.dir=/data/u01/app/hadoop/hadoop-3.3.6 -Dyarn.root.logger=INFO,console -Djava.library.path=/data/u01/app/hadoop/hadoop-3.3.6/lib/native -Xmx8192m -Dhadoop.log.dir=/data/u01/app/hadoop/hadoop-3.3.6/logs -Dhadoop.log.file=hadoop.log -Dhadoop.home.dir=/data/u01/app/hadoop/hadoop-3.3.6 -Dhadoop.id.str=hadoop -Dhadoop.root.logger=INFO,console -Dhadoop.policy.file=hadoop-policy.xml -Dhadoop.security.logger=INFO,NullAppender org.apache.hadoop.util.RunJar /data/u01/app/hive/apache-hive-3.1.3/lib/hive-service-3.1.3.jar org.apache.hive.service.server.HiveServer2
[hadoop@cesdb conf]$ 
```

## 4.Hive 的 JVM内存检查和优化

### 1.查看JVM进程，查找`-Xmx`参数(最大堆内存)和`-Xms`参数(初始堆内存)

```bash
[hadoop@cesdb conf]$ ps -ef | grep HiveServer2
hadoop    62338  83613  0 09:37 pts/0    00:00:00 grep --color=auto HiveServer2
hadoop   221219      1  0 4月07 pts/0   00:10:36 /usr/jdk/bin/java -Dproc_jar -XX:+UseG1GC -XX:InitiatingHeapOccupancyPercent=35 -Dproc_hiveserver2 -Dlog4j2.formatMsgNoLookups=true -Dlog4j.configurationFile=hive-log4j2.properties -Djava.util.logging.config.file=/data/u01/app/hive/apache-hive-3.1.3/conf/parquet-logging.properties -Djline.terminal=jline.UnsupportedTerminal -Dyarn.log.dir=/data/u01/app/hadoop/hadoop-3.3.6/logs -Dyarn.log.file=hadoop.log -Dyarn.home.dir=/data/u01/app/hadoop/hadoop-3.3.6 -Dyarn.root.logger=INFO,console -Djava.library.path=/data/u01/app/hadoop/hadoop-3.3.6/lib/native -Xmx8192m -Dhadoop.log.dir=/data/u01/app/hadoop/hadoop-3.3.6/logs -Dhadoop.log.file=hadoop.log -Dhadoop.home.dir=/data/u01/app/hadoop/hadoop-3.3.6 -Dhadoop.id.str=hadoop -Dhadoop.root.logger=INFO,console -Dhadoop.policy.file=hadoop-policy.xml -Dhadoop.security.logger=INFO,NullAppender org.apache.hadoop.util.RunJar /data/u01/app/hive/apache-hive-3.1.3/lib/hive-service-3.1.3.jar org.apache.hive.service.server.HiveServer2
```

### 2.监控内存使用情况

```bash
# 内存使用概况
jstat -gc <pid> 1000 5

jstat -gc <hiveserver2_pid> 1000 5

[hadoop@cesdb conf]$ jstat -gc 221219 1000 5
 S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT     GCT   
 0.0   114688.0  0.0   114688.0 911360.0 102400.0 1034240.0   757465.9  135932.0 127605.4 17204.0 13750.1     98    2.786   0      0.000    2.786
 0.0   114688.0  0.0   114688.0 911360.0 102400.0 1034240.0   757465.9  135932.0 127605.4 17204.0 13750.1     98    2.786   0      0.000    2.786
 0.0   114688.0  0.0   114688.0 911360.0 102400.0 1034240.0   757465.9  135932.0 127605.4 17204.0 13750.1     98    2.786   0      0.000    2.786
 0.0   114688.0  0.0   114688.0 911360.0 102400.0 1034240.0   757465.9  135932.0 127605.4 17204.0 13750.1     98    2.786   0      0.000    2.786
 0.0   114688.0  0.0   114688.0 911360.0 102400.0 1034240.0   757465.9  135932.0 127605.4 17204.0 13750.1     98    2.786   0      0.000    2.786
```

| S0C  | S1C    | S0U  | S1U    | EC     | EU     | OC      | OU       | MC     | MU       | CCSC  | CCSU    | YGC  | YGCT  | FGC  | FGCT | GCT   |
| ---- | ------ | ---- | ------ | ------ | ------ | ------- | -------- | ------ | -------- | ----- | ------- | ---- | ----- | ---- | ---- | ----- |
| 0    | 114688 | 0    | 114688 | 911360 | 102400 | 1034240 | 757465.9 | 135932 | 127605.4 | 17204 | 13750.1 | 98   | 2.786 | 0    | 0    | 2.786 |
| 0    | 114688 | 0    | 114688 | 911360 | 102400 | 1034240 | 757465.9 | 135932 | 127605.4 | 17204 | 13750.1 | 98   | 2.786 | 0    | 0    | 2.786 |
| 0    | 114688 | 0    | 114688 | 911360 | 102400 | 1034240 | 757465.9 | 135932 | 127605.4 | 17204 | 13750.1 | 98   | 2.786 | 0    | 0    | 2.786 |
| 0    | 114688 | 0    | 114688 | 911360 | 102400 | 1034240 | 757465.9 | 135932 | 127605.4 | 17204 | 13750.1 | 98   | 2.786 | 0    | 0    | 2.786 |
| 0    | 114688 | 0    | 114688 | 911360 | 102400 | 1034240 | 757465.9 | 135932 | 127605.4 | 17204 | 13750.1 | 98   | 2.786 | 0    | 0    | 2.786 |

**关键指标分析**

1. 各内存区域容量（单位KB）

   - `S0C/S1C`：Survivor 0/1区容量 (114,688KB ≈ 112MB)
   - `EC`：Eden区容量 (911,360KB ≈ 890MB)
   - `OC`：老年代容量 (1,034,240KB ≈ 1010MB)
   - `MC`：元空间容量 (135,932KB ≈ 133MB)

   从jstat数据计算实际堆内存：

   - S0C + S1C + EC + OC = 114688 + 114688 + 911360 + 1034240 = 2174976KB ≈ 2124MB
   - 这与预期的8GB(8192MB)相差甚远，说明可能有配置未生效或其他问题？
   - G1 GC的动态内存分配：G1垃圾回收器不像传统的分代GC（如Parallel或CMS）那样固定划分新生代（Young）和老年代（Old），而是将堆划分为多个 Region（默认2MB/个），并动态调整Eden、Survivor和Old区的大小。
   - 仅显示当前分配的区域，而非整个堆内存。剩余内存是未分配的free region，在jmap -heap里查看

2. 各内存使用量（单位KB）

   - `S0U/S1U`：Survivor区使用量 (0/114,688KB - 看起来所有对象都进入了S1)
   - `EU`：Eden区使用量 (102,400KB ≈ 100MB)
   - `OU`：老年代使用量 (757,465.9KB ≈ 740MB)
   - `MU`：元空间使用量 (127,605.4KB ≈ 125MB)

3. GC统计

   - `YGC`：Young GC次数 (98次)
   - `YGCT`：Young GC总时间 (2.786秒)
   - `FGC`：Full GC次数 (0次)
   - `FGCT`：Full GC总时间 (0秒)
   - `GCT`：GC总时间 (2.786秒)

**健康状况判断**

1. 内存使用情况：
   - 老年代使用率：OU/OC ≈ 740MB/1010MB ≈ 73%
   - Eden区使用率：EU/EC ≈ 100MB/890MB ≈ 11%
   - 元空间使用率：MU/MC ≈ 125MB/133MB ≈ 94% (这个略高)
2. GC情况：
   - 没有Full GC发生(FGC=0)，这是好的现象
   - Young GC平均耗时：2.786s/98 ≈ 28ms/次，属于正常范围
   - 对象晋升模式：所有对象都从Eden直接进入了S1(S0U=0, S1U=S1C)，说明可能配置了-XX:SurvivorRatio参数使得Survivor空间较大

**潜在问题点**

1. **元空间(Metaspace)使用接近上限** (94%)：
   - 可能导致频繁的元空间GC
   - 建议增加元空间大小：`-XX:MaxMetaspaceSize=256m`
2. **老年代使用率较高** (73%)：
   - 虽然目前安全，但如果有大查询可能导致OOM
   - 建议监控OU的增长趋势

### 3.查看更详细的内存分布

```bash
# 更详细的内存分布
jmap -heap <pid>

[hadoop@cesdb conf]$ jmap -heap 221219
Attaching to process ID 221219, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.333-b02

using thread-local object allocation.
Garbage-First (G1) GC with 43 thread(s)

Heap Configuration:
   MinHeapFreeRatio         = 40
   MaxHeapFreeRatio         = 70
   MaxHeapSize              = 8589934592 (8192.0MB)
   NewSize                  = 1363144 (1.2999954223632812MB)
   MaxNewSize               = 5152702464 (4914.0MB)
   OldSize                  = 5452592 (5.1999969482421875MB)
   NewRatio                 = 2
   SurvivorRatio            = 8
   MetaspaceSize            = 21807104 (20.796875MB)
   CompressedClassSpaceSize = 1073741824 (1024.0MB)
   MaxMetaspaceSize         = 17592186044415 MB
   G1HeapRegionSize         = 2097152 (2.0MB)

Heap Usage:
G1 Heap:
   regions  = 4096
   capacity = 8589934592 (8192.0MB)
   used     = 1406967936 (1341.7891845703125MB)
   free     = 7182966656 (6850.2108154296875MB)
   16.379262506961823% used
G1 Young Generation:
Eden Space:
   regions  = 242
   capacity = 981467136 (936.0MB)
   used     = 507510784 (484.0MB)
   free     = 473956352 (452.0MB)
   51.70940170940171% used
Survivor Space:
   regions  = 30
   capacity = 62914560 (60.0MB)
   used     = 62914560 (60.0MB)
   free     = 0 (0.0MB)
   100.0% used
G1 Old Generation:
   regions  = 401
   capacity = 1065353216 (1016.0MB)
   used     = 834445440 (795.7891845703125MB)
   free     = 230907776 (220.2108154296875MB)
   78.32570714274729% used

42746 interned Strings occupying 4469936 bytes.
```

**核心关注指标**

1. Heap Configuration
   - `MaxHeapSize`: 确认实际最大堆内存(这里是8GB，符合预期)
   - `G1HeapRegionSize`: G1 GC区域大小(2MB，标准配置)
   - `MaxMetaspaceSize`: 元空间上限(这里显示异常大，需要关注)
2. Heap Usage
   - `used/capacity`: 总堆内存使用率(16.38%，很低)
   - Old Gen使用率(78.33%，需警惕)
   - Survivor Space使用率(100%，正常现象)

**健康状况判断**

✅ **良好表现**：

- 总堆内存使用率仅16.38%，远低于警戒线
- 配置的8GB堆内存已正确生效
- 使用G1垃圾回收器(适合大内存场景)
- 没有内存泄漏迹象(used值稳定)

⚠️ **需关注点**：

1. 老年代使用率78.33%：
   - 接近G1的IHOP阈值(默认45%，您设置的是35%)
   - 可能触发Mixed GC，但您FGC=0说明控制良好
2. 元空间配置异常：
   - `MaxMetaspaceSize = 17592186044415 MB`(异常值)
   - 当前使用约20MB(健康)，但需要修复配置
3. Survivor区满负荷：
   - 这是正常现象(G1的特性)，但说明对象晋升频繁

### 4.HiveServer2内存优化

1.修改 hive-env.sh

```bash
# 设置堆内存（必须）
export HIVE_HEAPSIZE=8192  # HiveServer2的堆内存：8GB
export HADOOP_HEAPSIZE=8192  # Hive使用的Hadoop客户端内存：8GB
export HADOOP_OPTS="$HADOOP_OPTS -Xms8192m -Xmx8192m"  # 强制初始堆=最大堆，避免动态调整

# G1 GC优化参数
export HADOOP_OPTS="$HADOOP_OPTS -XX:+UseG1GC"
export HADOOP_OPTS="$HADOOP_OPTS -XX:InitiatingHeapOccupancyPercent=35"  # 老年代占用35%时启动Mixed GC
export HADOOP_OPTS="$HADOOP_OPTS -XX:G1ReservePercent=15"  # 保留15%内存防止OOM

# 元空间（Metaspace）配置（关键！）
export HADOOP_OPTS="$HADOOP_OPTS -XX:MetaspaceSize=256m -XX:MaxMetaspaceSize=512m"  # 默认256MB，上限512MB

# 其他优化
export HADOOP_OPTS="$HADOOP_OPTS -XX:+ParallelRefProcEnabled"  # 并行处理引用对象
export HADOOP_OPTS="$HADOOP_OPTS -XX:+HeapDumpOnOutOfMemoryError"  # OOM时生成Dump文件
export HADOOP_OPTS="$HADOOP_OPTS -XX:HeapDumpPath=/tmp/hive_heapdump.hprof"  # Dump文件路径
```

2.重启hive服务

```bash
# 2. 启动HiveServer2（后台运行）
nohup $HIVE_HOME/bin/hive --service metastore &> /data/u01/app/hive/apache-hive-3.1.3/logs/hive_metastore.log &
nohup $HIVE_HOME/bin/hive --service hiveserver2 &> /data/u01/app/hive/apache-hive-3.1.3/logs/hive_hiveserver2.log &
```

3.验证配置是否生效

```bash
ps -ef | grep HiveServer2

[hadoop@cesdb bin]$ ps -ef | grep HiveServer2
hadoop   130291      1 17 14:43 pts/0    00:00:39 /usr/jdk/bin/java -Dproc_jar -Xms8192m -Xmx8192m -XX:+UseG1GC -XX:InitiatingHeapOccupancyPercent=35 -XX:G1ReservePercent=15 -XX:MetaspaceSize=256m -XX:MaxMetaspaceSize=512m -XX:+ParallelRefProcEnabled -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/tmp/hive_heapdump.hprof -Dproc_hiveserver2 -Dlog4j2.formatMsgNoLookups=true -Dlog4j.configurationFile=hive-log4j2.properties -Djava.util.logging.config.file=/data/u01/app/hive/apache-hive-3.1.3/conf/parquet-logging.properties -Djline.terminal=jline.UnsupportedTerminal -Dyarn.log.dir=/data/u01/app/hadoop/hadoop-3.3.6/logs -Dyarn.log.file=hadoop.log -Dyarn.home.dir=/data/u01/app/hadoop/hadoop-3.3.6 -Dyarn.root.logger=INFO,console -Djava.library.path=/data/u01/app/hadoop/hadoop-3.3.6/lib/native -Dhadoop.log.dir=/data/u01/app/hadoop/hadoop-3.3.6/logs -Dhadoop.log.file=hadoop.log -Dhadoop.home.dir=/data/u01/app/hadoop/hadoop-3.3.6 -Dhadoop.id.str=hadoop -Dhadoop.root.logger=INFO,console -Dhadoop.policy.file=hadoop-policy.xml -Dhadoop.security.logger=INFO,NullAppender org.apache.hadoop.util.RunJar /data/u01/app/hive/apache-hive-3.1.3/lib/hive-service-3.1.3.jar org.apache.hive.service.server.HiveServer2
hadoop   143142  83613  0 14:47 pts/0    00:00:00 grep --color=auto HiveServer2

# 检查JVM参数
jcmd <PID> VM.flags | grep -E "MaxHeapSize|MetaspaceSize|UseG1GC"

jcmd 130291 VM.flags | grep -E "MaxHeapSize|MetaspaceSize|UseG1GC"
[hadoop@cesdb bin]$ jcmd 130291 VM.flags | grep -E "MaxHeapSize|MetaspaceSize|UseG1GC"
-XX:CICompilerCount=18 -XX:CompressedClassSpaceSize=528482304 -XX:ConcGCThreads=11 -XX:G1HeapRegionSize=4194304 -XX:G1ReservePercent=15 -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/tmp/hive_heapdump.hprof -XX:InitialHeapSize=8589934592 -XX:InitiatingHeapOccupancyPercent=35 -XX:MarkStackSize=4194304 -XX:MaxHeapSize=8589934592 -XX:MaxMetaspaceSize=536870912 -XX:MaxNewSize=5150605312 -XX:MetaspaceSize=268435456 -XX:MinHeapDeltaBytes=4194304 -XX:+ParallelRefProcEnabled -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:+UseG1GC 
[hadoop@cesdb bin]$ 

# 查看内存分布
jmap -heap <PID>
jmap -heap 130291

[hadoop@cesdb bin]$ jmap -heap 130291
Attaching to process ID 130291, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.333-b02

using thread-local object allocation.
Garbage-First (G1) GC with 43 thread(s)

Heap Configuration:
   MinHeapFreeRatio         = 40
   MaxHeapFreeRatio         = 70
   MaxHeapSize              = 8589934592 (8192.0MB)
   NewSize                  = 1363144 (1.2999954223632812MB)
   MaxNewSize               = 5150605312 (4912.0MB)
   OldSize                  = 5452592 (5.1999969482421875MB)
   NewRatio                 = 2
   SurvivorRatio            = 8
   MetaspaceSize            = 268435456 (256.0MB)
   CompressedClassSpaceSize = 528482304 (504.0MB)
   MaxMetaspaceSize         = 536870912 (512.0MB)
   G1HeapRegionSize         = 4194304 (4.0MB)

Heap Usage:
G1 Heap:
   regions  = 2048
   capacity = 8589934592 (8192.0MB)
   used     = 413087744 (393.951171875MB)
   free     = 8176846848 (7798.048828125MB)
   4.808974266052246% used
G1 Young Generation:
Eden Space:
   regions  = 55
   capacity = 398458880 (380.0MB)
   used     = 230686720 (220.0MB)
   free     = 167772160 (160.0MB)
   57.89473684210526% used
Survivor Space:
   regions  = 13
   capacity = 54525952 (52.0MB)
   used     = 54525952 (52.0MB)
   free     = 0 (0.0MB)
   100.0% used
G1 Old Generation:
   regions  = 31
   capacity = 8136949760 (7760.0MB)
   used     = 127875072 (121.951171875MB)
   free     = 8009074688 (7638.048828125MB)
   1.5715357200386597% used

38216 interned Strings occupying 3893992 bytes.

# 检查内存和gc
[hadoop@cesdb bin]$ jstat -gc 130291 1000 5
 S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT     GCT   
 0.0   53248.0  0.0   53248.0 458752.0 344064.0 7876608.0   139665.5  102784.0 101285.0 11904.0 11537.0      7    0.472   0      0.000    0.472
 0.0   53248.0  0.0   53248.0 458752.0 344064.0 7876608.0   139665.5  102784.0 101285.0 11904.0 11537.0      7    0.472   0      0.000    0.472
 0.0   53248.0  0.0   53248.0 458752.0 344064.0 7876608.0   139665.5  102784.0 101285.0 11904.0 11537.0      7    0.472   0      0.000    0.472
 0.0   53248.0  0.0   53248.0 458752.0 344064.0 7876608.0   139665.5  102784.0 101285.0 11904.0 11537.0      7    0.472   0      0.000    0.472
 0.0   53248.0  0.0   53248.0 458752.0 344064.0 7876608.0   139665.5  102784.0 101285.0 11904.0 11537.0      7    0.472   0      0.000    0.472
```

| S0C  | S1C   | S0U  | S1U   | EC     | EU     | OC      | OU       | MC     | MU     | CCSC  | CCSU  | YGC  | YGCT  | FGC  | FGCT | GCT   |
| ---- | ----- | ---- | ----- | ------ | ------ | ------- | -------- | ------ | ------ | ----- | ----- | ---- | ----- | ---- | ---- | ----- |
| 0    | 53248 | 0    | 53248 | 458752 | 344064 | 7876608 | 139665.5 | 102784 | 101285 | 11904 | 11537 | 7    | 0.472 | 0    | 0    | 0.472 |
| 0    | 53248 | 0    | 53248 | 458752 | 344064 | 7876608 | 139665.5 | 102784 | 101285 | 11904 | 11537 | 7    | 0.472 | 0    | 0    | 0.472 |
| 0    | 53248 | 0    | 53248 | 458752 | 344064 | 7876608 | 139665.5 | 102784 | 101285 | 11904 | 11537 | 7    | 0.472 | 0    | 0    | 0.472 |
| 0    | 53248 | 0    | 53248 | 458752 | 344064 | 7876608 | 139665.5 | 102784 | 101285 | 11904 | 11537 | 7    | 0.472 | 0    | 0    | 0.472 |
| 0    | 53248 | 0    | 53248 | 458752 | 344064 | 7876608 | 139665.5 | 102784 | 101285 | 11904 | 11537 | 7    | 0.472 | 0    | 0    | 0.472 |

关键指标：

| 指标                  | 健康范围 | 说明                    |
| --------------------- | -------- | ----------------------- |
| `OU` (Old区使用量)    | <80%     | 超过80%可能触发Full GC  |
| `FGC` (Full GC次数)   | 0或极少  | 频繁Full GC说明内存不足 |
| `YGCT` (Young GC时间) | <50ms/次 | 耗时过长需优化          |

4.实时监控

```bash
# 实时监控命令
watch -n 5 "jstat -gc <pid> 1000 3"

# 关键指标警报阈值：
# - Old Gen >80%
# - MetaSpace >90%
# - FGC >1次/小时
```

5.优化前后数据对比

![](D:\Github\MyKnowledgeRepository\img\bigdata\hive\JVM内存优化数据对比.png)

核心指标对比

| **指标**               | **优化前**         | **优化后**         | **改善效果**                                                 |
| ---------------------- | ------------------ | ------------------ | ------------------------------------------------------------ |
| **老年代使用(OU)**     | 757,465KB (≈740MB) | 139,665KB (≈136MB) | ✅ **降低81.5%**，大幅减少OOM风险                             |
| **Eden区使用(EU)**     | 102,400KB (≈100MB) | 344,064KB (≈336MB) | ➤ 使用量增加，但Eden区总容量从890MB→448MB，**利用率更合理** (75% vs 11%) |
| **Survivor区(S1U)**    | 114,688KB (满负荷) | 53,248KB (满负荷)  | ✅ **区域缩小但更高效**，说明对象晋升策略优化                 |
| **元空间(MU)**         | 127,605KB (≈125MB) | 101,285KB (≈99MB)  | ✅ **降低20%**，因`MaxMetaspaceSize`限制防止无限增长          |
| **Young GC次数(YGC)**  | 98次               | 7次                | ✅ **减少92.8%**，说明内存分配更合理，减少GC压力              |
| **Young GC耗时(YGCT)** | 2.786秒            | 0.472秒            | ✅ **降低83%**，平均每次GC耗时从28ms→67ms（因G1调整Region大小，单次GC处理更多对象）（YGCT/YGC） |
| **Full GC次数(FGC)**   | 0次                | 0次                | ✅ 保持无Full GC                                              |

优化**非常成功**，JVM内存管理从“勉强维持”变为“高效稳定”。关键改进：

1. **老年代使用量降低81.5%** → OOM风险解除
2. **GC次数减少92.8%** → 应用吞吐量提升
3. **元空间控制有效** → 避免内存泄漏

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



## 2.mr引擎丢数问题

同一sql，设置以下参数和不设置以下参数，跑出的数据不一致，丢失了很多数据。

```bash
set mapreduce.map.memory.mb=4096;
set mapreduce.map.java.opts=-Xmx3072m;
set hive.exec.reducers.bytes.per.reducer=512000000;-- 这个不一致，默认的是256000000
set hive.exec.reducers.max=100;-- 这个不一致
set mapreduce.reduce.memory.mb=8192;
set mapreduce.reduce.java.opts=-Xmx6144m;
set mapreduce.map.output.compress=true;-- 默认为false
set mapreduce.map.output.compress.codec=org.apache.hadoop.io.compress.SnappyCodec;-- 默认为org.apache.hadoop.io.compress.DefaultCodec
set hive.exec.compress.output=true;
set mapreduce.output.fileoutputformat.compress.codec=org.apache.hadoop.io.compress.SnappyCodec;
set mapreduce.output.fileoutputformat.compress.type=BLOCK;-- 默认是RECORD
set hive.auto.convert.join=true;
set hive.optimize.sort.dynamic.partition=true;-- 默认false
```

从设置来看，设置的这些参数确保了Hive任务有足够的内存、并行度和压缩机制来正确处理数据。如果没有设置这些参数，Hive任务可能会因为内存不足、并行度不够或压缩问题导致数据丢失或处理不完整。为了防止后续出现这种问题，现将这些参数永久设置。修改hive-site.xml文件

```xml
<configuration>
    <!-- 设置Reducer的字节数 -->
    <property>
        <name>hive.exec.reducers.bytes.per.reducer</name>
        <value>512000000</value> <!-- 默认值 -->
    </property>
    <!-- 设置Map输出压缩 -->
    <property>
        <name>mapreduce.map.output.compress</name>
        <value>true</value> <!-- 默认值 -->
    </property>
    <!-- 设置最大Reducer数量 -->
    <property>
        <name>hive.exec.reducers.max</name>
        <value>100</value> <!-- 默认值 -->
    </property>
    <!-- 设置Map输出压缩编解码器 -->
    <property>
        <name>mapreduce.map.output.compress.codec</name>
        <value>org.apache.hadoop.io.compress.SnappyCodec</value> <!-- 默认值 -->
    </property>
    <!-- 设置输出文件压缩类型 -->
    <property>
        <name>mapreduce.output.fileoutputformat.compress.type</name>
        <value>BLOCK</value> <!-- 默认值 -->
    </property>
    <!-- 设置动态分区优化 -->
    <property>
        <name>hive.optimize.sort.dynamic.partition</name>
        <value>true</value> <!-- 默认值 -->
    </property>
</configuration>
```



## 3.com.google.common.collect.Range.singleton(Ljava/lang/Comparable;)Lcom/google/common/collect/Range;

参考链接：[guava版本冲突](https://blog.csdn.net/qq_44766883/article/details/108582781)

1.查看一下hadoop guava包版本

```bash
cd /data/u01/app/hadoop/hadoop-3.3.6/share/hadoop/common/lib

[hadoop@cesdb lib]$ ls | grep "guava"
guava-27.0-jre.jar
hadoop-shaded-guava-1.1.1.jar
listenablefuture-9999.0-empty-to-avoid-conflict-with-guava.jar
```

2.检查hive guava包版本

```bash
cd /data/u01/app/hive/apache-hive-3.1.3/lib

[hadoop@cesdb lib]$ ls | grep "guava"
guava-13.0.jar
guava-19.0.jar.bak
jersey-guava-2.25.1.jar
```

3.很显然，两个包的版本不一样。因为我的spark版本较低，用较低版本的guava包，将hive包拷贝到hadoop包下

```bash
[hadoop@cesdb lib]$ mv guava-27.0-jre.jar guava-27.0-jre.jar.bak 
[hadoop@cesdb lib]$ ls | grep "guava"
guava-27.0-jre.jar.bak
hadoop-shaded-guava-1.1.1.jar
listenablefuture-9999.0-empty-to-avoid-conflict-with-guava.jar

 cp guava-13.0.jar /data/u01/app/hadoop/hadoop-3.3.6/share/hadoop/common/lib
```

4.最后重启hive服务

## 4.[42000][30041] Error while processing statement: FAILED: Execution Error, return code 30041 from org.apache.hadoop.hive.ql.exec.spark.SparkTask. Failed to create Spark client for Spark session 51484b3c-203f-452d-aafa-d225bb14c2c4

参考资料：[hive on spark：return code 30041 Failed to create Spark client for Spark session原因分析及解决方案探寻](https://www.cnblogs.com/JasonCeng/p/14237803.html)

先查看一下spark服务是否正常，可以没有spark master

```bash
[hadoop@cesdb hiveserver2_monitor]$ jps -l | grep -E 'Spark|spark'
139126 org.apache.spark.deploy.history.HistoryServer

[hadoop@cesdb hiveserver2_monitor]$ jps
139126 HistoryServer
71351 NodeManager
49974 Jps
188037 RunJar
71128 ResourceManager
29595 RunJar
216191 DataNode
29596 RunJar
215932 NameNode
```

因为之前都好好的，能跑程序，说明spark和hive版本是匹配的，然后查看yarn资源也是充足的。所以都不是这些问题，先重启一下hive服务。然后查看一下又正常了

```bash
[hadoop@cesdb sbin]$ jps -l | grep -E 'Spark|spark'
139126 org.apache.spark.deploy.history.HistoryServer
90252 org.apache.spark.deploy.master.Master

[hadoop@cesdb conf]$ jps
218466 Jps
168192 RunJar
168193 RunJar
139126 HistoryServer
71351 NodeManager
174612 ApplicationMaster
71128 ResourceManager
216191 DataNode
215932 NameNode
[hadoop@cesdb conf]$ 
```

## 4.com.google.common.collect.Range.atLeast(Ljava/lang/Comparable;)Lcom/google/common/collect/Range;

报这个错误是因为多创建了一个hiveserver2实例。重启一下服务就行

```bash
[hadoop@cesdb hiveserver2_monitor]$ jps
194514 RunJar
194515 RunJar
139126 HistoryServer
71351 NodeManager
150917 RunJar
71128 ResourceManager
216191 DataNode
215932 NameNode
34783 Jps

# 查看RunJar是哪个服务
ps -ef | grep 194514

# 查看进程监听的端口
[hadoop@cesdb bin]$ netstat -tulnp | grep 221219
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
tcp6       0      0 :::10000                :::*                    LISTEN      221219/java         
tcp6       0      0 :::10002                :::*                    LISTEN      221219/java         
tcp6       0      0 :::22427                :::*                    LISTEN      221219/java         
[hadoop@cesdb bin]$ netstat -tulnp | grep 221218
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
tcp6       0      0 :::9083                 :::*                    LISTEN      221218/java   
```

