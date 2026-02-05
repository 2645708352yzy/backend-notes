[[SQL]]

# 📘 数据定义语言（DDL）

> **DDL（Data Definition Language）** 用于 **定义、修改和删除数据库对象**，如：模式（Schema）、表（Table）、视图（View）、索引（Index）等。

## 一、操作对象与 DDL 命令对照表

| 操作对象 | 创建            | 删除          | 修改          |
| -------- | --------------- | ------------- | ------------- |
| **模式** | `CREATE SCHEMA` | `DROP SCHEMA` | —             |
| **表**   | `CREATE TABLE`  | `DROP TABLE`  | `ALTER TABLE` |
| **视图** | `CREATE VIEW`   | `DROP VIEW`   | —             |
| **索引** | `CREATE INDEX`  | `DROP INDEX`  | —             |

> ⚠️ 注：多数数据库系统不支持直接“修改”视图或索引，通常需先删除再重建。

## 二、模式（Schema）的定义与删除

### 2.1 定义模式

**模式（Schema）** 是一个命名空间，用于组织数据库对象（表、视图、索引等）。

```sql
-- 基本语法
CREATE SCHEMA <模式名> AUTHORIZATION <用户名>;

-- 示例：为用户 BitHachi 创建名为 "S-T" 的模式
CREATE SCHEMA "S-T" AUTHORIZATION BitHachi;

-- 若省略模式名，则默认使用用户名作为模式名
CREATE SCHEMA AUTHORIZATION BitHachi;
```

### 2.2 在创建模式时同时创建表

```sql
CREATE SCHEMA "S-T" AUTHORIZATION BitHachi
    CREATE TABLE TAB1 (
        COL1 SMALLINT,
        COL2 INT,
        COL3 CHAR(20),
        COL4 NUMERIC(10,3),
        COL5 DECIMAL(5,2)
    );
```

> 支持在 `CREATE SCHEMA` 中嵌套多个 DDL 语句（如 `CREATE TABLE`, `CREATE VIEW` 等）。

### 2.3 删除模式

```sql
DROP SCHEMA <模式名> [CASCADE | RESTRICT];
```

| 选项       | 行为说明                                                   |
| ---------- | ---------------------------------------------------------- |
| `CASCADE`  | **级联删除**：删除模式及其包含的所有对象（表、视图等）     |
| `RESTRICT` | **限制删除**：若模式中存在任何对象，则拒绝删除（默认行为） |

```sql
-- 级联删除
DROP SCHEMA "S-T" CASCADE;

-- 安全删除（仅当为空时）
DROP SCHEMA "S-T" RESTRICT;
```



## 三、基本表的定义、修改与删除

### 3.1 定义基本表

```sql
CREATE TABLE <表名> (
    <列名> <数据类型> [列级约束],
    ...
    [表级约束]
);
```

> **重要规则**：  
>
> - 若完整性约束涉及**多个列**（如复合主键、外键引用多列），**必须定义在表级**。  
> - 单列约束可写在列级或表级。

### 3.2 常用数据类型

| 数据类型                        | 说明                         | 示例               |
| ------------------------------- | ---------------------------- | ------------------ |
| `CHAR(n)`                       | 定长字符串                   | `CHAR(10)`         |
| `VARCHAR(n)`                    | 变长字符串（最大 n 字符）    | `VARCHAR(20)`      |
| `INT`                           | 整数                         | `INT`              |
| `SMALLINT`                      | 小整数                       | `SMALLINT`         |
| `NUMERIC(p,d)` / `DECIMAL(p,d)` | 定点数（p 位总长，d 位小数） | `NUMERIC(8,2)`     |
| `REAL`                          | 单精度浮点数                 | `REAL`             |
| `DOUBLE PRECISION`              | 双精度浮点数                 | `DOUBLE PRECISION` |
| `FLOAT(n)`                      | 浮点数（至少 n 位精度）      | `FLOAT(10)`        |
| `DATE`                          | 日期（如 `2025-01-01`）      | `DATE`             |
| `TIME`                          | 时间（如 `14:30:00`）        | `TIME`             |
| `TIMESTAMP`                     | 日期 + 时间                  | `TIMESTAMP`        |

### 3.3 完整性约束类型

| 约束类型 | 关键字        | 说明                          |
| -------- | ------------- | ----------------------------- |
| 主键约束 | `PRIMARY KEY` | 唯一标识每条记录              |
| 外键约束 | `FOREIGN KEY` | 引用其他表的主键              |
| 非空约束 | `NOT NULL`    | 列值不能为空                  |
| 唯一约束 | `UNIQUE`      | 列值必须唯一（允许多个 NULL） |
| 检查约束 | `CHECK`       | 限制列值范围（如 `Sage > 0`） |

