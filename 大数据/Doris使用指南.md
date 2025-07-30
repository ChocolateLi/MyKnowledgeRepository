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



# 按月分区按天同步策略

## 建表模板

根据您的需求（3节点集群、按月分区、保留3年、每日增量同步、未来扩容），我来为您设计一个优化的Doris ODS层建表模板，并解决您提到的关键问题：

---

### **关键问题分析与建议**
1. **分区策略**  
   **按月动态分区**是合理选择（避免每日分区导致大量小文件）。保留3年需配置动态分区参数，Doris会自动创建/删除分区。

2. **表模型选择**
   - **场景1：无数据更新（如日志）→ 用 `Duplicate` 明细模型**  
     直接存储所有明细，写入效率高。
   - **场景2：需要覆盖更新（如订单状态变更）→ 用 `Unique` 模型**  
     通过主键自动覆盖旧数据（需开启`merge_on_write`以优化查询性能）。

3. **分桶策略（Bucket）**
   - **当前3节点**：建议 **每个分区15~18个分桶**（单节点5~6桶，匹配32核CPU）。
   - **扩容到5节点后**：无需重建表！修改新分区的分桶数为 **15/20/25**（保持5的倍数），历史分区自动平衡。
   - **公式参考**：分桶数 ≈ 节点数 ×（3~5），且不超过100。

4. **数据量优化**
   - 每日数据量小 → 按月分区天然合并小文件。
   - 启用**动态分桶**（Doris 2.0+）或定期`ALTER TABLE`调整未来分区的分桶数。

---

### **建表示例模板（按需二选一）**
#### **选项1：明细表（`Duplicate`模型）**
```sql
CREATE TABLE ods.example_table (
    `event_time` DATETIME NOT NULL COMMENT "事件时间",
    `user_id` BIGINT COMMENT "用户ID",
    `event_type` VARCHAR(32) COMMENT "事件类型",
    -- 其他字段...
    `dt` DATE NOT NULL COMMENT "分区日期"  -- 用于分区
) ENGINE=OLAP
DUPLICATE KEY(`event_time`, `user_id`)  -- 按查询频率选择排序列
COMMENT "ODS层明细表"
PARTITION BY RANGE(`dt`) ()
DISTRIBUTED BY HASH(`user_id`) BUCKETS 18  -- 当前3节点用18桶
PROPERTIES (
    -- 动态分区配置（保留3年+预建未来分区）
    "dynamic_partition.enable" = "true",
    "dynamic_partition.time_unit" = "MONTH",
    "dynamic_partition.start" = "-36",  -- 保留36个月
    "dynamic_partition.end" = "3",      -- 预建未来3个月
    "dynamic_partition.prefix" = "p",
    "dynamic_partition.buckets" = "18", -- 新分区桶数（扩容后可改为25）
    
    -- 存储优化
    "replication_num" = "2",            -- 副本数（生产环境≥2）
    "storage_medium" = "SSD",           -- 存储介质
    "disable_auto_compaction" = "false" -- 启用自动压缩
);
```

#### **选项2：支持更新的表（`Unique`模型）**
```sql
CREATE TABLE ods.example_table (
    `order_id` BIGINT NOT NULL COMMENT "订单ID",
    `status` VARCHAR(32) COMMENT "订单状态",
    `update_time` DATETIME NOT NULL COMMENT "更新时间",
    -- 其他字段...
    `dt` DATE NOT NULL COMMENT "分区日期"
) ENGINE=OLAP
UNIQUE KEY(`order_id`)  -- 主键（唯一性约束）
COMMENT "支持更新的ODS表"
PARTITION BY RANGE(`dt`) ()
DISTRIBUTED BY HASH(`order_id`) BUCKETS 18
PROPERTIES (
    -- 唯一模型优化配置（Doris 2.0+）
    "enable_unique_key_merge_on_write" = "true",  -- 写入时合并提升查询性能
    "store_row_column" = "true",                   -- 支持部分列更新
    
    -- 动态分区（同明细表配置）
    "dynamic_partition.enable" = "true",
    "dynamic_partition.time_unit" = "MONTH",
    "dynamic_partition.start" = "-36",
    "dynamic_partition.end" = "3",
    "dynamic_partition.prefix" = "p",
    "dynamic_partition.buckets" = "18",
    
    "replication_num" = "2"
);
```

---

### **关键配置说明**
1. **动态分区**  
   - 自动按月创建分区（如 `p202501`），删除36个月前旧分区。
   - `dynamic_partition.buckets` 控制新分区桶数，**扩容后修改此参数即可**（历史分区不变）。

