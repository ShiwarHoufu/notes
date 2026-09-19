# SQL

一门操作关系型数据库的编程语言，定义了操作所有**关系型数据库的统一标准**。

## 1. SQL 的分类

| 分类      | 全称                         | 说明                              |
| ------- | -------------------------- | ------------------------------- |
| **DDL** | Data Definition Language   | 数据**定义**语言，用来定义数据库对象（数据库、表、字段）  |
| **DML** | Data Manipulation Language | 数据**操作**语言，用来对数据库表中的数据进行增删改     |
| **DQL** | Data Query Language        | 数据**查询**语言，用来查询数据库中表的记录         |
| **DCL** | Data Control Language      | 数据**控制**语言，用来创建数据库用户、控制数据库的访问权限 |
|         |                            |                                 |

```mermaid
flowchart LR
    C[客户端] -->|SQL| D[DBMS]
    D -->|DDL| DB[(数据库)]
    DB -->|DDL| T[表]
    T <-->|DML / DQL| R[表中的记录]
```


## 2. DQL 查询语句结构

完整的 DQL 语句骨架，各子句**按下列顺序书写**：

```sql
SELECT   字段列表
FROM     表名列表
WHERE    条件列表
GROUP BY 分组字段列表
HAVING   分组后条件列表
ORDER BY 排序字段列表
LIMIT    分页参数
```

常见的三个层次：**基本查询**、**条件查询**、**分组查询**

> 具体子句的语法与示例随用随查

### 2.1 聚合函数

**聚合函数**：以**多行**为输入、算出**一个值**输出的函数，是分组查询的核心工具。

| | 普通函数（`UPPER`、`ABS`、`IFNULL`） | 聚合函数（`COUNT`、`SUM`） |
| --- | --- | --- |
| 输入 | 一行的某个值 | **一组行** |
| 输出 | 一个值，**行数不变** | 一个值，**多行被压成一行** |
| 例子 | `UPPER('abc')` → `'ABC'` | `SUM(salary)` → `10000` |

常用的 5 个：

| 函数        | 作用   |
| --------- | ---- |
| `COUNT()` | 计数   |
| `SUM()`   | 求和   |
| `AVG()`   | 求平均值 |
| `MAX()`   | 最大值  |
| `MIN()`   | 最小值  |

> 除 `COUNT(*)` 外，**所有聚合函数都忽略`NULL`**——碰到 `NULL` 直接跳过，不参与计算。`COUNT(字段)` 与 `COUNT(*)` 结果不一致；`AVG`的分母不是总行数，根源都在这里。

**与 `GROUP BY` 的关系**：

- **不写 `GROUP BY`** → 整张表当成一组，聚合结果固定只有 **1 行**（表里 0 行时 `COUNT(*)` 返回 `0`，仍是 1 行）。
- **写了 `GROUP BY dept_id`** → 每个部门返回 1 行，聚合在**每个组内**独立进行。

## 3. 书写顺序 ≠ 执行顺序

SQL 是**声明式**语言：人按 `SELECT ... FROM ... WHERE ...` 的顺序写，但数据库**不是按这个顺序执行的**。逻辑执行顺序如下：

```mermaid
flowchart LR
    A["FROM / JOIN"] --> B[WHERE] --> C[GROUP BY] --> D[HAVING] --> E[SELECT] --> F[DISTINCT] --> G[ORDER BY] --> H[LIMIT]
```

| 步骤  | 子句              | 干了什么                               |
| --- | --------------- | ---------------------------------- |
| 1   | `FROM` / `JOIN` | 确定数据源，把多表连接成一张宽表（连接条件 `ON` 在这一步生效） |
| 2   | `WHERE`         | 对**原始行**逐行过滤                       |
| 3   | `GROUP BY`      | 把剩下的行分组                            |
| 4   | `HAVING`        | 对**分组后的结果**过滤                      |
| 5   | `SELECT`        | 选出输出列、计算表达式与聚合函数——**别名在这里才诞生**     |
| 6   | `DISTINCT`      | 去重                                 |
| 7   | `ORDER BY`      | 排序                                 |
| 8   | `LIMIT`         | 截断分页                               |


### 3.1 `WHERE` 不能用聚合函数

`WHERE`在第 2 步执行，那时还没分组，`COUNT(*)`无从算起。

> `SELECT`、`HAVING`、`ORDER BY` 都能用聚合函数，唯独`WHERE`不能——判断依据还是**执行位置在不在分组之后。**

```sql
-- 错：WHERE 早于 GROUP BY，聚合值还不存在
SELECT dept_id, COUNT(*) FROM tb_emp WHERE COUNT(*) > 3 GROUP BY dept_id;
-- 报错：Invalid use of group function

-- 对：HAVING 在分组之后执行
SELECT dept_id, COUNT(*) FROM tb_emp GROUP BY dept_id HAVING COUNT(*) > 3;
```

### 3.2 `WHERE`不能用`SELECT`的别名，`ORDER BY`可以

别名在第 5 步才产生，`WHERE`比它早，`ORDER BY`比它晚。

```sql
-- 错：WHERE 执行时 p2 还不存在
SELECT price * 2 AS p2 FROM tb_book WHERE p2 > 100;
-- 对：ORDER BY 晚于 SELECT
SELECT price * 2 AS p2 FROM tb_book ORDER BY p2 DESC;
```

> `ORDER BY` 是唯一能引用 `SELECT` 别名的地方，原因就是它几乎排在整个流程的最后。

### 3.3 能写 `WHERE` 的条件不要写进 `HAVING`

这不是玄学，是执行位置决定的：`WHERE` 先砍掉大量原始行，再拿去分组，分组的数据量就小了；`HAVING` 要等分组**全部算完**才过滤。



