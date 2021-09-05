```sql
desc student //查看student表的结构
show create database jdbc //查看创建jdbc数据库的语句
show create table people //查看创建people表的语句
```

```sql
/*关于数据库引擎
INNODB 默认使用
MYISAM 早些年使用
*/
```

|              | MYISAM | INNODB         |
| ------------ | ------ | -------------- |
| 事务支持     | 不支持 | 支持           |
| 数据行锁定   | 不支持 | 支持           |
| 外键约束     | 不支持 | 支持           |
| 全文索引     | 支持   | 不支持         |
| 表空间的大小 | 较小   | 较大，约为两倍 |

常规使用操作：

MYISAM:节约空间 速度较快

INNODB:安全性高，事务的处理，多表多用户操作

```
设置数据库表的字符编码
```

```sql
CHARSET=utf-8
```

不设置的话，会是mysql默认的字符集编码Latin1

## 1、**修改和删除**

```sql
--修改表名--
ALTER TABLE student RENAME AS student1
--增加表的字段 ALTER TABLE 表名 ADD 字段名 列属性
ALTER TABLE student ADD age INT(2)
--修改表的字段(重命名，修改约束！)
--ALTER TABLE 表名 MODIFY 字段名 列属性[]
ALTER TABLE student MODIFY age VARCHAR(11) --修改约束
--ALTER TABLE 表名 CHANGE 旧字段名 新字段名 列属性
ALTER TABLE student CHANGE age age1 INT(11) --修改字段名
--删除表的字段
ALTER TABLE student DROP age --删除student表的字段age
--删除表
DROP TABLE IF EXISTS student --删除student表(如果存在)




```

```sql
-- 学生表的gradeid字段要去引用年级表的gradeid
-- 定义外键key
-- 给这个外键添加约束(执行引用) references引用


create table if not exists `student`(
	`id` int(10) not null auto_increment comment 'ID',
    `name` varchar(10) not null default '匿名' comment '名称',
    `pwd` varchar(20) not null default '123456' comment '密码',
    `sex` varchar(2) not null default '男' comment '性别',
    `birthday` datetime default null comment '出生日期',
    `gradeid` int(10) not null comment '学生的年级',
    `address` varchar(50) default null comment '家庭地址',
    `email` varchar(20) default null comment '邮箱',
    primary key(`id`),
    key `FK_gradeid` (`gradeid`),
    constraint FK_gradeid foreign key(`gradeid`) references grade(`gradeid`)
)engine=InnoDB default charset=utf8

-- 删除有外键关系的表时，必须要先删除引用别人的表(student)，再删除被引用的表(grade)
-- 创建表的时候没有外键关系
alter table `student`
add constraint `FK_gradeid` foreign key(`gradeid`) references `grade`(`gradeid`)(
--alter table `表` add constraint `约束名` foreign key(`作为外键的列`) references `哪个表`(`哪个字段`)
```

以上的操作都是物理外键，数据库级别的外键，我们不建议使用(避免数据库过多造成困扰)

**最佳实践**

数据库就是单纯的表，只用来存数据，只有行(数据)和列(字段)

我们想使用多张表的数据，想使用外键(程序去实现)

# 2、DML语言(全部记住)

##### 数据库意义：数据存储，数据管理

DML语言：数据操作语言

·insert

```sql
-- 插入语句(添加)
-- insert into 表名([字段1,字段2,字段3]) values ('值1','值2','值3')
insert into `student`(`name`,`sex`,`pwd`,`gradeid`) values('李四','男','123456','002')
--注意事项
1,字段和字段之间使用英文逗号隔开
2,字段是可以省略的，但是后面的值必须一一对应
3,可以同时插入多条数据，values后面的值用逗号隔开 values(),()

```



·update

```sql
-- 指定id修改学生的名字
update `student` set `name`='李四' where id=1;
-- 不指定的条件下，会改动所有的表
update `student` set `name`='赵六'
--语法
update 表名 set 列名=value where 条件
--修改多个属性，用逗号隔开

```



·delete

语法：

```sql
delete from 表名 [where 条件]
```

truncate命令

作用：完全清空一个数据库表，表的结构和索引约束不会变

```sql
truncate `student`
```

delete和truncate的区别：

相同点：都能删除数据，都不会删除表结构

不同点：truncate 重新设置自增列，计数器归零

# 3、DQL语言

## 3.1、DQL

所有的查询操作都用它，select

简单的查询，复杂的查询都用到他

**数据库中最核心的语言**

使用频率最高的语言

## 3.2、查询指定字段

```sql
-- 函数 Concat(a,b) 将a，b拼接
select CONCAT('姓名',StudentName) as 新名字 from student

```

