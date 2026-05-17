---
aliases:
  - Client url
tags:
  - CTF
date: 2026-04-14
---
# 一、curl 是什么

`curl` 是一个非常常用的命令行工具，用来和 URL 打交道。最常见用途是：

- 发送 HTTP / HTTPS 请求
- 调用 Web API
- 下载文件
- 上传文件
- 测试服务器接口
- 查看响应头、状态码、重定向情况
    

它名字来自 **Client URL**。

你可以把它理解成：

> “在终端里模拟浏览器或程序去请求一个网址的工具”。

例如访问网页、提交表单、调用 REST API、带 token 鉴权请求接口，这些都能用 `curl` 完成。

---

# 二、为什么要学 curl

curl 很适合这些场景：

- 后端开发调试接口
- 测试登录、鉴权、上传下载
- 排查网络问题
- 学习 HTTP 协议
- 自动化脚本里调用接口
    

相比浏览器，curl 的优点是：

- 更精确控制请求
- 能清楚看到请求和响应细节
- 方便写进 shell 脚本
- 适合服务器环境和远程终端

---

# 三、curl 基本语法

最基本形式：

```bash
curl [选项] URL
```

例如：

```bash
curl https://example.com
```

这表示向 `https://example.com` 发起一个请求。

默认情况下，curl 发的是：

- **GET 请求**
- 并把响应内容输出到终端

---

# 四、先理解 HTTP 的基本概念

在学 curl 前，先知道几个 HTTP 基础概念会容易很多。

## 1. 请求方法（Method）

常见有：

- `GET`：获取数据
- `POST`：提交数据
- `PUT`：整体更新数据
- `PATCH`：部分更新数据
- `DELETE`：删除数据

## 2. 请求头（Headers）

告诉服务器一些附加信息，例如：

- 你接受什么格式：`Accept: application/json`
- 你发送的数据类型：`Content-Type: application/json`
- 认证信息：`Authorization: Bearer xxx`

## 3. 请求体（Body）

请求里真正提交的数据，通常用于 `POST` / `PUT` / `PATCH`。

## 4. 响应状态码

常见状态码：

- `200 OK`：成功
- `201 Created`：创建成功
- `400 Bad Request`：请求格式有问题
- `401 Unauthorized`：未认证
- `403 Forbidden`：没有权限
- `404 Not Found`：资源不存在
- `500 Internal Server Error`：服务器错误

---

# 五、安装与检查

## 1. 检查是否已安装

```bash
curl --version
```

## 2. Linux 安装

Debian / Ubuntu：

```bash
sudo apt update
sudo apt install curl
```

CentOS / RHEL：

```bash
sudo yum install curl
```

或：

```bash
sudo dnf install curl
```

## 3. macOS

macOS 通常自带 curl。

```bash
curl --version
```

## 4. Windows

新版本 Windows 通常也自带 curl。

命令提示符或 PowerShell 中执行：

```bash
curl --version
```

注意：在 PowerShell 里，`curl` 有时可能被映射成别名，不同环境下行为可能不完全一样。遇到问题时可显式使用：

```powershell
curl.exe --version
```

---

# 六、最基础用法

## 1. 发 GET 请求

```bash
curl https://httpbin.org/get
```

会返回服务器响应内容。

---

## 2. 请求一个带查询参数的 URL

```bash
curl "https://httpbin.org/get?name=tom&age=20"
```

这里 URL 中的 `?name=tom&age=20` 就是查询参数。

---

## 3. 保存响应到文件

```bash
curl -o result.html https://example.com
```

- `-o`：保存到指定文件
    

如果想用远程文件原名：

```bash
curl -O https://example.com/file.zip
```

- `-O`：按远端文件名保存
    

---

# 七、常用参数说明

下面是最常见的一批参数。
## 1. `-X` 指定请求方法

```bash
curl -X POST https://httpbin.org/post
```

指定使用 `POST` 请求。

