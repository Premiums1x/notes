1.DDL 数据定义语言，用来定义数据库对象（数据库、表、表中字段）
2.DML 数据操作语言，用来对数据库表中的数据进行增删改
3.DQL 数据查询语言，查询表中数据
4.DCL 数据控制语言，创建数据库用户，控制数据库访问权限

SQL语句不区分大小写

# DDL数据库操作：
一、查询：
1.查询所有数据库
`SHOW DATABASES;`

2.查询当前数据库：
`SELECT DATABASE();`

二、创建：
	`CREATE DATABASE [IF NOT EXISTS] 数据库名 [DEFAULT CHARSET 字符集] [COLLATE 查询规则];`

三、删除：
`DROP DATABASE [IF EXISTS] 数据库名称;`

四、使用：
`USE 数据库名称`

# DDL表操作：

小结：
![[Pasted image 20260412115816.png]]


 一、创建表：
 ```
 CREATE TABLE 表名(
	 字段1 字段1类型[COMMENT 字段1注释],
	 字段1 字段1类型[COMMENT 字段1注释],
	 .......
	 字段n ,字段n类型[COMMENT 字段n注释]
 )[COMMENT 表注释];
 ```
注：1.最后一个字段无逗号
	2.SQL中字符串是varchar(长度)

二、查询表：
1.查询当前数据库中所有表：
`SHOW TABLES;`

2.查表结构：
`DESC 表名;`

3.查指定表的建表语句：
`SHOW CREATE TABLE 表名;`


三、修改表：
1.添加字段：
`ALTER TABLE 表名 ADD 字段名 字段类型（长度） [COMMENT 注释] [约束]; `

2.修改字段：
（1）仅修改字段数据类型：
`alter table 表名 modify 字段名 新数据类型（长度）;`
  (2) 字段名称、字段类型均修改：
`alter table 表名 change 旧字段名 新字段名 新数据类型（长度） [comment 注释] [约束]`;

3.删除字段：
`alter table 表名 drop 字段名`;

4.修改表名：
`alter table 表名 RENAME TO 新表名;`

5.删除表：(删表时，表中数据会一并删除)
`DROP TBALE [IF EXISTS] 表名`;

!! 删除指定表，并重新创建表结构（目的是清除掉表中的数据，只保留表结构）
`TRUNCATE TABLE 表名;`

# DDL数据类型：
1.数值型：
![[Pasted image 20260412111551.png]]

2.字符串型：
![[Pasted image 20260412112003.png]]

3.时间类型：
![[Pasted image 20260412112146.png]]



# DML：数据操作语言
对数据库中表的数据记录进行增删改操作

一、insert添加记录
1.给指定字段添加数据：
`INSERT INTO 表名 (字段名1，字段名2....) VALUES (值1，值2...)`;

2.全部字段添加数据：
`INSERT INTO 表名 VALUES (值1，值2....);`

3.批量添加数据记录：
(1)指定字段
`INSERT INTO 表名 (字段名1，字段名2....) VALUES (值1，值2...),(值1，值2...),(值1，值2...);`
不同记录用逗号相隔

（2）全部字段
`INSERT INTO 表名 VALUES (值1，值2....), (值1，值2....), (值1，值2....);`

![[Pasted image 20260412151758.png]]

二、update修改记录:
 `UPDATE 表名 SET 字段1 = 值1 , 字段2 = 值2,.... [WHERE 条件]; `
！！！where可选，写了就会更新符合条件的值，如果不写，就会把表中所有数据都更新


三、delete删除记录
`DELETE FROM 表名 [WHERE 条件];`
！！！where可选，写了就会删除符合条件的值，如果不写，就会把表中所有数据删除
！！！DELETE语句无法删除记录的某个字段的值，而是整条记录删掉，若要实现删除某个字段的值，可通过UPDATE置为null。


# DQL数据查询语言：
语法（编写顺序）：
```
SELECT 
	字段名称
FROM 
	表名列表
WHERE
	条件列表
GROUP BY
	分组字段列表
HAVING
	分组后条件列表
ORDER BY
	排序字段列表
LIMIT
	分页参数
```

执行顺序：
![[Pasted image 20260412201206.png]]



1.基本查询：
（1）多个字段：
`SELECT 字段1，字段2.... FROM 表名;`
`SELECT * FROM 表名`

(2)设别名：as关键字
`SELECT 字段1 [as 别名1]，字段2 [别名2]....  FROM 表名;`

(3)去重：
`SELECT DISTINCT 字段列表 FROM 表名;`


2、条件查询：
语法：
`SELECT 字段列表 FROM 表名 WHERE 条件列表;`

条件：
![[Pasted image 20260412164451.png]]


3.聚合函数（常与分组查询配合使用）
将一列数据作为一个整体，做纵向计算：

语法：`SELECT 聚合函数(字段列表) FROM 表名`;

常用：
![[Pasted image 20260412173301.png]]
!!! null值不参与所有聚合函数的计算


4、分组查询：
`SELECT 字段列表 FROM 表名 [WHERE]条件 GROUP BY 分组字段名 [HAVING 分组后的过滤条件];`

Where与Having 的区别：
![[Pasted image 20260412174420.png]]


**注意：**
1.执行顺序：where->聚合函数->having， where在分组前执行，分组时执行聚合函数，分组后having再过滤。
2.分组后查询的字段一般为，分组字段及聚合函数。


5.排序查询：
语法：`SELECT 字段列表 FROM 表名 ORDER BY 字段1，字段2;`

排序方式：
ASC : 升序，默认
DESC ：降序

注：一般是单字段排序，但也支持多字段，多字段排序的含义：是当前方字段值相同时，再根据新的字段值来排序


6.分页查询：
语法：`SELECT 字段列表 FROM 表名 LIMIT 起始索引,查询记录数;`

**注**：
1.起始索引从0开始，起始索引 = （查询页码 - 1）* 每页显示记录数
（就相当于是“分割位点”，我现在这页显示的东西，就是把前面的都去掉，在规定上我现在这页要显示的记录数量）

2.分页查询为数据库方言，每个数据库有不同实现
3.若查询第一页数据，直接可省略起始索引


# DCL 数据控制语言：管理数据库用户、控制数据库访问权限
一、管理用户：

1、查询用户，mysql数据库的用户信息，存放在mysql数据库中的user表中
`USE mysql`
`SELECT * FROM user`;

需要host和user两个键同时来唯一定位一个用户，host规定了用户能在什么主机上访问mysql数据库。


2、创建用户：
`CREATE USER '用户名'@'主机名' IDENTIFIED BY '密码;'`
%通配符表示用户可在任意主机访问该MySQL服务器
*注意，此时只是把用户创建了，还没给权限，只能访问MySQL数据库，没有访问其他数据库权限*


3、修改用户密码：
`ALTER USER '用户名'@'主机名' IDENTIFIED WITH mysql_native_password BY '新密码';`



4、删除用户：
`DROP USER '用户名'@'主机名';`

# 二、权限控制:
![[Pasted image 20260412205218.png]]

1.查询权限：
`SHOW GRANTS FOR '用户名'@'主机名';`

2.授予权限：
`GRANT 权限列表 ON 数据库名.表名 TO '用户名'@'主机名';`

3.撤销权限：
`REVOKE 权限列表 ON 数据库名.表名 FROM '用户名'@'主机名';`

*注意：*
![[Pasted image 20260412210243.png]]





