---
aliases:
tags:
  - Python
date: 2026-04-29
---
## 🧭 第一部分：核心概念 —— 认识 marimo

### 它与 Jupyter 根本区别：响应式执行模型
marimo 最核心的哲学是：**笔记本不是代码块的无序集合，而是一个可复现、可交互、可分享的 Python 程序**。

*   **Jupyter (REPL 模式)**：你手动、按任意顺序执行代码块。Jupyter 不关心代码块之间的关系，这极易导致“隐藏状态”（hidden state）。一项针对 GitHub 上公开笔记本的研究发现，**36% 的 Jupyter 笔记本无法被复现**。
*   **marimo (响应式/反应式)**：当你运行一个单元格，或与一个UI元素（如滑块）交互时，marimo 会自动识别并运行所有**依赖于该变量**的其他单元格。这保证了你的代码、输出和程序状态始终保持一致，从根源上杜绝了“隐藏状态”。

这个执行的底层逻辑，是将你的笔记本解析成一个**有向无环图 (DAG)**。

### 文件存储：`.py` 即一切
marimo 笔记本**存储为纯 `.py` 文件**。这带来了革命性的好处：
*   **Git 友好**：可以像管理普通代码一样进行版本控制、代码审查和差异对比，告别了 `.ipynb` 的 JSON 混乱。
*   **既是笔记本也是脚本**：同一个文件，你可以像普通Python脚本一样用 `python notebook.py` 执行，也可以作为Web应用部署。

### 全局变量只能定义一次
marimo “编译”笔记本时，会分析代码中的变量定义和引用关系来构建 DAG。因此，**同一个全局变量不能在多个单元格中定义**，否则 marimo 将无法确定正确的执行顺序。

> **适应技巧**：多使用函数封装局部变量；为临时变量名加下划线前缀 `_`，marimo会自动忽略它。

## 🛠️ 第二部分：安装与启动

