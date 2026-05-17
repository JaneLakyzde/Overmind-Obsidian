---
aliases:
  - Numerical Python
tags:
  - Python
---
`NumPy`（Numerical Python）是几乎所有Python科学和数据分析库的基础，为数组和矩阵运算提供了高性能的核心支持。

- **主要功能**
    
    - **高效的 N 维数组对象 `ndarray`**：支持多维、同质的数据容器，数据操作性能远超原生 Python 列表[](https://numpy.org/doc/stable/user/absolute_beginners.html)。
        
    - **广播功能 (Broadcasting)**：对不同形状的数组进行算术运算，无需低效复制数据。
        
    - **丰富数学函数**：提供线性代数、傅里叶变换、随机数生成等工具[](https://numpy.net.cn/)。
        
    - **与底层语言集成**：核心计算基于C/Fortran，保证高性能，同时提供 Python 的灵活性[](https://numpy.net.cn/)。
        
- **核心概念**
    
    - `ndarray` 是其核心，代表N维数组，要求所有元素类型相同，创建后尺寸固定且形状“规整”[](https://numpy.org/doc/stable/user/absolute_beginners.html)。
        
- **安装与基本示例**
    
```bash
pip install numpy
```

```python
import numpy as np
# 创建一个2x3的数组并计算其最大值
arr = np.array([[1, 2, 3], [4, 5, 6]])
print(arr.max()) # 输出: 6
```