### 3.4 创建表示例

```sql
-- 学生表
CREATE TABLE Student (
    Sno   CHAR(9) PRIMARY KEY,      -- 学号（主键）
    Sname VARCHAR(20) UNIQUE,       -- 姓名（唯一）
    Ssex  CHAR(2),
    Sage  SMALLINT,
    Sdept VARCHAR(20)
);

-- 课程表（含自引用外键）
CREATE TABLE Course (
    Cno    CHAR(4) PRIMARY KEY,     -- 课程号
    Cname  VARCHAR(40) NOT NULL,    -- 课程名（非空）
    Cpno   CHAR(4),                 -- 先修课程号
    Ccredit SMALLINT,
    FOREIGN KEY (Cpno) REFERENCES Course(Cno)  -- 自引用
);
```

> **自引用外键（Self-referencing Foreign Key）**
>
> - **定义**：自引用外键是指**一个表的外键引用的是它自身的主键**。
> - **用途**：常用于表示**层次结构**或**递归关系**，例如组织架构（员工-经理）、评论的回复链、分类的父子关系等。

## 四、修改基本表（`ALTER TABLE`）

```sql
ALTER TABLE <表名>
    [ADD <列名> <数据类型> [约束]]
    [DROP COLUMN <列名>]                -- 某些 DBMS 支持
    [DROP CONSTRAINT <约束名>]
    [ALTER COLUMN <列名> <新数据类型>];
```

### 常见操作示例

```sql
-- ① 添加新列（默认值为 NULL）
ALTER TABLE Student ADD S_entrance DATE;

-- ② 修改列的数据类型
ALTER TABLE Student ALTER COLUMN Sage INT;

-- ③ 添加唯一约束（表级）
ALTER TABLE Course ADD UNIQUE (Cname);

-- ④ 删除约束（需指定约束名）
ALTER TABLE Student DROP CONSTRAINT uk_sname;
```

> ⚠️ 注意：
>
> - 新增列默认为 `NULL`，除非显式指定 `NOT NULL` 并提供默认值。
> - 不同数据库对 `ALTER COLUMN` 支持程度不同（如 MySQL 不支持直接改类型，需用 `MODIFY`）。

## 五、删除基本表

```sql
DROP TABLE <表名> [RESTRICT | CASCADE];
```

| 选项       | 行为说明                                                     |
| ---------- | ------------------------------------------------------------ |
| `RESTRICT` | 若表被视图、外键等引用，则**拒绝删除**（更安全）             |
| `CASCADE`  | **级联删除**：同时删除依赖该表的所有对象（如视图、索引、触发器等） |

```sql
DROP TABLE Student CASCADE;
```



## 六、索引的建立与删除

### 6.1 为什么需要索引？

-  **加速查询**（尤其在 `WHERE`, `JOIN`, `ORDER BY` 中）
- 由 **DBMS 自动维护**
- 查询优化器自动决定是否使用索引

> 通常由 **DBA 或表属主** 创建。



### 6.2 创建索引

```sql
CREATE [UNIQUE] [CLUSTER] INDEX <索引名>
ON <表名> (<列名> [ASC|DESC] [, ...]);
```

| 关键字    | 说明                                                       |
| --------- | ---------------------------------------------------------- |
| `UNIQUE`  | 索引值必须唯一（常用于主键/唯一约束列）                    |
| `CLUSTER` | **聚簇索引**：物理上按索引顺序存储数据（**每表最多一个**） |

#### 示例

```sql
-- 在 Student 表的 Sname 上建聚簇索引
CREATE CLUSTER INDEX Stusname ON Student(Sname);

-- 在 SC 表上建复合唯一索引（Sno 升序，Cno 降序）
CREATE UNIQUE INDEX SCno ON SC(Sno ASC, Cno DESC);
```

> 聚簇索引适合频繁范围查询的列；唯一索引防止重复值。



### 6.3 删除索引

```sql
DROP INDEX <索引名>;
```

>  索引删除不影响表数据，仅影响查询性能。



## 总结要点

| 操作     | 推荐实践                                             |
| -------- | ---------------------------------------------------- |
| **建表** | 明确主键、外键、非空、唯一等约束                     |
| **改表** | 谨慎修改列类型；新增列注意默认值                     |
| **删表** | 优先用 `RESTRICT` 避免误删；确认无依赖后再 `CASCADE` |
| **索引** | 在高频查询列上建索引；避免过度索引（影响写性能）     |

