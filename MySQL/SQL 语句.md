# SQL 语句

## DISTINCT

"Orders"表：

| Company  | OrderNumber |
| :------- | :---------- |
| IBM      | 3532        |
| W3School | 2356        |
| Apple    | 4698        |
| W3School | 6953        |

```sql
SELECT DISTINCT Company FROM Orders 
```

结果：

| Company  |
| :------- |
| IBM      |
| W3School |
| Apple    |

## WHERE

| 操作符  | 描述         |
| :------ | :----------- |
| =       | 等于         |
| <>      | 不等于       |
| >       | 大于         |
| <       | 小于         |
| >=      | 大于等于     |
| <=      | 小于等于     |
| BETWEEN | 在某个范围内 |
| LIKE    | 搜索某种模式 |

SQL 使用单引号来环绕*文本值*（大部分数据库系统也接受双引号）

## ORDER BY

对结果集进行排序，默认按照升序对记录进行排序。需要降序 使用 DESC。

以逆字母顺序显示公司名称，并以数字顺序显示顺序号

```sql
SELECT Company, OrderNumber FROM Orders ORDER BY Company DESC, OrderNumber ASC
```

结果：

| Company  | OrderNumber |
| :------- | :---------- |
| W3School | 2356        |
| W3School | 6953        |
| IBM      | 3532        |
| Apple    | 4698        |

## INSERT INTO

语法

```sql
INSERT INTO 表名称 VALUES (值1, 值2,....)
```

指定插入的列值

```sqlite
INSERT INTO table_name (列1, 列2,...) VALUES (值1, 值2,....)
```

## Update

语法：

```sql
UPDATE 表名称 SET 列名称 = 新值 WHERE 列名称 = 某值
```

## DELETE

语法

```sql
DELETE FROM 表名称 WHERE 列名称 = 值
```

## 通配符

| 通配符                     | 描述                       |
| :------------------------- | :------------------------- |
| %                          | 替代一个或多个字符         |
| _                          | 仅替代一个字符             |
| [charlist] 如[abcd]        | 字符列中的任何单一字符     |
| [^charlist]或者[!charlist] | 不在字符列中的任何单一字符 |

## IN

IN 操作符允许我们在 WHERE 子句中规定多个值。

```sql
SELECT column_name(s)
FROM table_name
WHERE column_name IN (value1,value2,...)
```

## BETWEEN

操作符 BETWEEN ... AND 会选取介于两个值之间的数据范围。这些值可以是数值、文本或者日期。

```sql
SELECT column_name(s)
FROM table_name
WHERE column_name
BETWEEN value1 AND value2
```

## Alias

通过使用 SQL，可以为列名称和表名称指定别名（Alias）。

```sql
SELECT column_name AS alias_name
FROM table_name
```

## JOIN

```sql
SELECT Persons.LastName, Persons.FirstName, Orders.OrderNo
FROM Persons
INNER JOIN Orders
ON Persons.Id_P = Orders.Id_P
ORDER BY Persons.LastName
```

- INNER JOIN（内连接），返回有关联的行

- JOIN: 如果表中有至少一个匹配，则返回行，等同于内连接

- LEFT JOIN: 即使右表中没有匹配，也从左表返回所有的行

- RIGHT JOIN: 即使左表中没有匹配，也从右表返回所有的行
- FULL JOIN: 只要其中一个表中存在匹配，就返回行。两张表中所有数据都显示，实际就是inner +(left-inner)+(right-inner)

## UNION

UNION 操作符用于合并两个或多个 SELECT 语句的结果集。

请注意，UNION 内部的 SELECT 语句必须拥有相同数量的列。列也必须拥有相似的数据类型。同时，每条 SELECT 语句中的列的顺序必须相同。

```sql
SELECT column_name(s) FROM table_name1
UNION
SELECT column_name(s) FROM table_name2
```

另外，UNION 结果集中的列名总是等于 UNION 中第一个 SELECT 语句中的列名。

## SELECT INTO

SELECT INTO 语句从一个表中选取数据，然后把数据插入另一个表中。

SELECT INTO 语句常用于创建表的备份复件或者用于对记录进行存档。

```sql
SELECT *
INTO new_table_name [IN externaldatabase] 
FROM old_tablename
```

例子

```sql
SELECT LastName,Firstname
INTO Persons_backup
FROM Persons
WHERE City='Beijing'
```

```sql
SELECT Persons.LastName,Orders.OrderNo
INTO Persons_Order_Backup
FROM Persons
INNER JOIN Orders
ON Persons.Id_P=Orders.Id_P
```

