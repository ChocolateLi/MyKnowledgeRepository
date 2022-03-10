# 手撕SQL题

## 常用函数用法

### 一、数值计算函数

#### 1、取整函数：round

**语法**：round(double a)

**返回值**：BIGINT

**说明**：返回double类型的整数部分

```sql
select round(3.1415) from test
3

select round(3.5) from test
4
```

#### 2、指定精度取整函数：round

**语法**：round(double a,int b)

**返回值**：BIGINT

**说明**：返回指定精度d的double类型

```sql
select round(3.14159,4) from test
3.1416
```

#### 3、向下取整函数：floor

**语法**：floor(double a)

**返回值**：BIGINT

**说明**：返回等于或者小于该double变量的最大的整数

```sql
select floor(3.1415) from test
3
```

#### 4、向上取整函数：ceil

**语法**：floor(double a)

**返回值**：BIGINT

**说明**：返回等于或者大于该double变量的最小的整数

```sql
select ceil(3.1415) from test
4
```

### 二、日期函数

#### 1、UNIX时间戳转日期函数：from_unixtime

**语法**：from_unixtime(bigint unixtime[, string format])

**返回值**：string

**说明**：转化UNIX时间戳到当前时区的时间格式

```sql
select from_unixtime(1323308943,'yyyyMMdd') from test;
20111208
```

#### 2、获取当前UNIX时间戳函数: unix_timestamp

**语法**：unix_timestamp()

**返回值**：bigint

**说明**：获得当前时区的UNIX时间戳

```sql
select unix_timestamp() from test;
1323309615
```

#### 3、日期转UNIX时间戳函数: unix_timestamp

**语法**：unix_timestamp(string date)

**返回值**：bigint

**说明**：转换格式为"yyyy-MM-dd HH:mm:ss"的日期到UNIX时间戳。如果转化失败，则返回0。

```sql
select unix_timestamp('2011-12-07 13:01:03') from test;
1323234063
```

#### 4、指定格式日期转UNIX时间戳函数: unix_timestamp

**语法**：unix_timestamp(string date,string pattern)

**返回值**：bigint

**说明**：转换pattern格式的日期到UNIX时间戳。如果转化失败，则返回0。

```sql
select unix_timestamp('20111207 13:01:03','yyyyMMdd HH:mm:ss') from test;
1323234063
```

#### 5、日期时间转日期函数：to_date

**语法**: to_date(string timestamp)
**返回值**: string
**说明**: 返回日期时间字段中的日期部分

```sql
select to_date('2011-12-08 10:03:01') from test;
2011-12-08
```

#### 6、日期转年(year)\转月(month)\转日(day)\转小时(hour)\转分钟(minute)\转秒(second)

**语法**: year(string date)\month(string date)\day(string date)\...
**返回值**: string
**说明**: 返回日期中的年\月\日

```sql
select year('2011-12-08 10:03:01') from test;
2011

select year('2011-12-08') from test;
2011

select month('2011-12-08 10:03:01') from test;
12

select day('2011-12-08 10:03:01') from test;
08
```

#### 7、日期转周函数：weekofyear

**语法**: weekofyear(string date)
**返回值**: int
**说明**: 返回日期在当前的周数

```sql
select weekofyear('2011-12-08 10:03:01') from test;
49
```

#### 8、日期比较函数: datediff
**语法**: datediff(string enddate, string startdate)
**返回值**: int
**说明**: 返回结束日期减去开始日期的天数

```sql
select datediff('2012-12-08','2012-12-01') from test;
7

select datediff('2012-12-01','2012-12-08') from test;
-7
```

#### 9、日期增加函数：date_add

**语法**: date_add(string startdate, int days)
**返回值**: string
**说明**: 返回开始日期startdate增加days天后的日期

```sql
select date_add('2012-12-08',10) from test;
2012-12-18
```

#### 10、日期减少函数：date_sub

**语法**: date_sub (string startdate, int days)
**返回值**: string
**说明**: 返回开始日期startdate减少days天后的日期