2. **分桶策略**
   - **扩容操作示例（5节点时）**：
     ```sql
     ALTER TABLE ods.example_table SET ("dynamic_partition.buckets" = "25");
     ```
   - 未来新分区将按25桶创建，旧数据自动通过副本平衡到新节点。

3. **写入与更新**
   
   - 每日增量用 `INSERT INTO` 或 Stream Load 写入对应月份分区。
   - `Unique` 模型更新：直接按主键写入新记录，旧数据自动标记删除。

---

### **性能调优建议**
1. **分桶监控**  
   查询单个分区大小：
   ```sql
   SHOW PARTITIONS FROM ods.example_table;
   ```
   - 若单分区数据 > **100GB**，考虑调大新分区的分桶数（如18→30）。
   - 若查询延迟高，检查BE节点CPU是否均衡（`SHOW PROC '/backends'`）。

2. **冷热数据分层**（未来扩展）  
   将旧分区转存到廉价存储（如S3）：
   ```sql
   ALTER TABLE ods.example_table 
   MODIFY PARTITION (p202301) SET("storage_medium"="S3");
   ```

3. **数据压缩**  
   每月初对历史分区手动压缩：
   ```sql
   ALTER TABLE ods.example_table COMPACT PARTITION (p202501);
   ```

---

### **总结方案**
- **分区**：按月动态分区（保留36个月 + 预建3个月）  
- **分桶**：当前18桶 → 扩容后25桶（通过`dynamic_partition.buckets`在线调整）  
- **表模型**：无更新用`Duplicate`，有更新用`Unique` + `merge_on_write`  
- **写入**：按天写入对应月份分区，利用月度分区合并小文件  

按此模板建表后，您的集群可高效支持每日增量同步，并在扩容时无缝扩展分桶数，同时动态分区机制自动管理生命周期，彻底避免手动维护成本。



## 删除分区数据

在Doris中，如果需要清空某个分区的数据并重新同步，可以通过以下步骤安全操作。这里提供两种常用方法（**推荐方法1**，更高效且避免误删）：

---

### **方法1：TRUNCATE PARTITION（推荐）**
直接清空指定分区的所有数据（保留分区结构，秒级完成）：
```sql
-- 清空单个月份分区（例如 p202501 对应 2025年1月）
TRUNCATE TABLE ods.example_table PARTITION(p202501);

-- 清空后重新同步数据（通过Stream Load/INSERT等）
-- 例如通过INSERT重新导入：
INSERT INTO ods.example_table PARTITION(p202501)
SELECT * FROM source_table WHERE dt BETWEEN '2025-01-01' AND '2025-01-31';
```
**优点**：  
- 仅删除数据，保留分区元信息，无需重建分区  
- 执行速度极快（直接删除文件，不逐行标记）  
- 事务安全，不影响其他分区  

---

### **方法2：DROP PARTITION + 重建分区**
适用于需要彻底删除分区结构的场景：
```sql
-- 1. 删除分区及数据
ALTER TABLE ods.example_table DROP PARTITION p202501;

-- 2. 重建分区（动态分区表会自动重建，无需手动执行）
-- 如果关闭了动态分区，需手动添加：
ALTER TABLE ods.example_table ADD PARTITION p202501 VALUES [('2025-01-01'), ('2025-02-01'));

-- 3. 重新同步数据...
```
**适用场景**：  

- 需要修改分区范围或属性时  
- 分区数据异常且无法通过TRUNCATE修复  

---

### **关键注意事项**
1. **数据备份**  
   如果分区数据可能被误删，建议先导出：
   ```sql
   -- 导出到HDFS/S3（需提前配置存储）
   EXPORT TABLE ods.example_table PARTITION(p202501)
   TO "hdfs://path/backup/p202501" WITH BROKER "broker_name";
   ```

2. **同步期间锁定（可选）**  
   避免清空过程中有新数据写入：
   ```sql
   -- 锁定分区
   LOCK TABLE ods.example_table PARTITION(p202501) WRITE;
   -- 执行TRUNCATE/重新同步...
   -- 解锁
   UNLOCK TABLES;
   ```

3. **动态分区表的特殊处理**  
   - 如果分区是动态创建的（如 `p202501`），清空后不需要手动重建，新数据写入时会自动创建  
   - 通过 `SHOW DYNAMIC PARTITION TABLES` 确认动态分区状态  

4. **监控执行状态**  
   清空后检查分区大小：
   ```sql
   SHOW PARTITIONS FROM ods.example_table 
   WHERE PartitionName = "p202501";
   ```

---

