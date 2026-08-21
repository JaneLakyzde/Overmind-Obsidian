---
aliases:
tags:
  - bash
  - shell
  - JSON
---
- **全称**：**J**SON **q**uery（JSON 查询）
- **核心思想**：**处理 JSON 数据的命令行瑞士军刀** —— 像 `grep`/`awk` 处理文本行、列一样，`jq` 用来过滤、变形、提取 JSON 中的结构化数据。

## 常用命令速查
| 写法 / 选项 | 作用 | 示例 |
|-------------|------|------|
| `jq '.'` | 原样格式化（美化输出） | `curl -s URL \| jq '.'` |
| `.[]` | 拆开数组，逐个处理元素 | `jq '.[]' data.json` |
| `.field` | 提取字段值 | `jq '.name'` |
| `select(条件)` | 按条件过滤 | `jq '.[] \| select(.version > 6)'` |
| `\|` (jq内部管道) | 连接过滤器，左边输出作为右边输入 | `jq '.[] \| .name'` |
| `-r` | 输出原始字符串（去掉 JSON 引号） | `jq -r '.name'` |
| `length` | 数组或字符串长度 | `jq 'length'` |
| `map()` | 对数组每个元素应用表达式 | `jq 'map(.name)'` |
| `@csv` / `@tsv` | 格式化为 CSV / TSV | `jq -r '.[] \| [.name,.version] \| @csv'` |
| `keys` | 输出对象的所有键 | `jq 'keys'` |

> `jq` 的条件判断支持 `==`、`>`、`<`、`and`、`or`、`not`，字符串可用 `startswith()`、`contains()` 等函数。

## 与 [[curl]]、[[grep]] 等联动
`jq` 常作为 JSON 数据的“前端处理器”，接收来自 `curl` 的 API 响应，筛选出必要字段后再交给 `grep`/`awk` 或直接输出。

**示例：从 API 提取满足条件的人名**
```bash
curl -s https://example.com/data.json | jq -r '.[] | select(.version > 6) | .name'
```
- `curl` 获取 JSON
- `jq` 过滤 `version > 6` 的元素，取出 `name` 字段，以纯文本输出

**再如：统计 JSON 数组中每个状态的计数**
```bash
curl -s URL | jq -r '.[].status' | sort | uniq -c | sort -nr
```
- `jq` 提取所有 `status` 字段，输出为文本流
- 后续 `sort | uniq -c` 等经典管道完成计数，就像处理普通文本一样

这样，`jq` 把 JSON 转成了文本行，无缝接入了你已熟悉的 `grep`/`awk` 生态。