```sql
select date_sub('2012-12-11',10) from test;
2012-12-01
```

#### 11、格式化函数：date_format

**语法**：date_format(string date,string format)

**返回值**：string

**说明**：返回format形式的日期

```sql
select date_format('2020-12-20,'yyyy-mm') from test
2020-12
```



### 三、条件函数

#### 1、if函数

**语法**: if(boolean testCondition, T valueTrue, T valueFalseOrNull)
**返回值**: T
**说明**: 当条件testCondition为TRUE时，返回valueTrue；否则返回valueFalseOrNull

```sql
select if(1=2,100,200) from test;
200

select if(1=1,100,200) from test;
100
```

#### 2、非空查找函数coalesce

**语法**: COALESCE(T v1, T v2, …)
**返回值**: T
**说明**: 返回参数中的第一个非空值；如果所有值都为NULL，那么返回NULL

```sql
select COALESCE(null,'100','50ʹ) from test;
100
```

#### 3、条件判断函数case

**语法**: CASE a WHEN b THEN c [WHEN d THEN e]* [ELSE f] END
**返回值**: T
**说明**：如果a等于b，那么返回c；如果a等于d，那么返回e；否则返回f

```sql
-- 简单 CASE表达式  
CASE 列(或表达式)  
     WHEN <匹配值1> THEN <表达式>  
     WHEN <匹配值2> THEN <表达式>  
     ......  
     ELSE <表达式>  
END 

Select case 100 when 50 then 'tom' when 100 then 'mary' else 'tim' end from test;
mary

-- 简单 CASE表达式 示例  
CASE sex  
    WHEN '1' THEN '男'  
    WHEN '2' THEN '女'  
    ELSE '其他'   
END 
```

**语法**: CASE WHEN a THEN b [WHEN c THEN d]* [ELSE e] END
**返回值**: T
**说明**：如果a为TRUE,则返回b；如果c为TRUE，则返回d；否则返回e

```sql
-- 搜索 CASE表达式  
CASE WHEN <判断表达式> THEN <表达式>  
     WHEN <判断表达式> THEN <表达式>  
     WHEN <判断表达式> THEN <表达式>  
     ......  
     ELSE <表达式>  
END 

select case when 1=2 then 'tom' when 2=2 then 'mary' else 'tim' end from test;
mary

-- 搜索CASE表达式 示例  
CASE WHEN sex = '1' THEN '男'  
     WHEN sex = '2' THEN '女'  
     ELSE '其他'   
END  
```

### 四、字符串函数

#### 1、字符串长度函数：length
**语法**: length(string A)
**返回值**: int
**说明**：返回字符串A的长度

```sql
select length('abcedfg') from test;
7
```

#### 2、字符串反转函数：reverse
**语法**: reverse(string A)
**返回值**: string
**说明**：返回字符串A的反转结果

```sql
select length('abcedfg') from test;
gfdecba
```

#### 3、字符串连接函数：concat
**语法**: concat(string A, string B…)
**返回值**: string
**说明**：返回输入字符串连接后的结果，支持任意个输入字符串

```sql
select concat('abc','def','gh') from test;
abcdefgh
```

#### 4、带分隔符字符串连接函数：concat_ws
**语法**: concat_ws(string SEP, string A, string B…)
**返回值**: string
**说明**：返回输入字符串连接后的结果，SEP表示各个字符串间的分隔符

```sql
select concat_ws('-','abc','def','gh') from test;
abc-def-gh
```

#### 5、字符串截取函数：substring
**语法**: substring(string A, int start, int len)
**返回值**: string
**说明**：返回字符串A从start位置开始，长度为len的字符串

```sql
select substring('abcde',3) from test;
cde

select substr('abcde',-1) from test;
e

select substring('abcde',3,2) from test;
cd

select substring('abcde',-2,2) from test;
de
```

#### 6、正则表达式替换函数：regexp_replace
**语法**: regexp_replace(string A, string B, string C)
**返回值**: string
**说明**：将字符串A中的符合java正则表达式B的部分替换为C。注意，在有些情况下要使用转义字符,类似
oracle中的regexp_replace函数。

