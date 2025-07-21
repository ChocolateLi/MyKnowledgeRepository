# Doris使用指南

# Doris 角色与权限管理详细指南

## 一、Doris 权限系统概述

Doris 提供了基于角色的访问控制(RBAC)系统，主要包含以下概念：
- **用户(User)**：登录系统的账户
- **角色(Role)**：权限的集合，可以分配给用户
- **权限(Privilege)**：对数据库对象的操作权限

## 二、创建角色和用户

### 1. 创建统计师角色

```sql
-- 创建统计师角色
CREATE ROLE statistician;
```

### 2. 创建用户

```sql
-- 创建4个统计师用户
CREATE USER 'stat_user1' IDENTIFIED BY 'password1';
CREATE USER 'stat_user2' IDENTIFIED BY 'password2';
CREATE USER 'stat_user3' IDENTIFIED BY 'password3';
CREATE USER 'stat_user4' IDENTIFIED BY 'password4';

create user 'chenli' identified by 'chenli26';
-- 创建的用户默认拥有information_schema和mysql数据库的查询权限，需要撤回。
-- 不能撤回，一旦撤回的话，其他库也看不到。mysql库的权限可以撤回，但是information_schema库权限不行，这是元数据。
-- 撤销information_schema库的所有权限
REVOKE SELECT_PRIV ON information_schema.* FROM 'chenli';
-- 撤销mysql库的所有权限
REVOKE SELECT_PRIV ON mysql.* FROM 'chenli';
```

### 3. 将用户分配给角色

```sql
-- 将用户分配给统计师角色
GRANT statistician TO 'stat_user1';
GRANT statistician TO 'stat_user2';
GRANT statistician TO 'stat_user3';
GRANT statistician TO 'stat_user4';
```

## 三、为角色授予数据库权限

### 1. 授予统计师角色基本权限

```sql
-- 授予统计师角色对ods、dwd、dws、ads库的所有表查询权限
GRANT SELECT_PRIV ON ods.* TO ROLE statistician;
GRANT SELECT_PRIV ON dwd.* TO ROLE statistician;
GRANT SELECT_PRIV ON dws.* TO ROLE statistician;
GRANT SELECT_PRIV ON ads.* TO ROLE statistician;

-- 授予创建、删除表权限
GRANT CREATE_PRIV ON ods.* TO ROLE statistician;
GRANT DROP_PRIV ON ods.* TO ROLE statistician;
GRANT CREATE_PRIV ON dwd.* TO ROLE statistician;
GRANT DROP_PRIV ON dwd.* TO ROLE statistician;
GRANT CREATE_PRIV ON dws.* TO ROLE statistician;
GRANT DROP_PRIV ON dws.* TO ROLE statistician;
GRANT CREATE_PRIV ON ads.* TO ROLE statistician;
GRANT DROP_PRIV ON ads.* TO ROLE statistician;

-- 授予数据加载权限(如果需要)
GRANT LOAD_PRIV ON ods.* TO ROLE statistician;
GRANT LOAD_PRIV ON dwd.* TO ROLE statistician;
GRANT LOAD_PRIV ON dws.* TO ROLE statistician;
GRANT LOAD_PRIV ON ads.* TO ROLE statistician;
```

### 2. 授予特定用户额外权限

假设ods库中的sensitive_table表只能由stat_user1访问：

```sql
-- 首先撤销角色对该表的权限(如果已授予)
REVOKE SELECT_PRIV ON ods.sensitive_table FROM ROLE statistician;

-- 然后直接授予特定用户权限
GRANT SELECT_PRIV ON ods.sensitive_table TO 'stat_user1';
```

## 四、权限验证

### 1. 查看角色权限

```sql
SHOW GRANTS FOR ROLE statistician;
```

### 2. 查看用户权限

```sql
SHOW GRANTS FOR 'stat_user1';
```

### 3. 查看用户拥有的角色

```sql
SHOW ROLES FOR 'stat_user1';
```

## 五、权限细化控制

### 1. 列级权限控制

Doris 支持列级别的权限控制：

```sql
-- 授予用户对特定表的特定列的查询权限
GRANT SELECT_PRIV ON ods.some_table(col1, col2) TO 'stat_user2';
```

### 2. 资源使用权限

可以限制用户或角色的资源使用：

```sql
-- 设置角色资源限制
SET PROPERTY FOR 'statistician' 'cpu_resource_limit' = '30';
SET PROPERTY FOR 'statistician' 'mem_resource_limit' = '50';
```

### 3. 操作权限控制

```sql
-- 授予ALTER权限(修改表结构)
GRANT ALTER_PRIV ON dwd.* TO ROLE statistician;

-- 授予GRANT权限(允许委派权限)
GRANT GRANT_PRIV ON dwd.* TO ROLE statistician;
```

