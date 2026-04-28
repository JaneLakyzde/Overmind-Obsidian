---
aliases:
tags:
  - Python
---

`Matplotlib` 是Python最基础、最流行的数据可视化库，用于创建高质量的静态、动态和交互式图表，甚至可以输出出版级别的图形[](https://developer.aliyun.com/article/1543859)。

- **主要功能**
    
    - **高度可定制**：几乎所有图表元素（颜色、字体、坐标轴等）都可精确控制[](https://www.dtstack.com/bbs/article/87648)。
        
    - **图表类型齐全**：支持折线图、散点图、柱状图、饼图、热力图等各类基础及复杂图表[](https://www.dtstack.com/bbs/article/87648)。
        
    - **支持交互与动画**：可在 Jupyter Notebook 等环境中交互操作，并支持创建动态图表[](https://www.dtstack.com/bbs/article/87648)。
        
    - **层级结构清晰**：以 `Figure` (画布) -> `Axes` (子图/坐标系) -> `Axis` (坐标轴) -> `Tick` (刻度) 的容器层级组织图形[](https://developer.aliyun.com/article/1543859)。
        
- **安装与基本示例**
    
```bash    
pip install matplotlib   
``` 
```python    
import matplotlib.pyplot as plt
# 创建数据并绘制带标题和标签的折线图
x = [1, 2, 3, 4, 5]
y = [2, 3, 5, 7, 11]
plt.plot(x, y, label='我的数据')
plt.xlabel('X轴标签')
plt.ylabel('Y轴标签')
plt.title('一个简单的折线图示例')
plt.legend()
plt.show()
```