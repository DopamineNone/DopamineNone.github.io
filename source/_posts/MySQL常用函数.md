---
layout: passage
title: MySQL常用函数
date: 2024-08-19 15:40:00
tag:
  - MySQL
categories: 数据库
---

## 字符串函数

- concat(s1, s2, ..., sn) 拼接多个字符串
- upper(s) 将字符串中所有字符转化为大写字母
- lower(s) 将字符串中所有字符转化为小写字母
- lpad(str, n, pad) 用pad左填充字符串str至长度为n的字符串
- rpad(str, n, pad) 类似lpad，但右填充
- trim(str) 去掉字符串头尾的空格
- length(str) 获取字符串长度
- substring(str, start, len) 返回字符串str从start起len长度的字符串(MySQL中字符串下标从1开始计算)

```sql
select concat('Hello', 'World'); -- Hello World

select upper('Hello'); -- HELLO

select lower('Hello'); -- hello

select lpad('Hello', 6, '0'); -- 0Hello

select rpad('Hello', 6, '0'); -- Hello0

select trim(' Hello  '); -- Hello

select substring('MySQL', 3, length('MySQL') - 2); -- SQL
```

## 数值函数

- ceil(x) 向上取整
- floor(x) 向下取整
- mod(x, y) 求 x % y
- rand() 返回一个0到1的浮点数
- round(x, y) 对x进行四舍五进，保留y位小数

```sql
select ceil(1.1); -- 2

select floor(1.1); -- 1

select mod(9, 4); -- 1

select round(1 / 3, 3); -- 0.333

-- 生成6位随机数字
select lpad(floor(rand() * 1000000), 6, '0');
```

## 日期函数

- curdate() 	返回当前日期（年月日）
- curtime()         返回当前时间（时分秒）
- now()              返回当前日期+时间（YY-MM-DD HH:MM:SS)
- year(date)       解析日期字符串并返回年份
- month(date)   解析日期字符串并返回月份
- day(date)        解析日期字符串并返回天数
- date_add(date, interval expr type) 计算date加上一个日期偏移量的结果
- datediff(date1, date2)  返回date1到date2的日数差

```sql
select now(); -- 2024-08-19 15:40:00

select curdate(); -- 2024-08-19

select curtime(); -- 15:40:00

select year('2024-08-19 15:40:00'); -- 2024

select month('2024-08-19 15:40:00'); -- 8

select day('2024-08-19 15:40:00'); -- 19

select date_add('2024-08-19 15:40:00', interval 1 day); -- 2024-08-20 15:40:00 

select datediff('2024-08-20 15:40:00', '2024-08-19 15:40:00'); -- 1
```

## 流程函数

- if(value, t, f), 若value为真，则返回t，否则返回f
- ifnull(value1, value2) , 若value1不为空，则返回value1，否则返回value2
- case when [val1] then [res] ... end 条件分支语句

```sql
select if(true, '1', '2'); -- 1

select ifnull('', '2'); -- ''

select ifnull(null, '2'); -- 2

-- http://sqlmother.yupi.icu/#/learn/level13
SELECT
  name,
  CASE
    WHEN (age > 60) THEN '老同学'
    WHEN (age > 20) THEN '年轻'
    ELSE '小同学'
  END AS age_level
FROM
  student
ORDER BY
  name asc;
```