常见例子：

```bash
curl -X GET https://httpbin.org/get
curl -X POST https://httpbin.org/post
curl -X PUT https://httpbin.org/put
curl -X DELETE https://httpbin.org/delete
```

说明：很多时候如果你用了 `-d`，curl 会自动变成 `POST`，所以并不总是需要手动写 `-X POST`。

---

## 2. `-H` 添加请求头

```bash
curl -H "Accept: application/json" https://httpbin.org/get
```

添加多个请求头：

```bash
curl \
  -H "Accept: application/json" \
  -H "User-Agent: my-curl-demo" \
  https://httpbin.org/get
```

常见请求头：

```bash
-H "Content-Type: application/json"
-H "Authorization: Bearer YOUR_TOKEN"
-H "Accept: application/json"
```

---

## 3. `-d` 发送请求体数据

### 表单格式

```bash
curl -X POST -d "name=tom&age=20" https://httpbin.org/post
```

这通常是 `application/x-www-form-urlencoded` 风格。

### JSON 格式

```bash
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{"name":"tom","age":20}' \
  https://httpbin.org/post
```

重点：

- JSON 请求通常要加 `Content-Type: application/json`
- JSON 字符串引号要正确转义
- 在 Linux/macOS 下常用单引号包住 JSON
- Windows 命令行里引号规则可能不同

---

## 4. `-i` 显示响应头

```bash
curl -i https://example.com
```

输出中会包含：

- 响应状态行
- 响应头
- 响应体

```http
HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 1234
...
```

---

## 5. `-I` 只获取响应头

```bash
curl -I https://example.com
```

适合查看：

- 状态码
- Content-Type
- 是否重定向
- 文件大小
- 最后修改时间

---

## 6. `-L` 跟随重定向

有些网址会 301 / 302 跳转，如果不加 `-L`，curl 默认不继续跳转。

```bash
curl -L http://example.com
```

---

## 7. `-v` 查看详细过程

```bash
curl -v https://example.com
```

这个非常实用，能看到：

- DNS 解析
- TCP 连接
- TLS 握手
- 请求头
- 响应头

```html
hacker@web-security~authentication-bypass-1:~$ curl -v -X POST "http://challenge.localhost/?admin=1" \1" \
  -d "username=admin&password=test"
  
Note: Unnecessary use of -X or --request, POST is already inferred.

* Host challenge.localhost:80 was resolved.
* IPv6: ::1
* IPv4: 127.0.0.1
  
*   Trying [::1]:80...
* connect to ::1 port 80 from ::1 port 49810 failed: Connection refused
  
*   Trying 127.0.0.1:80...
* Established connection to challenge.localhost (127.0.0.1 port 80) from 127.0.0.1 port 59330 
* using HTTP/1.x
  
> POST /?admin=1 HTTP/1.1
> Host: challenge.localhost
> User-Agent: curl/8.17.0
> Accept: */*
> Content-Length: 28
> Content-Type: application/x-www-form-urlencoded

* upload completely sent off: 28 bytes
  
127.0.0.1 - - [14/Apr/2026 07:40:16] "POST /?admin=1 HTTP/1.1" 403 -

< HTTP/1.1 403 FORBIDDEN
< Server: Werkzeug/3.0.6 Python/3.8.10
< Date: Tue, 14 Apr 2026 07:40:16 GMT
< Content-Type: text/html; charset=utf-8
< Content-Length: 115
< Connection: close
< 
<!doctype html>
<html lang=en>
<title>403 Forbidden</title>
<h1>Forbidden</h1>
<p>Invalid username or password</p>

* shutting down connection #0

```


适合排错。

---

## 8. `-u` 基本认证

```bash
curl -u username:password https://example.com/protected
```

相当于发送 Basic Auth。

---

## 9. `--connect-timeout` 设置连接超时

```bash
curl --connect-timeout 5 https://example.com
```

