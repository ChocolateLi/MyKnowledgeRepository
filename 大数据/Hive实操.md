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

## hive授权管理

编辑hive-site.xml，添加相应的配置

```
<configuration>
<property>
  <name>fs.defaultFS</name>
  <value>hdfs://cesdb:8020</value>
</property>

  <!-- Oracle JDBC connection string -->
  <property>
    <name>javax.jdo.option.ConnectionURL</name>
    <value>jdbc:oracle:thin:@10.201.100.75:1521:cesdb</value>
    <description>JDBC connect string for a JDBC metastore</description>
  </property>

  <property>
    <name>javax.jdo.option.ConnectionDriverName</name>
    <value>oracle.jdbc.OracleDriver</value>
    <description>Driver class name for a JDBC metastore</description>
  </property>

  <property>
    <name>javax.jdo.option.ConnectionUserName</name>
    <value>hive_user</value>
    <description>Username to use against metastore database</description>
  </property>

  <property>
    <name>javax.jdo.option.ConnectionPassword</name>
    <value>hive_0753</value>
    <description>Password to use against metastore database</description>
  </property>


  <!-- Hive Execution Configuration -->
  <property>
    <name>hive.execution.engine</name>
    <value>mr</value>
    <description>Execution engine to use. mr is mapreduce, tez is Tez, spark is Spark</description>
  </property>
    
    <property>
        <!--hive表在hdfs的位置-->
        <name>hive.metastore.warehouse.dir</name>
        <value>/user/hive/warehouse</value>
    </property>
<!--开启授权检查-->
    <property>
        <name>hive.security.authorization.enabled</name>
        <value>true</value>
    </property>
    <property>
        <name>hive.security.authorization.createtable.owner.grants</name>
        <value>ALL</value>
    </property>
<property>
  <name>hive.security.authorization.task.factory</name>
  <value>org.apache.hadoop.hive.ql.parse.authorization.HiveAuthorizationTaskFactoryImpl</value>
</property>
<property>
  <name>hive.users.in.admin.role</name>
  <value>hadoop</value>
  <description>定义超级管理员 启动的时候会自动创建comma separated list of users who are in admin role for bootstrapping. More users can be added in ADMIN role later.</description>
</property>

    <property>
        <name>hive.server2.enable.doAs</name>
        <value>true</value>
    </property>

<!--启用sql标准授权模式-->
<property>
  <name>hive.security.authorization.manager</name>
  <value>org.apache.hadoop.hive.ql.security.authorization.plugin.sqlstd.SQLStdHiveAuthorizerFactory</value>
</property>
<property>
  <name>hive.security.authenticator.manager</name>
  <value>org.apache.hadoop.hive.ql.security.SessionStateUserAuthenticator</value>
</property>

	<property>
  		<name>hive.metastore.partition.management.task.frequency</name>
  		<value>1d</value>
 		 <description>Frequency at which the partition management task runs to add partitions.</description>
	</property>    
<property>
  <name>hive.aux.jars.path</name>
  <value>/data/u01/app/hive/apache-hive-3.1.3/lib/hive-hcatalog-core-3.1.3.jar</value>
</property>

</configuration>

```

hive的授权操作

```sql
--展示当前角色
SHOW CURRENT ROLES;

--切换角色
SET ROLE admin;

-- 查看所有角色
SHOW ROLES;

-- 查看某个角色的权限
SHOW GRANT ROLE data_analyst;
SHOW GRANT ROLE public;

-- 创建数据统计人员角色
CREATE ROLE data_analyst;

-- 授予 data_analyst 角色对 ods、dwd、dws 和 ads 数据库的 SELECT 权限
GRANT SELECT ON DATABASE ods TO ROLE data_analyst;
GRANT SELECT ON DATABASE dwd TO ROLE data_analyst;
GRANT SELECT ON DATABASE dws TO ROLE data_analyst;
GRANT SELECT ON DATABASE ads TO ROLE data_analyst;
GRANT SELECT ON DATABASE tmp TO ROLE data_analyst;
grant select on DATABASE test to role data_analyst;

--授予角色某个表的权限
grant select on table ods.ods_med_his_users_ig to role data_analyst;
grant select on table tmp.temp_wardwordlog_mi to role data_analyst;
grant select on table test.t_user to role data_analyst;
grant select on table ods.ods_bagl_operation_mi to role data_analyst;
--直接授予用户库的权限
GRANT SELECT ON DATABASE ods TO user analyst_user;

--查看角色已有的权限
show grant role data_analyst;
SHOW GRANT ROLE public;

-- 为数据统计人员分配角色
GRANT ROLE data_analyst TO USER analyst_user;
--注意，需要Linux系统创建analyst_user用户
adduser analyst_user;
passwd analyst_user;

--列出给定角色/用户已授予的所有角色
--角色
show role grant role public;
show role grant role data_analyst;
--用户
show role grant user hadoop;
show role grant user analyst_user;

--查看角色已有权限
show grant role data_analyst;
--查看角色在某个库的权限
show grant role data_analyst on database ods;
--查看角色在某个表的权限
show grant role data_analyst on table ods.ods_med_his_users_ig ;
--查看指定用户已有权限
SHOW GRANT USER hadoop;
show grant user analyst_user;
--查看指定用户在某个库的权限
show grant user hadoop on database ods;
show grant user analyst_user on database ods;

--列出属于此角色的所有角色和用户
show principals data_analyst;
show principals admin;

--回收用户role角色
revoke role data_analyst from user analyst_user;
--回收用户权限
revoke all from user analyst_user;
--回收角色对库的所有权限
revoke select on database ods from role data_analyst;
revoke select on database dwd from role data_analyst;
revoke select on database dws from role data_analyst;
revoke select on database ads from role data_analyst;
revoke select on database tmp from role data_analyst;
revoke select on database test from role data_analyst;
--回收用户对表的权限
revoke select on TABLE ods.ods_med_dept_dict_ig from user analyst_user;
--回收用户对库的权限
revoke select on database ods from user analyst_user;

--查看当前用户信息
select current_user();

--查看某个表被谁拥有权限
SHOW GRANT ON TABLE ods.ods_med_his_users_ig;

--创建视图（源表删除了，视图也看不到了）
create view test_view as 
select * from test.ca_configcommon ;
--授权视图的查询权限给角色(直接把视图当成表来授权)
grant select on TABLE test.test_view to role data_analyst;
--授权视图的查询权限给用户
GRANT SELECT ON TABLE test.test_view TO USER username;
--回收视图权限
revoke select on TABLE test.test_view from role data_analyst;

-- 查看hdfs上的权限
hdfs dfs -getfacl /user/hive/warehouse

--hdfs上设置只读权限
hdfs dfs -setfacl -m user:view_user:rx /user/hive/warehouse
```

