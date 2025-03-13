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

## 使用内置Zookeeper

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



## 使用自己搭建的zookeeper集群

1. 无需修改conf/zookeeper.properties，保持默认配置即可。（即无需定义server.1，server.2，server.3等条目）

2. 无需在state/zookeeper文件夹下新建myid文件

3. 修改conf/nifi.properties。其他配置一致，按独立安装的Zookeeper实际情况填写。

   ```bash
   # 与使用内置的Zookeeper配置基本相同，不同的配置是
   #是否启动内置的zk
   nifi.state.management.embedded.zookeeper.start=false
   ```

4. 修改conf/state-management.xml。与使用内置的Zookeeper配置相同，按独立安装的Zookeeper实际情况填写。

5. 三个节点都要修改配置

6. 启动。三个节点都要启动。



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

## GenerateTableFetch

这个组件可实现分区查询

![](D:\Github\MyKnowledgeRepository\img\bigdata\nifi\GenerateTableFetch.png)

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

两个UpdateAttribute组件构建HDFS分区。即8月份跑七月份的数

第一个UpdateAttribute添加以下属性

```
currentMonth = ${now():format('MM')}
currentYear = ${now():format('yyyy')}
```

第二个UpdateAttribute添加以下属性

```
previousMonth = ${currentMonth:equals('01'):ifElse('12', ${currentMonth:minus(1)})}
previousYear = ${currentMonth:equals('01'):ifElse(${currentYear:minus(1)}, ${currentYear})}
```

![](D:\Github\MyKnowledgeRepository\img\bigdata\nifi\两个UpdateAttribute组件构成分区名.png)





## SplitJson

切分json数组，分成一条一条json

![](D:\Github\MyKnowledgeRepository\img\bigdata\nifi\SplitJson组件.png)



## ExecuteScript

![](D:\Github\MyKnowledgeRepository\img\bigdata\nifi\ExecuteScript.png)

**Groovy脚本示例：**

1.按天切分存储过程

```groovy
import java.time.LocalDate
import java.time.format.DateTimeFormatter
import org.apache.nifi.processor.io.OutputStreamCallback

// 定义输入参数
String startDateStr = "2022-01-01"  // 开始日期
String endDateStr = "2024-07-31"    // 结束日期

// 日期格式
DateTimeFormatter dateFormat = DateTimeFormatter.ofPattern("yyyy-MM-dd")

// 将字符串转换为日期对象
LocalDate startDate = LocalDate.parse(startDateStr, dateFormat)
LocalDate endDate = LocalDate.parse(endDateStr, dateFormat)

// 生成SQL语句
List<String> sqlStatements = []
LocalDate currentDate = startDate
while (!currentDate.isAfter(endDate)) {
    String currentDateStr = currentDate.format(dateFormat)
    //String sqlStatement = "CALL web.xspinterface4_QueryDHCWorkloadIP(\"${currentDateStr}\",\"${currentDateStr}\");"
    //String sqlStatement = "CALL web.xspinterface4_QueryDHCWorkloadOP(\"${currentDateStr}\",\"${currentDateStr}\");"
    //String sqlStatement = "CALL web.xspinterface4_QueryDHCWorkloadMaterial(\"${currentDateStr}\",\"${currentDateStr}\");"
    //String sqlStatement = "CALL web.xspinterface4_QueryDHCWorkloadDrug(\"${currentDateStr}\",\"${currentDateStr}\");"
    //String sqlStatement = "CALL web.xspinterface4_QueryDHCWorkloadOP6(\"${currentDateStr}\",\"${currentDateStr}\");"
    //String sqlStatement = "CALL web.xspinterface4_QueryDHCWorkloadOP7(\"${currentDateStr}\",\"${currentDateStr}\");"
    //String sqlStatement = "CALL web.xspinterface4_QueryDHCWorkloadIP6(\"${currentDateStr}\",\"${currentDateStr}\");"
    String sqlStatement = "CALL web.xspinterface4_QueryDHCWorkloadIP7(\"${currentDateStr}\",\"${currentDateStr}\");"
    sqlStatements.add(sqlStatement)
    currentDate = currentDate.plusDays(1)
}

// 将生成的SQL语句拼接成一个字符串，每个语句一行
String sqlStatementsStr = sqlStatements.join("\n")

// 定义一个回调类，用于将生成的SQL语句写入流文件
flowFile = session.create()
flowFile = session.write(flowFile, { outputStream ->
    outputStream.write(sqlStatementsStr.bytes)
} as OutputStreamCallback)
flowFile = session.putAttribute(flowFile, "mime.type", "text/plain")

// 将FlowFile传递给下游组件
session.transfer(flowFile, REL_SUCCESS)
```

