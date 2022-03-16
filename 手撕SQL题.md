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





## 面试题汇总

### 1、字节跳动面试题

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

### 2、字节跳动面试题

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
	row_number()over(order by total_play_cnt desc) as rank
from
	tmp
where 
	rank<=500
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

### 5、腾讯面试题

**如何对手机号码、身份证号、银行号脱敏?**

concat()、left()、right()字符串函数组合使用

concat(str1,str2,...)：返回结果为连接参数产生的字符串

left(str,len)：返回从字符串str开始的len最左字符

right(str,len)：返回从字符串str开始的len最右字符

```sql
SELECT 
    CONCAT(LEFT(phone,3), '****' ,RIGHT(phone,4)) AS 手机号
FROM phone;
```

### 6、万物心选面试题

查询一个表（tb1）的字段记录不在另一个表（tb2）中 

```sql
select tb1.* from tb1
left join tb2
on tb1.id=tb2.id
where tb2.id is null;
```

### 7、字节跳动面试题

求11月份的新用户次日留存率

牛客网：[2021年11月每天新用户的次日留存率](https://www.nowcoder.com/practice/1fc0e75f07434ef5ba4f1fb2aa83a450?tpId=268&tqId=2285344&ru=/exam/oj&qru=/ta/sql-factory-interview/question-ranking&sourceUrl=%2Fexam%2Foj%3Ftab%3DSQL%25E7%25AF%2587%26topicId%3D268)

```
id login_time 
1	2021-11-01 12：00：01	
2	2021-11-01 12：00：01	
3	2021-11-01 12：00：01	
1	2021-11-02 12：00：01	
2	2021-11-02 12：00：01	
4	2021-11-02 12：00：01	
...
比如这张表的新用户留存率为66.7%
11.1 有三个新用户 1 2 3
11.2 两个新用户留下来了 1 2
所以新用户留存率为 2/3=66.7%
```

```sql
--1.第一步，求出每天新用户登录的日期
select id,date_format(min(login_time),'yyyy-mm-dd') as login_time
from t_login
group by id;new_user

--2.将新用户登录的表和登录表进行left join
select t1.id as id1,t1.login_time as time1,t2.id as id2,t2.login_time as time2
from
(select id,login_time from new_user)t1
left join
(select id,date_format(login_time,'yyyy-mm-dd') from t_login)t2
on t1.id=t2.id and t1.login_time = date_sub(t2.login_time,1);

--3.求根据用户id求和

```



## 常见面试题

### SQL第一题

score表结构：uid,subject_id,score，问题：找出所有科目成绩都大于某一学科平均成绩的学生

```
1001 01 90
1001 02 85
...
```

思路：

1.先求出每个学科的平均分；t1表

2.使用if()函数判断学科成绩是否大于平均分，大于则记为0，否则记为1；t2表

3.根据学生id进行分组统计flag的和，和为0，说明所有学科都大于平均成绩



知识点：

SQL中增加having子句的原因是，where关键字无法与聚合函数一起使用。having子句可以让我们筛选分组后的各组数据

```sql
--1.先求出每个学科的平均分
select uid,score,
avg(score) over(partition by subject_id) avg_score
from score;t1

--2.根据是否大于平均成绩 记录flag，大于记为0否则记为1
select uid,if(score>avg_score,0,1) flag from t1;t2

--3.根据学生id进行分组统计flag的和，和为0，说明所有学科都大于平均成绩
select uid from t2
group by uid
having sum(flag)=0;
```



### SQL第二题

有如下用户访问数据，action表

```
userid	visitDate	visitCount
u01		2017/1/21	5
u02		2017/1/23	6
u01		2017/1/23	6
...
```

问题：要求使用SQL统计出每个用户的累积访问次数，如下表

```
userid	月份	  小计	累积
u01		2017-1	11	   11
u01		2017-2	12	   23
...
```

思路：

1.先对数据进行相应格式转换

2.计算出每人每月的访问量

3.按月访问进行累加

```sql
--1.先对数据进行格式转化
select 
userid,date_format(regexp_replace(visitDate,'/','-'),'yyyy-mm') as visitDate,visitCount
from action;

--2.计算出每人每月的访问量
select 
	userid,visitDate,sum(visitCount) as DateCount
from
	t1
group by userid,visitDate;t2

--3.对每个用户按月访问进行累加
select 
	userid,visttDate,DateCount,
	sum(DateCount) over(partition by userid order by visitDate)
from
	t2;
```



### SQL第三题

已知一个order表，有date,order_id,user_id,amount。请给出sql进行统计。

样例：2017-1-1，2012000，452115005，34

**1、给出2017年每个月的订单数、用户数、成交金额**

格式：2017-1  orderCount userCount amount

思路：

1.先进行格式转化

2.求出订单数、用户数(注意去重)、成交金额，注意where筛选条件为2017年，还要注意对每月进行分组

```sql
select 
	date_format(date,'yyyy-mm') as dt,--1.先进行格式转化
	count(order_id) as orderCount,
	count(distinct user_id) as userCount,
	sum(amount)
where
	date_format(date,'yyyy')='2017'
group by
	date_format(date,'yyyy-mm')
```

**2、给出2017年11月的新客数（指在11月才有第一笔订单）**

```sql
select 
	count(distinct userid)
from 
	`order`
group by
	userid
having
	date_format(min(date),'yyyy-mm')='2017-11'
```



### SQL第四题

请用sql写出所有用户中在今年10月份第一次购买商品的金额，表order字段:user_id,money,paymenttime(2017-10-01 16:04:01),orderid

思路：

1.先找出所有用户10月份第一次购买的记录

2.再两表join找到金额

```sql
--1.先找出所有用户10月份第一次购买的记录
select
	user_id,min(paymenttime) as paymenttime
from
	`table`
where
	date_format(paymenttime,'yyyy-mm')='2017-10'
group by
	user_id;t1
	
--2.再两表join找到金额
select
	t1.user_id,t1.paymenttime,t2.money
from
	(select user_id,paymenttime from t1)t1
join
	(select user_id,money from t2)t2
on t1.user_id=t2.user_id and t1.paymenttime=t2.paymenttime
```

### SQL第五题

有一个线上服务器访问日志格式如下 tabel

```
时间						接口						IP地址
2016-11-09 14:22:05		/api/user/login			110.03.523
...
```

求11月9号下午 14点（14-15点），访问/api/user/login接口的top10的ip地址

```sql
select 
	ip,count(*) as visit_count
from
	`table`
where
	date_format(time,'yyyy-mm-dd HH')>='2016-11-09 14' and
	date_format(time,'yyyy-mm-dd HH')<='2016-11-09 15' and
	interface = '/api/user/login'
group by ip
order by visit_count desc
limit 10;

```

**知识点**

limit子句用于限制查询结果返回的数量。

用法：`select * from tableName limit i,n`

参数：i 表示查询结果的索引值(默认从0开始，从0开始可以省略)

n表示查询结果返回的数量

**总结**

跟字节跳动面试题对比，如果需要加上排名的话，那么要考虑使用窗口函数，如果不加的话，可以考虑使用order by排序 limit取前几位



### SQL第六题

用一条SQL语句查询出每门课程都大于80分的学生姓名

```sql
select
	name
from
 	student
group by
	name
having
	min(score)>=80;--最小的分数都大于80分了，则所有的课程都大于80分
```

### SQL第七题

有下面的表

```
year month amount
1991	1	1.1
1991	2	1.2
1991	3	1.3
1991	4	1.4
1992	1	2.1
1992	2	2.2
1992	3	2.3
1992	4	2.4
...
```

查询结果为这样

```
year	m1	m2	m3	m4
1991	1.1	1.2	1.3	1.4
1992	2.1	2.2	2.3	2.4
```

知识点：使用case when

```sql
select
	year,
	max(case when month='1' then amount end) as m1,
	max(case when month='2' then amount end) as m2,
	max(case when month='3' then amount end) as m3,
	max(case when month='4' then amount end) as m4
from
	`table`
group by year;
```

### SQL第八题

一个info表

```
date	result
2005-05-09	win
2005-05-09	lose
2005-05-09	lose
2005-05-09	win
2005-05-10	win
2005-05-10	lose
2005-05-10	lose
```

统计成这样

```
date 		win lose
2005-05-09	2	2
2005-05-10	1	2
```

知识点：使用sum和case when(或者if)

```sql
--case when
select
	date,
	sum(case when result='win' then 1 else 0 end) as win,
	sum(case when result='lose' then 1 else 0 end) as lose
from
	`table`
group by
	date;
	
--if
select
	date,
	sum(if(result='win',1,0)) as win,
	sum(if(result='lose',1,0)) as lose
from
	`table`
group by
	date;
```











