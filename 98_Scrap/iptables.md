---
tags:
  - CTF
---
# 四表五链

iptables 的逻辑可以概括为：**规则放在链里，链放在表里**。
数据包在内核中会按照既定顺序，穿过不同的表和链，与上面的规则进行匹配。

|链名|触发时机|
|---|---|
|**PREROUTING**|数据包刚到达网络接口，还未进行路由判断|
|**INPUT**|数据包经路由判断后，目的地是本机|
|**FORWARD**|数据包经路由判断后，目的地是其他主机（本机作为路由器）|
|**OUTPUT**|本机进程生成的数据包，刚离开进程|
|**POSTROUTING**|数据包即将离开网络接口（路由决策已完成）|

| 表名         | 功能                     | 包含的链                            |
| ---------- | ---------------------- | ------------------------------- |
| **raw**    | 跳过连接跟踪（NOTRACK），用于性能优化 | PREROUTING, OUTPUT              |
| **mangle** | 修改数据包（如 TOS、TTL）       | 全部 5 个链                         |
| **nat**    | 网络地址转换（SNAT/DNAT）      | PREROUTING, OUTPUT, POSTROUTING |
| **filter** | 包过滤（允许/拒绝）             | INPUT, FORWARD, OUTPUT          |

# 数据包路径

- **入站:** `网卡` → `PREROUTING` → `路由判断` → `INPUT`→ `本机进程`
- **转发:** `网卡` → `PREROUTING` → `路由判断` → `FORWARD` → `POSTROUTING` → `网卡`
- **出站:** `本机进程` → `OUTPUT` → `路由判断` → `POSTROUTING` → `网卡`

# 实战命令


```bash
iptables [-t 表名] 命令 [链名] [匹配条件] [-j 目标动作]
```

#### **常用命令速查**

*   **查看规则**
```bash
# 查看 filter 表的所有规则（显示行号、详细信息）
iptables -L --line-numbers -v
# 查看 nat 表的规则
iptables -t nat -L -v -n
```
*   **管理规则**
```bash
# 添加规则（--Append 追加到末尾，--Insert 插入到开头）
iptables -A INPUT -s 192.168.1.100 -j DROP
# 删除规则（--source 表示地址, --port 表示端口, --jump 表示跳转）
iptables -D INPUT -s 192.168.1.100 -j DROP
# 清空规则（--Flush 全部或指定链）
iptables -F
```
*   **设置策略**
```bash
# 设置 INPUT 链的默认策略 Policy 为丢弃
iptables -P INPUT DROP
```

# **实战案例：构建安全的Web服务器**

假设有一台 Web 服务器（IP: 192.168.1.10），目标是：仅允许所有人访问其 HTTP/HTTPS 服务，运维人员（IP: 192.168.1.100）能 SSH 登录，其余一切入站流量均被拒绝。

1.  **清理与设定基础策略**
    ```bash
    # 清空所有现有规则，防止冲突
    iptables -F
    # 将所有链的默认策略设为 DROP (INPUT和FORWARD)，出站设为 ACCEPT
    iptables -P INPUT DROP
    iptables -P FORWARD DROP
    iptables -P OUTPUT ACCEPT
    ```

2.  **信任本地和“现有连接”**
    ```bash
    # 放行本地环回接口，许多服务依赖于此
    iptables -A INPUT -i lo -j ACCEPT
    # 允许已建立或相关的连接，避免已连接会话中断
    iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
    ```

3.  **按需开放端口**
    ```bash
    # 开放 HTTP (80) 和 HTTPS (443)
    iptables -A INPUT -p tcp --dport 80 -j ACCEPT
    iptables -A INPUT -p tcp --dport 443 -j ACCEPT
    # 仅允许运维IP访问 SSH (22)
    iptables -A INPUT -p tcp --dport 22 -s 192.168.1.100 -j ACCEPT
    ```

4.  **查看与验证**
    ```bash
    # 查看已生效的规则，确认配置无误
    iptables -L -v --line-numbers
    ```

# **高级进阶：NAT 与端口转发**

在构建小型局域网时，常用 iptables **将 Linux 主机配置为路由器**。这需要启用 IP 转发并添加 NAT 规则。

1.  **启用 IP 转发**（打开路由功能）
    ```bash
    echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
    sysctl -p
    ```

2.  **配置 SNAT（共享上网）**
    ```bash
    # 让内网 /24 网段通过 eth0 网卡的 IP 上网
    iptables -t nat -A POSTROUTING -s 192.168.1.0/24 -o eth0 -j MASQUERADE
    ```

3.  **配置 DNAT（端口转发）**
    ```bash
    # 将外网访问本机 8080 端口的流量，转发给内网主机 192.168.1.100 的 80 端口
    iptables -t nat -A PREROUTING -p tcp --dport 8080 -j DNAT --to-destination 192.168.1.100:80
    ```

#### **性能与安全：规则优化建议**

1.  **遵循“最小权限”与“策略顺序”**：规则按顺序匹配，应该将最具体、最可能匹配的规则放在前面。
2.  **善用“状态跟踪”**：使用 `-m state --state ESTABLISHED,RELATED` 能有效提升防火墙效率。
3.  **熟悉“常用模块”**：
    *   `state/conntrack`：连接状态跟踪。
    *   `multiport`：匹配多个不连续的端口。
    *   `limit`：限制流量速率，可用于防DDoS。
    *   `connlimit`：限制同一IP的最大并发连接数。
    *   `ipset`：管理大量IP地址池，性能极佳。

#### **如何让规则“永久生效”？**

命令行配置的规则在重启后会丢失，需要持久化保存。Debian/Ubuntu 系统方法如下：
1.  **安装持久化工具**：`sudo apt-get install iptables-persistent netfilter-persistent`
2.  **保存规则**：`sudo netfilter-persistent save`