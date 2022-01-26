# JD_Internship_Knowledge_record

## 数据仓库

### 数仓模型设计流程

1. 需求申请
2. 需求调研
3. 数据探查
4. 模型设计：概念模型、逻辑模型、物理模型
5. mapping设计
6. 脚本编写
7. 任务测试和模型验证
8. 模型完善
9. 模型上线
10. 模型运营

### 数据仓库位置

![数据仓库位置](/Users/chenli75/Desktop/MyFile/github/MyKnowledgeRepository/internship_picture/数据仓库位置.png)

### 数据仓库分层架构

数据流向是：业务系统日志数据 -> BDM -> FDM -> GDM -> ADM -> (DIM层、TMP层) -> APP层 -> BI 

为什么数据仓库要分层设计？利于回溯查询。比如在上层应用中缺少某个维度的数据，就可以回溯到最原始的数据层进行提取。

![分层架构](/Users/chenli75/Desktop/MyFile/github/MyKnowledgeRepository/internship_picture/分层架构.png)



**分层说明**

BDM（缓冲数据层）：源业务系统的快照、会保留一段时间。一个BDM对应业务系统的一个表或者一个日志文件。处理成 dt(日期) dh(小时) data(json格式数据)。以dt和ht为分区

FDM（基础数据层）：从BDM层中提取出基础的字段信息，以dt为分区。

GDM（通用数据层）：更详细的字段信息，扩展FDM层的数据维度，是一个宽表。其中每一列代表一个纬度

ADM（聚合数据层）：对GDM中的表做聚合，即进行join，聚合后的每一列数据代表一个指标，或者度量

![分层说明](/Users/chenli75/Desktop/MyFile/github/MyKnowledgeRepository/internship_picture/分层说明.png)



### 数据仓库设计流程

![数据仓库设计流程](/Users/chenli75/Desktop/MyFile/github/MyKnowledgeRepository/internship_picture/数据仓库设计流程.png)



### 数据仓库命名规范

| 分层 | 规范                                                         | 事例                      | 备注                                        |
| ---- | ------------------------------------------------------------ | ------------------------- | ------------------------------------------- |
| BDM  | BDM _ 源库名称 _ 源表名称                                    | bdm_jporders_orderoperate |                                             |
| FDM  | FDM _ 源库名称 _ 源表名称 _ 加载策略                         | fdm_pek_order_chain       | 加载策略：拉链抽取"_chain",增量抽取" _ 无 " |
| GDM  | GDM _ 主题前缀 _ 主题 _ (da)                                 | gdm_m03_slef_item_sku_da  | 包含"_da"后缀则为全量表，否则为增量表       |
| ADM  | 公共明细模型：ADM _ 业务域 _ d _ 主题编码 _ 主题域 _  模型主体_ 后缀<br />公共汇总模型：ADM _ 业务域 _ s _ 主题编码 _ 主题域 _  模型主体_ 后缀 |                           |                                             |
| DIM  | DIM _ 业务含义的表名                                         |                           |                                             |
| TMP  | TMP _ 业务含义的表名                                         |                           |                                             |



### 数据仓库设计-FDM层

设计原则：保留历史数据的同时，尽可能少的占用存储空间快速、高效的获取历史上任意一天的快照数据 

设计方法：通过拉链表实现，经比较记录数据的全生命周期，可快速还原任意天的历史快照，同时极大的节省空间，其中模型带“_chain”为拉链表（归档）、不带为流水表



**概念解析**

拉链表：所谓拉链就是记录历史，记录一个事物从开始一直到当前状态的所有变化的信息。链表中有开始时间（start_time）和结束时间（end_time）两个字段，同时有dt和dp两个分区字段，end_time也可做分区；

 start_time:数据开始的时间

end_time:数据结束的时间

dp:一般有ACTIVE（线上）和EXPIRED（过期）两个分区。ACTIVE表示数据当前在线上使用，所以其end_time为4712-12-31（系统能处理的最大时间，表示一个达不到的无限向后延伸的时间，意味着该数据在线上永久有效）；EXPIRED表示数据过期，已变更，为历史状态，其end_time是数据变更时具体的时间。

dt:数据所在的时间分区，记录数据从ACTIVE转移到EXPIRED的日期，即数据发生变更的时间，大部分与end_time一致。拉链表不建议使用dt分区， 标黄为变更前数据，标红为变更后数据

标黄为变更前数据，标红为变更后数据

![拉链表](/Users/chenli75/Desktop/MyFile/github/MyKnowledgeRepository/internship_picture/拉链表.png)

查询问题：如还原2020-06-02的历史快照，使用end_time> ‘2020-06-02’ and start_time<= ‘2020-06-02’ 查询，end_time过滤2020-06-02之前的旧数据，start_time过滤2020-06-02之后的新数据。 

sysdate(-1)表示昨天的日期

拉链表查询方式：

![拉链表查询方式](/Users/chenli75/Desktop/MyFile/github/MyKnowledgeRepository/internship_picture/拉链表查询方式.png)



拉链表场景：

1. 表中部分字段会被update，例如订单的状态，从下单到待付款到已付款
2. 需要查看某一个时间点或者时间段的历史快照信息，例如查看某一个订单在某一个时间点是什么状态
3. 变化比例或者频率不是很大





流水表：流水表存放的是一个用户的变更记录，比如在一个张流水表中，一天的数据，会存放一个用户的每条修改记录，但是在拉链表中只有一条记录

![流水表](/Users/chenli75/Desktop/MyFile/github/MyKnowledgeRepository/internship_picture/流水表示意图.png)



### 数据仓库设计-GDM层

GDM层的表数据是一个宽表，就是字段比较多的数据库表。宽表已经不符合三范式的规范，存在大量数据冗余，好处就是查询性能提高了。窄表是按照数据库三范式，减少了数据冗余，但缺点就是修改一个数据可能需要修改多张表。 

**设计原则**

数据通用性：模型设计的时候，尽可能的将各业务部常用、通用行较高的字段进行统一处理

