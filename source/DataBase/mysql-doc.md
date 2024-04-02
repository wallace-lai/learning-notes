# 【MySQL】MySQL初步

作者：wallace-lai <br/>
发布：2018-02-16 <br/>
更新：2024-04-02 <br/>

## 一、写在开头
本文的主要内容是MySQL的基本操作，资料来自MySQL 5.7官方文档。

### 工具准备
一台装好了mysql的ubuntu 16.04 LTS机器。

## 二、MySQL的连接与断开
### 连接与断开MySQL服务
MySQL提供了一个Linux命令行程序mysql，利用它我们可以连接到MySQL并执行SQL操作。想要查看mysql自带帮助文档，请使用以下命令。

```shell
mysql --help
```

### 连接MySQL

使用以下命令连接到MySQL
```shell
mysql -h localhost -u root -p
```

命令详解：

（1）-h后面参数表示主机名。如果MySQL安装在了本地，后面的参数可以使用localhost表示本地。也可以省略-h参数。

（2）-u后面的参数表示使用的账户名。可以使用root用户，前提是你得记得安装MySQL时输入的root用户密码。

（3）-p参数表示需要输入密码。一般来说-p参数是必须的。

### 退出

使用quit命令退出mysql命令行

```shell
quit
```

## 三、键入查询语句
### 查询MySQL版本和当前时间
```shell
SELECT VERSION(), CURRENT_DATE();
+-----------+----------------+
| VERSION() | CURRENT_DATE() |
+-----------+----------------+
| 5.7.21    | 2018-02-14     |
+-----------+----------------+
1 row in set (0.06 sec)
```

注意：SQL语句后面有个分号，quit命令后没有分号。

### 使用MySQL做简单计算
```shell
SELECT SIN(PI()/2), (4+1)*5;
+-------------+---------+
| SIN(PI()/2) | (4+1)*5 |
+-------------+---------+
|           1 |      25 |
+-------------+---------+
1 row in set (0.05 sec)
```

### 将多条命令写在同一行
```shell
SELECT VERSION(); SELECT NOW();
+-----------+
| VERSION() |
+-----------+
| 5.7.21    |
+-----------+
1 row in set (0.00 sec)

+---------------------+
| NOW()               |
+---------------------+
| 2018-02-14 15:11:56 |
+---------------------+
1 row in set (0.00 sec)
```

### 将一条命令写在多行中
```shell
SELECT 
USER()
,
CURRENT_DATE();
+----------------+----------------+
| USER()         | CURRENT_DATE() |
+----------------+----------------+
| root@localhost | 2018-02-14     |
+----------------+----------------+
1 row in set (0.05 sec)
```

注意：在MySQL中，识别一条SQL命令的标志是分号。所以我们可以把一条命令写在多行中。

### 撤销当前输入
```shell
mysql> SELECT
-> USER()
-> \c
mysql>
```

注意：c是小写的。

## 四、数据库的创建和使用
### 查看当前系统中的数据库
```shell
SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.15 sec)
```

注意：SHOW DATEBASES;只会将当前用户具有的SHOW DATABASES命令权限的数据库打印出来。

### 使用（进入）数据库
```shell
USE sys
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
```

注意：USE和前面的QUIT一样，后面是不接分号的。与之前命令不一样的是USE命令必须写在单独一行中，不能跨越多行。

### 创建一个数据库
```shell
CREATE DATABASE menagerie;
Query OK, 1 row affected (0.07 sec)
```

注意：在Linux下数据库名是大小写敏感的。

### 在连接MySQL时就指定使用menagerie数据库
```shell
mysql -u root -p menagerie
```

### 查看当前使用的是哪个数据库
```shell
SELECT DATABASE();
+------------+
| DATABASE() |
+------------+
| menagerie  |
+------------+
1 row in set (0.00 sec)
```

### 查看当前数据库中有哪些表
```shell
SELECT TABLES;
Empty set (0.00 sec)
```

注意：刚刚创建的数据库中是没有表的。