2.按月切分sql语句

```sql
import java.time.LocalDate
import java.time.format.DateTimeFormatter
import org.apache.nifi.processor.io.OutputStreamCallback

// 定义输入参数
String startDateStr = "2024-01-01"  // 开始日期
String endDateStr = "2024-08-01"    // 结束日期

// 日期格式
DateTimeFormatter dateFormat = DateTimeFormatter.ofPattern("yyyy-MM-dd")

// 将字符串转换为日期对象
LocalDate startDate = LocalDate.parse(startDateStr, dateFormat)
LocalDate endDate = LocalDate.parse(endDateStr, dateFormat)

// 生成SQL语句
List<String> sqlStatements = []
LocalDate currentDate = startDate
while (!currentDate.isAfter(endDate.minusDays(1))) {
    LocalDate nextMonthDate = currentDate.plusMonths(1)
    String currentDateStr = currentDate.format(dateFormat)
    String nextMonthDateStr = nextMonthDate.format(dateFormat)
    String sqlStatement = """select * from cdr.CIS_INHOS_MEDICAL_RECORD where DISCHARGE_DATE>=to_date('${currentDateStr}','yyyy-mm-dd') and DISCHARGE_DATE<to_date('${nextMonthDateStr}','yyyy-mm-dd')"""
    sqlStatements.add(sqlStatement)
    currentDate = nextMonthDate
}

// 将生成的SQL语句拼接成一个字符串，每个语句一行
String sqlStatementsStr = sqlStatements.join("\n")

// 定义一个回调类，用于将生成的SQL语句写入流文件
flowFile = session.create()
flowFile = session.write(flowFile, { outputStream ->
    outputStream.write(sqlStatementsStr.bytes)
} as OutputStreamCallback)
flowFile = session.putAttribute(flowFile, "mime.type", "text/plain")

// 将FlowFile传递给下游组件
session.transfer(flowFile, REL_SUCCESS)
```

3.按天切分sql语句

```groovy
import java.time.LocalDate
import java.time.format.DateTimeFormatter
import org.apache.nifi.processor.io.OutputStreamCallback

// 定义输入参数
String startDateStr = "2020-01-01"  // 开始日期
String endDateStr = "2021-01-01"    // 结束日期

// 日期格式
DateTimeFormatter dateFormat = DateTimeFormatter.ofPattern("yyyy-MM-dd")

// 将字符串转换为日期对象
LocalDate startDate = LocalDate.parse(startDateStr, dateFormat)
LocalDate endDate = LocalDate.parse(endDateStr, dateFormat)

// 生成SQL语句
List<String> sqlStatements = []
LocalDate currentDate = startDate
while (!currentDate.isAfter(endDate.minusDays(1))) {
    LocalDate nextDayDate = currentDate.plusDays(1)
    String currentDateStr = currentDate.format(dateFormat)
    String nextDayDateStr = nextDayDate.format(dateFormat)
    String sqlStatement = """select * from cdr.CIS_OP_MEDICAL_RECORD where VISIT_DATE>=to_date('${currentDateStr}','yyyy-mm-dd') and VISIT_DATE<to_date('${nextDayDateStr}','yyyy-mm-dd')"""
    sqlStatements.add(sqlStatement)
    currentDate = nextDayDate
}

// 将生成的SQL语句拼接成一个字符串，每个语句一行
String sqlStatementsStr = sqlStatements.join("\n")

// 定义一个回调类，用于将生成的SQL语句写入流文件
flowFile = session.create()
flowFile = session.write(flowFile, { outputStream ->
    outputStream.write(sqlStatementsStr.bytes)
} as OutputStreamCallback)
flowFile = session.putAttribute(flowFile, "mime.type", "text/plain")

// 将FlowFile传递给下游组件
session.transfer(flowFile, REL_SUCCESS)
```

4.切分上月的sql