### **自动化脚本示例（结合清空+重同步）**
```bash
#!/bin/bash
# 清空分区并触发重同步（示例）
PARTITION="p202501"
TABLE="ods.example_table"

# 1. 清空分区
mysql -hDORIS_FE_IP -P9030 -uroot -e \
"TRUNCATE TABLE ${TABLE} PARTITION(${PARTITION});"

# 2. 调用数据同步工具（根据实际工具调整）
/data_sync_tool --table=${TABLE} --partition=${PARTITION} --full-reload
```

---

### **总结建议**
- **99%场景用 `TRUNCATE PARTITION`**：简单高效，优先选择  
- **DROP PARTITION仅用于特殊需求**：如修改分区范围或Schema  
- **动态分区表无需干预重建**：Doris会自动维护分区生命周期  
- **清空前确认分区名**：通过 `SHOW PARTITIONS` 核对名称避免误操作  

按照此方案操作后，您可以安全地清理问题分区并重新同步数据，整个过程对线上查询无影响（除目标分区外）。



## 分桶建议

在Doris中，**分桶数（Bucket）的合理设置对查询性能和资源利用率至关重要**。根据您的集群配置（当前3节点，未来扩至5节点，32核64GB/节点）以及表的数据量差异，我为您总结以下分桶策略和具体建议：

---

### **分桶数决策核心原则**
#### 1. **基础公式参考**
   - **单分区建议大小**：每个分桶的数据量控制在 **100MB~1GB** 之间（理想值为500MB左右）。
   - **计算公式**：  
     ```
     分桶数 ≈ 单分区数据量 / 500MB
     ```
     - 例如：某表每月数据量为30GB → 分桶数 = 30GB / 0.5GB ≈ **60桶**。

#### 2. **与集群规模联动**
   - **最小分桶数** ≥ 节点数（保证数据均匀分布）。
   - **最大分桶数** ≤ 节点数 × 100（避免过多小文件）。
   - **推荐范围**：
     | 数据规模               | 当前3节点分桶数 | 未来5节点分桶数 |
     | ---------------------- | --------------- | --------------- |
     | **小表**（<10GB/月）   | 3~9             | 5~15            |
     | **中表**（10~50GB/月） | 12~18           | 15~30           |
     | **大表**（>50GB/月）   | 18~36           | 25~50           |

#### 3. **特殊调整因素**
   - **高频查询表**：适当增加分桶数（提升并行度）。
   - **JOIN频繁的表**：确保分桶数相同或成倍数关系（避免数据倾斜）。
   - **SSD存储**：可支持更多分桶（随机读写性能好）。

---

### **具体分桶建议（按表数据量分类）**
#### **场景1：小表（<10GB/月）**
- **典型表**：维度表、配置表、低频日志表。
- **分桶策略**：
  ```sql
  DISTRIBUTED BY HASH(key) BUCKETS 6  -- 当前3节点
  -- 未来5节点时新分区改为：
  "dynamic_partition.buckets" = "10"
  ```
- **理由**：
  - 过少分桶（如3个）可能导致查询并行度不足，6~10个桶可平衡小文件与并行效率。

#### **场景2：中表（10~50GB/月）**
- **典型表**：业务订单表、用户行为表。
- **分桶策略**：
  ```sql
  DISTRIBUTED BY HASH(user_id) BUCKETS 18  -- 当前3节点（单节点6桶）
  -- 未来5节点调整：
  "dynamic_partition.buckets" = "25"
  ```
- **理由**：
  - 单桶数据量 ≈ 0.8GB（30GB/18桶），符合最佳范围。

#### **场景3：大表（>50GB/月）**
- **典型表**：埋点日志、交易流水。
- **分桶策略**：
  ```sql
  DISTRIBUTED BY HASH(device_id) BUCKETS 36  -- 当前3节点
  -- 未来5节点调整：
  "dynamic_partition.buckets" = "50"
  ```
- **理由**：
  - 单桶数据量 ≈ 1.4GB（50GB/36桶），SSD存储可接受略大分桶。

---

### **动态调整与扩容方案**
#### 1. **在线修改分桶数（仅影响新分区）**
   ```sql
   -- 修改动态分区表的未来分桶数（扩容到5节点后执行）
   ALTER TABLE ods.large_table SET ("dynamic_partition.buckets" = "50");
   ```
   - **历史分区**：继续使用原分桶数，通过副本自动平衡到新节点。
   - **新分区**：按新分桶数创建。

#### 2. **手动平衡历史分区（可选）**
   如需重新分布旧数据：
   ```sql
   -- 增加临时副本并重新分桶
   ALTER TABLE ods.large_table MODIFY PARTITION p202501 
   SET ("replication_num" = "3", "bucket_num" = "50");
   ```

