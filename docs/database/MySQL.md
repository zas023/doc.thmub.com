### 1. 基本用法

#### 1.1 CRUD

**查询数据（select）**

```mysql
SELECT
    column_1, column_2, ...
FROM
    table_1
[INNER | LEFT |RIGHT] JOIN table_2 ON conditions
WHERE
    conditions
GROUP BY column_1
HAVING group_conditions
ORDER BY column_1
LIMIT offset, length;
```

- `SELECT`之后是逗号分隔列或星号(`*`)的列表，表示要返回所有列
- `FROM`指定要查询数据的表或视图
- `JOIN`根据某些连接条件从其他表中获取数据
- `WHERE`过滤结果集中的行
- `GROUP BY`将一组行组合成小分组，并对每个小分组应用聚合函数
- `HAVING`过滤器基于`GROUP BY`子句定义的小分组
- `ORDER BY`指定用于排序的列的列表
- `LIMIT`限制返回行的数量

（`SELECT`和`FROM`语句是必须的，其他部分可选）



**选择数据（where）**

| 操作符       | 描述                                    |
| ------------ | --------------------------------------- |
| `=`          | 等于，几乎任何数据类型都可以使用它。    |
| `<>` 或 `!=` | 不等于                                  |
| `<`          | 小于，通常使用数字和日期/时间数据类型。 |
| `>`          | 大于，                                  |
| `<=`         | 小于或等于                              |
| `>=`         | 大于或等于                              |

- `BETWEEN`选择在给定范围值内的值
- `LIKE`匹配基于模式匹配的值
- `IN]`指定值是否匹配列表中的任何值
- `IS NULL`检查该值是否为`NULL`



**插入数据（insert）**

`INSERT`语句允许您将一行或多行插入到表中：

```mysql
INSERT INTO table(column1,column2...)
VALUES (value1,value2,...);

#多行插入
INSERT INTO table(column1,column2...)
VALUES (value1,value2,...),
       (value1,value2,...),
...;
```

如果为表中的所有列指定相应列的值，则可以忽略`INSERT`语句中的列列表：

```mysql
INSERT INTO table
VALUES (value1,value2,...);
```

在MySQL中，可以使用`SELECT`语句返回的列和值来填充`INSERT`语句的值：

```mysql
INSERT INTO table_1
SELECT c1, c2, FROM table_2;
```

在`INSERT`语句中指定`ON DUPLICATE KEY UPDATE`选项，MySQL将插入新行或使用新值更新原行记录。



**更新数据（update）**

```mysql
UPDATE [LOW_PRIORITY] [IGNORE] table_name
# LOW_PRIORITY修饰符指示UPDATE语句延迟更新，直到没有从表中读取数据的连接
# 即使发生错误，IGNORE修饰符也可以使UPDATE语句继续更新行。导致错误(如重复键冲突)的行不会更新。
SET
    column_name1 = expr1,
    column_name2 = expr2,
    ...
WHERE
    condition;
```

- 在`UPDATE`关键字后面指定要更新数据的表名。
- `SET`子句指定要修改的列和新值。要更新多个列，请使用以逗号分隔的列表。以字面值，表达式或子查询的形式在每列的赋值中来提供要设置的值。
- 使用`WHERE`子句中的条件指定要更新的行。`WHERE`子句是可选的。 如果省略`WHERE`子句，则`UPDATE`语句将更新表中的所有行。

```mysql
# 为每个客户安排一个销售代表
UPDATE customers
SET
    salesRepEmployeeNumber = (SELECT
            employeeNumber
        FROM
            employees
        WHERE
            jobtitle = 'Sales Rep'
        LIMIT 1)
WHERE
    salesRepEmployeeNumber IS NULL;
```



**删除数据（delete）**

```mysql
DELETE FROM table_name
WHERE condition
ORDER BY c1, c2, ...
LIMIT row_count;
```

#### 1.2 数据库

```mysql
# 创建数据库
CREATE DATABASE [IF NOT EXISTS] database_name;

# 显示数据库
SHOW DATABASES

# 使用数据库
USE database_name;

# 删除数据库
DROP DATABASE [IF EXISTS] database_name;
```

#### 1.3 创建表

```mysql
CREATE TABLE [IF NOT EXISTS] table_name(
        column_list
) engine=table_type;
```

- `column_list`部分指定表的列表。字段的列用逗号(`，`)分隔；
- 需要为`engine`子句中的表指定存储引擎(如：*InnoDB*，*MyISAM*，*HEAP*，*EXAMPLE*，*CSV*，*ARCHIVE*，*MERGE*， *FEDERATED*或*NDBCLUSTER*)。默认使用*InnoDB*。

```mysql
column_name data_type[size] [NOT NULL|NULL] [DEFAULT value] [AUTO_INCREMENT]
```

- `column_name`指定列的名称。每列具有特定数据类型和大小，例如：`VARCHAR(255)`。
- `NOT NULL`或`NULL`表示该列是否接受`NULL`值。
- `DEFAULT`值用于指定列的默认值。
- `AUTO_INCREMENT`指示每当将新行插入到表中时，列的值会自动增加。每个表都有一个且只有一个`AUTO_INCREMENT`列。

