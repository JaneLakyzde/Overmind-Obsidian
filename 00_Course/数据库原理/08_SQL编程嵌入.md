---
aliases:
tags:
  - SQL
---


将 SQL 语句嵌入到高级程序设计语言（如 C、Java）中，借助宿主语言的控制功能实现过程化编程。
# 嵌入式SQL使用时必须解决的问题

1. 为区分SQL语句与宿主语言语句，在所有的SQL语句前必须加上
	前缀标识“EXEC  SQL”，并以“END EXEC”作为语句结束标志。
	
2. 数据库工作单元和主程序工作单元之间的通讯 
	  允许嵌入的SQL语句引用宿主语言的程序变量 （称为共享变量）。在引用这些变量时必须在这些变量前加冒号“ ：”作为前缀标识，以示与数据库中变量有区别;
3. 引入游标机制：    将集合操作转换为单元组处理。
# 嵌入式 SQL 的处理过程

- 预编译转换为函数调用
- 主语言编译
- 变成主语言所编译的类型

# SQL 与主语言的通信

## SQL 给主语言传递状态

数据库系统通过 SQLCA 向主语言返回 SQL 语句的执行状态，包括错误代码、警告信息等。

SQLCODE：执行结果代码（0 表示成功，负数表示错误，正数表示警告）

```c
EXEC SQL CONNECT TO schooldb USER 'user' PASSWORD 'password';
if (sqlca.sqlcode != 0) {
    printf("连接失败：%s\n", sqlca.sqlerrm.sqlerrmc);
    return 1;
}

EXEC SQL SELECT name INTO :student_name FROM students WHERE id = :student_id;
if (sqlca.sqlcode == 0) {
    printf("学生姓名：%s\n", student_name);
} else {
    printf("查询失败：%s\n", sqlca.sqlerrm.sqlerrmc);
}
```

## 主语言给 SQL 提供参数

主语言变量通过占位符（如 `:variable`）与 SQL 语句交互。在 JDBC 中通过 `PreparedStatement` 设置参数，防止 SQL 注入。

```java
String url = "jdbc:mysql://localhost:3306/schooldb";
String user = "user";
String password = "password";

try (Connection conn = DriverManager.getConnection(url, user, password)) {
    // 参数化查询
    String sql = "SELECT name FROM students WHERE id = ?";
    PreparedStatement pstmt = conn.prepareStatement(sql);
    pstmt.setInt(1, 101); // 设置参数值

    ResultSet rs = pstmt.executeQuery();
    if (rs.next()) {
        System.out.println("学生姓名：" + rs.getString("name"));
    }
} catch (SQLException e) {
    e.printStackTrace();
}
```

## SQL 把查询结果交给主语言处理（游标和主变量实现）

- **游标（Cursor）**：用于遍历多行查询结果集。
- **主变量映射**：将查询结果赋值到主语言变量中。

```java
try (Connection conn = DriverManager.getConnection(url, user, password)) {
    Statement stmt = conn.createStatement();
    ResultSet rs = stmt.executeQuery("SELECT id, name FROM students");
    while (rs.next()) {
        int id = rs.getInt("id");
        String name = rs.getString("name");
        System.out.println("ID：" + id + "，姓名：" + name);
    }
} catch (SQLException e) {
    e.printStackTrace();
}
```


## 游标
```SQL
DECLARE 游标名 CURSOR FOR 查询语句;
OPEN 游标名;
FETCH 游标名 INTO 主变量列表;
CLOSE 游标名;
```