主题建模：按照主题进行建模

统计周期：模型可区分全量表、增量表，其中模型带“_da”为全量表，否则为增量表规范易懂：命名规范 设计方法：业务分析：分析各个部门的需求

 **概念解析**

增量表：记录更新周期内的数据，即在原表的基础上新增本周期内产生的数据。比如按天更新的流量表，每次更新只新增一天内产生的新数据。注意，每次新产生的数据是以最新分区增加到表中，原先数据依然存在表中。

![增量表](/Users/chenli75/Desktop/MyFile/github/MyKnowledgeRepository/internship_picture/增量表.png)

![增量表2](/Users/chenli75/Desktop/MyFile/github/MyKnowledgeRepository/internship_picture/增量表2.png)

全量表：每次记录都会记录全量数据。包括原全量数据和本次新增数据。注意，全量表中每个分区内都是截至分区时间的全量数据，原先分区的数据依然存在于表中，只是每次更新会在最新分区内再更新一遍全量数据。

![全量表](/Users/chenli75/Desktop/MyFile/github/MyKnowledgeRepository/internship_picture/全量表.png)

![全量表2](/Users/chenli75/Desktop/MyFile/github/MyKnowledgeRepository/internship_picture/全量表2.png)



### 用户主题模型设计

![用户主题模型设计](/Users/chenli75/Desktop/MyFile/github/MyKnowledgeRepository/internship_picture/用户主题模型设计.png)



## 指标体系

概念：用**来量化事物的一个工具，帮助我们去将一些抽象的事件得出一个轮廓上的描述**。例如我们可以从指标上判断一个产品的好坏，用户粘性等等，例如我们通过日活能去判断出我们整个产品的用户量，**从而能反应出我们这个产品的一个健康程度，也就是否处于增长过程中**。指标，实际上就是一种度量。大到用于监控和评估商业进程的状态，小到衡量某个功能模块的情况，或者是自己的活动效果。指标体系的构建，是为了让业务目标**可度量、可描述、可拆解**，所以我们的指标是基于业务目标抽取和评估的数据维度，这些数据为度可以用来帮助达到业务目标。



## Hive优化

### 优化的根本思想：

1、尽早过滤数据，减少每个阶段的数据量。数据量少了，减少了网络传输

2、减少job数。所谓的job就是你提交一个复杂的SQL后，hive会把复杂的sql转换成若干个mapreduce的job任务，每个job对应你sql中的部分逻辑，每个job之间任务是独立的。任务是独立的，但数据是依赖的。就是下一个job依赖上一个job的数据文件，上一个job任务执行完会落盘到hdfs上，你job越多，落盘次数也越多，磁盘IO也越多。

3、解决数据倾斜

### 优化方法

1、列裁剪。hive在读取数据的时候，只读取查询中所需要的列，而忽略其他列。

2、分区裁剪。在查询过程中减少不必要的分区。如果所需要的数据在某一个分区，就指定那个分区进行查询。那怎么知道查询的数据在哪个分区？通过Explain dependency获取。如Explain dependency select count(*) from table where dt>="2014-06-01"

3、利用hive的优化机制减少job数。不论是外关联outer join还是内关联inner join，如果join的key相同，不管有多少个表，都会合并为一个MapReduce任务。

```sql
1个job
select a.val,b.val,c.val 
from a join b on (a.key=b.key1) 
join c on (c.key=b.key1)

2个job
select a.val,b.val,c.val 
from a join b on (a.key=b.key1) 
join c on (c.key=b.key2)

```

4、善用muti-insert、unoin all，不同表的union all相当于多次输入，同一个表的union all，相当于map一次输出多条

```sql
1、相当于查询了表a两次
insert overwriter table tmp1
  select ... from a where 条件1
insert overwriter table tmp2
  select ... from a where 条件2
  
2、相当于查询了表a一次
from a
  insert overwriter table tmp1
    select ...  where 条件1
  insert overwriter table tmp2
    select ...  where 条件2

```

5、避免笛卡尔积。在两个表进行关联时，写上on关联条件

6、在join前过滤掉不需要的数据。

```sql
1、优化前
select a.val,b.val from a left outer join b on (a.key=b.key)
where a.ds='2009-07-07'and b.ds='2009-07-07'

2、优化后
select x.val,y.val from
(select key,val from a where a.ds='2009-07-07') x
left outer join
(select key,val from b where b.ds='2009-07-07') y
on x.key=y.key

```

7、小表放前大表放后原则。在编写带有join操作的代码语句时，应该将条目少的表放在join操作符的左边。因为Reduce阶段，位于join操作符左边的表的内容会被加载进内存，载入数据量较小的表可以有效减少OOM即内存溢出。

8、mapjoin()。当小表与大表join时，采用mapjoin，即在map端完成。同时可以避免小表与大表join产生的数据倾斜

```sql
select /*+ mapjoin(b)*/ a.key,a.value
from a join b
on a.key=b.key

b是小表

```

9、去重优化。尽量避免使用distinct进行去重，特别是大表操作，使用group by代替。

```sql
1、select distinct key from a
如果a表数据特别庞大的话，distinct key很容易造成数据倾斜，就是key一样的话他会放到
同一个mapreduce进行处理

2、select key from a group by key
效果是等价的，但效率比较高。因为group by可以避免数据倾斜
```

10、排序优化。

只有order by产生的结果是全局有序的，可以根据实际场景进行选择排序

- order by 实现全局排序，一个reduce实现，由于不能并发执行，所以效率偏低
- sort by 实现部分有序，单个reduce输出的结果是有序的，效率高。通过和distribute by关键字一起使用（distribute by关键字可以指定map到reduce端的分发key）
- cluster by col1等价于distribute by col1 sort by col1，但不能指定排序规则 如何判断出现数据倾斜：任务进度长时间维持在99%，查看任务监控画面，发现只有少量reduce子任务未完成。因为其处理的数据量和其他reduce差异过大



left semi join与join的区别

join不会去重，left semi join会去重



合理使用动态分区,使用动态分区要开启两个参数