授权视图给data_view
```sql
--切换角色
SET ROLE admin;
--查看有哪些视图
show views in view;
--一张张表授权
grant select on TABLE view.dwd_operation_view to role data_view;
grant select on TABLE view.dwd_opsafe_01_preoperative_discussion_view to role data_view;
grant select on TABLE view.dwd_opsafe_02_preoperative_interview_view to role data_view;
grant select on TABLE view.dwd_opsafe_03_unplan_stop_operation_view to role data_view;
grant select on TABLE view.dwd_opsafe_04_safecheck1_before_ane_view to role data_view;
grant select on TABLE view.dwd_opsafe_05_safecheck2_before_op_view to role data_view;
grant select on TABLE view.dwd_opsafe_06_authorization_mismatch_view to role data_view;
grant select on TABLE view.dwd_opsafe_07_active_insulation_view to role data_view;
grant select on TABLE view.dwd_opsafe_08a_temperature_view to role data_view;
grant select on TABLE view.dwd_opsafe_08b_lowtemperature_view to role data_view;
grant select on TABLE view.dwd_opsafe_09_opdoctor_different_view to role data_view;
grant select on TABLE view.dwd_opsafe_10_cardiac_arrest_view to role data_view;
grant select on TABLE view.dwd_opsafe_11_anes_anaphylaxis_view to role data_view;
grant select on TABLE view.dwd_opsafe_12_safecheck3_after_op_view to role data_view;
grant select on TABLE view.dwd_opsafe_14a_in_pacu_view to role data_view;
grant select on TABLE view.dwd_opsafe_14b_lowtemperature_in_pacu_view to role data_view;
grant select on TABLE view.dwd_opsafe_15_delay_out_pacu_view to role data_view;
grant select on TABLE view.dwd_opsafe_16_anevisit_afterop_view to role data_view;
grant select on TABLE view.dwd_opsafe_18_complete_op_record_view to role data_view;
grant select on TABLE view.dwd_opsafe_19_op_death_view to role data_view;

--2024.12.11授权
SET ROLE admin;
grant select on TABLE view.view_ads_op_mini_test20241211 to role data_view;
```

python授权脚本

```python
from pyhive import hive
# 设置连接到Hive的参数
hive_host = "*"  # 替换为你的Hive服务器地址
hive_port = 10000  # 默认Hive端口
hive_username = "*"  # 替换为你的Hive用户名（如果需要）
hive_password = "*"  # 替换为你的Hive密码（如果需要）
# 创建Hive连接
conn = hive.Connection(host=hive_host, port=hive_port, username=hive_username, password=hive_password, auth='LDAP')
cursor = conn.cursor()
# 切换到admin角色
cursor.execute("SET ROLE admin")
# 获取view库中的所有视图
cursor.execute("SHOW VIEWS IN view")
views = cursor.fetchall()
for view in views:
    view_name = view[0]
    try:
        # 检查是否已经授予了SELECT权限
        cursor.execute(f"SHOW GRANT ROLE data_view ON TABLE view.{view_name}")
        grants = cursor.fetchall()
        
        # 检查是否有SELECT权限
        already_granted = any("SELECT" in grant[6] for grant in grants)  # grant[2] 是权限类型字段

        if not already_granted:
            # 如果未授予，则执行授权
            grant_sql = f"GRANT SELECT ON TABLE view.{view_name} TO ROLE data_view"
            cursor.execute(grant_sql)
            print(f"Granted SELECT permission on view.{view_name} to data_view")
        else:
            print(f"SELECT permission on view.{view_name} is already granted to data_view")
    except Exception as e:
        # 只输出简洁提示
        print(f"Error processing view {view_name}: Permission check or grant failed")
# 关闭连接
cursor.close()
conn.close()
```