marimo 为新手提供了无需本地安装的在线环境 **[molab](https://molab.marimo.io/)**。本地安装可通过 `pip`、`conda` 或 `pixi` 完成。

```bash
# 1. 安装
pip install marimo
# 或使用现代工具
uv tool install marimo

# 2. 启动内置互动教程（推荐新手）
marimo tutorial intro
```

**核心开发命令**:

| 命令                        | 功能                        | 典型场景         |
| :------------------------ | :------------------------ | :----------- |
| `marimo edit notebook.py` | 启动交互式编辑器，创建或编辑 `.py` 笔记本  | 日常开发、探索性分析   |
| `marimo run notebook.py`  | 将笔记本部署为**只读Web应用**，默认隐藏代码 | 分享报告、制作仪表盘   |
| `python notebook.py`      | 作为**普通Python脚本**直接运行      | 集成到自动化流程、批处理 |


## 💻 第三部分：编辑器与开发体验

通过 `marimo edit` 启动的编辑器是一个功能强大的 IDE。

### 变量浏览器
编辑器侧边栏的**变量面板**可以让你实时查看所有全局变量的状态、类型和数据预览，这比在 Jupyter 中手动检查变量直观得多。

### 数据流可视化
为了帮助你理解 DAG 结构，marimo 提供了强大的**数据流工具**，包括依赖关系图、迷你地图和响应式引用高亮。这对调试复杂的依赖链非常有帮助。

### 内置的拓展插件管理
打开编辑器右上角的设置菜单，可以在“软件包”选项卡中为笔记本安装Python包。marimo 会在 `.py` 文件头部的 **PEP 723** 注释中管理这些依赖，实现了环境级的可复现。

### AI 驱动开发
marimo 编辑器深度集成了 AI 功能：
*   **AI 代码补全**：根据上下文自动补全。
*   **AI 聊天**：内置聊天面板，可直接与 LLM 对话。在对话中用 `@` 符号可以引用报错信息，让 AI 帮你调试。
*   **AI 代码重构**：一键重构选中的代码。

> **配置 LLM**：需要在 `marimo.toml` 文件中配置。支持 OpenAI、Anthropic、Ollama 等主流提供商。


## ⚛️ 第四部分：深度理解响应式 (Reactivity) —— 核心机制

### 执行规则
当一个单元格被运行时，marimo 会找出所有**读取了该单元格定义的全局变量**的单元格，并自动运行它们。这个规则也同样适用于 UI 元素的交互。

### 对可变对象的精妙处理
marimo 通过静态分析代码来建立依赖，**不会追踪运行时的对象变异**。如果你在一个单元格中定义了一个 `DataFrame`，在另一个单元格中给它增加一列，marimo 是无法感知到这个变化的。因此，官方强烈建议**在定义变量的同一个单元格中完成对它的所有变异**。

### 惰性执行与 `mo.stop`
对于包含耗时运算（如训练模型）的笔记本，marimo 提供了灵活的控制手段：
*   **惰性运行模式**：可在设置中将运行时配置为“惰性”（lazy）。此时，当你修改代码或交互，受影响的单元格**不会自动运行，而是被标记为“过时” (stale)**。你可以稍后通过一个“运行所有过时单元格”的按钮来手动刷新。
*   **`mo.stop()` 函数**：它可以在运行时根据条件停止单元格的执行，常用于保护性编程或需要大量计算的场景。

### 高级状态管理 `mo.state`
当你需要引入“历史状态”（例如记录用户输入过的所有值）或同步不同的 UI 元素时，可以使用 `mo.state`。它提供一个 getter 和 setter 函数，更新状态会触发依赖单元格的重跑。**但在 99% 的情况下，直接使用 UI 元素的值就足够了**。

**案例：使用 `mo.state` 实现计数器累加**
```python
import marimo as mo

# 创建一个初始值为0的响应式状态，setter的更新会触发getter所在单元格重跑
get_counter, set_counter = mo.state(0)

# 定义按钮并设置点击行为
button = mo.ui.button(label="点击 +1", on_change=lambda _: set_counter(lambda v: v + 1))
```
```python
mo.hstack([button, mo.md(f"当前计数: {get_counter()}")])
```


## 第五部分：构建交互式 UI（无回调地狱）

marimo 提供了一套原生、无需编写回调函数的 UI 组件库。

### 基础使用范式
1.  通过 `mo.ui` 创建 UI 元素（如 `slider`， `button`）。
2.  将其赋值给一个**全局变量**。
3.  在其他单元格中**直接访问该变量的 `.value` 属性**。
4.  当用户与 UI 元素交互时，marimo 会自动重跑所有引用了这个变量的单元格。

**案例：创建一个动态滑块并实时绘图**
```python
import marimo as mo
import matplotlib.pyplot as plt
import numpy as np

# 创建一个范围在(0.1, 5.0)，步长为0.1的滑块
amplitude = mo.ui.slider(start=0.1, stop=5.0, value=1.0, step=0.1, label="振幅")
amplitude
```
```python
# 访问滑块的 .value 属性，此单元格会自动重新执行
x = np.linspace(0, 4 * np.pi, 400)
y = amplitude.value * np.sin(x)

plt.figure(figsize=(6, 3))
plt.plot(x, y)
plt.title(f"y = {amplitude.value} * sin(x)")
plt.grid(True)
plt.gca()
```

### 高级布局：`batch` 与 `form`
*   **`mo.ui.array` 和 `mo.ui.dictionary`**：用于将一组在运行时才知道的 UI 元素逻辑地组合在一起。
*   **`mo.ui.batch` 和 `mo.ui.form`**：这两个组件可以将多个控件打包，它们的 `.value` 会返回包含所有子元素值的字典，特别适合创建复杂的配置面板。

## 📊 第六部分：原生数据处理能力

### 1. SQL 单元格
marimo 使得在 Python 笔记本中编写 SQL 变得像呼吸一样自然。

**安装额外依赖**：`pip install duckdb`

**案例：混合使用 Python 和 SQL**
```python
import marimo as mo
import polars as pl

# 创建一个 DataFrame
df = pl.DataFrame({"name": ["产品A", "产品B"], "sales": [100, 200]})
```
```sql
-- 这是一个SQL单元格，可以直接查询上面定义的 df 变量
SELECT *, sales * 2 AS doubled_sales FROM df
```
查询结果 `output_df` 会作为一个 **Polaras DataFrame**（若已安装）或 **Pandas DataFrame** **自动生成**，并可以在后续的 Python 单元格中直接使用。SQL 单元格本身支持 f-string，你可以将 Python 变量、UI 控件的值动态插入到 SQL 查询中，实现交互式查询。

### 2. 交互式数据框
`mo.ui.dataframe` 能将一个 DataFrame 变成一个可交互的表格，**支持在前端直接进行排序、筛选、分组等操作**。转换后的数据可以通过 `.value` 在 Python 中获取。

**案例：过滤大数据集**
```python
import polars as pl

# 假设我们有一个很大的航班数据集
flights = pl.read_parquet("flights.parquet")

# 开启 lazy 模式，表格只会加载前10行，点击 Apply 才会执行全部转换
t = mo.ui.dataframe(flights, page_size=10, lazy=True)
t
```
```python
# 获取经过用户筛选后的数据
filtered_data = t.value
```

### 3. 内置图表构建器
编辑器提供了一个**无代码的图表构建器**，你可以通过点击和拖拽来为你的 DataFrame 创建可视化图表。所有操作都会被自动转换成 Python 代码，你可以直接复制到自己的单元格中。


## 🚀 第七部分：部署、分享与多格式导出

### 1. 分享为 Web 应用
这是 marimo 的另一大优势。使用 `marimo run notebook.py` 可以一键将你的笔记本部署成一个独立的、交互式的 web 应用，代码默认隐藏。

*   **布局自定义**：你可以在编辑器中使用**拖拽式网格编辑器**来自由排列你的输出，创造出专业的应用布局。所有布局信息会保存在一个 `layouts/` 文件夹中。
*   **幻灯片布局**：在应用预览界面，你还可以选择“幻灯片”模式，将你的笔记本变成一场演示。

### 2. 多种分享方式
*   **molab (最简单)**：免费云平台。你可直接在 molab 上创建或从 GitHub 导入笔记本，然后通过链接分享。查看者可以预览静态版本，也可以一键 fork 笔记本进行交互。
*   **导出为 WASM HTML**：以 `marimo export html-wasm notebook.py` 生成一个独立的 HTML 文件。这个文件可以在浏览器中运行（利用 WebAssembly），完全无需后端服务器。你可以把它托管在 GitHub Pages 或任何静态文件服务器上。
*   **导出为其他格式**：`marimo export ipynb` 命令可以将 notebook 转换成 Jupyter 格式，方便你融入现存的 Jupyter 生态。

### 3. 从 Jupyter 无缝迁移
marimo 提供了平滑的过渡路径：
*   使用 `marimo convert your_notebook.ipynb` 可以**自动将 Jupyter 笔记本转为 marimo 格式**。
*   如果你习惯了 Jupyter 的 REPL 模式但想享受 marimo 的好处，可以在 `marimo.toml` 配置文件中开启**惰性运行时**来模拟手动执行体验，同时保留状态一致性检查。


## ⚙️ 第八部分：高级工作流与配置

### 1. 运行时配置
如果你不适应全自动的响应式，可以在 `marimo.toml` 配置文件里设置：
```toml
[runtime]
auto_instantiate = false  # 启动时不自动运行笔记本
on_cell_change = "lazy"  # 修改单元格或UI时，只标记stale，不自动运行
```
你可以通过 `marimo config show` 找到该文件路径，若没有则自行创建。

### 2. 作为脚本运行与参数化
由于存储为 `.py` 文件，你可以像普通脚本一样调用它并传递参数：
```bash
python notebook.py --learning_rate 0.01 --epochs 10
```
在笔记本内部，你可以通过 `mo.cli_args()` 来获取这些命令行参数。

### 3. 测试笔记本
你可以直接用 `pytest` 来测试你的 marimo 笔记本。
```bash
pytest your_notebook.py
```
同时，它也支持 `pdb` 调试器，你可以直接在单元格里设置 `breakpoint()`，或在 VS Code 中用 `launch.json` 配置来调试。

### 4. 自定义 UI 插件 (AnyWidget)
利用 `anywidget` 库，你可以用 HTML/JavaScript 创建完全自定义的响应式 UI 组件，并完美集成到 marimo 的响应式执行引擎中。

## 第十部分：最佳实践与性能优化

### 1. 控制全局变量
多使用函数封装，减少全局变量数量。对于临时变量，使用 `_temp` 这样的命名方式（以 `_` 开头），marimo 会自动忽略它们，不会建立依赖关系。

### 2. 管理耗时单元格
对耗时的计算，使用 `mo.ui.run_button()` 配合 `mo.stop()` 来手动触发执行，避免不必要的计算开销。

### 3. 变量变异须在同单元格完成
如前所述，所有对对象的修改（如添加列、修改属性）务必在定义该对象的单元格中完成，否则依赖追踪会失效。

### 4. SQL 与大数据集处理
处理大型数据集时，推荐在`marimo.toml` 中将 SQL 输出类型设为 `native` 或 `lazy-polars`，以利用惰性求值，避免一次性加载所有数据。

### 5. 版本控制
记得把 `layouts/` 文件夹也加入 Git 管理，这样部署为 App 时的拖拽布局才能被复现。


希望这份详尽的速查手册能帮你系统性地掌握 marimo！如果你想深入了解其中任何一个模块（比如如何在云端集群部署、如何结合 SkyPilot 做分布式训练等），随时可以继续问我。