```sql
set hive.exec.dynamic.partition =true   开启hive动态分区表
set hive.exec.dynamic.partition.mode = nonstrict   开启允许所有分区都是动态的，否则必须要有静态分区才能用
```







## Hive函数相应用法

Hive常用数据类型：String、Int、Double

类型转换操作：cast(字段名 or 具体值 as 指定类型)如：select cast(1 as double)from table; 

explode(myCol) 行转列函数，把一行转化为一列



### Hive分区表

所谓的分区表就是将数据以一种方式进行组织，比如分层存储。简单理解就是分区就是目录，建立一个目录进行存储。

分区表分为静态分区和动态分区。 

所谓静态分区就是自己手动设置分区字段。什么时候需要动态分区？加入中国有50个省份，每个省有50个市，每个市都有100个区，如果使用静态分区就不知道什么时候搞完，工作量也大，此时就需要动态分区。

 静态分区的值必须在动态分区值的前面



## 知识盲区

#### 参数设置

##### spark

spark.rdd.compress

这个参数决定了RDD Cache过程中，RDD数据在序列化之后是否进一步进行压缩再存储到内存或磁盘上。

##### hive

set hive. exec.dynamic.partition =true.   开启hive动态分区表

set hive.exec.dynamic.partition.mode = nonstrict.   开启允许所有分区都是动态的，否则必须要有静态分区才能用

#### Hive SQL

from_unixtime(timestamp,format)：即将时间戳转换为日期类型进行显示  

cast(expression AS data_type)：将某种数据类型的表达式转换为另一种数据类型 

nvl(表达式1，表达式2)：空值转换函数。其表达式的值可以是数字型、字符型和日期型。但是表达式1和表达式2的数据类型必须为同一个类型。对数字型： NVL（ comm,0);对字符型 NVL( TO_CHAR(comm), 'No Commission')对日期型 NVL（hiredate,' 31-DEC-99')

