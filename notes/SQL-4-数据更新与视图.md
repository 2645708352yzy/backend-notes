[[SQL]]

# 📝 数据更新与视图

## —— DML 操作与虚拟表机制

> ✅ 本章涵盖 **数据操纵语言（DML）** 的三大核心操作（增、删、改）以及 **视图（View）** 的定义、使用与限制。

------

## 一、DML 概述：数据操纵语言

> 💡 **DML（Data Manipulation Language）** 用于对**表中的数据**进行操作，不改变数据库结构。

### 1.1 DML 核心操作

```
┌───────────────────────────────────────────────┐
│              DML 四大操作                     │
├───────────────────────────────────────────────┤
│ ➕ INSERT  → 插入新记录（增）                  │
│ ✏️ UPDATE  → 修改现有记录（改）                │
│ ❌ DELETE  → 删除记录（删）                    │
│ 🔍 SELECT  → 查询数据（查）← 有时归为 DQL     │
└───────────────────────────────────────────────┘
```

### ⚠️ DML vs DDL 关键区别

| 类型    | 操作对象       | 示例                                       |
| ------- | -------------- | ------------------------------------------ |
| **DML** | **数据内容**   | `INSERT`, `UPDATE`, `DELETE`               |
| **DDL** | **数据库结构** | `CREATE TABLE`, `DROP VIEW`, `ALTER INDEX` |

> ✅ DML 操作通常可**事务回滚**；DDL 操作多数自动提交，不可回滚。

------

## 二、插入数据：`INSERT`

### 2.1 基本语法

```sql
-- ① 插入完整行（按表列顺序）
INSERT INTO <表名> VALUES (值1, 值2, ...);

-- ② 插入指定列（推荐！更安全）
INSERT INTO <表名> (列1, 列2, ...) 
VALUES (值1, 值2, ...);
```

### 2.2 示例

```sql
-- 完整插入
INSERT INTO Student 
VALUES ('201215128', '陈冬', '男', 18, 'IS');

-- 部分列插入（未指定列为 NULL）
INSERT INTO Student (Sno, Sname, Ssex) 
VALUES ('201215129', '张三', '男');

-- 显式插入 NULL
INSERT INTO SC (Sno, Cno, Grade) 
VALUES ('201215128', '1', NULL);
```

### 2.3 批量插入

```sql
INSERT INTO Student (Sno, Sname, Ssex, Sage, Sdept) VALUES
('201215130', '李四', '男', 20, 'CS'),
('201215131', '王五', '女', 19, 'MA'),
('201215132', '赵六', '男', 21, 'IS');
```

### 2.4 从查询结果插入（ETL 场景）

```sql
-- 将 CS 系学生复制到新表
INSERT INTO Student2 (Sno, Sname, Sdept)
SELECT Sno, Sname, Sdept 
FROM Student 
WHERE Sdept = 'CS';
```

> ✅ 此方式常用于数据迁移、备份、聚合表构建。

------

## 三、修改数据：`UPDATE`

### 3.1 基本语法

```sql
UPDATE <表名>
SET 列1 = 表达式1, 列2 = 表达式2, ...
[WHERE 条件];
```

### 3.2 示例

```sql
-- 单记录更新
UPDATE Student 
SET Sage = 22 
WHERE Sno = '201215121';

-- 多列更新
UPDATE Student 
SET Sage = 22, Sdept = 'MA' 
WHERE Sno = '201215121';

-- 全表更新（⚠️ 谨慎！）
UPDATE Student SET Sage = Sage + 1;

-- 基于子查询更新
UPDATE SC 
SET Grade = 0
WHERE Sno IN (
    SELECT Sno FROM Student WHERE Sdept = 'CS'
);
```

### ⚠️ 重要警告

> ❗ **省略 `WHERE` 子句将修改表中所有行！**
> 建议：先用 `SELECT` 验证条件，再执行 `UPDATE`。

------

## 四、删除数据：`DELETE`

### 4.1 基本语法

```sql
DELETE FROM <表名> [WHERE 条件];
```

### 4.2 示例

```sql
-- 删除单条记录
DELETE FROM Student 
WHERE Sno = '201215128';

-- 清空整表（保留结构）
DELETE FROM SC;

-- 条件删除
DELETE FROM SC
WHERE Sno IN (
    SELECT Sno FROM Student WHERE Sdept = 'CS'
);
```

### ⚠️ `DELETE` vs `TRUNCATE` vs `DROP`

| 操作         | 作用       | 是否可回滚        | 速度 | 删除结构 |
| ------------ | ---------- | ----------------- | ---- | -------- |
| `DELETE`     | 删除数据行 | ✅ 是              | 慢   | ❌ 否     |
| `TRUNCATE`   | 快速清空表 | ❌ 否（多数 DBMS） | 快   | ❌ 否     |
| `DROP TABLE` | 删除整个表 | ❌ 否              | —    | ✅ 是     |

