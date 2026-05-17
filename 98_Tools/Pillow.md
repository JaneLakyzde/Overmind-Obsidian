---
aliases:
  - PIL
tags:
  - Python
---
### Pillow (PIL)：图像处理的事实标准

`Pillow` 是 Python 图像处理的事实标准库，基于已停止维护的 PIL 分支发展而来，功能强大且API设计简洁

- **主要功能**
    
    - **格式支持广泛**：支持超过30种图像格式（如 JPEG, PNG, GIF），涵盖日常开发需求[](https://developer.aliyun.com/article/1712022)。
        
    - **操作丰富**：内置 `Image` 对象，轻松实现裁剪、缩放、旋转、滤镜、色彩调整等操作[](https://cloud.tencent.com.cn/developer/article/2448221?from=15425&frompage=seopage)。
        
    - **像素级操作**：支持对图像像素进行精细的读取和修改[](https://developer.aliyun.com/article/1712022)。
        
- **核心概念**  
    `Image` 类是核心，用于表示图像对象。通过它可以获取图像的模式（如RGB）、尺寸、格式等元数据[](https://developer.aliyun.com/article/1712022)。
    
- **安装与基本示例**
    
```bash
pip install Pillow
```

```python
from PIL import Image
#打开图片，缩放到指定尺寸，并保存
img = Image.open('input.jpg')
resized_img = img.resize((800, 600))
resized_img.save('output.jpg')
```