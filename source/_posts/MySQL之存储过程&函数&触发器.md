---
title: MySQL之存储过程&函数&触发器
date: 2024-08-27 14:00:00
tags: 
    - MySQL
categories: 数据库
---

## 存储过程

存储过程是实现经过编译并存储在数据库中的一段SQL语句的集合，即实现了SQL代码的封装与重用

特点：

- 封装、复用
- 可以接收参数，也可以返回数据
- 减少网络交互，效率提升

### 存储过程的使用

#### 创建

```sql
CREATE PROCEDURE procedure_name([参数列表])
BEGIN
	-- sql
END;
```

> 命令行中创建存储过程需要通过delimiter指定SQL语句的结束符

#### 调用

```sql
call procedure_name (arguments...)
```

#### 查看

```sql
-- 查看指定数据库的存储过程
select * from information_schema.ROUTINES where ROUTINE_SCHEMA = 'database_name';
-- 查看创建存储过程的sql语句
show create procedure procedure_name;
```

#### 删除

```sql
drop procedure if exists procedure_name;
```

### 变量

#### 系统变量

MySQL提供了一些系统变量，分为**全局变量**和**会话变量**

##### 查看

```sql
show [session|global] variables;	-- 查看所有系统变量
show [session|global] variables like '...';
select @@[session|global].系统变量名;
```

##### 设置

```sql
set [session|global] variable_name = '...';
```

> 重启MySQL后，全局变量会恢复为配置文件中设置的默认值

#### 用户自定义变量

##### 赋值

```sql
set @variable = 'value';
set @variable := 'value'; -- 推荐，用于区分赋值和等于
select @variable := 'value';
select column_a into @variable from table_name ...;
```

##### 查看

```sql
select @variable;
```

#### 局部变量

在局部生效的变量，通过`DECLARE`声明，作用域处于变量所在的`BEGIN...END`块中。

##### 声明

```sql
DECLARE variable_name type [default default_value];
```

##### 赋值

```sql
set variable_name := value;
select column_a into variable_name from table_name;
```

### if语句

```sql
if condition then
	-- sql
elseif condition then
	-- sql
else
	-- sql
end if;
```

### 参数

类型：

- IN 输入参数
- OUT 返回值
- INOUT 既作为输入参数，也作为输出参数

```sql
create procedure p_name(IN/OUT/INOUT pramameter_name type)
begin
end;
```

### case语句

```sql
case
	when condition then
		-- sql
	when condition then
		-- sql
	else
		-- sql
end case;
```

### while语句

类似C的while循环

```sql
WHILE condition DO
	-- sql
END WHILE;
```

### repeat 语句

类似C的do while循环

```sql
REPEAT
	-- sql
	UNTIL condition
END REPEAT;
```

### loop语句

```sql
[label:] LOOP
	-- sql
	LEAVE label; -- break
	ITERATE label; -- continue
END LOOP [label];
```

### 游标

cursor，是用来存储查询结果集的数据类型。

#### 声明

```sql
DECLARE cursor_name CURSOR FOR select_statement;
```

> 游标的声明必须在普通变量的声明之后

#### 打开

```sql
OPEN cursor_name
```

#### 获取游标记录

```sql
FETCH cursor_name INTO variable_name, [...]; 
```

#### 关闭游标

```sql
CLOSE cursor_name;
```

### 条件处理程序

```sql
declare handler_action handler for condition_value statement;
```

- handler_action
  - continue
  - exit
- condition_value
  - sqlstate sqlstate_value
  - sqlwarning 01开头的SQLSTATE
  - not found 02开头的SQLSTATE
  - sqlexception 

## 存储函数

```sql
create function func_name ([arguments ...])
returns type [characteristic...]
begin
	-- sql
	return ...;
end;
```

- characteristic
  - deterministic 系统的输入参数总是产生相同的结果
  - no sql 不包含SQL语句
  - reads sql data 包含读取数据的语句，但不包含写入数据的语句

## 触发器

触发器能在`insert/update/delete`之前之后，触发并执行触发器中定义的SQL语句集合。触发器使用OLD和NEW来引用触发器中发生变化的记录内容。MySQL只支持行级触发器，不支持语句级触发器。

### 创建

```sql
CREATE TRIGGER trigger_name
[BEFORE|AFTER] [INSERT|UPDATE|DELETE]
ON table_name FOR EACH ROW
BEGIN
	-- sql
END;
```

### 查看

```sql
SHOW TRIGGERS;
```

### 删除

```sql
DROP TRIGGER [schema_name.]trigger_name;
```

