---
aliases:
tags:
  - bash
---
- **全称**：**A**ho, **W**einberger, **K**ernighan（三位作者姓氏，直接读作 “awk”）
- **核心思想**：**按列处理** —— 将文本视为“记录（行）+ 字段（列）”的结构，逐行执行过滤、计算、格式化输出。

## 常用命令速查
| 写法 / 选项 | 作用 | 示例 |
|------------|------|------|
| `'{print $1, $3}'` | 输出第1、3列（默认空格分隔） | `awk '{print $1, $3}' data.txt` |
| `-F:` | 指定输入分隔符 | `awk -F: '{print $1}' /etc/passwd` |
| `$2 > 100` | 条件过滤：第2列大于100的行 | `awk '$2 > 100' file` |
| `NR` | 内置变量：当前行号 | `awk 'NR==1 {print}'` (打印第1行) |
| `NF` | 内置变量：当前行的字段数 | `awk '{print $NF}'` (打印最后一列) |
| `BEGIN{}` | 在处理第一行**之前**执行 | `awk 'BEGIN {sum=0} {sum+=$1} END{print sum}'` |
| `END{}` | 在处理完所有行**之后**执行 | 同上（计算总和） |
| `-v var=value` | 从外部传入变量 | `awk -v threshold=100 '$2 > threshold'` |

> 条件与动作可自由组合：`awk '条件 {动作}'`，若省略动作则默认 `{print}`；省略条件则每一行都执行动作。

## 与 [[grep]] 联动
`grep` 粗选行，`awk` 精切列、算数值——二者搭配构成文本处理的核心流水线。

**示例**：统计 ERROR 日志中每个模块的出现次数  
```bash
grep 'ERROR' app.log | awk '{cnt[$3]++} END {for (m in cnt) print m, cnt[m]}'
```
- `grep` 过滤出含 `ERROR` 的行  
- `awk` 用第3列（假设是模块名）作为数组键计数，`END` 块输出统计结果

**再如**：找出 `/etc/passwd` 中 UID 大于 1000 的用户名  
```bash
grep -v '^#' /etc/passwd | awk -F: '$3>1000 {print $1}'
```
- `grep -v` 排除注释行，`awk -F:` 按冒号切分并过滤第3列。