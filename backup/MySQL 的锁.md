线上系统中，在高并发的访问之下，出现了 死锁问题 或者 等待锁时间过长导致超时，那么碰到这些情况，就可能问你锁相关的问题
首先还是先给锁分类，之后再来逐个了解：
按照功能划分：

- 共享锁：也叫 S 锁 或 读锁，是共享的，不互斥
- 加锁方式：select ... lock in share mode
- 排他锁：也叫 X 锁 或 写锁，写锁阻塞其他锁
- 加锁方式：select ... for update

按照锁的粒度划分：

- 全局锁：锁整个数据库
- 表级锁：锁整个表
- 行级锁：锁一行记录的索引
- 记录锁：锁定索引的一条记录
- 间隙锁：锁定一个索引区间
- 临键锁：记录锁和间隙锁的结合，解决幻读问题
- 插入意向锁：执行 insert 时添加的行记录 id 的锁
- 意向锁：存储引擎级别的“表级锁”

相关命令：
```
-- 加锁
flush tables with read lock;

-- 释放锁
unlock tables;
```

表级锁:
表级锁中又分为以下几种：

- 表读锁：阻塞对当前表的写，但不阻塞读
- 表写锁：阻塞队当前表的读和写
- 元数据锁：这个锁不需要我们手动去添加，在访问表的时候，会自动加上，这个锁是为了保证读写的正确
- 当对表做 增删改查 时，会自动添加元数据读锁
- 当对表做 结构变更 时，会自动添加元数据写锁

- 自增锁：是一种特殊的表级锁，自增列事务执行插入操作时产生
查看表级锁的命令：

```
-- 查看表锁定状态
show status like 'table_locks%';
-- 添加表读锁
lock table user read;
-- 添加表写锁
lock table user write;
-- 查看表锁情况
show open tables;
-- 删除表锁
unlock tables;
```

![image](https://github.com/China-Wei/China-Wei.github.io/assets/52816610/306ad97d-9743-4e62-8eb9-1ede2bd8ff7d)

**行级锁**

MySQL 的行级锁是由存储引擎是实现的，InnoDB 的行锁就是通过给 索引加锁 来实现

注意：InnoDB 的行锁是针对索引加的锁，不是针对记录加的锁。并且该索引不能失效，否则会从行锁升级为表锁

行锁根据 范围 分为：记录锁（Record Locks）、间隙锁（Gap Locks）、临键锁（Next-Key Locks）、插入意向锁（Insert Intention Locks）

行锁根据 功能 分为：读锁和写锁

**什么时候会添加行锁呢？**

> 对于 update、insert 语句，InnoDB 会自动添加写锁（具体添加哪一种锁会根据 where 条件判断，后边会提到 加锁规则）
> 对于 select 不会添加锁
> 事务手动给 select 记录集添加读锁或写锁

接下来对记录锁、间隙锁、临键锁、插入意向锁来一个一个解释，这几个锁还是比较重要的，一定要学习！

> 记录锁：
记录锁：锁的是一行索引，而不是记录

那么可能有人会有疑问了，如果这一行数据上没有索引怎么办呢？

其实如果一行数据没有索引，InnoDB 会自动创建一个隐藏列 ROWID 的聚簇索引，因此每一行记录是一定有一个索引的

下边给出记录锁的一些命令:
-- 加记录读锁
```
select * from user where id = 1 lock in share mode;
-- 加记录写锁
select * from user where id = 1 for update;
-- 新增、修改、删除会自动添加记录写锁
insert into user values (1, "lisi");
update user set name = "zhangsan" where id = 1;
delete from user where id = 1;
```

> 间隙锁
间隙锁用于锁定一个索引区间，开区间，不包括两边端点，用于在索引记录的间隙中加锁，不包括索引记录本身

间隙锁的作用是 **防止幻读**，保证索引记录的间隙不会被插入数据
间隙锁在 **可重复读** 隔离级别下才会生效