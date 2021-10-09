---
tags: 数据库
mathjax: true 
title: 数据库/MySQL
---

> MySQL，最流行的关系型数据库管理系统。

<!--MORE-->

## 数据库基本操作

```mysql
show database;  #查看现有的数据库
create database bjpowernode; #创建数据库
use bjpowernode;  #指定当前缺省数据库
select database;  #查看当前使用的库
drop database bjpowernode; #删除数据库
```

## 数据表基本操作

```mysql
show tables;  #查看当前库中的表
show tables from <database name>;  #查看其他库中的表
desc <table name>;  #查看表的结构

#创建表,create table,表名在数据库当中一般建议以t_或者tbl_开始。
create table t_student(
    no bigint,
    name varchar(255),
    sex char(1),
    classno varchar(255),
    birth char(10)
);
# 将查询结果当做表创建出来。		
create table 表名 as select语句;
		
#删除数据表 DROP TABLE		
DROP TABLE if exists t_student; 

#插入数据 insert into
insert into t_student
	(no,name,sex,classno,birth) 
values
	(3,'rose','1','gaosi2ban','1952-12-14'),
	(4,'laotie','1','gaosi2ban','1955-12-14');
# 除name字段之外，剩下的所有字段自动插入NULL	
insert into t_student(name) values('wangwu'); 
# 字段可以省略不写，但是后面的value对数量和顺序都有要求。
insert into t_student values(1,'jack','0','gaosan2ban','1986-10-23');
# 将查询结果插入到一张表中
insert into dept1 select...

#修改数据，update...set...，没有条件整张表数据全部更新。
update 表名 set 字段名1=值1,字段名2=值2... where 条件;
update dept1 set loc = 'SHANGHAI', dname = 'RENSHIBU' where deptno = 10;

#删除数据，delete from，没有条件全部删除。
delete from 表名;
delete from 表名 where 条件;
delete from dept1 where deptno = 10;
```

> MySQL当中字段的常见数据类型：
> 		int		整数型(java中的int)
> 		bigint	长整型(java中的long)
> 		float		浮点型(java中的float double)
> 		char		定长字符串(String)
> 		varchar	可变长字符串(StringBuffer/StringBuilder)
> 		date		日期类型 （对应Java中的java.sql.Date类型）
> 		BLOB		二进制大对象（存储图片、视频等流媒体信息） Binary Large OBject （对应java中的Object）
> 		CLOB		字符大对象（存储较大文本，比如，可以存储4G的字符串。） Character Large OBject（对应java中的Object）

> char和varchar？
> 当某个字段中的数据长度不发生改变的时候，是定长的，例如：性别、生日等采用char。当一个字段的数据长度不确定，例如：简介、姓名等用varchar。

```mysql
#查

	select		5
		...
	from			1
		...		
	where			2
		...	
	group by		3
		...
	having		4
		...
	order by		6
		...
	limit			7
		...;

select empno, ename from emp;
select * from emp;
select empno, ename, sal*12 from emp; #select语句中可以使用运算符
select empno as ‘员工编号’, ename as ‘员工姓名’, sal*12 as ‘年薪’ from emp;
SELECT * from runoob_tbl  WHERE runoob_author LIKE '%COM'; # LIKE 子句中使用 % 字符来表示任意字符,类似正则表达式中 *
select distinct deptno,job from emp; # distinct关键字去除重复记录。distinct只能出现在所有字段的最前面。

#条件查询where
select empno, ename, sal from emp where sal=5000;
select empno, ename from emp where job="manager"; #字符串加" "或' ',不区分大小写

#排序，系统默认由小到大 order by
select * from emp order by sal;
select * from emp order by job,sal; #按照多个字段排序，首先按照job排序，再按照sal排序
select * from emp order by sal asc/desc; #小到大/大到小

#union，将查询结果集相加
select ename,job from emp where job = 'MANAGER'
union
select ename,job from emp where job = 'SALESMAN';

#分组,对分组之后的数据进行再次过滤 group by...having...
#分组函数/多行处理函数/聚合函数，count sum avg max min
#分组函数不可直接使用在where子句当中,在group by语句执行结束之后才会执行,一般用在select子句中
# 当一条语句中有group by的话，select后面只能跟分组函数和参与分组的字段。

#连接查询
#语法：
		...
			A
		join
			B
		on
			连接条件
		where
			...
# limit startIndex, length,分页查询,取结果集中的部分数据,从0开始，0表示第一条数据
#取出工资前5名的员工
select ename,sal from emp order by sal desc limit 0, 5;
select ename,sal from emp order by sal desc limit 5;
#找出工资排名在第4到第9名的员工
select ename,sal from emp order by sal desc limit 3,6;
#通用的标准分页sql,每页显示3条记录,pageSize = 3,pageNo是显示第几页
limit (pageNo - 1) * pageSize, pageSize
```

