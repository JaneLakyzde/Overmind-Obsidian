---
aliases:
tags:
  - Python
---

> 基于官方文档精华，深度剖析 Pixi 的所有核心用法

## 目录

1. [安装与配置](#安装与配置)
2. [核心概念速览](#核心概念速览)
3. [项目初始化与基础配置](#项目初始化与基础配置)
4. [依赖管理：Conda + PyPI 的双生态世界](#依赖管理conda--pypi-的双生态世界)
5. [锁文件：可复现环境的基石](#锁文件可复现环境的基石)
6. [任务运行系统：消灭 Makefile](#任务运行系统消灭-makefile)
7. [Shell 与环境激活](#shell-与环境激活)
8. [全局工具安装：取代 Homebrew/apt](#全局工具安装取代-homebrewapt)
9. [配置项的完整参考](#配置项的完整参考)
10. [多平台工作流与 CI/CD](#多平台工作流与-cicd)
11. [高级特性：构建后端与扩展性](#高级特性构建后端与扩展性)
12. [速查表](#速查表)


## 安装与配置

### 快速安装

Pixi 是用 Rust 编写的单个二进制文件，安装非常简便：

```bash
# Linux / macOS
curl -fsSL https://pixi.sh/install.sh | sh

#macOS
brew install pixi

# Windows (PowerShell，建议以管理员身份运行)
irm -useb https://pixi.sh/install.ps1 | iex

# 或使用 winget
winget install prefix-dev.pixi
```

安装完成后，重启终端或手动加载 shell 配置：`source ~/.bashrc`（bash）或 `source ~/.zshrc`（zsh）。

**验证安装**：`pixi --version`，输出类似 `pixi 0.28.0`。

**自动补全配置**：
```bash
# bash
echo 'eval "$(pixi completion --shell bash)"' >> ~/.bashrc
# zsh
echo 'eval "$(pixi completion --shell zsh)"' >> ~/.zshrc
# fish
pixi completion --shell fish | source
```


### 升级

```bash
pixi self-update
```

如果你通过 brew、conda、paru 等包管理器安装了 Pixi，请用对应的包管理器原生机制升级（例如 `brew upgrade pixi`）。

### 配置文件位置

Pixi 的用户级配置文件位于 `~/.pixi/config.toml`。你可以设置默认 channels 来避免每次初始化项目时手动指定。

---

## 核心概念速览

在深入之前，先建立对 Pixi 核心设计理念的理解：

| 概念 | 说明 | 类比 |
|:---|:---|:---|
| **项目（Project）** | 一个带配置文件的目录，定义依赖和工作流 | Rust Cargo 的项目 |
| **manifest** | `pixi.toml` 或 `pyproject.toml`，声明式配置文件 | `Cargo.toml` / `pyproject.toml` |
| **lockfile** | `pixi.lock`，精确锁定所有依赖的版本和哈希 | `Cargo.lock` / `uv.lock` |
| **环境（Environment）** | 依赖的具体安装实例，存储在项目目录 `.pixi/` 下 | 虚拟环境，但更高级 |
| **任务（Task）** | 在 `pixi.toml` 中定义的命令，通过 `pixi run` 执行 | Makefile targets |

Pixi 的核心承诺：**用一个文件描述一切，一个命令跑通一切，在任何机器上得到完全一致的环境。**

---

## 项目初始化与基础配置

### 创建新项目

```bash
mkdir my-project
cd my-project
pixi init           #会创建一个pixi.toml
pixi init --format pyproject
```

这将生成一个 `pixi.toml` 文件：

```toml
[project]
name = "my-project"
version = "0.1.0"
description = "Add a short description here"
authors = []
channels = ["conda-forge"]
platforms = ["osx-arm64"]  # 根据你的系统自动填写
```

**关于 `platforms` 字段**：它会根据当前操作系统自动填入（如 `linux-64`、`win-64`、`osx-arm64`），确保环境在相同架构上可复现。你可以手动编辑此列表来声明项目支持的多个平台。

### 使用 pyproject.toml（推荐 Python 项目）

对于已有的 Python 项目，你可以直接使用标准的 `pyproject.toml`，直接运行`pixi init`, Pixi 的配置放在 `[tool.pixi]` 段中：

```toml
[project]
name = "my-python-project"
version = "0.1.0"
description = "..."

[tool.pixi]
channels = ["conda-forge"]
platforms = ["linux-64", "osx-arm64", "win-64"]

[tool.pixi.dependencies]
python = "3.11.*"
numpy = ">=1.24"

[tool.pixi.tasks]
start = "python main.py"
```

两种 manifest 格式都是 Pixi 原生的，选择权在你。**`pyproject.toml` 带来的最大好处**：你可以将 Python 打包信息（PEP 621）和 Pixi 环境配置放在同一个文件中，一处定义、处处使用。

### 设置默认 Channels

为避免每次初始化都手动指定 `--channel`，在 `~/.pixi/config.toml` 中设置默认 channels：

```toml
default-channels = ["conda-forge"]
```

---

## 依赖管理：Conda + PyPI 的双生态世界

Pixi 最强大的特性之一，就是**原生支持 Conda 生态和 PyPI 生态的无缝混合**。解析时，Pixi 先解析 Conda 依赖，然后针对 Conda 已安装的内容解析 PyPI 包（内部使用 `uv` 进行 PyPI 解析）。

### 添加 Conda 依赖（默认）

```bash
pixi add python numpy pandas
```

这会将包添加到 `[dependencies]` 表中。

**指定版本**：
```bash
pixi add "python=3.11.*"
pixi add "numpy>=1.24,<2.0"
pixi add 'r-base=4.1.3'  # 用单引号包裹避免 shell 解析问题
```


### 添加 PyPI 依赖

```bash
pixi add --pypi requests fastapi
```

这会添加一个 `[pypi-dependencies]` 表，专门存放 PyPI 包。

### 平台特定依赖

```bash
# 仅 Linux-64 平台安装 gcc
pixi add --platform linux-64 gcc
```


### 团队协作最佳实践

如果团队混合使用 Conda 和 PyPI 包，建议：
1. 在 `pixi.toml` 顶部设置 `default-channel = ["conda-forge"]`
2. 将所有 PyPI 包统一列在 `[pypi-dependencies]` 下
3. 避免在命令行中反复混合使用 `--conda` / `--pypi`

这种做法能最大程度避免依赖冲突，也让 lockfile 的生成更加稳定可预测。

---

## 锁文件：可复现环境的基石

### 核心理念

`pixi.lock` 是 Pixi 实现 **Bit‑for‑bit reproducibility（逐比特可复现）** 的核心武器。它精确锁定每个依赖包的版本和校验和（Checksum），确保你在任何机器、任何时间都能重建**完全一致**的环境。

相比 Conda（即使指定了所有版本号仍可能产生不同环境），Pixi 的 lockfile 确保安装结果真正可预测。

### 锁文件的生成与更新

默认情况下，`pixi add` 会**自动更新 lockfile 并立即安装**。你还可以分步操作：

```bash
# 只更新 manifest，不安装
pixi add --no-install package1
pixi add --no-install package2

# 最后一次性安装所有依赖
pixi install
```


### 手动更新

当直接编辑 `pixi.toml` 后，运行 `pixi install` 重新生成锁文件并安装依赖。`pixi update` 则用于将已锁定的依赖升级到最新兼容版本。

---

## 任务运行系统：消灭 Makefile

Pixi 内置的任务运行器（Task Runner）是其与 Conda / Mamba 最显著的差异之一——它让你可以**声明式地定义项目的所有工作流命令**，完全替代 Makefile。

### 基础用法：添加任务

```bash
# 最简单的形式——直接命令字符串
pixi task add hello "echo 'Hello, Pixi!'"

# 运行任务
pixi run hello
```


在 `pixi.toml` 中，任务定义在 `[tasks]` 表下：
```toml
[tasks]
hello = "echo 'Hello, Pixi!'"
start = "python main.py"
```

### 高级任务：依赖链与工作目录

```bash
# 添加带依赖关系的任务
pixi task add configure "cmake -G Ninja -S . -B .build"
pixi task add build "ninja -C .build" --depends-on configure
pixi task add start ".build/bin/sdl_example" --depends-on build
```


在 `pixi.toml` 中体现为：
```toml
[tasks]
configure = "cmake -G Ninja -S . -B .build"
build = { cmd = "ninja -C .build", depends_on = ["configure"] }
start = { cmd = ".build/bin/sdl_example", depends_on = ["build"] }
```

### 任务别名：组合多个任务

```bash
pixi task add fmt "ruff"
pixi task add lint "pylint"
pixi task alias style fmt lint
```

这生成：
```toml
[tasks]
fmt = "ruff"
lint = "pylint"
style = { depends_on = ["fmt", "lint"] }
```

运行 `pixi run style` 会按序执行 `fmt` 和 `lint`，任一失败则停止。

### 工作目录（cwd）设置

当任务需要在项目子目录中运行时：
```bash
pixi task add bar "python bar.py" --cwd scripts
```
```toml
[tasks]
bar = { cmd = "python bar.py", cwd = "scripts" }
```


### 跨平台差异处理

Pixi 的任务 Shell 基于 `deno_task_shell`，提供了一套有限但统一的跨平台命令集合（如 `cp`、`mv`、`mkdir`、`rm`），确保同一任务申明在 Windows、macOS 和 Linux 上行为一致。

当需要不同平台的专用命令时，使用 `[target]` 表：

```toml
[target.win-64.tasks]
build = "build.bat"

[target.linux-64.tasks]
build = "make"
```


### 环境变量访问

在任务中可以使用 Pixi 提供的环境变量：
- `$PIXI_PROJECT_ROOT`：项目的根目录
- `$CONDA_PREFIX`：当前环境的安装路径

```toml
[tasks]
run = "python $PIXI_PROJECT_ROOT/main.py"
```


---

## Shell 与环境激活

Pixi 不同于 Conda 的集中式环境管理模式——**每个项目有自己的环境，存储在项目目录下的 `.pixi/` 中**。这种设计的优点：环境与项目代码紧密耦合，便于版本控制和团队协作。

### 激活环境

```bash
cd my-project
pixi shell
```

这会在当前终端中启动一个新的子 shell，所有依赖都在其中可用。退出该 shell 只需输入 `exit`。

### 不激活直接执行命令

更推荐的方式——无需显式激活环境，直接运行命令：

```bash
# 运行 Python 脚本
pixi run python my_script.py

# 运行已定义的任务
pixi run test

# 运行任意命令
pixi run pytest tests/
```

**为什么推荐 `pixi run` 而非 `pixi shell`**：
- 不需要记住先激活再执行
- 避免忘记退出导致的污染
- 脚本化和 CI/CD 中更易用

### 分离环境配置（高级）

默认情况下，环境存储在项目目录下。当存储空间受限或需要集中管理时，可以通过 `detached-environments` 将环境存储到其他位置：
```toml
[project]
detached-environments = true
```
或通过命令行参数指定路径。


## 全局工具安装：取代 Homebrew/apt

Pixi 不仅能管理项目级依赖，还能作为**系统级包管理器**使用。`pixi global` 子命令让你安装可全局访问的 CLI 工具和桌面应用，不需要手动激活任何环境。

### 安装全局 CLI 工具

```bash
pixi global install --channel conda-forge --channel bioconda trackplot
pixi global install -c conda-forge -c bioconda trackplot  # 简写形式
```


### 桌面应用程序（Shortcuts 功能）

从 conda-forge 安装 Zed IDE 后，它会自动出现在应用启动器中：
```bash
pixi global install zed
# macOS: 安装到 ~/Applications/Zed.app，可从 Launchpad 启动
# Linux: 安装到 ~/.local/share/applications，可从应用菜单启动
```

### 更新全局工具

```bash
pixi global update zed
```


### 使用场景建议

| 场景 | 推荐方式 | 理由 |
|:---|:---|:---|
| 项目开发依赖（Python、NumPy、PyTorch） | `pixi add`（项目级） | 与项目版本绑定，lockfile 管理 |
| 系统级 CLI 工具（ripgrep、fd、jq） | `pixi global install` | 全局可用，版本固定 |
| 桌面应用（IDE、浏览器） | `pixi global install` | 自动创建桌面快捷方式和文件关联 |

---

## 配置项的完整参考

### project 表

| 字段                                          | 必需？ | 说明                                              |
| :------------------------------------------ | :-- | :---------------------------------------------- |
| `name`                                      | ✅   | 项目名称                                            |
| `channels`                                  | ✅   | 包来源列表，如 `["conda-forge"]`                       |
| `platforms`                                 | ✅   | 支持的平台列表，如 `["linux-64", "osx-arm64", "win-64"]` |
| `version`                                   | ❌   | 项目版本（Conda version spec）                        |
| `authors`                                   | ❌   | 作者列表                                            |
| `description`                               | ❌   | 简短描述                                            |
| `license`                                   | ❌   | SPDX 格式的许可证字符串                                  |
| `license-file`                              | ❌   | 许可证文件路径                                         |
| `readme`                                    | ❌   | README 文件路径                                     |
| `homepage` / `repository` / `documentation` | ❌   | URL                                             |



### dependencies 表

```toml
[dependencies]
python = "3.11.*"
numpy = ">=1.24"
pytorch = "*"
```

### pypi-dependencies 表

```toml
[pypi-dependencies]
requests = "*"
fastapi = ">=0.100"
```

### 表优先级与解析顺序

Pixi 的依赖解析遵循严格顺序：先解析 `[dependencies]` 中的所有 Conda 依赖，然后针对已解析的 Conda 环境解析 `[pypi-dependencies]`。这意味着：
- Conda 包可以满足 PyPI 包的某些依赖（例如 numpy 既可通过 Conda 也可通过 PyPI 安装）
- 如果同一个包同时出现在两个表中，Pixi 会优先使用 Conda 版本

---

## 多平台工作流与 CI/CD

### 配置多平台支持

在 `[project]` 中声明支持的平台后，Pixi 会在 lockfile 中**为每个平台分别求解并锁定依赖**。这意味着：
- 你可以在 macOS 上开发，lockfile 中已提前包含 Linux 和 Windows 的依赖信息
- CI 中只需 `pixi install`，无需重新求解依赖，大幅提速

```toml
[project]
platforms = ["linux-64", "osx-arm64", "win-64"]
```


### GitHub Actions 集成

Pixi 官方提供了 GitHub Action：
```yaml
- name: Install dependencies with pixi
  uses: prefix-dev/setup-pixi@v0.8.0
  with:
    environments: default

- name: Run tests
  run: pixi run test
```
默认情况下，如果存在 `pixi.lock`，Action 会启用缓存，根据 lockfile 生成的哈希值来缓存整个环境。

### 多环境支持（dev / prod / test）

你可以在一个项目中定义多个环境：
```yaml
[environments]
dev = { features = ["dev"] }
prod = { features = ["prod"] }
test = { features = ["test"] }
```

然后通过环境变量指定要安装的环境：
```yaml
- uses: prefix-dev/setup-pixi@v0.8.0
  with:
    environments: dev
```


在 CLI 中：`pixi install --environment dev`

---

## 高级特性：构建后端与扩展性

### Pixi-build：从开发到发布的一体化

Pixi 超越了单纯的包管理，**通过 Build Backend 体系直接支持将项目构建为可发布的 Conda 包**。构建后端的核心设计是**通过 JSON-RPC 协议与 Pixi 通信**，这使得任何语言生态都可以定义自己的构建规则，而无需修改 Pixi 本身。

### 配置 CMake 项目构建

```toml
[package]
name = "my-package"
version = "0.1.0"
description = "My cool package"

[package.build.backend]
name = "pixi-build-cmake"
version = "*"

[package.build.config]
extra-args = ["-DCMAKE_BUILD_TYPE=Release"]

[package.build-dependencies]
# 编译时依赖
[package.host-dependencies]
# 仅头文件依赖，如 fmt
[package.run-dependencies]
# 运行时依赖，如 SDL2
```


运行 `pixi build` 即可将项目构建为可分发的 Conda 包。

### 从源码安装（Monorepo 场景）

```bash
pixi install --path ./local-package
```


这让你可以在 monorepo 内跨项目引用尚未发布的本地包。

### 构建后端的生态扩展性

构建后端的设计使 Pixi 具备了无限扩展性。目前已有的构建后端包括：
- `pixi-build-cmake`：用于 CMake C++ 项目
- `pixi-build-python`：用于 Python 包
- `rattler-build`：通用 Conda 包构建

任何语言的开发者都可以编写自己的构建后端以接入 Pixi 生态。

---

## 速查表

### 项目生命周期管理

| 命令 | 说明 |
|:---|:---|
| `pixi init` | 在当前目录初始化新项目 |
| `pixi add <pkg>` | 添加 Conda 依赖 |
| `pixi add --pypi <pkg>` | 添加 PyPI 依赖 |
| `pixi install` | 根据 lockfile 安装所有依赖 |
| `pixi update` | 更新锁定的依赖到最新兼容版本 |
| `pixi remove <pkg>` | 移除依赖 |

### 环境与执行

| 命令 | 说明 |
|:---|:---|
| `pixi shell` | 激活环境的交互式 shell |
| `pixi run <cmd>` | 在环境中执行命令 |
| `pixi run <task>` | 执行 `pixi.toml` 中定义的任务 |
| `pixi info` | 查看 Pixi 和环境信息 |
| `pixi list` | 列出环境中已安装的包 |

### 任务管理

| 命令 | 说明 |
|:---|:---|
| `pixi task add <name> <cmd>` | 添加简单任务 |
| `pixi task add <name> <cmd> --depends-on <tasks>` | 添加带依赖的任务 |
| `pixi task add <name> <cmd> --cwd <dir>` | 设置任务的工作目录 |
| `pixi task alias <name> <task1> <task2>` | 创建任务别名（顺序执行） |

### 全局包管理

| 命令 | 说明 |
|:---|:---|
| `pixi global install <pkg>` | 全局安装工具（CLI 或桌面应用） |
| `pixi global update <pkg>` | 更新全局工具 |
| `pixi global list` | 列出已安装的全局包 |
| `pixi global remove <pkg>` | 卸载全局包 |

### 调试与诊断

| 命令 | 说明 |
|:---|:---|
| `pixi --version` | 显示版本 |
| `pixi self-update` | 更新 Pixi 自身 |
| `pixi config` | 查看/修改 Pixi 配置 |
| `pixi search <pkg>` | 在 Conda 仓库中搜索包 |

---

这份手册覆盖了 Pixi 从入门到精通的核心内容。实际使用中，建议始终从 **`pixi init` → 配置依赖 → `pixi.lock` 版本控制 → `pixi run` 执行任务** 这个完整的工作流出发。Pixi 的优秀之处在于每一次 `pixi install` 都给你完全确定的依赖图，每一次 `pixi run` 都在完全相同的中执行指令——这正是现代化软件开发中对环境管理工具的核心要求。