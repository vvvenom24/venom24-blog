---
title: "MySQL 基础"
date: 2018-08-29T15:31:42+08:00
lastmod: 2018-08-29T15:31:42+08:00
draft: false
series: [MySQL]
tags: [MySQL]
summary: "MySQL 基础概念与操作"
---
## 连接数据库
```mysql
mysql -u <user> -p
<password>
```

## 退出连接
```
QUIT 或者 Ctry + D
```

## 操作数据库
```mysql
show databases;
```

- 创建数据库：

```mysql
-- utf8编码
create database db1 DEFAULT CHARSET utf8 COLLATE utf8_general_ci;

-- gbk编码
create database db1 DEFAULT CHARACTER SET gbk COLLATE gbk_chinese_ci;
```

- 使用数据库：

```mysql
use db1;
```

- 显示当前使用的数据库中所有表：

```mysql
SHOW TABLES;
```

## 用户管理

- 创建用户：

```mysql
create user '用户名'@'IP地址' identified by '密码'；
```

- 删除用户：

```mysql
drop user '用户名'@'IP地址';
```

- 修改用户：

```mysql
rename user '用户名'@'IP地址'；
to  '新用户名'@'IP地址'；
```

- 修改密码：

```mysql
set password for '用户名'@'IP地址' = Password('新密码')；
```

**注**：用户权限相关数据保存在 mysql 数据库的 user 表中，所以也可以直接对其进行操作（不建议）

## 权限管理

mysql 对于权限这块有一下限制：

- all privileges：除 grant 外的所有权限

- select：仅查权限

- select,insert：查和插入权限

- usage：无访问权限

- alter：使用 alter table

- alter routine：使用 alter procedure 和 drop procedure

- create：使用 create table

- create routine：使用 create procedure

- create temporary tables：使用 create temporary tables

- create user：使用 create user、drop user、rename user 和 revoke  all privileges

- create view：使用 create view

- delete：使用 delete

- drop：使用 drop table

- execute：使用 call 和存储过程

- file：使用 select into outfile 和 load data infile

- grant option：使用 grant 和 revoke

- index：使用 index

- insert：使用 insert

- lock tables：使用 lock table

- process：使用 show full processlist

- select：使用 select

- show databases：使用 show databases

- show view：使用 show view

- update：使用 update

- reload：使用 flush

- shutdown：使用 mysqladmin shutdown (关闭MySQL)

- super：使用 change master、kill、logs、purge、master和set global。还允许mysqladmin调试登陆

- replication client：服务器位置的访问

- replication slave：由复制从属使用

对于数据库及内部其他权限如下：

- 数据库名.*：数据库中的所有

- 数据库名.表：指定数据库中的某张表

- 数据库名.存储过程：指定数据库中的存储过程

- *.*：所有数据库

对于用户和IP的权限如下：

- 用户名@IP地址：用户只能在改IP下才能访问

- 用户名@192.168.1.%：用户只能在改IP段下才能访问（通配符%表示任意）

- 用户名@%：用户可以在任意IP下访问（默认IP地址为%）

- 查看权限：

```mysql
show grants for '用户'@'IP地址'
```

- 授权：

```mysql
grant 权限 on 数据库.表 to '用户'@'IP地址'
```

- 取消授权：

```mysql
revoke 权限 on 数据库.表 from '用户名'@'IP地址'
```

- 授权实例：

```mysql
grant all privileges on db1.tb1 TO '用户名'@'IP'

grant select on db1.* TO '用户名'@'IP'

grant select,insert on *.* TO '用户名'@'IP'

revoke select on db1.tb1 from '用户名'@'IP'
```

## MySQL 表操作

- 查看表：

```mysql
-- 查看数据库全部表
show tables;

-- 查看表所有内容
select * from 表名;
```

- 创建表：

```mysql
create table 表名(
    列名  类型  是否可以为空，
    列名  类型  是否可以为空
)ENGINE=InnoDB DEFAULT CHARSET=utf8
```

- e.g.

