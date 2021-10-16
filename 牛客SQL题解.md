# 牛客SQL题解

## 1、查找最晚入职员工的所有信息

题目链接：[查找最晚入职员工的所有信息](https://www.nowcoder.com/practice/218ae58dfdcd4af195fff264e062138f?tpId=82&&tqId=29753&rp=1&ru=/activity/oj&qru=/ta/sql/question-ranking)

解题思路：

1. 第一步先用max()函数查找出最晚入职的时间
2. 再查找出最晚入职的员工

代码：

```sql
select * 
from employees
where hire_date=(
    select max(hire_date)
    from employees
)
```

