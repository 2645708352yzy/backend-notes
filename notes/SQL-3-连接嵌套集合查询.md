[[SQL]]

# 多表查询：连接 · 嵌套 · 集合 · 派生表

> ✅ 本节聚焦 **跨表数据检索**，涵盖四大核心查询方式：**连接查询、嵌套查询、集合查询、派生表查询**。

## 一、连接查询（JOIN）

> 💡 **连接是关系型数据库的核心能力**，用于从多个相关表中提取关联数据。

### 1.1 连接类型总览

```
┌───────────────────────────────────────────────┐
│              连接查询分类体系                    │
├───────────────────────────────────────────────┤
│ 🔹 **内连接（INNER JOIN）**                    │
│    ├─ 等值连接（=）                             │
│    ├─ 自然连接（自动去重列）                     │
│    └─ 自连接（表与自身连接）                     │
│                                              │
│ 🔹 **外连接（OUTER JOIN）**                   │
│    ├─ 左外连接（LEFT JOIN） → 保留左表全部行     │
│    ├─ 右外连接（RIGHT JOIN）→ 保留右表全部行     │
│    └─ 全外连接（FULL JOIN） → 保留双方全部行     │
│                                              │
│ 🔹 **交叉连接（CROSS JOIN）**                 │
│    └─ 笛卡尔积（无条件组合）                    │
└──────────────────────────────────────────────┘
```

------

### 1.2 等值连接 vs 非等值连接

#### ✅ 等值连接（最常用）

```sql
-- 传统写法（隐式连接）
SELECT Student.*, SC.*
FROM Student, SC
WHERE Student.Sno = SC.Sno;

-- 显式 JOIN 写法（推荐）
SELECT Student.*, SC.*
FROM Student
INNER JOIN SC ON Student.Sno = SC.Sno;
```

#### ⚠️ 非等值连接（较少用）

```sql
-- 查询成绩在 80~90 分的学生
SELECT Sname, Grade
FROM Student, SC
WHERE Student.Sno = SC.Sno 
  AND Grade BETWEEN 80 AND 90;
```

------

### 1.3 自连接（Self-Join）

> 🔄 **同一张表以不同角色参与连接**，需使用**别名**区分。

```sql
-- 查询每门课程的“间接先修课”（先修课的先修课）
SELECT 
    FIRST.Cno AS 课程号,
    FIRST.Cname AS 课程名,
    SECOND.Cpno AS 间接先修课
FROM Course FIRST
JOIN Course SECOND ON FIRST.Cpno = SECOND.Cno;
```

------

### 1.4 外连接（Outer Join）

| 类型         | 行为                                  | 适用场景                              |
| ------------ | ------------------------------------- | ------------------------------------- |
| `LEFT JOIN`  | 保留左表所有行，右表无匹配则补 `NULL` | 查“所有学生 + 选课情况（含未选课者）” |
| `RIGHT JOIN` | 保留右表所有行，左表无匹配则补 `NULL` | 查“所有课程 + 被选情况（含未被选者）” |
| `FULL JOIN`  | 保留两表所有行                        | 全面比对（如审计、对账）              |

#### 示例

```sql
-- 所有学生（含未选课者）
SELECT Student.Sno, Sname, Cno, Grade
FROM Student
LEFT JOIN SC ON Student.Sno = SC.Sno;

-- 所有课程（含未被选者）
SELECT Course.Cno, Cname, Sno, Grade
FROM SC
RIGHT JOIN Course ON SC.Cno = Course.Cno;
```

> ⚠️ **注意**：MySQL 不支持 `FULL OUTER JOIN`，需用 `UNION` 模拟。

------

### 1.5 多表连接

```sql
-- 学生姓名 + 课程名 + 成绩
SELECT 
    Student.Sno, Sname, Cname, Grade
FROM Student
JOIN SC      ON Student.Sno = SC.Sno
JOIN Course  ON SC.Cno = Course.Cno;
```

> ✅ **建议使用显式 `JOIN ... ON` 语法**，避免隐式逗号连接导致的歧义或笛卡尔积。

------

## 二、嵌套查询（子查询）

> 🧩 **将一个查询嵌入另一个查询中**，常用于复杂条件判断。

### 2.1 子查询分类

| 分类维度     | 类型         | 返回形式     | 示例场景                     |
| ------------ | ------------ | ------------ | ---------------------------- |
| **结果形状** | 标量子查询   | 单值（1×1）  | `SELECT MAX(Grade)`          |
|              | 行子查询     | 一行多列     | `(Sno, Cno)`                 |
|              | 列子查询     | 一列多行     | `SELECT Sno FROM SC`         |
|              | 表子查询     | 多行多列     | `SELECT * FROM SC WHERE ...` |
| **执行依赖** | 非相关子查询 | 独立执行     | 外层不引用内层               |
|              | 相关子查询   | 依赖外层变量 | 内层引用外层表字段           |

------

### 2.2 常见子查询模式

#### 🔹 `IN / NOT IN` —— 集合成员判断

