---
aliases:
gloss: 用于图片隐写术分析的图形化工具
tags:
  - CTF
---


Stegsolve 是一款用于**隐写术分析**的图形化工具，主要用于检测图像中隐藏的信息（如文本、加密数据或另一张图片）。以下是详细使用指南：

---

### 📥 **安装 Stegsolve**
1. **下载工具**（需 Java 环境）：
   - 访问 [GitHub 发布页](https://github.com/zardus/ctf-tools/blob/master/stegsolve/install) 下载 `stegsolve.jar`
   - 或通过命令下载：
     ```bash
     wget http://www.caesum.com/handbook/Stegsolve.jar -O stegsolve.jar
     ```

2. **运行工具**（确保已安装 Java）：
   ```bash
   œ
   ```
   > ✅ 如果系统有 `openjdk`（已在你的 Brew 列表），直接运行即可。

---

### 🔍 **核心功能使用**
#### 1. **打开图像文件**
   - 菜单栏选择 `File > Open` 载入待分析图片（支持 PNG、BMP、JPG 等）。

#### 2. **常用分析模式**
   - **左右箭头切换视图** → 遍历所有分析模式
   - **关键模式说明**：

| **模式**                        | **作用**                |
| ----------------------------- | --------------------- |
| `Red/Green/Blue Plane`        | 分离 RGB 颜色通道，查看单通道隐藏信息 |
| `Alpha Plane`                 | 检查透明度通道（PNG 常用）       |
| `LSB (Least Significant Bit)` | 查看最低有效位隐藏的数据（隐写高频区域）  |
| `Color Inversion`             | 反色显示，突出隐藏内容           |
| `Row/Column Scan`             | 按行/列扫描图像，检测规律性异常      |


#### 3. **提取隐藏数据**
   - 发现异常时，用 `Analyse > Image Combiner` 对比原图与处理后的差异
   - 通过 `File > Save` 导出可疑图像层进一步分析

---

### 🛠️ **实战技巧**
1. **CTF 隐写题常用场景**：
   - **Flag 隐藏在 LSB 层** → 切换到 `LSB` 模式查看噪点/文字
   - **二维码分片在颜色通道** → 用 `Red/Green/Blue` 分离后拼接
   - **文本隐藏在 Alpha 通道** → 检查透明图层

2. **自动分析**：
   - 使用 `Analyse > Stereogram Solver` 自动解立体隐写
   - `Analyse > Frame Browser` 分析 GIF 帧序列

---

### ⚠️ **注意事项**
1. 如果图像显示为纯黑/白：
   - 尝试 `Image > Adjust Brightness` 调整亮度/对比度
   - 切换到 `LSB` 或 `MSB` 模式查看细节

2. **常见错误解决**：
   ```plaintext
   "Unable to access jarfile stegsolve.jar"
   → 检查文件路径：java -jar /完整路径/stegsolve.jar
   ```
   ```plaintext
   "Java not found"
   → 安装 Java：brew install openjdk
   ```

---

### 📚 **学习资源**
- 官方指南：[Stegsolve 使用文档](https://github.com/eugenekolo/sec-tools/tree/master/stego/stegsolve/stegsolve)
- 实战案例：[CTF 隐写术挑战库](https://ctftime.org/writeups?tags=stego)

需要分析具体图像？可上传文件（或描述现象），我会指导操作步骤！ 🔍