# 1. 查询列

```sql
# 查询所有列
SELECT * FROM user_profile

# 查询多列
SELECT device_id, gender, age, university FROM user_profile

# 查询结果去重
SELECT DISTINCT university FROM user_profile

# 查询结果限制返回行数
SELECT device_id FROM user_profile LIMIT 2

# 查询后重命名
SELECT device_id FROM user_profile AS user_infos_example LIMIT 2

# 否定查询
select device_id, gender, age, university
from user_profile
where not university = '复旦大学'

# 过滤空值
select device_id, gender, age, university
from user_profile
where not age is null

# in
select device_id, gender, age, university, gpa 
from user_profile
where university in ('北京大学', '复旦大学', '山东大学')

# like
select device_id, age, university
from user_profile
where university like '%北京%'

# max() min()
select max(gpa)
from user_praofile
where university = '复旦大学'

# avg() count()
select count(gender) as male_num, avg(gpa) as avg_gpa
from user_profile
where gender = 'male'

# having
select university, avg(question_cnt) avg_question_cnt, avg(answer_cnt) avg_answer_cnt
from user_profile
group by university
having avg_question_cnt < 5 or avg_answer_cnt < 20
# 生成新字段后不能用where要用having

```

# 2. 涉及到多个表的情况

```sql
select university, (count(q.question_id)/count(distinct(q.device_id))) avg_answer_cnt
from user_profile u
join question_practice_detail q
on u.device_id = q.device_id
group by university
order by university asc
```

```sql
select u.university, q.difficult_level, count(qp.question_id)/count(distinct qp.device_id) avg_answer_cnt
from user_profile u, question_practice_detail qp, question_detail q
where u.university = '山东大学' and u.device_id = qp.device_id and q.question_id = qp.question_id
group by u.university, difficult_level
order by avg_answer_cnt
```



## **SQL25** 查找山东大学或者性别为男生的信息

题目：现在运营想要分别查看学校为山东大学或者性别为男性的用户的device_id、gender、age和gpa数据，请取出相应结果，结果不去重。

示例：user_profile

| id   | device_id | gender | age  | university | gpa  | active_days_within_30 | question_cnt | answer_cnt |
| ---- | --------- | ------ | ---- | ---------- | ---- | --------------------- | ------------ | ---------- |
| 1    | 2138      | male   | 21   | 北京大学   | 3.4  | 7                     | 2            | 12         |
| 2    | 3214      | male   |      | 复旦大学   | 4    | 15                    | 5            | 25         |
| 3    | 6543      | female | 20   | 北京大学   | 3.2  | 12                    | 3            | 30         |
| 4    | 2315      | female | 23   | 浙江大学   | 3.6  | 5                     | 1            | 2          |
| 5    | 5432      | male   | 25   | 山东大学   | 3.8  | 20                    | 15           | 70         |
| 6    | 2131      | male   | 28   | 山东大学   | 3.3  | 15                    | 7            | 13         |
| 7    | 4321      | male   | 26   | 复旦大学   | 3.6  | 9                     | 6            | 52         |

根据示例，你的查询应返回以下结果（注意输出的顺序，先输出学校为山东大学再输出性别为男生的信息）：

| device_id | gender | age  | gpa  |
| --------- | ------ | ---- | ---- |
| 5432      | male   | 25   | 3.8  |
| 2131      | male   | 28   | 3.3  |
| 2138      | male   | 21   | 3.4  |
| 3214      | male   | None | 4    |
| 5432      | male   | 25   | 3.8  |
| 2131      | male   | 28   | 3.3  |
| 4321      | male   | 28   | 3.6  |

```sql
select device_id, gender, age, gpa
from user_profile
where university = '山东大学'
union all 
select device_id, gender, age, gpa
from user_profile
where gender = 'male'
```

# 3. if

## SQL26 **计算25岁以上和以下的用户数量**

题目：现在运营想要将用户划分为25岁以下和25岁及以上两个年龄段，分别查看这两个年龄段用户数量

**本题注意：age为null 也记为 25岁以下**

示例：user_profile

| id   | device_id | gender | age  | university | gpa  | active_days_within_30 | question_cnt | answer_cnt |
| ---- | --------- | ------ | ---- | ---------- | ---- | --------------------- | ------------ | ---------- |
| 1    | 2138      | male   | 21   | 北京大学   | 3.4  | 7                     | 2            | 12         |
| 2    | 3214      | male   |      | 复旦大学   | 4    | 15                    | 5            | 25         |
| 3    | 6543      | female | 20   | 北京大学   | 3.2  | 12                    | 3            | 30         |
| 4    | 2315      | female | 23   | 浙江大学   | 3.6  | 5                     | 1            | 2          |
| 5    | 5432      | male   | 25   | 山东大学   | 3.8  | 20                    | 15           | 70         |
| 6    | 2131      | male   | 28   | 山东大学   | 3.3  | 15                    | 7            | 13         |
| 7    | 4321      | male   | 26   | 复旦大学   | 3.6  | 9                     | 6            | 52         |

根据示例，你的查询应返回以下结果：

| age_cut    | number |
| ---------- | ------ |
| 25岁以下   | 4      |
| 25岁及以上 | 3      |

```sql
select age_cut, count(device_id) number
from(select if(age >= 25, '25岁及以上', '25岁以下') as age_cut, device_id from user_profile) t1
group by age_cut
```

# 4. case

## **SQL27** **查看不同年龄段的用户明细**

题目：现在运营想要将用户划分为**20岁以下，20-24岁，25岁及以上**三个年龄段，分别查看不同年龄段用户的明细情况，请取出相应数据。（注：若**年龄为空**请返回**其他**。）

示例：user_profile

| id   | device_id | gender | age  | university | gpa  | active_days_within_30 | question_cnt | answer_cnt |
| ---- | --------- | ------ | ---- | ---------- | ---- | --------------------- | ------------ | ---------- |
| 1    | 2138      | male   | 21   | 北京大学   | 3.4  | 7                     | 2            | 12         |
| 2    | 3214      | male   |      | 复旦大学   | 4    | 15                    | 5            | 25         |
| 3    | 6543      | female | 20   | 北京大学   | 3.2  | 12                    | 3            | 30         |
| 4    | 2315      | female | 23   | 浙江大学   | 3.6  | 5                     | 1            | 2          |
| 5    | 5432      | male   | 25   | 山东大学   | 3.8  | 20                    | 15           | 70         |
| 6    | 2131      | male   | 28   | 山东大学   | 3.3  | 15                    | 7            | 13         |
| 7    | 4321      | male   | 26   | 复旦大学   | 3.6  | 9                     | 6            | 52         |

根据示例，你的查询应返回以下结果：

| device_id | gender | age_cut    |
| --------- | ------ | ---------- |
| 2138      | male   | 20-24岁    |
| 3214      | male   | 其他       |
| 6543      | female | 20-24岁    |
| 2315      | female | 20-24岁    |
| 5432      | male   | 25岁及以上 |
| 2131      | male   | 25岁及以上 |
| 4321      | male   | 25岁及以上 |

```sql
select device_id, gender, 
case
    when age < 20 then '20岁以下'
    when age < 25 then '20-24岁'
    when age >= 25 then '25岁及以上'
    else '其他'
end age_cut
from user_profile
```

# 5. day()

- day()

- month()

- year()


# 6. 用户留存率计算

[SQL-计算用户次日留存率.md](/Users/xiaoyu/Documents/博客/SQL-计算用户次日留存率.md)

# 7. 字符串相关

[SQL-字符串相关.md](/Users/xiaoyu/Documents/博客/SQL-字符串相关.md)
