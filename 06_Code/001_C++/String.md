---
aliases:
  - 字符串
tags:
  - cpp
  - Code
date: 2026-03-03
---
# 1 初始化
| **方式**    | **代码示例**              | **说明**                           |
| --------- | --------------------- | -------------------------------- |
| **默认构造**  | `string s;`           | 生成一个空字符串 `""`，长度为 0。             |
| **拷贝构造**  | `string s1(s2);`      | 复制 `s2` 的内容。                     |
| **C 风格**  | `string s("hello");`  | 最常用，直接用字符串字面量初始化。                |
| **截取初始化** | `string s(cp, n);`    | 取 C 风格字符串 `cp` （指针）的前 `n` 个字符。   |
| **截取初始化** | `string s(str,i,n);`  | 取 C++ 字符串 `str` 从索引`i`开始 的`n`个字符 |
| **填充初始化** | `string s(n, 'c');`   | 生成由 `n` 个字符 `'c'` 组成的字符串。        |
| **区间初始化** | `string s(it1, it2);` | 使用迭代器区间初始化。                      |
# 2 基础信息与容量

- **`size()` / `length()`**: 返回字符串长度。两者完全等价。
- **`capacity()`**: 返回字符串占用的物理空间大小。 
- **`empty()`**: 检查字符串是否为空。
- **`clear()`**: 逻辑清空字符串，保留物理空间地址。

# 3 元素访问与修改

- **`operator[]`**: 像数组一样访问（不检查越界）。
- **`push_back()` / `pop_back()`**: 在末尾添加/删除单个字符。
- **`append()` / `+=`**: 追加字符串。通常 `+=` 更直观常用。
- `stoi(str,pos,base)`:

# 4 核心查找操作

- **`s.find(str, pos)`**: 从 `pos` 位置开始查找并返回子串 `str` 第一次出现的位置
- **`rfind(str)`**: 从后往前找。
- **`find_first_of(chars)`**: 查找参数中**任意一个**字符首次出现的位置。
- 注意：如果找不到，会返回一个常量 `string::npos`。
- **定义**：`static const size_t npos = -1;`
# 5 字串与提取

- **`substr(pos, len)`**: 从 `pos` 开始截取长度为 `len` 的子串. 第二个参数是长度不是位置
- **`replace(pos, len, str)`**: 替换指定范围的内容。
- **`erase(pos, len)`**: 删除指定范围的内容。

# 6 比较与转换

- **`.compare()`**: 类似 `strcmp`，相等返回 0 。但通常直接用 =（相等返回 1） 
	当前字符串字典序大于参数字符串时返回一个大于 0 的数，不一定是 1 ; 反之亦然
	
- **`stoi()`, `stol()`, `stod()`**: 将字符串转为整数、长整型、浮点数。
- **`to_string()`**: 将数值转为字符串。

# 7 [[迭代器]]

- `.begin()`指向第一个有效元素的地址，`rbegin()`指向最后一个有效元素的地址
- `.end()`指向最后一个有效元素的下一个地址，`rend()`指向第一个有效元素的前一地址
	对反向迭代器执行 `++` 操作，在物理内存地址上相当于 `--`
	可以利用这个`sub(str.rbegin(), str.rend())`反向创建新字串。
- `.insert()`：指定位置插入内容
- `.erase()`：删除指定位置的字符
- `.replace()`：替换指定范围的字符
- `reverse(s.begin(), s.end())`：翻转字符串中的元素（要求显式传入两个迭代器）
	无返回值，原地修改
- `string sub(str.rbegin(), str.rend()); `调用构造函数创建一个新的反转字符串