## 连接查询

> 笛卡尔积现象，也称交叉连接。返回左表中的每一行与右表中的所有行组合。

### 内连接（等值连接）

组合两个表中的记录，返回关联字段相符的记录，也就是返回两个表的交集部分。**显示部分数据**。

### 外连接

以主表为基准(将主表的数据全部显示)，从表显示与主表对应的数据，如果对应的没有，则以null补齐。即**结果全部显示**。

#### 左

```mysql
SELECT * 
FROM a_table a 
left join b_table b # 左的左是左表
ON a.a_id = b.b_id;
```

左表（主）(a_table)的记录将会全部表示出来，而右表(b_table)只会显示符合搜索条件的记录。右表记录不足的地方均为NULL。

#### 右

```mysql
SELECT * 
FROM a_table a 
right outer join b_table b # 右的右是右表
on a.a_id = b.b_id;
```

右表（主）(b_table)的记录将会全部表示出来,左表(a_table)只会显示符合搜索条件的记录。左表记录不足的地方均为NULL。

#### 全连接

全连接=左连接+右连接，`full join`

## 约束

非空约束(not null)：约束的字段不能为NULL

```mysql
create table t_user(
    id int,
    username varchar(255) not null,
    password varchar(255)
);
# not null约束只有列级约束。没有表级约束。
```

唯一约束(unique)：约束的字段不能重复，可以为NULL。

```mysql
create table t_user(
    id int,
    username varchar(255) unique # 列级约束
);

create table t_user(
    id int, 
    usercode varchar(255),
    username varchar(255),
    unique(usercode,username) # 多个字段联合起来添加1个约束unique 【表级约束】
);
```

### 主键约束(primary key)

主键值是这行记录在这张表当中的唯一标识。约束的字段**既不能为NULL，也不能重复**（简称PK）。一张表的主键约束只能有1个。第一范式要求任何一张表都应该有主键。

**主键的分类**

根据主键字段的字段数量来划分：
			单一主键（推荐的，常用的。）
			复合主键（多个字段联合起来添加一个主键约束）（复合主键不建议使用，因为违背三范式。）

根据主键性质来划分：
			自然主键：**主键值最好就是一个和业务没有任何关系的自然数。**（这种方式是推荐的）
			业务主键：主键值和系统的业务挂钩，例如：拿着银行卡的卡号做主键，拿着身份证号码作为主键。（不推荐用）最好不要拿着和业务挂钩的字段作为主键。因为以后的业务一旦发生改变的时候，主键值可能也需要随着发生变化，但有的时候没有办法变化，因为变化可能会导致主键值重复。

```mysql
# 建表时添加主键约束
create table t_user(
    id int primary key,
    username varchar(255),
    email varchar(255)
);
		
#mysql提供主键值自增
drop table if exists t_user;
create table t_user(
    id int primary key auto_increment, # id字段自动维护一个自增的数字，从1开始，以1递增。
    username varchar(255)
);
```

### 外键约束(foreign key)

（简称FK）外键值可以为NULL。外键字段引用其他表的某个字段的时候，被引用的字段不一定是主键，但至少具有unique约束。