#### 3. **分桶数监控SQL**
   ```sql
   -- 检查分桶数据量分布
   SELECT 
     PARTITION_NAME,
     SUM(DATA_SIZE)/1024/1024 AS "Size_MB",
     COUNT(DISTINCT BUCKET_ID) AS "Bucket_Count",
     SUM(DATA_SIZE)/COUNT(DISTINCT BUCKET_ID)/1024/1024 AS "MB_per_Bucket"
   FROM INFORMATION_SCHEMA.PARTITIONS 
   WHERE TABLE_NAME = 'your_table'
   GROUP BY PARTITION_NAME;
   ```
   - **健康指标**：`MB_per_Bucket` 应在100~1024之间。

---

### **建表示例模板（差异化分桶）**
#### **小表模板（6桶）**
```sql
CREATE TABLE ods.small_table (
    `id` BIGINT,
    `config_value` VARCHAR(256)
) ENGINE=OLAP
DUPLICATE KEY(`id`)
DISTRIBUTED BY HASH(`id`) BUCKETS 6  -- 小表固定分桶
PROPERTIES (
    "replication_num" = "2",
    "dynamic_partition.enable" = "false"  -- 小表通常无需分区
);
```

#### **大表模板（动态分桶）**
```sql
CREATE TABLE ods.large_table (
    `log_time` DATETIME,
    `user_id` BIGINT,
    `event` TEXT
) ENGINE=OLAP
DUPLICATE KEY(`log_time`, `user_id`)
PARTITION BY RANGE(`log_time`) ()
DISTRIBUTED BY HASH(`user_id`) BUCKETS 36  -- 初始分桶
PROPERTIES (
    "dynamic_partition.enable" = "true",
    "dynamic_partition.buckets" = "36",    -- 与初始分桶一致
    "dynamic_partition.start" = "-36",
    "dynamic_partition.end" = "3"
);
```

---

### **最终建议**
1. **先按数据量分类**：对现有表按月度数据量分为小/中/大三档。
2. **初始分桶参考**：
   - 小表：6桶  
   - 中表：18桶  
   - 大表：36桶  
3. **扩容后调整**：仅修改`dynamic_partition.buckets`为节点数的倍数（如5→25/50）。
4. **定期检查**：每月运行分桶监控SQL，确保单桶数据量在理想范围内。

通过此策略，您可以在不同数据规模的表上实现**查询性能**与**存储效率**的最佳平衡，同时为集群扩容预留灵活性。



## 小表无分区

您的问题非常关键！对于小表（数据量小但需要按天/月增量同步）的场景，**无需分区 ≠ 无法按时间管理数据**。以下是针对小表数据管理的完整解决方案，既保持分桶简单性，又能支持时间维度的数据订正：

---

### **方案1：增加时间字段 + 条件删除（推荐）**
**适用场景**：小表（<10GB/月），但有按时间订正需求。  
**核心思路**：保留单分区，但通过时间字段逻辑控制数据范围。

#### **建表示例**
```sql
CREATE TABLE ods.small_table (
    `id` BIGINT,
    `data` VARCHAR(256),
    `dt` DATE NOT NULL COMMENT '业务日期'  -- 关键时间字段
) ENGINE=OLAP
DUPLICATE KEY(`id`, `dt`)  -- 将dt放入排序列
DISTRIBUTED BY HASH(`id`) BUCKETS 6
PROPERTIES (
    "replication_num" = "2"
);
```

#### **数据订正操作**
```sql
-- 1. 删除指定时间范围数据（如重刷2025年1月数据）
DELETE FROM ods.small_table WHERE dt BETWEEN '2025-01-01' AND '2025-01-31';

-- 2. 重新同步该时间段数据
INSERT INTO ods.small_table
SELECT * FROM source_table WHERE dt BETWEEN '2025-01-01' AND '2025-01-31';
```

**优势**：  
- 无需分区，保持小表简单性  
- 精确控制时间范围数据删除  
- 删除操作是事务性的（不影响并发查询）  

**注意事项**：  
- Doris的`DELETE`是标记删除，后续需自动压缩（`disable_auto_compaction=false`默认开启）  
- 如果单次删除数据量极大（>100万行），建议分批执行  

---

### **方案2：动态分区小表（灵活版）**
**适用场景**：小表但可能有某个月数据突增。  
**核心思路**：仍使用分区，但设置更大的分桶下限。