```mysql
CREATE TABLE IF NOT EXISTS tasks (
  task_id INT(11) NOT NULL AUTO_INCREMENT,
  subject VARCHAR(45) DEFAULT NULL,
  start_date DATE DEFAULT NULL,
  end_date DATE DEFAULT NULL,
  description VARCHAR(200) DEFAULT NULL,
  PRIMARY KEY (task_id)
) ENGINE=InnoDB;
```

#### 1.4 修改表

可以使用`ALTER TABLE`语句来更改现有表的结构。

**添加列**

```mysql
ALTER TABLE tasks
ADD COLUMN complete DECIMAL(2,1) NULL
AFTER description;
```

**修改列**

```mysql
ALTER TABLE tasks
CHANGE COLUMN task_id task_id INT(11) NOT NULL AUTO_INCREMENT;
```

**删除列**

```mysql
ALTER TABLE tasks
DROP COLUMN description;
```

**重命表名**

```mysql
ALTER TABLE tasks
RENAME TO work_items;
```

或

```mysql
RENAME TABLE old_table_name TO new_table_name;
```

#### 1.5 备份还原

**备份**

```mysql
mysqlump -u <username> -p <password> <database_name> ><path>
```

**还原**

```mysql
source <path>;
```



### 2. 高阶技巧

#### 2.1 多表查询

**笛卡尔积**

```mysql
select * from A, B;
```

**内连接**

- 隐式内连接：使用`where`条件消除无效数据

```mysql
select * from A a, B b where a.id=b.id;
```

- 显示内连接

```mysql
select * from A [inner] join B on A.id = B.id;
```

**外连接**

- 左连接

```mysql
select * from A left [outer] join B on A.id =B.id ;
```

- 右连接

```mysql
select * from A right [outer] join B on A.id =B.id ;
```

左右连接无本质区别，一般习惯使用左连接。

#### 2.2 事务

一个含多个步骤的业务被事务管理，这些步骤要么一起完成，否则一起失败。

```mysql
#开启事务
start trasaction;
#提交shiw
commit；
#回滚事务
rollback;
```

MySQL默认提交事务：一条DML（增删改）语句会自动提交一次事务。

```mysql
#查看事务默认提交方式
select @@autocommit;  # 1代表自动提交
#修改默认提交方式
set @@autocommit =0 ;
```

**事务的四大特征（ACID）**

| 特征                  | 描述                                         |
| --------------------- | -------------------------------------------- |
| 原子性（Atomicity）   | 事务时不可分割的最小单位，同时成功或同时失败 |
| 持久性（Consistency） | 当事务提交或回滚后，数据库会持久化的保存数据 |
| 隔离性（Isolation）   | 多个事务之间相互独立                         |
| 一致性（Durability）  | 事务操作前后，数据总量不变                   |

**事务的隔离级别**

事务时具有隔离性的，但当多个事务操作同一数据时，会引发一些问题，设置不同的隔离级别可以解决。

引发的问题：

1. 脏读：一个事务读取到另一个事务没有提交的数据；
2. 不可重复读（虚读）：在同一事务中，两次读取的数据不一致；
3. 幻读：某一次的DML操作得到的结果所表征的数据状态无法支撑后续的业务操作。

| 隔离级别                     | 脏读 | 不可重复读 | 幻读 |
| ---------------------------- | ---- | ---------- | ---- |
| 读未提交（read-uncommitted） | 是   | 是         | 是   |
| 不可重复读（read-committed） | 否   | 是         | 是   |
| 可重复读（repeatable-read）  | 否   | 否         | 是   |
| 串行化（serializable）       | 否   | 否         | 否   |

**管理事务隔离级别**

```mysql
#查询数据库事务隔离级别
select @@tx_isolaion;

#修改数据库事务隔离级别
set tx_isolation='read-committed';
#or
set global transaction isolation level <级别字符串>;
```

#### 2.3 CET

> 公共表表达式是一个命名的临时结果集，仅在单个SQL语句(例如`SELECT`，`INSERT`，`UPDATE`或`DELETE`)的执行范围内存在。

```mysql
WITH cte_name (column_list) AS (
    query
)
SELECT * FROM cte_name;

# 实例
WITH customers_in_usa AS (
    SELECT
        customerName, state
    FROM
        customers
    WHERE
        country = 'USA'
) SELECT
    customerName
 FROM
    customers_in_usa
 WHERE
    state = 'CA'
 ORDER BY customerName;
 # CTE的名称为customers_in_usa，定义CTE的查询返回两列：customerName和state。
 # 因此，customers_in_usa CTE返回位于美国的所有客户。
```

**递归CET**

#### 2.4 三大范式

**第一范式（1NF）**

每一列都是不可分割的原子数据项

**第二范式（2NF)**

在1NF的基础上，非码属性必须完全依赖于候选码。（即消除非主属性对主码 的部分函数依赖）

- **函数依赖**：A-->B，如果通过A属性（属性蔟）的值可以确定唯一B属性的值，则B依赖于A
- **完全函数依赖**：A-->B，如果A是一个属性组，则确定B属性的值需要A中所有的属性
- **部分函数依赖**：A-->B，如果A是一个属性组，则确定B属性的值需要A中部分属性
- **传递函数依赖**：A-->B，B-->C，则A-->C
- **码**：如果一张表中的某一属性或属性组被其余属性完全依赖，则称该属性（属性组）为该表的码

**第三范式（3NF）**

在2NF的基础上，任何非主属性不依赖其它非主属性。（即消除传递依赖）