```mysql
CREATE TABLE 'tab1'(
    'nid' int(11) NOT NULL auto_increment,
    'name' varchar(255) DEFAULT zhangyanlin,
    'email' varchar(255),
    PRIMARY KEY('nid')
)ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

> **注**：
> - 默认值，创建列时可以指定默认值，当插入数据时如果未主动设置，则地洞添加默认值
> - 自增，如果某列设置自增列，插入数据时无需设置此列，默认将自增（表中只能有一个自增列）注意：1.对于自增列，必须是索引（含主键）2.对于自增可以设置步长和起始值
> - 主键，一种特殊的唯一索引，不允许有控制，如果主键使用单个列，则它的值必须唯一，如果是多列，则其组合必须唯一。

- 删除表：

```mysql
drop table 表名
```

- 清空表内容：

```mysql
delete from 表名
trucate table 表名
```

- 修改表：

```mysql
-- 添加列：
alter table 表名 add 列名 类型

-- 删除列：   
alter table 表名 drop column 列名

-- 修改列：
alter table 表名 modify column 列名 类型;  -- 类型
alter table 表名 change 原列名 新列名 类型; -- 列名，类型

-- 添加主键：
alter table 表名 add primary key(列名);

-- 删除主键：
alter table 表名 drop primary key;
alter table 表名  modify  列名 int, drop primary key;

-- 添加外键： 
alter table 从表 add constraint 外键名称（形如：FK_从表_主表） foreign key 从表(外键字段) references 主表(主键字段);

-- 删除外键： 
alter table 表名 drop foreign key 外键名称

-- 修改默认值：
ALTER TABLE testalter_tbl ALTER i SET DEFAULT 1000;

