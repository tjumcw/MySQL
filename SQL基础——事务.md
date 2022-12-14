# SQL基础——事务

### 一、事务简介

事务是一组操作的集合，它是一个不可分割的工作单位。这些操作要么同时成功，要么同时失败。

>注意事项：
>
>-  默认MySQL的事务是自动提交的，也就是说，当执行完一条DML语句时，MySQL会立即隐式的提交事务。



### 二、事务操作

数据准备：

```mysql
drop table if exists account;

create table account(
    id int primary key AUTO_INCREMENT comment 'ID',
    name varchar(10) comment '姓名',
    money double(10,2) comment '余额'
) comment '账户表';

insert into account(name, money) VALUES ('张三',2000), ('李四',2000);

mysql> select * from account;

+----+------+---------+
| id | name | money   |
+----+------+---------+
|  1 | 张三 | 2000.00 |
|  2 | 李四 | 2000.00 |
+----+------+---------+
```



#### 1、未控制事务

##### 1）测试正常情况

```mysql
-- 1. 查询张三余额
select * from account where name = '张三';
-- 2. 张三的余额减少1000
update account set money = money - 1000 where name = '张三';
-- 3. 李四的余额增加1000
update account set money = money + 1000 where name = '李四';

mysql> select * from account;

+----+------+---------+
| id | name | money   |
+----+------+---------+
|  1 | 张三 | 1000.00 |
|  2 | 李四 | 3000.00 |
+----+------+---------+
```

- 前后数据一致



##### 2）测试异常情况

```mysql
-- 1. 查询张三余额
select * from account where name = '张三';
-- 2. 张三的余额减少1000
update account set money = money - 1000 where name = '张三';
出错了....
-- 3. 李四的余额增加1000
update account set money = money + 1000 where name = '李四';

mysql> select * from account;

+----+------+---------+
| id | name | money   |
+----+------+---------+
|  1 | 张三 | 1000.00 |
|  2 | 李四 | 2000.00 |
+----+------+---------+
```



#### 2、控制事务一

##### 1）查看/设置事务提交方式

```mysql
SELECT @@autocommit;
SET @@autocommit = 0;
```



##### 2）提交事务

```mysql
COMMIT;
```



##### 3）回滚事务

```mysql
ROLLBACK;
```

>注意事项：
>
>- 上述的这种方式，我们是修改了事务的自动提交行为, 把默认的自动提交修改为了手动提 交, 此时我们执行的DML语句都不会提交, 需要手动的执行commit进行提交。



#### 3、控制事务二

##### 1）开启事务

```mysql
START TRANSACTION 或 BEGIN;
```



##### 2）提交事务

```mysql
COMMIT;
```



##### 3）回滚事务

```mysql
ROLLBACK;
```



##### 4）转账案例

```mysql
-- 开启事务
start transaction

-- 1. 查询张三余额
select * from account where name = '张三';

-- 2. 张三的余额减少1000
update account set money = money - 1000 where name = '张三';

-- 3. 李四的余额增加1000
update account set money = money + 1000 where name = '李四';

-- 如果正常执行完毕, 则提交事务
commit;

-- 如果执行过程中报错, 则回滚事务
-- rollback;
```



### 三、事务四大特性

- 原子性：事务是不可分割的最小操作单元，要么全部成功，要么全部失败。
- 一致性：事务完成时，必须使所有的数据都保持一致状态。
- 隔离性：数据库系统提供的隔离机制，保证事务在==不受外部并发操作影响的独立环境下==运行。
- 持久性：事务一旦==提交或回滚==，它对数据库中的数据的改变就是永久的。

上述事务四大特性简称ACID特性



### 四、==并发事务问题==

#### 1、脏读

- 一个事务读到另一个事务还未提交的数据（读到了中间状态）



#### 2、不可重复读

- 一个事务先后读取同一条记录，但两次读取的数据不同，称之为不可重复读。（两次读取中另一个事务更新并提交了）



#### 3、==幻读==

- 一个事务按照条件查询数据时，没有对应的数据行，但是在插入数据时，又发现这行数据 已经存在，好像出现了 "幻影"。

![image-20220810104305928](C:\Users\mcw\AppData\Roaming\Typora\typora-user-images\image-20220810104305928.png)

>理解：
>
>- 第一步事务A查询id = 1的数据行，发现没有，准备插入对应数据
>- 第二步事务B插入了一条id = 1的数据并提交数据库
>- 第三步事务A插入时报错，因为id是主键，数据库中已有id = 1数据无法插入
>- 第四步事务A再去查询id = 1的数据行，发现还是查不到（已经解决不可重复读的情况下，查不到该数据）
>  - 解决不可重复读，即要求两次读取的数据需要保持一致
>  - 即事务A两次读取读不到其他事务已经提交的数据，但数据库中实际已经有了