#### **建表示例**
```sql
CREATE TABLE ods.small_table (
    `id` BIGINT,
    `data` VARCHAR(256),
    `dt` DATE NOT NULL
) ENGINE=OLAP
DUPLICATE KEY(`id`, `dt`)
PARTITION BY RANGE(`dt`) ()
DISTRIBUTED BY HASH(`id`) BUCKETS 3  -- 最小分桶数
PROPERTIES (
    "dynamic_partition.enable" = "true",
    "dynamic_partition.time_unit" = "MONTH",
    "dynamic_partition.start" = "-12",  -- 保留1年
    "dynamic_partition.end" = "1",
    "dynamic_partition.buckets" = "3",  -- 固定小分桶
    "replication_num" = "2"
);
```

#### **数据订正操作**
```sql
-- 清空某月分区（如p202501）
TRUNCATE TABLE ods.small_table PARTITION(p202501);

-- 重新同步
INSERT INTO ods.small_table PARTITION(p202501)
SELECT * FROM source_table WHERE dt BETWEEN '2025-01-01' AND '2025-01-31';
```

**优势**：  
- 保留分区灵活性，应对可能的月度数据增长  
- 分桶数极少（3个），避免小文件问题  

---

### **方案对比**
| 方案            | 是否分区 | 适合表大小 | 订正操作复杂度     | 存储效率 |
| --------------- | -------- | ---------- | ------------------ | -------- |
| 时间字段+DELETE | 否       | <1GB/月    | 中等（需写DELETE） | 高       |
| 动态分区小表    | 是       | 1~10GB/月  | 低（TRUNCATE）     | 中       |

---

### **终极选择建议**
1. **稳定小表（每月数据量差异<2倍）** → 选**方案1（时间字段+DELETE）**  
   - 例如：配置表、低频维度表  
   - 优点：完全避免分区开销，建表最简单  

2. **波动小表（可能某月数据激增）** → 选**方案2（动态分区小表）**  
   - 例如：按月的统计结果表  
   - 优点：突发数据量时仍可高效管理  

3. **特殊技巧**：  
   ```sql
   -- 建表后通过时间字段预聚合（优化查询）
   ALTER TABLE ods.small_table ADD ROLLUP rollup_dt_sum(dt, COUNT(*));
   ```

---

### **操作安全提示**
1. **先验证再删除**：  
   ```sql
   -- 先查询确认数据范围
   SELECT COUNT(*) FROM ods.small_table 
   WHERE dt BETWEEN '2025-01-01' AND '2025-01-31';
   ```

2. **备份重要数据**：  
   ```sql
   -- 导出待删除数据
   SELECT * INTO OUTFILE 'file:///tmp/backup_202501.csv'
   FROM ods.small_table 
   WHERE dt BETWEEN '2025-01-01' AND '2025-01-31';
   ```

3. **低峰期操作**：DELETE/TRUNCATE 会短暂占用集群资源。

---

通过这种设计，您的小表既能享受**无分区的简洁性**，又能获得**时间维度数据订正**的能力，完美平衡运维复杂度和灵活性。

## 分区是否正确

在Doris中检查分区数据是否正确分布，可以通过以下步骤验证您的按月动态分区表 `test.test_student_day` 的数据归属情况。以下是完整操作指南：

---

### **第一步：修改表结构添加分区（如果尚未分区）**
您的当前建表语句没有分区，需要先**添加按月分区**（假设按`createtime`字段分区）：
```sql
ALTER TABLE test.test_student_day
ADD PARTITION BY RANGE(createtime) (
    PARTITION p202301 VALUES LESS THAN ('2023-02-01'),
    PARTITION p202302 VALUES LESS THAN ('2023-03-01')
    -- 后续分区可通过动态分区自动生成
);

-- 启用动态分区（按月自动创建）
ALTER TABLE test.test_student_day SET (
    "dynamic_partition.enable" = "true",
    "dynamic_partition.time_unit" = "MONTH",
    "dynamic_partition.start" = "-12",
    "dynamic_partition.end" = "3",
    "dynamic_partition.prefix" = "p"
);
```

---

### **第二步：验证数据进入正确分区的4种方法**

#### **方法1：直接查询分区数据（最直观）**
```sql
-- 查询特定分区数据（例如p202301分区）
SELECT * FROM test.test_student_day PARTITION(p202301) 
WHERE createtime BETWEEN '2023-01-01' AND '2023-01-31' LIMIT 10;

-- 检查各分区数据量
SELECT 
    PARTITION_NAME,
    COUNT(*) AS row_count,
    MIN(createtime) AS min_time,
    MAX(createtime) AS max_time
FROM test.test_student_day
GROUP BY PARTITION_NAME
ORDER BY PARTITION_NAME;
```