```sql
-- 去重
select distinct `studentNo` from result
```

 ```sql
select version() --查询系统版本(函数
select 100*3-1 as 计算结果 --用来计算(表达式)
select @@auto_increment_increment --查询自增的步长(变量)
-- 学生成绩+1查看
select `studentNo`,`studentResult`+1 as `提分后` from result
 ```

数据库中的表达式：文本值，列，Null，函数，计算表达式，系统变量

select 表达式 from 表

## 3.3、where条件子句

作用：检索数据中符合条件的值

搜索的条件由一个或多个表达式组成

逻辑运算符

| 运算符  | 语法               | 描述   |
| ------- | ------------------ | ------ |
| not !   | not a !a           | 逻辑非 |
| and &&  | a and b   a&&b     | 逻辑与 |
| or \|\| | a or b      a\|\|b | 逻辑或 |

```sql
-- 模糊查询(区间)
select `studentno`,`studentresult` from result where `studentresult`between 95 and 100
-- != not
select studentno,studentresult from result where not studentno = 1000
```

模糊查询：比较运算符

| 运算符      | 语法               | 描述                                       |
| ----------- | ------------------ | ------------------------------------------ |
| is null     | a is null          | 如果操作符为null，结果为真                 |
| is not null | a is not null      | 如果操作符不为null，结果为真               |
| between     | a between b and c  | 若a在bc之间，结果为真                      |
| like        | a like b           | sql匹配，如果a匹配b，则结果为真            |
| in          | a in (a1,a2,a3...) | 假设a在a1,a2....其中的某一个值中，结果为真 |

```sql
-- 查询姓刘的同学
-- like结合 %(代表0到任意个字符) _（一个字符）
select `studentno`,`studentname` from student where `studentname` like '刘%'
select `studentno`,`studentname` from student where `studentname` like '刘_'
-- in
-- 查询1001、1002、1003号学院
select `studentno`,`studentname` from student where `studentno` in(1001,1002,1003)
```

## 3.4、联表查询

```sql
-- 查询参加了考试的同学(学号，姓名，科目编号，分数)
-- 思路
1、分许需求，分析查询的字段来自哪些表，(连接查询)
2、确定使用哪些连接查询？七中
确定交叉点(这两个表中哪些数据是相同的)
判断的条件：学生表中的 studentno=成绩表中的 studentno
select s.studentno,studentname,subjectno,studentresult from student as s inner join result as r where s.studentno =  r.studentno
-- right join
select s.studentno,studentname,subjectno,studentresult from student as s right join result as r on s.studentno=r.studentno
-- left join
select s.studentno,studentname,subjectno,studentresult from student as s right join result as r on s.studentno=r.studentno

```

| 操作       | 描述                                         |
| ---------- | -------------------------------------------- |
| inner join | 如果表中至少有一个匹配，就返回行             |
| left join  | 即使右表中没有匹配，也会从左表中返回所有的值 |
| right join | 与left join相反                              |

```sql
-- 查询缺考的同学
select s.studentno,studentname,subjectno,studentresult from student as s left join result as r on s.studentno = r.studentno where studentresult is null

```

自连接

自己的表和自己的表连接，核心：**一张表拆成两张一样的表**

父类

| categoryid | categoryName |
| ---------- | ------------ |
| 2          | 信息技术     |
| 3          | 软件开发     |
| 5          | 美术设计     |
|            |              |
|            |              |

子类

| pid  | categoryid | categoryName |
| ---- | ---------- | ------------ |
| 3    | 4          | 数据库       |
| 2    | 8          | 办公信息     |
| 3    | 6          | web开发      |
| 5    | 7          | 美术设计     |
|      |            |              |

操作：查询父类对应的子类关系

| 父类     | 子类     |
| -------- | -------- |
| 信息技术 | 办公信息 |
| 软件开发 | 数据库   |
| 软件开发 | web开发  |
| 美术设计 | ps技术   |

```sql
-- 查询父子信息:把一张表看为两张一模一样的表
select a.`categoryName` as '父栏目',b.`categoryName` as '子栏目' from `category` as a,`category` as b where a.`categoryid` = b.`pid`

```

## 3.5、分页和排序

```sql
-- ==============分页limit和排序order by============
-- 排序：升序asc， 降序desc
-- 查询的结果根据 成绩降序 排序
select s.`studentno`,`studentname`,`subjectname`,`studentresult` from student as s inner join `result` as r on s.`subjectno` = sub.`subjectno` where `subjectname`='数据结构'
order by studentresult desc	

-- 100万条数据
-- 为什么分页？
-- 缓解数据库压力，给人的体验更好，瀑布流
-- 分页：每页只显示五条数据
-- 语法：limit 起始值，页面的大小
-- 网页应用：当前，总的页数，页面的大小
-- limit 0,5 1-5
-- limit 1,5 2-6
select s.`studentno`,`studentname`,`subjectname`,`studentresult` from student as s inner join `result` as r on s.`subjectno` = sub.`subjectno` where `subjectname`='数据结构'
order by studentresult desc	limit 0,5
-- 第一页 limit 0,5
-- 第二页 limit 5,5
-- 第三页 limit 10,5
```

