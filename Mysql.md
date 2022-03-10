# 什么是事务？

- 事务（Transaction），一般是指要做的或所做的事情。在计算机术语中是指访问并可能更新数据库中各种数据项的一个程序执行单元(unit)。事务通常由高级数据库操纵语言或编程语言（如SQL，C++或Java）书写的用户程序的执行所引起，并用形如begin transaction和end transaction语句（或函数调用）来界定。事务由事务开始(begin transaction)和事务结束(end transaction)之间执行的全体操作组成。

# 数据库中的事务概念

- 数据库事务( transaction)是访问并可能操作各种数据项的一个数据库操作序列，这些操作要么全部执行，要么全部不执行，是一个不可分割的工作单位。事务由事务开始与事务结束之间执行的全部数据库操作组成。

# 事务的ACID原则

- 数据库管理系统中事务(transaction)的四个特性：原子性（Atomicity）、一致性（Consistency）、隔离性（Isolation）、持久性（Durability）
  1. 原子性
     - 原子性是指事务是一个不可再分割的工作单元，事务中的操作要么都发生，要么都不发生。
  2. 一致性
     - 一致性是指在事务开始之前和事务结束以后，数据库的完整性约束没有被破坏。也就是说，数据库事务不能破坏<u>关系数据的完整性</u>以及<u>业务逻辑上的一致性</u>。
  3. 隔离性
     - 多个事务并发访问时，事务之间是隔离的，一个事务不应该影响其他事务运行效果。
     - 在并发环境中，当不同的事务同时操纵相同的数据时，每个事务都有各自的完整数据空间。由编发事务所作出的修改，必须与任何其他并发事务所作出的修改隔离。事务查看数据更新时，数据所处的状态要么是另一个事务修改它之前的状态，要么是另一个事务修改它之后的状态，事务不会查看到中间状态的数据。
     - 事务中最复杂的问题都是由事务隔离性引起的。完全的隔离性是不现实的，完全的隔离性要求数据库在同一时间只执行一条事务，这样会严重影响性能。
  4. 持久性
     - 在事务完成以后，该事务对数据库所作出的更改，是持久保存在数据库中，并不会被回滚。完成的事务是系统永久的部分，对系统的影响也是永久性的，该修改即便在系统遇到致命故障时也将一直保持。


# MySql中的事务管理

## 事务的控制语句

- Mysql中通常使用一些方式处理事务：
  - 使用BEGIN、ROLLBACK、COMMIT实现；
  - 直接使用SET来改变Mysql的自动提交模式（默认是自动提交）：
    - SET AUTOCOMMIT=0，禁止自动提交
    - SET AUTOCOMMIT=1，开启自动提交

1. 设置事务隔离级别：
   - 查看当前会话隔离级别：SELECT @@tx_isolation;
   - 查看系统当前隔离级别：SELECT @@global.tx_isolation;
   - 设置当前会话隔离级别：SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ;
   - 设置系统当前隔离级别：SET GLOBAL TRANSACTION ISOLATION LEVEL REPEATABLE READ;
2. 开启事务：BEGIN或START TRANSACTION显式地开启一个事务，二者等价；
3. 提交事务：COMMIT或COMMIT WORK提交事务，二者等价；
4. 回滚事务：ROLLBACK或ROLLBACK WORK，二者等价；
5. 保存点：SAVEPOINT identifier，SAVEPOINT允许在事务中创建一个保存点，一个事务中可以有多个SAVEPOINT；
6. 删除保存点：RELEASE SAVEPOINT identifier，可以删除一个保存点，如果当前没有指定任何保存点，执行该语句会抛出一个异常；
7. 回滚到标记点：ROLLBACK TO identifier，把事务回滚到标记点。

## 事务并发下产生的问题

### 更新丢失

- 两个事务都同时更新一行数据，一个事务对数据的更新把另一个事务对数据的更新覆盖了。这是因为系统没有执行任何的锁操作，因此并发事务并没有被隔离开来。

### 脏读

- 一个事务读取到了另一个事务未提交的数据操作结果。这是相当危险的，因为很可能所有的操作都被回滚。

|              会话1               | 值1  |               会话2               | 值2  |
| :------------------------------: | :--: | :-------------------------------: | :--: |
|              BEGIN               |  -   |               BEGIN               |  -   |
|                -                 |  -   | SELECT age FROM user WHERE id=1;  |  5   |
|                -                 |  -   | UPDATE user SET age=10 WHER id=1; |  √   |
| SELECT age FROM user WHERE id=1; |  10  |                 -                 |  -   |
|              COMMIT              |  √   |             ROLLBACK              |  √   |
|                -                 |  -   |              COMMIT               |  √   |