#### **方法2：通过`EXPLAIN`查看查询计划**
```sql
-- 查看SQL实际扫描的分区
EXPLAIN SELECT * FROM test.test_student_day 
WHERE createtime = '2025-7-21';
```
输出中会显示：
```
PARTITION PREDICATES: `createtime` < '2023-02-01', `createtime` >= '2023-01-01'
```

#### **方法3：检查分区元信息**
```sql
-- 查看所有分区信息
SHOW PARTITIONS FROM test.test_student_day;

-- 结果示例：
| PartitionName | VisibleVersion | VisibleVersionTime | VisibleVersionHash | State | Buckets | ReplicationNum | LastConsistencyCheckTime | DataSize | IsInMemory | RowCount |
|---------------|----------------|--------------------|--------------------|-------|---------|----------------|--------------------------|----------|------------|----------|
| p202301       | 1              | 2023-01-01 00:00:00| 123456789          | NORMAL| 3       | 1              | NULL                     | 2.5 GB   | false      | 1000000  |
```

#### **方法4：强制查询非预期分区（验证数据隔离）**
```sql
-- 查询本应不在该分区的数据（应返回空）
SELECT * FROM test.test_student_day PARTITION(p202301) 
WHERE createtime = '2023-02-01';
```

---

### **第三步：数据同步后的验证流程**
1. **插入测试数据**：
   ```sql
   INSERT INTO test.test_student_day VALUES
   ('001', '张三', 18, '一中', 'A班', '2023-01-15 10:00:00', NOW()),
   ('002', '李四', 17, '二中', 'B班', '2023-02-20 11:00:00', NOW());
   ```

2. **验证分布**：
   ```sql
   -- 检查数据是否进入正确分区
   SELECT 
       id, name, 
       createtime, 
       PARTITION_NAME 
   FROM test.test_student_day
   LATERAL VIEW(INTERPRET_PARTITION_ID(PARTITION_ID())) t AS PARTITION_NAME;
   ```

---

### **关键注意事项**
1. **分区字段选择**：
   - 确保分区字段（如`createtime`）在插入数据时**非NULL**
   - 动态分区依赖该字段的日期格式必须规范（`YYYY-MM-DD HH:MM:SS`）

2. **常见问题排查**：
   - **问题**：数据未进入预期分区  
     **解决**：检查`createtime`字段值是否符合分区范围（如`p202301`只能包含2023-01-01至2023-01-31的数据）
   
   - **问题**：动态分区未自动创建  
     **解决**：确认`dynamic_partition.enable`为true，且时间字段值在`start/end`范围内

3. **性能建议**：
   ```sql
   -- 为分区字段创建索引（加速查询）
   ALTER TABLE test.test_student_day ADD INDEX idx_createtime(createtime) USING BITMAP;
   ```

---

### **完整建表模板（含分区）**
```sql
CREATE TABLE test.test_student_day (
    id VARCHAR(50),
    name VARCHAR(50),
    age INT,
    school VARCHAR(50),
    class VARCHAR(50),
    createtime DATETIME NOT NULL COMMENT "分区字段",
    updatetime DATETIME
)
ENGINE=OLAP
DUPLICATE KEY(id, createtime)
PARTITION BY RANGE(createtime) ()
DISTRIBUTED BY HASH(id) BUCKETS 3
PROPERTIES (
    "replication_num" = "1",
    "dynamic_partition.enable" = "true",
    "dynamic_partition.time_unit" = "MONTH",
    "dynamic_partition.start" = "-12",
    "dynamic_partition.end" = "3",
    "dynamic_partition.prefix" = "p"
);
```

通过以上方法，您可以清晰掌握数据在分区中的分布情况，确保ETL过程的数据准确性。

## 时间戳如何做分区

在Doris中，您完全可以将源数据的**时间戳字段**（如`updatetime`）作为按月分区的依据，即使Doris表的分区字段类型为`DATE`。以下是具体解决方案和注意事项：

---

### **方案1：同步时自动转换（推荐）**
在数据同步工具（如Flink/Spark Job）或INSERT语句中，**将时间戳转为DATE类型**再写入Doris：
```sql
-- 建表语句（分区字段用DATE类型）
CREATE TABLE test.test_student_day (
    id VARCHAR(50),
    updatetime DATE COMMENT "分区字段"  -- 注意这里是DATE类型
) ENGINE=OLAP
PARTITION BY RANGE(updatetime) ()
DISTRIBUTED BY HASH(id) BUCKETS 3
PROPERTIES (
    "dynamic_partition.enable" = "true",
    "dynamic_partition.time_unit" = "MONTH"
);
```

