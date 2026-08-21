---
aliases:
  - Secure Sockets Layer
  - 安全嵌套层
tags:
date: 2026-03-26
gloss: 加密与证书工具箱
---
OpenSSL 可以把它理解成一个**加密与证书工具箱**。  
它既是一套程序库，也带一个很常用的命令行工具 `openssl`。官方文档把它定位为：可在命令行里做密钥管理、证书/CSR 处理、摘要计算、加解密，以及 SSL/TLS 连接测试。

对新手来说，最常见的用途就 4 类：

1. **生成密钥**  
    比如生成 RSA、EC 私钥。OpenSSL 3.x 里常用 `genpkey` 来生成私钥。
    
2. **制作和查看证书**  
    比如自签名证书、CSR（证书签名请求）、查看证书内容。`req` 负责 CSR 和部分证书生成，`x509` 负责查看和处理证书。
    
3. **做哈希/加解密**  
    例如算 SHA-256、对文件做对称加密等。OpenSSL 命令集合中包含摘要和密码算法相关功能。
    
4. **测试 HTTPS / TLS 连接**  
    比如排查一个网站证书是否正常、TLS 握手是否成功。`s_client` 是最常用的排查命令之一。官方说明它主要用于测试。
    
---

## 先记住这句话

**OpenSSL 不是“只做 HTTPS”的工具。**  
它覆盖了“密钥、证书、签名、加密、[[SSL和TLS|TLS]] 测试”这一整套常见安全操作。

---

## 新手最常用的命令

### 1）看版本

```bash
openssl version
```

用途：确认机器上有没有装 OpenSSL，以及版本号。  
官方也提供 `version` 子命令用于查看版本信息。

---

### 2）查看有哪些命令/算法

```bash
openssl list -commands
openssl list -digest-algorithms
openssl list -cipher-algorithms
```

用途：不知道能做什么时先看看。`list` 可以列出命令、摘要算法、加密算法等。

---

### 3）生成私钥

```bash
openssl genpkey -algorithm RSA -out private.key
```

用途：生成一把 RSA 私钥。  
`genpkey` 是 OpenSSL 3.x 官方文档里的通用私钥生成命令。

如果想生成 2048 位 RSA：

```bash
openssl genpkey -algorithm RSA \
  -pkeyopt rsa_keygen_bits:2048 \
  -out private.key
```

---

### 4）根据私钥生成 CSR

```bash
openssl req -new -key private.key -out request.csr
```

用途：生成证书申请文件，通常拿去给 CA 签发证书。  
`req` 的主要职责就是创建和处理 CSR。

---

### 5）生成自签名证书

```bash
openssl req -x509 -new -key private.key -out cert.pem -days 365
```

用途：自己签一个证书，常用于本地开发、测试环境。  
`req` 可以附带生成自签名证书。
---

### 6）查看证书内容

```bash
openssl x509 -in cert.pem -text -noout
```

用途：查看证书的有效期、主题、颁发者、公钥信息等。  
`x509` 是处理和显示 X.509 证书的核心命令。

---

### 7）测试网站 TLS/HTTPS

```bash
openssl s_client -connect example.com:443 -servername example.com
```

用途：检查某网站的 TLS 握手、证书链、协议协商情况。  
`-servername` 用于 SNI，现代 HTTPS 站点经常需要。`s_client` 官方就说它是通用 TLS 客户端测试工具。

---

### 8）计算文件哈希

```bash
openssl dgst -sha256 file.txt
```

用途：算文件 SHA-256，常用于校验文件是否被改动。  
OpenSSL 官方命令集合包含 Message Digests 功能。

---

## 一个最实用的理解方式

你可以把 OpenSSL 里的对象分成 3 个：

- **私钥**：最重要，不能泄露
    
- **CSR**：向证书颁发机构“申请证书”的材料
    
- **证书**：公开给别人看的身份证明
    

最常见流程就是：

```text
生成私钥 -> 生成 CSR -> CA 签发证书 -> 用证书部署 HTTPS
```

而在开发测试时，常会走简化版：

```text
生成私钥 -> 自签名证书 -> 本地 HTTPS 测试
```

这些流程正对应 `genpkey`、`req`、`x509` 这些命令。

---

## 新手最容易踩的坑

### 私钥和证书别混了

- `.key` 往往是私钥
    
- `.csr` 是申请文件
    
- `.crt` / `.pem` 常是证书
    

### `PEM` 和 `DER` 是格式，不是“不同类型证书”

`req` 和 `x509` 文档都提到输入输出格式可选 PEM/DER。PEM 通常是文本，DER 通常是二进制。

### `s_client` 主要是测试，不是日常浏览器替代品

官方明确说它主要用于测试 TLS。

### 有些老命令还在，但新版本更推荐通用命令

例如官方文档里提到 `rsautl` 已被 `pkeyutl` 取代。新手尽量优先学新文档里的主流命令。

---

## 给新手的最小学习顺序

建议按这个顺序学：

1. `openssl version`
    
2. `openssl list -commands`
    
3. `openssl genpkey`
    
4. `openssl req`
    
5. `openssl x509`
    
6. `openssl s_client`
    

这样基本就能覆盖：  
**看环境 → 生成密钥 → 生成证书请求/自签证书 → 查看证书 → 排查 HTTPS**

---

## 一句话总结

**OpenSSL 是一个用来处理密钥、证书、加密和 TLS 连接测试的命令行工具箱。**  
新手最常用的是：`genpkey`、`req`、`x509`、`s_client`。