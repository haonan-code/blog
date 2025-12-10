+++
date = '2025-12-09T21:00:19+08:00'
title = 'MySQL 常见问题'
categories = ["技术"] 
tags = ["Hugo", "博客搭建"]
description = "用于列表页显示的摘要"
featured_image = "mysql.jpg"
+++

## MySQL 数据库
### MySQL 查询语句的书写顺序 ?
MYSQL 基本查询的书写顺序主要包括几个关键词, 分别是 :
**select** [distinct] <字段名称> **from** 表名 **where** <查询条件> **group by** <字段> **having** <分组后筛选条件> **order by** 排序字段 **limit** 查询起始行号 , 查询数量

**MYSQL** **去重的方式有哪些 ?**
- distinct : 对结果集去重
- group by : 根据某些字段分组 , 该字段在结果集中只会出现一次
**where 和 having 区别 ?**
- where : 对分组之前的数据进行筛选
- having : 对分组之后的数据进行筛选

### MySQL查询语句的执行顺序 ?
MYSQL查询语句的执行顺序和书写顺序并不一致 , 查询顺序为 :
select [distinct] <字段名称> from where <查询条件> group by <分组字段> having <分组后筛选条件> order by 排序字段 limit 查询起始行号 , 查询数量
**执行顺序是 :**
**from 表名** **where** <查询条件> **group by** <分组字段> **having** <分组后筛选条件> **select [distinct]** <字段名称> **order by** 排序字段 **limit** 查询起始行号 , 查询数量

### MySQL中 CHAR 和 VARCHAR 的区别有哪些？
1. char的长度是不可变的，没有装满会使用空格填充到指定长度大小，而varchar的长度是可变的。
	char(10) : 'hello ' varchar(10) 'hello'
2. char的存储方式是：对英文字符（ASCII）占用1个字节，对一个汉字占用两个字节。varchar的存储方式是：对每个英文字符占用2个字节，汉字也占用2个字节

### 如何删除一张表的所有数据
MYSQL删除一张表的所有数据有三种方式 , 分别是
1. **delete from 表名 :** 将Mysql表中的数据一行一行的删除，不删除表的结构，也不释放表的空间，可以回滚（rollback）
2. **drop table 表名 :** 将表的数据直接删除，以及删除表的结构同时释放空间，删除数据后无法找回
3. **truncate table 表名 :** 删除表中的所有数据，释放空间，但是保留表的结构 , 删除数据后不可以回滚。

### MySQL常用的函数有哪些 ?
常用的数值处理函数 :

常用的字符串处理函数 :

常用的时间日期处理函数 :

常用的流程处理函数 :

**MYSQL常用开窗函数/分析函数**








## 数据库事务
### 什么是事物
事务就是用户定义的一系列数据库操作，这些操作可以视为一个完整的逻辑处理工作单元，要么全部执行成功，要么全部执行失败，是不可分割的工作单元。

### 说一说事务的ACID特性 ?
原子性（Atomicity）：事务是不可分割的最小单位，所有操作要么全部执行成功，要么全部不执行回滚，不存在中间状态。
一致性（Consistency）：事务完成时，必须使所有的数据都保持一致状态。
隔离性（Isolation）：并发执行的事务之间互不干扰，一个事务的执行不能被其他事务的中间状态影响，就像事务是独立串行执行一样。
持久性（Durability）：事务一旦提交或回滚，它对数据库中的数据的改变就是永久的。

### 不考虑事务会引发什么问题 ?
**赃读**：一个事务读到另外一个事务还没有提交的数据。
**不可重复读**：一个事务先后读取同一条记录，但两次读取的数据不同，称之为不可重复读。
**幻读**：一个事务按照条件查询数据时，没有对应的数据行，但是在插入数据时，又发现这行数据已经存在，好像出现了 "幻影"。

### 事务的隔离级别有哪些
**read uncommitted** : 读取尚未提交的数据 ：哪个问题都不能解决
**read committed**：读取已经提交的数据 ：可以解决脏读  ---- oracle、sql server、postgresql 默认的
**repeatable read**：重读读取：可以解决脏读 和 不可重复读  ---mysql默认的
**serializable：**串行化：可以解决 脏读 不可重复读 和 虚读 ---相当于锁表

### 事务隔离级别是怎么实现的
隔离级别的实现主要有读写锁和MVCC( Multi-Version Concurrency Control )多版本并发处理方式。