-- 删除默认值：
ALTER TABLE testalter_tbl ALTER i DROP DEFAULT;
```

## 基本数据类型
MySQL的数据类型大致分为：数值、时间和字符串

- bit[(M)]

    二进制位(101001)
    
    「m」表示二进制位的长度(1-64)
    
    默认m=1

- tinyint[(m)] [unsigned] [zerofill]

    小整数，数据类型用于保存一些范围的整数数值范围

    有符号：-128 ~ 127

    无符号：0 ~255

    **特别的**：MySQL 中无布尔值，使用 tinyint(1) 构造。

- int[(m)] [unsigned] [zerofill]

    整数，数据类型用于保存一些范围的整数数值范围

    有符号：-2147483648 ～ 2147483647

    无符号：0 ～ 4294967295

    特别的：整数类型中的「m」仅用于显示，对存储范围无限制。例如： int(5)，当插入数据2时，select 时数据显示为：00002

- bigint[(m)] [unsigned] [zerofill]

    大整数,数据类型用于保存一些范围的整数数值范围

    有符号：-9223372036854775808 ~ 9223372036854775807

    无符号：0 ~ 18446744073709551615

- decimal[(m[,d])] [unsigned] [zerofill]

    准确的小数值，「m」是数字总个数（负号不算），「d」是小数点后个数。「m」最大值为65，「d」最大值为30。

    特别的：对于精确数值计算时需要用此类型

    「decaimal」能存储精确值的原因在于其内部按照字符串存储。

- Float[(M,D)] [UNSIGNED] [ZEROFILL]

    单精度浮点数（非准确小数值），「m」是数字总个数，「d」是小数点后个数

    无符号：-3.402823466E+38 to -1.175494351E-38 ， 0

    有符号：0 ， 1.175494351E-38 to 3.402823466E+38

    **数值越大，越不准确**

- DOUBLE[(M,D)] [UNSIGNED] [ZEROFILL]

    双精度浮点数（非准确小数值），「m」是数字总个数，「d」是小数点后个数

    无符号：-1.7976931348623157E+308 to -2.2250738585072014E-308 ， 0 ，2.2250738585072014E-308 to 1.7976931348623157E+308

    有符号：0 ， 2.2250738585072014E-308 to 1.7976931348623157E+308

    **数值越大，越不准确**

- char(m)

    「char」数据类型用于表示固定长度的字符串，可以包含最多达 255 个字符。其中「m」代表字符串的长度

    即使数据小于「m」长度，也会占用「m」长度

- varchar(m)

    「varchar」数据类型用于变长的字符串，可以包含最多达 255 个字符。其中m代表该数据类型所允许保存的字符串的最大长度，只要长度小于该最大值的字符串都可以被保存在该数据类型中

    **注**：虽然「varchar」使用起来较为灵活，但是从整个系统的性能角度来说，「char」数据类型的处理速度更快，有时甚至可以超出「varchar」处理速度的 50%。因此，用户在设计数据库时应当综合考虑各方面的因素，以求达到最佳的平衡

- text

    「text」数据类型用于保存变长的大字符串，可以组多到 65535(2**16 − 1) 个字符

- mediutext

    A TEXT column with a maximum length of 16,777,215 (2**24 − 1) characters

- longtext

    A TEXT column with a maximum length of 4,294,967,295 or 4GB (2**32 − 1) characters

- enum

    枚举类型
    An ENUM column can have a maximum of 65,535 distinct elements. (The practical limit is less than 3000.)

    e.g.
    ```mysql
    CREATE TABLE shirts (
        name VARCHAR(40),
        size ENUM('x-small', 'small', 'medium', 'large', x-large)
    );
    INSERT INTO shirts(name, size) VALUES('dress shirt', 'large'),('t-shirt','medium'),('polo shirt','small');
    ```

- set

    集合类型
    A SET can have a maximum of 64 distinct members
    
    e.g.
    ```mysql
    CREATE TABLE myset(col SET('a', 'b','c', 'd'));
    INSERT INTO myset(col)VALUES('a,d'),('d,a'),("a,d,a"),('a,d,d'), ('d,a,d');
    ```

- DATE

    YYYY-MM-DD（1000-01-01/9999-12-31）

- TIME

    HH:MM:SS（'-838:59:59'/'838:59:59'）

- YEAR

    YYYY（1901/2155）

- DATETIME

    YYYY-MM-DD HH:MM:SS（1000-01-01 00:00:00/9999-12-31 23:59:59    Y）

- TMESTAMP

    YYYYMMDD HHMMSS（1970-01-01 00:00:00/2037 年某时）

## MySQL 表内容操作

- 增

    insert into 表(列名,列名...) values(值,值...)

    insert into 表(列名,列名...) values(值,值...),(值,值,值...)

    insert into 表(列名,列名...) select(列名,列名...) from 表

    e.g.
    ```mysql
    insert into tab1(name,email) values('zhangsan','zhangsan@111.com');
    ```
- 删

    ```mysql
    -- 删除表里全部内容
    delete from 表;

    -- 删除 ID =1 和name='zhangsan' 那一行数据
    delete from 表 where id=1 and name='zhangsan';
    ```

- 改

    ```mysql
    update 表 set name='zhangsan' where id > 1;
    ```

- 查

    ```mysql
    select * from 表;

    select * from 表 where id > 1;
    
    select nid,name,gender as gg from 表 where id > 1;
    ```

- 条件判断where

    ```mysql
    select * from 表 where id > 1 and name != 'zhangsan' and num=12;
    
    select * from 表 where id between 5 and 16;
    
    select * from 表 where id in (11,22,33);
    
    select * from 表 where id not in (11,22,33);
    
    select * from 表 where id in (select nid from 表);
    ```

- 通配符like

    ```mysql
    -- zhang 开头的所有（多个字符串）
    select * from 表 where name like 'zhang%';

    -- zhang 开头的所有（一个字符）
    select * from 表 where name like 'zhang_';
    ```

- 限制limit

    ```mysql
    -- 前 5 行
    select * from 表 limit 5;

    -- 从第 4 行到第 5 行
    select * from 表 limit 4,5;

    -- 从第 4 行到第 5 行
    select * from 表 limit 5 offset 4;
    ```

- 排序 asc, desc

    ```mysql
    -- 根据「列」从小到大排列
    select * from 表 order by 列 asc;

    -- 根据「列」从大到小排列
    select * from 表 order by 列 desc;

    -- 根据「列1」从大到小排列，如果相同则按「列2」从小到大排列
    select * from 表 order by 列1 desc,列2 asc;
    ```

- 分组 group by

    ```mysql
    select num from 表 group by num
    
    select num,nid from 表 group by num,nid
    
    select num,nid from 表 where nid > 10 group by num,nid order nid desc
    
    select num,nid,count(*),sum(score),max(score),min(score) from 表 group by num,nid
    
    select num from 表 group by num having max(id) > 10
    ```

    **特别的**：group by 必须在 where 之后，order by 之前

## 视图 view
视图是一个`虚拟表`，其内容由查询定义。同真实的表一样，视图包含一系列带有名称的列和行数据。但是，视图并不在数据库中以存储的数据值集形式存在。行和列数据来自由定义视图的查询所引用的表，并且在引用视图时动态生成。对其中所引用的基础表来说，视图的作用类似于筛选。定义视图的筛选可以来自当前或其它数据库的一个或多个表，或者其它视图。通过视图进行查询没有任何限制，通过它们进行数据修改时的限制也很少。`视图是存储在数据库中的查询的SQL语句`，它主要出于两种原因：安全原因，视图可以隐藏一些数据。

## 创建视图

CREATE VIEW 视图名称 AS SQL语句

e.g.
```mysql
CREATE VIEW v1 AS SELECT nid, name FROM tab1 WHERE nid > 4
```

## 删除视图

DROP VIEW 视图名称

## 修改视图

ALTER VIEW 视图名称 AS SQL语句

e.g.
```mysql
ALTER VIEW v1 AS
SELECT A.nid,B.NAME FROM tab1
LEFT JOIN B ON A.id = B.nid
LETF JOIN C on A.id = C.nid
WHERE tab1.id > 2
```

## 使用视图
使用视图时，将其当作表进行操作即可，由于视图是虚拟表，所以无法使用其对真实表进行创建、更新和删除操作，**仅能做查询用**。

```mysql
select * from v1
```

## 存储过程 procedure

### 为什么要用存储过程

我们都知道应用程序分为两种，一种是基于 web，一种是基于桌面，他们都和数据库进行交互来完成数据的存取工作。假设现在有一种应用程序包含了这两 种，现在要修改其中的一个查询 sql 语句，那么我们可能要同时修改他们中对应的查询 sql 语句，当我们的应用程序很庞大很复杂的时候问题就出现这，不易维护！另外把 sql 查询语句放在我们的 web 程序或桌面中很容易遭到 sql 注入的破坏。而存储例程正好可以帮我们解决这些问题。

### 创建存储过程

创建存储过程这块主要有两种，一种是带参数的，一种是不带参数的。

- 不带参数示例

```mysql
-- 自定义语句结尾符号，因为这里要执行好多句 SQL 语句，所以就得自定义，以防出错
delimiter
create procedure p1()
BEGIN
    select * from tab1;