### 创建表格
```shell
mysql> CREATE TABLE pet
    -> (
    -> name VARCHAR(20),
    -> owner VARCHAR(20),
    -> species VARCHAR(20),
    -> sex CHAR(1),
    -> birth DATE,
    -> death DATE
    -> );
Query OK, 0 rows affected (0.15 sec)

mysql> SHOW TABLES;
+---------------------+
| Tables_in_menagerie |
+---------------------+
| pet                 |
+---------------------+
1 row in set (0.01 sec)
```

### 查看表的信息
```shell
mysql> DESCRIBE pet;
+---------+-------------+------+-----+---------+-------+
| Field   | Type        | Null | Key | Default | Extra |
+---------+-------------+------+-----+---------+-------+
| name    | varchar(20) | YES  |     | NULL    |       |
| owner   | varchar(20) | YES  |     | NULL    |       |
| species | varchar(20) | YES  |     | NULL    |       |
| sex     | char(1)     | YES  |     | NULL    |       |
| birth   | date        | YES  |     | NULL    |       |
| death   | date        | YES  |     | NULL    |       |
+---------+-------------+------+-----+---------+-------+
6 rows in set (0.00 sec)
```

### 向表中输入信息
第一步：创建一个pet.sql文件，文件内容如下。

```shell
INSERT INTO pet
VALUES ('Fluffy', 'Harold', 'cat', 'f', '1993-02-04', NULL);

INSERT INTO pet
VALUES ('Claws', 'Gwen', 'cat', 'm', '1994-03-17', NULL);

INSERT INTO pet
VALUES ('Buffy', 'Harold', 'dog', 'f', '1989-05-13', NULL);

INSERT INTO pet
VALUES ('Fang', 'Benny', 'dog', 'm', '1990-08-27', NULL);

INSERT INTO pet
VALUES ('Bowser', 'Diane', 'dog', 'm', '1979-08-31', '1995-07-29');

INSERT INTO pet
VALUES ('Chirpy', 'Gwen', 'bird', 'f', '1998-09-11', NULL);

INSERT INTO pet
VALUES ('Whistler', 'Gwen', 'bird', NULL, '1997-12-09', NULL);

INSERT INTO pet
VALUES ('Slim', 'Benny', 'snake', 'm', '1996-04-29', NULL);

INSERT INTO pet
VALUES ('Puffball', 'Diane', 'hamster', 'f', '1999-03-30', NULL);
```

第二步：使用下列命令执行sql文件，随后查看表的内容。
```shell
mysql> source ~/pet.sql
Query OK, 1 row affected (0.13 sec)

Query OK, 1 row affected (0.01 sec)

Query OK, 1 row affected (0.02 sec)

Query OK, 1 row affected (0.01 sec)

Query OK, 1 row affected (0.01 sec)

Query OK, 1 row affected (0.00 sec)

Query OK, 1 row affected (0.00 sec)

Query OK, 1 row affected (0.00 sec)

Query OK, 1 row affected (0.01 sec)

mysql> SELECT * FROM pet;
+----------+--------+---------+------+------------+------------+
| name     | owner  | species | sex  | birth      | death      |
+----------+--------+---------+------+------------+------------+
| Fluffy   | Harold | cat     | f    | 1993-02-04 | NULL       |
| Claws    | Gwen   | cat     | m    | 1994-03-17 | NULL       |
| Buffy    | Harold | dog     | f    | 1989-05-13 | NULL       |
| Fang     | Benny  | dog     | m    | 1990-08-27 | NULL       |
| Bowser   | Diane  | dog     | m    | 1979-08-31 | 1995-07-29 |
| Chirpy   | Gwen   | bird    | f    | 1998-09-11 | NULL       |
| Whistler | Gwen   | bird    | NULL | 1997-12-09 | NULL       |
| Slim     | Benny  | snake   | m    | 1996-04-29 | NULL       |
| Puffball | Diane  | hamster | f    | 1999-03-30 | NULL       |
+----------+--------+---------+------+------------+------------+
9 rows in set (0.00 sec)
```

