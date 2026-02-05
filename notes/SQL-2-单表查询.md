[[SQL]]

# 🔍 单表查询

> 聚焦 **单张表内的数据检索**，涵盖 `SELECT` 基础语法、条件过滤、排序、聚合与分组。

## 一、`SELECT` 语句核心结构

### 1.1 基本语法模板

```sql
SELECT   <目标列表达式>        -- ⑤ 最终输出列（可含计算、别名）
FROM     <表名>                -- ① 数据来源
[WHERE   <行过滤条件>]         -- ② 筛选原始记录
[GROUP BY <分组列>]            -- ③ 按列分组
[HAVING  <组过滤条件>]         -- ④ 过滤分组结果
[ORDER BY <排序列> [ASC|DESC]];-- ⑥ 结果排序
```

### 1.2 执行顺序

> **SQL 并非按书写顺序执行！理解执行流是写对复杂查询的关键。**

```
1️⃣ FROM      → 确定数据源表
2️⃣ WHERE     → 过滤行（不能用聚合函数）
3️⃣ GROUP BY  → 将结果划分为若干组
4️⃣ HAVING    → 过滤组（可用聚合函数）
5️⃣ SELECT    → 计算并选择输出列
6️⃣ ORDER BY  → 对最终结果排序
```

------

## 二、基础查询操作

### 2.1 查询列的选择

```sql
-- 查询所有列
SELECT * FROM Student;

-- 查询指定列
SELECT Sno, Sname FROM Student;

-- 查询计算列（带别名）
SELECT Sname, 2024 - Sage AS 出生年份 FROM Student;
```

### 2.2 去重：`DISTINCT`

```sql
-- 获取所有不重复的系别
SELECT DISTINCT Sdept FROM Student;
```

>  适用于去除结果集中完全相同的行。

### 2.3 列别名：`AS`（可省略）

```sql
SELECT Sname AS 姓名, Sage AS 年龄 FROM Student;
-- 或
SELECT Sname 姓名, Sage 年龄 FROM Student;
```

>  别名在 `ORDER BY` 中可直接使用，但在 `WHERE`/`GROUP BY` 中**通常不可用**（因执行顺序靠后）。



## 三、条件过滤：`WHERE` 子句

> ⚠️ **作用于分组前的每一行**，**不能使用聚合函数**。

### 3.1 比较运算符

| 运算符               | 含义     |
| -------------------- | -------- |
| `=`                  | 等于     |
| `<>` 或 `!=`         | 不等于   |
| `>`, `<`, `>=`, `<=` | 大小比较 |

```sql
SELECT * FROM Student WHERE Sdept = 'CS';
SELECT * FROM Student WHERE Sage > 20;
```

### 3.2 逻辑运算符

| 运算符 | 含义           |
| ------ | -------------- |
| `AND`  | 且（同时满足） |
| `OR`   | 或（满足其一） |
| `NOT`  | 非（取反）     |

```sql
SELECT * FROM Student 
WHERE Sdept = 'CS' AND Sage > 20;

SELECT * FROM Student 
WHERE Sdept IN ('CS', 'IS');
```

### 3.3 范围查询：`BETWEEN ... AND`

```sql
-- 包含边界值（闭区间）
SELECT * FROM Student WHERE Sage BETWEEN 20 AND 23;
-- 等价于：Sage >= 20 AND Sage <= 23
```

### 3.4 集合成员判断：`IN / NOT IN`

```sql
SELECT * FROM Student 
WHERE Sdept IN ('CS', 'MA', 'IS');
```

### 3.5 空值判断：`IS NULL / IS NOT NULL`

```sql
-- ✅ 正确写法
SELECT * FROM SC WHERE Grade IS NULL;

-- ❌ 错误！NULL 不能用 = 判断
-- SELECT * FROM SC WHERE Grade = NULL;
```

> **空值（NULL）表示“未知”或“不存在”，任何与 NULL 的比较结果都是 UNKNOWN（非 TRUE/FALSE）**。

### 3.6 模糊匹配：`LIKE`

| 通配符 | 含义                         |
| ------ | ---------------------------- |
| `%`    | 匹配任意长度字符串（包括空） |
| `_`    | 匹配单个字符                 |

```sql
SELECT * FROM Student WHERE Sname LIKE '刘%';      -- 姓刘
SELECT * FROM Student WHERE Sname LIKE '刘_';      -- 刘+一个字
SELECT * FROM Student WHERE Sname LIKE '%明%';     -- 名含“明”
SELECT * FROM Student WHERE Sname LIKE '_伟%';     -- 第二字是“伟”
```

#### 转义特殊字符

```sql
-- 查询课程名含下划线 "_" 的记录
SELECT * FROM Course 
WHERE Cname LIKE '%\_%' ESCAPE '\';
```

> `ESCAPE` 定义转义字符（此处为 `\`），使 `_` 被视为普通字符。



## 四、结果排序：`ORDER BY`

```sql
SELECT * FROM Student
ORDER BY Sage DESC, Sdept ASC;
```

| 关键字 | 含义         |
| ------ | ------------ |
| `ASC`  | 升序（默认） |
| `DESC` | 降序         |

> 可按多列排序；**排序列可以是 SELECT 中未出现的列**（但某些 DBMS 限制）。

## 五、聚合函数（Aggregate Functions）

> 📊 对一组值进行计算，返回单个汇总结果。

| 函数        | 功能说明               |
| ----------- | ---------------------- |
| `COUNT(*)`  | 统计总行数（含 NULL）  |
| `COUNT(列)` | 统计该列**非空值**个数 |
| `SUM(列)`   | 求和                   |
| `AVG(列)`   | 平均值                 |
| `MAX(列)`   | 最大值                 |
| `MIN(列)`   | 最小值                 |

```sql
SELECT COUNT(*) FROM Student;                          -- 总人数
SELECT COUNT(DISTINCT Sno) FROM SC;                    -- 选课学生数（去重）
SELECT AVG(Grade), MAX(Grade) FROM SC WHERE Cno = '1'; -- 课程1的成绩统计
```

> ⚠️ **聚合函数自动忽略 NULL 值！**

------

## 六、分组查询：`GROUP BY` + `HAVING`

### 6.1 基本分组

```sql
-- 每个系的学生人数
SELECT Sdept, COUNT(*) AS 人数
FROM Student
GROUP BY Sdept;