- 读未提交，它是性能最好，也可以说它是最野蛮的方式，因为它压根儿就不加锁，所以根本谈不上什么隔离效果，可以理解为没有隔离。
- 串行化。读的时候加共享锁，也就是其他事务可以并发读，但是不能写。写的时候加排它锁，其他事务不能并发写也不能并发读。
- 读已提交在读已提交隔离级别下，在事务中每一次执行快照读时生成ReadView。多个事物有多个ReadView , 事务没有结束之前只能读取自己的ReadView , 实现了读已提交 , 不能读取到其他事物未提交的数据
- 可重复读在可重复读隔离级别下，仅在事务中第一次执行快照读时生成ReadView，后续复用该ReadView。 而**Repeatable Read** 是可重复读，在一个事务中，执行两次相同的select语句，查询到的结果是一样的

### 说一说数据库表锁和行锁 ?
**表级锁 :** 每次操作锁住整张表。锁定粒度大，发生锁冲突的概率最高，并发度最低。应用在MyISAM、InnoDB等存储引擎中
表级锁又分 : **表锁** , **元数据锁** , **意向锁**
- 表锁 : 表共享读锁, 表独占写锁
- 加锁语法 : `lock tables 表名... read/write` , 读读共享, 读写互斥 , 写写互斥
- 释放锁：`unlock tables`
- 元数据锁 : 元数据锁锁主要作用是维护表元数据的数据一致性，在表上有活动事务的时候，不可以对元数据进行写入操作 , 在访问一张表的时候会自动加上。
- 意向锁 : 在获取到一张表中某一条数据的行锁的时候, 会自动为该表加上意向锁

**行级锁 :** 每次操作锁住对应的行数据。锁定粒度最小，发生锁冲突的概率最低，并发度最高 , 行锁是对索引加锁
行级锁主要有三类 : **行锁** , **间隙锁**, **临键锁**
- 行锁（Record Lock）：锁定单个行记录的锁，防止其他事务对此行进行update和delete , 在RC、RR隔离级别下都支持行锁 , 防止出现脏读
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250622115230399.png)
加锁语法 :

- 读锁 : `SELECT ... LOCK IN SHARE MODE`
- 写锁 : `SELECT ... FOR UPDATE`

- 间隙锁（Gap Lock）：锁定索引记录间隙（不含该记录），确保索引记录间隙不变，在RR隔离级下支持间隙锁 , 防止其他事务在这个间隙进行insert，产生幻读
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250622115423385.png)
- 临键锁（Next-Key Lock）：行锁和间隙锁组合，同时锁住数据，并锁住数据前面的间隙Gap ,在RR隔离级下支持间隙锁 , 防止其他事务在这个间隙进行insert，产生幻读
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250622115513088.png)
查询加锁情况 :
```sql
select object_schema,object_name,index_name,lock_type,lock_mode,lock_data from performance_schema.data_locks;
```

