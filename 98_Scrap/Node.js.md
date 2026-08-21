---
aliases:
  - npm
  - pnpm
  - nvm
  - yarn
  - npx
tags:
  - javascript
date: 2026-05-25
---
## 1. 基础概念回顾

### Node.js
- **定义**：基于 Chrome V8 引擎的 JavaScript 运行时环境。
- **作用**：让 JavaScript 脱离浏览器，在服务器端、命令行或桌面应用中执行。
- **特点**：事件驱动、非阻塞 I/O，适合构建高并发的网络应用。

Node.js 能够实现高并发的本质，是它放弃了“每个连接一个线程”的同步阻塞模型，转而采用 **单线程 + 异步非阻塞 I/O + 事件驱动** 的架构。这种架构将 I/O 等待时间从主线程剥离，让一个线程可以高效地管理成千上万个并发连接，在有限系统资源下实现极高的吞吐量和响应速度，尤其适合构建 I/O 密集型网络应用。

### NVM (Node Version Manager)
- **定义**：Node.js 版本管理工具。
- **作用**：在同一台机器上安装、切换、卸载多个 Node.js 版本，方便项目对不同版本的需求。
- **常用命令**：
  ```bash
  nvm install 20.11.0    # 安装指定版本
  nvm use 18.19.0        # 切换版本
  nvm alias default 20   # 设置默认版本
  nvm ls                 # 列出已安装版本
  ```

### NPM (Node Package Manager)
- **定义**：Node.js 自带的包管理器。
- **作用**：
  - 管理项目依赖（`package.json`、`node_modules`）
  - 发布和分享自己的包
  - 运行自定义脚本（`npm run`）
- **关键文件**：
  - `package.json`：项目清单，声明依赖及脚本。
  - `package-lock.json`：锁定依赖的精确版本，保证环境一致性。

### NPX (Node Package Execute)
- **定义**：npm 5.2+ 附带的命令行工具。
- **作用**：
  - 直接执行 npm 包中的可执行文件，无需全局安装。
  - 可运行一次性命令，例如项目初始化工具（`create-react-app`）、代码格式化工具等。
  - 可临时安装并运行某个特定版本的包。
- **示例**：
  ```bash
  npx create-react-app my-app   # 无需全局安装 create-react-app
  npx eslint --init             # 直接运行本地或远程的 eslint
  ```

---

## 2. 与 TypeScript 的兼容性

TypeScript 是 JavaScript 的超集，最终需要编译为 JavaScript 才能在 Node.js 中运行。Node.js 本身不能直接执行 `.ts` 文件（实验性功能除外），因此有几种主流方案实现 Node.js 与 TypeScript 的兼容。

### 2.1 编译后执行（传统方式）
先用 TypeScript 编译器 `tsc` 将 `.ts` 编译为 `.js`，再用 `node` 运行。
```bash
npm install -D typescript
npx tsc --init                # 生成 tsconfig.json
npx tsc                      # 编译
node dist/index.js           # 运行编译结果
```
**优点**：稳定、标准。  
**缺点**：开发时需要反复编译，反馈慢；需维护输出目录。

### 2.2 即时编译运行：ts-node
`ts-node` 提供一个可以直接运行 `.ts` 文件的环境，内部即时编译并执行。
```bash
npm install -D ts-node typescript
npx ts-node src/index.ts
```
- 可以配合 `nodemon` 实现热重载：
  ```bash
  npx nodemon --exec ts-node src/index.ts
  ```
- **性能问题**：默认使用 TypeScript 编译器，启动和运行较慢，生产环境不建议直接使用。

### 2.3 极速运行时：tsx
`tsx` 是基于 `esbuild` 的高性能 TypeScript 执行器，速度远超 `ts-node`。
```bash
npm install -D tsx
npx tsx src/index.ts
```
- 同样可与 `nodemon` 配合。
- 支持运行 `.ts`、`.tsx`、ESM 及 CommonJS 模块。
- **特点**：即用即执行，无需生成声明文件，适合开发与快速测试。

### 2.4 原生实验性支持
Node.js 从 v20 开始引入实验性 TypeScript 支持，可通过特定标志让 Node.js 直接处理类型注释（仅移除类型，不做类型检查）。
```bash
node --experimental-strip-types src/index.ts
```
或在 `package.json` 中配置 `"type": "module"` 后运行。
- **注意**：目前仍在实验阶段，功能有限（不支持枚举、命名空间等复杂语法），适合尝鲜和简单脚本。
- 也可以通过 `--loader ts-node/esm` 等方式提供 ESM 支持。

### 2.5 类型检查与运行的分离
现代最佳实践：用 `tsx` 或 `esbuild` 快速运行，用 `tsc --noEmit` 单独进行类型检查。

```json
// package.json 脚本示例
{
  "scripts": {
    "dev": "tsx watch src/index.ts",
    "type-check": "tsc --noEmit",
    "build": "tsc"
  }
}
```

---

## 3. 包管理器生态：Yarn 与 pnpm

虽然 npm 已非常完善，但 Yarn 和 pnpm 在性能、磁盘空间管理和 monorepo 支持等方面引入了创新。

### 3.1 Yarn：快速、可靠、安全的依赖管理

Yarn 由 Facebook 在 2016 年发布，旨在解决当时 npm 在速度、一致性和安全性上的痛点。

