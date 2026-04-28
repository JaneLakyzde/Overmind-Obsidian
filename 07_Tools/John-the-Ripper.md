---
aliases:
  - John
tags:
  - CTF
gloss: 开源密码破解工具
---

John the Ripper（简称 John）是一款强大的开源密码破解工具，主要用于检测弱密码。以下是详细使用指南：

---

### **安装方法**
#### Linux (Debian/Ubuntu)
```bash
sudo apt update
sudo apt install john -y
```

#### macOS
```bash
brew install john
```

#### Windows
1. 下载官方二进制包：  
   [https://www.openwall.com/john/](https://www.openwall.com/john/)
2. 解压后通过命令行使用

---

### **核心使用步骤**
#### 1. **准备密码文件**
- 从目标系统提取密码哈希（如 Linux 的 `/etc/shadow`）  
- 合并 `passwd` 和 `shadow` 文件（Linux 系统）：
  ```bash
  unshadow /etc/passwd /etc/shadow > hashes.txt
  ```

#### 2. **基础破解命令**
```bash
john [选项] 哈希文件
```
**常用选项**：
- `--wordlist=字典路径`：指定字典文件（如 `rockyou.txt`）
- `--format=哈希类型`：手动指定哈希格式（如 `md5crypt`, `sha512crypt`）
- `--show`：显示已破解的密码
- `--users=用户名`：只破解指定用户

---

### **实战示例**
#### 示例 1：字典攻击
```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt
```

#### 示例 2：破解特定哈希类型
```bash
john --format=sha512crypt --wordlist=dict.txt hashes.txt
```

#### 示例 3：显示结果
```bash
john --show hashes.txt
```
输出格式：`用户名:密码`

---

### **高级技巧**
#### 1. **暴力破解模式**
```bash
john --incremental hashes.txt  # 使用内置字符集
john --incremental=Alpha hashes.txt  # 指定字符集（Alpha 字母）
```

#### 2. **规则攻击**
修改字典中的单词（如大小写变换、添加数字）：
```bash
john --wordlist=dict.txt --rules hashes.txt
```

#### 3. **会话管理**
- 暂停破解：`Ctrl+C`
- 恢复破解：`john --restore`
- 查看状态：`john --status`

---

### **常用命令速查**
| 命令                               | 作用        |
| -------------------------------- | --------- |
| `john --list=formats`            | 查看支持的哈希类型 |
| `john --test`                    | 测试破解速度    |
| `john --make-charset=custom.chr` | 生成自定义字符集  |
| `john --shells=跳过用户`             | 排除特定用户    |

---

### **注意事项**
1. ⚠️ **法律合规**：仅用于授权测试或自有系统审计，禁止非法使用。
2. **哈希识别**：John 通常能自动识别常见哈希类型，若不成功需手动指定 `--format`。
3. **性能优化**：
   - 使用 GPU 加速：安装 `john-opencl` 包
   - 多线程：`--fork=4`（使用 4 个线程）
4. **字典资源**：
   - 推荐字典：`rockyou.txt`、`SecLists`
   - 生成字典：使用 `crunch` 或 `cewl`

---

### **示例完整流程**
```bash
# 提取哈希
sudo unshadow /etc/passwd /etc/shadow > target_hashes.txt

# 字典攻击
john --wordlist=/path/to/rockyou.txt target_hashes.txt

# 显示结果
john --show target_hashes.txt
```

输出：
```
user1:password123
user2:admin@2024
```

> 💡 提示：首次运行会生成配置文件 `~/.john/john.conf` 可自定义规则。