```sql
select regexp_replace('2020/12/12', '/', '-') from test;
2020-12-12
```

#### 7、json解析函数：get_json_object

**语法**：Get_json_object(string json_string,string path)

**返回值**：string

**说明**：第一个参数是要解析的字符串、第二个参数是要获取的key,第二个参数使用$表示json变量标识，然后用.或者[]读取对象或数组

```sql
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

-- get单层值
select  get_json_object(data, '$.owner') from test;
amy

-- get多层值
select  get_json_object(data, '$.store.bicycle.price') from test;
19.95

-- get数组值
select  get_json_object(data, '$.store.fruit[0]') from test;
{"weight":8,"type":"apple"}
```

#### 8、分割字符串函数: split
**语法**: split(string str, string pat)
**返回值**: array
**说明**: 按照pat字符串分割str，会返回分割后的字符串数组

```sql
select split('ab\cd\ef','\') from test;
["ab","cd","ef"]
```

### 五、聚合统计函数

注意：

```sql
1、当聚集函数和非聚集函数一起时，需要将非聚集函数进行group by
2、当只做聚集函数时，就不需要group by了

select id,sum(score) from student
group by id
```

#### 1、个数统计count

**语法**: count(), count(expr), count(DISTINCT expr[, expr_.])
**返回值**：int
**说明**：count()统计检索出的行的个数，包括NULL值的行；count(expr)返回指定字段的非空值的个数；
count(DISTINCT expr[, expr_.])返回指定字段的不同的非空值的个数

```sql
select count(*) from test;
30
select count(t) from test
20
select count(distinct t) from test;
10
```

#### 2、总和统计函数: sum
**语法**: sum(col), sum(DISTINCT col)
**返回值**: double
**说明**: sum(col)统计结果集中col的相加的结果；sum(DISTINCT col)统计结果中col不同值相加的结果

```sql
select sum(t) from test;
100

hive> select sum(distinct t) from test;
70
```

#### 3、平均值统计函数: avg
**语法**: avg(col), avg(DISTINCT col)
**返回值**: double
**说明**: avg(col)统计结果集中col的平均值；avg(DISTINCT col)统计结果中col不同值相加的平均值

```sql
select avg(t) from test;
50

hive> select avg (distinct t) from test;
30
```

#### 4、最小值统计函数: min
**语法**: min(col)
**返回值**: double
**说明**: 统计结果集中col字段的最小值

```sql
select min(t) from test;
20
```

#### 5、最大值统计函数: max
语法: max(col)
返回值: double
说明: 统计结果集中col字段的最大值

```sql
select max(t) from test;
120
```



### 六、常见窗口函数

数据准备

```
孙悟空 语文 87
孙悟空 数学 95
孙悟空 英语 68
唐僧 语文 94
唐僧 数学 56
唐僧 英语 84
猪八戒 语文 64
猪八戒 数学 86
猪八戒 英语 84
沙僧 语文 65
沙僧 数学 85
沙僧 英语 78
```

#### 1、rank() 分析函数
**语法**：rank()over(partition by field order by field desc)
**说明**： rank() 排序相同时会重复，总数不会变。比如 1 2 2 4

**使用**：

```sql
select name,subject,score,rank()over(partition by subject order by score
desc) as rank from score;
```

**结果**：

```
name subject score rank
孙悟空 数学 95 1
猪八戒 数学 86 2
沙僧 数学 85 3
唐僧 数学 56 4
猪八戒 英语 84 1
唐僧 英语 84 1
沙僧 英语 78 3
孙悟空 英语 68 4
唐僧 语文 94 1
孙悟空 语文 87 2
沙僧 语文 65 3
猪八戒 语文 64 4
```

#### 2、dense_rank() 分析函数
**语法**：dense_rank() over(partition by field order by field desc)
**说明**： 排序相同时会重复，总数会减少。比如 1 2 2 3
**使用**：