### 不可重复读

- 一个事务对同一行数据重复读取两次，但是却得到了不同的结果。
- 虚读：事务T1读取某一数据后，事务T2对其做了修改，当事务T1再次读该数据时得到与前一次不同的值。

|              会话1               | 值1  |               会话2               | 值2  |
| :------------------------------: | :--: | :-------------------------------: | :--: |
|              BEGIN               |  √   |               BEGIN               |  √   |
| SELECT age FROM user WHERE id=1; |  5   |                 -                 |  -   |
|                -                 |  -   | UPDATE user SET age=10 WHER id=1; |  √   |
|                -                 |  -   |              COMMIT               |  √   |
| SELECT age FROM user WHERE id=1; |  10  |                 -                 |  -   |
|              COMMIT              |  √   |                 -                 |  -   |

- 幻读：事务在操作过程中进行两次查询，第二次查询的结果包含了第一次查询中未出现的数据或者缺少了第一次查询中出现的数据（这里并不要求两次查询的SQL语句相同）。这是因为在两次查询过程中有另外一个事务插入或删除数据造成的。

|            会话1            | 值1  |                       会话2                       | 值2  |
| :-------------------------: | :--: | :-----------------------------------------------: | :--: |
|            BEGIN            |  √   |                       BEGIN                       |  √   |
| SELECT COUNT(id) FROM user; |  3   |                         -                         |  -   |
|              -              |  -   | INSERT INTO user(id, username) VALUE(6, "dylan"); |  √   |
|              -              |  -   |                      COMMIT                       |  √   |
| SELECT COUNT(id) FROM user; |  4   |                         -                         |  -   |
|           COMMIT            |  √   |                         -                         |  -   |



### 幻读

- 这里的幻读是不可重复读的一种特殊情况。主要偏向于插入和删除数据。

|                       会话1                       | 值1  |                       会话2                       | 值2  |
| :-----------------------------------------------: | :--: | :-----------------------------------------------: | :--: |
|                       BEGIN                       |  √   |                       BEGIN                       |  √   |
|               SELECT \* FROM user;                |  √   |                         -                         |  -   |
|                         -                         |  -   | INSERT INTO user(id, username) VALUE(2, "dylan"); |  √   |
|                         -                         |  -   |                         -                         |  -   |
|                         -                         |  -   |                      COMMIT                       |  √   |
| INSERT INTO user(id, username) VALUE(2, "dylan"); |  ×   |                         -                         |  -   |
|               SELECT \* FROM user;                |  √   |                         -                         |  -   |
| INSERT INTO user(id, username) VALUE(2, "dylan"); |  ×   |                         -                         |  -   |
|                     ROLLBACK                      |  √   |                         -                         |  -   |



## 事务的隔离级别

Mysql中使用InnoDB存储引擎，提供的事务隔离级别有4种：

### READ UNCOMMITTED（读未提交）

- 允许脏读取，但不允许更新丢失。如果一个事务已经开始写数据，则另外一个事务则不允许同时进行写操作，但允许其他事务读此行数据。该隔离级别可以通过“排他写锁”实现。

### READ COMMITTED（读提交）

- 允许不可重复读取不可重复读取)，但不允许脏读取。这可以通过“瞬间共享读锁”和“排他写锁”实现。读取数据的事务允许其他事务继续访问该行数据，但是未提交的写事务将会禁止其他事务访问该行。

### REPEATABLE READ（可重复读）

- 禁止不可重复读取和脏读取，但是有时可能出现幻读数据。这可以通过“共享读锁”和“排他写锁”实现。读取数据的事务将会禁止写事务（但允许读事务），写事务则禁止任何其他事务。

### SERIALIZABLE（串行化）

- 提供严格的事务隔离。它要求事务序列化执行，事务只能一个接着一个地执行，不能并发执行。仅仅通过“行级锁”是无法实现事务序列化的，必须通过其他机制保证新插入的数据不会被刚执行查询操作的事务访问到。

| 事务隔离级别                 | 脏读 | 不可重复读 | 幻读 |
| ---------------------------- | ---- | ---------- | ---- |
| READ UNCOMMITTED（读未提交） | 是   | 是         | 是   |
| READ COMMITTED（读提交）     | 否   | 是         | 是   |
| REPEATABLE READ（可重复读）  | 否   | 否         | 是   |
| SERIALIZABLE（串行化）       | 否   | 否         | 否   |

## MySQL中的REPEATABLE READ

