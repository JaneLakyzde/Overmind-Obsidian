---
aliases:
  - 全局正则表达式打印
  - Global Regular Expression Print
tags:
  - shell
  - bash
---
- **全称**：**G**lobal **R**egular **E**xpression **P**rint（全局正则表达式打印）
- **核心思想**：**行过滤** —— 从文本流或文件中，筛选出匹配指定模式的行，并输出。

## 常用命令速查
| 选项 | 作用 | 示例 |
|------|------|------|
| `-i` | 忽略大小写 | `grep -i 'error' log` |
| `-v` | 反向选择（输出不匹配的行） | `grep -v 'DEBUG' log` |
| `-r` | 递归搜索目录 | `grep -r 'TODO' ./src` |
| `-c` | 统计匹配行数 | `grep -c '200' access.log` |
| `-n` | 显示行号 | `grep -n 'main' app.py` |
| `-o` | 只输出匹配的部分 | `grep -oE '[0-9.]+'` |
| `-E` | 扩展正则（支持 `+`、`|` 等） | `grep -E 'error|fail'` |
| `-w` | 匹配整个单词 | `grep -w 'main'` |

> 多个选项可组合：`grep -in 'error' server.log`（忽略大小写 + 显示行号）

## 与 [[awk]] 联动
`grep` 负责**筛出行**，`awk` 负责**切出列**，两者通过管道串联，形成经典处理流水线。

**示例**：从日志中提取错误时间与模块
```bash
grep 'ERROR' app.log | awk '{print $1, $3}'
```
- `grep` 过滤出包含 `ERROR` 的行
- `awk` 输出每行第 1 列（时间）和第 3 列（模块名）

**更复杂的例子**：统计失败登录的 IP
```bash
grep 'Failed password' auth.log | grep -oE '[0-9.]+' | sort | uniq -c
```
这里先用 `grep` 过滤，再用 `grep -o` 提取 IP，最后交给 `sort`/`uniq` 统计；如果还需要进一步计算，可在末尾加上 `awk` 处理。