```mysql
create table t_class(
    cno int,
    cname varchar(255),
    primary key(cno)
);

create table t_student(
    sno int,
    sname varchar(255),
    classno int,
    primary key(sno),
    foreign key(classno) references t_class(cno)
);
# t_student中的classno字段引用t_class表中的cno字段，此时t_class表叫做父表。t_student表叫做子表。
```

## 事务（Transaction）

一个事务是一个完整的业务逻辑单元，不可再分。事务的存在是为了保证数据的完整性，安全性。和事务相关的语句只有：DML语句。（insert delete update）

> 比如：银行账户转账，从A账户向B账户转账10000.需要执行两条update语句：
> 		update t_act set balance = balance - 10000 where actno = 'act-001';
> 		update t_act set balance = balance + 10000 where actno = 'act-002';
>
> 以上两条DML语句必须同时成功，或者同时失败，不允许出现一条成功，一条失败。
>
> 要想保证以上的两条DML语句同时成功或者同时失败，那么就需要使用数据库的“事务机制”。

### 事务包括四大特性：ACID

- A: 原子性：事务是最小的工作单元，不可再分。
- C: 一致性：事务必须保证多条DML语句同时成功或者同时失败。
- I：隔离性：事务A与事务B之间具有隔离。	
- D：持久性：持久性说的是最终数据必须持久化到硬盘文件中，事务才算成功的结束。

### 事务隔离性存在隔离级别

理论上隔离级别包括4个

- 第一级别：读未提交（read uncommitted）对方事务还没有提交，我们当前事务可以读取到对方未提交的数据。

读未提交存在脏读（Dirty Read）现象：表示读到了脏的数据。

- 第二级别：读已提交（read committed）对方事务提交之后的数据我方可以读取到。

解决了: 脏读。
存在的问题是：不可重复读。

- 第三级别：可重复读（repeatable read）

解决了：不可重复读。
存在的问题是：幻读，读取到的数据是幻象。

- 第四级别：序列化读/串行化读（serializable） 

解决了所有问题。效率低。需要事务排队。

> oracle数据库默认的隔离级别是：读已提交。
> mysql数据库默认的隔离级别是：可重复读。

mysql事务默认情况下是自动提交，只要执行任意一条DML语句则提交一次。

关闭自动提交：`start transaction;`提交语句：`commit;`回滚：`rollback;`

## 锁

### 乐观锁和悲观锁(逻辑上的锁)

- 乐观锁：对于数据冲突保持一种乐观态度，操作数据时不会对操作的数据进行加锁，只有到数据提交的时候才通过一种机制来验证数据是否存在冲突。

- 悲观锁：对于数据冲突保持一种悲观态度，在修改数据之前把数据锁住，然后再对数据进行读写，在它释放锁之前任何人都不能对其数据进行操作，直到前面一个人把锁释放后下一个人数据加锁才可对数据进行加锁，然后才可以对数据进行操作，一般数据库本身锁的机制都是基于悲观锁的机制实现的。

### 死锁

如何解决数据库死锁

1. 预先检测到死锁的循环依赖，并立即返回一个错误。
2. 当查询的时间达到锁等待超时的设定后放弃锁请求。

## 索引

索引可以大大提高MySQL的提高检索效率。索引也是数据库当中的对象，也需要数据库不断的维护，有维护成本的。比如，经常被修改的数据不适合添加索引，因为数据一旦修改，索引需要重新排序，进行维护。

添加索引是给某一个字段，或者说某些字段添加索引。

创建索引对象：`create index 索引名称 on 表名(字段名);`
删除索引对象：`drop index 索引名称 on 表名;`

添加索引需满足什么条件？

- 数据量庞大。（根据客户的需求，根据线上的环境）

- 该字段很少的DML操作。（因为字段进行修改操作，索引也需要维护）

- 该字段经常出现在where子句中。（经常根据哪个字段查询）

