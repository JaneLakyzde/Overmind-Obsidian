---
aliases:
  - SQL
  - 结构化查询语言
  - struct query language
date: 2026-03-31
tags:
  - SQL
---
# 1. SQL 特点

1. 综合统一 (DDL +DQL + DML + [[04_数据库的安全性|DCL]])
2. 高度非过程化
3. 面向集合的操作方式
4. 以同一种语法结构提供多种使用方法
5. 语言简洁, 易学易用

# 2. 数据定义语言 DDL

Data Definition Language : 定义数据库结构

### 数据类型
```SQL
CHAR(n)             //定长字符串
VARCHAR(n)          //变长字符串,不超过n
INT(n)
DATE YYYY-MM-DD
TIME HH-MM-SS
```


### 创建表与约束条件
```sql
CREATE DATABASE 数据库名;
CREATE TABLE 表名;
	列名 数据类型 「约束」,
	....
	
);

约束名:

PRIMARY KEY
NOT NULL
UNIQUE
CHECK
DEFAULT
FOREIGN KEY

```
### 核心示例：学生-课程数据库 (S-C-SC)

```SQL

-- 1. 创建学生表 S
CREATE TABLE s (
    sno CHAR(4) NOT NULL, 
    sname CHAR(8), 
    age INT, 
    sex CHAR(2), 
    PRIMARY KEY (sno) -- 实体完整性：主键
);

-- 2. 创建课程表 C
CREATE TABLE c (
    cno CHAR(4) NOT NULL, 
    cname CHAR(10), 
    tname CHAR(8), 
    PRIMARY KEY (cno) -- 实体完整性：主键
);

-- 3. 创建选课表 SC
CREATE TABLE sc (
    sno CHAR(4) NOT NULL, 
    cno CHAR(4) NOT NULL, 
    grade INT, 
    PRIMARY KEY (sno, cno), -- 实体完整性：联合主键
    FOREIGN KEY (sno) REFERENCES s(sno), -- 参照完整性：外键
    FOREIGN KEY (cno) REFERENCES c(cno), -- 参照完整性：外键
    CHECK (grade BETWEEN 0 AND 100)      -- 用户定义的完整性：检查约束
);
````

易错点提示 - 在定义外键时，标准 SQL 语法应使用 `REFERENCES` 关键字
> 
> - 建表与删表顺序：由于 `sc` 表依赖于 `s` 和 `c` 表，因此**建表**时必须先建 `s` 和 `c`，后建 `sc`；
> - 删表 (DROP) 时则相反，必须先删 `sc`，再删 `s` 和 `c`。

### 修改表结构
```sql
ALTER TABLE 表名 ADD/DROP/MODIFY COLUMN 列名 数据类型 「约束」;

- 添加邮箱列:
ALTER TABLE Student ADD email VArCHAR(100) UNIQUE;

- 修改列类型:
ALTER TABLE Student MODIFY age SMALLINT;

- 删除列:
ALTER TABLE Student DROP COLUMN email;
```

### 删除对象
```SQL
删库跑路: DROP DATABASE 数据库名;
删表:    DROP TABLE 表名 [ RESTRICT | CASCADE ];
CASCADE  表示如果有外键、试图触发器的话,一起强行删除; RESTRICT 恰恰相反,不让删!
```

---
# 3. 数据查询语言 DQL

## SELECT  

	在 SELECT 子句中, 凡是没有放进聚合函数里的列，原则上都必须出现在 GROUP BY 中。因为没有分组时，聚合函数会把所有行压成一行结果。  既然只剩一行，那 C.Cname 也只能对应一个值。  但现在明明有多个不同的课程名，所以就冲突了。

```SQL
SELECT 列名 FROM 表名
[WHERE 条件]
[GROUP BY 分组列]
[HAVING 分组条件] 依据分组的选择条件对组进行过滤 (与 GROUP BY 搭配使用)
[ORDER BY 排序列 顺序] ASC表示升序(默认), DESC表示降序



- 简单查询
  SELECT * FROM Student;
  SELECT 列名 别名, 列名 别名 FROM 表名;(其实就是省略了一个 AS)
  SELECT DISTINCT ... 去重