```sql
select name,subject,score,dense_rank() over(partition by subject order by score desc)
as rank from score;
```

**结果**

```
name subject score rank
孙悟空 数学 95 1
猪八戒 数学 86 2
沙僧 数学 85 3
唐僧 数学 56 4
猪八戒 英语 84 1
唐僧 英语 84 1
沙僧 英语 78 2
孙悟空 英语 68 3
唐僧 语文 94 1
孙悟空 语文 87 2
沙僧 语文 65 3
猪八戒 语文 64 4
```

#### 3、row_number()分析函数

**语法**：row_number() over(partition by field order by field desc)
**说明**： 即使有相同的数据，也会按照连续排序。比如 1 2 3 4
**使用**：

```sql
select name,subject,score,row_number()over(partition by subject order by score
desc) as rank from score;
```

**结果**

```
name subject score rank
孙悟空 数学 95 1
猪八戒 数学 86 2
沙僧 数学 85 3
唐僧 数学 56 4
猪八戒 英语 84 1
唐僧 英语 84 2
沙僧 英语 78 3
孙悟空 英语 68 4
唐僧 语文 94 1
孙悟空 语文 87 2
沙僧 语文 65 3
猪八戒 语文 64 4
```



### 七、行列转换

#### 1、行转列

**语法**：concat_ws(',',collect_set(col))

**说明**：多行转成一列。collect_list 不去重，collect_set 去重，两者都是产生array类型字段。 column 的数据类型要求是 string

**公式**：concat_ws(",",collect_set(要转成行的列)) group by 分组列 （多列间隔符concat_ws+封装多列
组合体 collect_set+ 分组字段 grtoup by ）

**使用**：

数据集emp表

```
deptno  |  ename  
+---------+---------+--+
20        SMITH   
30        ALLEN   
30        WARD    
20        JONES   
30        MARTIN  
30        BLAKE   
10        CLARK   
20        SCOTT   
10        KING    
30        TURNER  
20        ADAMS   
30        JAMES   
20        FORD    
10        MILLER
```

行转列，COLLECT_SET(col)：函数只接受基本数据类型，它的主要作用是将某字段的值进行去重汇总，产生array类型字段。

```sql
select deptno,concat_ws('|',collect_set(ename)) as ems
from emp
group by deptno
```

结果

```
10    CLARK|KING|MILLER
20    SMITH|JONES|SCOTT|ADAMS|FORD
30    ALLEN|WARD|MARTIN|BLAKE|TURNER|JAMES
```