```sql
-- 与“刘晨”同系的学生
SELECT Sno, Sname, Sdept
FROM Student
WHERE Sdept IN (
    SELECT Sdept FROM Student WHERE Sname = '刘晨'
);

-- 选修了“信息系统”课程的学生
SELECT Sno, Sname
FROM Student
WHERE Sno IN (
    SELECT Sno FROM SC WHERE Cno IN (
        SELECT Cno FROM Course WHERE Cname = '信息系统'
    )
);
```

> ⚠️ 若子查询结果含 `NULL`，`NOT IN` 可能返回空结果（因 `x <> NULL` 为 UNKNOWN）！

------

#### 🔹 比较运算符 + 标量子查询

```sql
-- 每个学生中，成绩高于自己平均分的课程
SELECT Sno, Cno
FROM SC x
WHERE Grade >= (
    SELECT AVG(Grade)
    FROM SC y
    WHERE y.Sno = x.Sno  -- 相关子查询
);
```

------

#### 🔹 `ANY` / `ALL` —— 与集合比较

| 表达式           | 含义                     |
| ---------------- | ------------------------ |
| `> ANY (子查询)` | 大于子查询中的**某个值** |
| `> ALL (子查询)` | 大于子查询中的**所有值** |
| `= ANY (...)`    | 等价于 `IN`              |
| `<> ALL (...)`   | 等价于 `NOT IN`          |

```sql
-- 非CS系中，年龄小于CS系**任意一人**的学生
SELECT Sname, Sage
FROM Student
WHERE Sage < ANY (SELECT Sage FROM Student WHERE Sdept = 'CS')
  AND Sdept <> 'CS';

-- 非CS系中，年龄小于CS系**所有人**的学生
SELECT Sname, Sage
FROM Student
WHERE Sage < ALL (SELECT Sage FROM Student WHERE Sdept = 'CS')
  AND Sdept <> 'CS';
```

------

#### 🔹 `EXISTS / NOT EXISTS` —— 存在性测试

> ✅ **高效处理“存在/不存在”类问题**，尤其适合相关子查询。

```sql
-- 选修了1号课程的学生
SELECT Sname
FROM Student
WHERE EXISTS (
    SELECT * FROM SC 
    WHERE SC.Sno = Student.Sno AND Cno = '1'
);

-- 未选修1号课程的学生
SELECT Sname
FROM Student
WHERE NOT EXISTS (
    SELECT * FROM SC 
    WHERE SC.Sno = Student.Sno AND Cno = '1'
);
```

#### 💡 经典难题：**选修了全部课程的学生**

```sql
-- 双重否定：不存在“某门课他没选”
SELECT Sname
FROM Student
WHERE NOT EXISTS (
    SELECT * FROM Course
    WHERE NOT EXISTS (
        SELECT * FROM SC
        WHERE SC.Sno = Student.Sno 
          AND SC.Cno = Course.Cno
    )
);
```

------

## 三、集合查询（Set Operations）

> 🧮 对多个 `SELECT` 结果进行**集合运算**，要求：**列数相同、对应类型兼容**。

### 3.1 集合运算符对照表

| 运算符             | 功能 | 是否去重   |
| ------------------ | ---- | ---------- |
| `UNION`            | 并集 | ✅ 去重     |
| `UNION ALL`        | 并集 | ❌ 保留重复 |
| `INTERSECT`        | 交集 | ✅ 去重     |
| `EXCEPT` / `MINUS` | 差集 | ✅ 去重     |

> ⚠️ **MySQL 不支持 `INTERSECT` 和 `EXCEPT`**，需用 `IN`/`NOT IN` 或 `EXISTS` 模拟。

------

### 3.2 示例

#### 并集（UNION）

```sql
-- CS系学生 ∪ 年龄 ≤19 的学生
SELECT * FROM Student WHERE Sdept = 'CS'
UNION
SELECT * FROM Student WHERE Sage <= 19;
```

#### 交集（INTERSECT）

```sql
-- 同时选修1号和2号课程的学生
SELECT Sno FROM SC WHERE Cno = '1'
INTERSECT
SELECT Sno FROM SC WHERE Cno = '2';
```

#### 差集（EXCEPT）

```sql
-- 选了1号但没选2号课程的学生
SELECT Sno FROM SC WHERE Cno = '1'
EXCEPT
SELECT Sno FROM SC WHERE Cno = '2';
```

------

## 四、基于派生表的查询

> 📊 **子查询出现在 `FROM` 子句中，生成临时“派生表”**。

```sql
-- 查询每个学生中，成绩高于自己平均分的记录
SELECT SC.Sno, SC.Cno
FROM SC
JOIN (
    SELECT Sno, AVG(Grade) AS Avg_g
    FROM SC
    GROUP BY Sno
) AS Avg_SC ON SC.Sno = Avg_SC.Sno
WHERE SC.Grade > Avg_SC.Avg_g;
```

### ✅ 使用要点：

- 派生表**必须起别名**（如 `AS Avg_SC`）
- 可像普通表一样参与 `JOIN`、`WHERE` 等操作
- 适用于需要**中间聚合结果**的场景