```groovy
import java.time.LocalDate
import java.time.format.DateTimeFormatter
import org.apache.nifi.processor.io.OutputStreamCallback

// 获取当前日期
LocalDate now = LocalDate.now()

// 计算开始日期为上个月的第一天
LocalDate startDate = now.withDayOfMonth(1).minusMonths(1)

// 计算结束日期为本月的第一天
LocalDate endDate = now.withDayOfMonth(1)

// 日期格式
DateTimeFormatter dateFormat = DateTimeFormatter.ofPattern("yyyy-MM-dd")

// 将日期对象转换为字符串
String startDateStr = startDate.format(dateFormat)
String endDateStr = endDate.format(dateFormat)

// 生成SQL语句
List<String> sqlStatements = []
LocalDate currentDate = startDate
while (!currentDate.isEqual(endDate)) {
    LocalDate nextDayDate = currentDate.plusDays(1)
    String currentDateStr = currentDate.format(dateFormat)
    String nextDayDateStr = nextDayDate.format(dateFormat)
    String sqlStatement = """select * from cdr.CIS_OP_MEDICAL_RECORD where VISIT_DATE>=to_date('${currentDateStr}','yyyy-mm-dd') and VISIT_DATE<to_date('${nextDayDateStr}','yyyy-mm-dd')"""
    sqlStatements.add(sqlStatement)
    currentDate = nextDayDate
}

// 将生成的SQL语句拼接成一个字符串，每个语句一行
String sqlStatementsStr = sqlStatements.join("\n")

// 定义一个回调类，用于将生成的SQL语句写入流文件
flowFile = session.create()
flowFile = session.write(flowFile, { outputStream ->
    outputStream.write(sqlStatementsStr.bytes)
} as OutputStreamCallback)
flowFile = session.putAttribute(flowFile, "mime.type", "text/plain")

// 将FlowFile传递给下游组件
session.transfer(flowFile, REL_SUCCESS)

---------------------------------------
//2025.3.11
import java.time.LocalDate;
import java.time.format.DateTimeFormatter;
import org.apache.nifi.processor.io.OutputStreamCallback;

// 获取当前日期
LocalDate now = LocalDate.now();

// 计算开始日期为上个月的第一天
LocalDate startDate = now.withDayOfMonth(1).minusMonths(1);

// 计算结束日期为本月的第一天
LocalDate endDate = now.withDayOfMonth(1)

// 日期格式
DateTimeFormatter dateFormat = DateTimeFormatter.ofPattern("yyyy-MM-dd");

// 将日期对象转换为字符串
String startDateStr = startDate.format(dateFormat);
String endDateStr = endDate.format(dateFormat);

// 生成SQL语句
String sqlStatement = """select * from CDR.CIS_INHOS_ADT_RECORD WHERE OUT_DATE >= TO_DATE('${startDateStr}', 'yyyy-mm-dd') AND OUT_DATE < TO_DATE('${endDateStr}', 'yyyy-mm-dd')
""";

// 定义一个回调类，用于将生成的SQL语句写入流文件
flowFile = session.create();
flowFile = session.write(flowFile, { outputStream ->
    outputStream.write(sqlStatement.getBytes());
} as OutputStreamCallback);
flowFile = session.putAttribute(flowFile, "mime.type", "text/plain");

// 将FlowFile传递给下游组件
session.transfer(flowFile, REL_SUCCESS);
```

5.按yyyy-MM格式切分数据

```groovy
import java.time.LocalDate
import java.time.format.DateTimeFormatter
import org.apache.nifi.processor.io.OutputStreamCallback

// 定义输入参数
String startDateStr = "2023-01"  // 开始日期
String endDateStr = "2024-01"    // 结束日期

// 日期格式
DateTimeFormatter dateFormat = DateTimeFormatter.ofPattern("yyyy-MM")

// 将字符串转换为日期对象
LocalDate startDate = LocalDate.parse(startDateStr, dateFormat).withDayOfMonth(1)
LocalDate endDate = LocalDate.parse(endDateStr, dateFormat).withDayOfMonth(1).plusMonths(1)

// 生成SQL语句
List<String> sqlStatements = []
LocalDate currentDate = startDate
while (!currentDate.isAfter(endDate.minusMonths(1))) {
    LocalDate nextMonthDate = currentDate.plusMonths(1)
    String currentDateStr = currentDate.format(dateFormat)
    String nextMonthDateStr = nextMonthDate.format(dateFormat)
    String sqlStatement = """SELECT * FROM TZYHOSPITALWORKREPORT WHERE FREPORTDATESTR >= '${currentDateStr}' AND FREPORTDATESTR < '${nextMonthDateStr}'"""
    sqlStatements.add(sqlStatement)
    currentDate = nextMonthDate
}

// 将生成的SQL语句拼接成一个字符串，每个语句一行
String sqlStatementsStr = sqlStatements.join("\n")

// 定义一个回调类，用于将生成的SQL语句写入流文件
flowFile = session.create()
flowFile = session.write(flowFile, { outputStream ->
    outputStream.write(sqlStatementsStr.bytes)
} as OutputStreamCallback)
flowFile = session.putAttribute(flowFile, "mime.type", "text/plain")

// 将FlowFile传递给下游组件
session.transfer(flowFile, REL_SUCCESS)

```