NVL2(表达式1，表达式2，表达式3）：如果表达式1为空，返回值为表达式3的值。如果表达式1不为空，返回值为表达式2的值

 lag() over()与lead() over()函数是跟偏移量相关的两个分析函数

lag()：可以在一次查询中取出同一字段的前N行数据

lead()：可以在一次查询中取出同一字段的后N行数据

例如：这是原始数据表

![原始数据表](/Users/chenli75/Desktop/MyFile/github/MyKnowledgeRepository/internship_picture/lag()前原始数据表.png)

获取当前记录的ID以及下一条记录的ID

```sql
select t.id id ,
       lead(t.id, 1, null) over (order by t.id)  next_record_id, 
       t.cphm
from tb_test t       
order by t.id asc

```

![lead()函数数据](/Users/chenli75/Desktop/MyFile/github/MyKnowledgeRepository/internship_picture/lead()函数数据.png)

获取当前记录的ID以及上一条记录的ID

```sql
select t.id id ,
       lag(t.id, 1, null) over (order by t.id)  next_record_id,
       t.cphm
from tb_test t       
order by t.id asc

```

![lag()函数数据](/Users/chenli75/Desktop/MyFile/github/MyKnowledgeRepository/internship_picture/lag()函数数据.png)

获取号码牌号码相同，获取当前记录的ID以及下一条记录的ID（使用partition by）

```sql
select t.id id, 
       lead(t.id, 1, null) over(partition by cphm order by t.id) next_same_cphm_id, 
       t.cphm
from tb_test t
     order by t.id asc 

```

![lead()partition by](/Users/chenli75/Desktop/MyFile/github/MyKnowledgeRepository/internship_picture/lead()partition by数据.png)

insert into 与 insert overwriter的异同

insert into与insert overwriter都可以向hive表中插入数据，但是insert into直接追加到表中数据的尾部；insert overwriter会重写数据。



like与rlike的区别

like的内容不是正则，而是通配符。

两个模式：_ 和%

_ ：表示单个字符，用来查询定长的数据

%：表示0个或者多个任意字符

rlike是正则，正则写法和java一样。'\'需要使用'\\'

.：匹配任何单字符

*：匹配前面表达式零次或多次

+：匹配前面表达式一次或多次

？：匹配前面表达式零次获一次



union与union all

union用于把来自多个select语句结果组合到一个结果集合中，语法为：

```sql
select  column,......from table1
union [all]
select  column,...... from table2
```

union 会去重

union all 不会去重 



With table as :创建一个临时表，放在select 语句前面

语法：

```sql
with temptable as(select ...)
select ...
```

这个临时表只能用于查询，不能用于更新



Hive 解析json

Get_json_object(string json_string,string path):用于解析json字符串，支持多层嵌套解析

第一个参数是要解析的字符串

第二个参数是要获取的key,第二个参数使用$表示json变量标识，然后用.或者[]读取对象或数组

```json
data =
{
 "store":
        {
         "fruit":[{"weight":8,"type":"apple"}, {"weight":9,"type":"pear"}],  
         "bicycle":{"price":19.95,"color":"red"}
         }, 
 "email":"amy@only_for_json_udf_test.net", 
 "owner":"amy" 
}
```

get单层值

```sql
hive> select  get_json_object(data, '$.owner') from test;
结果：amy
```

get多层值

```sql
hive> select  get_json_object(data, '$.store.bicycle.price') from test;
结果：19.95
```

get数组值

```sql
hive> select  get_json_object(data, '$.store.fruit[0]') from test;
结果：{"weight":8,"type":"apple"}
```





Case when then else end多条件判断

```sql
SELECT
    STUDENT_NAME,
    (CASE WHEN score < 60 THEN '不及格'
        WHEN score >= 60 AND score < 80 THEN '及格'
        WHEN score >= 80 THEN '优秀'
        ELSE '异常' END) AS REMARK
FROM
    TABLE
```



count(常量)、count(*)、count(列名)的区别

count(常量)、count(*)表示直接查询符合条件的数据库表的行数

count(列名)表示查询符合条件的并且列的值不为null的行数

count( * )和count(1)是实现上没有任何区别，效率一样，但还是推荐使用count(*)

Count(列名)需要对字段进行非null判断，效率会低一些



group by、order by后面跟数字指的是 select后面选择的列，1代表第一个列



truncate与drop，delete对比

truncate的作用是清空表，只能作用于表。它会清空表中的所有行，但表结构及其约束



**日期函数**

1、to_date:日期时间转日期函数

```sql
select to_date('2015-04-02 13:34:12');
输出：2015-04-02
```

2、from_unixtime:转化unix时间戳到当前时区的时间格式

```sql
select from_unixtime(1323308943,’yyyy-MM-dd’);
输出：2011-12-08
```

3、unix_timestamp:获取当前unix时间戳

```sql
select unix_timestamp();
输出：1430816254
select unix_timestamp('2015-04-30 13:51:20');
输出：1430373080
```

4、year:返回日期中的年、 month:返回日期中的月份、 day:返回日期中的天、 hour:返回日期中的小时、 minute: 返回日期中的分钟、second:返回日期中的秒

```sql
select year('2015-04-02 11:32:12');
输出：2015
```

5、datediff:返回开始日期减去结束日期的天数

```sql
select datediff('2015-04-09','2015-04-01');
输出：8
```

6、date_sub:返回日期前n天的日期

```sql
select date_sub('2015-04-09',4);
输出：2015-04-05
```

7、date_add:返回日期后n天的日期

 ```sql
 select date_add('2015-04-09',4);
 输出：2015-04-13
 ```

8、from_unixtime+ unix_timestamp Hive中yyyymmdd和yyyy-mm-dd日期之间的切换

```sql
思想：先转换成时间戳，再由时间戳转换为对应格式。
--20171205转成2017-12-05 

select from_unixtime(unix_timestamp('20171205','yyyymmdd'),'yyyy-mm-dd') from dual;

--2017-12-05转成20171205

select from_unixtime(unix_timestamp('2017-12-05','yyyy-mm-dd'),'yyyymmdd') from dual;
```



Instr(str,substr):返回substr在str第一次出现的位置（从1开始计数），如果substr在str中不存在则返回0

```sql
select instr('23e,wec',',') -- 4

select instr('23e,wec','f') -- 0 

select instr('23e,wec','') -- 1

select instr('2f3ef,wec','f') -- 2

select instr('23e,wec','wec') -- 5

select instr('23e,wec','wecv') -- 0

```



hive中的字符连接函数：Concat()、concat_ws

Concat(st1,str2,str3,...)是将字符串连接起来

```sql
select concat('jack',',','mike')
结果：jack,mike
```

Concat_ws(分隔符,str1,str2,...):将每个参数值和第一个参数指定的分隔符依次连接到一起组成新的字符串

注意：如果分隔符取值为null，则将分隔符视作与空串进行拼接。如果其它参数为NULL，在执行拼接过程中跳过取值为NULL的参数。

```sql
select concat_ws('|','jack','mike','may')
结果：jack|mike|may
```



hive中多行合并为一行：collect_set()函数和collect_list()函数。列转行。经常喝和concat_ws()结合使用

Collect_set和collect_list都是将分组中的某一列转换为一个数组返回，collect_set去重，collect_list不去重

```sql
select user,concat_ws('|',collect_set(cast(id as string))) id from table;
结果：张三 1｜2｜3. .
```



Cascade 关键字

删除：删除主表时自动删除从表。删除从表，主表不变

更新：更新主表时自动更新从表。更新从表，主表不变



**删除表**

仅删除表中数据，保留表结构

```sql
hive> truncate table 表名;
```

truncate操作用于删除指定表中的所有行，truncate 不能删除外部表！因为外部表里的数据并不是存放在Hive Meta store中。创建表的时候指定了EXTERNAL，外部表在删除分区后，hdfs中的数据还存在，不会被删除。因此要想删除外部表数据，可以把外部表转成内部表或者删除hdfs文件。



删除表的所有，包括数据和结构

```sql
hive> drop table if exists 表名;
```



删除分区

```sql
alter table table_name drop partition (partition_name='分区名')
```







查看自己跑的数据文件生成时间

```sql
进入hive

show create table 表名
查看在hdfs上的地址

dfs -ls 地址
```



hive表加载数据

```sql
load data local inpath '文件路径'
overwrite into table 表名
partiton(dt='2021-11-11')
```

说明：

有local表示从本地文件系统加载（文件会拷贝到HDFS中）

无local表示从HDFS加载数据（注意：是文件移动，而不是拷贝）

overwrite表示是否覆盖表中的数据（可以指定分区覆盖）（没有overwrite会直接append，不会去重）



空值处理

nvl(表达式1，表达式2)：空值转换函数。其表达式的值可以是数字型、字符型和日期型。但是表达式1和表达式2的数据类型必须为同一个类型。对数字型： NVL（ comm,0);对字符型 NVL( TO_CHAR(comm), 'No Commission')对日期型 NVL（hiredate,' 31-DEC-99')