## 六、权限撤销

### 1. 撤销角色权限

```sql
REVOKE SELECT_PRIV ON ads.* FROM ROLE statistician;
```

### 2. 撤销用户角色

```sql
REVOKE statistician FROM 'stat_user4';
```

### 3. 撤销用户特定权限

```sql
REVOKE DROP_PRIV ON ods.* FROM 'stat_user3';
```

## 七、最佳实践

1. **最小权限原则**：只授予必要的权限
2. **角色分层**：可以创建基础角色和继承角色
   ```sql
   CREATE ROLE junior_statistician;
   CREATE ROLE senior_statistician;
   GRANT junior_statistician TO ROLE senior_statistician;
   ```
3. **定期审计**：定期检查权限分配情况
4. **密码策略**：设置密码复杂度要求
   ```sql
   SET PASSWORD POLICY FOR 'stat_user1' 'min_length' = '8';
   ```
5. **权限模板**：为常见职位创建权限模板角色

## 八、完整示例

```sql
-- 1. 创建角色
CREATE ROLE statistician;
CREATE ROLE senior_statistician;

-- 2. 创建用户
CREATE USER 'stat_junior1' IDENTIFIED BY 'Jpass123!';
CREATE USER 'stat_junior2' IDENTIFIED BY 'Jpass456!';
CREATE USER 'stat_senior1' IDENTIFIED BY 'Spass789!';
CREATE USER 'stat_senior2' IDENTIFIED BY 'Spass012!';

-- 3. 分配角色
GRANT statistician TO 'stat_junior1', 'stat_junior2';
GRANT senior_statistician TO 'stat_senior1', 'stat_senior2';
GRANT statistician TO ROLE senior_statistician;

-- 4. 授予基础权限
GRANT SELECT_PRIV, CREATE_PRIV, DROP_PRIV, LOAD_PRIV 
ON ods.*, dwd.*, dws.*, ads.* 
TO ROLE statistician;

-- 5. 授予高级权限
GRANT ALTER_PRIV, GRANT_PRIV 
ON dwd.*, dws.*, ads.* 
TO ROLE senior_statistician;

-- 6. 特殊表权限处理
REVOKE SELECT_PRIV ON ods.salary_data FROM ROLE statistician;
GRANT SELECT_PRIV ON ods.salary_data TO 'stat_senior1', 'stat_senior2';

-- 7. 设置资源限制
SET PROPERTY FOR 'statistician' 'cpu_resource_limit' = '20';
SET PROPERTY FOR 'senior_statistician' 'cpu_resource_limit' = '40';
```

通过以上步骤，您可以建立一个完善的Doris权限体系，满足不同用户和角色的数据访问需求。



# Doris 动态分区表创建与管理指南

动态分区表是Doris中一种非常实用的功能，特别适合按T+1方式同步数据的场景。下面我将详细介绍如何创建和管理动态分区表。

## 一、动态分区基本概念

动态分区功能允许系统自动管理分区创建和删除，根据规则自动创建新分区并删除旧分区，非常适合时间序列数据。

## 二、创建动态分区表

### 1. 基本创建语法

```sql
CREATE TABLE dynamic_partition_table (
    `dt` DATE NOT NULL COMMENT "分区列",
    `user_id` BIGINT COMMENT "用户ID",
    `event_type` VARCHAR(32) COMMENT "事件类型",
    `value` DECIMAL(18,2) COMMENT "数值"
)
PARTITION BY RANGE(`dt`) ()
DISTRIBUTED BY HASH(`user_id`) BUCKETS 10
PROPERTIES (
    "dynamic_partition.enable" = "true",
    "dynamic_partition.time_unit" = "DAY",
    "dynamic_partition.start" = "-7",
    "dynamic_partition.end" = "3",
    "dynamic_partition.prefix" = "p",
    "dynamic_partition.buckets" = "10",
    "replication_num" = "3"
);
```

### 2. 参数详细说明

| 参数名称                    | 说明                         | 示例值                    |
| --------------------------- | ---------------------------- | ------------------------- |
| dynamic_partition.enable    | 是否开启动态分区             | true/false                |
| dynamic_partition.time_unit | 分区时间单位                 | DAY/WEEK/MONTH            |
| dynamic_partition.start     | 保留的最早分区的偏移量(负数) | -7 (保留最近7天)          |
| dynamic_partition.end       | 预先创建的未来分区的偏移量   | 3 (提前创建未来3天的分区) |
| dynamic_partition.prefix    | 分区名前缀                   | "p"                       |
| dynamic_partition.buckets   | 动态分区的分桶数量           | 10                        |
| replication_num             | 副本数量                     | 3                         |

## 三、动态分区高级配置

### 1. 按周分区配置