-- 每门课的平均成绩
SELECT Cno, AVG(Grade) AS 平均分
FROM SC
GROUP BY Cno;
```

> `SELECT` 中的非聚合列**必须出现在 `GROUP BY` 中**（SQL 标准要求）。

### 6.2 分组后过滤：`HAVING`

```sql
-- 选修超过 2 门课的学生
SELECT Sno, COUNT(*) AS 选课数
FROM SC
GROUP BY Sno
HAVING COUNT(*) > 2;

-- 平均分 > 85 的学生
SELECT Sno, AVG(Grade) AS 平均分
FROM SC
GROUP BY Sno
HAVING AVG(Grade) > 85;
```

### 6.3 `WHERE` vs `HAVING` 对比

| 特性               | `WHERE`          | `HAVING`         |
| ------------------ | ---------------- | ---------------- |
| **作用时机**       | 分组前           | 分组后           |
| **作用对象**       | 行（Row）        | 组（Group）      |
| **能否用聚合函数** | ❌ 否             | ✅ 是             |
| **性能**           | 更高效（早过滤） | 较低（需先分组） |

> **最佳实践**：能用 `WHERE` 过滤的，不要放到 `HAVING`！



## 七、综合示例

### 示例 1：多条件分组筛选

```sql
-- 查询选修 >2 门课 且 平均分 >85 的学生
SELECT 
    Sno,
    AVG(Grade) AS 平均成绩,
    COUNT(*)   AS 选课门数
FROM SC
GROUP BY Sno
HAVING COUNT(*) > 2 AND AVG(Grade) > 85;
```

### 示例 2：条件 + 排序 + 限制（如 MySQL/PostgreSQL）

```sql
-- 计算机系年龄最大的前 3 名学生
SELECT Sname, Sage
FROM Student
WHERE Sdept = 'CS'
ORDER BY Sage DESC
LIMIT 3;  -- 注意：SQL Server 用 TOP，Oracle 用 ROWNUM
```

> ⚠️ `LIMIT` 不是 SQL 标准，不同数据库语法不同：
>
> - MySQL / PostgreSQL：`LIMIT n`
> - SQL Server：`SELECT TOP n ...`
> - Oracle：`ROWNUM <= n`

------

##  关键总结

| 概念                | 要点                                               |
| ------------------- | -------------------------------------------------- |
| **执行顺序**        | F → W → G → H → S → O                              |
| **NULL 处理**       | 用 `IS NULL`，不用 `= NULL`                        |
| **聚合函数**        | 忽略 NULL，不能用于 `WHERE`                        |
| **GROUP BY**        | SELECT 中非聚合列必须在 GROUP BY 中                |
| **WHERE vs HAVING** | 行过滤 vs 组过滤，优先用 WHERE                     |
| **别名使用**        | 可用于 `ORDER BY`，一般不可用于 `WHERE`/`GROUP BY` |

## 附录：表结构

### **表 1：**`Student`**（学生表）**

| 列名    | 类型        | 含义   | 约束说明            |
| :------ | :---------- | :----- | :------------------ |
| `Sno`   | CHAR(10)    | 学号   | 主键                |
| `Sname` | VARCHAR(20) | 姓名   | 非空                |
| `Ssex`  | CHAR(2)     | 性别   | '男' 或 '女'        |
| `Sage`  | INT         | 年龄   | 通常 16～25         |
| `Sdept` | VARCHAR(20) | 所在系 | 如 'CS'（计算机）等 |



#### **示例数据（**`Student` **表）：**

| Sno     | Sname | Ssex | Sage | Sdept |
| :------ | :---- | :--- | :--- | :---- |
| 2023001 | 张伟  | 男   | 20   | CS    |
| 2023002 | 李娜  | 女   | 19   | MA    |
| 2023003 | 王强  | 男   | 21   | CS    |
| 2023004 | 刘芳  | 女   | 20   | IS    |
| 2023005 | 赵明  | 男   | 22   | CS    |
| 2023006 | 陈静  | 女   | 18   | MA    |
| 2023007 | 黄磊  | 男   | 20   | NULL  |

> 注：最后一行 `Sdept` 为 `NULL`，用于演示空值处理。



### **表 2：**`SC`**（选课表，Student-Course）**

| 列名    | 类型         | 含义           |
| :------ | :----------- | :------------- |
| `Sno`   | CHAR(10)     | 学号           |
| `Cno`   | CHAR(5)      | 课程号         |
| `Grade` | DECIMAL(5,2) | 成绩（可为空） |

#### **示例数据（**`SC` **表）：**

| Sno     | Cno  | Grade |
| :------ | :--- | :---- |
| 2023001 | C1   | 88.00 |
| 2023001 | C2   | 92.00 |
| 2023002 | C1   | 78.00 |
| 2023002 | C2   | NULL  |
| 2023003 | C1   | 85.00 |
| 2023003 | C2   | 87.00 |
| 2023003 | C3   | 90.00 |
| 2023004 | C1   | 76.00 |
| 2023005 | C2   | NULL  |

> 注意：`Grade` 列包含 `NULL`，用于演示 `COUNT(Grade)` 与 `COUNT(*)` 的区别。