```
`WHERE`: 分组前筛行
`HAVING`: 分组后筛组

条件查询

| 语法                    | 作用         | 示例                                                                                       |
| --------------------- | ---------- | ---------------------------------------------------------------------------------------- |
| `WHERE 比较`            | 指定查询条件     | `SELECT * FROM student WHERE age > 18;`                                                  |
| `BETWEEN ... AND ...` | 在某个范围内     | `SELECT * FROM student WHERE score BETWEEN 60 AND 100;`                                  |
| `IN (...)`            | 在指定集合中     | `SELECT * FROM student WHERE class_id IN (1,2,3);`                                       |
| `NOT IN (...)`        | 不在指定集合中    | `SELECT * FROM student WHERE class_id NOT IN (1,2,3);`                                   |
| `LIKE`                | 模糊查询       | `SELECT * FROM student WHERE name LIKE '张%';`                                            |
| `%`                   | 匹配任意多个字符   | `LIKE '张%'` 表示以“张”开头                                                                     |
| `_`                   | 匹配任意一个字符   | `LIKE '张_'` 表示“张”后只有一个字符                                                                 |
| `IS NULL`             | 判断是否为空     | `SELECT * FROM student WHERE address IS NULL;`                                           |
| `IS NOT NULL`         | 判断是否不为空    | `SELECT * FROM student WHERE address IS NOT NULL;`                                       |
| `AND`                 | 多个条件同时成立   | `SELECT * FROM student WHERE age > 18 AND sex = '女';`                                    |
| `OR`                  | 多个条件任一成立   | `SELECT * FROM student WHERE age < 18 OR age > 60;`                                      |
| `NOT`                 | 对条件取反      | `SELECT * FROM student WHERE NOT age = 18;`                                              |
| `EXISTS`              | 判断子查询是否有结果 | `SELECT * FROM student s WHERE EXISTS (SELECT 1 FROM score sc WHERE sc.sid = s.id);`     |
| `NOT EXISTS`          | 判断子查询是否无结果 | `SELECT * FROM student s WHERE NOT EXISTS (SELECT 1 FROM score sc WHERE sc.sid = s.id);` |
### 核心示例：统计查询

课件中给出了一个典型的高级查询语句，用于统计平均成绩大于 80 分的学生的选课数量 ：
```SQL
SELECT SNO, COUNT(CNO) AS C_NUM       
FROM SC       
GROUP BY SNO            
HAVING AVG(GRADE) > 80;
```

深入理解：`WHERE` vs `HAVING`
> 
> - **`WHERE`**：作用于基本表或视图，在**分组之前**过滤数据。不能包含聚合函数（如 `COUNT`, `AVG`）
> - **`HAVING`**：作用于组，在**分组之后**对分组结果进行过滤。通常与聚合函数配合使用
> 
> **SQL 逻辑执行顺序**：`FROM` -> `WHERE` -> `GROUP BY` -> `HAVING` -> `SELECT` -> `ORDER BY`

## JOIN 多表连接

```SQL
SELECT s.sno, s.sname, sc.cno, sc.grade //查询每个学生的选课和成绩
FROM s
INNER JOIN sc
ON s.sno = sc.sno;


SELECT s.sname, c.cname, sc.grade  //查询学生姓名、课程名、成绩
FROM sc
INNER JOIN s
ON sc.sno = s.sno
INNER JOIN c
ON sc.cno = c.cno; //三表连接 


```


当需要查询的数据分布在多个表中时，必须使用连接操作。课件中列举了所有主流的 SQL 连接类型 。

- **内连接 (INNER JOIN) 等价于 JOIN**:  仅返回两个表中满足连接条件的交集行。如果不加限制，就是两个表的笛卡尔积。
        
- **左外连接 (LEFT OUTER JOIN)** ：返回左表的所有行。如果右表中没有匹配的行，则结果中右表的部分用 `NULL` 填充。
    
```sql
SELECT s.sno, s.sname, sc.cno, sc.grade //查询所有学生及其选课情况
FROM s
LEFT JOIN sc        //如果某个学生没有选课，则课程号、课程成绩为 `NULL`
ON s.sno = sc.sno;
```
        
- **右外连接 (RIGHT  JOIN)** ：与左连接相反，返回右表的所有行.

- **全外连接 (FULL JOIN)** ： 返回左表和右表中的所有行。当某行在另一个表中没有匹配时，另一个表的部分用 `NULL` 填充。
    
- `JOIN ... ON s.sno = sc.sno`：把学生和选课记录连起来
- `JOIN ... ON c.cno = sc.cno`：把课程和选课记录连起来
- 通过 `sc` 这张中间表，就能把学生和课程关联起来

---
## EXIST 子查询

```SQL
- 从学生表中查询学生选修了全部课程的学生信息
SELECT * FROM students s
	WHERE NOT EXISTS(    //当不存在未选修课程时, 说明学生选修了全部课程
		SELECT * FROM courses c
		WHERE NOT EXISTS(    //查找该学生未选修的课程
			SELECT * FROM sc
			WHERE sc.student_id = s.student_id AND sc.course_id = c.course_id //学生选修的课程信息
		)
	);