> 💡 **`TRUNCATE` 重置自增 ID，`DELETE` 不重置**。

------

## 五、视图（VIEW）

### 5.1 什么是视图？

> 🧩 **视图是一个虚拟表**，其内容由查询定义，**不存储实际数据**。

- 数据仍保存在基本表中  
- 视图只保存 **SELECT 语句定义**  
- 查询视图时，DBMS 自动将其“展开”为对基表的查询（**视图消解**）

### 5.2 视图的核心价值

```
┌───────────────────────────────────────────────┐
│                视图的优势                      │
├───────────────────────────────────────────────┤
│ ✅ 简化复杂查询（封装 JOIN / 子查询）          │
│ ✅ 提供逻辑独立性（基表重构不影响应用）        │
│ ✅ 增强安全性（仅暴露部分列/行）               │
│ ✅ 多角度查看同一数据                          │
│ ✅ 提高查询可读性                              │
└───────────────────────────────────────────────┘
```

------

### 5.3 创建视图

```sql
CREATE VIEW <视图名> 
AS <SELECT 查询>
[WITH CHECK OPTION];
```

#### `WITH CHECK OPTION` 说明：

> 若存在此选项，则通过视图进行 `INSERT`/`UPDATE` 时，**新数据必须满足视图的 `WHERE` 条件**，否则拒绝操作。

### 5.4 创建示例

```sql
-- ① 信息系学生视图（带检查）
CREATE VIEW IS_Student AS
SELECT Sno, Sname, Sage
FROM Student
WHERE Sdept = 'IS'
WITH CHECK OPTION;

-- ② 学生平均成绩视图
CREATE VIEW Student_GPA (Sno, AvgGrade) AS
SELECT Sno, AVG(Grade)
FROM SC
GROUP BY Sno;

-- ③ 计算列视图
CREATE VIEW Student_Info AS
SELECT 
    Sno, 
    Sname, 
    Ssex, 
    2024 - Sage AS BirthYear
FROM Student;
```

------

### 5.5 删除视图

```sql
DROP VIEW <视图名> [CASCADE];
```

| 选项      | 行为                                 |
| --------- | ------------------------------------ |
| （默认）  | 仅删除该视图                         |
| `CASCADE` | 级联删除该视图及**依赖它的其他视图** |

```sql
DROP VIEW IS_Student;           -- 安全删除
DROP VIEW IS_Student CASCADE;   -- 强制级联
```

------

### 5.6 查询视图

```sql
-- 查询视图如同查询普通表
SELECT Sno, Sname 
FROM IS_Student 
WHERE Sage < 20;

SELECT Sno, AvgGrade 
FROM Student_GPA 
WHERE AvgGrade >= 90;
```

> 🔁 DBMS 自动将视图查询转换为对基表的等价查询。

------

### 5.7 更新视图（有条件支持）

```sql
-- 插入（若视图可更新）
INSERT INTO IS_Student VALUES ('201215140', '赵新', 20);

-- 更新
UPDATE IS_Student SET Sname = '刘辰' WHERE Sno = '201215122';

-- 删除
DELETE FROM IS_Student WHERE Sno = '201215140';
```

### ⚠️ **不可更新的视图类型**

以下视图**禁止** `INSERT` / `UPDATE` / `DELETE`：

| 情况                                  | 原因               |
| ------------------------------------- | ------------------ |
| 包含 `GROUP BY`                       | 结果非原始行       |
| 使用聚合函数（`AVG`, `SUM`, `COUNT`） | 非基础数据         |
| 包含 `DISTINCT`                       | 行可能被合并       |
| 基于多表连接（JOIN）                  | 无法确定更新目标表 |
| 包含计算列（如 `2024-Sage`）          | 无对应物理列       |
| 使用 `UNION` / `INTERSECT` 等集合操作 | 结构复杂           |

> ✅ **简单、单表、无聚合、无计算列的视图通常可更新**。

------

## 六、总结对比表

| 操作          | 语法                            | 作用       | 注意事项         |
| ------------- | ------------------------------- | ---------- | ---------------- |
| `INSERT`      | `INSERT INTO ... VALUES (...)`  | 添加新行   | 检查主键/约束    |
| `UPDATE`      | `UPDATE ... SET ... WHERE ...`  | 修改现有行 | **务必加 WHERE** |
| `DELETE`      | `DELETE FROM ... WHERE ...`     | 删除行     | **务必加 WHERE** |
| `CREATE VIEW` | `CREATE VIEW ... AS SELECT ...` | 创建虚拟表 | 注意可更新性     |
| `DROP VIEW`   | `DROP VIEW ... [CASCADE]`       | 删除视图   | 级联影响依赖     |