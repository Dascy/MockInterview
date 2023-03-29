# MySQL学习

## 事务

### 事务的概念

事务就是一组原子性的SQL查询，或者说独立的工作单元。如果数据库能够应用一组查询，那么该组的查询的全部语句的执行结果应该是一致的。要么全部不执行。要么全部执行成功过。

### 事务的特性（ACID）

ACID表示原子性（atomicity）、一致性（consistency）、隔离性（isolation）和持久性（durability）。

- 原子性   一个事务必须视为一个不可分割的整体，事务中的所有操作要么全部执行、要么全部回滚。
- 一致性  数据库总是从一个状态转换成另外一个一致性状态。
- 隔离性  多事务之间，在事务提交之前。对其他事务是不可见的。
- 持久性  事务一旦提交，将永久保留在数据库中，即使数据库崩溃，修改的数据也不会丢失。

#### 原子性和一致性的区别

原子性是为了保证一致性。比较经典的例子是A、B同时修改数据库中的同一条记录。A执行+100操作。B同时也执行了+100的操作。因为A、B的操作同时发生，导致结果只+100的情况。就是满足了原子性。没有满足一致性原则。

### 事务的隔离级别

#### READ UNCOMMITTED（读未提交）

在事务未提交之前，也可读取到事务中的数据。这种操作也称为脏读。

#### READ COMMITTED(读已提交)

事务开始到提交，所做的所有操作对其他事务不可见。

#### REPEATABLE READ(可重复读)

解决了脏读问题。该级别保证了再同一个事务中多次读取相同记录的结果一致。可重复读无法解决幻读问题。

事务未提交时反复读取的同一数据一致。

是MySQL的默认事务隔离级别。

#### SERALIZABLE（串行化）

最高隔离级别，强制事务串行执行。避免了幻读问题。串行化会在读取的每一行数据加锁。

### MVCC（多版本并发控制解决幻读）

InnoDB的MVCC，是采用了乐观锁的机制。在每行记录后，保存了两个隐藏的列来实现的。一列保存了创建时间，一列保存过期时间（或删除时间）。存储的内容为系统版本号。仅读已提交和可重复读支持MVCC

#### SELECT

​     InnoDb会根据两个条件进行检索：

1. ​     早于当前事务版本的数据行
2. ​     行的删除版本未定义，或者删除版本大于当前事务的版本号

#### INSERT

   为新插入的每一行保存当前系统版本号作为行版本号

#### DELETE

   为删除的每一行保存当前系统版本号作为行删除标识

#### UPDATE

  插入一行新的记录，保存当前系统版本号作为行版本号，同时保存当前系统版本号到原来行的删除标识。

## MySQL存储引擎

5.5版本之前默认的MyISAM存储引擎。现在默认的是Innodb存储引擎。

### MyISAM

MySQL 5.5 之前，MyISAM 引擎是 MySQL 的默认存储引擎。

MyISAM 不支持事务和行级锁，而且最大的缺陷就是崩溃后无法安全恢复。不支持外键

### Innodb

 支持行级锁(row-level locking)和表级锁,默认为行级锁,支持事务。支持崩溃后恢复。支持外键。

## 数据类型

### 数字类型

数字类型分为整数类型和实数类型两种。

#### 整数类型

