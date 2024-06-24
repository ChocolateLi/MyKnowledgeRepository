# Nifi

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

更新属性，比如说添加时间属性等

![](D:\Github\MyKnowledgeRepository\img\bigdata\nifi\UpdateAttribute组件.png)



## SplitJson

切分json数组，分成一条一条json

![](D:\Github\MyKnowledgeRepository\img\bigdata\nifi\SplitJson组件.png)





# 数据源同步

## oracle_to_oracle(sqlserver_to_oracle)

- QueryDatabaseTable方式

  ![](D:\Github\MyKnowledgeRepository\img\bigdata\nifi\QueryDatabaseTable同步方式.png)

- ExecuteSQL方式

  ![](D:\Github\MyKnowledgeRepository\img\bigdata\nifi\ExecuteSQL同步方式.png)

- JsonSQL方式

  ![](D:\Github\MyKnowledgeRepository\img\bigdata\nifi\JsonSQL同步方式.png)

## oracle_to_hdfs

- 以json格式存储到hdfs中

  ![](D:\Github\MyKnowledgeRepository\img\bigdata\nifi\json格式存储到hdfs.png)

- 以csv格式存储到hdfs中

  - ExecuteSQL组件和QueryDatabaseTable组件可以交替使用
  - ConvertRecord 要选择这两个配置 Reader选择AvroReader Writer选择CSVRecordSetWriter
  - UpdateAttribute组件根据需要添加，可以添加可不添加。

​	![](D:\Github\MyKnowledgeRepository\img\bigdata\nifi\csv格式存储到Hdfs.png)