```sql
"dynamic_partition.time_unit" = "WEEK",
"dynamic_partition.start" = "-4",  -- 保留最近4周
"dynamic_partition.end" = "2"       -- 提前创建未来2周
```

### 2. 按月分区配置

```sql
"dynamic_partition.time_unit" = "MONTH",
"dynamic_partition.start" = "-6",  -- 保留最近6个月
"dynamic_partition.end" = "1"      -- 提前创建下个月
```

### 3. 自定义分区格式

```sql
"dynamic_partition.time_format" = "%Y%m%d"  -- 分区格式如20230101
```

## 四、动态分区表管理

### 1. 查看动态分区配置

```sql
SHOW DYNAMIC PARTITION TABLES;
```

### 2. 修改动态分区参数

```sql
ALTER TABLE dynamic_partition_table SET (
    "dynamic_partition.enable" = "true",
    "dynamic_partition.time_unit" = "DAY",
    "dynamic_partition.start" = "-30",
    "dynamic_partition.end" = "7"
);
```

### 3. 临时禁用动态分区

```sql
ALTER TABLE dynamic_partition_table SET ("dynamic_partition.enable" = "false");
```

## 五、T+1同步场景实践

### 1. 适合T+1的配置

```sql
CREATE TABLE t_plus_1_sync (
    `sync_date` DATE NOT NULL COMMENT "同步日期",
    `data_id` BIGINT COMMENT "数据ID",
    `content` TEXT COMMENT "内容"
)
PARTITION BY RANGE(`sync_date`) ()
DISTRIBUTED BY HASH(`data_id`) BUCKETS 8
PROPERTIES (
    "dynamic_partition.enable" = "true",
    "dynamic_partition.time_unit" = "DAY",
    "dynamic_partition.start" = "-15",  -- 保留15天历史数据
    "dynamic_partition.end" = "1",     -- 提前创建明天的分区
    "dynamic_partition.prefix" = "p",
    "dynamic_partition.buckets" = "8",
    "replication_num" = "3",
    "storage_medium" = "SSD",          -- 使用SSD存储
    "storage_cooldown_time" = "7 DAY"  -- 7天后转移到HDD
);
```

### 2. 数据加载示例

```sql
-- 加载数据到指定分区(系统会自动创建分区)
INSERT INTO t_plus_1_sync PARTITION(p20230101) 
VALUES ('2023-01-01', 1001, '测试数据');

-- 或者让系统自动路由到正确分区
INSERT INTO t_plus_1_sync 
VALUES ('2023-01-01', 1001, '测试数据');
```

## 六、动态分区与冷热数据分离

结合动态分区和存储策略实现自动冷热数据分离：

```sql
ALTER TABLE t_plus_1_sync SET (
    "storage_medium" = "SSD",
    "storage_cooldown_time" = "7 DAY",
    "dynamic_partition.enable" = "true",
    "dynamic_partition.time_unit" = "DAY",
    "dynamic_partition.start" = "-365",  -- 保留1年数据
    "dynamic_partition.end" = "1",
    "dynamic_partition.hot_partition_num" = "7"  -- 最近7天为热数据
);
```

## 七、注意事项

1. **分区列类型**：动态分区仅支持DATE或DATETIME类型的分区列
2. **时区问题**：动态分区使用FE的时区设置
3. **性能影响**：过多的分区会影响元数据管理性能
4. **修改限制**：已存在的分区不会因参数修改而改变
5. **监控建议**：定期检查分区创建情况

## 八、动态分区监控

```sql
-- 查看分区创建情况
SHOW PARTITIONS FROM dynamic_partition_table;

-- 查看分区数据量
SELECT partition_name, table_rows 
FROM information_schema.partitions 
WHERE table_name = 'dynamic_partition_table';
```

通过以上配置，Doris可以自动管理T+1同步场景下的分区创建和删除，大大简化了分区管理工作。

# Doris 按月分区实现T+1数据同步方案

对于按T+1同步数据但需要按月分区的场景，可以通过以下方式实现：

## 一、基础按月分区表创建

```sql
CREATE TABLE monthly_partition_t1_sync (
    `sync_date` DATE NOT NULL COMMENT "数据同步日期",
    `data_id` BIGINT COMMENT "数据ID",
    `content` TEXT COMMENT "内容",
    `month_col` DATE GENERATED ALWAYS AS (DATE_TRUNC('MONTH', `sync_date`)) COMMENT "生成列-月份"
)
PARTITION BY RANGE(`month_col`) ()
DISTRIBUTED BY HASH(`data_id`) BUCKETS 8
PROPERTIES (
    "dynamic_partition.enable" = "true",
    "dynamic_partition.time_unit" = "MONTH",  -- 按月分区
    "dynamic_partition.start" = "-12",       -- 保留12个月数据
    "dynamic_partition.end" = "1",           -- 提前创建下个月分区
    "dynamic_partition.prefix" = "p",
    "dynamic_partition.buckets" = "8",
    "replication_num" = "3"
);
```