| 类型          | 大小     | 范围（有符号）                                  | 范围（无符号）                        | 用途    |
| ----------- | ------ | ---------------------------------------- | ------------------------------ | ----- |
| TINYINT     | 1Bytes | (-128,127)                               | (0,255)                        | 小整数值  |
| SMALLINT    | 2Bytes | (-32768，32767）                           | (0，65535）                      | 大整数值  |
| MEDIUMINT   | 3Bytes | (-8 388 608，8 388 607)                   | (0，16 777 215)                 | 大整数值  |
| INT或INTEGER | 4Bytes | (-2 147 483 648，2 147 483 647)           | (0，4 294 967 295)              | 大整数值  |
| BIGINT      | 8Bytes | (-9,223,372,036,854,775,808，9 223 372 036 854 775 807) | (0，18 446 744 073 709 551 615) | 极大整数值 |

整数类型有UNSIGNED属性表示不允许为负值。

#### 实数类型

| 类型      | 大小                              | 范围（有符号）                                  | 范围（无符号）                                  | 用途      |
| ------- | ------------------------------- | ---------------------------------------- | ---------------------------------------- | ------- |
| FLOAT   | 4Bytes                          | (-9,223,372,036,854,775,808，9 223 372 036 854 775 807) | 0，(1.175 494 351 E-38，3.402 823 466 E+38) | 单精度浮点数值 |
| DOUBLE  | 8 Bytes                         | (-1.797 693 134 862 315 7 E+308，-2.225 073 858 507 201 4 E-308)，0，(2.225 073 858 507 201 4 E-308，1.797 693 134 862 315 7 E+308 | 0，(2.225 073 858 507 201 4 E-308，1.797 693 134 862 315 7 E+308) | 双精度浮点数值 |
| DECIMAL | 对DECIMAL(M,D) ，如果M>D，为M+2否则为D+2 | 依赖于M和D的值                                 | 依赖于M和D的值                                 | 小数值     |

### 字符串类型

#### VARCHAR和CHAR类型

VARCHAR类型和CHAR类型的主要区别在于CHAR是定长。VARCHAR是可变长度的字符串。

| 类型         | 大小                    | 用途               |
| ---------- | --------------------- | ---------------- |
| CHAR       | 0-255 bytes           | 定长字符串            |
| VARCHAR    | 0-65535 bytes         | 变长字符串            |
| TINYBLOB   | 0-255 bytes           | 不超过255个字符的二进制字符串 |
| TINYTEXT   | 0-255 bytes           | 短文本字符串           |
| BLOB       | 0-65 535 bytes        | 二进制形式的长文本数据      |
| TEXT       | 0-65 535 bytes        | 长文本数据            |
| MEDIUMBLOB | 0-16 777 215 bytes    | 二级制形式的中等长度文本     |
| MEDIUMTEXT | 0-16 777 215 bytes    | 中等长度文本           |
| LONGBLOB   | 0-4 294 967 295 bytes | 二进制形式的极大文本数据     |
| LONGTEXT   | 0-4 294 967 295 bytes | 极大文本数据           |

### 时间类型

| 类型        | 大小（bytes） | 格式                  | 用途           |
| --------- | --------- | ------------------- | ------------ |
| DATE      | 3         | YYYY-MM-DD          | 日期值          |
| TIME      | 3         | HH:MM:SS            | 时间值或持续时间     |
| YEAR      | 1         | YYYY                | 年份值          |
| DATETIME  | 8         | YYYY-MM-DD hh:mm:ss | 混合日期和时间值     |
| TIMESTAMP | 4         | YYYY-MM-DD hh:mm:ss | 混合日期和时间值，时间戳 |

## 索引

### 索引的原理

执行一条语句 SELECT * FROM USER WHERE  USERNAME='aa'.  USERNAME 上加索引。这时，查下原理如下：

- 先根据USERNAME 检索到 值为aa的行。返回他的主键和当前列的信息。
- 根据返回的主键信息，查询对应行的所有列数据。这个过程也称为回表。即，查询的数据中包含非索引的列，需要根据主键来检索出所有数据的过程。

InnoDB有两大索引：聚簇索引和普通索引

### 聚簇索引（clustered index）

聚簇索引就是按照表的主键构造一颗B+树，同时叶子节点存放整张表的行记录数据。也将叶子节点称为数据页。聚簇索引的叶子节点就是数据节点。每张表只能有一个聚簇索引。

- 如果表定义了主键，则主键为聚簇索引
- 如果表没有定义主键，则第一个非空唯一索引列就是聚簇索引
- 否则，会建立隐式的主键作为聚簇索引

#### 优点

理论上比非聚簇索引检索要快，因为可以直接通过索引检索到目标数据，比普通索引少了一次检索过程。

由于定义了数据的逻辑顺序，聚簇索引可以快算的检索范围内的数据，进行范围查找。

#### 聚簇索引是否是物理连续的

聚簇索引并非物理连续的，而是逻辑上连续的。数据页之间通过双向链表连接，按照主键排序。数据页内的数据是按照单向链表存储的。可以不按照主键顺序存储。

### 普通索引（secondary index）

非聚簇索引，也称二级索引和辅助索引。可以有多个，我们日常工作中添加的索引都是辅助索引。InnoDB的普通索引的叶子节点存储的是主键的值（聚簇索引）。MyIsAM的普通索引存的是记录指针。

### 回表示例

#### 创建一张表

```sql
CREATE TABLE t_back_to_table (
	id INT PRIMARY KEY,
	drinker_id INT NOT NULL,
	drinker_name VARCHAR (15) NOT NULL,
	drinker_feature VARCHAR (15) NOT NULL,
	INDEX (drinker_id)
) ENGINE = INNODB;
```

#### 插入数据

```sql
INSERT INTO t_back_to_table ( id, drinker_id, drinker_name, drinker_feature ) VALUES 
( 1, 2, '广西-玉林', '喝到天亮' ), 
( 2, 1, '广西-河池', '白酒三斤半啤酒随便灌' ), 
( 3, 3, '广西-贵港', '喝到晚上' ), 
( 4, 4, '广西-柳州', '喝酒不吃饭' );
```

#### 聚簇索引示意图

![](/picture/MySQL/index.jpg)

创建的表的索引构成如上图所示，通过主键检索时，会先确定主键所属范围，之后在向下层检索。最终检索到叶子节点，通过主键ID获取到对应行的记录，返回。发生了三次IO。MySQL每次IO会检索一页数据。

#### 非聚簇索引示意图

![](/picture/MySQL/secondaryIndex.jpg)

根据上图可以看出，非聚簇索引的叶子节点存放的是主键ID。

#### 无回表示例

```sql
SELECT * FROM t_back_to_table WHERE id = 3;
```

#### 回表示例

```sql
SELECT * FROM t_back_to_table WHERE drinker_id = 3;
```

通过使用 drinker_id 这个辅助索引。先获取到主键id。再通过主键id获取到该行数据。发生了回表

### 索引覆盖

#### 是什么？

索引覆盖，即仅仅通过检索索引，就可以获取到所要数据信息。不必读取行数据。即为索引覆盖。

- 使用覆盖索引，只需要从索引中就能检索到需要的数据，而不要再扫描数据表（索引为select 列）
- 索引的体量往往要比数据表小很多，因此只读取索引速度会非常快，也会极大减少数据访问量；

- MySQL的查询优化器会在执行查询前判断，是否有一个索引可以覆盖所有的查询列；

- 并非所有类型的索引都可以作为覆盖索引，覆盖索引必须要存储索引列的值。像哈希索引、空间索引、全 文索引等并不会真正存储索引列的值

**判断是否使用了覆盖索引**：

当一个查询使用了覆盖索引，在查询分析器EXPLAIN的Extra列可以看到“Using index” 。

#### 如何实现

通常的做法将需要检索的列创建在联合索引中，通过使用联合索引，在查询这些列时，会实现索引覆盖。

### 索引下推

#### 是什么？

索引下推，英文全程为index condition pushdown 简称ICP。5.6版本的MySQL推出的功能。在使用非主键索引进行检索时，会根据索引检索到数据，在SQL SERVER 层进行条件过滤。判断是否符合查询条件。当存在索引的列作为查询条件时，MySQL server 会将这一部分的判断条件传递给存储引擎。存储引擎会根据这些条件做筛选。之后通过回表操作返回结果。

**判断是否使用了索引下推：**

当一个查询使用了覆盖索引，在查询分析器EXPLAIN的Extra列可以看到extra = Using index condition 表示索引下推

#### 查询是否开启

```sql
select @@optimizer_switch
```

#### 设置开启

```sql
set ="index_condition_pushdown=off";
set ="index_condition_pushdown=on";
```

#### 如何实现

因为索引下推是优化了回表的过程。所以只适用于二级索引。且必须有回表，即查询语句不能出现索引覆盖。

##### 创建一张表

```sql
CREATE TABLE test_user (
	id INT,
	`NAME` VARCHAR (50) NOT NULL,
	age INT NOT NULL,
     grade varchar(50) NOT NULL,
	PRIMARY KEY (id),
	UNIQUE KEY `name_age` (`NAME`, `age`)
) ENGINE = INNODB;
```

##### 插入数据

```sql
INSERT INTO test_user
VALUES
	(1, '小明', 10,'15'),
	(2, '大明', 18,'20'),
	(3, '小红', 11,'25'),
	(4, '大红', 18,'30')
```

###### 执行查询

```sql
select * from test_user where name like '小%' and age=10;
```

![](/picture/MySQL/indexConditionPush.jpg)

### 数据结构划分索引类型

#### B+Tree

- 叶子节点存放数据和索引，非叶子节点只存放指针和索引
- 叶子节点构成一个双向循环链表

#### 哈希索引

基于哈希表实现。只有精确匹配索引所有的列的查询才会生效。每一行数据，存储引擎会对所有的索引列计算出一个hashCode。哈希索引将所有的哈希码保存在索引中，同时在哈希表中记录指向每个数据行的指针。

##### 优点

索引结构紧促，查询快。

##### 缺点

- 哈希索引只包含了哈希值和行指针，不存储字段值。不能使用索引中的值避免读取行。
- 无法排序
- 不支持部分索引列匹配。
- 只支持等值比较查询，不支持模糊查询及多列的最左前缀匹配。
- 哈希冲突多的情况下索引维护代价高。

#### 空间索引（R-TREE）

MyISAM支持空间索引，仅支持geometry类型。优势在于范围查找。

#### 全文索引

MyISAM支持全文索引。只有CHAR、VARCAHR、TEXT上可以创建。

### 最左前缀原则

#### 是什么？

对于索引中的字段，MySQL会一直向右匹配，直到遇到范围查询就停止匹配。

#### 如何实现

当创建一个（a,b,c）的联合索引时。如果想要索引生效，只能有a,a,b和a,b,c三种组合。b,c不会走索引。

如果查询条件为a=1 and b>2 and c=3。那么a，b两个字段可以走索引。c不能走索引

### 前缀索引

#### 是什么？

前缀索引是指针对文本或者字符串的前几个字符创建索引，这样索引长度短，检索快。主要是为了解决在长字符列上创建索引检索慢的问题。

#### 如何创建

创建的原则是选取足够长的前缀以保证较高的索引选择性。索引的选择性越高，可过滤的数据行就越多。

#### 实现

##### 计算全列选择性

可以通过以下SQL来计算选择性

```sql
select count(distinct cloumn_name)/count(*) from table_name;
```

##### 计算定长前缀的选择性

```sql
select count(distinct left(column_name,prefix_length))/count(*) from table_name;
```

原则是接近全列选择性即可。

##### 创建前缀索引

```sql
alter table system_user add index user_uuid_index(user_uuid(10));
```

### 索引设计原则

- 使用区分度较高的字段作为索引。使用区分度低的字段，例如性别作为索引。查询效果就会很差
- 尽量使用短索引，对于较长字符串的索引创建，可以指定长度较短的前缀作为索引。
- 索引不是越多越好。
- 利用最左前缀原则

### 索引失效的场景

- 对于组合索引，如果不是使用最左边的字段，不会使用索引
- 以%开头的字段无法使用索引。
- 隐式转换导致索引失效
- 索引列进行运算

### MySQL查询需要进行几次IO

#### 一次IO的计算方式

MySQL查询数据库时，不论读一行，还是读多行，都是将这些行所在的整页数据加载，然后在内存中匹配过滤出最终结果。即一次页加载就是一次IO。

如果主键索引有3层。二级索引有3层。那么一次二级索引的条件检索就是6次IO

## MySQL锁机制

MySQL根据为了解决并发问题、数据安全问题，使用了锁机制。可以按照颗粒度分为行级锁和表级锁。

### 表级锁

Mysql中锁定粒度最大 的一种锁，对当前操作的整张表加锁，实现简单 ，资源消耗也比较少，加锁快，不会出现死锁 。其锁定粒度最大，触发锁冲突的概率最高，并发度最低，MyISAM和 InnoDB引擎都支持表级锁。按照使用方法分为共享锁和排他锁

#### 发生情况

- 锁表发生在insert update 、delete 中
- 表的原理是 数据库使用独占式封锁机制，当执行上面的语句时，对表进行锁住，直到发生commite 或者 回滚 或者退出数据库用户；
- 锁表的原因 ：

1. A程序执行了对 tableA 的 insert ，并还未 commite时，B程序也对tableA 进行insert 则此时会发生资源正忙的异常 就是锁表；
2. 锁表常发生于并发而不是并行（并行时，一个线程操作数据库时，另一个线程是不能操作数据库的，cpu 和i/o 分配原则）

#### 解除办法

**查看进程id，然后用kill id杀掉进程**

```sql
show processlist;
SELECT * FROM information_schema.PROCESSLIST；
```

**查看正在执行的进程**

```sql
SELECT * FROM information_schema.PROCESSLIST where length(info) >0 ;
```

**查看是否锁表**

```sql
show OPEN TABLES where In_use > 0;
```

**查看被锁住的表**

```sql
SELECT * FROM INFORMATION_SCHEMA.INNODB_LOCKS;
```

**杀掉锁表进程**

```sql
kill  5601
```

#### 实现

##### 共享锁

```sql
LOCK TABLE table_name  READ
```

##### 排他锁

```sql
LOCK TABLE table_name WRITE
```

##### 解锁

```sql
UNLOCK TABLE;
```

### 行级锁

行级锁是Mysql中锁定粒度最细的一种锁，表示只针对当前操作的行进行加锁。行级锁能大大减少数据库操作的冲突。其加锁粒度最小，但加锁的开销也最大。有可能会出现死锁的情况。

#### 实现

##### 共享锁

```sql
select ... lock in share mode;
```

##### 排他锁

```sql
select ... for update;
```

需要配合事务使用，事务提交。锁释放。

### 悲观锁

悲观控制并发（Pessimistic Concurrency Control）简称PCC。可以阻止一个事务以影响其他用户的方式修改数据。如果一个事务对数据进行加锁。只有当这个事务把锁释放，其他事务才可以进行操作。

悲观锁顾名思义，对事务操作持有保守（悲观）的态度。在数据修改过程中，数据将处于加锁状态。

#### InnoDB中的实现

```sql
begin;
select .. for update;
commit;
```

#### 简单用例

```sql
#窗口1
set autocommit=0;（先关闭mysql中的autocommit）
select * from user for update;
#窗口2
update test_user SET age=22  where id=1;
```

在Navicat的操作窗口分别执行这两条语句。发现窗口2的语句一直未执行完成。这是因为窗口1的事务未提交，未释放锁，窗口2的事务无法操作数据。

当在窗口1中执行commit之后， 窗口2的sql可以顺利完成执行。因为窗口1的事务提交。锁也随之释放。

#### 优点与不足

悲观锁采用了先加锁后访问的机制，为数据的安全性提供了保障。但是在效率方面，由于加锁机制会产生额外的性能开销，并且增加了死锁的几率。降低了并发性。

### 乐观锁

乐观并发控制（又名“乐观锁”，Optimistic Concurrency Control，缩写“OCC”），假设多并发的用户事务处理数据时不会发生冲突，彼此之间不会有影响。在事务提交之前，该事务会检测是否有其他事务修改了数据。没有就提交，有就回滚。

乐观锁（ Optimistic Locking ） 相对悲观锁而言，乐观锁假设认为数据一般情况下不会造成冲突，所以在数据进行提交更新的时候，才会正式对数据的冲突与否进行检测，如果发现冲突了，则让返回用户错误的信息，让用户决定如何去做。

#### 乐观锁的实现

乐观锁的实现主要有两种方式：

- 通过使用版本号（version）。每次取出的版本号+1进行更新。更新的同时判断当前数据的版本号与取出的版本号是否一致，一致则认为没有冲突。正常提交，不一致则认为存在冲突或者过期。回滚事务。
- 通过添加时间戳，比较时间戳来进行冲突判定。逻辑与第一种方式基本一致。

#### 优点与不足

 乐观并发控制相信事务之间的数据竞争(data race)的概率是比较小的，因此尽可能直接做下去，直到提交的时候才去锁定，所以不会产生任何锁和死锁。但如果直接简单这么做，还是有可能会遇到不可预期的结果，例如两个事务都读取了数据库的某一行，经过修改以后写回数据库，这时就遇到了问题。

## MySQL安装

### 压缩包安装

#### 前往官网下载对应Linux所需的安装包

[官网地址](https://www.mysql.com/)

![](/picture/MySQL/MySQL_downLoad_01.jpg)

![](/picture/MySQL/MySQL_downLoad_02.jpg)

![](/picture/MySQL/MySQL_downLoad_03.jpg)

#### 上传压缩包并解压

```shell
cd  /data/
tar -xvf   mysql-5.7.39-el7-x86_64.tar.gz  
mv   mysql-5.7.39-el7-x86_64    mysql 
mkdir   /data/mysql/mysql/data
```

#### 创建mysql用户组和修改权限

```shell
groupadd  mysql  
useradd -g   mysql mysql
chown -R   mysql.mysql /usr/local/mysql/ 
```

#### 执行初始化命令

```shell
[root@localhost mysql] ./bin/mysql_install_db --user=mysql --basedir=/usr/local/mysql/ --datadir=/usr/local/mysql/data/ 
```

#### mysql的服务脚本放到系统服务中

```shell
cp -a ./support-files/mysql.server /etc/init.d/mysqld 
```

#### 修改my.cnf配置

```shell
[mysql]
# 设置mysql客户端默认字符集
default-character-set=utf8mb4
[mysqld]
#设置端口，默认为3306
port = 3306
#设置数据库默认不区分大小写
lower_case_table_names=1
# 设置mysql的安装目录
basedir=/usr/local/mysql
# 设置mysql数据库的数据的存放目录
datadir=/usr/local/mysql/data
# 允许最大连接数
max_connections=200
# 服务端使用的字符集默认为8比特编码的latin1字符集
character-set-server=utf8
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB
sql_mode = "STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION,NO_ZERO_DATE,NO_ZERO_IN_DATE,ERROR_FOR_DIVISION_BY_ZERO"
# 允许最大连接数
max_connections=10000
# 允许连接失败的次数。这是为了防止有人从该主机试图攻击数据库系统
max_connect_errors=10
# 默认使用“mysql_native_password”插件认证
default_authentication_plugin=mysql_native_password
server-id=10
max_allowed_packet =16777216
#跳过密码
skip-grant-tables
```

#### 启动MySQL

```shell
service mysql start  
service mysql restart  
```

#### 修改密码

```shell
mysql -u root mysql 
update mysql.user set password=password('新密码') where user='root' and host='localhost';
```

#### 创建同步连接

```shell
#创建快捷方式： 服务启动后，直接运行mysql -u root -p即可登录，不需要进入到对应的目录
ln -s /usr/local/mysql/bin/mysql /usr/bin  
```

#### 配置远程登录

```shell
use mysql;
update user set user.Host='%' where user.User='root';
update user set authentication_string=password('root') where user='root';
flush privileges;

#解决缺少libncurses.so.5文件的报错
ln -s /usr/lib64/libncurses.so.6 /usr/lib64/libncurses.so.5
```



## 主备配置

### 准备两台服务器

一台作为主服务器、一台作为备份服务器

### 主服务器配置

```shell
[root@master ~]# vim /etc/my.cnf
#在[mysqld]中添加
#启用二进制日志 
log-bin= mysql-bin-master 
#本机数据库 ID 标示，主从配置中ID要唯一
server-id= 1 
#可以被从服务器复制的库, 二进制需要同步的数据库名
binlog-do-db= test 
#如果有多个数据库，需要重复配置，不能直接在后面用逗号增加，否则mysql 会把这里当成一个数据库，会有坑
#binlog-do-db= test2 
#binlog-do-db= test3 
#注释掉 binlog_do_db 和 binlog_ignore_db ，则表示备份全部数据库
#不可以被从服务器复制的库
binlog-ignore-db= mysql 
#保存并重启MySQL服务(如果重启卡死现象，kill 掉再启动) 
[root@master ~]# service mysql restart
```

### 授权主从同步slave用户权限

```shell
[root@master ~]# mysql -uroot -p'root123'
mysql> grant replication slave on *.* to slave@192.168.183.139 identified by "root123"; 
#ip地址为从库的IP  创建slave账号slave，密码root123
Query OK, 0 rows affected, 1 warning (0.00 sec)
mysql> show master status; #查看主服务器状态信息
mysql> show binlog events\G;  #展示相关状态

```

### MySQL从服务器配置

```shell
[root@slave ~]# vim /etc/my.cnf
#在[mysqld]中添加
#从服务器没必要开启bin-log日志，
server-id=2 
#从服务器ID号，不要和主ID相同，如果设置多个从服务器，每个从服务器必须有一个唯一的server-id值，必须与主服务器的以及其它从服务器的不相同。
#可以认为 server-id值类似于IP地址：这些ID值能唯一识别复制服务器群集中的每个服务器实例。
#重启从服务器MySQL服务
[root@slave ~]# service mysql restart
#从服务器设置主服务器配置
[root@slave ~]# mysql -uroot -proot123
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
#授权
mysql> change master to master_host='192.168.183.140',master_user='slave',master_password='123456'; #IP地址为主服务器地址。
Query OK, 0 rows affected, 2 warnings (0.01 sec)
mysql> start slave; #启动slave
mysql> show slave status\G； #查看状态
```

