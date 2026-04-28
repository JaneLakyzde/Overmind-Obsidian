---
aliases:
  - ctype.h
tags:
  - cpp
  - c
  - Code
date: 2026-03-04
---
cpp 风格和 c 风格的头文件之间的内容基本没有区别。

| 函数        | 判断条件             |
| --------- | ---------------- |
| `isalpha` | 字母（a-z, A-Z）     |
| `isalnum` | 字母或数字            |
| `islower` | 小写字母             |
| `isupper` | 大写字母             |
| `isspace` | 空白字符（空格、换行、制表符等） |
| `ispunct` | 标点符号             |
| `iscntrl` | 控制字符             |
| `isprint` | 可打印字符（包括空格）      |
| `isgraph` | 可打印字符（不包括空格）     |
| `isdigit` | 数字（1-0）          |
满足返回 1 ，否则返回 0 