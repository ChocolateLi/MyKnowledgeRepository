# KingBase

# Kingbase安装

参考链接：[Linux平台安装](https://docs.kingbase.com.cn/cn/KES-V9R1C10/install/iso/iso-linux-install#%E5%91%BD%E4%BB%A4%E8%A1%8C%E5%A2%9E%E5%88%A0%E7%BB%84%E4%BB%B6)

## 安装前准备工作

安装用户

```bash
[root@doris soft]# useradd -u2000 kingbase
[root@doris soft]# passwd kingbase
```

小插曲

```sql
[kingbase@openEuler V9]$ su root
密码： 
su: 权限被拒绝

说明Kingbase没有权限切换到root，给它添加权限
[root@openEuler ~]# groups kingbase
kingbase : kingbase
[root@openEuler ~]# usermod -aG wheel kingbase
[root@openEuler ~]# groups kingbase
kingbase : kingbase wheel

或者直接编辑/etc/sudoers，不推荐。
```

安装目录

```bash
# 自定义的安装目录
[root@doris app]# mkdir -p /data/u01/app/kingbase/ES/V9
[root@doris app]# chmod o+rwx /data/u01/app/kingbase/ES/V9/
# 更改用户组和用户
[root@doris app]# chown -R kingbase:kingbase /data/u01/app/kingbase
```

数据目录

```bash
# 切换kingbase用户操作，无特殊说明，接下来都是用kingbase用户操作
su kingbase
[kingbase@doris kingbase]$ mkdir -p /data/u01/app/kingbase/ES/V9/data
```

安装包准备

```bash
# 切换root用户
# 将安装文件上传至/data/u01/soft
KingbaseES_V009R001C010B0004_Lin64_install.iso
# 创建目录目录
[root@doris soft]# mkdir KingbaseESV9
[root@doris soft]# ll
总用量 6790868
drwxr-xr-x. 3 root root         24 7月   3 16:45 apache-doris-2.1.10-bin-x64
-rw-r--r--. 1 root root 2933616665 6月  26 14:34 apache-doris-2.1.10-bin-x64.tar.gz
drwxr-xr-x. 2 root root       4096 6月  24 17:34 gcc
-rw-r--r--. 1 root root 2974226432 8月   5 16:19 KingbaseES_V009R001C010B0004_Lin64_install.iso
drwxr-xr-x. 2 root root          6 8月   6 10:35 KingbaseESV9
-rw-r--r--. 1 root root 1045995520 7月   4 10:14 mysql-8.0.38-1.el7.x86_64.rpm-bundle.tar
drwxrwxr-x. 2 root root        130 7月   4 15:07 telnet
```

安装包的挂载与取消

```bash
# iso格式的安装程序包需要先挂载才能使用,需要使用root用户挂载iso文件
[root@doris soft]# cd /data/u01/soft/
[root@doris soft]# mount KingbaseES_V009R001C010B0004_Lin64_install.iso ./KingbaseESV9
mount: /dev/loop0 写保护，将以只读方式挂载
[root@doris soft]# cd ./KingbaseESV9/
# 可以看到setup目录和setup.sh脚本
[root@doris KingbaseESV9]# ls
setup  setup.sh
# 安装完成后可以运行如下命令取消挂载.iso文件
umount ./KingbaseESV9
# 此时KingbaseES已经和iso文件解除挂载关系，在KingbaseES目录下不会再看到安装相关文件。
```

## 安装KingbaseES

1.启动安装程序

```bash
# 先查看系统语言
[root@doris backup]# echo $LANG
zh_CN.UTF-8
# 进入setup.sh所在目录，以kingbase用户执行
[root@doris backup]# cd /data/u01/soft/KingbaseESV9/
[root@doris KingbaseESV9]# su kingbase
[kingbase@doris KingbaseESV9]$ sh setup.sh -i console

# 输入授权地址
/data/u01/soft/license_58791_0.dat

# 输入一个绝对路径，或按ENTER键以接受缺省路径 [默认： /opt/Kingbase/ES/V9]
/data/u01/app/kingbase/ES/V9

# 最后成功的界面
  恭喜您！安装完成

  安装目录：
    /data/u01/app/kingbase/ES/V9

  
    引导组件:/data/u01/app/kingbase/ES/V9/KESRealPro/V009R001C010/install
    产品手册:/data/u01/app/kingbase/ES/V9/KESRealPro/V009R001C010/doc
    数据库运维工具:/data/u01/app/kingbase/ES/V9/KESRealPro/V009R001C010/SupTools
    数据库服务器:/data/u01/app/kingbase/ES/V9/KESRealPro/V009R001C010/Server
    高可用组件:/data/u01/app/kingbase/ES/V9/KESRealPro/V009R001C010/KingbaseHA
    接口:/data/u01/app/kingbase/ES/V9/KESRealPro/V009R001C010/Interface
    数据库集群部署工具:/data/u01/app/kingbase/ES/V9/KESRealPro/V009R001C010/ClientTools/guitools/DeployTools
    数据库迁移工具:/data/u01/app/kingbase/ES/V9/KESRealPro/V009R001C010/ClientTools/guitools/KDts
    数据库开发工具(CS):/data/u01/app/kingbase/ES/V9/KESRealPro/V009R001C010/ClientTools/guitools/KStudio


如需初始化数据库，请启动Kconsole:
/data/u01/app/kingbase/ES/V9/Server/bin/kconsole.sh
手动初始化数据库:
/data/u01/app/kingbase/ES/V9/Server/bin/initdb -U "system" -W -D "/data/u01/app/kingbase/ES/V9/data"
[ Writing the uninstaller data ... ]
[ 命令行安装完成 ]
```

## 初始化数据库

```bash
[kingbase@doris bin]$ /data/u01/app/kingbase/ES/V9/Server/bin/initdb -U "system" -W -D "/data/u01/app/kingbase/ES/V9/data"
创建目录 /data/u01/app/kingbase/ES/V9/data ... 成功
正在创建子目录 ... 成功
选择动态共享内存实现 ......posix
选择默认最大联接数 (max_connections) ... 100
选择默认共享缓冲区大小 (shared_buffers) ... 128MB
选择默认时区...Asia/Shanghai
创建配置文件 ... 成功
开始设置加密设备
正在初始化加密设备...成功
正在运行自举脚本 ...成功
正在执行自举后初始化 ...成功
创建安全数据库...成功
加载安全数据库...成功
同步数据到磁盘...成功

initdb: 警告: enabling "trust" authentication for local connections
你可以通过编辑 sys_hba.conf 更改或你下次
执行 initdb 时使用 -A或者--auth-local和--auth-host选项.

成功。您现在可以用下面的命令开启数据库服务器：

    /data/u01/app/kingbase/ES/V9/Server/bin/sys_ctl -D /data/u01/app/kingbase/ES/V9/data -l 日志文件 start


超级用户口令：kingbase_0753
```

启动数据库

```bash
[kingbase@doris data]$ /data/u01/app/kingbase/ES/V9/Server/bin/sys_ctl -D /data/u01/app/kingbase/ES/V9/data -l 日志文件 start
等待服务器进程启动 .... 完成
服务器进程已经启动
[kingbase@doris data]$ netstat -tulnp | grep 54321
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
tcp        0      0 0.0.0.0:54321           0.0.0.0:*               LISTEN      24934/kingbase      
tcp6       0      0 :::54321                :::*                    LISTEN      24934/kingbase      
[kingbase@doris data]$ netstat -tulnp | grep 54321
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
tcp        0      0 0.0.0.0:54321           0.0.0.0:*               LISTEN      24934/kingbase      
tcp6       0      0 :::54321                :::*                    LISTEN      24934/kingbase      
[kingbase@doris data]$ 

#关闭数据库
/data/u01/app/kingbase/ES/V9/Server/bin/sys_ctl -D /data/u01/app/kingbase/ES/V9/data stop

# 2. 重载配置（无需重启）
/data/u01/app/kingbase/ES/V9/Server/bin/sys_ctl reload -D /data/u01/app/kingbase/ES/V9/data
```

登录数据

```bash
./ksql -U system test;
./ksql -U system kingbase;
```

重新加载授权文件（直接替换就行）

```bash
# 替换license.dat后边的软连接文件
[root@openEuler soft]# mv license.dat /data/u01/app/kingbase/ES/V9/KESRealPro/V009R001C010/
mv: 是否覆盖 '/data/u01/app/kingbase/ES/V9/KESRealPro/V009R001C010/license.dat'？

# 记得修改授权文件的权限
cd /data/u01/app/kingbase/ES/V9/KESRealPro/V009R001C010/
chown kingbase:kingbase license.dat 
```

# KFS安装

参考链接：[KFS安装部署](https://bbs.kingbase.com.cn/docHtml?recId=c3f448eede450dfbd8cadf68a991b40f&url=aHR0cHM6Ly9iYnMua2luZ2Jhc2UuY29tLmNuL2tpbmdiYXNlLWRvYy9rZnMvaW5zdGFsbC11cGRhdGUvaW5zdGFsbGF0aW9uLWd1aWRlL2luZGV4Lmh0bWw)

最后界面

```bash
===============================================================================
安装完成
----

恭喜！金仓数据同步管理平台 已成功地安装到：

/data/u01/app/KFS

 如果您需要将 金仓数据同步管理平台 注册为系统服务，
请以 root 身份运行

/data/u01/app/KFS/scripts/Root.sh

启动服务后访问金仓数据同步管理平台，地址:

http://localhost:8089/login

单击“完成”以退出安装程序。

按 <ENTER> 键以退出安装程序: 
```

# SQL

## 设置库属性

```sql
-- 允许包含空字符（不推荐，可能影响数据完整性）
ALTER DATABASE test SET bytea_output = 'escape';
-- 验证设置已恢复
SHOW bytea_output;
-- 恢复到默认状态
ALTER DATABASE test RESET bytea_output;
```



## 创建和管理用户

创建用户并授予权限

```sql
-- 创建用户
CREATE USER data WITH 
  PASSWORD 'data_0753'
  NOSUPERUSER
  NOCREATEDB
  NOCREATEROLE
  INHERIT
  NOREPLICATION
  CONNECTION LIMIT -1;
  
-- 创建了 data 用户,授予了该用户对 ods 模式的所有权限，但是，模式权限不会自动延伸到该模式中已存在的表上。您需要显式地授予表权限。
  
-- 仅授予data用户对ods.measurement_interval表的查询权限
GRANT SELECT ON TABLE ods.measurement_interval TO data;

-- 授予data用户对ods模式中所有表的全部权限
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA ods TO data;

-- 设置ods模式中未来创建的表自动授予data用户全部权限
ALTER DEFAULT PRIVILEGES IN SCHEMA ods GRANT ALL PRIVILEGES ON TABLES TO data;

-- 授予 data 角色对 dwd schema 的所有权限
GRANT ALL PRIVILEGES ON SCHEMA dwd TO data;
-- 授予 data 角色对 dws schema 的所有权限
GRANT ALL PRIVILEGES ON SCHEMA dws TO data;
-- 授予 data 角色对 ads schema 的所有权限
GRANT ALL PRIVILEGES ON SCHEMA ads TO data;
-- 授予 data 角色在这些 schema 中创建表的权限
ALTER DEFAULT PRIVILEGES IN SCHEMA ods, dwd, dws, ads GRANT ALL PRIVILEGES ON TABLES TO data;
-- 授予 data 角色在这些 schema 中创建序列的权限
ALTER DEFAULT PRIVILEGES IN SCHEMA ods, dwd, dws, ads GRANT ALL PRIVILEGES ON SEQUENCES TO data;
-- 授予 data 角色在这些 schema 中创建函数的权限
ALTER DEFAULT PRIVILEGES IN SCHEMA ods, dwd, dws, ads GRANT ALL PRIVILEGES ON FUNCTIONS TO data;

-- 以超级用户或system用户执行（这个后续再执行，先验证一下system用户创建的ods层表data是否有权限访问）
ALTER DEFAULT PRIVILEGES 
FOR ROLE system  -- 指定当system用户创建对象时
IN SCHEMA ods    -- 在ods schema中
GRANT ALL ON TABLES TO data;  -- 自动授予data用户所有表权限
```

迁移数据库后授权

```sql
-- 授予数据库连接权限
GRANT CONNECT, TEMPORARY ON DATABASE datawarehouse TO data;

-- 为每个模式授予权限
GRANT USAGE ON SCHEMA ods TO data;
GRANT USAGE ON SCHEMA dwd TO data;
GRANT USAGE ON SCHEMA dws TO data;
GRANT USAGE ON SCHEMA ads TO data;
GRANT USAGE ON SCHEMA public TO data;

-- 授予现有表的所有 DML 权限
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA ods TO data;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA dwd TO data;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA dws TO data;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA ads TO data;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO data;

-- 授予序列权限
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA ods TO data;
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA dwd TO data;
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA dws TO data;
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA ads TO data;
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA public TO data;

-- 设置默认权限（对未来创建的对象生效）
ALTER DEFAULT PRIVILEGES IN SCHEMA ods GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO data;
ALTER DEFAULT PRIVILEGES IN SCHEMA dwd GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO data;
ALTER DEFAULT PRIVILEGES IN SCHEMA dws GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO data;
ALTER DEFAULT PRIVILEGES IN SCHEMA ads GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO data;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO data;

ALTER DEFAULT PRIVILEGES IN SCHEMA ods GRANT USAGE, SELECT ON SEQUENCES TO data;
ALTER DEFAULT PRIVILEGES IN SCHEMA dwd GRANT USAGE, SELECT ON SEQUENCES TO data;
ALTER DEFAULT PRIVILEGES IN SCHEMA dws GRANT USAGE, SELECT ON SEQUENCES TO data;
ALTER DEFAULT PRIVILEGES IN SCHEMA ads GRANT USAGE, SELECT ON SEQUENCES TO data;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT USAGE, SELECT ON SEQUENCES TO data;
```

查看用户权限

```sql
-- 查看用户属性
SELECT * FROM pg_roles WHERE rolname = 'data';
SELECT * FROM pg_user WHERE usename = 'data';

-- 查看用户对数据库的权限
SELECT datname, datacl FROM pg_database;

-- 查看用户对特定模式的权限
SELECT nspname, nspacl FROM pg_namespace WHERE nspname = 'ods';

-- 查看用户对特定表的权限
SELECT table_schema, table_name, privilege_type 
FROM information_schema.table_privileges 
WHERE grantee = 'data';

-- 查看ods模式中所有表的权限
SELECT grantee, table_schema, table_name, privilege_type 
FROM information_schema.table_privileges 
WHERE table_schema = 'ods' AND grantee = 'data';

-- 查看用户对表中列的权限
SELECT column_name, privilege_type 
FROM information_schema.column_privileges 
WHERE grantee = 'data' AND table_name = 'measurement_interval';

-- 查看用户对函数的权限
SELECT proname, proacl FROM pg_proc;
```

回收权限并删除用户

```sql
-- 先撤销角色的所有权限
REVOKE ALL PRIVILEGES ON ALL TABLES IN SCHEMA ods, dwd, dws, ads FROM data;
REVOKE ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA ods, dwd, dws, ads FROM data;
REVOKE ALL PRIVILEGES ON ALL FUNCTIONS IN SCHEMA ods, dwd, dws, ads FROM data;
REVOKE ALL PRIVILEGES ON SCHEMA ods, dwd, dws, ads FROM data;
REVOKE CONNECT ON DATABASE test FROM data;
-- 清除表默认权限
ALTER DEFAULT PRIVILEGES IN SCHEMA ods REVOKE ALL ON TABLES FROM data;
ALTER DEFAULT PRIVILEGES IN SCHEMA dwd REVOKE ALL ON TABLES FROM data;
ALTER DEFAULT PRIVILEGES IN SCHEMA dws REVOKE ALL ON TABLES FROM data;
ALTER DEFAULT PRIVILEGES IN SCHEMA ads REVOKE ALL ON TABLES FROM data;
-- 清除序列默认权限
ALTER DEFAULT PRIVILEGES IN SCHEMA ods REVOKE ALL ON SEQUENCES FROM data;
ALTER DEFAULT PRIVILEGES IN SCHEMA dwd REVOKE ALL ON SEQUENCES FROM data;
ALTER DEFAULT PRIVILEGES IN SCHEMA dws REVOKE ALL ON SEQUENCES FROM data;
ALTER DEFAULT PRIVILEGES IN SCHEMA ads REVOKE ALL ON SEQUENCES FROM data;
-- 清除函数默认权限
ALTER DEFAULT PRIVILEGES IN SCHEMA ods REVOKE ALL ON FUNCTIONS FROM data;
ALTER DEFAULT PRIVILEGES IN SCHEMA dwd REVOKE ALL ON FUNCTIONS FROM data;
ALTER DEFAULT PRIVILEGES IN SCHEMA dws REVOKE ALL ON FUNCTIONS FROM data;
ALTER DEFAULT PRIVILEGES IN SCHEMA ads REVOKE ALL ON FUNCTIONS FROM data;
-- 删除用户
DROP USER IF EXISTS data;
```

创建cwm_db

```sql
CREATE SCHEMA cwm_db;

-- 授予data用户对cwm_db模式中所有表的全部权限
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA cwm_db TO data;

-- 设置cwm_db模式中未来创建的表自动授予data用户全部权限
ALTER DEFAULT PRIVILEGES IN SCHEMA cwm_db GRANT ALL PRIVILEGES ON TABLES TO data;

-- 授予 data 角色对 cwm_db schema 的所有权限
GRANT ALL PRIVILEGES ON SCHEMA cwm_db TO data;
```

## 创建表

创建模式和表空间

```sql
-- 创建分层Schema
CREATE SCHEMA ods;   -- 贴源层
CREATE SCHEMA dwd;   -- 明细层
CREATE SCHEMA dws;   -- 汇总层
CREATE SCHEMA ads;   -- 应用层

-- 表空间分配（按性能需求）
CREATE TABLESPACE ods_ts LOCATION '/data/kingbase/ods';  -- 普通存储
CREATE TABLESPACE dws_ts LOCATION '/data/kingbase/dws';  -- SSD存储

-- 创建表时执行表空间
CREATE TABLE dws.dept_reg_daily (
    -- 字段定义
) TABLESPACE dws_ts;

-- 修改现有表空间结构
ALTER TABLE ods.patient_visit SET TABLESPACE ods_ts;

-- 将2020年分区移动到归档表空间
ALTER TABLE measurement_interval_p202001 SET TABLESPACE archive_ts;
```

分区表

```sql
-- 查看所有分区物理文件位置及大小
SELECT 
    child.relname AS partition_name,
    pg_relation_filepath(child.oid) AS file_path,
    pg_get_expr(child.relpartbound, child.oid) AS partition_range,
    pg_size_pretty(pg_total_relation_size(child.oid)) AS partition_size
FROM pg_inherits
JOIN pg_class parent ON pg_inherits.inhparent = parent.oid
JOIN pg_class child ON pg_inherits.inhrelid = child.oid
WHERE parent.relname = 'measurement_interval';

-- 重命名分区
ALTER TABLE measurement_interval_p202001 
RENAME TO measurement_interval_202001;

-- 查看分区扫描情况（实际查询过的分区）
SELECT
    relname AS partition_name,
    seq_scan AS full_scans,
    idx_scan AS index_scans,
    n_live_tup AS live_rows
FROM pg_stat_user_tables
WHERE relname LIKE 'ods_med_operation_master_df_p%'
ORDER BY relname;

-- 根据以下oracle表结构，帮我生成一个一模一样的kingbase数据库的表结构，表的名称为ods.ods_med_operation_master_df，其中varchar字段长度乘以3倍，顺便帮我添加一个etl_date这个字段，它是根据系统时间自动生成的
-- 创建分区表，以3个月为一个分区
CREATE TABLE ods.ods_med_operation_name_df (
    PATIENT_ID VARCHAR(60) NULL,
    VISIT_ID NUMERIC(2,0) NULL,
    OPER_ID NUMERIC(2,0) NULL,
    OPERATION_NO NUMERIC(2,0) NULL,
    OPERATION VARCHAR(600) NULL,
    OPERATION_CODE VARCHAR(1200) NULL,
    OPERATION_SCALE VARCHAR(150) NULL,
    WOUND_GRADE VARCHAR(6) NULL,
    RESERVED1 VARCHAR(60) NULL,
    RESERVED2 VARCHAR(60) NULL,
    RESERVED3 VARCHAR(60) NULL,
    RESERVED4 VARCHAR(60) NULL,
    RESERVED5 NUMERIC(6,0) NULL,
    OPER_VERSION VARCHAR(90) NULL,
    START_DATE_TIME TIMESTAMP NULL,
    etl_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP -- etl时间
)PARTITION BY RANGE (START_DATE_TIME) INTERVAL ('3 MONTH'::INTERVAL) (
    PARTITION p1 VALUES LESS THAN ('2024-04-01') 
);

-- 快速清空单个分区（保留分区结构）
TRUNCATE TABLE ods.patient_visit_202401;
```

灵活的分区管理，自定义创建未来分区

```sql
-- 创建分区表（基础表）
CREATE TABLE ods.med_anesthesia_event_di (
    PATIENT_ID VARCHAR2(60) NULL,
    VISIT_ID NUMBER(2,0) NULL,
    OPER_ID NUMBER(2,0) NULL,
    ITEM_NO NUMBER(3,0) NULL,
    ITEM_CLASS VARCHAR2(3) NULL,
    EVENT_NO NUMBER(3,0) NULL,
    ITEM_NAME VARCHAR2(180) NULL,
    ITEM_CODE VARCHAR2(48) NULL,
    ITEM_SPEC VARCHAR2(60) NULL,
    DOSAGE_UNITS VARCHAR2(30) NULL,
    DOSAGE NUMBER(8,4) NULL,
    ADMINISTRATOR VARCHAR2(24) NULL,
    START_TIME DATE NULL,
    END_DATE DATE NULL,
    BILL_INDICATOR NUMBER(1,0) NULL,
    DURATIVE_INDICATOR NUMBER(1,0) NULL,
    "METHOD" VARCHAR2(48) NULL,
    PERFORM_SPEED NUMBER(8,4) NULL,
    SPEED_UNIT VARCHAR2(30) NULL,
    PARENT_ITEM_NO NUMBER(3,0) NULL,
    EVENT_ATTR VARCHAR2(30) NULL,
    CONCENTRATION NUMBER(8,4) NULL,
    CONCENTRATION_UNIT VARCHAR2(30) NULL,
    BILL_ATTR NUMBER(1,0) NULL,
    SUPPLIER_NAME VARCHAR2(180) NULL,
    METHOD_PARENT_NO NUMBER(3,0) NULL,
    etl_time TIMESTAMP DEFAULT SYSTIMESTAMP
)
PARTITION BY RANGE (START_TIME);

-- 创建分区（从2020年开始）
ALTER TABLE ods.med_anesthesia_event_di ADD PARTITION p2020 VALUES LESS THAN (TO_DATE('2021-01-01', 'YYYY-MM-DD'));
ALTER TABLE ods.med_anesthesia_event_di ADD PARTITION p2021 VALUES LESS THAN (TO_DATE('2022-01-01', 'YYYY-MM-DD'));
ALTER TABLE ods.med_anesthesia_event_di ADD PARTITION p2022 VALUES LESS THAN (TO_DATE('2023-01-01', 'YYYY-MM-DD'));
ALTER TABLE ods.med_anesthesia_event_di ADD PARTITION p2023 VALUES LESS THAN (TO_DATE('2024-01-01', 'YYYY-MM-DD'));
ALTER TABLE ods.med_anesthesia_event_di ADD PARTITION p2024 VALUES LESS THAN (TO_DATE('2025-01-01', 'YYYY-MM-DD'));
ALTER TABLE ods.med_anesthesia_event_di ADD PARTITION p_max VALUES LESS THAN (MAXVALUE);
```



## 查询表

查询同步的时间段范围是否有为0的数据

```sql
-- 使用generate_series生成日期序列
SELECT all_dates.visit_date, 
       COALESCE(ccomrd.record_count, 0) as data_count
FROM (
    SELECT generate_series(
        (SELECT MIN(visit_date) FROM ods.cdr_cis_op_medical_record_di),
        (SELECT MAX(visit_date) FROM ods.cdr_cis_op_medical_record_di),
        '1 day'::interval
    )::date as visit_date
) all_dates
LEFT JOIN (
    SELECT visit_date, COUNT(*) as record_count
    FROM ods.cdr_cis_op_medical_record_di 
    GROUP BY visit_date
) ccomrd ON all_dates.visit_date = ccomrd.visit_date
WHERE COALESCE(ccomrd.record_count, 0) = 0
ORDER BY all_dates.visit_date;

-- 知道具体的时间范围
SELECT all_dates.visit_date, 
       COALESCE(ccomrd.record_count, 0) as data_count
FROM (
    SELECT generate_series(
        '2022-01-01'::date,  -- 开始日期
        '2024-12-31'::date,  -- 结束日期
        '1 day'::interval
    )::date as visit_date
) all_dates
LEFT JOIN (
    SELECT visit_date, COUNT(*) as record_count
    FROM ods.cdr_cis_op_medical_record_di 
    GROUP BY visit_date
) ccomrd ON all_dates.visit_date = ccomrd.visit_date
WHERE COALESCE(ccomrd.record_count, 0) = 0
ORDER BY all_dates.visit_date;

-- 使用EXTRACT确保日期匹配（针对有时分秒的）
SELECT all_dates.visit_date, 
       COALESCE(ccomrd.record_count, 0) as data_count
FROM (
    SELECT generate_series(
        '2025-01-01'::date,
        '2025-09-10'::date,
        '1 day'::interval
    )::date as visit_date
) all_dates
LEFT JOIN (
    SELECT 
        EXTRACT(YEAR FROM visit_date) as year,
        EXTRACT(MONTH FROM visit_date) as month,
        EXTRACT(DAY FROM visit_date) as day,
        COUNT(*) as record_count
    FROM ods.cdr_cis_op_medical_record_di 
    GROUP BY EXTRACT(YEAR FROM visit_date), 
             EXTRACT(MONTH FROM visit_date), 
             EXTRACT(DAY FROM visit_date)
) ccomrd ON EXTRACT(YEAR FROM all_dates.visit_date) = ccomrd.year
        AND EXTRACT(MONTH FROM all_dates.visit_date) = ccomrd.month
        AND EXTRACT(DAY FROM all_dates.visit_date) = ccomrd.day
WHERE COALESCE(ccomrd.record_count, 0) = 0
ORDER BY all_dates.visit_date;
```

按天统计数据

```sql
-- kingbase
SELECT TO_CHAR(visit_date, 'yyyy-MM-dd') as visit_date_formatted, 
       COUNT(*) as record_count
FROM ods.cdr_cis_op_medical_record_di_p4 
GROUP BY TO_CHAR(visit_date, 'yyyy-MM-dd')
ORDER BY TO_CHAR(visit_date, 'yyyy-MM-dd');

-- oracle
SELECT TRUNC(VISIT_DATE) as visit_day,
       COUNT(*) as record_count
FROM CDR.CIS_OP_MEDICAL_RECORD
WHERE VISIT_DATE >= to_date('2025-01-01','yyyy-mm-dd')
  AND VISIT_DATE < to_date('2025-09-11','yyyy-mm-dd')
GROUP BY TRUNC(VISIT_DATE)
ORDER BY TRUNC(VISIT_DATE);
```

## 重命名数据库

```sql
-- 查询当前连接到test数据库的会话
SELECT pid, usename, application_name, client_addr, state 
FROM sys_stat_activity 
WHERE datname = 'test';

-- 终止所有连接到test数据库的会话
SELECT pg_terminate_backend(pid) 
FROM sys_stat_activity 
WHERE datname = 'test' AND pid <> pg_backend_pid();

-- 现在可以重命名数据库
ALTER DATABASE test RENAME TO datawarehouse;
```

## 备份全库

```sql
# 新库创建数据库
create DATABASE datawarehouse
# 新库创建用户
CREATE USER data WITH 
  PASSWORD 'data_0753'
  NOSUPERUSER
  NOCREATEDB
  NOCREATEROLE
  INHERIT
  NOREPLICATION
  CONNECTION LIMIT -1;

# 备份全库：
cd /data/u01/app/kingbase/ES/V9/Server/bin
# ./sys_dump -h ip -p 端口 -U 用户  -f 备份路径/xxx.sql 库名
./sys_dump -h 192.168.200.180 -p 54321 -U system  -f /data/chenli/backup/datawarehouse.sql test
# 还原全库：
# ./ksql  -h ip -U用户名  -d 库名 -f  备份路径/xxx.sql
./ksql  -h 192.168.200.181 -U system  -d datawarehouse -f  /data/chenli/backup/datawarehouse.sql
```

