# Hive建表语句模板

## 表操作

### 数据类型

| **分类** | **类型**  | **描述**                                       | **字面量示例**                                               |
| -------- | --------- | ---------------------------------------------- | ------------------------------------------------------------ |
| 原始类型 | BOOLEAN   | true/false                                     | TRUE                                                         |
|          | TINYINT   | 1字节的有符号整数 -128~127                     | 1Y                                                           |
|          | SMALLINT  | 2个字节的有符号整数，-32768~32767              | 1S                                                           |
|          | INT       | 4个字节的带符号整数                            | 1                                                            |
|          | BIGINT    | 8字节带符号整数                                | 1L                                                           |
|          | FLOAT     | 4字节单精度浮点数1.0                           |                                                              |
|          | DOUBLE    | 8字节双精度浮点数                              | 1.0                                                          |
|          | DEICIMAL  | 任意精度的带符号小数                           | 1.0                                                          |
|          | STRING    | 字符串，变长                                   | “a”,’b’                                                      |
|          | VARCHAR   | 变长字符串                                     | “a”,’b’                                                      |
|          | CHAR      | 固定长度字符串                                 | “a”,’b’                                                      |
|          | BINARY    | 字节数组                                       |                                                              |
|          | TIMESTAMP | 时间戳，毫秒值精度                             | 122327493795                                                 |
|          | DATE      | 日期                                           | ‘2016-03-29’                                                 |
|          |           | 时间频率间隔                                   |                                                              |
| 复杂类型 | ARRAY     | 有序的的同类型的集合                           | array(1,2)                                                   |
|          | MAP       | key-value,key必须为原始类型，value可以任意类型 | map(‘a’,1,’b’,2)                                             |
|          | STRUCT    | 字段集合,类型可以不同                          | struct(‘1’,1,1.0), named_stract(‘col1’,’1’,’col2’,1,’clo3’,1.0) |
|          | UNION     | 在有限取值范围内的一个值                       | create_union(1,’a’,63)                                       |

### 基础建表

```sql
CREATE [EXTERNAL] TABLE tb_name
	(col_name col_type [COMMENT col_comment], ......)
	[COMMENT tb_comment]
	[PARTITIONED BY(col_name, col_type, ......)]
	[CLUSTERED BY(col_name, col_type, ......) INTO num BUCKETS]
	[ROW FORMAT DELIMITED FIELDS TERMINATED BY '']
	[LOCATION 'path']
```

- `[EXTERNAL]`，外部表。（全部建议采用外部表的方式）

  - `[ROW FORMAT DELIMITED FIELDS TERMINATED BY '']`指定列分隔符

  - `[LOCATION 'path']`表数据路径（不知道路径可省略）

    ```sql
    -- 外部表示意
    CREATE EXTERNAL TABLE test_ext(
        id int,
        name string
    ) 
    COMMENT 'external table' 
    ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t';
    ```

- `[COMMENT tb_comment]`表注释，可选

- `[PARTITIONED BY(col_name, col_type, ......)]`基于列分区

  ```sql
  -- 分区表示意
  CREATE EXTERNAL TABLE test_ext(
      id int,
      name string
  ) 
  COMMENT 'partitioned table' 
  PARTITION BY(year string, month string, day string) 
  ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t';
  ```

### 示例

**科室病房日志事实表创建示例**

```sql
-- 科室病房日志事实表，分区表
CREATE EXTERNAL TABLE IF NOT EXISTS dwd.dwd_inhos_wardwordlog_mi (
    dept_id STRING COMMENT '科室ID',
    dept_name STRING COMMENT '科室名称',
    date_id DATE COMMENT '日期ID',
    openbeds BIGINT  COMMENT '开放床位数',
    patient_original BIGINT  COMMENT '原有人数',
    patient_newly BIGINT  COMMENT '入院人数',
    patient_transin BIGINT  COMMENT '他科转入',
    patient_discharge BIGINT  COMMENT '出院人数',
    patient_transout BIGINT  COMMENT '转往他科',
    patient_last BIGINT  COMMENT '现有人数',
    beddays_act_occupy BIGINT  COMMENT '实际占用床日数',
    beddays_discha_occupy BIGINT  COMMENT '出院者占用床日数'
)
COMMENT '科室病房日志事实表'
PARTITIONED BY (year STRING, month STRING)
STORED AS ORC;

-- 插入数据到表中
SET hive.exec.dynamic.partition = true;
SET hive.exec.dynamic.partition.mode = nonstrict;

with t1 as(
SELECT fcytykh,fcydept,fcydate
,sum(case when A.FCYDATE=A.FRYDATE then 1 else 0 end) frs--当天出入院人数
,sum(a.fdays) fds --出院患者住院天数和
FROM ods.ods_bagl_patientvisit_mi a
WHERE a.year='2024'
group by fcytykh,fcydept,fcydate
)
INSERT OVERWRITE TABLE dwd.dwd_inhos_wardwordlog_mi PARTITION (year, month)
select  
c.dept_biz_id as dept_id,              --科室ID
b.fksname as dept_name,                --科室名称
b.frq     AS date_id,    --日期ID
b.fbedsum as openbeds,                 --开放床位数
b.FYRS as patient_original,            --原有人数
b.fryrs as patient_newly,              --入院人数
b.FTKZR as patient_transin,            --他科转入
b.FCYRS as patient_discharge,          --出院人数
b.FZRTK as patient_transout,           --转往他科
b.FXYRS as patient_last,               --现有人数
case when t1.frs is null then b.FXYRS else t1.frs+b.FXYRS end  as beddays_act_occupy,--实际占用床日数
case when t1.fds is null then 0 else t1.fds end as  beddays_discha_occupy,       --出院者占用床日数
YEAR(b.frq) AS year, --年份分区
MONTH(b.frq) AS month
FROM ods.ods_bagl_wardworklog_mi b 
left join  t1 on t1.fcytykh=b.ftykh and t1.fcydate=b.frq
left join ods.ods_selform_compare1deptid_ig as c  on c.fcytykh =b.ftykh
WHERE b.year='2024';
```

**时间维表创建示例**

```sql
-- 创建时间维度表，不是分区表
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
```