END
-- 自定义局域结尾符号结束
delimiter;
```

- 带参数示例

```mysql
delimiter\\
create procedure p1(
    -- 传入参数 i1
    in i1 int,
    -- 传入参数 i2
    in i2 int,
    -- 即能传入又能得到返回值
    inout i3 int,
    -- 得到返回值
    out r1 int
)
BEGIN
    DECLARE temp1 int;
    DECLARE temp2 int default 0;
    set temp1=1;
    set r1 = i1 + i2 + temp1 + temp2;
    set i3 = i3 + 100;
END\\
delimiter;
```

### 执行存储过程

```mysql
-- 设置变量默认值为 3
DECLARE @t1 INT default 3;

-- 设置变量
DECLARE @t2 INT;

-- 执行存储过程，并传入参数，t2 自动取消
CALL p1(1,2,@t1,@t2);

-- 查看存储过程输入结果
SELECT @t1,@t2;
```

### 删除存储过程

```mysql
drop procedure p1;
```

## 函数function

在MySQL中有很多内置函数，比如我们经常用的求平均值，求和，个数，各式各样，先给大家来一部门内置函数，然后再说说自定义函数吧，函数也可以传参数，也可以接收返回值，但是函数没办法得到执行语句得到的结果，存储过程可以。

- CHAR_LENGTH(str)
 
    返回值为字符串 str 的长度，长度的单位为字符。一个多字节字符算作一个单字符。

    对于一个包含五个二字节字符集, LENGTH() 返回值为 10, 而 CHAR_LENGTH() 的返回值为 5。

- CONCAT(str1, str2, ...)
 
    字符串拼接
 
    如有任何一个参数为 NULL ，则返回值为 NULL。

- CONCAT_WS(separator, str1, str2, ...)
 
    字符串拼接（自定义连接符）
 
    CONCAT_WS() 不会忽略任何空字符串。（然而会忽略所有的 NULL）。

- CONV(N, from_base, to_base)
 
    进制转换
 
    e.g.
    ```mysql
    -- 将 a 由 16进制转换为 2进制字符串表示
    SELECT CONV('a', 16, 2);
    ```

- FORMAT(X, D)
 
    将数字 X 的格式写为'#,###,###.##'，以四舍五入的方式保留小数点后 D 位， 并将结果以字符串的形式返回。若 D 为 0, 则返回结果不带有小数点，或不含小数部分。
 
    e.g.
    ```mysql
    -- 结果为：'12,332.1000'
    SELECT FORMAT(12332.1, 4);

    -- 在 str 的指定位置插入字符串
    -- pos：要替换位置其实位置
    -- len：替换的长度
    -- newstr：新字符串
    INSERT(str, pos, len, newstr);
    ```
    
    **特别的**：
    如果 pos 超过原字符串长度，则返回原字符串
    如果 len 超过原字符串长度，则由新字符串完全替换

- INSTR(str, substr)
 
    返回字符串 str 中子字符串的第一个出现位置。

- LEFT(str, len)
 
    返回字符串 str 从开始的 len 位置的子序列字符。

- LOWER(str)
 
    变小写

- UPPER(str)
 
    变大写

- LTRIM(str)
 
    返回字符串 str ，其引导空格字符被删除。

- RTRIM(str)
 
    返回字符串 str ，结尾空格字符被删去。

- SUBSTRING(str, pos, len)
 
    获取字符串子序列

- LOCATE(substr, str, pos)
 
    获取子序列索引位置

- REPEAT(str, count)
 
    返回一个由重复的字符串 str 组成的字符串，字符串 str 的数目等于 count 。
 
    若 count <= 0，则返回一个空字符串。
 
    若 str 或 count 为 NULL，则返回 NULL 。

- REPLACE(str, from_str, to_str)
 
    返回字符串 str 以及所有被字符串 to_str 替代的字符串 from_str 。

- REVERSE(str)
 
    返回字符串 str ，顺序和字符顺序相反。

- RIGHT(str, len)
 
    从字符串 str 开始，返回从后边开始 len 个字符组成的子序列

- SPACE(N)
 
    返回一个由 N 空格组成的字符串。

- SUBSTRING(str, pos) , 
  
  SUBSTRING(str FROM pos) 
  
  SUBSTRING(str, pos, len)
  
  SUBSTRING(str FROM pos FOR len)
 
    不带有 len 参数的格式从字符串 str 返回一个子字符串，起始于位置 pos
    
    带有 len 参数的格式从字符串 str 返回一个长度同 len 字符相同的子字符串，起始于位置 pos 
    
    使用 FROM 的格式为标准 SQL 语法。也可能对 pos 使用一个负值。假若这样，则子字符串的位置起始于字符串结尾的 pos 字符，而不是字符串的开头位置。
    
    在以下格式的函数中可以对 pos 使用一个负值。

    ```mysql
    -- 'ratically'
    SELECT SUBSTRING('Quadratically', 5);
    
    -- 'barbar'
    SELECT SUBSTRING('foobarbar' FROM 4);
    
    -- 'ratica'
    SELECT SUBSTRING('Quadratically',5,6);
    
    -- 'ila'
    SELECT SUBSTRING('Sakila', -3);
    
    -- 'aki'
    SELECT SUBSTRING('Sakila', -5, 3);
    
    -- 'ki'
    SELECT SUBSTRING('Sakila' FROM -4 FOR 2);
    ```

- TRIM([{BOTH | LEADING | TRAILING} [remstr] FROM] str) 

  TRIM([remstr] FROM str)
  
  返回字符串 str ，其中所有 remstr 前缀和/或后缀都已被删除。若分类符 BOTH、LEADIN 或TRAILING 中没有一个是给定的，则假设为 BOTH 。remstr 为可选项，在未指定情况下，可删除空格。

  ```mysql
  -- 'bar'
  SELECT TRIM('  bar   ');
  
  -- 'barxxx'
  SELECT TRIM(LEADING 'x' FROM 'xxxbarxxx');
  
  -- 'bar'
  SELECT TRIM(BOTH 'x' FROM 'xxxbarxxx');
  
  -- 'barx'
  SELECT TRIM(TRAILING 'xyz' FROM 'barxxyz');
  ```

### 自定义创建函数

```mysql
delimiter\\
create function f1(
    i1 int,
    i2,int)
