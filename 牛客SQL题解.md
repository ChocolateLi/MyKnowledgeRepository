# 牛客SQL题解

## 1、查找最晚入职员工的所有信息

**题目链接**：[查找最晚入职员工的所有信息](https://www.nowcoder.com/practice/218ae58dfdcd4af195fff264e062138f?tpId=82&&tqId=29753&rp=1&ru=/activity/oj&qru=/ta/sql/question-ranking)

**解题思路**：

1. 第一步先用max()函数查找出最晚入职的时间
2. 再查找出最晚入职的员工

**代码**：

```sql
select * 
from employees
where hire_date=(
    select max(hire_date)
    from employees
)
```

## 2、查找入职员工时间排名倒数第三的员工所有信息

**题目链接**：[查找入职员工时间排名倒数第三的员工所有信息](https://www.nowcoder.com/practice/ec1ca44c62c14ceb990c3c40def1ec6c?tpId=82&&tqId=29754&rp=1&ru=/activity/oj&qru=/ta/sql/question-ranking)

**解题思路**：

使用子查询，先查出hire_date的排名倒数第三的时间点，主要distinct进行去重

**代码**：

```sql
select *
from employees
where hire_date=(
    select distinct hire_date
    from employees
    order by hire_date desc
    limit 2,1
)
```