## 五、SELECT语句
### 选择所有的列
```shell
mysql> SELECT * FROM pet;
+----------+--------+---------+------+------------+------------+
| name     | owner  | species | sex  | birth      | death      |
+----------+--------+---------+------+------------+------------+
| Fluffy   | Harold | cat     | f    | 1993-02-04 | NULL       |
| Claws    | Gwen   | cat     | m    | 1994-03-17 | NULL       |
| Buffy    | Harold | dog     | f    | 1989-05-13 | NULL       |
| Fang     | Benny  | dog     | m    | 1990-08-27 | NULL       |
| Bowser   | Diane  | dog     | m    | 1979-08-31 | 1995-07-29 |
| Chirpy   | Gwen   | bird    | f    | 1998-09-11 | NULL       |
| Whistler | Gwen   | bird    | NULL | 1997-12-09 | NULL       |
| Slim     | Benny  | snake   | m    | 1996-04-29 | NULL       |
| Puffball | Diane  | hamster | f    | 1999-03-30 | NULL       |
+----------+--------+---------+------+------------+------------+
9 rows in set (0.00 sec)
```

### 修改数据
```shell
mysql> UPDATE pet SET birth = '1989-08-31' WHERE name = 'Bowser';
Query OK, 1 row affected (0.17 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> SELECT birth FROM pet WHERE name = 'Bowser';
+------------+
| birth      |
+------------+
| 1989-08-31 |
+------------+
1 row in set (0.00 sec)
```

### 清空表内容
```shell
mysql> DELETE FROM pet;
Query OK, 9 rows affected (0.08 sec)

mysql> SELECT * FROM pet;
Empty set (0.00 sec)
```

### 选择特定行
```shell
mysql> SELECT * FROM pet WHERE name = 'Bowser';
+--------+-------+---------+------+------------+------------+
| name   | owner | species | sex  | birth      | death      |
+--------+-------+---------+------+------------+------------+
| Bowser | Diane | dog     | m    | 1989-08-31 | 1995-07-29 |
+--------+-------+---------+------+------------+------------+
1 row in set (0.00 sec)
```

### 根据条件选择
```shell
mysql> SELECT * FROM pet WHERE birth >= '1998-1-1';
+----------+-------+---------+------+------------+-------+
| name     | owner | species | sex  | birth      | death |
+----------+-------+---------+------+------------+-------+
| Chirpy   | Gwen  | bird    | f    | 1998-09-11 | NULL  |
| Puffball | Diane | hamster | f    | 1999-03-30 | NULL  |
+----------+-------+---------+------+------------+-------+
2 rows in set (0.00 sec)
```

### 根据组合条件选择
```shell
mysql> SELECT * FROM pet WHERE species = 'dog' AND sex = 'f';
+-------+--------+---------+------+------------+-------+
| name  | owner  | species | sex  | birth      | death |
+-------+--------+---------+------+------------+-------+
| Buffy | Harold | dog     | f    | 1989-05-13 | NULL  |
+-------+--------+---------+------+------------+-------+
1 row in set (0.04 sec)
```

```shell
mysql> SELECT * FROM pet WHERE
    -> (species = 'cat' AND sex = 'm')
    -> OR (species = 'dog' AND sex = 'f');
+-------+--------+---------+------+------------+-------+
| name  | owner  | species | sex  | birth      | death |
+-------+--------+---------+------+------------+-------+
| Claws | Gwen   | cat     | m    | 1994-03-17 | NULL  |
| Buffy | Harold | dog     | f    | 1989-05-13 | NULL  |
+-------+--------+---------+------+------------+-------+
2 rows in set (0.00 sec)
```

注意：AND优先级高于OR。

### 选择特定列
```shell
mysql> SELECT name, birth FROM pet;
+----------+------------+
| name     | birth      |
+----------+------------+
| Fluffy   | 1993-02-04 |
| Claws    | 1994-03-17 |
| Buffy    | 1989-05-13 |
| Fang     | 1990-08-27 |
| Bowser   | 1989-08-31 |
| Chirpy   | 1998-09-11 |
| Whistler | 1997-12-09 |
| Slim     | 1996-04-29 |
| Puffball | 1999-03-30 |
+----------+------------+
9 rows in set (0.00 sec)
```

