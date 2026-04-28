---
tags:
  - SQL
---
### 1. 增 (Create / Insert)

向表中插入新数据。

- **基本语法：**

```SQL    
INSERT INTO table_name (column1, column2)
VALUES (value1, value2);
```


---

### 2. 查 (Read / Select)

从表中提取数据。

- **基本语法：**

```SQL
SELECT column1, column2 FROM table_name
WHERE condition
ORDER BY column1 DESC
LIMIT 10;
```

- **常用过滤与排序：**

```SQL
SELECT * FROM products
WHERE price > 100 AND category = 'Electronics'
ORDER BY price ASC
OFFSET 20 LIMIT 10; -- 跳过前20条，取10条（分页常用）
```
    

---

### 3. 改 (Update / Modify)

修改表中已有的数据。

- 基本语法：⚠️ 注意：永远不要忘记 `WHERE` 子句，否则会更新全表数据！

```SQL
UPDATE table_name
SET column1 = value1, column2 = value2
WHERE id = 1;
```

---

### 4. 删 (Delete)

从表中删除数据。
- 基本语法：⚠️ 注意：同样必须紧记 `WHERE` 子句！

```SQL
DELETE FROM table_name
WHERE id = 1;
```

- **清空全表 (Truncate)：**
    如果你想删除表中**所有**数据且重置自增 ID，`TRUNCATE` 比 `DELETE` 快得多。

```SQL
TRUNCATE TABLE table_name RESTART IDENTITY;
```

---