```

## ANY ALL 字查询

---
# 4. 数据操作语言 DML

- 不同于 `ALTER TABLE` 中的 `ADD/DROP/MODIFY` 主要修改的是表的定义,
- `INSERT/UPDATE/DELETE` 修改的是表中的数据

### 4.1 INSERT

```SQL
INSERT INTO 表名 (列1, 列2, 列3)  
VALUES (值1, 值2, 值3);


INSERT INTO S (Sno, Sname, Sage, Ssex)  
VALUES ('S10', '王明', 20, '男'); //表示向学生表插入一条新记录。

INSERT INTO S                    //省略列名的写法,要求个数与顺序和表的定义完全一致
VALUES ('S10', '王明', 20, '男');

INSERT INTO S (Sno, Sname)       //插入部分列,没写的填入默认值或 NULL
VALUES ('S11', '赵强');

INSERT INTO S (Sno, Sname, Sage, Ssex)  //某些数据库支持：一次插入多行
VALUES  
('S12', '甲', 19, '男'),  
('S13', '乙', 20, '女');

INSERT INTO NewTable (Sno, Sname)  //通过查询结果插入另一张表
SELECT Sno, Sname  
FROM S  
WHERE Sage > 20;
```

### 4.2 UPDATE

```SQL
UPDATE 表名  
SET 列名1 = 值1, 列名2 = 值2  
WHERE 条件;             //不写 where 会修改整个表的所有数据!

UPDATE S  
SET Sage = 21  
WHERE Sno = 'S1';      //表示把学号为 `S1` 的学生年龄改成 21。

UPDATE S  
SET Sname = '张明', Sage = 22  //同时修改多个字段
WHERE Sno = 'S1';

UPDATE S  
SET Sage = Sage + 1    //修改多行,右边可以是表达式，不一定是固定值
WHERE Ssex = '男';

UPDATE S  
SET Sage = Sage + 1  
WHERE Sno IN (         //根据子查询修改,表示把选修了 `C1` 的学生年龄都加 1。
    SELECT Sno  
    FROM SC  
    WHERE Cno = 'C1'  
);

```

### 4.3 DELETE

```SQL
DELETE FROM 表名  
WHERE 条件;

DELETE FROM S  
WHERE Sno = 'S1';      //表示删除学号为 `S1` 的学生记录。

DELETE FROM SC  
WHERE Cno = 'C4';      //表示删除所有选修 `C4` 课程的选课记录。

DELETE FROM SC
WHERE sno IN(          
  SELECT sno          //字查询,优先执行
  FROM s
  WHERE sname = 'WANG'
);

DELETE FROM S;         //这会删除表中所有数据,但是表结构还在
DROP TABLE S;          //删除整张表
```

# 5. 视图 View

## 5.1 创建视图

```SQL
CREATE VIEW view_name AS  
SELECT column1, column2  
FROM table_name  
WHERE condition;
```
## 5.2 查询视图

```SQL
SELECT * FROM view_name;
```

## 5.3 修改视图

```SQL
CREATE OR REPLACE VIEW view_name AS  
SELECT ...  
FROM ...  
WHERE ...;
```
## 5.4 删除视图

```SQL
DROP VIEW view_name;
```
## 更新机制

对于可更新视图，通常要求：$$视图中的一行，能明确对应到基本表中的某一行或某些确定的行。$$
```SQL
DELETE FROM S_GRADE 
WHERE C_NUM > 4;
```
不允许执行。
**理由：**  
`S_GRADE` 是在基本表 `SC` 上经过 `GROUP BY` 和聚集函数 `COUNT`、`AVG` 定义的分组视图。
视图中的一行对应 `SC` 中某个学生的一组元组，而不是某一个元组。
删除满足 `C_NUM > 4` 的视图行时，无法唯一确定应该删除基本表 `SC` 中哪些记录：
可以删除该学生全部选课记录，也可以只删除部分记录使其不再满足 `C_NUM > 4`。
由于对基本表的操作不唯一，因此该删除操作**不能执行**，也不能自然转换为对基本表 `SC` 的删除操作。