```shell
mysql> SELECT owner FROM pet;
+--------+
| owner  |
+--------+
| Harold |
| Gwen   |
| Harold |
| Benny  |
| Diane  |
| Gwen   |
| Gwen   |
| Benny  |
| Diane  |
+--------+
9 rows in set (0.00 sec)
```

过滤重复项，使重复项仅出现一次
```shell
mysql> SELECT DISTINCT owner FROM pet;
+--------+
| owner  |
+--------+
| Harold |
| Gwen   |
| Benny  |
| Diane  |
+--------+
4 rows in set (0.05 sec)
```

### 选择特定的行与列
```shell
mysql> SELECT name, species, birth FROM pet
    -> WHERE species = 'dog' OR species = 'cat';
+--------+---------+------------+
| name   | species | birth      |
+--------+---------+------------+
| Fluffy | cat     | 1993-02-04 |
| Claws  | cat     | 1994-03-17 |
| Buffy  | dog     | 1989-05-13 |
| Fang   | dog     | 1990-08-27 |
| Bowser | dog     | 1989-08-31 |
+--------+---------+------------+
5 rows in set (0.00 sec)
```

### 对输出结果进行排序
```shell
mysql> SELECT name, birth FROM pet ORDER BY birth;
+----------+------------+
| name     | birth      |
+----------+------------+
| Buffy    | 1989-05-13 |
| Bowser   | 1989-08-31 |
| Fang     | 1990-08-27 |
| Fluffy   | 1993-02-04 |
| Claws    | 1994-03-17 |
| Slim     | 1996-04-29 |
| Whistler | 1997-12-09 |
| Chirpy   | 1998-09-11 |
| Puffball | 1999-03-30 |
+----------+------------+
9 rows in set (0.00 sec)
```

### 使用DESC关键字进行降序排列
```shell
mysql> SELECT name, birth FROM pet ORDER BY birth DESC;
+----------+------------+
| name     | birth      |
+----------+------------+
| Puffball | 1999-03-30 |
| Chirpy   | 1998-09-11 |
| Whistler | 1997-12-09 |
| Slim     | 1996-04-29 |
| Claws    | 1994-03-17 |
| Fluffy   | 1993-02-04 |
| Fang     | 1990-08-27 |
| Bowser   | 1989-08-31 |
| Buffy    | 1989-05-13 |
+----------+------------+
9 rows in set (0.00 sec)
```

### 对多个列使用不同的排序顺序
```shell
mysql> SELECT name, species, birth FROM pet ORDER BY species, birth DESC;
+----------+---------+------------+
| name     | species | birth      |
+----------+---------+------------+
| Chirpy   | bird    | 1998-09-11 |
| Whistler | bird    | 1997-12-09 |
| Claws    | cat     | 1994-03-17 |
| Fluffy   | cat     | 1993-02-04 |
| Fang     | dog     | 1990-08-27 |
| Bowser   | dog     | 1989-08-31 |
| Buffy    | dog     | 1989-05-13 |
| Puffball | hamster | 1999-03-30 |
| Slim     | snake   | 1996-04-29 |
+----------+---------+------------+
9 rows in set (0.00 sec)
```
## 六、常用的计算函数
### TIMESTAMPDIFF()函数

（1）TIMESTAMPDIFF()

```shell
mysql> SELECT name, birth, CURDATE(),
    -> TIMESTAMPDIFF(YEAR, birth, CURDATE()) AS age
    -> FROM pet;
+----------+------------+------------+------+
| name     | birth      | CURDATE()  | age  |
+----------+------------+------------+------+
| Fluffy   | 1993-02-04 | 2018-02-15 |   25 |
| Claws    | 1994-03-17 | 2018-02-15 |   23 |
| Buffy    | 1989-05-13 | 2018-02-15 |   28 |
| Fang     | 1990-08-27 | 2018-02-15 |   27 |
| Bowser   | 1989-08-31 | 2018-02-15 |   28 |
| Chirpy   | 1998-09-11 | 2018-02-15 |   19 |
| Whistler | 1997-12-09 | 2018-02-15 |   20 |
| Slim     | 1996-04-29 | 2018-02-15 |   21 |
| Puffball | 1999-03-30 | 2018-02-15 |   18 |
+----------+------------+------------+------+
9 rows in set (0.05 sec)
```