### 什么是死锁 , 如何避免死锁
死锁是指两个或两个以上的线程在执行过程中,因争夺资源而造成的一种互相等待的现象这就是死锁
**例如 :** MYSQL中 事物A获取了一部分数据的锁(10-20) , 事物B也获取一部分数据的锁(20-30)
这个时候事物A想获取id=25这条数据的锁 , 因为这条数据的锁被事物B持有就会阻塞程序
事物B这个时候想获取id=15这条数据的锁 , 因为这条数据的锁被事物A持久, 也会进行等待
这个时候事物A等事物B , 事物B等事物A , 就产生了死锁 , 出现死锁之后MYSQL会报一个异常
```sql
CREATE TABLE `emp`  (
  `id` int NOT NULL AUTO_INCREMENT COMMENT 'ID',
  `name` varchar(50) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '姓名',
  `age` int NULL DEFAULT NULL COMMENT '年龄',
  `job` varchar(20) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '职位',
  `salary` int NULL DEFAULT NULL COMMENT '薪资',
  `entrydate` date NULL DEFAULT NULL COMMENT '入职时间',
  `managerid` int NULL DEFAULT NULL COMMENT '直属领导ID',
  `dept_id` int NULL DEFAULT NULL COMMENT '部门ID',
  `gender` char(1) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '性别',
  `workaddress` varchar(50) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '工作城市',
  PRIMARY KEY (`id`) USING BTREE,
  INDEX `index_age`(`age` ASC) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 39 CHARACTER SET = utf8 COLLATE = utf8_general_ci COMMENT = '员工表' ROW_FORMAT = Dynamic;

-- ----------------------------
-- Records of emp
-- ----------------------------
INSERT INTO `emp` VALUES (1, '金庸', 66, '总裁', 20000, '2000-01-01', NULL, 5, '男', '北京');
INSERT INTO `emp` VALUES (2, '郭靖', 20, '项目经理', 12500, '2005-12-05', 1, 1, '男', '上海');
INSERT INTO `emp` VALUES (3, '杨逍', 33, '开发', 8400, '2000-11-03', 2, 1, '男', '长沙');
INSERT INTO `emp` VALUES (4, '韦一笑', 48, '开发', 11000, '2002-02-05', 2, 1, '男', '北京');
INSERT INTO `emp` VALUES (5, '常遇春', 43, '开发', 10500, '2004-09-07', 3, 1, '男', '长沙');
INSERT INTO `emp` VALUES (6, '小昭', 19, '程序员鼓励师', 6600, '2004-10-12', 2, 1, '女', '武汉');
INSERT INTO `emp` VALUES (8, '周芷若', 19, '会计', 48000, '2006-06-02', 7, 3, '女', '北京');
INSERT INTO `emp` VALUES (9, '丁敏君', 23, '出纳', 3000, '2009-05-13', 7, 3, '女', '北京');
INSERT INTO `emp` VALUES (10, '郭靖', 20, '市场部总监', 12500, '2004-10-12', 1, 2, '女', '武汉');
INSERT INTO `emp` VALUES (12, '杨过', 16, '职员', 6000, '2023-04-02', 0, 1, '男', '湖北武汉');
INSERT INTO `emp` VALUES (13, '方东白', 19, '职员', 5500, '2009-02-12', 10, 2, '男', '上海');
INSERT INTO `emp` VALUES (14, '张三丰', 88, '销售总监', 14000, '2004-10-12', 1, 4, '男', '杭州');
INSERT INTO `emp` VALUES (16, '宋远桥', 40, '销售', 4600, '2004-10-12', 14, 4, '男', '上海');
INSERT INTO `emp` VALUES (17, '陈友谅', 42, NULL, 2000, '2011-10-12', 1, NULL, '男', '杭州');
INSERT INTO `emp` VALUES (23, '俞莲舟', 38, '销售', 4600, '2004-10-12', 14, 4, '男', '杭州');
INSERT INTO `emp` VALUES (25, '鹤笔翁', 19, '职员', 3750, '2007-05-09', 10, 2, '男', '武汉');
INSERT INTO `emp` VALUES (28, '鹿杖客', 56, '职员', 3750, '2006-10-03', 10, 2, '男', '武汉');
INSERT INTO `emp` VALUES (30, '杨康', 16, '职员', 6500, '2023-04-03', 1, 1, '男', '湖北武汉');
```

> 死锁演示 :
1. 事物A : `start transaction ;`
2. 事物B : `start transaction ;`
3. 事物A执行语句 : `update emp set salary = 3000 where age= 23;`
4. 事物B执行语句 : `update emp set salary = 7000 where age= 30;`
5. 事物A执行语句 :
`insert into emp value (35,'小龙女',25,'职员',6500,'2023-04-03',1,1,'女','湖北武汉');`
6. 事物B执行语句 :
`insert into emp value (38,'小红',21,'职员',6500,'2023-04-03',1,1,'女','湖北武汉');`
这个时候就出现了死锁 , 控制台报异常
Deadlock found when trying to get lock; try restarting transaction

**数据库的死锁的发生通常由以下原因导致：**

1. 多个事务同时请求相同的资源，但是它们请求资源的顺序不同，导致互相等待。
2. 事务在操作过程中，没有按照相同的顺序获取锁，导致互相等待。
3. 操作的数据量过大，在持有锁的同时，又请求获取更多的锁，导致互相等待。

**解决死锁的方法有：**

1. 减少锁的数量：比如使用RC来代替RR来避免因为gap锁和next-key锁而带来的死锁情况。
2. 减少锁的时长：加快事务的执行速度，降低执行时间，也能减少死锁发生的概率。
3. 固定顺序访问数据：事务在访问同一张表时，应该以相同的顺序获取锁，这样可以避免死锁的发生。
4. 减少操作的数据量：尽量减少事务操作的数据量，尽量减少事务的持有时间，这样可以降低死锁的发生几率。


### 什么是`redolog` , 有什么用 ?
### 什么是`undo log`, 有什么用 ?
### 什么是`binlobg`, 有什么用 ?
### 有没有了解过 MVCC , 说一说你的理解

## 分库分表