6.按yyyyMM格式切分数据

```sql
import java.time.LocalDate;
import java.time.format.DateTimeFormatter;
import org.apache.nifi.processor.io.OutputStreamCallback;

// 定义输入参数
String startDateStr = "202301";  // 开始日期
String endDateStr = "202312";    // 结束日期

// 日期格式
DateTimeFormatter dateFormat = DateTimeFormatter.ofPattern("yyyyMM");

// 修复问题：在解析时显式指定日为1日
LocalDate startDate = LocalDate.parse(startDateStr + "01", DateTimeFormatter.ofPattern("yyyyMMdd"));
LocalDate endDate = LocalDate.parse(endDateStr + "01", DateTimeFormatter.ofPattern("yyyyMMdd"));

// 生成SQL语句
List<String> sqlStatements = [];
LocalDate currentDate = startDate;
while (!currentDate.isAfter(endDate)) { // 包含结束月份
    String currentDateStr = currentDate.format(dateFormat); // 格式化当前日期为yyyyMM
    String sqlStatement = "select * from F_BASYMX_RZ where BBQ_='${currentDateStr}'";
    sqlStatements.add(sqlStatement); // 添加到列表
    currentDate = currentDate.plusMonths(1); // 加一个月
}

// 将生成的SQL语句拼接成一个字符串，每个语句一行
String sqlStatementsStr = String.join("\n", sqlStatements);

// 定义一个回调类，用于将生成的SQL语句写入流文件
flowFile = session.create();
flowFile = session.write(flowFile, new OutputStreamCallback() {
    @Override
    public void process(java.io.OutputStream outputStream) throws java.io.IOException {
        outputStream.write(sqlStatementsStr.getBytes());
    }
});
flowFile = session.putAttribute(flowFile, "mime.type", "text/plain");

// 将FlowFile传递给下游组件
session.transfer(flowFile, REL_SUCCESS);
```

切分上月Sql，只取上月

```sql
import java.time.LocalDate;
import java.time.format.DateTimeFormatter;
import org.apache.nifi.processor.io.OutputStreamCallback;

// 获取当前日期并计算上个月
LocalDate now = LocalDate.now(); // 当前日期
LocalDate lastMonth = now.minusMonths(1); // 上个月

// 日期格式
DateTimeFormatter dateFormat = DateTimeFormatter.ofPattern("yyyyMM");

// 格式化为上个月的时间
String lastMonthStr = lastMonth.format(dateFormat); // 形如 "202410"

// 生成SQL语句
String sqlStatement = "select * from F_BASYMX_RZ where BBQ_='${lastMonthStr}'";

// 定义一个回调类，用于将生成的SQL语句写入流文件
flowFile = session.create();
flowFile = session.write(flowFile, new OutputStreamCallback() {
    @Override
    public void process(java.io.OutputStream outputStream) throws java.io.IOException {
        outputStream.write(sqlStatement.getBytes());
    }
});
flowFile = session.putAttribute(flowFile, "mime.type", "text/plain");

// 将FlowFile传递给下游组件
session.transfer(flowFile, REL_SUCCESS);
```



## SplitText

切分文本组件，配合ExecuteScript组件可以切分脚本生成的Sql语句等。Line Split Count设置为1

![](D:\Github\MyKnowledgeRepository\img\bigdata\nifi\SplitText.png)

## JoltTransformJSON

可以对json数据的中文字段进行映射，转换为英文字段。

![](D:\Github\MyKnowledgeRepository\img\bigdata\nifi\JoltTransformJSON组件.jpg)

JoltSpecification内容

```json
[
  {
    "operation": "shift",
    "spec": {
      "EMPID": "empid",
      "是否在册": "is_register",
      "入册时间": "register_date",
      "在岗开始时间": "duty_startdate",
      "在岗结束时间": "duty_enddate",
      "在册_医生": "register_doctor",
      "在册_技师": "register_technician",
      "在册_药师": "register_pharmacist",
      "在册_护士": "register_nurse",
      "在册_工勤": "register_worker",
      "是否在岗": "is_duty",
      "在岗_医生": "duty_doctor",
      "在岗_技师": "duty_technician",
      "在岗_药师": "duty_pharmacist",
      "在岗_护士": "duty_nurse",
      "在岗_工勤": "duty_worker"
    }
  }
]

```



## MergeContent

合并内容。可以将切分的数据合并在一起，一起写入一个文件。比如将多条json合并在一起。

