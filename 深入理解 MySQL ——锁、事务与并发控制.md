# 深入理解 MySQL ——锁、事务与并发控制
> 本文对 MySQL 数据库中有关锁、事务及并发控制的知识及其原理做了系统化的介绍和总结，希望帮助读者能更加深刻地理解 MySQL 中的锁和事务，从而在业务系统开发过程中可以更好地优化与数据库的交互。  

**1、MySQL 服务器逻辑架构**

![](https://mmbiz.qpic.cn/mmbiz_jpg/4g5IMGibSxt7w6aCaVLsJda5uF3szciabDVNEYo0UMA0n4HicLJxIap24r3CTIvaAzeYribP4Z5HhnDsMdB98c5uRA/640?wx_fmt=jpeg)

（图片来源MySQL官网）

每个连接都会在 MySQL 服务端产生一个线程（内部通过线程池管理线程），比如一个 select 语句进入，MySQL 首先会在查询缓存中查找是否缓存了这个 select 的结果集，如果没有则继续执行解析、优化、执行的过程；否则会之间从缓存中获取结果集。

**2、MySQL 锁**

**2.1、Shared and Exclusive Locks （共享锁与排他锁）**

它们都是标准的**行级锁**。

*   **共享锁（S）**共享锁也称为读锁，读锁允许多个连接可以同一时刻并发的读取同一资源,互不干扰；
    
*   **排他锁（X）**排他锁也称为写锁，一个写锁会阻塞其他的写锁或读锁，保证同一时刻只有一个连接可以写入数据，同时防止其他用户对这个数据的读写。
    

**注意：所谓共享锁、排他锁其实均是锁机制本身的策略，通过这两种策略对锁做了区分。**   

**2.2、Intention Locks（意向锁）**

InnoDB 支持多粒度锁(锁粒度可分为行锁和表锁)，允许行锁和表锁共存。例如，一个语句，例如 LOCK TABLES…WRITE 接受指定表上的独占锁。为了实现多粒度级别的锁定，InnoDB 使用了意图锁。  

**意向锁：表级别的锁。先提前声明一个意向，并获取表级别的意向锁（共享意向锁 IS 或排他意向锁 IX），如果获取成功，则稍后将要或正在(才被允许)，对该表的某些行加锁(S或X)了。（除了 LOCK TABLES ... WRITE,会锁住表中所有行，其他场景意向锁实际不锁住任何行)**

举例来说：

SELECT ... LOCK IN SHARE MODE，要获取IS锁；An intention shared lock (IS)

SELECT ... FOR UPDATE ，要获取IX锁；An intention exclusive lock (IX) i

**意向锁协议** 在事务能够获取表中的行上的共享锁之前，它必须首先获取表上的IS锁或更强的锁。 在事务能够获取表中的行上的独占锁之前，它必须首先获取表上的IX锁。

前文说了，意向锁实现的背景是多粒度锁的并存场景。如下兼容性的汇总：