索引底层采用的数据结构是：B+ Tree。详情：[数据结构/多路查找树 | aoif-hikari](https://aoif-hikari.github.io/2021/07/30/多路查找树/#more)

主键和具有unique约束的字段自动会添加索引。根据主键查询效率较高。尽量根据主键检索。
模糊查询的时候，第一个通配符使用的是%，这个时候索引是失效的。

```mysql
select ename from emp where ename like '%A%'; # 索引失效的情况
```

## 存储引擎

建表的时候可以指定存储引擎，也可以指定字符集。mysql默认使用的存储引擎是InnoDB，默认采用的字符集是UTF8。

```mysql
CREATE TABLE t_x(
    id int(11) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8; # 完整的建表语句
```

- MyISAM

MySQL5.1及之前，MyISAM 是默认存储引擎。MyISAM不支持事务，Myisam支持表级锁，不支持行级锁，表不支持外键，该存储引擎存有表的行数，count运算会更快。适合查询频繁，不适合对于增删改要求高的情况

- InnoDB

InnoDB 是 MySQL 的默认事务型引擎，支持事务，表是基于聚簇索引建立的。支持表级锁和行级锁，支持外键，适合数据增删改查都频繁的情况。InnoDB 采用 MVCC 来支持高并发，并且实现了四个标准的隔离级别。其默认级别是 REPEATABLE READ，并通过间隙锁策略防止幻读，间隙锁使 InnoDB 不仅仅锁定查询涉及的行，还会对索引中的间隙进行锁定防止幻行的插入。

## 数据库设计三范式

- 第一范式：任何一张表都应该有主键，并且每一个字段原子性不可再分。


- 第二范式：建立在第一范式的基础之上，所有非主键字段完全依赖主键，不能产生部分依赖。

- 第三范式：建立在第二范式的基础之上，所有非主键字段直接依赖主键，不能产生传递依赖。

在实际的开发中，以满足客户的需求为主，有的时候会拿冗余换执行速度。

## 窗口函数

窗口函数 over (partition by 用于分列的列名 order by 用于排序的列名）；

原则上一般写在**select子句**中。

`over`用来指定函数执行的窗口范围，若后面括号中什么都不写，则意味着窗口包含满足WHERE条件的所有行，窗口函数基于所有行进行计算。如果不为空，则支持以下4中语法来设置窗口。

①window_name：给窗口指定一个别名。如果SQL中涉及的窗口较多，采用别名可以看起来更清晰易读；
②PARTITION BY 子句：窗口按照哪些字段进行分组，窗口函数在不同的分组上分别执行；
③ORDER BY子句：按照哪些字段进行排序，窗口函数将按照排序后的记录顺序进行编号；
④FRAME子句：FRAME是当前分区的一个子集，子句用来定义子集的规则，通常用来作为滑动窗口使用。

- 解决排名问题，e.g.每个班级按成绩排名

序号函数：

`ROW_NUMBER()`：顺序排序——1、2、3
`RANK()`：并列排序，跳过重复序号——1、1、3
`DENSE_RANK()`：并列排序，不跳过重复序号——1、1、2

```mysql
SELECT stu_id,
       ROW_NUMBER() OVER (PARTITION BY stu_id ORDER BY score DESC) AS score_order,       
FROM ...

# 为窗口设置别名
SELECT ROW_NUMBER() OVER w AS rk,
       stu_id, lesson_id, score
FROM ...
WHERE ...
WINDOW w AS (PARTITION BY stu_id ORDER BY score)
```

- 解决TOP N问题，e.g.每个班级前两名的学生

```mysql
#查询每个学生成绩最高的两个科目
SELECT *
FROM 
	(SELECT*,row_number() over (PARTITION BY 姓名 ORDER BY 成绩 DESC) AS ranking 
     FROM test1) 
     AS newtest
WHERE ranking<=2;
```

- **聚合函数作为窗口函数**

**作用**：聚合函数作为窗口函数，是起到"**累加/累计**"的效果，比如，就是截止到本行，最大值？最小值是多少

**与专用窗口函数的区别**：括号中需要有指定列，不能为空