# 8 插入操作
| 函数签名 (重载形式)                       | 功能描述                                            | 返回值及注意事项                                                     |
| --------------------------------- | ----------------------------------------------- | ------------------------------------------------------------ |
| `insert(pos,count,ch)`            | 在索引 `pos` 之前插入 `count` 个字符 `ch`                 | 起返回 `*this` 引用。若 `pos > size()` 抛出 `out_of_range`            |
| `insert(pos,str)`                 | 将整个 `str` 插到`pos` 之前                            | 返回 `*this` 引用。`pos > size()` 时抛出异常。                          |
| `insert(pos,str, subpos, sublen)` | 将 `str` 中从 `subpos` 开始的 `sublen` 个字符插到 `pos` 之前 | 返回 `*this` 引用。若 `pos > size()` 或 `subpos > str.size()` 抛出异常。 |
| `insert(pos,cstr)`                | 将 C `cstr` 插到`pos` 之前                           | 返回 `*this` 引用。`pos > size()` 时抛出异常。                          |
| `insert(pos,cstr,n)`              | 将 `cstr` 指向的前 `n` 个字符插入到 `pos` 前                | 返回 `*this` 引用。`pos > size()` 时抛出异常。                          |
| `insert(p, ch)`                   | 在迭代器 `p` 指向的位置前插入一个字符 `ch`                      | 返回指向被插入字符的迭代器。插入后迭代器可能失效。                                    |
| `insert(p,count,ch)`              | 在迭代器 `p` 前插入 `count` 个字符 `ch`                   | 返回指向第一个插入字符的迭代器；若 `count==0` 返回 `p`                          |
| `insert(p,first,last)`            | 将范围`[first, last)`中的字符插入到 `p` 之前                | 返回指向第一个插入字符的迭代器。注意范围不能来自 `*this` 内部                          |

---

**字符判断头文件 `<cctype>`（C++ 风格）或 `<ctype.h>`（C 风格）**
两者之间的内容完全一样。

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

---
 **[[remove_if|erase_remove]] 惯用法**：
 
 ```cpp
  // 1. 使用 remove_if 将所有偶数移到后面，并获取新逻辑末尾的迭代器
  // remove_if 传入三个参数：首迭代器、尾迭代器、一元谓词，需<algorithm>
    auto new_end = remove_if(numbers.begin(), numbers.end(),
                                   [](int n) { return n % 2 == 0; });
    // 2. 调用 erase 真正删除从 new_end 到 numbers.end() 之间的“垃圾”
    numbers.erase(new_end, numbers.end()); 
 ```
- **`remove_if` 的作用**：将满足谓词的元素移动到末尾，并返回新的逻辑结尾。
	
- `remove_if` 内部会对每个元素执行：  
`if (predicate(*it)) { ... }`  参数列表不能省略 → **必须传入当前元素**
	
```cpp
s.erase(
	remove_if(s.begin(), s.end(),
		[](char c) { return!isalpha(static_cast<unsigned char>(c)); }),
	s.end());
```
---
size_t : size type，string 和 vector 标准库中定义的一种表示大小的数据类型，无符号整型

---

# 与 vector 进行对比
### . 专门为文本设计的 API

`vector<char>` 只是一个通用的“字节容器”，而 `string` 拥有一整套处理文本的“工具箱”，这些是 `vector` 所没有的：

- **查找功能**：`string` 有 `find()`, `rfind()`, `find_first_of()` 等，可以轻松寻找子串或特定字符。
    
- **子串提取**：`s.substr(pos, len)` 可以直接切片，这在 `vector` 中需要手动拷贝迭代器。
    
- **比较逻辑**：`string` 重载了各种比较运算符，并且有 `compare()` 方法（正如你代码里用的），支持词典序比较。
    
- **连接操作**：你可以直接用 `s1 + s2` 连接两个字符串，`vector` 则需要调用 `insert()`。
    

### 与 C 语言风格字符串（char*）的兼容性

C++ 运行在一个充满 C 语言遗迹的世界里（比如系统底层 API）。

- `std::string` 提供了一个关键方法：`.c_str()`。
    
- 它保证返回的指针指向一个以 `\0` (null terminator) 结尾的连续空间。
    
- **`vector<char>` 并不保证结尾有 `\0`**，所以你不能直接把 `vector` 传给像 `printf` 或 `fopen` 这样需要字符串参数的函数。
    

---

### SSO（短字符串优化 / Small String Optimization）