returns int
BEGIN
    declare num int;
    set num = i1 + i2;
    return(num);
END\\
delimiter;
```

### 删除函数

```mysql
drop function f1;
```

### 执行函数

```mysql
-- 获取返回值
declare @i VARCHAR(32);
select UPPER('alex') into @i;
SELECT @i;

-- 在查询中使用
select f1(11,nid),name from tb2;
```

## 事务

事务用于将某些操作的多个 SQL 作为原子性操作，一旦有某一个出现错误，即可回滚到原来的状态，从而保证数据库数据完整性。例如：当两张银行卡之间进行转账，甲方钱转出去了，突然光缆坏了，乙方还没收到钱，钱跑哪里去了，就为了防止这种情况，事务就出来了，事务可以防止这种事情发生

### 应用事务示例

```mysql
delimiter \\
create PROCEDURE p1(
    OUT p_return_code tinyint
)
BEGIN 
  DECLARE exit handler for sqlexception 
  BEGIN 
    -- ERROR 
    set p_return_code = 1; 
    rollback; 
  END; 

  DECLARE exit handler for sqlwarning 
  BEGIN 
    -- WARNING 
    set p_return_code = 2; 
    rollback; 
  END; 

  START TRANSACTION;
    -- sql 语句都放在这个里面 
    DELETE from tb1;
    insert into tb2(name)values('seven');
  COMMIT; 

  -- SUCCESS 
  set p_return_code = 0; 

  END\\