（2）排序
```shell
mysql> SELECT name, birth, CURDATE(),
    -> TIMESTAMPDIFF(YEAR, birth, CURDATE()) AS age
    -> FROM pet ORDER BY name;
+----------+------------+------------+------+
| name     | birth      | CURDATE()  | age  |
+----------+------------+------------+------+
| Bowser   | 1989-08-31 | 2018-02-15 |   28 |
| Buffy    | 1989-05-13 | 2018-02-15 |   28 |
| Chirpy   | 1998-09-11 | 2018-02-15 |   19 |
| Claws    | 1994-03-17 | 2018-02-15 |   23 |
| Fang     | 1990-08-27 | 2018-02-15 |   27 |
| Fluffy   | 1993-02-04 | 2018-02-15 |   25 |
| Puffball | 1999-03-30 | 2018-02-15 |   18 |
| Slim     | 1996-04-29 | 2018-02-15 |   21 |
| Whistler | 1997-12-09 | 2018-02-15 |   20 |
+----------+------------+------------+------+
9 rows in set (0.00 sec)
```

（3）计算寿命
```shell
mysql> SELECT name, birth, death,
    -> TIMESTAMPDIFF(YEAR, birth, death) AS age
    -> FROM pet WHERE death IS NOT NULL ORDER BY age;
+--------+------------+------------+------+
| name   | birth      | death      | age  |
+--------+------------+------------+------+
| Bowser | 1989-08-31 | 1995-07-29 |    5 |
+--------+------------+------------+------+
1 row in set (0.00 sec)
```

注意：NULL是一个特殊的值，判断值不为空（即不为NULL）需要使用death IS NOT NULL，而不能使用death <> NULL。


### MONTH()函数
（1）MONTH()
```shell
mysql> SELECT name, birth, MONTH(birth) FROM pet;
+----------+------------+--------------+
| name     | birth      | MONTH(birth) |
+----------+------------+--------------+
| Fluffy   | 1993-02-04 |            2 |
| Claws    | 1994-03-17 |            3 |
| Buffy    | 1989-05-13 |            5 |
| Fang     | 1990-08-27 |            8 |
| Bowser   | 1989-08-31 |            8 |
| Chirpy   | 1998-09-11 |            9 |
| Whistler | 1997-12-09 |           12 |
| Slim     | 1996-04-29 |            4 |
| Puffball | 1999-03-30 |            3 |
+----------+------------+--------------+
9 rows in set (0.00 sec)
```

注意：受上面结果的启发，YEAR(birth)，DAYOFMONTH(birth)的结果是什么呢？

（2）找到5月出生的动物
```shell
mysql> SELECT name, birth FROM pet WHERE MONTH(birth) = 5;
+-------+------------+
| name  | birth      |
+-------+------------+
| Buffy | 1989-05-13 |
+-------+------------+
1 row in set (0.05 sec)
```


### DATE_ADD()
```shell
mysql> SELECT name, birth FROM pet
    -> WHERE MONTH(birth) = MONTH(DATE_ADD(CURDATE(), INTERVAL 1 MONTH));
+----------+------------+
| name     | birth      |
+----------+------------+
| Claws    | 1994-03-17 |
| Puffball | 1999-03-30 |
+----------+------------+
2 rows in set (0.08 sec)
```

### 如何处理为空值
```shell
mysql> SELECT 1 IS NULL, 1 IS NOT NULL;
+-----------+---------------+
| 1 IS NULL | 1 IS NOT NULL |
+-----------+---------------+
|         0 |             1 |
+-----------+---------------+
1 row in set (0.00 sec)
```

## 七、模式匹配
### 匹配规则

（1）_（下划线）- 表示单个字符

（2）%（百分号） - 表示任意个字符

注意：模式匹配时是大小写敏感的，匹配使用的运算符是LIKE和NOT LIKE。不能使用数学运算符。

### 小例子
（1）找到以字母b开头的宠物