- Mysql中的REPEATABLE READ是解决了部分幻读的问题：

  - 它可以保证事务1在仅进行查询的情况下，无论事务2对同样的表进行何种增删改操作，事务1查询到的表数据都是一致的；

  - 如果事务1开启，此时事务2开启并对表进行了增删改操作，之后事务2被提交，事务1紧接着进行增删改操作，并涉及到需要刷新行或刷新表数据的操作时，在事务2中被修改、增加或删除的数据，将有机会出现在事务1的查询中：

    - 事务1导致行刷新的操作中，包括了被事务2修改的行，那么在事务1中会出现新数据，或不变（相同修改的情况下）；
    - 事务1导致行刷新的操作中，包括了被事务2删除的行，那么该数据行将在事务1中呈现未成功更改的状况；
    - 事务1导致行刷新的操作和事务2增加行没有冲突，数据在事务1中查询结果没有变化；
    - 事务1导致表刷新的操作，会使事务2中所有修改、新增的数据出现在事务1的查询中，但事务2中被删除的数据将在事务1中呈现未成功更改的状况。

  - 如果事务1开启，此时事务2开启并对表进行了增删改操作，之后事务2被提交，事务1紧接着进行增删改操作，但并未涉及到需要刷新行或刷新表数据的操作时，会出现以下情况：

    - 事务1删除事务2已删除的行，不会报错，但数据将呈现未成功删除的状况；
- 事务1正常地新增行，并得到与预期一样的结果。

# MySql中的联结表

- SQL join字句可以基于这些表之间的共同字段，把来自两个或多个表的数据结合起来。
- 最常见的join字句类型：
  1. INNER JOIN
  2. LEFT JOIN
  3. RIGHT JOIN
  4. FULL JOIN

## 数据表准备

```mysql
CREATE TABLE username (
	id INT(10) NOT NULL AUTO_INCREMENT PRIMARY KEY,
	user_name VARCHAR(32) NOT NULL
);

CREATE TABLE useraddress (
	id INT(10) NOT NULL AUTO_INCREMENT PRIMARY KEY,
	user_address VARCHAR(32) NOT NULL
);

SELECT * FROM username;
SELECT * FROM useraddress;

INSERT INTO username(id, user_name)
VALUES
	(1, "Google"),
	(2, "Taobao"),
	(3, "Weibo"),
	(4, "Facebook");
	
INSERT INTO useraddress(id, user_address)
VALUES
	(1, "美国"),
	(5, "中国"),
	(3, "中国"),
	(6, "美国");
```

#### table.username

|  id  | user_name |
| :--: | :-------: |
|  1   |  Google   |
|  2   |  Taobao   |
|  3   |   Weibo   |
|  4   | Facebook  |

#### table.useraddress

|  id  | address |
| :--: | :-----: |
|  1   |  美国   |
|  5   |  中国   |
|  3   |  中国   |
|  6   |  美国   |

## 联结表的种类与示例

### INNER JOIN

- 内联结是最常见的一种联结，只要联结匹配即可。

![img](images/Mysql.images/1074709-20171229165319538-1026266241.png)

```mysql
# 使用where字句也可以实现内联结
SELECT username.id, username.user_name, useraddress.user_address FROM username, useraddress
WHERE username.id = useraddress.id;

# 使用inner join字句实现的内联结，inner join关键字等同于join，inner可省略
SELECT username.id, username.user_name, useraddress.user_address
FROM username
INNER JOIN useraddress
ON username.id = useraddress.id;
```

|  id  | user_name | user_address |
| :--: | :-------: | :----------: |
|  1   |  Google   |     美国     |
|  3   |   Weibo   |     中国     |



### LEFT JOIN

- LEFT JOIN返回左表的全部行和右表满足ON条件的行，如果左表的行在右表中没有匹配，那么这一行右表中对应数据用NULL代替。

![img](images/Mysql.images/1074709-20171229170434726-2010021622.png)

```mysql
# left join外联结
SELECT username.*, useraddress.user_address
FROM username
LEFT JOIN useraddress
ON username.id = useraddress.id;
```

|  id  | user_name | user_address |
| :--: | :-------: | :----------: |
|  1   |  Google   |     美国     |
|  2   |  Taobao   |  *（NULL）*  |
|  3   |   Weibo   |     中国     |
|  4   | Facebook  |  *（NULL）*  |



### RIGHT JOIN

- RIGHT JOIN返回右表的全部行和左表满足ON条件的行，如果右表的行在左表中没有匹配，那么这一行左表中对应数据用NULL代替。

