# 保姆级Doris使用指南

# Nifi数据同步

## Oracle T+1 同步

只需要修改SQL部分就行

```groovy
import java.time.LocalDate;
import java.time.format.DateTimeFormatter;
import org.apache.nifi.processor.io.OutputStreamCallback;

// 获取当前日期
LocalDate now = LocalDate.now();

// 默认取昨天的数据（按天取数）
LocalDate startDate = now.minusDays(1);
LocalDate endDate = now;

// 日期格式
DateTimeFormatter dateFormat = DateTimeFormatter.ofPattern("yyyy-MM-dd");

// 将日期对象转换为字符串
String startDateStr = startDate.format(dateFormat);
String endDateStr = endDate.format(dateFormat);

// 生成SQL语句 - 按天查询
// 只需要修改SQL部分就行
String sqlStatement = """ select * from ODS_USER.test_student 
where createtime >= to_date('${startDateStr}', 'yyyy-mm-dd') and createtime < to_date('${endDateStr}', 'yyyy-mm-dd')""";

// 创建FlowFile并写入SQL语句
flowFile = session.create();
flowFile = session.write(flowFile, { outputStream ->
    outputStream.write(sqlStatement.getBytes("UTF-8"));
} as OutputStreamCallback);
flowFile = session.putAttribute(flowFile, "mime.type", "text/plain");

// 将FlowFile传递给下游组件
session.transfer(flowFile, REL_SUCCESS);
```

## Oracle 自定义按天切分SQL同步

```groovy
import java.time.LocalDate
import java.time.format.DateTimeFormatter
import org.apache.nifi.processor.io.OutputStreamCallback

// 定义输入参数
String startDateStr = "2025-07-01"  // 开始日期
String endDateStr = "2025-08-02"    // 结束日期

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
    String sqlStatement = """select a.*,b.START_DATE_TIME from MED_OPERATION_NAME a inner join MED_OPERATION_MASTER b on a.patient_id=b.patient_id and a.VISIT_ID=b.VISIT_ID and a.OPER_ID=b.OPER_ID where b.START_DATE_TIME>=to_date('${currentDateStr}','yyyy-mm-dd') and b.START_DATE_TIME<to_date('${nextDayDateStr}','yyyy-mm-dd')"""
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



## SqlServer T+1同步



## SqlServer 自定义按月切分SQL同步



# SQL语句

## 建库建表语句

建库

```sql
create database ods
```

建表

`根据以下表结构帮我创建一个doris明细表 ods.ods_med_operation_name_df`

```sql
-- 明细表
CREATE TABLE ods.ods_med_operation_name_df (
    `patient_id` VARCHAR(20) COMMENT '患者ID',
    `visit_id` SMALLINT COMMENT '就诊ID',
    `oper_id` SMALLINT COMMENT '手术ID',
    `operation_no` SMALLINT COMMENT '手术序号',
    `operation` VARCHAR(200) COMMENT '手术名称',
    `operation_code` VARCHAR(400) COMMENT '手术代码',
    `operation_scale` VARCHAR(50) COMMENT '手术等级',
    `wound_grade` VARCHAR(2) COMMENT '切口等级',
    `reserved1` VARCHAR(20) COMMENT '预留字段1',
    `reserved2` VARCHAR(20) COMMENT '预留字段2',
    `reserved3` VARCHAR(20) COMMENT '预留字段3',
    `reserved4` VARCHAR(20) COMMENT '预留字段4',
    `reserved5` INT COMMENT '预留字段5',
    `oper_version` VARCHAR(30) COMMENT '手术版本',
    `START_DATE_TIME` DATE COMMENT '开始时间',
    `etl_date` DATE DEFAULT CURRENT_DATE COMMENT 'ETL处理日期', -- 这个是记录是哪个时间同步的数据
    `etl_time` DATETIME DEFAULT CURRENT_TIMESTAMP COMMENT 'ETL处理时间'
)
ENGINE=OLAP
DUPLICATE KEY(`patient_id`, `visit_id`, `oper_id`, `operation_no`)
COMMENT '手术名称明细表'
PARTITION BY RANGE(`START_DATE_TIME`)()
DISTRIBUTED BY HASH(`patient_id`) BUCKETS 6 -- 按照目前的数据量，分6个桶可以
PROPERTIES (
    "dynamic_partition.enable" = "true",
    "dynamic_partition.time_unit" = "MONTH", -- 按月分区
    "dynamic_partition.start" = "-36",  -- 保留3年的数据
    "dynamic_partition.end" = "3",      -- 预建未来3个月
    "dynamic_partition.prefix" = "p",
    "dynamic_partition.buckets" = "6", -- 每月数据小于10GB的话建议就分6个桶，大于10GB小于50GB的话就分15个桶。
    "replication_num" = "2", -- 副本为2就行
    "dynamic_partition.create_history_partition" = "true" -- 确保此项为true，这样才会创建历史分区
);
```