表示连接 5 秒还没建立就超时。

---

## 10. `-m` 设置总超时

```bash
curl -m 10 https://example.com
```

整个请求最多 10 秒。

---

## 11. `-s` 静默模式

```bash
curl -s https://example.com
```

不显示进度信息。

通常脚本里很常用。

如果希望静默但保留错误信息，可用：

```bash
curl -sS https://example.com
```

---

## 12. `-k` 忽略 HTTPS 证书校验

```bash
curl -k https://example.com
```

适合测试环境、自签名证书环境。

但生产环境不要随便这么用，因为不安全。

---

# 八、最常见的请求场景

---

## 场景 1：GET 获取接口数据

```bash
curl https://api.example.com/users
```

如果返回 JSON，会直接输出 JSON。

例如：

```bash
curl https://httpbin.org/json
```

---

## 场景 2：带查询参数

```bash
curl "https://api.example.com/users?page=1&size=10"
```

URL 中有 `&` 时最好加引号，避免被 shell 误解释。

---

## 场景 3：POST 提交表单

```bash
curl -X POST \
  -d "username=tom&password=123456" \
  https://httpbin.org/post
```

---

## 场景 4：POST 提交 JSON

```bash
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{"username":"tom","password":"123456"}' \
  https://httpbin.org/post
```

---

## 场景 5：带 token 的接口调用

```bash
curl -H "Authorization: Bearer YOUR_TOKEN" \
  https://api.example.com/profile
```

这是最常见的 API 鉴权方式之一。

---

## 场景 6：PUT 更新资源

```bash
curl -X PUT \
  -H "Content-Type: application/json" \
  -d '{"name":"new name"}' \
  https://httpbin.org/put
```

---

## 场景 7：DELETE 删除资源

```bash
curl -X DELETE https://httpbin.org/delete
```

---

# 九、上传文件

## 1. 用 `-F` 模拟表单上传

```bash
curl -X POST \
  -F "file=@test.txt" \
  https://httpbin.org/post
```

说明：

- `-F` 表示 `multipart/form-data`
    
- `@test.txt` 表示上传本地文件
    

也可以同时上传其他字段：

```bash
curl -X POST \
  -F "file=@test.txt" \
  -F "username=tom" \
  https://httpbin.org/post
```

---

# 十、下载文件

## 1. 下载到指定文件

```bash
curl -o demo.zip https://example.com/file.zip
```

## 2. 按原始文件名下载

```bash
curl -O https://example.com/file.zip
```

## 3. 断点续传

```bash
curl -C - -O https://example.com/file.zip
```

`-C -` 表示自动从中断位置继续下载。

---

# 十一、查看返回状态码

有时候你不关心响应体，只想知道请求是否成功。

```bash
curl -o /dev/null -s -w "%{http_code}\n" https://example.com
```

解释：

- `-o /dev/null`：丢弃响应体
    
- `-s`：静默
    
- `-w`：格式化输出
    

输出例如：

```text
200
```

如果还想看总耗时：

```bash
curl -o /dev/null -s -w "code=%{http_code} time=%{time_total}\n" https://example.com
```

---

# 十二、cookie 的基本用法

有些网站登录后会依赖 cookie。

## 1. 保存 cookie

```bash
curl -c cookies.txt https://example.com/login
```

- `-c`：把服务器返回的 cookie 保存到文件
    

## 2. 携带 cookie

```bash
curl -b cookies.txt https://example.com/profile
```

- `-b`：从文件读取 cookie 并发送
    

也可以：

```bash
curl -b "sessionid=abc123" https://example.com
```

---

# 十三、代理请求

如果要通过代理访问：

```bash
curl -x http://127.0.0.1:7890 https://example.com
```

带认证的代理：

```bash
curl -x http://user:pass@127.0.0.1:7890 https://example.com
```

---

# 十四、实战例子

下面给你一些非常典型的例子。

---

