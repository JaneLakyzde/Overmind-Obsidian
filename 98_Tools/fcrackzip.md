---
aliases:
tags:
  - CTF
gloss: ZIP 加密压缩包密码破解工具
---

## 🔐 fcrackzip 使用指南及 CTF 应用

fcrackzip 是一款高效的 ZIP 密码破解工具，在 CTF 比赛中常用于解决加密压缩包类题目（Misc/Forensics 方向）。

---

### 🔧 安装方法
```bash
# Debian/Ubuntu
sudo apt update && sudo apt install fcrackzip

# macOS
brew install fcrackzip

# Windows
下载二进制包：https://github.com/hyc/fcrackzip/releases
```

---

### 🧠 核心命令详解

#### 🔍 基础格式
```bash
fcrackzip [选项] 加密的ZIP文件
```

#### ⚙️ 常用选项
| 选项 | 作用 | 示例 |
|------|------|------|
| `-b` | 暴力破解模式 | `fcrackzip -b -c 'aA1' -l 4-6 file.zip` |
| `-D` | 字典攻击模式 | `fcrackzip -D -p rockyou.txt file.zip` |
| `-p` | 指定字典文件 | `-p /path/to/dict.txt` |
| `-c` | 指定字符集 | `-c 'aA1!'` (小写+大写+数字+符号) |
| `-l` | 密码长度范围 | `-l 6-8` (尝试6-8位密码) |
| `-u` | 跳过错误密码加速 | 必选参数 |
| `-v` | 显示详细过程 | 查看实时破解进度 |

#### 💡 字符集标识
- `a` : 小写字母 (a-z)
- `A` : 大写字母 (A-Z)
- `1` : 数字 (0-9)
- `!` : 特殊符号 (!@#$%^&*等)

---

### 🚀 CTF 实战应用场景

#### 📌 场景 1：基础密码破解
```bash
# 使用 rockyou 字典破解
fcrackzip -D -p /usr/share/wordlists/rockyou.txt -u ctf.zip

# 暴力破解纯数字密码（4-6位）
fcrackzip -b -c '1' -l 4-6 -u ctf.zip
```

#### 📌 场景 2：带提示的密码
```bash
# 已知密码包含 "flag" + 3位数字
fcrackzip -b -c '1' -p 'flag' -l 7-7 -u ctf.zip
```

#### 📌 场景 3：多层压缩包
```python
#!/bin/bash
while [ -f *.zip ]; do
    pass=$(fcrackzip -b -c 'aA1' -l 3-5 -u *.zip | grep pw | awk '{print $NF}')
    unzip -P "$pass" *.zip
    rm -f *.zip
done
```

#### 📌 场景 4：已知部分密码
```bash
# 知道前3位是 "CTF{"，后2位是数字
fcrackzip -b -c '1' -p 'CTF{' -l 5-5 -u ctf.zip
```

---

### 🛠️ CTF 解题技巧

1. **优先使用字典攻击**  
   90% 的 CTF 题目密码在常见字典中：
   ```bash
   fcrackzip -D -p /usr/share/wordlists/rockyou.txt -u ctf.zip
   ```

2. **长度优先策略**  
   优先尝试短密码（4-6位）：
   ```bash
   fcrackzip -b -c 'aA1' -l 4-6 -u ctf.zip
   ```

3. **字符集缩小技巧**  
   根据题目提示缩小范围：
   - 纯数字：`-c '1'`
   - 字母+数字：`-c 'aA1'` 
   - 含符号：`-c 'aA1!'`

4. **批量处理脚本**  
   嵌套压缩包自动化解压：
   ```bash
   while [ -f *.zip ]; do
     pass=$(fcrackzip -b -c 1 -l 4-6 -u *.zip | grep pw | awk '{print $NF}')
     unzip -P $pass *.zip
     rm *.zip
   done
   ```

5. **结合其他工具**  
   当密码在文件注释中：
   ```bash
   zipinfo -v ctf.zip | grep -i "password"
   ```

---

### 💡 性能优化技巧
```bash
# 使用多核加速 (需安装 parallel)
parallel -j 4 fcrackzip -b -c 'aA1' -l {} -u ctf.zip ::: {4..8}

# 生成针对性字典
crunch 6 8 0123456789ABCDEF -o hex_dict.txt
fcrackzip -D -p hex_dict.txt ctf.zip
```

---

### ⚠️ 注意事项
1. 添加 `-u` 参数可提升 30% 以上速度
2. CTF 中密码通常≤8位，优先尝试短密码
3. Linux 下使用 `unzip -P 密码 文件` 解压
4. 遇到伪加密 ZIP 时，使用 `zipdetails` 检查加密头

> 💡 CTF 密码常见规律：题目名缩写、年份、flag、ctf、简单数字序列（123456）等。破解前先尝试这些常见组合！