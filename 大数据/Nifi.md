# Nifi

# Nifi集群搭建

1.下载安装包解压

[下载地址](https://nifi.apache.org/download/)

```bash
tar -zxvf nifi-1.26.0-bin.zip -C /data/u01/app/nifi/
```

2.修改端口号和路径

```
[root@cesdb conf]# vim nifi.properties 

nifi.web.http.host=10.201.100.75
nifi.web.http.port=8081

```

3.启动

```bash
bin/nifi.sh start  #启动命令
bin/nifi.sh stop   #关闭命令
bin/nifi.sh status #查看运行状态
```

这种方式运行的是单机版的NiFi，如果要进行分布式安装，还需要进行其他一些配置。

分布式的NiFi需要使用Zookeeper作为集群的管理工具，NiFi本身内置了Zookeeeper，也可以使用独立安装的Zookeeper。

[参考链接](https://www.jianshu.com/p/20f9ac79f8d0)

**使用内置Zookeeper**

对每一个节点完成以下配置（建议在一个节点配置好，分发给其他节点再稍作修改）

1.修改conf/zookeeper.properties

```bash
server.1=cesdb:2888:3888;2181
server.2=cesdb1:2888:3888;2181
server.3=cesdb2:2888:3888;2181
```

2.执行以下脚本（zookeeper不同节点需要不同id）

```
cd state/
mkdir zookeeper
echo 1 > ./zookeeper/myid
```

3.修改conf/nifi.properties

```bash
#是否启动内置的zk
nifi.state.management.embedded.zookeeper.start=true
#配置zk节点
nifi.zookeeper.connect.string=cesdb:2181,cesdb1:2181,cesdb2:2181
nifi.state.management.embedded.zookeeper.start=true
nifi.cluster.is.node=true
# nifi.cluster.node.address=
nifi.cluster.node.address=cesdb (根据节点修改)
#nifi.cluster.node.address=cesdb1 
#nifi.cluster.node.address=cesdb2
nifi.cluster.node.protocol.port=9998
nifi.cluster.flow.election.max.candidates=1

nifi.web.http.host=cesdb(根据节点修改)
# nifi.web.http.host=cesdb1(根据节点修改)
# nifi.web.http.host=cesdb2(根据节点修改)

nifi.remote.input.host=cesdb
# nifi.remote.input.host=cesdb1
# nifi.remote.input.host=cesdb2
nifi.remote.input.socket.port=10443

```

4.修改conf/state-management.xml

主要是修改`Connect String`，和第三步中的`nifi.zookeeper.connect.string`一致。

```
<cluster-provider>
        <id>zk-provider</id>
        <class>org.apache.nifi.controller.state.providers.zookeeper.ZooKeeperStateProvider</class>
        <property name="Connect String">cesdb:2181,cesdb1:2181,cesdb2:2181</property>
        <property name="Root Node">/nifi</property>
        <property name="Session Timeout">10 seconds</property>
        <property name="Access Control">Open</property>
    </cluster-provider>

```

5.分发安装包

```bash
scp -r /data/u01/app/nifi root@cesdb1:/data/u01/app/
scp -r /data/u01/app/nifi root@cesdb2:/data/u01/app/
```

6.修改cesdb1和cesdb2的myid和nifi.properties文件

```bash
# cesdb1
vim myid 
2

vim nifi.properties 
nifi.remote.input.host=cesdb1
nifi.web.http.host=cesdb1
nifi.cluster.node.address=cesdb1 
# cesdb2
vim myid 
3

vim nifi.properties 
nifi.remote.input.host=cesdb2
nifi.web.http.host=cesdb2
nifi.cluster.node.address=cesdb2
```

7.三个节点都要启动

```bash
bin/nifi.sh start

# 访问哪一个都行
http://cesdb:8081/nifi/
http://cesdb1:8081/nifi/
http://cesdb2:8081/nifi/
```

![](D:\Github\MyKnowledgeRepository\img\bigdata\nifi\nifi集群.png)



# 数据源配置

## Oracle数据源配置

Database Connetion URL

```sql
//oracle服务的方式
jdbc:oracle:thin:@//199.168.200.55:1521/docare

//oracle sid的方法是
jdbc:oracle:thin:@10.201.100.75:1521:cesdb
```

Database Driver Class Name

```sql
oracle.jdbc.driver.OracleDriver
```

Database Driver Location(s)

需要下载相应的jar包放在对应的路径上

```sql
/data/u01/soft/ojdbc8.jar
```

![](D:\Github\MyKnowledgeRepository\img\bigdata\nifi\oracle数据源配置.png)

## Sqlserver数据源配置

Database Connetion URL

```
jdbc:sqlserver://199.168.200.244;databasename=bagl_java
```

Database Driver Class Name

```
com.microsoft.sqlserver.jdbc.SQLServerDriver
```

Database Driver Location(s)

需要下载相应的jar包放在对应的路径上

```
/data/u01/soft/sqljdbc4.jar
```



# 组件用法

## QueryDatabaseTable

查询数据库表的数据，输出的是Avro格式的数据。

![](D:\Github\MyKnowledgeRepository\img\bigdata\nifi\QueryDatabaseTable组件.png)

## ExecuteSQL

执行sql查询，跟QueryDatabaseTable类似，也是查询数据，但这个可以自定义sql数据，比较灵活，让多个表进行关联等。QueryDatabaseTable只能查询一个表。

![](D:\Github\MyKnowledgeRepository\img\bigdata\nifi\ExecuteSQL组件.png)





## PutSQL

将sql数据插入到数据库中，搭配ConvertAvroToJSON使用。

![](D:\Github\MyKnowledgeRepository\img\bigdata\nifi\PutSQL组件.png)

## PutDatabaseRecord

批量插入数据，需要配置对应上游数据的reader。比如上游的数据格式是Avro，那么这里就要配置Avro相关的Reader。

![](D:\Github\MyKnowledgeRepository\img\bigdata\nifi\PutDatabaseRecord组件.png)

## PutHDFS

把数据存储到hdfs路径上

```
# Hadoop配置文件路径
Hadoop Configuration Resources：/data/u01/app/hadoop/hadoop-3.3.6/etc/hadoop/hdfs-site.xml,/data/u01/app/hadoop/hadoop-3.3.6/etc/hadoop/core-site.xml

# Hadoop存储的目录
Directory:/user/hive/warehouse/test.db/surgery/med_dept_dict_2

# 添加动态分区
/user/hive/warehouse/test.db/surgery/MED_OPERATION_SCHEDULE_4/year=${year}/month=${month}
```



![](D:\Github\MyKnowledgeRepository\img\bigdata\nifi\PutHDFS组件.png)



## ConvertRecord

转化格式的一个组件，比如说想将Avro的格式转化为csv格式等。所以这个组件需要配置Reader和Writer组件配置

![](D:\Github\MyKnowledgeRepository\img\bigdata\nifi\ConvertRecord组件.png)



## ConvertAvroToJSON

将Avro格式的数据转化为json格式的数据

![](D:\Github\MyKnowledgeRepository\img\bigdata\nifi\ConvertAvroToJSON组件.png)

## ConvertJSONToSQL

将json格式数据转化为sql形式，从而方便插入数据库。Statement Type要选择插入方式，还有更新方式可以选择。要选择插入的数据库表等

![](D:\Github\MyKnowledgeRepository\img\bigdata\nifi\ConvertJSONToSQL.png)

## UpdateAttribute

更新属性，比如说添加时间属性等。也可以添加文件名等操作。

![](D:\Github\MyKnowledgeRepository\img\bigdata\nifi\UpdateAttribute组件.png)

![](D:\Github\MyKnowledgeRepository\img\bigdata\nifi\添加文件名.png)



## SplitJson

切分json数组，分成一条一条json

![](D:\Github\MyKnowledgeRepository\img\bigdata\nifi\SplitJson组件.png)



# 同步配置

## 按天同步

每天凌晨一点同步

![](D:\Github\MyKnowledgeRepository\img\bigdata\nifi\按天同步配置.png)

## 按月同步

按照每月1号凌晨一点开始同步

![](D:\Github\MyKnowledgeRepository\img\bigdata\nifi\按月同步配置.png)

# 数据源同步

## oracle_to_oracle(sqlserver_to_oracle)

- QueryDatabaseTable方式

  ![](D:\Github\MyKnowledgeRepository\img\bigdata\nifi\QueryDatabaseTable同步方式.png)

- ExecuteSQL方式

  ![](D:\Github\MyKnowledgeRepository\img\bigdata\nifi\ExecuteSQL同步方式.png)

- JsonSQL方式

  ![](D:\Github\MyKnowledgeRepository\img\bigdata\nifi\JsonSQL同步方式.png)

## oracle_to_hdfs

### 以csv格式存储到hdfs中

- ExecuteSQL组件和QueryDatabaseTable组件可以交替使用
- ConvertRecord 要选择这两个配置 Reader选择AvroReader Writer选择CSVRecordSetWriter
- UpdateAttribute组件根据需要添加，可以添加可不添加。

​	![](D:\Github\MyKnowledgeRepository\img\bigdata\nifi\csv格式存储到Hdfs.png)

### 以json格式存储（成功实践）

由于csv格式会因为分隔符号","、"\t"导致数据切分错误，从而导致数据不一致的问题，并且格式错乱。因为将数据以json格式存储，可以保证数据的完整性。

![](D:\Github\MyKnowledgeRepository\img\bigdata\nifi\oracle_to_hdfs_json.png)

ConvertRecord配置如下
![](D:\Github\MyKnowledgeRepository\img\bigdata\nifi\JsonRecordSetWriter配置1.png)

![](D:\Github\MyKnowledgeRepository\img\bigdata\nifi\JsonRecordSetWriter配置2.png)

## 集群方式的同步

如果使用了nifi集群，那么在配置上需要注意两点：一是头结点Execution的配置，二是连接器Load Balance Strategy的配置。

Execution有两个选择：1.All nodes：所有节点 2.Primary node：主节点。

**对应单节点的NiFi来说上面两种选择都是没有区别的，对于集群来说的话，头处理器一般都是选择Primary node，其他处理器选择All nodes，因为在头处理器是整个任务的起点，选择所有节点的话，每个节点都会去执行相同的任务，这肯定不是我们所想要的，我们只需要一个节点执行就可以了。集群模式下，如果头处理器选择All nodes，就会导致同步到多份数据，三个节点就会同步到三份数据。**

连接器的Setting中有一个Load Balance Strategy，它有四种负载均衡策略选项：

**1.Do not load balance**：不进行负载均衡

**2.Partition by attribute**：按属性分区

**3.Round robin**：轮询

**4.Single node**：单节点

如果没有特别要求，建议选择轮询的方式。

![](D:\Github\MyKnowledgeRepository\img\bigdata\nifi\集群配置1.png)

![](D:\Github\MyKnowledgeRepository\img\bigdata\nifi\集群配置2.png)

![](D:\Github\MyKnowledgeRepository\img\bigdata\nifi\集群配置3.png)

# 报错总结

## java.lang.NoClassDefFoundError: oracle/xdb/XMLType

nifi的ExecuteSQL组件同步oracle数据库时，报这个错误。只需要下载xdb6.jar，放在nifi安装目录下的lib包下重启即可。

## java.lang.NoClassDefFoundError: oracle/xml/binxml/BinXMLMetadataProvider

添加了xdb6.jar，重启后，还是同步不了数据，报这个错误。按照Oracle官方的意思是需要xmlparsev2.jar包。但是当我把jar包放在Nifi的lib包下时，会导致nifi重启失败，重启之后立马关闭。可能是因为jar包冲突的原因。

后面找到的原因是因为同步的表中的某个字段是SYS.XMLTYPE这个类型，所以导致报xml相关jar包的错误。

解决的办法就是，同步表的时候，不同步这个SYS.XMLTYPE字段就行，这样就可以解决这个问题。