```shell
mysql> SELECT * FROM pet WHERE name LIKE 'b%';
+--------+--------+---------+------+------------+------------+
| name   | owner  | species | sex  | birth      | death      |
+--------+--------+---------+------+------------+------------+
| Buffy  | Harold | dog     | f    | 1989-05-13 | NULL       |
| Bowser | Diane  | dog     | m    | 1989-08-31 | 1995-07-29 |
+--------+--------+---------+------+------------+------------+
2 rows in set (0.00 sec)
```

（2）找到以fy结尾的宠物

```shell
mysql> SELECT * FROM pet WHERE name LIKE '%fy';
+--------+--------+---------+------+------------+-------+
| name   | owner  | species | sex  | birth      | death |
+--------+--------+---------+------+------------+-------+
| Fluffy | Harold | cat     | f    | 1993-02-04 | NULL  |
| Buffy  | Harold | dog     | f    | 1989-05-13 | NULL  |
+--------+--------+---------+------+------------+-------+
2 rows in set (0.00 sec)
```

（3）找到名字中含有字母w的宠物

```shell
mysql> SELECT * FROM pet WHERE name LIKE '%w%';
+----------+-------+---------+------+------------+------------+
| name     | owner | species | sex  | birth      | death      |
+----------+-------+---------+------+------------+------------+
| Claws    | Gwen  | cat     | m    | 1994-03-17 | NULL       |
| Bowser   | Diane | dog     | m    | 1989-08-31 | 1995-07-29 |
| Whistler | Gwen  | bird    | NULL | 1997-12-09 | NULL       |
+----------+-------+---------+------+------------+------------+
3 rows in set (0.00 sec)
```

（4）找到名字为5个字符的宠物

```shell
mysql> SELECT * FROM pet WHERE name LIKE '_____';
+-------+--------+---------+------+------------+-------+
| name  | owner  | species | sex  | birth      | death |
+-------+--------+---------+------+------------+-------+
| Claws | Gwen   | cat     | m    | 1994-03-17 | NULL  |
| Buffy | Harold | dog     | f    | 1989-05-13 | NULL  |
+-------+--------+---------+------+------------+-------+
2 rows in set (0.00 sec)
```

## 八、行计数
### COUNT()函数
（1）共有多少宠物

```shell
mysql> SELECT COUNT(*) FROM pet;
+----------+
| COUNT(*) |
+----------+
|        9 |
+----------+
1 row in set (0.04 sec)
```

（2）每个主人共有几只宠物

```shell
mysql> SELECT owner, COUNT(*) FROM pet GROUP BY owner;
+--------+----------+
| owner  | COUNT(*) |
+--------+----------+
| Benny  |        2 |
| Diane  |        2 |
| Gwen   |        3 |
| Harold |        2 |
+--------+----------+
4 rows in set (0.05 sec)
```

（3）每个物种共有几只宠物

```shell
mysql> SELECT species, COUNT(*) FROM pet GROUP BY species;
+---------+----------+
| species | COUNT(*) |
+---------+----------+
| bird    |        2 |
| cat     |        2 |
| dog     |        3 |
| hamster |        1 |
| snake   |        1 |
+---------+----------+
5 rows in set (0.01 sec)
```

（4）每种性别共有几只宠物

```shell
mysql> SELECT sex, COUNT(*) FROM pet GROUP BY sex;
+------+----------+
| sex  | COUNT(*) |
+------+----------+
| NULL |        1 |
| f    |        4 |
| m    |        4 |
+------+----------+
3 rows in set (0.00 sec)
```

（5）每种物种和性别共有几只宠物

```shell
mysql> SELECT species, sex, COUNT(*) FROM pet GROUP BY species, sex;
+---------+------+----------+
| species | sex  | COUNT(*) |
+---------+------+----------+
| bird    | NULL |        1 |
| bird    | f    |        1 |
| cat     | f    |        1 |
| cat     | m    |        1 |
| dog     | f    |        1 |
| dog     | m    |        2 |
| hamster | f    |        1 |
| snake   | m    |        1 |
+---------+------+----------+
8 rows in set (0.00 sec)
```

（6）只考虑狗和猫的情况下，每种物种和性别共有几只宠物

