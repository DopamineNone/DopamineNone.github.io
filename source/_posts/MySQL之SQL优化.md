---
title: MySQL之SQL优化
date: 2024-08-25 10:09:04
tags: 
    - MySQL
    - SQL
categories: 数据库
---

## 插入数据

1. 批量插入（但也不用一次插入超过1000条记录）

   `insert into ... values (...) (...) ... (...)`

2. 手动提交事务（可以避免频繁的事务开启和关闭）

   ```sql
   start transaction;
   insert ...
   insert ...
   ...
   commit;
   ```

3. 主键顺序插入性能高于乱序插入

4. 大批量插入数据使用MySQL提供的load指令

   ```cmd
   # 客户端连接服务端加上参数 --local-infile
   mysql --local-infile -u root -p
   
   # 设置全局参数local_infile=1
   set global local_infile = 1;
   
   # 执行load指令
   load data local infile '/path/to/file' into table `table_name` fields terminated by ',' lines terminated by '\n';
   ```

   ## 主键优化
   
   InnoDB的存储方式为**索引组织表（IOT）**,即表数据是根据主键顺序组织存放的。
   
   ### 页分裂
   
   插入新数据时，若当前页没有足够的空间容纳新插入的数据，则会进行页分裂，即
   
   - 确认分割点
   - 创建新页
   - 重新分配记录
   - 更新父节点
   
   > 乱序插入可能导致频繁的页分裂，增加更多的磁盘I/O开销
   
   ### 页合并
   
   删除记录时，往往不会被物理删除，而是标记被删除记录空间，供其他记录声明使用；当页中删除的记录达到一定比例（**MERGE_THRESHOLD**），InnoDB会尝试合并相邻的页。
   
   ### 主键设计原则
   
   1. 满足业务需求的情况下，尽量降低主键长度（长主键会导致更多的磁盘空间占用和增加更多磁盘I/O开销）
   2. 插入时间尽量选择顺序插入，使用auto_increment自增主键
   3. 业务操作中，避免对主键的修改
   
   > 尽量不要使用UUID或其他自然主键作为主键，如身份证，理由同1
   
   ## order by
   
   InnoDB中排序的实现：
   
   1. Using filesort: 提供索引或全表扫描，读取相应的记录，在缓冲区进行排序
   2. Using index: 通过有序索引顺序扫描直接返回结果
   
   给相应被排序字段创建索引即可实现优化（1->2）。
   
   如果已经存在相关字段的（联合）索引，以下情况仍会导致using filesort:
   
   - order by 字段顺序违背最左前缀法则
   - order by 不同字段的升降序性不同
     - 创建索引时默认字段按升序排列
     - 创建索引时指定相应字段的升降序性可以实现优化
   
   > - using index优化的前提是用到索引的覆盖索引
   > - 大数据量排序，不可避免filesort时，可以适当提高排序缓冲区大小
   
   ## group by
   
   类似order by，也可以通过建立索引提高效率
   
   ## order by + limit
   
   对于limit操作，数据量（偏离量）越大，检索效率越低
   
   优化方案：覆盖索引+子查询
   
   ```sql
   select t1.* from Table t1, (select id from Table order by id limit 10000000, 10) t2 where t1.id=t2.id
   ```
   
   ## count
   
   - MyISAM引擎把一张表总行数存在磁盘上，执行`count(*)`时就返回这个数，故效率很高
   - InnoDB则每次执行`count(*)`会全表遍历计数
   
   优化方案：维护自定义计数表
   
   > count的四种用法：
   >
   > - count(*) 统计表的总行数， 不取值
   > - count(id/主键) 取值判断，再按行累加
   > - count(字段) 统计该字段不为NULL的记录总数， 取值判断
   > - count(1) 按行累加1 (1可以替换成其他数字)， 不取值
   >
   > 性能上：**count(*) ≈ count(1) > count(id) > count(字段)**
   
   ## update
   
   优化方案：尽量根据主键/索引字段进行更新；避免使用非索引字段进行更新，导致表锁阻塞其他更新操作
