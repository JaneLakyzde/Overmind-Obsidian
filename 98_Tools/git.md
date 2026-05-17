---
aliases:
tags:
  - Code
date: 2026-05-16
---
![[9626046e84ef367ebb0bef5434c1d90d.jpg]]
# Git 概念辨析

1. Working Directory 工作区
	- working tree
2. Staging Area 暂存区
	- Index
3. Local Repository 本地仓库
	- HEAD `cat .git/head`
	
4. Remote Repository 远程仓库
5. remote branch tracking 远程分支追踪
6. stash 藏匿区

---

# 开发流程

### 1. 初始化
```bash
# 1. 进入你的项目文件夹 
cd /path/to/your/project 
# 2. 初始化 Git 仓库 
git init 
# 3. 配置当前仓库的用户名和邮箱 
git config user.name "你的名字" git config user.email "你的邮箱" 
# 4. 创建 .gitignore 文件，忽略不需要的垃圾文件（如日志、系统缓存等） 
# 这一步最好在第一次提交前完成！ 
echo "*.log" >> .gitignore echo ".DS_Store" >> .gitignore 
# 5. 将文件添加到暂存区并提交 
git add . 
git commit -m "feat: 初始化项目"
```

- 或者克隆仓库:
```bash
# 1. 克隆远程仓库到本地（会自动生成一个与仓库同名的文件夹） 
git clone https://github.com/username/repo.git 
# 或者，克隆时自定义本地的文件夹名字 
git clone https://github.com/username/repo.git my_project 
# 2. 进入克隆下来的项目目录 
cd repo # 或 cd my_project 
# 3. 正常进行开发、提交和推送
git clone --depth 1 https://github.com/username/repo.git#浅克隆节省空间
```

- 忽略文件:
```bash
# 场景：一开始忘了写 .gitignore，导致文件已经被 commit 到仓库了。
# 此时再去改 .gitignore 是无效的，因为 Git 已经开始追踪它们了。
# 解决办法：从暂存区和历史记录中删除追踪，但保留本地物理文件
git rm --cached .DS_Store
git rm -r --cached logs/
git commit -m "chore: 移除不小心提交的忽略文件"
```
## 2. 修改代码

- 准备工作
```bash
# 切换到主分支 
git checkout main 
git switch main     # 新版

# 获取远程仓库最新提交但不修改当前代码
git fetch origin
# 将远程 main 的改动合并到本地 main
git merge origin/main # 或者直接一步到位 git pull origin main

# 创建并切换到你的修改分支（分支名最好见名知意）
git branch fix/xxx
git checkout fix/xxx
# 或者直接一步到位 git checkout -b fix/xxx
```

## 3. 提交代码

- 单人提交
```bash
# 查看当前状态(修改、已上传、所在分支)
git status
# 工作区 -> 暂存区
git add .            # 修改和新增文件
git add -A           # 所有文件,包括被删除的
# 暂存区 -> 本地仓库
git commit -m "fix: 修复了xxx导致的空指针异常" # 提交并写明清晰的修改说明
# 刚执行完 commit，发现漏提交了一个文件，或者 commit message 写错了 
git add <漏掉的文件> git commit --amend -m "新的完整提交说明" 
# 结果：不会产生新的 commit，而是将改动合并到上一次 commit 中，保持历史整洁。
```

- 多人同步推送
```bash
# 在 push 前, 必须同步最新 main 避免冲突
# 拉取远程主分支最新代码并合并 
git pull origin main  
# 如果提示冲突(CONFLICT)，需要手动打开冲突文件解决
# 解决后执行 git add . 和 git commit 
# 推荐写法（变基拉取）：
git pull --rebase origin main # 放到远程最新提交的最后面, 保持美观

# 藏匿区避免冲突
git stash # 临时保存当前修改并恢复干净工作区
git stash apply # 恢复 stash 且保留记录
git stash pop   # 恢复 stash 并删除单条记录  pop = apply + drop
git stash clear # 清空 stash
git stash list  # 查看 stash

# 本地分支 -> 远程分支
git push origin fix/xxx 
# 第一次建议 git push -u origin fix/xxx 建立追踪关系, 下次直接 git push即可
# -u = --set-upstream
```

## 4. 撤销与回退

- 撤销工作区的修改 ( 还没 add )
```bash
git restore <file>
git checkout -- <file>
git restore .     # 丢弃当前目录下所有未追踪的修改(危险)
```

