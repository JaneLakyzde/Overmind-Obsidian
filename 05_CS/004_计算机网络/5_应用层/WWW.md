---
aliases:
  - 万维网
  - World Wide Web
  - HTTP
tags:
  - 计算机网络
  - 应用层
  - TCP
date: 2026-09-21
---
# 统一资源定位符URL
`协议://主机:端口/路径`
HTML + CSS + Javascript

# 超文本传输协议 HTTP
定义了浏览器怎么向万维网服务器请求文档
以及万维网服务器怎么 把万维网文档传送给浏览器
HTTP 面向文本, 其报文中的每一个字段都是[[ASCII]]码串, 且每个字段的长度都是不确定的

**HTTP/1.0 非持续连接**
- 每请求一个文档就有2 x RTT + 传输时延
- 为了减小时延, 建立多个 TCP 连接同时请求多个对象, 负担重
**HTTP/1.1 持续连接**
- 流水线工作方式, 减少了很多RTT
- 可选择非持续连接, 具体情况查看具体请求报文



# HTTP 报文格式

## 请求报文

```http
GET /index.html HTTP/1.1
Host: www.example.com
User-Agent: Mozilla/5.0
Accept: text/html
Connection: keep-alive
```
## 响应报文

```http
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8
Content-Length: 138
Cache-Control: no-cache

<html>
  <body>Hello World</body>
</html>
```

---

- HTTP 是一种无状态的协议
	使用[[Cookie]]保存用户的状态, 实现服务器对用户的识别(有状态)
- 万维网可以通过缓存机制以提高万维网的效率
- 万维网缓存又称为 Web缓存, 中间系统上的 Web缓存称为代理服务器
- 可以将最近的一些请求/响应缓存在本地磁盘中, 不用再次访问因特网
	每个缓存资源都有计时器保证时效性
- 请求一个网页+两个图像, 非流水线方式下, 需要4个RTT
	三次握手一个RTT, html, JPEG, 各一个

# HTTP 状态码