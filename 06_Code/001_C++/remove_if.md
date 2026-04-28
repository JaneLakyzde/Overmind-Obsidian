---
aliases:
  - erase_if
  - erase_remove
tags:
  - cpp
  - Code
date: 2026-03-04
---
引入了大量 [[Lambda]] 匿名函数语法
# remove_if
```cpp
#include <algorithm>
#include <string>
#include <cctype>
string s;
auto new_end =remove_if(
	s.begin(),s.end(),
		[](char c){
		return !isdigit(static_cast<unsigned char>(c));
		}
	);
```

- **`remove_if` 的作用**：它不会物理删除元素，而是将满足条件的元素移动到容器末尾，并返回指向新逻辑结尾的迭代器。
	
- “将满足谓词的元素移动到末尾，并返回新的逻辑结尾”。

# erase_remove

由于弃置的字符处于未定义状态，直接 erase 是比较安全的做法，因此常常将两者结合起来使用，用来筛选并删除除连续型元素中不符合要求的元素

```cpp
#include <algorithm>
#include <string>
#include <cctype>
string s;
s.erase(remove_if(s.begin(),s.end(),[](char c){return !isdigit(static_cast<unsigned char>(c));}),s.end())
```

# erase_if

在 C++20 标准中，为了简化上面的繁琐步骤，于是使用 erase_if 简化语法。不同容器的erase_if 定义在不同的头文件中，直接引入即可。

```cpp
#include <string>
#include <cctype>
string s;
erase_if( s,[](char c){
		return !isdigit(static_cast<unsigned char> c);
		}
	)
```

---
如果字符串包含非 ASCII 字符，`static_cast<unsigned char>` 是必要的