- 撤销暂存区的修改 ( 已经 add , 还没 commit )
```bash
git reset HEAD <文件名>       # 旧写法
git restore --staged <文件名> # 新写法
```

- 撤销本地仓库的提交 ( 已经 commit , 还没 push )
```bash
# 轻度后悔, 撤销commit, 代码改动保留在暂存区
git reset --soft HEAD~1    
# 中度后悔, 撤销commit和add, 代码保留工作区
git reset --mixed HEAD~1   # --mixed可省略, 默认中度后悔
# 重度后悔, 撤销所有改动, 代码直接回滚到上一个干净状态
git reset --hard HEAD~1  
```

- 撤销远程仓库的提交 ( 已经 push )
```bash
# 生成一个新的提交, 还原了对应commit的改动
# 不要使用reset撤销, 会导致冲突, 应该创建一个新的回退版本
git revert <commit哈希值>
```

- 高级操作
```bash
# 把本地commit合成一个干净的提交
git rebase -i <commit哈希值>
# 急救措施
git reflog                      # 记录了每一次HEAD指针的移动
git reset --hard HEAD@{1}       # 时光倒流回原始位置
```
---
# git 新特性
### git checkout (旧)
原本承担了三类完全不同的操作：
1. 切换分支
2. 恢复工作区文件
3. 创建并切换到新分支
```bash
git checkout <branch>           # 切换到已有分支
git checkout -b <new>           # 创建并切换到新分支
git checkout --detach <commit>  # 切换到游离 HEAD（不在任何分支上）
git checkout -- <file>          # 用暂存区或指定提交中的文件覆盖工作区
git checkout <commit> -- <file> # 从某次提交中提取文件到工作区并加入暂存区
git checkout -f                 # 强行切换，丢弃本地未提交修改        
git checkout -m                 # 切换时尝试合并本地修改，失败则进入冲突状态
```
### git switch

- 修改本地仓库中的 `HEAD` 指针，将目标分支最新提交的代码快照强制覆盖并同步到工作区和暂存区中. 可能会引发冲突
```bash
git switch <branch>                # 切换到已有分支
git switch -c <new>                # 创建并切换，等同于 checkout -b
git switch -C <new>                # 强制创建，若分支已存在则重置(危险)
git switch --detach <commit>       # 进入游离 HEAD
git switch -d <commit>             # 同上
git switch --discard-changes dev   # 切换并丢弃本地修改（checkout -f dev）
git switch -                       # 切回上一个分支
```

### git restore

- 专门用来恢复或撤销工作区和暂存区的内容, 不修改  `HEAD`指针
```bash
git restore <file>                 # 从暂存区恢复工作区文件（丢弃工作区修改）
git restore --staged <file>        # 从 HEAD 恢复暂存区文件（unstage）
git restore --source=<commit> <file>  # 从特定提交恢复文件
git restore --worktree --staged <file> # 同时恢复工作区和暂存区
```

---

# Detached HEAD

正常情况下 `HEAD` 指针指向当前所在的分支如 `main` , 分支再指向最新的提交`Commit`
当直接 `checkout` 到一个具体的历史 `Commit` 哈希值时`HEAD` 会直接指向这个 `Commit`，而不是分支名。这就是“游离状态”, 很容易被 Git 垃圾回收掉. 用 `switch` 可以避免.


如果你只是看看历史代码，看完直接切回主分支即可： 
```bash
git switch main
```

如果你在游离状态下修改了代码并想保留这些修改:
```bash
# 创建一个新分支
git switch -c fix/history-bug
git reflog
```

---
# 分支命名规范
```bash
# 分支命名规范
feature/login   # 功能开发
feature/order

fix/null-pointer  # Bug 修复
fix/login-error

hotfix/payment  # 热修复
refactor/cache  # 优化

release/v1.2.0 # 发布预备分支（测试阶段使用，测试通过后合并到 main 并打 tag）
```
# 提交信息命名规范
```bash
# Commit Message 规范
feat: 新增用户登录功能
fix: 修复空指针异常
refactor: 重构（既不是新增功能，也不是修改 bug 的代码变动）
docs: 更新 README
style: 代码格式修改（空格、缩进、逗号等，不影响代码运行）
test: 添加登录单元测试
chore: 构建过程或辅助工具的变动（如更新依赖包、修改 webpack 配置） perf: 性能优化 ci: 持续集成相关文件的修改（如 Github Actions, Jenkins 配置）
```