![](D:\Github\MyKnowledgeRepository\img\bigdata\nifi\MergeContent组件.jpg)



**Maximum Number of Entries**：最多几条数据合并在一起

**Delimiter Strategy**：Text

Demarcator：shift + Enter进行换行，这样就可以按照一条条数据存储，hive就可以识别json条数了。

# 同步配置

格式如下

```bash
秒 分 时 日 月 星期 年
```

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

[nifi的负载均衡策略](https://cloud.tencent.com/developer/article/2214741)

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

## 分页查询 + 集群方式同步数据

由于数据量过大，一次性读取oracle导入到hdfs上显然不太现实，因此需要分页查询数据进行导入。

整体组件如下

![](D:\Github\MyKnowledgeRepository\img\bigdata\nifi\分页查询+集群1.png)

**GenerateTableFetch组件**

Execution设置Primary node only

ConcurrentTasks设置为1

Partition Size设置分区大小为20000

**连接器设置**

Load Balance Strategy设置为Round robin

Load Balance Compression设置为Compress attributes and content

**其他组件**

Eexcution设置为All nodes

ConcurrentTasks设置为4。（因为我的数据库连接池设置为12个，3节点组成集群，所以平均每个节点为4。调节这些参数可以提高数据同步的效率。）

![](D:\Github\MyKnowledgeRepository\img\bigdata\nifi\集群数值.png)

![](D:\Github\MyKnowledgeRepository\img\bigdata\nifi\集群细节.png)

## ExecuteScript脚本的应用

![](D:\Github\MyKnowledgeRepository\img\bigdata\nifi\ExecuteScript应用1.png)

这种应用流程可以同步大数据量，多少年的数据都能同步。只需要使用ExecuteScript脚本把sql语句切分好。

为了防止写入PutHDFS上的文件被覆盖或替换，可以添加一个UpdateAttribute组件中添加filename属性，是`filename = ${UUID()} `

会随机生成文件名。

还有关键的一点就是ExecuteSQL只需要配置数据库连接那些，其他都不用设置了。

![](D:\Github\MyKnowledgeRepository\img\bigdata\nifi\ExecuteScript应用2.png)

## Json映射字段转换流程

SplitText组件是切分json为一条条数据，然后送入JoltTransformJSON进行一条条转换，最后通过MergeContent组件将一条条转换好的数据合并在一个文件进行传递。

![](D:\Github\MyKnowledgeRepository\img\bigdata\nifi\Json映射字段转换.jpg)

# Nifi集群调优

## 集群信息

三台机器，每台机器的配置是16核 32G。

nifi.properties配置文件参数调整

```bash
nifi.cluster.load.balance.connections.per.node=20
nifi.cluster.load.balance.max.thread.count=50

这两个参数与集群负载均衡的配置有关。
nifi.cluster.load.balance.connections.per.node:
含义: 这个参数指定了每个集群节点在负载均衡时允许的最大连接数。换句话说，这是集群中每个节点与其他节点之间可以建立的负载均衡连接的最大数量。
建议值: 15 到 30 之间。考虑到每台机器都有16核处理器，这样的设置能够充分利用处理器的多核能力。
作用: 如果这个值设置为20，则每个节点最多可以与集群中其他节点建立20个负载均衡连接。这有助于防止单个节点因为负载均衡连接过多而导致资源紧张或性能下降。

nifi.cluster.load.balance.max.thread.count:
含义: 这个参数指定了集群中用于处理负载均衡操作的最大线程数。
建议值: 32 到 64 之间。考虑到每台机器都有16核处理器，这样的设置能够充分利用处理器的多核能力。
作用: 如果这个值设置为50，则NiFi集群在进行负载均衡时，最多会启用50个线程来处理数据流的分发和接收。这一设置有助于控制集群节点在负载均衡过程中使用的线程资源，避免过多的线程导致系统资源耗尽或线程竞争。
```

NiFi的Controller Settings

```bash
Maximum Timer Driven Thread Count 设置为50
Maximum Event Driven Thread Count 设置为30
```

数据库连接池设置

```
Max Total Connections:
描述: 连接池中允许的最大数据库连接数，包括空闲连接和活跃连接。
当前值: 50
建议: 50个连接对于一般使用场景可能足够，但如果你的集群有多个节点，且每个节点有多个并发任务，需要确保这个值足够大以满足所有任务的需求。如果发现连接池经常耗尽，可以考虑增加此值。

Minimum Idle Connections:
描述: 连接池中保持的最小空闲连接数。当连接数低于此值时，连接池会创建新的连接以达到最小空闲连接数。
当前值: 3
建议: 如果将其设置为0，连接池将不会主动创建空闲连接，这在低负载下可以节省资源，但在高负载下可能导致连接请求等待时间增加。可以根据实际需求将其设置为一个较小的正整数。因为有三台节点，所以设置为3。

Max Idle Connections:
描述: 连接池中允许的最大空闲连接数。超过此数目的空闲连接将被关闭。
当前值: 12
建议: 12个空闲连接的设置可以在保证连接池响应速度的同时，避免过多连接占用资源。如果你的系统在高峰时段需要更快的响应，可以适当增加这个值。
```

集群策略采用的是Round robin

```
Load Balance Strategy 设置为Round robin
Load Balance Compression 设置为Do not compress
```

背压值设置

```bash
Back Pressure Object Threshold 设置为 20000
Size Threshold 设置为 3GB
```

## 分区数和处理的线程数调优

### 错误使用

| Partition Size、Max Rows Per Flow File、Output Batch Size | Concurrent Tasks | 总线程数 | 数据量   | 处理完成时间 | 平均每分钟处理数据 |
| --------------------------------------------------------- | ---------------- | -------- | -------- | ------------ | ------------------ |
| 10000                                                     | 12               | 36       | 2650395  | 11min        | 24w                |
| 10000                                                     | 15               | 45       | 2717625  | 11min        | 24.7w              |
| 20000                                                     | 15               | 45       | 2638644  | 10min        | 26.4w              |
| 40000                                                     | 15               | 45       | 2661862  | 9min         | 29.5w              |
| 20000                                                     | 15               | 45       | 2363157  | 8min         | 29.5w              |
| 20000                                                     | 15               | 45       | 7427224  | 27min        | 27.5w              |
| 20000                                                     | 15               | 45       | 6101878  | 17min        | 35.8w              |
| 20000                                                     | 15               | 45       | 6980872  | 16min        | 43.6w              |
| 20000                                                     | 15               | 45       | 7613864  | 16min        | 47.5w              |
| 20000                                                     | 15               | 45       | 8893198  | 22min        | 40.4w              |
| 20000                                                     | 15               | 45       | 10577631 | 队列阻塞     | 队列阻塞           |

**总结**

从给定的数据中，我们可以分析出以下几点：

1. **分区大小和处理效率的关系：**

   - 当`Partition Size`较小时（如10,000），即使增加了并发任务和线程数，总体的处理效率提升并不明显。例如，`Partition Size`为10,000时，无论`Concurrent Tasks`是12还是15，处理时间都保持在11分钟左右，且平均每分钟处理数据在24w左右。
   - 当`Partition Size`增大到20,000和40,000时，处理时间明显缩短，且平均每分钟处理的数据量增加。例如，在`Partition Size`为20,000时，处理时间从10分钟减少到8分钟，而在40,000时，处理时间进一步缩短至9分钟。

2. **数据量与处理时间的关系：**

   - 当数据量较小时（如2,638,644），处理时间和每分钟处理的数据量之间存在明显的提升趋势。例如，数据量为2,638,644时，处理时间为10分钟，平均每分钟处理数据为26.4w；而当数据量增加到7,427,224时，处理时间为27分钟，但平均每分钟处理数据量也提高到27.5w。
   - 当数据量继续增加到8,893,198时，处理时间延长到22分钟，但平均每分钟处理数据量下降到40.4w，说明系统在处理大数据量时效率有所下降。
   - 数据量为10,577,631时系统卡住，表明此时系统达到了瓶颈，无法继续处理。

3. **总线程数的影响：**

   - `总线程数`与处理数据的速度和效率有一定关系，但并不是唯一的决定因素。例如，在多个实例中，`总线程数`保持在45，但处理时间和效率仍然有较大差异，这表明影响因素还包括分区大小和数据量。

4. **系统瓶颈：**

   - 数据显示，当`Partition Size`为20,000，数据量达到10,577,631时，系统出现卡住的情况，这说明系统在处理较大数据量时可能存在内存或处理能力的瓶颈。

总结：

- **适当增加`Partition Size`** 可以显著提高处理效率，特别是当`Partition Size`从10,000增加到20,000或40,000时，处理时间明显减少。
- **数据量过大** 可能导致系统效率降低或直接卡住，表明需要对系统的配置（如内存、CPU等）进行优化以应对更大数据量的处理需求。
- **总线程数** 对于处理效率有一定帮助，但在达到一定数值后，单纯增加线程数并不会继续显著提升处理效率。

建议在实际应用中，根据数据量和处理需求，合理设置`Partition Size`，并监控系统资源使用情况，以避免出现处理瓶颈。

### 正确使用

通过切分数据查询的方式可以实现亿万级别的数据同步。

| 数据量   | 处理完成时间 | 平均每分钟处理数据 |
| -------- | ------------ | ------------------ |
| 48295103 | 14 min       | 345w               |
| 52537171 | 8 min        | 656.7w             |
| 59883152 | 8 min        | 748.5w             |



# 报错总结

## java.lang.NoClassDefFoundError: oracle/xdb/XMLType

nifi的ExecuteSQL组件同步oracle数据库时，报这个错误。只需要下载xdb6.jar，放在nifi安装目录下的lib包下重启即可。

## java.lang.NoClassDefFoundError: oracle/xml/binxml/BinXMLMetadataProvider

添加了xdb6.jar，重启后，还是同步不了数据，报这个错误。按照Oracle官方的意思是需要xmlparsev2.jar包。但是当我把jar包放在Nifi的lib包下时，会导致nifi重启失败，重启之后立马关闭。可能是因为jar包冲突的原因。

后面找到的原因是因为同步的表中的某个字段是SYS.XMLTYPE这个类型，所以导致报xml相关jar包的错误。

解决的办法就是，同步表的时候，不同步这个SYS.XMLTYPE字段就行，这样就可以解决这个问题。



## X.509 Peer certificate not valid

检查证书有效期，证书在nifi的conf文件夹下

```bash
keytool -list -v -keystore keystore.p12 -storetype PKCS12
keytool -list -v -keystore truststore.p12 -storetype PKCS12

keytool -list -v -keystore /data/u01/app/nifi/nifi-1.26.0/conf/keystore.p12 -storepass 12ecb0217c0eefb17d5f419dcec16ad4
keytool -list -v -keystore /data/u01/app/nifi/nifi-1.26.0/conf/truststore.p12 -storepass 5274324704b2e56de62c2936d401e216

```

1. **生成新的自签名证书**

首先，在第一个节点上生成新的 keystore 和信任库，并导出证书：

在节点1上：

```bash
# 生成新的 keystore 和自签名证书
keytool -genkeypair -alias nifi-key -keyalg RSA -keystore keystore.p12 -storetype PKCS12 -storepass new_keystore_password -keypass new_key_password -validity 365
# 1.自己操作的测试，可行
keytool -genkeypair -alias nifi-key -keyalg RSA -keystore /data/chenli/crt/keystore.p12 -storetype PKCS12 -storepass 12ecb0217c0eefb17d5f419dcec16ad4 -keypass 12ecb0217c0eefb17d5f419dcec16ad4 -validity 365


# 导出自签名证书
keytool -export -alias nifi-key -keystore keystore.p12 -storepass new_keystore_password -file nifi-cert.cr
# 2.自己操作的测试，可行
keytool -export -alias nifi-key -keystore /data/chenli/crt/keystore.p12 -storepass 12ecb0217c0eefb17d5f419dcec16ad4 -file /data/chenli/crt/nifi-cert.crt

# 3.导入证书到信任库，自己测试可行
keytool -importcert -alias nifi-cert -file /data/chenli/crt/nifi-cert.crt -keystore /data/chenli/crt/truststore.p12 -storetype PKCS12 -storepass 5274324704b2e56de62c2936d401e216

# 4.将证书拷贝到conf文件夹下
cp keystore.p12 /data/u01/app/nifi/nifi-1.26.0/conf/
cp truststore.p12 /data/u01/app/nifi/nifi-1.26.0/conf/

```

2. **将证书分发到其他节点**

将生成的 `keystore.p12`、`truststore.p12` 和 `nifi-cert.crt` 文件复制到其他两个节点上（节点2和节点3）。你可以使用 `scp` 或其他文件传输方法。

在节点1上：

```bash
# 分发证书
scp keystore.p12 root@cesdb1:/data/u01/app/nifi/nifi-1.26.0/conf/
scp truststore.p12 root@cesdb1:/data/u01/app/nifi/nifi-1.26.0/conf/

scp keystore.p12 root@cesdb2:/data/u01/app/nifi/nifi-1.26.0/conf/
scp truststore.p12 root@cesdb2:/data/u01/app/nifi/nifi-1.26.0/conf/

```

3. **更新 NiFi 配置文件**

在每个节点上更新 `nifi.properties` 配置文件，使其指向新的 keystore 和 truststore 文件，并使用新的密码。

编辑 `nifi.properties` 文件：

```properties
nifi.security.keystore=./conf/keystore.p12
nifi.security.keystoreType=PKCS12
nifi.security.keystorePasswd=new_keystore_password
nifi.security.keyPasswd=new_key_password
nifi.security.truststore=./conf/truststore.p12
nifi.security.truststoreType=PKCS12
nifi.security.truststorePasswd=new_truststore_password

# 自己操作的
nifi.security.autoreload.enabled=false
nifi.security.autoreload.interval=10 secs
nifi.security.keystore=./conf/keystore.p12
nifi.security.keystoreType=PKCS12
nifi.security.keystorePasswd=12ecb0217c0eefb17d5f419dcec16ad4
nifi.security.keyPasswd=12ecb0217c0eefb17d5f419dcec16ad4
nifi.security.truststore=./conf/truststore.p12
nifi.security.truststoreType=PKCS12
nifi.security.truststorePasswd=5274324704b2e56de62c2936d401e216
nifi.security.user.authorizer=single-user-authorizer
nifi.security.allow.anonymous.authentication=false
nifi.security.user.login.identity.provider=single-user-provider
nifi.security.user.jws.key.rotation.period=PT1H
nifi.security.ocsp.responder.url=
nifi.security.ocsp.responder.certificate=
```

4. **重启所有 NiFi 节点**

在每个节点上分别重启 NiFi：

在节点1、节点2和节点3上分别执行：

```bash
./bin/nifi.sh restart
```

5.**验证和监控**

**节点列表**：

- `cesdb2:8081`：状态为 `CONNECTED`。
- `cesdb1:8081`：状态为 `CONNECTED`。
- `cesdb:8081`：状态为 `CONNECTED, PRIMARY, COORDINATOR`。

**活跃线程计数（Active Thread Count）**： 所有节点的活跃线程计数为 0，表示当前没有正在处理的数据流任务。

**队列大小（Queue / Size）**： 所有节点的队列大小为 0 / 0 bytes，表示当前没有排队等待处理的数据。

**最后一次心跳（Last Heartbeat）**： 所有节点的最后一次心跳时间都在近期，表明节点之间的通信是持续且正常的。

**启动时间（Started At）**： 所有节点的启动时间相近，表明它们最近被同时启动。

**PRIMARY 节点**

`PRIMARY` 节点是指当前集群中选定的主要节点。某些处理器可以配置为仅在 `PRIMARY` 节点上运行，以避免在多个节点上重复执行相同的任务。这在处理特定任务时非常有用，例如数据库查询或生成唯一标识符等。

**COORDINATOR 节点**

`COORDINATOR` 节点是集群中的协调节点，负责管理集群中其他节点的状态和任务分配。`COORDINATOR` 节点处理集群中的一些管理任务，例如：

- 管理集群成员的加入和离开。
- 处理节点之间的通信和协调。
- 分配数据流任务和处理器的执行。

![](D:\Github\MyKnowledgeRepository\img\bigdata\nifi\nifi集群状态显示图.png)

## NotAuthorizedException: Client IDs[] are not authorized to Load Balance data

2024-08-06 10:58:22,509 ERROR [Load-Balanced Client Thread-5] o.a.n.c.q.c.c.a.n.NioAsyncLoadBalanceClient Unable to connect to cesdb1:8081 for load balancing java.io.IOException: Failed to decrypt data from Peer /10.201.100.75:47627::cesdb1/10.201.100.84:6342 because Peer unexpectedly closed connection

这个一般是集群通信的问题，没有认证等之类的，涉及到的配置是nifi.properties这个文件下的security配置，将这些置为空就行。将这些置为空意味着不使用安全认证了。

```
nifi.security.keystore=
nifi.security.keystoreType=
nifi.security.keystorePasswd=
nifi.security.keyPasswd=
nifi.security.truststore=
nifi.security.truststoreType=
nifi.security.truststorePasswd=
```

## ProcessException:java.sql.SQLException: Cannot get a connection, pool error Timeout waitingfor idle object, borrowMaxWaitDuration=PT0.5S.Caused by: java.sql.SQLException

确认NiFi的数据库连接池配置是否正确。这包括最大连接数、最小空闲连接数、最大等待时间等。

如果错误是由于连接池资源不足引起的，考虑增加最大连接数。这可以在NiFi的数据库连接池配置中设置。

调整连接池的空闲连接数和连接超时设置，以优化资源使用和响应时间。

这是因为超过了我数据库连接池设置的线程数，原本默认的线程数为8，但我在组件配置了10个线程，导致报错。

修改数据库连接池的线程数或者组件配置的线程数，使组件配置的线程数低于数据库连接池的线程数就行。