#### 核心特性
- **确定性安装**：`yarn.lock` 锁定精确版本和依赖树结构，保证团队环境一致。
- **并行下载**：极大加快安装速度。
- **离线缓存**：已下载的包缓存在本地，再次安装无需联网。
- **工作区（Workspaces）**：原生 monorepo 支持，可在一个仓库管理多个包。
- **插件系统（Yarn 2+）**：可扩展核心功能，如支持不同版本的依赖安装策略。

#### 版本分支
- **Yarn Classic（v1）**：经典的类 npm 架构，`node_modules` 模式。
- **Yarn Berry（v2+）**：重构版本，默认启用 **Plug’n’Play (PnP)**，抛弃 `node_modules`，使用 `.pnp.cjs` 文件直接映射依赖到缓存。
  - PnP 优势：零 I/O 安装、更严格的依赖访问、启动更快。
  - 缺点：与某些未按标准声明依赖的工具可能存在兼容性问题。
  - 可通过配置切换回 `node_modules` 模式。

#### 常用命令
```bash
yarn init
yarn add <package>
yarn remove <package>
yarn                # 安装所有依赖
yarn dlx <command>  # 类似 npx，运行临时包命令
yarn workspaces foreach run build
```

### 3.2 pnpm：高性能、磁盘空间节约的包管理器

pnpm 代表“performant npm”，通过硬链接和符号链接实现高效的依赖存储和严格的依赖隔离。

#### 核心原理
- **全局存储 + 硬链接**：所有项目共享一个全局包存储（`.pnpm-store`），`node_modules` 中的文件通过硬链接指向该存储，不会重复占用磁盘。
- **非扁平化的 node_modules**：
  - 只有项目直接声明的依赖会出现在根 `node_modules` 中。
  - 间接依赖（依赖的依赖）位于 `.pnpm` 目录并通过符号链接连接。
- **严格模式**：默认阻止访问未在 `package.json` 中声明的依赖（“幽灵依赖”），避免意外使用。

#### 主要优势
- **磁盘效率极高**：多个项目共享同一版本包时不重复存储，显著节省空间。
- **安装速度快**：无需复制文件，只需创建链接。
- **Monorepo 支持**：提供 `pnpm workspaces`，支持过滤、执行命令等高级操作。
- **兼容性好**：采用与 npm 几乎一致的 `node_modules` 结构（只是内部链接），生态兼容性极佳。

#### 常用命令
```bash
pnpm init
pnpm add <package>
pnpm remove <package>
pnpm install
pnpm dlx <command>  # 类似 npx
pnpm exec <command> # 执行依赖中的命令
pnpm run <script>
```

### 3.3 功能对比

| 功能/特性            | npm                          | Yarn Classic                | Yarn Berry (PnP)           | pnpm                        |
|---------------------|------------------------------|-----------------------------|----------------------------|-----------------------------|
| **依赖安装速度**     | 中等（串行，新版本已改进）   | 快（并行下载）              | 极快（零 I/O）             | 快（硬链接 + 并行）         |
| **磁盘空间利用**     | 一般（每个项目独立副本）     | 一般（有缓存但项目独立）    | 较好（全局缓存）           | 极优（全局存储 + 硬链接）   |
| **依赖树结构**       | 扁平化（有幽灵依赖）         | 扁平化（同 npm）            | 无 node_modules，依赖映射  | 非扁平化，严格隔离         |
| **锁定文件**         | `package-lock.json`          | `yarn.lock`                 | `yarn.lock` + `.pnp.cjs`   | `pnpm-lock.yaml`           |
| **Monorepo 支持**   | 基础 workspaces              | 原生 workspaces             | 原生 workspaces            | 强大的 workspaces + 过滤   |
| **幽灵依赖保护**     | 无                           | 无                          | 部分（访问控制）           | 默认禁止                   |
| **生态兼容性**       | 完全兼容                     | 高度兼容                    | 可能需调整配置             | 高度兼容                   |

---

## 4. NPX 的扩展与类比

`npx` 让临时运行包变得简单，Yarn 和 pnpm 也提供了等效工具：

- **yarn dlx**：功能同 npx，用于执行一次性命令。
  ```bash
  yarn dlx create-react-app my-app
  ```
- **pnpm dlx**：同样功能，可以自动清理临时下载。
  ```bash
  pnpm dlx eslint --init
  ```
- **pnpm exec**：运行项目本地依赖中的命令，类似于 `npx` 的本地查找行为。

这些工具使得无需全局污染、始终使用最新或指定版本的工具变得极其方便。

---

## 5. 总结笔记要点

1. **Node.js** 提供运行时，**NVM** 管理其版本，**NPM** 管理第三方包，**NPX** 让一次性执行变得简单。
2. TypeScript 兼容性通过 **编译后运行**、**ts-node**、**tsx** 或 **原生实验性支持** 实现，开发常用 `tsx` + 独立类型检查。
3. **Yarn** 强调可靠性和速度，Yarn Berry 的 PnP 创新尝试消除 `node_modules`；**pnpm** 凭借硬链接和严格依赖树极大节省磁盘空间并避免幽灵依赖。
4. 三者都有 Monorepo 支持，pnpm 和 Yarn 在大型项目管理上略胜一筹，而 npm 作为默认选择门槛最低。
5. 所有包管理器均提供类似 `npx` 的一次性执行功能（`yarn dlx`，`pnpm dlx`）。

根据项目规模和团队偏好选择合适的工具链，可显著提升开发效率和依赖管理的可靠性。