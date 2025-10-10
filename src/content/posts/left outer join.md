---
title: left outer join
published: 2025-10-10
tags: [mysql]
image: "https://image.521500.xyz/file/join.png"
description: 左连接例题
category: mysql
draft: false
---

### 题目

![image.png](https://image.521500.xyz/file/image(2).png)


#  标准解法

```sql
SELECT
  p.FirstName,
  p.LastName,
  a.City,
  a.State
FROM Person AS p
LEFT JOIN Address AS a
  ON p.PersonId = a.PersonId;
```

**要点：**

- 用 `LEFT JOIN`（左连接）从 `Person` 出发，把 `Address` 里能匹配到的人拼上地址信息；
  
- 没有匹配到的行，`City` 和 `State` 会得到 `NULL`（题目要求如此）；
  
- 题目允许任意顺序返回，所以不需要 `ORDER BY`。
  

---

# 为什么是 LEFT JOIN？

- **INNER JOIN**：只返回两表都存在匹配的行。这样没有地址的人会被丢掉，**不符合题意**。
  
- **LEFT JOIN**：保留左表（Person）全部行；右表（Address）没有匹配时用 `NULL` 补上；**正合题意**。
  
- **RIGHT JOIN**：要达到同样效果，需要把表位次调换、思维也更绕，不如 `LEFT JOIN` 直观。
  
- **FULL OUTER JOIN**：两边都保留，超出题目需求。
  

---

#  等价写法与小技巧

### 1）USING 语法（列名相同更简洁）

（MySQL / PostgreSQL 支持）

```sql
SELECT
  p.FirstName,
  p.LastName,
  a.City,
  a.State
FROM Person AS p
LEFT JOIN Address AS a
USING (PersonId);
```

### 2）显示“未知”而不是 NULL（不是本题要求，仅扩展）

```sql
SELECT
  p.FirstName,
  p.LastName,
  COALESCE(a.City,  'Unknown') AS City,
  COALESCE(a.State, 'Unknown') AS State
FROM Person AS p
LEFT JOIN Address AS a
  ON p.PersonId = a.PersonId;
```

---

# 可能的坑与拓展思考

1. **一人多地址怎么办？**  
    题目通常默认 0 或 1 条地址；若存在多条，你会得到**多行**同一个人的输出。  
    想固定只保留一条（比如 `addressId` 最小的）：
    
    - **MySQL 8+/PostgreSQL**（窗口函数）：
      
        ```sql
        WITH one AS (
          SELECT a.*,
                 ROW_NUMBER() OVER (PARTITION BY a.PersonId ORDER BY a.AddressId) AS rn
          FROM Address a
        )
        SELECT p.FirstName, p.LastName, o.City, o.State
        FROM Person p
        LEFT JOIN one o
          ON p.PersonId = o.PersonId AND o.rn = 1;
        ```
        
    - **聚合 + 子查询**（通用思路）：
      
        ```sql
        SELECT p.FirstName, p.LastName, a.City, a.State
        FROM Person p
        LEFT JOIN (
          SELECT a1.*
          FROM Address a1
          JOIN (
            SELECT PersonId, MIN(AddressId) AS min_id
            FROM Address
            GROUP BY PersonId
          ) pick ON a1.PersonId = pick.PersonId AND a1.AddressId = pick.min_id
        ) a ON p.PersonId = a.PersonId;
        ```
    
2. **NULL 的理解**  
    `NULL` 表示“未知/不存在”，不是空字符串 `''`，也不是文字 `"Null"`。比较时需要用 `IS NULL`、`IS NOT NULL`，不能用 `=`。
    
3. **性能与约束**
   
    - 给 `Address(PersonId)` 建**索引**或**外键**能加速连接并保证数据一致性：
      
        ```sql
        -- 仅索引
        CREATE INDEX idx_address_person_id ON Address(PersonId);
        
        -- 外键（如果你的库/表引擎支持）
        ALTER TABLE Address
        ADD CONSTRAINT fk_address_person
        FOREIGN KEY (PersonId) REFERENCES Person(PersonId);
        ```
        
    - `Person(PersonId)` 作为主键天然有索引。
    
4. **命名大小写**  
    SQL 关键字大小写不敏感；列名是否大小写敏感取决于数据库及配置。跨库写法建议与建表时一致。
    

---

# 对照题例验证

输入：

```
Person
personId | lastName | firstName
1        | Wang     | Allen
2        | Alice    | Bob

Address
addressId | personId | city          | state
1         | 2        | New York City | New York
2         | 3        | Leetcode      | California
```

连接后：

- `personId=1` 在 Address 中**无匹配** → `City=NULL, State=NULL`
  
- `personId=2` 匹配到 NYC → 正常填充
  

输出与题目示例一致。

---



下面先写一份“人”和“地址”的测试数据，再分别看 **INNER / LEFT / RIGHT / FULL OUTER JOIN** 的结果与常见坑。

# 测试数据

**Person**

```
+----------+----------+-----------+
| PersonId | LastName | FirstName |
+----------+----------+-----------+
| 1        | Wang     | Allen     |   -- 无地址
| 2        | Alice    | Bob       |   -- 有1个地址
| 4        | King     | Diana     |   -- 有2个地址
| 5        | Zhou     | Eva       |   -- 无地址
+----------+----------+-----------+
```

**Address**

```
+-----------+----------+---------------+-----------+
| AddressId | PersonId | City          | State     |
+-----------+----------+---------------+-----------+
| 1         | 2        | New York City | New York  |  -- 匹配 PersonId=2
| 2         | 3        | Leetcode      | California|  -- “孤儿”地址：无对应Person
| 3         | 4        | Seattle       | WA        |  -- 匹配 PersonId=4
| 4         | 4        | Bellevue      | WA        |  -- 仍匹配 PersonId=4（多地址）
| 5         | NULL     | Los Angeles   | CA        |  -- PersonId NULL（无法匹配）
+-----------+----------+---------------+-----------+
```

------

# 1) INNER JOIN（只要双方都匹配）

```sql
SELECT p.FirstName, p.LastName, a.City, a.State
FROM Person p
JOIN Address a
  ON p.PersonId = a.PersonId;
```

**输出（说明）：**

```
+-----------+----------+---------------+-----------+
| FirstName | LastName | City          | State     |
+-----------+----------+---------------+-----------+
| Bob       | Alice    | New York City | New York  |  -- PersonId=2 匹配
| Diana     | King     | Seattle       | WA        |  -- PersonId=4 的第1条
| Diana     | King     | Bellevue      | WA        |  -- PersonId=4 的第2条（行放大）
+-----------+----------+---------------+-----------+
```

- **丢掉**：Allen(1)、Eva(5)（因为无地址）；Address(2) 的“孤儿”行（PersonId=3）；Address(5) 的 NULL PersonId。
- **多对一放大**：Diana(4) 有两条地址 → 两行。

> 用途：当且仅当你**必须保证**两边都存在时。

------

# 2) LEFT JOIN（左表全保留，右表匹配不到填 NULL）

```sql
SELECT p.FirstName, p.LastName, a.City, a.State
FROM Person p
LEFT JOIN Address a
  ON p.PersonId = a.PersonId;
```

**输出（说明）：**

```
+-----------+----------+---------------+-----------+
| FirstName | LastName | City          | State     |
+-----------+----------+---------------+-----------+
| Allen     | Wang     | NULL          | NULL      |  -- PersonId=1 无匹配 → NULL
| Bob       | Alice    | New York City | New York  |  -- PersonId=2 匹配
| Diana     | King     | Seattle       | WA        |  -- PersonId=4 的第1条
| Diana     | King     | Bellevue      | WA        |  -- PersonId=4 的第2条
| Eva       | Zhou     | NULL          | NULL      |  -- PersonId=5 无匹配 → NULL
+-----------+----------+---------------+-----------+
```

- **保留所有人**：1、2、4、5 都在；
- **无地址** → `City/State` 为 `NULL`；
- **多地址**仍然“行放大”。

> 用途：当你要“以人（左表）为准”，即**结果必须包含所有人**时（正是本题）。

------

# 3) RIGHT JOIN（右表全保留，对称于 LEFT）

```sql
SELECT p.FirstName, p.LastName, a.City, a.State
FROM Person p
RIGHT JOIN Address a
  ON p.PersonId = a.PersonId;
```

**输出（说明）：**

```
+-----------+----------+---------------+-----------+
| FirstName | LastName | City          | State     |
+-----------+----------+---------------+-----------+
| Bob       | Alice    | New York City | New York  |  -- 匹配到 PersonId=2
| Diana     | King     | Seattle       | WA        |  -- 匹配到 PersonId=4
| Diana     | King     | Bellevue      | WA        |  -- 仍然行放大
| NULL      | NULL     | Leetcode      | California|  -- Address(PersonId=3) 孤儿行
| NULL      | NULL     | Los Angeles   | CA        |  -- Address(PersonId=NULL) 仍保留
+-----------+----------+---------------+-----------+
```

- 跟 LEFT 的思路**相反**：以地址为准，保留所有地址；
- **孤儿地址**（找不到人）也会出现，人的列为 `NULL`。

> 实战里更常用 **LEFT JOIN**；RIGHT 常可通过调换表位次改写成 LEFT，更直观：
>
> ```sql
> -- 等价改写
> SELECT ...
> FROM Address a
> LEFT JOIN Person p ON a.PersonId = p.PersonId;
> ```

------

# 4) FULL OUTER JOIN（两边都保留）

> 注：PostgreSQL / SQL Server / Oracle 支持；MySQL 原生**不支持**，需用 `LEFT JOIN UNION RIGHT JOIN` 模拟（见下）。

**标准写法：**

```sql
SELECT p.FirstName, p.LastName, a.City, a.State
FROM Person p
FULL OUTER JOIN Address a
  ON p.PersonId = a.PersonId;
```

**输出（说明）：**

```
+-----------+----------+---------------+-----------+
| FirstName | LastName | City          | State     |
+-----------+----------+---------------+-----------+
| Allen     | Wang     | NULL          | NULL      |  -- 左独有：PersonId=1
| Bob       | Alice    | New York City | New York  |  -- 匹配：PersonId=2
| Diana     | King     | Seattle       | WA        |  -- 匹配：PersonId=4 (1)
| Diana     | King     | Bellevue      | WA        |  -- 匹配：PersonId=4 (2)
| Eva       | Zhou     | NULL          | NULL      |  -- 左独有：PersonId=5
| NULL      | NULL     | Leetcode      | California|  -- 右独有：Address.PersonId=3
| NULL      | NULL     | Los Angeles   | CA        |  -- 右独有：Address.PersonId=NULL
+-----------+----------+---------------+-----------+
```

- **两边独有的记录也都出现**；

- **MySQL 模拟**（避免“双重计数”，最好 `WHERE a.PersonId IS NULL`/`p.PersonId IS NULL` 过滤）：

  ```sql
  SELECT p.FirstName, p.LastName, a.City, a.State
  FROM Person p
  LEFT JOIN Address a ON p.PersonId = a.PersonId
  UNION
  SELECT p.FirstName, p.LastName, a.City, a.State
  FROM Person p
  RIGHT JOIN Address a ON p.PersonId = a.PersonId
  WHERE p.PersonId IS NULL;   -- 这句避免把匹配到的行重复
  ```

------

## 高频“坑点”与最佳实践

### A. **WHERE 条件位置会“意外把 LEFT 变 INNER”**

> 想“保留所有人，即便没地址”，但只想要 WA 州的地址字段。

**错误写法（把条件放在 WHERE）：**

```sql
SELECT p.FirstName, p.LastName, a.City, a.State
FROM Person p
LEFT JOIN Address a ON p.PersonId = a.PersonId
WHERE a.State = 'WA';   -- ❌ 这会过滤掉 a.State 为 NULL 的人，等同 INNER
```

**正确写法 1（把条件放到 ON）：**

```sql
SELECT p.FirstName, p.LastName, a.City, a.State
FROM Person p
LEFT JOIN Address a
  ON p.PersonId = a.PersonId
 AND a.State = 'WA';     -- ✅ 保留所有人；非 WA 或无地址 → City/State 为 NULL
```

**正确写法 2（WHERE 保留空值）：**

```sql
... 
WHERE a.State = 'WA' OR a.State IS NULL;  -- ✅ 显式保留无地址的人
```

### B. **多对一导致“行放大”**

- 一个人多条地址 → 结果多行。若你想“只留一条”（比如最新/最小 `AddressId`），用窗口函数：

  ```sql
  WITH one AS (
    SELECT a.*,
           ROW_NUMBER() OVER (PARTITION BY a.PersonId ORDER BY a.AddressId) AS rn
    FROM Address a
  )
  SELECT p.FirstName, p.LastName, o.City, o.State
  FROM Person p
  LEFT JOIN one o
    ON p.PersonId = o.PersonId AND o.rn = 1;
  ```

### C. **NULL 连接键永不相等**

- `a.PersonId = NULL` 与任何值比较都不是 TRUE；因此**不会匹配**；
- 在 RIGHT/FULL OUTER JOIN 中，这类右表行仍会被“全保留”策略保留下来，但左列为 `NULL`。

### D. **RIGHT JOIN 常可改 LEFT JOIN**

- 为保持风格统一与可读性，优先用 **LEFT JOIN**，需要“以右表为准”时，调换顺序即可。

### E. **MySQL 的 FULL OUTER JOIN**

- 用 `LEFT JOIN` 和 `RIGHT JOIN` 的 `UNION` 来模拟（见上），并注意去重/过滤条件避免重复。

### F. **友好展示**

- 想把 `NULL` 展示成“Unknown”等提示词，可用 `COALESCE`：

  ```sql
  SELECT
    p.FirstName,
    p.LastName,
    COALESCE(a.City,  'Unknown') AS City,
    COALESCE(a.State, 'Unknown') AS State
  FROM Person p
  LEFT JOIN Address a ON p.PersonId = a.PersonId;
  ```

### G. **性能**

- 连接键上建索引（`Person.PersonId` 主键自带索引；`Address(PersonId)` 建次级索引）能明显加速。

------

## 什么时候选哪种 JOIN？

- **INNER**：两边都必须有记录（例如“只看有地址的人”）。
- **LEFT**：以左表为准（例如“所有人都要列出来，没有地址就空着”）。
- **RIGHT**：以右表为准（不如调换表位次写 LEFT 更直观）。
- **FULL OUTER**：两边独有的也要保留（数据对账、并集视图）。MySQL 需用 `UNION` 模拟。

