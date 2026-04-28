---
tags:
  - python
gloss: 交互式报文操作库 + 报文构造工具
---
**Scapy 是一个用 Python“造包、发包、抓包、看包”的工具。**

你现在做实验时，最常用它来：

- 手工构造网络包
    
- 把包发出去
    
- 抓别人的包
    
- 看包里每个字段长什么样
    

---

## 你可以把它想成什么

平时网络通信里，系统会自动帮你处理很多东西。  
Scapy 的作用就是：

**让你自己决定包里面每个字段写什么。**

比如你可以自己指定：

- 目标 IP
    
- 目标 MAC
    
- TCP 端口
    
- TCP flags
    
- ARP 里的 IP 和 MAC 对应关系
    

这就是它为什么很适合做网络实验。

---

## Scapy 主要能干什么

先记这 4 个功能就够了：

### 1. 造包

比如造一个 TCP 包：

```python
IP(dst="10.0.0.2") / TCP(dport=80)
```

意思是：

- 发给 `10.0.0.2`
    
- TCP 目标端口是 80
    

---

### 2. 发包

Scapy 可以把你造好的包发出去。

常见有两个：

- `send()`：发三层包（IP、TCP、UDP 这些）
    
- `sendp()`：发二层包（Ethernet 帧）
    

---

### 3. 抓包

比如抓网络里的 TCP/ARP 包：

```python
sniff()
```

---

### 4. 看包内容

比如查看包里各字段：

```python
pkt.show()
```

这个非常好用，做实验前建议经常看。

---

## 最常见的协议对象

你先记这几个名字就够用了：

- `Ether()`：以太网层
    
- `ARP()`：ARP 包
    
- `IP()`：IP 包
    
- `TCP()`：TCP 包
    
- `UDP()`：UDP 包
    
- `ICMP()`：ICMP 包（比如 ping）
    

---

## 最常见的写法

Scapy 里最核心的写法就是：

```python
Ether() / IP() / TCP()
```

这个 `/` 的意思可以理解成：

**把下一层装到上一层里面。**

比如：

```python
Ether() / IP(dst="10.0.0.2") / TCP(dport=80)
```

意思是：

- 最外层是 Ethernet
    
- 里面是 IP
    
- 里面再装 TCP
    

这就是“封装”。

---

## 你实验里最常用的几个命令

### 1. 导入

```python
from scapy.all import *
```

或者更稳一点：

```python
from scapy.all import Ether, ARP, IP, TCP, send, sendp, sniff
```

---

### 2. 创建一个包

```python
pkt = IP(dst="10.0.0.2") / TCP(dport=80)
```

---

### 3. 查看包内容

```python
pkt.show()
```

---

### 4. 简单查看摘要

```python
pkt.summary()
```

---

### 5. 发三层包

```python
send(pkt)
```

适合：

```python
IP() / TCP()
IP() / ICMP()
```

---

### 6. 发二层包

```python
sendp(pkt, iface="eth0")
```

适合：

```python
Ether() / ARP()
Ether() / IP() / TCP()
```

---

### 7. 抓包

```python
sniff(iface="eth0")
```

指定抓几个：

```python
sniff(iface="eth0", count=5)
```

---

## `send()` 和 `sendp()` 的区别

这是新手最容易混的点。

### `send()`

发的是 **三层包**

比如：

```python
send(IP(dst="10.0.0.2") / TCP(dport=80))
```

你不用自己写 MAC 地址。

---

### `sendp()`

发的是 **二层帧**

比如：

```python
sendp(Ether(dst="62:40:a8:89:65:11") / ARP(...), iface="eth0")
```

你要自己指定网卡，有时还要自己写目标 MAC。

---

## 你的实验里最常见的 3 类写法

### 1. 发 Ethernet 帧

```python
pkt = Ether(dst="62:40:a8:89:65:11", type=0xFFFF) / b"hello"
sendp(pkt, iface="eth0")
```

---

### 2. 发 TCP 包

```python
pkt = IP(dst="10.0.0.2") / TCP(sport=31337, dport=31337)
send(pkt)
```

或者完整一点：

```python
pkt = Ether(dst="62:40:a8:89:65:11") / IP(dst="10.0.0.2") / TCP(sport=31337, dport=31337)
sendp(pkt, iface="eth0")
```

---

### 3. 发 ARP 包

```python
pkt = Ether(dst="62:40:a8:89:65:11") / ARP(op=2, psrc="10.0.0.42", hwsrc="42:42:42:42:42:42", pdst="10.0.0.2")
sendp(pkt, iface="eth0")
```

---

## 常见字段你只要先认识这些

### Ether 里常见的

- `src`：源 MAC
    
- `dst`：目标 MAC
    
- `type`：协议类型
    

### IP 里常见的

- `src`：源 IP
    
- `dst`：目标 IP
    

### TCP 里常见的

- `sport`：源端口
    
- `dport`：目标端口
    
- `seq`：序列号
    
- `ack`：确认号
    
- `flags`：标志位
    

### ARP 里常见的

- `op`：1 是请求，2 是应答
    
- `psrc`：发送方 IP
    
- `hwsrc`：发送方 MAC
    
- `pdst`：目标 IP
    
- `hwdst`：目标 MAC
    

---