![](https://mmbiz.qpic.cn/mmbiz_jpg/4g5IMGibSxt71YNMJva8uj3gbxJica5QBHo8kicPhicQhMdxVVMr0SJ42gN6mzmuePJPHdQ3QhYNH9xx8Uo8iaJtckQ/640?wx_fmt=jpeg)

意向锁仅表意向，是一种较弱的锁，意向锁之间兼容并行(IS、IX 之间关系兼容并行)。 X与IS\\IX互斥；S与IX互斥。可以体会到，意向锁是比X\\S更弱的锁，存在一种预判的意义！先获取更弱的IX\\IS锁，如果获取失败就不必要再花费跟大开销获取更强的X\\S锁 ... ...

**2.3、Record Locks (索引行锁)**

**record lock 是一个在索引行记录的锁。** 

比如，SELECT c1 FROM t WHERE c1 = 10 FOR UPDATE，如果c1 上的索引被使用到。防止任何其他事务变动 c1 = 10 的行。

record lock 总是会在索引行上加锁。即使一个表并没有设置任何索引，这种时候 innoDB 会创建一个隐式的聚集索引（primary Key）,然后在这个聚集索引上加锁。

**当查询字段没有索引时，**比如 update table set columnA="A" where columnB=“B".如果 columnB 字段不存在索引（或者不是组合索引前缀），这条语句会锁住所有记录也就是锁表。如果语句的执行能够执行一个 columnB 字段的索引，那么仅会锁住满足 where 的行(RecordLock)。

**锁出现查看示例：** 

(使用 show engine innodb status 命令查看)：

```
```范围查询RECORD LOCKS space id 58 page no 3 n bits 72 index `PRIMARY` of table `test`.`t` trx id 10078 lock_mode X locks rec but not gapRecord lock, heap no 2 PHYSICAL RECORD: n_fields 3; compact format; info bits 0 0: len 4; hex 8000000a; asc     ;; 1: len 6; hex 00000000274f; asc     'O;; 2: len 7; hex b60000019d0110; asc        ;;
```

**2.4、Gap locks（间隙锁）**

Gap Locks: **锁定索引记录之间的间隙(\[2\])，或者锁定一个索引记录之前的间隙(\[1\])，或者锁定一个索引记录之后的间隙(\[3\])。** 

示例：如图\[1\]、\[2\]、\[3\]部分。一般作用于我们的范围筛选查询> 、< 、between...... 

![](https://mmbiz.qpic.cn/mmbiz_jpg/4g5IMGibSxt71YNMJva8uj3gbxJica5QBH8hyj1S5EUgFI8iaY25gXciauiapXAeAXVkVzUVfX06ibabpDz8YdhrrJsw/640?wx_fmt=jpeg)

例如， SELECT userId FROM t1 WHERE userId BETWEEN 1 and 4 FOR UPDATE; 阻止其他事务将值3插入到列 userId 中。因为该范围内所有现有值之间的间隙都是锁定的。

*   **对于使用**唯一索引**来搜索唯一行的语句 select a from ，不产生间隙锁定**。(不包含**组合唯一索引**，也就是说 gapLock 不作用于单列唯一索引）
    

例如，如果id列有唯一的索引，下面的语句只对id值为100的行使用索引记录锁，其他会话是否在前一个间隙中插入行并不重要:

> \`\`\` SELECT \* FROM t1 WHERE id = 100;
> 
> \`\`\`如果id\*\*没有索引或具有非惟一索引，则语句将锁定前面的间隙\*\*。

*   **间隙可以跨越单个索引值、多个索引值(如上图2,3)，甚至是空的。** 
    
*   **间隙锁是性能和并发性之间权衡的一种折衷，用于某些特定的事务隔离级别，如RC级别**（RC级别：REPEATABLE READ，我司为了减少死锁，关闭了gap锁，使用RR级别）。
    
*   **在重叠的间隙中（或者说重叠的行记录）中允许gap共存**
    
    比如同一个 gap 中，允许一个事务持有 gap X-Lock（gap 写锁\\排他锁)，同时另一个事务在这个 gap 中持有(gap 写锁\\排他锁)
    

```
CREATE TABLE `new_table` (  `id` int(11) NOT NULL AUTO_INCREMENT,  `a` int(11) DEFAULT NULL,  `b` varchar(45) DEFAULT NULL,  PRIMARY KEY (`id`),  KEY `idx_new_table_a` (`a`),  KEY `idx_new_table_b` (`b`)) ENGINE=InnoDB AUTO_INCREMENT=15 DEFAULT CHARSET=utf8INSERT INTO `new_table` VALUES (1,1,'1'),(2,3,'2'),(3,5,'3'),(4,8,'4'),(5,11,'5'),(6,2,'6'),(7,2,'7'),(8,2,'8'),(9,4,'9'),(10,4,'10');######## 事务一 ########START TRANSACTION;SELECT * FROM new_table WHERE a between 5 and 8 FOR UPDATE;##暂不commit######## 事务二 ########SELECT * FROM new_table WHERE a = 4 FOR UPDATE;##顺利执行！ 因为gap锁可以共存；######## 事务三 ######## SELECT * FROM new_table WHERE b = 3 FOR UPDATE;##获取锁超时，失败。因为事务一的gap锁定了 b=3的数据。
```

**2.5、****next-key lock**

**next-key lock 是 record lock 与 gap lock 的组合。** 

**比如 存在一个查询匹配 b=3 的行(b上有个非唯一索引)，那么所谓 NextLock 就是：在b=3 的行加了 RecordLock 并且使用 GapLock 锁定了 b=3 之前（“之前”：索引排序）的所有行记录。** 

MySQL 查询时执行 行级锁策略，会对扫描过程中匹配的行进行加锁（X 或 S），也就是加Record Lock，同时会对这个记录之前的所有行加 GapLock 锁。 假设一个索引包含值10、11、13和20。该索引可能的NexKey Lock锁定以下区间：

```
(negative infinity, 10](10, 11](11, 13](13, 20](20, positive infinity)
```

**另外，值得一提的是 ： innodb 中默认隔离级别(RR)下，next key Lock 自动开启。**  （很好理解，因为 gap 作用于RR，如果是 RC，gapLock 不会生效，那么 next key lock 自然也不会）

**锁出现查看示例：**  (使用 show engine innodb status 命令查看)：

```
RECORD LOCKS space id 58 page no 3 n bits 72 index `PRIMARY` of table `test`.`t` trx id 10080 lock_mode XRecord lock, heap no 1 PHYSICAL RECORD: n_fields 1; compact format; info bits 0 0: len 8; hex 73757072656d756d; asc supremum;;Record lock, heap no 2 PHYSICAL RECORD: n_fields 3; compact format; info bits 0 0: len 4; hex 8000000a; asc     ;; 1: len 6; hex 00000000274f; asc     'O;; 2: len 7; hex b60000019d0110; asc        ;;
```

**2.6、Insert Intention Locks（插入意向锁）**

**一个 insert intention lock 是一种发生在 insert 插入语句时的 gap lock 间隙锁，锁定插入行之前的所有行。** 

这个锁以这样一种方式表明插入的意图，如果插入到同一索引间隙中的多个事务没有插入到该间隙中的相同位置，则它们不需要等待对方。

假设存在值为4和7的索引记录。尝试分别插入值为5和6的独立事务，在获得所插入行上的独占锁之前，每个事务使用 insert intention lock 锁定4和7之间的间隙，但不会阻塞彼此，因为这些行不冲突。

示例：

```
mysql> CREATE TABLE child (id int(11) NOT NULL, PRIMARY KEY(id)) ENGINE=InnoDB;mysql> INSERT INTO child (id) values (90),(102);##事务一mysql> START TRANSACTION;mysql> SELECT * FROM child WHERE id > 100 FOR UPDATE;+-----+| id  |+-----+| 102 |+-----+
```

```
##事务二mysql> START TRANSACTION;mysql> INSERT INTO child (id) VALUES (101);##失败，已被锁定mysql> SHOW ENGINE INNODB STATUSRECORD LOCKS space id 31 page no 3 n bits 72 index `PRIMARY` of table `test`.`child`trx id 8731 lock_mode X locks gap before rec insert intention waitingRecord lock, heap no 3 PHYSICAL RECORD: n_fields 3; compact format; info bits 0 0: len 4; hex 80000066; asc    f;; 1: len 6; hex 000000002215; asc     " ;; 2: len 7; hex 9000000172011c; asc     r  ;;...
```

**2.7、 AUTO-INC Locks**
-----------------------

AUTO-INC 锁是一种特殊的表级锁，产生于这样的场景：事务插入(inserting into )到具有 AUTO\_INCREMENT 列的表中。

在最简单的情况下，如果一个事务正在向表中插入值，那么其他任何事务必须等待向该表中插入它们自己的值，以便由第一个事务插入的行接收连续的主键值。

**2.8 Predicate Locks for Spatial Indexes 空间索引的谓词锁**
----------------------------------------------------

略

**3、事务**

**事务就是一组原子性的 sql，或者说一个独立的工作单元。 事务就是说，要么 MySQL 引擎会全部执行这一组sql语句，要么全部都不执行（比如其中一条语句失败的话）。** 

*   **自动提交（AutoCommit，MySQL 默认）**
    

```
show variables like "autocommit";set autocommit=0; //0表示AutoCommit关闭set autocommit=1; //1表示AutoCommit开启
```

MySQL 默认采用 AutoCommit 模式，也就是每个 sql 都是一个事务，并不需要显示的执行事务。如果 autoCommit 关闭，那么每个 sql 都默认开启一个事务，只有显式的执行“commit”后这个事务才会被提交。

*   **显****示事务 (****START TRANSACTION...COMMIT****)**
    

比如，tim 要给 bill 转账100块钱：

1.检查 tim 的账户余额是否大于100块；

2.tim 的账户减少100块；

3.bill 的账户增加100块；

这三个操作就是一个事务，必须打包执行，要么全部成功， 要么全部不执行，其中任何一个操作的失败都会导致所有三个操作“不执行”——回滚。

```
CREATE DATABASE IF NOT EXISTS employees;USE employees;CREATE TABLE `employees`.`account` (  `id` BIGINT (11) NOT NULL AUTO_INCREMENT,  `p_name` VARCHAR (4),  `p_money` DECIMAL (10, 2) NOT NULL DEFAULT 0,  PRIMARY KEY (`id`)) ;INSERT INTO `employees`.`account` (`id`, `p_name`, `p_money`) VALUES ('1', 'tim', '200'); INSERT INTO `employees`.`account` (`id`, `p_name`, `p_money`) VALUES ('2', 'bill', '200'); START TRANSACTION;SELECT p_money FROM account WHERE p_name="tim";-- step1UPDATE account SET p_money=p_money-100 WHERE p_name="tim";-- step2UPDATE account SET p_money=p_money+100 WHERE p_name="bill";-- step3COMMIT;
```

一个良好的事务系统，必须满足ACID特点：

**3.1、事务的ACID：** 

*   **A:atomiciy 原子性：一个事务必须保证其中的操作要么全部执行，要么全部回滚，不可能存在只执行了一部分这种情况出现。** 
    
*   **C:consistency 一致性：数据必须保证从一种一致性的状态转换为另一种一致性状态。**  比如上一个事务中执行了第二步时系统崩溃了，数据也不会出现 bill 的账户少了100块，但是 tim 的账户没变的情况。要么维持原装（全部回滚），要么 bill 少了100块同时 tim 多了100块，只有这两种一致性状态的。
    
*   **I：isolation 隔离性：在一个事务未执行完毕时，通常会保证其他 Session 无法看到这个事务的执行结果。** 
    
*   **D:durability 持久性：事务一旦 commit，则数据就会保存下来，即使提交完之后系统崩溃，数据也不会丢失。** 
    

**4、隔离级别**

![](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt7w6aCaVLsJda5uF3szciabDqbTb06jSgicPgmlReUQaXP718ZUELB8ypy9cBWTTlrDsicIK1NnggKPw/640?wx_fmt=png)

```
查看系统隔离级别：select @@global.tx_isolation;查看当前会话隔离级别select @@tx_isolation;设置当前会话隔离级别SET session TRANSACTION ISOLATION LEVEL serializable;设置全局系统隔离级别SET GLOBAL TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
```

**4.1、 READ UNCOMMITTED (未提交读,可脏读)**

**事务中的修改，即使没有提交，对其他会话也是可见的。可以读取未提交的数据——脏读**。脏读会导致很多问题，一般不适用这个隔离级别。 实例：

```
-- ------------------------- read-uncommitted实例 -------------------------------- 设置全局系统隔离级别SET GLOBAL TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;-- Session ASTART TRANSACTION;SELECT * FROM USER;UPDATE USER SET NAME="READ UNCOMMITTED";-- commit;-- Session BSELECT * FROM USER;//SessionB Console 可以看到Session A未提交的事物处理，在另一个Session 中也看到了，这就是所谓的脏读id    name2    READ UNCOMMITTED34    READ UNCOMMITTED
```

**4.2、READ COMMITTED (提交读或不可重复读，幻读)**

一般数据库都默认使用这个隔离级别（MySQL 不是）， 这个隔离级别保证了一个事务如果没有完全成功（commit 执行完），事务中的操作对其他会话是不可见的。

```
-- ------------------------- read-cmmitted实例 -------------------------------- 设置全局系统隔离级别SET GLOBAL TRANSACTION ISOLATION LEVEL READ  COMMITTED;-- Session ASTART TRANSACTION;SELECT * FROM USER;UPDATE USER SET NAME="READ COMMITTED";-- COMMIT;-- Session BSELECT * FROM USER;//Console OUTPUT:id    name2    READ UNCOMMITTED34    READ UNCOMMITTED----------------------------------------------------- 当 Session  A执行了commit，Session B得到如下结果：id    name2    READ COMMITTED34    READ COMMITTED
```

也就验证了 **read committed**级别在事物未完成 commit 操作之前修改的数据对其他 Session 不可见，执行了 commit 之后才会对其他 Session 可见。 我们可以看到 Session B 两次查询得到了不同的数据。

**read committed 隔离级别解决了脏读的问题，但是会对其他 Session 产生两次不一致的读取结果（因为另一个 Session 执行了事务，一致性变化）。** 

**4.3、 REPEATABLE READ (可重复读)**

一个事务中多次执行统一读 SQL,返回结果一样。 这个隔离级别解决了脏读的问题，幻读问题。这里指的是 innodb 的 rr 级别，innodb 中使用 next-key 锁对"当前读"进行加锁，锁住行以及可能产生幻读的插入位置，阻止新的数据插入产生幻行。 下文中详细分析。具体请参考 MySQL 手册：

> [https://dev.mysql.com/doc/refman/5.7/en/innodb-storage-engine.html](https://dev.mysql.com/doc/refman/5.7/en/innodb-storage-engine.html)

**4.4、 SERIALIZABLE (可串行化)**

最强的隔离级别，通过给事务中每次读取的行加锁，写加写锁，保证不产生幻读问题，但是会导致大量超时以及锁争用问题。

**5、并发控制 与 MVCC**

MVCC (multiple-version-concurrency-control）

它是个**行级锁**的变种， 在**普通读情况下避免了加锁操作，因此开销更低。** 虽然实现不同，但通常都是实现**非阻塞读**，对于**写操作只锁定必要的行。** 

*   **一致性读 （就是读取快照）**select \* from table ....
    
*   **当前读(就是读取实际的持久化的数据)**特殊的读操作，插入/更新/删除操作，属于当前读，处理的都是当前的数据，需要加锁。 select \* from table where ? lock in share mode; select \* from table where ? for update; insert; update ; delete;
    

注意：select ...... from where...... （没有额外加锁后缀）使用MVCC，保证了读快照(MySQL 称为 consistent read)，所谓一致性读或者读快照就是读取当前事务开始之前的数据快照，在这个事务开始之后的更新不会被读到。详细情况下文 select 的详述。

**对于加锁读** **SEL****ECT with** **FOR UPDATE (排他锁) or LOCK IN SHARE MODE (****共****享锁)、 update、delete语句，要考虑是否是唯一索引的等值查询。**

**INNODB 的 MVCC 通常是通过在每行数据后边保存两个隐藏的列来实现(其实是三列，第三列是用于事务回滚，此处略去)，**一个保存了行的创建版本号，另一个保存了行的更新版本号（上一次被更新数据的版本号） 这个版本号是每个事务的版本号，递增的。这样保证了 innodb 对读操作不需要加锁也能保证正确读取数据。

**5.1、MVCC select无锁操作 与 维护版本号**

下边在 MySQL 默认的 Repeatable Read 隔离级别下，具体看看 MVCC 操作：

*   **Select（快照读，所谓读快照就是读取当前事务之前的数据。）：** 
    
    a.**InnoDB 只 select 查找版本号早于当前版本号的数据行**，这样保证了读取的数据要么是在这个事务开始之前就已经 commit 了的（早于当前版本号），要么是在这个事务自身中执行创建操作的数据（等于当前版本号）。
    
    b.查找行的更新版本号要么未定义，要么大于当前的版本号(为了保证事务可以读到老数据)，这样保证了事务读取到在当前事务开始之后未被更新的数据。
    
    注意： 这里的 select 不能有 for update、lock in share 语句。 总之要只返回满足以下条件的行数据，达到了快照读的效果：
    

```
(行创建版本号< =当前版本号 && (行更新版本号==null or 行更新版本号>当前版本号 ) )
```

*   **Insert**
    
    InnoDB为这个事务中新插入的行，保存当前事务版本号的行作为行的行创建版本号。
    
*   **Delete** InnoDB 为每一个删除的行保存当前事务版本号，作为行的删除标记。
    
*   **Update**
    
    将存在两条数据，保持当前版本号作为更新后的数据的新增版本号，同时保存当前版本号作为老数据行的更新版本号。
    

```
当前版本号—写—>新数据行创建版本号 && 当前版本号—写—>老数据更新版本号();
```

**5.2、脏读 vs 幻读 vs 不可重复读**

**脏读：一事务未提交的中间状态的更新数据 被其他会话读取到。** 

当一个事务正在访问数据，并且对数据进行了修改， 而这种修改还没有 提交到数据库中(commit 未执行)， 这时，另外会话也访问这个数据，因为这个数据是还没有提交， 那么另外一个会话读到的这个数据是脏数据，依据脏数据所做的操作也可能是不正确的。

**不可重复读：简单来说就是在一个事务中读取的数据可能产生变化，ReadCommitted 也称为不可重复读。** 

在同一事务中，多次读取同一数据返回的结果有所不同。 换句话说就是，后续读取可以读到另一会话事务已提交的更新数据。 相反，“可重复读”在同一事务中多次读取数据时，能够保证所读数据一样， 也就是，后续读取不能读到另一会话事务已提交的更新数据。

**幻读：** 会话T1事务中执行一次查询，然后会话T2新插入一行记录，**这行记录恰好可以满足T1所使用的查询的条件。** 然后T1又使用相同 的查询再次对表进行检索，但是此时却看到了事务T2刚才插入的新行。这个新行就称为“幻像”，因为对T1来说这一行就像突然 出现的一样。innoDB 的 RR 级别无法做到完全避免幻读，下文详细分析。

![](https://mmbiz.qpic.cn/mmbiz_jpg/4g5IMGibSxt71YNMJva8uj3gbxJica5QBH4QEhKluxzMwKiblPp3ybLJJEwI5yhzWLTEYlR6bco8ycqvy8ibusJsAw/640?wx_fmt=jpeg)

**5.3、 如何保证 rr 级别绝对不产生幻读？**  

在使用的 select ...where 语句中加入 for update (排他锁) 或者 lock in share mode (共享锁)语句来实现。**其实就是锁住了可能造成幻读的数据，阻止数据的写入操作。** 

其实是因为数据的写入操作(insert 、update)需要先获取写锁，由于可能产生幻读的部分，已经获取到了某种锁，所以要在另外一个会话中获取写锁的前提是当前会话中释放所有因加锁语句产生的锁。

**5.4、 从另一个角度看锁：显式锁、隐式锁**

**隐式锁**：我们上文说的锁都属于不需要额外语句加锁的隐式锁。

**显示锁：** 

```
SELECT ... LOCK IN SHARE MODE(加共享锁);SELECT ... FOR UPDATE(加排他锁);
```

详情上文已经说过。

**5.5、查看锁情况**

通过如下 sql 可以查看等待锁的情况

```
select * from information_schema.innodb_trx where trx_state="lock wait";或show engine innodb status;
```

**6、MySQL 死锁问题**

死锁，就是产生了循环等待链条，我等待你的资源，你却等待我的资源，我们都相互等待，谁也不释放自己占有的资源，导致无线等待下去。 比如：

```
//Session ASTART TRANSACTION;UPDATE account SET p_money=p_money-100 WHERE p_name="tim";UPDATE account SET p_money=p_money+100 WHERE p_name="bill";COMMIT;//Thread BSTART TRANSACTION;UPDATE account SET p_money=p_money+100 WHERE p_name="bill";UPDATE account SET p_money=p_money-100 WHERE p_name="tim";COMMIT;
```

当线程A执行到第一条语句_UPDATE account SET p\_money=p\_money-100 WHERE p\_name="tim"_;锁定了_p\_name="tim" _的行数据；并且试图获取 _p\_name="bill" _的数据；

此时，恰好，线程B也执行到第一条语句：_UPDATE account SET p\_money=p\_money+100 WHERE p\_name="bill"_;锁定了 _p\_name="bill"_的数据，同时试图获取 _p\_name="tim" _的数据；

此时，两个线程就进入了死锁，谁也无法获取自己想要获取的资源，进入无线等待中，直到超时！

_**innodb\_lock\_wait\_timeout**_**等待锁超时回滚事务：** 

直观方法是在两个事务相互等待时，当一个等待时间超过设置的某一阀值时，对其中一个事务进行回滚，另一个事务就能继续执行。

这种方法简单有效，在i nnodb 中，参数 _innodb\_lock\_wait\_timeout _用来设置超时时间。

_**wait-for graph**_**算法来主动进行死锁检测：** innodb 还提供了 _wait-for graph_ 算法来主动进行死锁检测，每当加锁请求无法立即满足需要并进入等待时，_wait-for graph_ 算法都会被触发。

**6.1、如何尽可能避免死锁**

*   以固定的顺序访问表和行。比如两个更新数据的事务，事务A 更新数据的顺序 为1，2；事务B更新数据的顺序为2，1。这样更可能会造成死锁；
    
*   大事务拆小。大事务更倾向于死锁，如果业务允许，将大事务拆小；
    
*   在同一个事务中，尽可能做到一次锁定所需要的所有资源，减少死锁概率；
    
*    降低隔离级别。如果业务允许，将隔离级别调低也是较好的选择，比如将隔离级别从RR调整为RC，可以避免掉很多因为gap锁造成的死锁。（我司 MySQL 规范做法）；
    
*   为表添加合理的索引。可以看到如果不走索引将会为表的每一行记录添加上锁，死锁的概率大大增大。
    

* * *

延伸阅读：  

*    MySQL官网参考文档：
    
    [https://dev.mysql.com/doc/refman/5.7/en/innodb-locking.html](https://dev.mysql.com/doc/refman/5.7/en/innodb-locking.html)
    
*   本文写于2016年
    

![](https://mmbiz.qpic.cn/mmbiz_jpg/4g5IMGibSxt4BP1d5Xk4NjGB5TtlONnxWXiaiapmuEP3k27kFJWvpm6JNqwIW7S13ZiazhPPVuZdDs5vqZtghpQt2A/640?wx_fmt=jpeg)