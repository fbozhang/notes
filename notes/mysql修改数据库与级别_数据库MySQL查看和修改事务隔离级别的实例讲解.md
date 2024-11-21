## mysql事务隔离级别
事务的隔离级别分为：未提交读(read uncommitted)、已提交读(read committed)、可重复读(repeatable read)、串行化(serializable)。

 - **Read Uncommitted(读取未提交内容)**
 
	 在该隔离级别，所有事务都可以看到其他未提交事务的执行结果。本隔离级别很少用于实际应用，因为它的性能也不比其他级别好多少。读取未提交的数据，也被称之为脏读(Dirty Read)。
 - **Read Committed(读取提交内容)**

	这是大多数数据库系统的默认隔离级别(但不是MySQL默认的)。它满足了隔离的简单定义：一个事务只能看见已经提交事务所做的改变。这种隔离级别 也支持所谓的不可重复读(Nonrepeatable Read)，因为同一事务的其他实例在该实例处理其间可能会有新的commit，所以同一select可能返回不同结果。



 - **Repeatable Read(可重读)**
这是MySQL的默认事务隔离级别，它确保同一事务的多个实例在并发读取数据时，会看到同样的数据行。不过理论上，这会导致另一个棘手的问题：幻读 (Phantom Read)。简单的说，幻读指当用户读取某一范围的数据行时，另一个事务又在该范围内插入了新行，当用户再读取该范围的数据行时，会发现有新的“幻影” 行。InnoDB和Falcon存储引擎通过多版本并发控制(MVCC，Multiversion Concurrency Control)机制解决了该问题。


 - **Serializable(可串行化)**
这是最高的隔离级别，它通过强制事务排序，使之不可能相互冲突，从而解决幻读问题。简言之，它是在每个读的数据行上加上共享锁。在这个级别，可能导致大量的超时现象和锁竞争。


## mysql修改事务隔离级别


### 方法1：执行命令修改
**查看当前事物级别**
```python
// 查看当前事物级别：

// 8.0.3 之前
SELECT @@tx_isolation;
// 8.0.3
SELECT @@transaction_isolation;
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5bad6f292b0f5bf276ab312003d9c930.png)


**修改事务隔离级别**

MySQL 提供了 SET TRANSACTION 语句，该语句可以改变单个会话或全局的事务隔离级别。语法格式如下：

```python
SET [SESSION | GLOBAL] TRANSACTION ISOLATION LEVEL {READ UNCOMMITTED | READ COMMITTED | REPEATABLE READ | SERIALIZABLE}
```

其中，SESSION 和 GLOBAL 关键字用来指定修改的事务隔离级别的范围：
SESSION：表示修改的事务隔离级别将应用于当前 session(当前 cmd 窗口)内的所有事务；
GLOBAL：表示修改的事务隔离级别将应用于所有 session(全局)中的所有事务，且当前已经存在的session 不受影响；
如果省略 SESSION 和 GLOBAL，表示修改的事务隔离级别将应用于当前 session 内的下一个还未开始的事务。

任何用户都能改变会话的事务隔离级别，但是只有拥有 SUPER 权限的用户才能改变全局的事务隔离级别。

如果使用普通用户修改全局事务隔离级别，就会提示需要超级权限才能执行此操作的错误信息，SQL 语句和运行结果如下:



```python
C:\Users\leovo>mysql -utestuser -p

Enter password: ******

Welcome to the MySQL monitor. Commands end with ; or \g.

Your MySQL connection id is 41

Server version: 5.7.29-log MySQL Community Server (GPL)

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its

affiliates. Other names may be trademarks of their respective

owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> SET GLOBAL TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;

ERROR 1227 (42000): Access denied; you need (at least one of) the SUPER privilege(s) for this operation

mysql> SET SESSION TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;

Query OK, 0 rows affected (0.00 sec)

```

---

**修改事务隔离级别session 例子**

```python
//设置read uncommitted级别：

set session transaction isolation level read uncommitted;

 

//设置read committed级别：

set session transaction isolation level read committed;

 

//设置repeatable read级别：

set session transaction isolation level repeatable read;

 

//设置serializable级别：

set session transaction isolation level serializable;
```


### 方法2：mysql.ini配置修改
打开mysql.ini配置文件，在最后加上


```python
#可选参数有：READ-UNCOMMITTED, READ-COMMITTED, REPEATABLE-READ, SERIALIZABLE.

[mysqld]

transaction-isolation = REPEATABLE-READ
```

这里全局默认是REPEATABLE-READ，其实MySQL本来默认也是这个级别



## Mysql事务隔离级别之读提交流程图
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d5fde302d621e45945d6880d83b2f3e1.png)







## 脏读

A事务，会读取到B事务还未提交的数据。因为B事务可能会因为各种原因数据回滚，所以如果A事务读取了B未提交的数据，然后基于此进行一些业务操作，但是B事务发生错误回滚了，那A事务的业务操作就错了。
不可重复读
在同一个事务生命周期内，也就是这个事务还未提交之前。如果另外一个事务，对数据进行了编辑(update)或者删除(delete)操作。那么A事务就会读取到。简单理解，就是在一个事务生命周期内，多次查询数据，每次都可能查出来的不一样。

## 幻读

幻读的结果其实和不可重复读是一样的表现，差异就在于，不可重复读，主要是针对其他事务进行了编辑(update)和删除(delete)操作。而幻读主要是针对插入(insert)操作。也就是在一个事务生命周期内，会查询到另外一个事务新插入的数据。