## 3.6、子查询

```sql
-- 1、查询 数据库结构-1的所有考试结果(学号，科目编号。成绩)，降序排列
-- 方式一：使用连接查询
select `studentno`,r.`subjectno`,`studentresult` from `result` as r
inner join `subject` as sub
on r.subjectno = sub.subjectno
where subjectname = '数据库结构-1'
order by studentresult desc
--方式二：使用子查询(有里到外)
select `studentno`,`subjectno`,`studentresult`
from `result` where studentno=(select subjectno from `subject` where `subjectname`='数据库结构-1') desc
-- 查询所有数据库结构-1的学生学号
select subjectno from `subject` where `subjectname`='数据库结构-1'
```

# 4、MySql函数

## 4.1、常用函数

```sql
-- 数学运算
select abs(-9) --绝对值
select celling(0.4) --向上取整
select floor(9.4) --向下取整
select rand() --返回一个0-1之间的随机数
select sign() --判断一个数的符号，负数返回-1，正数返回1

-- 字符串函数
select char_length() --字符串长度
select concat(a,b) --合并字符串
select insert() --插入，替换
select lower() --转小写
select upper() --转大写
select instr('kuangshen','h') --返回第一次出现的子串的索引
select replace('ab','a','c') --返回cb，替换

-- 时间和日期函数(记住)
select cunrrent_date() --获取当前日期
select curdate() --获取当前日期
select now() --获取当前时间
select localtime() --获取本地时间

--系统
select user()
```



## 4.2、聚合函数(常用)

| 函数名称 | 描述   |
| -------- | ------ |
| count()  | 计数   |
| sum()    | 求和   |
| avg()    | 平均值 |
| max()    | 最大值 |
| min()    | 最小值 |
|          |        |

```sql

```

```sql
--查询一个表中有多少个记录，就是用count()
select count(studentname) from student --count(字段)，会忽略null值
select count(*) from student --count(*) 不会忽略null值 本质 计算行数
select count(1) from result --count(1)，不会或略所有的null值

select sum(`studentresult`) as 总和
select avg(`studentresult`) as 平均分
select max(`studentresult`) as 最高分

```

## 4.3、分组以及过滤

```sql
--查询不同课程的平均分，最高分，最低分,平均分大于80
-- 核心：(根据不同的课程分组)
select subjectname,avg(studentresult) as 平均分,max(studentresult),min(studentresult) from result as r
inner join `subject` as sub
on r.`subjectno` = sub.`studentno`
group by r.subjectno --通过什么字段分组
having 平均分>80
```

# 5、数据库级别的MD5加密(扩展)

什么是MD5?

主要增强算法复杂度和不可逆性

MD5不可逆，具体的值的md5是一样的

MD5破解网站的原理，背后有一个字典，MD5加密后的值，加密前的值

```sql
create table `testmd5`(
	`id` int(4) not null,
  `name` varchar(20) not null,
  `pwd` varchar(100) not null,
  primary_key(`id`)
)engine=innodb default charset=utf8

--明文密码
insert into testmd5 values(1,'张三','123456'),(2,'李四','123456'),(3,'王五','123456')
--加密
update testmd5 set pwd=md5(pwd) where id =1
update testmd5 set pwd=md5(pwd) --加密全部密码
-- 插入的时候加密
insert into testmd5 values(4,'小明',md5('123456'))
--如何校验，将用户传进来的密码，进行md5加密，然后对面加密后的值
select * from testmd5 where `name`='小明' and pwd=mad5('123456')
```

# 6、事务

## 6.1、什么是事务

要么都成功，要么都失败

**事务原则**：ACID原则：原子性，一致性，隔离性，持久性，(脏读，幻读...)

**原子性**：a给b转账200块包含两个步骤：a：800-200=600，b：200+200=400

原子性表示：这个两个步骤要么一起成功，或者让一起失败，不能只发生其中一个动作

**一致性**：还是a给b转账，无论怎么转，最后的值一定是1000，针对一个事务操作前与后的状态一致

**持久性**：表示事务结束后的数据不随着外界原因导致数据丢失，比如a给b转账，转账前(事务还没有提交)服务器宕机或者断电，那么重启后，a还是800，b还是200，如果在转账后(事务提交之后)服务器宕机或者断电，那么重启后，a：600，b：400，事务一旦提交就不可逆

**隔离性**：针对多个用户同时操作，注释派出其他事务对本次事务的影响

**脏读**：指一个事务读取了另一个事务未提交的数据