```shell
mysql> SELECT species, sex, COUNT(*) FROM pet
    -> WHERE species = 'dog' OR species = 'cat'
    -> GROUP BY species, sex;
+---------+------+----------+
| species | sex  | COUNT(*) |
+---------+------+----------+
| cat     | f    |        1 |
| cat     | m    |        1 |
| dog     | f    |        1 |
| dog     | m    |        2 |
+---------+------+----------+
4 rows in set (0.06 sec)
```

（7）排除性别未知的宠物情况下，每种物种和性别共有几只宠物

```shell
mysql> SELECT species, sex, COUNT(*) FROM pet
    -> WHERE sex IS NOT NULL
    -> GROUP BY species, sex;
+---------+------+----------+
| species | sex  | COUNT(*) |
+---------+------+----------+
| bird    | f    |        1 |
| cat     | f    |        1 |
| cat     | m    |        1 |
| dog     | f    |        1 |
| dog     | m    |        2 |
| hamster | f    |        1 |
| snake   | m    |        1 |
+---------+------+----------+
7 rows in set (0.00 sec)
```

## 九、使用多个表

### 创建一个event表以记录宠物的其他信息

```shell
mysql> CREATE TABLE event -> (
    -> name VARCHAR(20),
    -> date DATE,
    -> type VARCHAR(15),
    -> remark VARCHAR(255)
    -> );
Query OK, 0 rows affected (0.49 sec)
```

### 往表中插入数据

```shell
INSERT INTO event
VALUES ('Fluffy', '1995-05-15', 'litter', '4 kittens, 3 female, 1 male');

INSERT INTO event
VALUES ('Buffy', '1993-06-23', 'litter', '5 puppies, 2 female, 3 male');

INSERT INTO event
VALUES ('Buffy', '1994-06-19', 'litter', '3 puppies, 3 female');

INSERT INTO event
VALUES ('Chirpy', '1999-03-21', 'vet', 'needed beak straightened');

INSERT INTO event
VALUES ('Slim', '1997-08-03', 'vet', 'broken rib');

INSERT INTO event
VALUES ('Bowser', '1991-10-12', 'kennel', NULL);

INSERT INTO event
VALUES ('Fang', '1991-10-12', 'kennel', NULL);

INSERT INTO event
VALUES ('Fang', '1998-08-28', 'birthday', 'Gave him a new chew toy');

INSERT INTO event
VALUES ('Claws', '1998-03-17', 'birthday', 'Gave him a new flea collar');

INSERT INTO event
VALUES ('Whistler', '1998-12-09', 'birthday', 'First birthday');
```

### 多表连接

（1）

```shell
mysql> SELECT pet.name
    -> ,
    -> (YEAR(date)-YEAR(birth)) - (RIGHT(date, 5) < RIGHT(birth, 5)) AS age,
    -> remark
    -> FROM pet INNER JOIN event
    -> ON pet.name = event.name
    -> WHERE event.type = 'litter';
+--------+------+-----------------------------+
| name   | age  | remark                      |
+--------+------+-----------------------------+
| Fluffy |    2 | 4 kittens, 3 female, 1 male |
| Buffy  |    4 | 5 puppies, 2 female, 3 male |
| Buffy  |    5 | 3 puppies, 3 female         |
+--------+------+-----------------------------+
3 rows in set (0.16 sec)
```

（2）
```shell
mysql> SELECT p1.name, p1.sex, p2.name, p2.sex, p1.species
    -> FROM pet AS p1 INNER JOIN pet AS p2
    -> ON p1.species = p2.species AND p1.sex = 'f' AND p2.sex = 'm';
+--------+------+--------+------+---------+
| name   | sex  | name   | sex  | species |
+--------+------+--------+------+---------+
| Fluffy | f    | Claws  | m    | cat     |
| Buffy  | f    | Fang   | m    | dog     |
| Buffy  | f    | Bowser | m    | dog     |
+--------+------+--------+------+---------+
3 rows in set (0.00 sec)
```

## 十、获取数据库信息


|语句|功能|
|--|--|
|SHOW DATABASES;|显示MySQL Server所管理的数据库|
|SELECT DATABASE();|当前选择的是哪个数据库|
|SHOW TABLES;|当前数据库中有哪些表|
|DESCRIBE pet;|显示pet表的表结构|


