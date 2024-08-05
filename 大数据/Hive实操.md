# Hive实操

## Hive设置动态分区

1.设置Hive动态分区设置

```hive
SET hive.exec.dynamic.partition = true;
SET hive.exec.dynamic.partition.mode = nonstrict;
SET hive.exec.max.dynamic.partitions.pernode = 1000;
SET hive.exec.max.dynamic.partitions = 1000;
```

2.创建外部表

```hive
CREATE EXTERNAL TABLE test.MED_OPERATION_SCHEDULE (
  PATIENT_ID STRING,
  VISIT_ID STRING,
  SCHEDULE_ID STRING,
  DEPT_STAYED STRING,
  BED_NO STRING,
  SCHEDULED_DATE_TIME TIMESTAMP,
  OPERATING_ROOM STRING,
  OPERATING_ROOM_NO STRING
)
PARTITIONED BY (year STRING, month STRING, day STRING)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
--STORED AS ORC
LOCATION '/user/hive/warehouse/test.db/surgery/MED_OPERATION_SCHEDULE';
```

3.加载分区

```hive
ALTER TABLE test.MED_OPERATION_SCHEDULE ADD IF NOT EXISTS PARTITION (year='2024', month='06', day='21');

--查看表分区
SHOW PARTITIONS test.MED_OPERATION_SCHEDULE;

--修复Hive表分区
MSCK REPAIR TABLE test.MED_OPERATION_SCHEDULE;

```

4.使用动态分区加载数据

```hive
INSERT INTO TABLE test.MED_OPERATION_SCHEDUL PARTITION (year, month, day)
SELECT
  PATIENT_ID,
  VISIT_ID,
  SCHEDULE_ID,
  DEPT_STAYED,
  BED_NO,
  SCHEDULED_DATE_TIME,
  OPERATING_ROOM,
  OPERATING_ROOM_NO,
  year,
  month,
  day
FROM
  temp_table;
```

## Hive加载hdfs上json格式数据

```sql
CREATE EXTERNAL TABLE IF NOT EXISTS test.ods_batj_patientvisit_mi (
  FID BIGINT,
  FPRN STRING,
  FTIMES INT,
  FICDVERSION TINYINT,
  FZYID STRING,
  FAGE STRING,
  FNAME STRING,
  FSEXBH STRING,
  FSEX STRING,
  FBIRTHDAY TIMESTAMP
)
PARTITIONED BY (year STRING, month STRING)
ROW FORMAT SERDE 'org.apache.hive.hcatalog.data.JsonSerDe'
STORED AS TEXTFILE
LOCATION '/user/hive/warehouse/test.db/patientvisit3';

SHOW PARTITIONS test.ods_batj_patientvisit_mi;

MSCK REPAIR TABLE test.ods_batj_patientvisit_mi;

select * from test.ods_batj_patientvisit_mi where year='2024' and month = '06';

ADD JAR /data/u01/app/hive/apache-hive-3.1.3/lib/hive-hcatalog-core-3.1.3.jar;

select count(*) from test.ods_batj_patientvisit_mi where year='2024' and month = '06';

SELECT 
    YEAR(FCYDATE) AS year,
    MONTH(FCYDATE) AS month,
    COUNT(*) AS visit_count
FROM 
    ods_batj_patientvisit_mi
WHERE 
    FCYDATE >= '2024-01-01' AND FCYDATE <= '2024-05-31'
GROUP BY 
    YEAR(FCYDATE), MONTH(FCYDATE)
ORDER BY 
    year, month;
```

## 创建时间维表

```sql
-- 创建时间维度表
CREATE EXTERNAL TABLE IF NOT EXISTS dim.dim_date (
    date_id DATE COMMENT '日期ID，格式为YYYY-MM-DD', 
    date_year INT COMMENT '年份',
    date_season INT COMMENT '季度',
    date_month INT COMMENT '月份',
    date_week INT COMMENT '周几（1表示星期一，7表示星期日）',
    date_halfyear STRING COMMENT '半年（上半年，下半年）',
    date_partmonth STRING COMMENT '旬（上旬，中旬，下旬）',
    date_monweek INT COMMENT '本月第几周',
    date_yearweek INT COMMENT '本年第几周'
)
COMMENT '时间维度表'
STORED AS ORC;


-- 使用临时表生成2024年的所有日期
CREATE EXTERNAL TABLE IF NOT EXISTS tmp.tmp_date_2024 (date_id DATE);

with dates as(
    select date_add("2024-01-01", a.pos) as d
    from (select posexplode(split(repeat("m", datediff("2024-12-31", "2024-01-01")), "m"))) a
)
-- 插入生成的日期到临时表中
INSERT INTO tmp.tmp_date_2024 (date_id)
SELECT d from dates;

select * from tmp.tmp_date_2024;

-- 插入数据到dim_time表
INSERT INTO TABLE dim.dim_date
SELECT
    d.date_id,
    YEAR(d.date_id) AS date_year,
    QUARTER(d.date_id) AS date_season,
    MONTH(d.date_id) AS date_month,
    DAYOFWEEK(d.date_id) AS date_week,
    CASE
        WHEN MONTH(d.date_id) BETWEEN 1 AND 6 THEN '上半年'
        ELSE '下半年'
    END AS date_halfyear,
    CASE
        WHEN DAY(d.date_id) BETWEEN 1 AND 10 THEN '上旬'
        WHEN DAY(d.date_id) BETWEEN 11 AND 20 THEN '中旬'
        ELSE '下旬'
    END AS date_partmonth,
    CEIL(DAY(d.date_id) / 7.0) AS date_monweek,
    WEEKOFYEAR(d.date_id) AS date_yearweek
FROM
    tmp.tmp_date_2024 d;
   
 select * from dim.dim_date;

```

## hive加载excel数据

1.先要删除表头数据，将excel另存或者到处为csv格式，数据格式要为utf-8，不然会乱码。

2.将csv文件上传到hadoop分布式文件系统中

```shell
hadoop fs -put /data/u01/soft/excel/ods_selform_compare1deptid_ig.csv /user/hive/warehouse/ods.db/excel/ods_selform_compare1deptid_ig
```

3.创建外部表加载数据

```sql
-- 创建一个与CSV文件结构匹配的表
CREATE EXTERNAL TABLE IF NOT EXISTS ods_selform_compare1deptid_ig (
    DEPT_BIZ_Id STRING,
    FCYTYKH STRING,
    FCYDEPT STRING
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE
LOCATION '/user/hive/warehouse/ods.db/excel/ods_selform_compare1deptid_ig';
```



# 问题处理

## 1.插入表数据为空的情况

跑数是正常的有数据，但是一旦插入表数据就为空。通过排查发现，数据处理和提取是正常的，但是插入表的时候数据就是为空。原因是表的数据类型不一致导致的。我创建的表的那个类型为int，而sum()函数得到的数据类型是bigint，所以数据类型不一致导致插入数据为空。将建表语句改为一致，都为bigint就可以解决问题了。

![](D:\Github\MyKnowledgeRepository\img\bigdata\hive\跑数有数据.png)

![](D:\Github\MyKnowledgeRepository\img\bigdata\hive\插入表的数据为空.png)