### 五、==事务隔离级别==

事务隔离级别

| 隔离级别 |   脏读   | 不可重复读 |   幻读   |
| :------: | :------: | :--------: | :------: |
| 读未提交 | 不能解决 |  不能解决  | 不能解决 |
| 读已提交 |  能解决  |  不能解决  | 不能解决 |
| 可重复读 |  能解决  |   能解决   | 不能解决 |
|  串行化  |  能解决  |   能解决   |  能解决  |

>注意事项：
>
>- 从上往下隔离级别越高，数据越安全，但性能越低



##### 1）查看事务隔离级别

```mysql
SELECT @@TRANSACTION_ISOLATION;
```

##### 2）修改事务隔离级别

```mysql
SET [ SESSION | GLOBAL ] TRANSACTION ISOLATION LEVEL { READ UNCOMMITTED | READ COMMITTED | REPEATABLE READ | SERIALIZABLE }
# SESSION指当前会话、GLOBAL指全局
```

##### 3）初始表结构

```mysql
id   name    money
1	 张三	    2000
2	 李四		2000
```



#### 1、读未提交

- 该隔离级别最低，解决不了任何问题（读未提交表示可以读到未提交的数据）
- 脏读就是指读到了数据的中间状态，即另一事务未提交的数据

>模拟过程如下：
>
>- 分别在两个会话开启事务A和事务B
>- 事务B执行了一个更新操作（张三money - 1000），但未提交
>- 事务A可以读到张三money变化，即读到了其他事务未提交的数据
>
>==理解==：该级别相当于两个事务之间是透明的，不存在任何隔离



#### 2、读已提交

- 能解决脏读，因为该隔离级别保证只能读到已提交的数据

>模拟过程如下：
>
>- 分别在两个会话开启事务A和事务B
>- 事务B执行了一个更新操作（张三money - 1000），但未提交
>- 事务A查询数据时，数据未发生变化
>- 事务B提交后，事务A再去查询，才能看到数据的变化
>- 解决了脏读，但出现了不可重复读的问题，即两次读取同一数据结果不一致



- 不能解决其他问题，比如不可重复读就是再次读时读到了其他事务已提交的变化数据

>模拟过程如下：
>
>- 分别在两个会话开启事务A和事务B
>- 事务A查询数据
>- 事务B执行了一个更新操作（张三money - 1000），但未提交（此时A肯定查不到变化）
>- 事务B提交后，事务A再次查询数据，发现与第一次查询结果不同



#### 3、可重复读（MySQL默认隔离级别）

- 可重复读能解决不可重复读（必然能解决脏读），不能解决幻读
- 他保证了一个事务两次读取的数据一致
  - 对另一事务的隔离做的更彻底，其提交带来的影响更小

>模拟过程如下：
>
>- 分别在两个会话开启事务A和事务B
>- 事务A查询数据
>- 事务B执行了一个更新操作（张三money - 1000）并提交
>- 事务A再去查询数据，发现未发生变化，即保证了可重复读
>  - ==在一个事务中执行两次相同的读操作，结果是一样的==
>- 此时提交了事务A，再去查询发现数据变化了（或者直接在事务B的会话也能直接查到变化）



- 在该隔离级别下，依然会出现幻读问题

>模拟过程如下：
>
>- 分别在两个会话开启事务A和事务B
>- 在事务A中查询id = 3的数据，发现没有，于是准备插入该数据，此刻
>- 在事务B中先插入一条id = 3的数据，并提交（数据库中已经有该id = 3的数据了）
>- 此时事务A去插入id = 3的数据，发现报错，已经存在主键id = 3的数据，重复了
>- 事务A再去查id = 3的数据，还是没有（==因为已经保证了可重复读==）
>  - 可重复读保证事务执行两次一样的读操作读到的数据是一致的
>- 事务A查不到，但也无法插入，就出现了幻读



#### 4、串行化

- 可以解决所有问题，但性能最差，相当于没有并发了

>模拟过程如下：
>
>- 分别在两个会话开启事务A和事务B
>- 在事务A中查询id = 3的数据，发现没有，于是准备插入该数据，此刻
>- 在事务B中先插入一条id = 3的数据，但是该SQL没有执行成功，阻塞在那里（光标一直闪）
>  - 因为事务A先开启，并且仍在操作还未提交，直到提交后事务B才可以执行
>- 此时事务A插入一条id = 3的数据，插入成功，然后事务A提交
>- 提交的同时，事务B阻塞的SQL执行，并且报错，因为数据库中已经有id = 3的数据了