![img](images/Mysql.images/1074709-20171229171503867-2027149651.png)

```mysql
# right join外联结
SELECT useraddress.*, username.user_name
FROM username
RIGHT JOIN useraddress
ON username.id = useraddress.id;
```

|  id  | user_address | user_name  |
| :--: | :----------: | :--------: |
|  1   |     美国     |   Google   |
|  3   |     中国     |   Weibo    |
|  5   |     中国     | *（NULL）* |
|  6   |     美国     | *（NULL）* |



### FULL OUTER JOIN

- MySQL不支持FULL OUTER JOIN。
- FULL JOIN 会从左表 和右表 那里返回所有的行。如果其中一个表的数据行在另一个表中没有匹配的行，那么对面的数据用NULL代替。

![img](images/Mysql.images/1074709-20171229172802179-389908324.png)

# MySql数据类型

## 数值类型

- MySql支持所有标准sql数值数据类型。
- 扩展，关于创建表时使用INT(11)。INT类型占用4个字节，也就是它的范围是：
  - -2147483648~2147483647
- 而创建表示使用的INT(11)中的11表示字符显示宽度，显而易当一个INT类型取值最多占用的也就是11个字符的宽度，不可能再多了，诸如INT(20)之类的都不是特别严谨的写法。

|      类型      |             大小（bytes）             |     用途     |
| :------------: | :-----------------------------------: | :----------: |
|    TINYINT     |                   1                   |   小整数值   |
|    SMALLINT    |                   2                   |   大整数值   |
|   MEDIUMINT    |                   3                   |   大整数值   |
| INT 或 INTEGER |                   4                   |   大整数值   |
|     BIGINT     |                   8                   |  极大整数值  |
|     FLOAT      |                   4                   | 单精度浮点数 |
|     DOUBLE     |                   8                   | 双精度浮点数 |
|    DECIMAL     | DECIMAL(M, D)，M为总位数，D为小数位数 |    小数值    |

## 日期和时间类型

- 表示时间值的日期和时间类型为DATETIME、DATE、TIMESTAMP、TIME和YEAR。

|   类型    | 大小（bytes） |        格式         |           用途           |
| :-------: | :-----------: | :-----------------: | :----------------------: |
|   DATE    |       3       |     yyyy-MM-dd      |          日期值          |
|   TIME    |       3       |      HH:mm:ss       |     时间值或持续时间     |
|   YEAR    |       1       |        yyyy         |          年份值          |
| DATETIME  |       8       | yyyy-MM-dd HH:mm:ss |     混合日期和时间值     |
| TIMESTAMP |       4       | yyyy-MM-dd HH:mm:ss | 混合日期和时间值，时间戳 |

## 字符串类型

- 字符串类型指CHAR、VARCHAR、BINARY、VARBINARY、BLOB、TEXT、ENUM和SET。
- CHAR(N)和VARCHAR(N)中的N代表字符的个数，比如也就是中文字符或英文字符的数量，不代表字节数，例：CHAR(30)可以存储30个字符。
- 根据字符集的不同，中文在MySql中所占的字节数是不一样的：
  - GBK：一个汉字占2个字节（byte）
  - UTF-8：一个汉字占3个字节（byte）
- 大多数表使用的都是UTF-8的字符集，那么一个中文占3个字节。如果使用VARCHAR类型，大约可以存储21845个汉字，但一般情况下不会存储那么多个，除非你是很长的文章。
- VARCHAR类型实际上可以存储21845个汉字，但是这受限于你定义该表字段时所定义VARCHAR(n)中的n的大小，如果n为30，那么也就是只能存30个汉字了。
- byte，一般指字节。

|    类型    | 大小（byte） |             用途              |
| :--------: | :----------: | :---------------------------: |
|    CHAR    |    0~255     |          定长字符串           |
|  VARCHAR   |   0~65535    |          变长字符串           |
|  TINYTEXT  |    0~255     |         短文本字符串          |
|    TEXT    |   0~65535    |         长文本字符串          |
| MEDIUMTEXT |  0~16777215  |       中等长度文本数据        |
|  LONGTEXT  | 0~4294967295 |         极大文本数据          |
|  TINYBLOB  |    0~255     | 不超过255个字符的二进制字符串 |
|    BLOB    |   0~65535    |    二进制形式的长文本数据     |
| MEDIUMBLOB |  0~16777215  | 二进制形式的中等长度文本数据  |
|  LONGBLOB  | 0~4294967295 |   二进制形式的极大文本数据    |

# MySql中的行锁和表锁

- 待续。