比如a给b转账200，b变成400，但此时事务未提交，c又给b转了100，b不会变成500而是变成300，这就是隔离性没做好造成的脏读

**不可重复读**：在一个事务内读取表中的某一行数据，多次读取结果不同(这不一定是错误，只是某些场合不对)

**幻读**：是指在一个事务内读取到了别的事务插入的数据，导致前后读取不一致(一般是行影响，多了一行)

```sql
--mysql是默认开启事务自动提交的
set autocommit=0 --关闭
set autocommit=1 --开启(默认的)
-- 手动处理事务

-- 事务开启
start transaction -- 标记一个事务的开始，从这之后的sql语句都在同一个事务中
insert xx
insert xxx
--只要有一个sql失败就都不会提交事务
-- 提交 持久化（成功）
commit
-- 回滚 (失败)
rollback
-- 事务结束
set autocommit=1 --开启自动提交
savepoint 保存点 --设置一个事务的保存点
rollback to savepoint -- 回滚到保存点

-- 转账
create database shop character set utf8 collate utf8_general_ci
use shop
create table `account`(
	`id` int(3) not null auto_increment,
  `name` varchar(30) not null,
  `money` decimal(9,2) not null,
  primary_key(`id`)
)engine=innodb default charset=utf8

insert into `account`(`name`,`money`) values('A',2000.00),('B',10000.00)

```

# 7、索引

Mysql官方对索引的定义为：**索引(index)是帮助Mysql高效获取数据的数据结构**。

提取句子主干，就可以得到索引的本质：索引是数据结构

## 7.1、索引的分类

主键索引：PRIMARY KEY

​	唯一标识：主键不可重复，只能有一个列作为主键	

唯一索引：UNIQE KEY

​	避免重复的列出现，唯一索引可以重复

常规索引：KEY/INDEX

​	默认的,index，key关键字来设置

全文索引：FullText

​	在特定的数据库引擎下才有：myisam

​	快速定位数据

```sql
-- 索引的使用
-- 1、在创建表的时候给字段增加索引
-- 2、创建完毕后，增加索引
-- 显示所有的索引信息
show index from student
-- 增加一个全文索引(索引名)列名
alter table school.student add fulltext index `studentname`(`studentname`);
-- explain 分析sql执行情况
explian select * from student -- 非全文索引
-- 使用全文检索,在SELECT的WHERE字句中用MATCH函数,索引的关键词用AGAINST标识
explain select * from student where match(studentname) aginst('刘');
```

## 7.2、测试索引

```sql
-- id_表名_字段名
-- create index 索引名 on 表(字段名)
create index id_app_user_name on app_user(`name`)
```

索引在小数据量的时候，用处不大，但是在大数据的时候，区别十分明显

## 7.3、索引原则

索引不是越多越好

不要对进程变动数据加索引

小数据量的表不需要加索引

索引一般加在常用来查询的字段上

# 8、权限管理和备份

## 8.1、用户管理

```sql
-- 创建用户 create user 用户名 identitied by '密码'
create user liuhao identified by '123456'
```

三次握手：

​		首先：服务器为LISTEN状态，然后客户端向服务端发送SYN = 1 ,seq = x；发送完成后客户端进入SYN-SENT(同步-已发送)状态。请求连接状态为SYN=1,连接成功状态为SYN=1,ACK=1

​		然后：服务端向客户端发送确认，回复客户端：SYN = 1,ACK = 1, seq = y,ack = x+1。发送完成后，服务端进入SYN-RCVD(同步-已接收)状态。

​		然后：客户端向服务端发送确认，回复服务端：ACK = 1,seq = x+1,ack = y+1，然后客户端进入ESTABLISHED(已建立连接)状态。当服务端收到客户端的确认后，也进入ESTABLISHED状态。

​	四次挥手：

​		首先：服务端和客户端都是ESTABLISHED状态，然后客户端向服务端发送FIN=1(关闭客户端向服务端的数据传输),,seq = u（u为客户端最后向服务端发送的seq+1），然后客户端状态变为FIN-WAIT-1。

​		然后服务端向客户端发送确认： ACK=1,seq=v,ack=u+1，然后服务端进入CLOSE-WAIT状态。当客户端收到服务端的确认后，进入FIN-WAIT-2状态。

​		然后服务端向客户端发送FIN=1(关闭服务端向客户端的数据传输),ACK=1,seq=w(w为服务端最后一次向客户端发送的seq+1)，ack=u+1后客户端进入LAST-ACK状态。

​		然后客户端向服务端发送确认：ACK=1, seq = u+1,ack=w+1，然后客户端进入TIME-WAIT状态，等待2MSL(最长报文段寿命)后进入CLOSED状态；服务端收到客户端的确认后，进入CLOSED状态。