![](https://img2018.cnblogs.com/i-beta/1687933/201912/1687933-20191211160806248-1113810143.png)



#### 2、列转行

**语法**：lateral view explode(split(column, ',')) as 别名

**说明**：一列转多行。

EXPLODE(col)：将hive一列中复杂的array或者map结构拆分成多行。

LATERAL VIEW
用法：LATERAL VIEW udtf(expression) tableAlias AS columnAlias
解释：用于和split, explode等UDTF一起使用，它能够将一列数据拆成多行数据，在此基础上可以对拆分后的数据进行聚合。

**使用**：

数据集 test

```
10    CLARK|KING|MILLER
20    SMITH|JONES|SCOTT|ADAMS|FORD
30    ALLEN|WARD|MARTIN|BLAKE|TURNER|JAMES
```

```sql
select deptno,name
from test
lateral view explode(split(test.name,'|')) as name
```

![](https://img2018.cnblogs.com/i-beta/1687933/201912/1687933-20191211161009433-13957448.png)





## 常见面试题

### 1、字节跳动一面题

```
第一道，有一个成绩表数据t1
uid	class_name score
A	Chinese		90
A	English		89
A	Math		88
B	...			...

求出总分并排序
uid	chinese_score	English_score	Math_score	total_score	rank
A	90				89				88			267			1
B	...
```

思路：

1.可以先求出uid,total_score的值 total_score表

2.t1表展开，注意case when用法转换

3.两表进行关联

4.最后使用排序函数排序

```sql
--1.先求出total_score表
select uid,sum(score) as total_score from t1 group by uid;total_score表

--2.t1表的展开
select uid,
	max(case when class_name='Chinese' then score end) as chinese_score,
	max(case when class_name='English' then score end) as English_score,
	max(case when class_name='Math' then score end) as Math_score
from
	t1
group by
	uid;stu
	
--3.两表关联stu_score
select 
	t1.uid,t1.chinese_score,t1.English_score,t1.Math_score,t2.total_score
from
	(select * from stu)t1
left join
	(select * from total_score)t2
on
	t1.uid=t2.uid;stu_score

--4.对上述表进行排序
select 
	*,row_number()over(order by total_score desc) as rank
from
	stu_score
```

### 2、字节跳动二面

```
两张表，第一张表是用户看某个视频的播放量，比如用户A看视频Java看了10次
uid	video_id play_cnt
A	java		10
A	python		5
B	Java		20
...

第二张是视频维表video表
video_id	video_name	video_url	video_author

最后求出视频播放量前500的视频信息
video_id	video_name	video_url	total_play_cnt	rank
```

思路：

1.先求出每个video被播放的总数

2.再和视频维表相关联

3.最后排序

```sql
--1.求出被播放的总数
select video_id,sum(play_cnt) as total_play_cnt
from t1
group by video_id;total

--2.和视频维表相关联
select
	v.video_id,v.video_name,v.video_url,t.total_play_cnt
from
	(select video_id,video_name,video_url from video)v
left join
	(select video_id,total_play_cnt)t
on v.video=t.video;tmp

--3.使用排序函数排序
select 
	video_id,video_name,video_url,total_play_cnt,
	row_number()over(order by total_play_cnt desc)
from
	tmp
```

### 3、京东面试题

有50W个京东店铺，每个顾客访问过任何一个店铺的任何一个商品都会产生一条访问日志，访问日志表名为visit，用户id为user_id，店铺名称为shop

```
user_id	shop
u1		a
u2		b
u1		b
...
```

**1、统计店铺的UV(访客数)**

```
select shop,count(distinct user_id) from shop
```

**2、每个店铺访问次数top3的访客信息。输出店铺名称、访客id、访问次数**

```sql
--1.统计出每个用户在店铺中的访问次数
select 
	shop,user_id,count(*) as total_count
from
	shop
group by
	shop,user_id;s1
--2.使用窗口函数排序只取前三个
select 
	shop,user_id,total_count
from
(
    select
        shop,user_id,total_count,
        rank()over(partition by shop order by total count desc) as rank
    from s1
)
where rank<=3;
```

### 4、京东面试题

求非北京市的学生中，每个班级 平均分最高的TO10用户及信息，结果格式为：class, uid, avg_score, scores,province

```
用户学分表Score：
字段名	中文名	字段类型	字段示例
uid	用户id	bigint	23145
class	班级	string	2-1
scores	用户信息	array<string>	['math_100','music_90',......]

用户信息表   stu
字段名	中文名	字段类型	字段示例
uid	用户id	bigint	23145
province	省份	string	北京
```

思路：

1.先将scores分数展开

2.求出每个班级、每个用户的平均分

3.关联省份表过滤出非北京市的学生

4.使用窗口函数取前十的学生

```sql
--1.先将scores分数展开
select 
	uid,class,score
from Score
lateral view explode(scores) as score;score

--2.求出每个班级、每个用户的平均分
select
	class,uid,avg(score) as avg_score
from 
	score
group by
	class,uid;avg_score
	
--3.关联省份过滤出非北京市的学生
select 
	a.class,a.uid,a.avg_score,s.province
from
(select * from avg_score)a
left join
(select * from stu) s
on a.uid=s.uid
where s.province!='北京';stu_info

--4.使用窗口函数取出每个班级、平均分前十的学生用户
select class,uid,avg_score,province
from
(
select
	*,row_number()over(partition by class order by avg_score desc) as rank
from
	stu_info
) t
where rank<=10;
```

























