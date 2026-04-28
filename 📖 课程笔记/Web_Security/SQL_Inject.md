---
aliases:
  - SQL注入
---

# 一、SQL 的基本框架和注入原理

## 1. 基础 SQL 查询框架

在 Web 应用中，程序需要根据用户输入来查询数据库。一个典型的登录验证查询如下：
```sql
SELECT * FROM users WHERE username = 'account' AND password = 'secret';
```

这里的 `'account'` 和 `'secret'` 通常来自用户的输入。

## 2. SQL 注入的核心原理

**根本原因：** 程序将用户输入的数据错误地当作 SQL 命令（代码）来执行。

当一个 Web 应用直接将用户输入拼接到 SQL 查询字符串中时，攻击者可以构造特殊输入，闭合原有的数据引号并注入新的 SQL 命令。

> **注意（示例 - 不安全的伪代码）**
>
```python
 # 危险：直接使用字符串拼接
 user_input = get_user_input() # 假设用户输入了 "' OR '1'='1"
 query = "SELECT * FROM users WHERE username = '" + user_input + "'"
 ```

拼接后的恶意 SQL 可能变为：

```sql
SELECT * FROM users WHERE username = '' OR '1'='1';
```

数据库会执行这个被篡改后的命令，导致查询条件总为真，从而绕过验证。

---

# 二、常见 SQL 注入手段与实例

## 1. 经典布尔盲注（绕过登录）

这是最基础的注入，目标是让 **WHERE 查询条件** 永远为真。

- **Payload:** `' OR '1'='1`

**例题 1：绕过登录验证**

- 场景：登录表单需要用户名和密码。  
- 后台 SQL（预期）：

```sql
SELECT * FROM users WHERE username = '[username]' AND password = '[password]';
```

- 攻击者输入：
  - Username: `' OR '1'='1' --`
  - Password: `password`（任意）

- 服务器拼接后：

```sql
SELECT * FROM users WHERE username = '' OR '1'='1' -- ' AND password = 'password';
```

- 原理解析：
  1. `username = ''` 为 False。
  2. `OR '1'='1'` 为 True。
  3. `--`（或 `#` 在 MySQL 中）为注释符，注释掉后续内容。
  4. 最终 `WHERE (False OR True)`，条件为真，查询返回记录，可能导致绕过认证。

---

## 2. UNION 注入（窃取数据）

使用 `UNION` 合并查询结果以泄露敏感数据。

**例题 2：获取数据库所有用户名和密码**

- 场景：产品页 `.../products.php?id=1`。  
- 后台 SQL（预期）：

```sql
SELECT name, description, price FROM products WHERE id = 1;
```

- 攻击步骤：
  1. **猜解列数**（使 `UNION` 两侧列数一致），例如试 `ORDER BY` 找到列数。
     - `1' ORDER BY 3 --` 正常，`1' ORDER BY 4 --` 报错 -> 原始查询有 3 列。
  2. **窃取数据**：
     - Payload: `1' UNION SELECT NULL, username, password FROM users --`
     - `NULL` 用作占位。

- 拼接后：

```sql
SELECT name, description, price FROM products WHERE id = '1'
UNION
SELECT NULL, username, password FROM users
-- ';
```

- 原理：数据库合并结果，前端显示 `users` 表中的 `username`、`password`，造成数据泄露。

---

## 3. 绕过单引号过滤

当 `'` 被过滤或转义时，攻击者常用替代手段：

1. **十六进制表示（MySQL）**
   - `'admin'` 等同于 `0x61646d696e`。
   - Payload：`... WHERE username = 0x61646d696e`

2. **CHAR() 函数（MySQL）**
   - `'admin'` 等同于 `CHAR(97,100,109,105,110)`。
   - Payload：`... WHERE username = CHAR(97,100,109,105,110)`

3. **宽字节编码绕过（GBK）**
   - 当数据库/过滤器使用 GBK 编码时，某些字节组合可使转义字符与后续字符合并，导致过滤失效（示例：`%df%27` 等）。
   - 这类攻击依赖字符集与编码处理细节，较为复杂但有效。

---

# 三、怎样防止 SQL 注入

## 1. 参数化查询（Parameterized Queries）——根本解决方案

**核心思想：** 将 SQL 命令（代码）和用户数据（参数）彻底分离。使用占位符（`?`、`:name` 等）并通过数据库驱动绑定参数，驱动会确保参数被当作数据处理。

**示例（Python + sqlite3）**

- 不安全（字符串拼接）：

```python
import sqlite3
db = sqlite3.connect(':memory:')

username_input = "' OR '1'='1"
query = "SELECT * FROM users WHERE username = '" + username_input + "'"
# db.execute(query)  # 会导致注入
```

- 安全（参数化查询）：

```python
import sqlite3
db = sqlite3.connect(':memory:')

username_input = "' OR '1'='1"
query_template = "SELECT * FROM users WHERE username = ?"
data_values = (username_input,)

cursor = db.execute(query_template, data_values)
```

> 数据库会把 `?` 视为占位符，参数作为纯数据绑定，不会被当作 SQL 语句的一部分，因此防止注入。

---

## 2. 输入过滤与验证（纵深防御）

- **白名单（推荐）**：只允许符合预期格式的输入（如用户 ID 仅允许数字）。
- **黑名单（不推荐）**：阻止已知危险字符/关键字（容易被绕过，如编码、大小写变体、函数替代等）。

## 3. 最小权限原则（Least Privilege）

为应用分配数据库账号**仅具有必须权限**：

- 展示产品的应用账号只需对 `products` 表有 `SELECT` 权限。
- 不授予 `DROP`、`TRUNCATE`、对 `mysql.user` / `information_schema` 的访问权限。
- 若注入成功，权限受限可降低攻击影响范围。

---

# 附录：防护清单（Checklist）

- [ ] 全站使用参数化查询 / ORM 的绑定参数功能
- [ ] 对外输入实施白名单校验（按字段）
- [ ] 禁用数据库多语句执行（若数据库驱动支持）
- [ ] 为不同服务使用不同数据库账号并最小化权限
- [ ] 安全日志与报警：检测异常查询模式（大量 `UNION`、`ORDER BY` 猜列尝试等）
- [ ] 定期进行安全扫描与渗透测试（包括盲注、时间盲注、编码绕过测试）
- [ ] 对敏感数据使用加密/哈希存储（如密码使用 bcrypt/scrypt/argon2）

---

# 参考示例（常见 Payload 汇总）

- 绕过登录（MySQL）：
  - `username: ' OR '1'='1' --`
- UNION 注入占位：
  - `id=1' UNION SELECT NULL, username, password FROM users -- `
- 使用 CHAR：
  - `username=CHAR(97,100,109,105,110)`(admin)