NVL2(表达式1，表达式2，表达式3）：如果表达式1为空，返回值为表达式3的值。如果表达式1不为空，返回值为表达式2的值

NULLIF(表达式1，表达式2)：如果表达1和表达2相等则返回空(null)，否则返回第一个值

Coalesce(表达式1，表达式2，...，表达式n)，遇到非null值即停止并返回该值，如果所有表达式都是空值，最终返回一个空值。

```
select coalesce(a,b,c);
参数说明：如果a==null,则选择b；如果b==null,则选择c；如果a!=null,则选择a；如果a b c 都为null ，则返回为null。
```













#### Java

Calendar类

作用：为操作日历字段提供一些方法

构造方法：protected Calendar() :由于修饰符是protected，所以无法直接创建该对象。需要通过别的途径生成该对象。通过Calendar.getInstance()生成实例对象

方法：

setTime(Date date):使用给定的Date设置此日历的时间

getTime():返回一个Date表示此日历的时间

addTime():按照日历规则，给指定字段添加或减少时间



String.format()字符串格式化



#### Scala

scala中使用"""创建多行字符串，这样的话，多行的字符前面都会有空格。可以在多行字符的末尾stripMargin，并且在第二行开始的每一行前面添加上｜。

Scala 字符串中取变量的值，在变量字符前面加$，${}可以嵌入表达式。



List/ListBuffer

不可变List：不可变集合，元素的追加会返回新的List，因为它是immutable的

可变ListBuffer：可变集合，必须导入包，可以随时添加删除元素



Scala 使用一个特定的类型来表示可能会导致异常的计算，这个类型就是 Try。

Try 有两个子类型：

1. `Success[A]`：代表成功的计算。
2. 封装了 `Throwable` 的 `Failure[A]`：代表出了错的计算。



#### Spark

Row的数值获取

Row.getAs [T] ("字段" )

Row.getString(index)



spark.sql(sql)返回的是RDD类型的数据



#### Flink

数据会随着时间的流失迅速降低

##### ParameterTool 管理配置

Flink 为了解决读取配置文件问题了提供了一个工具类ParameterTool，它可以读取环境变量、运行参数、配置文件

读取运行参数

```
在 Flink 程序中你可以直接使用 ParameterTool.fromArgs(args) 获取到所有的参数，然后如果你要获取某个参数对应的值的话，可以通过 parameterTool.get("username") 方法
```





## 数据埋点

埋点是一种用户行为数据化的记录，基于业务或者产品的需求，对用户在产品内产生行为的每一个时间对应的页面、位置、属性等植入相关代码。

通常的记录维度为who、when、where、what、how。即用户通过某种方式，在何时何地做了何事。

| who   | 用户ID                   | 1001                   |
| ----- | ------------------------ | ---------------------- |
| When  | 时间戳                   | 16000152233            |
| where | 位置（不局限于地理位置） | 首页                   |
| what  | 行为对象                 | 点击了商品，进入商详页 |
| how   | 行为方式                 | 点击                   |



## Presto

### 定义

Presto是一个开源的分布式查询引擎，适用于交互式分析查询，数据量支持GB到PB级别。它本身不存储数据，但他可以接入多种数据源，比如Hadoop、hive、kafka、redis、mysql等。

它的作用：用于交互式分布式查询

### 工作原理

Presto是在Hadoop上运行的分布式系统，也是采用大规模并行处理相似的架构。它有一个协调节点，与多个工作线程节点同步工作。用户将SQL提交给协调器，由其查询和执行引擎进行解析、计划并将分布式查询计划安排到工作线程节点之间。

所有处理都在内存进行，添加更多工作线程节点可提高并行能力，并加快处理速度。

为了使它可扩展到任何数据源，它的设计采用了抽象存储化，可轻松构建连接器。

数据在其存储位置接受查询，无需将其移动到独立的分析系统中。

### 使用场景

1、mysql跨数据库查询。Presto支持扩展任何数据源

2、数据仓库表的查询。它是基于内存的查询，速度快

### 为什么Presto查询速度比Hive快？

Presto是常驻任务，接受请求立即执行，全内存计算并行计算。

Hive需要使用yarn做资源调度，查询前需要先申请资源，启动进程，并且中间结果会经过磁盘。



## 沟通

如何进行需求沟通？5W2H

5W

1、why-为什么？为什么要这么做？原因是什么？

2、what-是什么？目的是什么？指标是什么？

3、where-何处？在哪里做？从哪里下手？

4、when-何时？什么时间完成？

5、who-谁？谁来完成？谁负责？

2H

1、how-怎么做？如何实施？方法是怎样的？

2、how much-多少？做到什么程度？想要的效果是怎么样的？



## 总结模板

1、背景（为什么做）=》往业务挂靠（解决了什么问题，为什么需要）

2、目标（比如几号上线）

3、进展。比如目前完成了代码评审、代码开发到了哪一步；遇到了什么问题，怎么解决的

4、思考。=》避免出现相同的问题/提升效率

## 各种表的概念

### 一维表和二维表

需要行和列来定位数值的就是二维表，仅靠单行就能锁定全部信息的就是一维表

### 拉链表和流水表

拉链表：所谓拉链就是记录历史，记录一个事物从开始一直到当前状态的所有变化的信息。链表中有开始时间（start_time）和结束时间（end_time）两个字段，同时有dt和dp两个分区字段，end_time也可做分区；

 start_time:数据开始的时间

end_time:数据结束的时间

dp:一般有ACTIVE（线上）和EXPIRED（过期）两个分区。ACTIVE表示数据当前在线上使用，所以其end_time为4712-12-31（系统能处理的最大时间，表示一个达不到的无限向后延伸的时间，意味着该数据在线上永久有效）；EXPIRED表示数据过期，已变更，为历史状态，其end_time是数据变更时具体的时间。

dt:数据所在的时间分区，记录数据从ACTIVE转移到EXPIRED的日期，即数据发生变更的时间，大部分与end_time一致。拉链表不建议使用dt分区



流水表：流水表存放的是一个用户的变更记录，比如在一个张流水表中，一天的数据，会存放一个用户的每条修改记录，但是在拉链表中只有一条记录



### 全量表和增量表

前面有

### 事实表和维度表

维度表：表格里存放了具有独立属性和层次结构的数据，一般有维度编码金和对应的维度说明（标签）。

事实表：表格里存储了能体现实际数据或详细数值，一般由维度编码和事实数据组成。

![魔方-维度表和事实表](/Users/chenli75/Desktop/MyFile/github/MyKnowledgeRepository/internship_picture/魔方-维度表和事实表.jpg)

![维度表和事实表](/Users/chenli75/Desktop/MyFile/github/MyKnowledgeRepository/internship_picture/维度表和事实表.jpg)



## 电商概念

### SPU和SKU

SPU(Standard Product Unit)，标准化产品单元，简称商品。

SKU(Stack Keeping Unit)，库存单位，简称单品。

比如iPhone 8为例，他是一个SPU，是一个商品。但它规格有很多种，比如颜色包含金色、黑色、白色这3种，内存包含32G、64G、128 G这3种，对应9种SKU(3*3)。比如”iPhone 8 白色 128G“，这是一个SKU，它具体到了一个实物。

SPU和SkU是包含关系，一个SPU可以包含若干个SKU，SKU从属于SPU



## 实时计算

 Flink + clickhouse

Jdq -> fdm -> jdq -> gdm ->clickhouse



## 项目





负责内容：

1、大数据应用开发。具体是使用Spark统计屏前客流数据

2、数据仓库的建设和完善。根据业务方的需求，如果数据仓库表指标不能满足业务方的需求，需要回溯数据仓库表字段，从数据源中把数据字段给提取出来，然后一步步往上推到数据仓库应用层，给业务方使用。

3、数据埋点方面的工作。迭代20的客户端和小程序的埋点都有我来设计

4、商品维表数据回刷



**1、屏前客流统计**

需求描述：小程序上展示屏前客流人次指标，需要大数据将租户设备的屏前客流回写给业务系统。

需求实现：需要统计两份表写入业务系统。一份是统计曝光人数、屏前触达人次、互动人数的数据的表，一份是统计不同停留阶段的人数的表。人数是去重的人脸数据，人次是不去重的

具体实现：

1、通过一个设备报警接口，它是单例模式的(写入数据库的类也是单例的)，获取到mysql表中的场景id，存储到 ListBuffer 数据结构中。（场景表数据是存储到mysql数据库的。什么叫场景？比如说设备会放到便利店、药店、商超等场景）spark调用parallelize将ListBuffer集合中的数据转化成 RDD ，createOrReplaceTempView注册为临时试图表。

2、spark从hive中读取adm层的屏前客流数据（曝光人数（有人脸数据去重的）、触达人次（没有正脸的faceid不去重的）、互动人数（有人脸停留时间超过3秒，小于6小时算是互动人数））、设备表数据，分别注册为临时表。将三张表进行关联，将场景、设备、屏前客流数据关联上。分别提取曝光人数、屏前触达人次、互动人数进行union all，通过stay_type字段进行区分（stat_type=1表示曝光人数，stat_type=2表示触达人次，stat_type=3表示互动人数）。并且通过filter算子过滤出设备产品key不为空的数据，写入数据库。

3、统计不同停留阶段的人数：由于adm层的屏前客流指标是设备、年龄、性别粒度的聚合数据，而结果表需要的是设备、停留时长聚合的数据。所以不能使用adm层的屏前客流数据，需要使用gdm层的屏前客流数据。所以这里需要将gdm层的屏前客流表、设备表、场景表进行关联，注册为临时表。通过不同停留阶段进行分组求count(distinct faceid)，将数据结果写入mysql数据库。

```scala
case when(durn<3) then 1 when (durn>=3 and durn<15) then 2 when (durn>=15 and durn<60) then 3 when (durn>=60) then 4 end as stage
```

项目问题：

1、设备id和场景id一直匹配不对。通过数据排查，发现有些设备会出现换绑，或者有些设备是已经弃用了的，所以导致把无用的设备和已经换绑的设备拿来一起计算，导致数据错误。

最后的解决办法就是通过row_number()函数按照最新的日期数据去取设备id。本身屏前客流就是要最新日期的数据，也并不需要历史回刷。

```mysql
row_number() over(partition BY product_key, device_code order by create_time DESC) AS rk
```

2、统计不同停留时间段的客流人数，一直出现错误。最后通过回查源数据表发现，adm层统计的指标是按照设备、性别、年龄粒度进行聚合的，而需求是按照设备、durn停留时长的聚合数据，所以adm表不能使用，需要使用adm表的上一层数据表gdm表。

项目亮点：

通过窗口函数解决设备id和场景id匹配的问题



1.5、迭代20客户端埋点问题

出了自动页面自动上报的问题，需要区分这个自动上报和手动上报的问题



**2、动力报表嵌入平台**

需求背景：就是项目里的数据情况只能由我们内部人来看，京东内部的合作伙伴没有权限查看。所以需要一个字段来进行权限的控制，使他们有权限查看到他们相应的权限数据。发现线上数据为空的情况。

需求实现：通过数据探查，其实只要在app层中basedata表添加一个office_id（组织机构id），后端接口通过这个字段就可以完成权限的控制。

项目问题：沟通。这里的沟通包括需求的沟通、数据表权限申请的沟通，以及应用权限的沟通。

项目亮点：发现线上数据为空的情况，通过排查发现是因为任务调度的顺序出现了问题，导致线上数据为空。在开发过程中，发现device_shop_mapping数据表中门店区域经理、门店省经理这些数据为空，通过任务调度发现，万家那边的任务是3:30分开始跑，而我们的数据是2点半开始跑，所以导致这些数据为空。



**3、商品维表的构建和数据回刷**

需求背景：我们小店需要构建与线上商城商品的品牌品类映射信息

具体实现：

1、取出小店（T+1）所有的商品信息（sku_id）（将曝光、点击、浏览和拉链表中的所有sku_id进行union，并且group by进行去重），缓存到一个表中（cache table）

2、取主站T+1的商品数据和小店的商品信息进行join

难点：百亿商品数据的处理

亮点：hive sql的优化，使用map join()，将小店的sku广播出去



思路：先把2021-01-01到2021-03-31的全量数据都写到dt=2021-03-31这个分区。再用dt=2021-03-31这个分区的全量数据回刷2021-01-01到2021-03-31的数据。这样做虽然数据量比真实的多了，但下游表使用这个表是使用left join的，多了总比少了的好。如果一天一天的刷，那要刷到什么时候。

具体做法：取小店的所有 sku_id，去和主站的sku_id进行join

难点：这样做的难点在于如何确保确保sku_id唯一？比如某个sku_id在这个月属于一级品类下的某个二级品类，下个月他就更换了，属于另一个二级品类，那这样的话就会出现两个sku_id

优化：Mapjoin()。我们小店的sku_id去join主站的sku_id，我们小店一天的sku_id数据量为三千五百万左右,去重后才十万条数据，主站的sku_id数据量为一百四十五亿左右的数据量（14541954244）

(小店sku_id三个月八千五百万，去重后有二十万)

(一天跑完之后join出来的数据：160167，原本数据量是194862，差了20个百分点，要不误差小于等于5个百分点)

(join出来的数据：169968，原本数据量是195005，差了12.8%)

问题：spark作业出现maxResultSize异常而失败。大概意思是有多少个任务的序列化结果总大小大于spark.driver.maxResultSize。

发生此错误的原因是超出了配置的大小限制，配置是2G。spark.driver.maxResultSize是每个Spark action分区的序列化结果的总大小限制（例如，collection 行动算子），如果超过了限制大小，job就会被终止，job aborted 任务流产。

解决办法就是调大spark.driver.maxResultSize的相关参数，设置大一些，我设置成了8G。但是这样做是不妥的，因为集群的环境设置本身是设置好的了，最好不要去破坏，你调大了参数，可能会使集群资源紧缺。再加上本身数据量很大，即使增加了一点作用也不是很大。所以最后的解决办法就是我取一天的主站数据进行join，但是出来的数据少了20个百分点，误差控制到小于等于5个百分点才是可以接受的。所以把主站时间放大到一个月，应该足以包括我们小店的sku_id。



四月到七月没数

小店四月到七月sku去重总量为64万左右（636169），和主站join之后有60万左右（595422），差了6%左右



**4、gdm层和adm层数据回刷**

问题：回刷过程中发现回刷机制出现问题，就是底层数据源表的回刷天数（回刷四天）是和现有表的回刷天数（回刷三天）不一致。

为什么要有回刷机制？那是因为埋点日志有上报延迟的问题



adm层回刷的过程中出现spark异常，spark作业出现maxResultSize异常而失败，排查原因发现是商品维表数据量太大导致的。因为商品维表一天的数据量是三个月的



设置spark.driver.maxResultSize=10G

报错：Cannot broadcast the table that is larger than 8GB: 8 GB

原因：map join(a,b,c)三个表



5、A/B test 商品实验设计

设置三个组，ABC，A 组是对照组、B组是结果展示组、C组是主站点击高的组

难点：结果展示组的商品排序规则



**6、京东180新和360新的数据统计**

主站订单宽表数据量为五千万左右，小店订单数据量为一千左右

思路：小店订单表left join主站的，取主站360天的数据，关联不上的就是360天新。那180天新呢？看下单时间的差值是否大于180

难点：处理百亿级别的数据（每天五千万，处理360天的数据），涉及到spark性能调优

优化：73min(调节参数，分配更多资源) -> 28min(mapjoin优化，主站订单表的优化，减少数据传输) -> 18min(在集群资源充足的情况下，可以达到12min)

参数资源优化

```
--executor-cores 8 \--10   从8提到了10
--num-executors 10 \--16   从10提到了16
--executor-memory 20g  \--40 从20g提到了40g
```

代码优化

```
spark.sqlContext.setConf("spark.default.parallelism", "1000") --并行度提到最大
使用mapjoin方式，小表关联大表
优化主站订单表的数量，也是通过mapjoin
减少了shuffle
```

京东新数据回刷存在的问题

```
1、主站数据只取小店相关数据时，出现了小文件过多的问题，导致运行速度很慢很慢，
解决方案：降低并行度，因为在程序里设置并行度为1000导致小文件过多

2、数据回刷时量级对不上，因为没去重
```



**7、实时扫码**

实现过程：FDM层：从jdq（kafka）中取出消息数据，使用flink自定义flatMap对其进行数据解析封装成对象，再写进jdq中 GDM层：从jdq中取出消息数据，使用flinksql将数据写进clickhouse中

问题：线上问题，fdm层实时任务运行一段时间挂掉，gdm层写入clickhouse失败。

解决：因为数据类型的问题导致，最后统一修改成字符串类型就解决问题了

难点：自定义flatMap函数，继承RichFlatMapFunction

亮点：自定义flatMap中在open方法中使用了线程池，AtomicReference

```scala
private var deviceInfo: AtomicReference[java.util.Map[DeviceIndex, DeviceInfo]] = _
DeviceIndex=(product_key, device_name)  DeviceInfo=(tenant_id, tenant_name, shop_id, shop_name)
```

 ```scala
 //创建单独的线程池 -- 刷新缓存（缓存是本地的）
     val executors = Executors.newSingleThreadScheduledExecutor()
     executors.scheduleAtFixedRate(new Runnable {
       override def run(): Unit = reGetLogicInfo()
     }, 5, 5, TimeUnit.HOURS)
 ```

缓存的是什么信息？为什么要使用缓存？

因为是实时的，埋点上报可能会数据缺失（比如只有id没有name等信息），那我使用缓存可以做一个信息的增强（用它存在的字段去关联已经存在的表，得到更完整的信息）

设置五小时进行一次是为了保持缓存的有效性！因为数据是会变化的（比如出现换绑的情况），如果一直不变缓存，那写入数据库的信息就是错误的

使用缓存的目的：

1、提高性能（不用每次都去查表）

2、保持信息的有效性（保持信息的变化和更新）



为什么使用原子类？

使用原子类是



newSingleThreadScheduledExecutor 创建一个单线程化的支持**定时**的线程池，可以用一个线程周期性执行任务(比如周期7天，一次任务才用1小时,使用多线程就会浪费资源)

scheduleAtFixedRate(commod,initialDelay,period,unit)

initialDelay是说系统启动后，需要等待多久才开始执行。

period为固定周期时间，按照一定频率来重复执行任务。
如果period设置的是3秒，系统执行要5秒；那么等上一次任务执行完就立即执行，也就是任务与任务之间的差异是5s；

如果period设置的是3s，系统执行要2s；那么需要等到3S后再次执行下一次任务。



AtomicReference是作用是对"对象"进行原子操作。

```
AtomicReference的源码比较简单。它是通过"volatile"和"Unsafe提供的CAS函数实现"原子操作。
(01) value是volatile类型。这保证了：当某线程修改value的值时，其他线程看到的value值都是最新的value值，即修改之后的volatile的值。
(02) 通过CAS设置value。这保证了：当某线程池通过CAS函数(如compareAndSet函数)设置value时，它的操作是原子的，即线程在操作value时不会被中断。
```





## 实习

项目内容：

1、公司开发人数？人员分配：几个前后端，几个大数据，几个算法？

整个全渠道广告业务部大团队目前130多人。开发方面的话，30多个研发的（前后端）、20多个算法的、10个大数据的、10个左右的测试团队

2、项目开发流程、发布流程、代码提交规范

开发流程：需求申请、需求调研、数据探查、开发设计、设计评审、开始开发、数据提测、代码review、开始上线

发布流程：在BDP平台调度中心中上传代码jar包，上传脚本

代码提交规范：自己新建一个分支，代码上传合并到dev分支，这一步进行评审，评审通过才正式合并到dev，合并成功后再dev合并到master分支，这一步再次评审

3、项目架构、技术选型、技术难点、哪些优化

离线数仓和实时数仓

项目模块：离线数仓模块、埋点解析模块、实时数仓（flink）模块、实时数仓（spark stream）模块（已弃用）等

项目架构：hdfs + yarn + hive + hbase  + Kafka + spark + flink + clickhouse + mysql

离线数仓： spark(bdm -> fdm -> gdm -> adm)

实时数仓：flink(jdq -> fdm -> jdp -> gdm -> clickhouse)

4、项目有哪些统计指标

200多个指标。这是不合理的指标，因为很多指标感觉重复开发，也没啥关联。今天流量上来了，为什么上来，不知道，今天流量下去了，为什么下去了，不知道。也就说他不能进行追溯。一个优秀的指标体系应该是金字塔的，从最顶层的指标看，发现某个指标下去了，往下追溯，为什么下去了，哪里出现问题了？这也是目前我们小店面临的问题，如何去构建好这个指标体系，是2022年需要攻破的一个难点

点击指标、曝光指标、页面pv指标、扫码指标、下单指标、设备指标（设备总数一万四千台）

5、数仓分层、任务调度、数据流向等

数仓分层：bdm -> fdm -> gdm -> adm

任务调度：BDP上有一个调度中心，我们都在上面调度任务

数据流向：埋点日志上报写进 jdq(kafka)，我们再从 jdq数据同步到fdm层，加上时间分区，然后流向 gdm层进行数据解析，最后流向 adm层进行数据聚合等



## 开会

1、他们要你解决什么问题？

2、你解决的是什么问题？

3、还有更好的方法吗？

4、只讨论现阶段能做的需求，没有收益的需求不做。

5、现阶段要解决什么问题



## 讲座

自动卖货

机器学习 - ”没有免费的午餐“定理的理解

最大化数据价值的关键在于：一定要有体系(全链路)



## 项目总结

### 项目背景

京屏小店是京东实现全渠道广告业务数字化的重要一环，京屏小店分析平台对京屏小店每日的业务交易数据进行实时计算和监控，业务部门需要实时监控曝光、浏览、点击、扫码、成单等各种数据指标。由BI报表进行数据的展示，来实现对业务交易数据的统计分析。京屏小店对交易数据进行实时采集、实时数据分析等、做到实时大屏监控和展示等。



### 项目介绍

京屏小店是一款线下可交互广告屏，目前已经在全国铺设1w+台，铺设场景/门店有京东之家、京东家电、京东健康、京车会、见福便利店、地铁站等。

京屏小店是京东实现全渠道广告业务数字化的重要一环，该项目会对屏上的数据指标比如曝光、浏览、点击、扫码等进行离线和实时统计，并通过实时大屏和BI报表平台进行实时展示。它通过jdq也就是Kafka进行数据的接入，使用hdfs作为数据的存储、yarn作为资源的调度，离线数据层采用hive、spark等进行离线计算，实时数据层采用flink、clickhouse等进行实时计算。

我主要负责的是离线数仓模块和离线计算模块，一小部分实时数仓模块。具体是使用spark统计屏前客流人数和人次、京东新用户指标，并对spark程序进行优化，解决数据倾斜问题；使用flink对实时扫码指标开发。



### 项目职责

使用spark统计屏前客流人数和人次

使用spark进行京东新用户指标开发

使用flink对实时扫码指标开发



1、离线计算模块，主要实现屏前客流人数和人次、不同时间段的停留人数、京东新用户指标开发等

2、实时计算模块，主要实现实时扫码指标开发

3、数据权限应用开发，主要是针对不同租户对数据权限的查看。

4、数据仓库历史数据的回刷



### 项目环境

hdfs + yarn + zookeeper + kafka + hive + spark + flink + clickhouse + hbase 



### 项目问题

1、统计屏前客流人数和人次时，设备id和场景id一直匹配不上。通过数据排查，发现有些设备会出现换绑，或者有些设备是已经弃用了的，所以导致把无用的设备和已经换绑的设备拿来一起计算，导致数据错误。最后的解决办法就是按照最新的日期数据去取设备id。本身屏前客流就是要最新日期的数据，也并不需要历史回刷。

2、统计不同停留时间段的客流人数，一直出现错误。最后通过回查源数据表发现，adm层统计的指标是按照设备、性别、年龄粒度进行聚合的，而需求是按照设备、durn停留时长的聚合数据，所以adm表不能使用，需要使用adm表的上一层数据表gdm表。

3、数据权限应用开发中，发现线上数据为空的情况，通过任务调度发现，我们依赖的父任务是3:30分开始跑，而我们的数据是2点半开始跑。所以导致这些数据为空。

4、回刷历史数据时，由京东主站数据量太大导致spark作业异常问题

5、实时扫码中使用flink和clickhouse都是自己之前没用过的数据，经常利用课余时间学习

6、数据模型设计中需要考虑字段的全面性和后面需要使用的便捷性





