## 例 1：调用公开测试接口

```bash
curl https://httpbin.org/get
```

作用：测试 GET 请求。

---

## 例 2：POST 一个 JSON

```bash
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{"title":"hello","body":"world"}' \
  https://httpbin.org/post
```

作用：测试 JSON 请求体是否正确发送。

---

## 例 3：带鉴权头访问接口

```bash
curl -H "Authorization: Bearer abcdef123456" \
  -H "Accept: application/json" \
  https://api.example.com/userinfo
```

---

## 例 4：查看重定向链路

```bash
curl -I http://example.com
```

如果发现 301/302，可以再用：

```bash
curl -IL http://example.com
```

- `-I` 看头
    
- `-L` 跟随跳转
    

---

## 例 5：调试接口失败原因

```bash
curl -v \
  -H "Content-Type: application/json" \
  -d '{"name":"tom"}' \
  https://api.example.com/test
```

用 `-v` 看清：

- 请求是否发出
    
- 请求头是否对
    
- TLS 是否异常
    
- 服务器返回了什么
    

---

# 十五、Linux/macOS 与 Windows 引号差异

这是初学者很容易踩坑的点。

## Linux/macOS

常见写法：

```bash
curl -H "Content-Type: application/json" \
  -d '{"name":"tom"}' \
  https://httpbin.org/post
```

## Windows CMD

双引号处理更麻烦，JSON 常需要转义：

```cmd
curl -H "Content-Type: application/json" -d "{\"name\":\"tom\"}" https://httpbin.org/post
```

## PowerShell

有时也要注意引号规则，必要时可用：

```powershell
curl.exe -H "Content-Type: application/json" -d '{"name":"tom"}' https://httpbin.org/post
```

如果命令不符合预期，先确认你是在：

- CMD
    
- PowerShell
    
- Git Bash
    
- WSL
    

不同终端规则不同。

---

# 十六、curl 返回内容看不懂怎么办

很多 API 返回 JSON，终端里一行很长，不好看。

可以配合 `jq` 美化：

```bash
curl -s https://httpbin.org/json | jq
```

如果没装 `jq`，也可以先原样看。

---

# 十七、最常见报错与排查思路

## 1. `Could not resolve host`

意思：域名解析失败。

排查：

- URL 是否写错
    
- DNS 是否有问题
    
- 网络是否通
    
- 是否需要代理
    

---

## 2. `Connection refused`

意思：目标地址能找到，但端口没人监听。

排查：

- 服务是否启动
    
- 端口是否正确
    
- 防火墙是否拦截
    

---

## 3. `SSL certificate problem`

意思：证书校验失败。

排查：

- 证书是否过期
    
- 域名是否匹配
    
- 是否测试环境自签名证书
    
- 临时测试可用 `-k`
    

---

## 4. 返回 `401`

意思：未认证。

排查：

- token 是否正确
    
- Authorization 头是否带了
    
- token 是否过期
    
- 鉴权格式是否正确，如 `Bearer xxx`
    

---

## 5. 返回 `403`

意思：服务器认识你，但不允许访问。

排查：

- 权限不足
    
- IP 被限制
    
- 账号无该资源权限
    

---

## 6. 返回 `404`

意思：URL 不存在。

排查：

- 路径拼错
    
- 接口版本不对
    
- 请求方法不对时，某些服务也可能表现成 404
    

---

## 7. 返回 `415 Unsupported Media Type`

意思：请求体格式不被接受。

排查：

- 是否缺少 `Content-Type`
    
- JSON 是否真的按 JSON 发
    
- 表单是不是该用 `-F` 或 `-d`
    

---

# 十八、学习 curl 最重要的思路

不要死记参数，先建立这个思路：

一个 curl 请求通常由这几部分组成：

1. **URL**
    
2. **请求方法**
    
3. **请求头**
    
4. **请求体**
    
5. **是否显示详细日志**
    