delimiter ;
```

### 执行存储过程

```mysql
DECLARE @i TINYINT;
call p1(@i);
select @i;
```

## 触发器TRIGGER

触发器，简单来说就是当你在执行这条语句之前或者之后触发一次增删改查，触发器用于定制用户对表的行进行【增/删/改】前后的行为。

### 基本语法

- 插入前

```mysql
CREATE TRIGGER tri_before_insert_tb1 BEFORE INSERT ON tb1 FOR EACH ROW
BEGIN
   ...
END
```

- 插入后

```mysql
CREATE TRIGGER tri_after_insert_tb1 AFTER INSERT ON tb1 FOR EACH ROW
BEGIN
    ...
END
```

- 删除前

```mysql
CREATE TRIGGER tri_before_delete_tb1 BEFORE DELETE ON tb1 FOR EACH ROW
BEGIN
    ...
END
```

- 删除后

```mysql
CREATE TRIGGER tri_after_delete_tb1 AFTER DELETE ON tb1 FOR EACH ROW
BEGIN
    ...
END
```

- 更新前

```mysql
CREATE TRIGGER tri_before_update_tb1 BEFORE UPDATE ON tb1 FOR EACH ROW
BEGIN
    ...
END
```

- 更新后

```mysql
CREATE TRIGGER tri_after_update_tb1 AFTER UPDATE ON tb1 FOR EACH ROW
BEGIN
    ...
END
```

- e.g.插入前

```mysql
-- 在往 tab1 插入数据之前往 tab2 中插入一条 name = 张岩林，当然是在判断往 tab1 中插入的名字是不是等于 aylin
delimiter //
CREATE TRIGGER tri_before_insert_tb1 BEFORE INSERT ON tb1 FOR EACH ROW
BEGIN

IF NEW. NAME == 'aylin' THEN
    INSERT INTO tb2 (NAME)
VALUES
    ('张三')
END
END//
delimiter ;

```

- e.g.插入后

```mysql
delimiter //
CREATE TRIGGER tri_after_insert_tb1 AFTER INSERT ON tb1 FOR EACH ROW
BEGIN
    IF NEW. num = 666 THEN
        INSERT INTO tb2 (NAME)
        VALUES
            ('张三'),
            ('很帅') ;
    ELSEIF NEW. num = 555 THEN
        INSERT INTO tb2 (NAME)
        VALUES
            ('aylin'),
            ('非常帅') ;
    END IF;
END//
delimiter ;
```

同样的删，改，查都是同样的道理

**特别的**：NEW表示即将插入的数据行，OLD 表示即将删除的数据行

### 删除触发器

```mysql
DROP TRIGGER tri_after_insert_tb1;
```

### 使用触发器

触发器无法由用户直接调用，而是由于对表的【增/删/改】操作被动引发的。

```mysql
insert into tb1(name) values(‘张三’)
```