#### **数据同步示例**
```sql
-- 方式1：通过INSERT转换（适合小批量数据）
INSERT INTO test.test_student_day 
SELECT 
    id, 
    FROM_UNIXTIME(updatetime, '%Y-%m-%d') AS updatetime  -- 时间戳转DATE
FROM source_table;

-- 方式2：在Stream Load的JSONPath中转换
curl -X POST \
  -H "format: json" \
  -H "strip_outer_array: true" \
  -H "jsonpaths: [\"$.id\", \"$.updatetime\"]" \
  -H "columns: id, updatetime=FROM_UNIXTIME(updatetime, '%Y-%m-%d')" \
  http://FE_HOST:8030/api/DB/TABLE/_stream_load
```

---

### **方案2：建表时保留原始时间戳（兼容性更强）**
如果希望保留原始时间戳字段，可**新增一个DATE类型的分区字段**：
```sql
CREATE TABLE test.test_student_day (
    id VARCHAR(50),
    updatetime_unix BIGINT COMMENT "原始时间戳",
    updatetime_date DATE COMMENT "分区字段（自动生成）"  -- 用于分区
) ENGINE=OLAP
PARTITION BY RANGE(updatetime_date) ()
DISTRIBUTED BY HASH(id) BUCKETS 3
PROPERTIES (
    "dynamic_partition.enable" = "true",
    "dynamic_partition.time_unit" = "MONTH"
);

-- 同步时自动填充两个字段
INSERT INTO test.test_student_day
SELECT 
    id,
    updatetime AS updatetime_unix,
    FROM_UNIXTIME(updatetime, '%Y-%m-%d') AS updatetime_date
FROM source_table;
```

---

### **关键注意事项**
1. **时间戳格式必须合法**  
   - 确保源数据的时间戳是**秒级或毫秒级**（毫秒级需除以1000）：
     ```sql
     -- 处理毫秒时间戳
     FROM_UNIXTIME(updatetime/1000, '%Y-%m-%d')
     ```

2. **动态分区依赖字段非NULL**  
   - 分区字段（如`updatetime_date`）不能为NULL，否则数据会进入默认分区

3. **时区问题**  
   - 如果源数据时间戳是UTC时间，需显式指定时区转换：
     ```sql
     FROM_UNIXTIME(updatetime, '%Y-%m-%d', 'Asia/Shanghai')
     ```

4. **验证分区正确性**  
   ```sql
   -- 检查分区边界
   SHOW PARTITIONS FROM test.test_student_day;
   
   -- 确认数据分布
   SELECT 
       PARTITION_NAME, 
       COUNT(*) 
   FROM test.test_student_day
   GROUP BY PARTITION_NAME;
   ```

---

### **错误场景模拟与解决**
#### **问题现象**  
同步失败，报错：`Invalid date format for partition column`

#### **原因排查**  
1. 时间戳值超出范围（如负数或未来10年后的时间）
2. 未正确处理毫秒级时间戳

#### **解决方案**  
```sql
-- 修复SQL示例（过滤异常时间戳）
INSERT INTO test.test_student_day
SELECT 
    id,
    FROM_UNIXTIME(
        CASE 
            WHEN updatetime BETWEEN 0 AND 2147483647 THEN updatetime  -- 秒级
            WHEN updatetime BETWEEN 0 AND 2147483647000 THEN updatetime/1000  -- 毫秒级
            ELSE UNIX_TIMESTAMP()  -- 异常值替换为当前时间
        END,
        '%Y-%m-%d'
    ) AS updatetime_date
FROM source_table;
```

---

### **最终建议**
1. **首选方案1**：在数据同步层做类型转换，保持Doris表结构简洁  
2. **处理极端情况**：添加数据清洗逻辑，避免非法时间戳导致同步中断  
3. **验证工具**：使用`SELECT FROM_UNIXTIME(你的时间戳字段)`抽样检查转换结果  

通过这种方式，您可以安全地将时间戳字段用于Doris的按月分区，且完全不会导致同步失败。



## 修改表结构

在Doris中处理源表结构变更（如字段改名、新增列）时，**不需要删除重建ODS表**，可以通过以下方案实现无损平滑升级：

---

### **方案1：Schema Evolution（推荐）**
适用于**字段新增/类型修改**，Doris 1.2+版本原生支持：
```sql
-- 1. 查看当前表结构
DESC test.test_student_day;

-- 2. 新增列（源表新增gender字段）
ALTER TABLE test.test_student_day ADD COLUMN gender VARCHAR(10) DEFAULT NULL COMMENT '性别';

-- 3. 修改列名（源表class改名为class_name）
ALTER TABLE test.test_student_day RENAME COLUMN class TO class_name;

-- 4. 修改列类型（如age从INT改为BIGINT）
ALTER TABLE test.test_student_day MODIFY COLUMN age BIGINT;
```