## hive修改分区

Hive本身不支持直接修改分区名，比如你不能直接通过一个命令将 `year=2024/month=06` 修改为 `year=2024/month=05`。然而，你可以通过以下步骤间接地完成这个操作：

**重命名分区（通过添加新分区并移动数据）**

1. 创建新分区： 创建一个新的分区，目标为你希望的新分区名，如 `year=2024/month=05`。

2. 移动数据： 将原分区的数据文件从 `year=2024/month=06` 移动到 `year=2024/month=05` 的路径下。

3. 修复元数据： 使用 `MSCK REPAIR TABLE` 或 `ALTER TABLE table_name PARTITION` 更新表的元数据。

4. 删除旧分区： 删除旧分区 `year=2024/month=06`。

   ```sql
   -- 创建新的分区
   ALTER TABLE your_table_name ADD PARTITION (year=2024, month=05);
   
   -- 移动数据
   -- 需要在HDFS上操作，移动原分区的数据到新的分区路径
   hdfs dfs -mv /path/to/your_table/year=2024/month=06/* /path/to/your_table/year=2024/month=05/
   
   -- 修复表元数据
   MSCK REPAIR TABLE your_table_name;
   
   -- 删除旧分区
   ALTER TABLE your_table_name DROP PARTITION (year=2024, month=06);
   
   ```


## Hadoop删除数据

删除操作是不可逆的，请谨慎操作。

1.删除文件数据

```bash
hdfs dfs -rm /user/hadoop/example.txt
```

2.删除目录及其所有内容(-r 表示递归删除)

```bash
hdfs dfs -rm -r /user/hadoop/data
```

# 问题处理

## 1.插入表数据为空的情况

跑数是正常的有数据，但是一旦插入表数据就为空。通过排查发现，数据处理和提取是正常的，但是插入表的时候数据就是为空。原因是表的数据类型不一致导致的。我创建的表的那个类型为int，而sum()函数得到的数据类型是bigint，所以数据类型不一致导致插入数据为空。将建表语句改为一致，都为bigint就可以解决问题了。

![](D:\Github\MyKnowledgeRepository\img\bigdata\hive\跑数有数据.png)

![](D:\Github\MyKnowledgeRepository\img\bigdata\hive\插入表的数据为空.png)



## 2.FAILED: Execution Error, return code 1 from org.apache.hadoop.hive.ql.exec.StatsTask

这个问题是发生在执行StatsTask阶段，这是hive在加载数据到表时自动统计表和分区统计信息的任务。

hadoop日志以及hive日志排查方法

```
grep -i "ERROR" hadoop-hadoop-datanode-cesdb.log
grep -i "permission" hadoop-hadoop-namenode-cesdb.log
grep -i "ERROR" hadoop-hadoop-namenode-cesdb.log
grep -i "ERROR" hadoop-hadoop-historyserver-cesdb.log
grep "StatsTask" hive.log
grep -i 'ERROR' hive.log
grep "application_1733822991327_0710" hadoop-hadoop-resourcemanager-cesdb.log
```

| 配置项                       | 影响范围       | 默认值 | 作用                                                         | 适用场景                                                     |
| ---------------------------- | -------------- | ------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| hive.stats.autogather        | 表级统计信息   | true   | 是否自动收集 **表级统计信息**（如行数、文件大小等）          | 加载数据频繁但查询优化依赖表级统计时开启，否则可关闭。       |
| hive.stats.column.autogather | 列级统计信息   | true   | 是否自动收集 **列级统计信息**（如列的最大值、最小值、基数、分布等），用于更精细化的查询优化器优化。 | 数据量较大、加载频繁但查询优化不依赖列级统计信息时可关闭；对于需要复杂过滤或查询优化器依赖列的场景，建议开启或手动更新。 |
| hive.txn.stats.enabled       | 事务表统计信息 | true   | 是否自动为 **事务表**（ACID 表）维护统计信息。事务表的 `INSERT`、`UPDATE` 或 `DELETE` 操作后，会自动更新表和列的统计信息。 | 只适用于事务表（ACID 表），对非事务表无效。高频更新事务表但不依赖统计信息时可关闭。 |

目前我没有做太多操作，只是设置了以下内容，问题就解决了。

```
SET hive.stats.column.autogather=false;

<property>
  <name>hive.stats.column.autogather</name>
  <value>false</value>
</property>
```

针对重要表手动收集统计信息

```bash
ANALYZE TABLE <important_table> COMPUTE STATISTICS;
ANALYZE TABLE <important_table> COMPUTE STATISTICS FOR COLUMNS;
```









 