6. **是否保存输出**
    

例如你看到一个接口文档写着：

- 方法：POST
    
- 地址：`https://api.test.com/login`
    
- Header：`Content-Type: application/json`
    
- Body：
    
    ```json
    {
      "username": "tom",
      "password": "123456"
    }
    ```
    

你就能翻译成：

```bash
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{"username":"tom","password":"123456"}' \
  https://api.test.com/login
```

这就是学 curl 的核心。

---

# 十九、给初学者的高频命令模板

下面这些模板很值得保存。

## 1. GET 请求

```bash
curl "https://api.example.com/data?id=1"
```

## 2. GET + 请求头

```bash
curl -H "Authorization: Bearer TOKEN" \
  https://api.example.com/user
```

## 3. POST 表单

```bash
curl -X POST \
  -d "name=tom&age=20" \
  https://api.example.com/form
```

## 4. POST JSON

```bash
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{"name":"tom","age":20}' \
  https://api.example.com/users
```

## 5. 上传文件

```bash
curl -X POST \
  -F "file=@a.txt" \
  https://api.example.com/upload
```

## 6. 下载文件

```bash
curl -O https://example.com/file.zip
```

## 7. 查看响应头

```bash
curl -I https://example.com
```

## 8. 调试模式

```bash
curl -v https://example.com
```

## 9. 只取状态码

```bash
curl -o /dev/null -s -w "%{http_code}\n" https://example.com
```

---

# 二十、一个完整的练习路线

你可以按这个顺序练：

## 第 1 步：只会 GET

```bash
curl https://httpbin.org/get
```

## 第 2 步：会带参数

```bash
curl "https://httpbin.org/get?name=tom"
```

## 第 3 步：会看响应头

```bash
curl -i https://httpbin.org/get
curl -I https://httpbin.org/get
```

## 第 4 步：会 POST 表单

```bash
curl -X POST -d "name=tom" https://httpbin.org/post
```

## 第 5 步：会 POST JSON

```bash
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{"name":"tom"}' \
  https://httpbin.org/post
```

## 第 6 步：会带 token

```bash
curl -H "Authorization: Bearer testtoken" \
  https://httpbin.org/get
```

## 第 7 步：会上传和下载

```bash
curl -F "file=@test.txt" https://httpbin.org/post
curl -O https://example.com/file.zip
```

## 第 8 步：会排错

```bash
curl -v https://example.com
```

---

# 二十一、什么时候该用 curl，什么时候不该用

适合用 curl：

- 调接口
    
- 验证 API
    
- 自动化脚本
    
- 快速排查网络请求问题
    

不太适合用 curl：

- 复杂浏览器交互页面
    
- 依赖 JavaScript 渲染的网页
    
- 大型接口测试管理场景
    

那种情况下，可能更适合：

- Postman
    
- Insomnia
    
- 浏览器开发者工具
    
- Python requests
    
- 前端脚本 / 自动化测试框架
    

---

# 二十二、一句话总结

你可以把 curl 记成下面这句话：

> curl 就是在命令行里构造 HTTP 请求、观察 HTTP 响应的万能小工具。

入门最重要的是会这四件事：

- 发 GET
    
- 发 POST
    
- 加 Header
    
- 传 Body
    

只要这四个掌握了，后面上传、下载、鉴权、排错都只是继续加参数。

---

# 二十三、一个最小速查表

```bash
# GET
curl https://example.com

# GET with header
curl -H "Accept: application/json" https://example.com

# POST form
curl -X POST -d "a=1&b=2" https://example.com

# POST json
curl -X POST -H "Content-Type: application/json" -d '{"a":1}' https://example.com

# show headers
curl -i https://example.com

# head only
curl -I https://example.com

# verbose
curl -v https://example.com

# follow redirect
curl -L http://example.com

# download
curl -O https://example.com/file.zip

# upload
curl -F "file=@test.txt" https://example.com/upload
```