## 二、关键设计说明

### 1. 使用生成列(GENERATED COLUMN)实现月份提取

```sql
`month_col` DATE GENERATED ALWAYS AS (DATE_TRUNC('MONTH', `sync_date`))
```

- 自动从`sync_date`提取月份第一天
- 作为分区键使用
- 存储不占用额外空间

### 2. 动态分区参数配置

| 参数      | 值    | 说明               |
| --------- | ----- | ------------------ |
| time_unit | MONTH | 按月分区           |
| start     | -12   | 保留最近12个月数据 |
| end       | 1     | 提前创建下个月分区 |

## 三、数据加载示例

### 1. 自动路由到对应月份分区

```sql
-- 数据会自动进入p202301分区(1月数据)
INSERT INTO monthly_partition_t1_sync(sync_date, data_id, content)
VALUES ('2023-01-15', 1001, '测试数据1');

-- 数据会自动进入p202302分区(2月数据)
INSERT INTO monthly_partition_t1_sync(sync_date, data_id, content)
VALUES ('2023-02-03', 1002, '测试数据2');
```

### 2. 批量加载T+1数据

```sql
-- 加载前一天的数据到对应月份分区
INSERT INTO monthly_partition_t1_sync
SELECT * FROM source_table 
WHERE sync_date = DATE_SUB(CURRENT_DATE(), 1);
```

## 四、分区维护策略

### 1. 查看当前分区

```sql
SHOW PARTITIONS FROM monthly_partition_t1_sync;
```

结果示例：
```
PartitionName | VisibleVersion | VisibleVersionTime | State | PartitionKey | Range 
--------------|----------------|--------------------|-------|--------------|-------
p202301       | 3              | 2023-01-01 00:00:00 | NORMAL | month_col    | [types: [DATE]; keys: [2023-01-01]; ..types: [DATE]; keys: [2023-02-01]; )
p202302       | 2              | 2023-02-01 00:00:00 | NORMAL | month_col    | [types: [DATE]; keys: [2023-02-01]; ..types: [DATE]; keys: [2023-03-01]; )
```

### 2. 手动添加历史分区(可选)

```sql
ALTER TABLE monthly_partition_t1_sync
ADD PARTITION p202212 VALUES LESS THAN ('2023-01-01');
```

### 3. 修改分区保留策略

```sql
ALTER TABLE monthly_partition_t1_sync SET (
    "dynamic_partition.start" = "-24",  -- 保留24个月数据
    "dynamic_partition.end" = "2"       -- 提前创建未来2个月分区
);
```

## 五、查询优化建议

### 1. 按月份查询

```sql
-- 高效查询(利用分区裁剪)
SELECT * FROM monthly_partition_t1_sync
WHERE month_col = '2023-01-01';

-- 等价于
SELECT * FROM monthly_partition_t1_sync
WHERE sync_date >= '2023-01-01' AND sync_date < '2023-02-01';
```

### 2. 跨月查询

```sql
-- 查询1-3月数据(同样会利用分区裁剪)
SELECT * FROM monthly_partition_t1_sync
WHERE month_col >= '2023-01-01' AND month_col <= '2023-03-01';
```

## 六、特殊场景处理

### 1. 月末数据同步

如果T+1同步的数据正好跨月(如1月31日同步的数据可能在2月1日处理)，系统会自动将数据归入正确的月份分区。

### 2. 历史数据回溯

对于需要重新处理的历史月份数据：

```sql
-- 先检查分区是否存在
SHOW PARTITIONS FROM monthly_partition_t1_sync LIKE 'p202212';

-- 如果不存在则手动添加
ALTER TABLE monthly_partition_t1_sync
ADD PARTITION p202212 VALUES LESS THAN ('2023-01-01');

-- 然后加载数据
INSERT INTO monthly_partition_t1_sync
SELECT * FROM source_table 
WHERE sync_date >= '2022-12-01' AND sync_date < '2023-01-01';
```

## 七、性能考虑

1. **分区数量**：每月一个分区，通常不会导致分区过多问题
2. **分桶数量**：根据数据量调整，建议每个分区8-32个桶
3. **冷热数据**：可结合存储策略设置历史分区自动降级

```sql
ALTER TABLE monthly_partition_t1_sync SET (
    "storage_medium" = "SSD",
    "storage_cooldown_time" = "30 DAY"  -- 30天后转移到HDD
);
```

这种设计既满足了T+1的数据同步需求，又实现了按月分区的数据管理，兼顾了操作便利性和查询性能。