**优势**：
- 无需停机，在线执行
- 历史数据保留，新增字段对已有数据自动填充`NULL`或默认值

---

### **方案2：动态列（Dynamic Column）**
适用于**频繁变更的JSON半结构化数据**：
```sql
-- 建表时启用动态列
CREATE TABLE test.test_student_day (
    id VARCHAR(50),
    name VARCHAR(50),
    `__dynamic_columns` JSON COMMENT '动态列'
) ENGINE=OLAP
DUPLICATE KEY(id)
PROPERTIES (
    "enable_dynamic_schema" = "true"
);

-- 插入数据时自动扩展列
INSERT INTO test.test_student_day 
VALUES ('001', '张三', '{"age":18, "school":"一中"}');
```

**查询动态列**：
```sql
SELECT 
    id, 
    name,
    __dynamic_columns->'$.age' AS age,
    __dynamic_columns->'$.school' AS school
FROM test.test_student_day;
```

---

### **方案3：版本化表设计（CDC场景）**
适用于**严格跟踪历史变更**的数据仓库：
```sql
-- 添加元数据字段
ALTER TABLE test.test_student_day ADD COLUMN (
    `__src_op` VARCHAR(10) COMMENT '操作类型(INSERT/UPDATE/DELETE)',
    `__src_ts` DATETIME COMMENT '源系统变更时间'
);

-- 同步工具处理逻辑示例：
-- 1. 检测源表ALTER操作
-- 2. 在Doris执行相同ADD COLUMN
-- 3. 同步数据时填充__src_op和__src_ts
```

---

### **关键操作注意事项**
1. **字段删除的特殊处理**  
   Doris不支持直接删除非Key列，需通过替换实现：
   ```sql
   -- 先创建新表（不含要删除的列）
   CREATE TABLE test.test_student_day_new LIKE test.test_student_day;
   ALTER TABLE test.test_student_day_new DROP COLUMN school;
   
   -- 数据迁移
   INSERT INTO test.test_student_day_new 
   SELECT id,name,age,class,createtime,updatetime 
   FROM test.test_student_day;
   
   -- 原子替换
   RENAME TABLE test.test_student_day TO test.test_student_day_old,
   test.test_student_day_new TO test.test_student_day;
   ```

2. **同步工具集成**  
   在Flink/Spark作业中配置自动检测DDL变更：
   ```python
   # Flink CDC示例
   source = MySQLSource.builder()
       .hostname("mysql_host")
       .database("test")
       .table("student_day")
       .deserdeber(JsonDebeziumDeserializationSchema())
       .includeSchemaChanges(true)  # 关键配置
       .build()
   ```

3. **数据一致性验证**  
   ```sql
   -- 检查行数差异
   SELECT 
       (SELECT COUNT(*) FROM source_db.student_day) AS source_count,
       (SELECT COUNT(*) FROM test.test_student_day) AS doris_count;
   
   -- 抽样比对数据
   SELECT * FROM test.test_student_day 
   WHERE id IN ('001','002') 
   EXCEPT 
   SELECT * FROM source_db.student_day 
   WHERE id IN ('001','002');
   ```

---

### **不同场景的最佳实践**
| 变更类型         | 推荐方案                     | 执行示例                     |
| ---------------- | ---------------------------- | ---------------------------- |
| **新增字段**     | Schema Evolution             | `ALTER TABLE ADD COLUMN`     |
| **字段改名**     | 先新增列再迁移数据           | 见字段删除处理               |
| **字段类型修改** | 小范围用MODIFY，大范围建新表 | `ALTER TABLE MODIFY COLUMN`  |
| **删除字段**     | 创建不含该列的新表           | 见字段删除处理               |
| **不确定变更**   | 动态列(JSON)                 | `enable_dynamic_schema=true` |

---

### **自动化方案建议**
1. **监听源库DDL**  
   使用MySQL的`binlog`或PostgreSQL的`logical decoding`捕获ALTER事件，触发Doris表变更

2. **CI/CD流程**  
   ```mermaid
   graph LR
   A[源表ALTER] --> B(自动生成Doris ALTER SQL)
   B --> C{安全审核}
   C -->|通过| D[执行变更]
   C -->|拒绝| E[人工干预]
   ```

3. **版本回滚准备**  
   ```sql
   -- 每次变更前备份schema
   SHOW CREATE TABLE test.test_student_day 
   INTO OUTFILE '/backup/schema_20240501.sql';
   ```

---

通过以上方法，您可以实现ODS层表的**无缝结构变更**，完全避免"删表重建"的暴力操作，确保数据同步流程的持续稳定运行。
