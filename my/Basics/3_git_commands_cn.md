# Git 常用指令速查与详解

覆盖仓库初始化、日常提交、分支协作、远端同步，以及误操作后的救场命令。

> 核心原则：Git 的本质不是“记命令”，而是分清工作区、暂存区、本地提交、远端分支各自处在哪一层；一旦层次想清楚，命令就不容易混。

## 第一部分：速查列表

### 初始化与远端连接

| 命令 | 一句话作用 |
| --- | --- |
| git config | 设置 Git 用户名、邮箱和全局偏好。 |
| git init | 把当前目录初始化成 Git 仓库。 |
| git clone | 把远端仓库完整复制到本地。 |
| git remote -v | 查看当前仓库关联了哪些远端。 |
| git remote add origin <url> | 给本地仓库绑定一个远端地址。 |

### 日常提交

| 命令 | 一句话作用 |
| --- | --- |
| git status | 看工作区和暂存区当前状态。 |
| git add | 把改动放进暂存区。 |
| git commit -m | 生成一次本地提交。 |
| git log --oneline --graph --decorate | 紧凑查看提交历史和分支结构。 |
| git diff | 查看尚未提交的代码差异。 |
| git restore <file> | 撤销工作区里未暂存的改动。 |
| git restore --staged <file> | 把文件从暂存区拿出来，但保留本地修改。 |
| git rm | 从仓库中删除文件。 |
| git mv | 用 Git 语义移动或重命名文件。 |

### 分支与合并

| 命令 | 一句话作用 |
| --- | --- |
| git branch | 查看或管理分支。 |
| git switch -c <branch> | 创建并切换到新分支。 |
| git switch <branch> | 切换到已有分支。 |
| git merge <branch> | 把别的分支合并进当前分支。 |
| git rebase <branch> | 把当前分支改写到另一条分支最新位置之上。 |

### 同步远端

| 命令 | 一句话作用 |
| --- | --- |
| git fetch | 只拉远端最新状态，不自动合并。 |
| git pull | 拉远端并尝试合并到当前分支。 |
| git push -u origin <branch> | 把本地分支推到远端并建立上游关系。 |
| git tag | 给某个提交打版本标签。 |

### 救场与修复

| 命令 | 一句话作用 |
| --- | --- |
| git stash | 临时把当前未提交改动收起来。 |
| git cherry-pick <commit> | 把某个提交单独摘到当前分支。 |
| git reset --soft HEAD~1 | 撤回最近一次提交，但保留改动在暂存区。 |
| git reset --hard HEAD | 把工作区和暂存区都强制回到当前提交。 |
| git revert <commit> | 用一个新提交去反向撤销旧提交。 |
| git reflog | 查看 HEAD 变动轨迹，找回“丢失”的提交。 |

## 第二部分：逐条详解

### 初始化与远端连接

#### `git config`
- **功能**：配置提交身份和常用行为，是第一次装 Git 后最先该做的步骤之一。
- **常见场景**：新电脑初始化、给公司账号和个人账号分别配置环境。
- **示例**：
```bash
git config --global user.name "Your Name"
```

#### `git init`
- **功能**：创建 .git 元数据，让这个目录开始具备版本控制能力。
- **常见场景**：从零开始新项目，或把老项目纳入 Git 管理。
- **示例**：
```bash
git init
```

#### `git clone`
- **功能**：下载仓库代码和历史记录，建立本地副本。
- **常见场景**：接手新项目、把 GitHub/GitLab 仓库拉到本机。
- **示例**：
```bash
git clone https://github.com/org/repo.git
```

#### `git remote -v`
- **功能**：列出 origin 等远端的读写地址。
- **常见场景**：确认 push 会推到哪、排查远端 URL 配错。
- **示例**：
```bash
git remote -v
```

#### `git remote add origin <url>`
- **功能**：把本地仓库和远端仓库建立连接，后续才能 push/pull。
- **常见场景**：本地先开发，之后才决定推到 GitHub/GitLab。
- **示例**：
```bash
git remote add origin git@github.com:org/repo.git
```

### 日常提交

#### `git status`
- **功能**：告诉你哪些文件改了、哪些已暂存、哪些还没被 Git 跟踪。
- **常见场景**：每次 add、commit、pull、merge 前后都该先看一眼。
- **示例**：
```bash
git status
```

#### `git add`
- **功能**：告诉 Git 下一次提交要包含哪些修改。
- **常见场景**：只提交一部分文件，或分批提交一个大改动。
- **示例**：
```bash
git add src/app.js
```

#### `git commit -m`
- **功能**：把暂存区内容固化成一个版本节点，并写上提交说明。
- **常见场景**：完成一个最小可回滚单元后立即提交。
- **示例**：
```bash
git commit -m "fix login redirect bug"
```

#### `git log --oneline --graph --decorate`
- **功能**：用更适合人看的方式展示历史，便于理解分支和合并关系。
- **常见场景**：追查某次改动来源、看当前分支落后或领先多少。
- **示例**：
```bash
git log --oneline --graph --decorate --all
```

#### `git diff`
- **功能**：比较工作区与暂存区、暂存区与最近提交之间的差异。
- **常见场景**：commit 前自检、code review 前快速过一遍。
- **示例**：
```bash
git diff
```

#### `git restore <file>`
- **功能**：把文件恢复到最近一次提交或暂存区状态，适合撤掉本地试错。
- **常见场景**：改坏了一个文件，不想手工一点点改回去。
- **示例**：
```bash
git restore src/config.js
```
- **注意**：这会丢掉该文件当前未提交的内容。

#### `git restore --staged <file>`
- **功能**：取消 add，不是撤销代码本身。
- **常见场景**：你 add 早了，想重新拆分提交范围。
- **示例**：
```bash
git restore --staged src/config.js
```

#### `git rm`
- **功能**：删除文件并把“删除动作”也放进版本控制。
- **常见场景**：确认某文件以后不再需要，想让仓库里正式删掉它。
- **示例**：
```bash
git rm old-script.sh
```

#### `git mv`
- **功能**：移动文件位置并让后续历史更清晰。
- **常见场景**：重构目录结构、统一命名。
- **示例**：
```bash
git mv old_name.js new_name.js
```

### 分支与合并

#### `git branch`
- **功能**：列出本地分支，也可以创建、删除分支。
- **常见场景**：确认自己在哪条线开发，或清理废弃分支。
- **示例**：
```bash
git branch
```

#### `git switch -c <branch>`
- **功能**：这是现代 Git 更清晰的开新分支方式。
- **常见场景**：开始新功能、新修复、新实验时。
- **示例**：
```bash
git switch -c feature/payment
```

#### `git switch <branch>`
- **功能**：在不同任务线之间切换工作区。
- **常见场景**：从开发分支回 main，或者切到某个修复分支。
- **示例**：
```bash
git switch main
```

#### `git merge <branch>`
- **功能**：把目标分支的提交并入当前分支，必要时会产生 merge commit。
- **常见场景**：功能分支开发完成，要合回主分支。
- **示例**：
```bash
git merge feature/payment
```

#### `git rebase <branch>`
- **功能**：让历史更线性，减少不必要的 merge 结点。
- **常见场景**：提交 PR 前，先把功能分支基于最新 main 重放一遍。
- **示例**：
```bash
git rebase main
```
- **注意**：已经共享给别人的历史不要轻易 rebase。

### 同步远端

#### `git fetch`
- **功能**：更新远端分支信息到本地，风险比 pull 小，因为它不直接改当前工作树。
- **常见场景**：想先看看远端发生了什么，再决定怎么合。
- **示例**：
```bash
git fetch origin
```

#### `git pull`
- **功能**：相当于 fetch + merge（某些配置下也可能是 rebase）。
- **常见场景**：准备继续开发前，把当前分支与远端对齐。
- **示例**：
```bash
git pull origin main
```
- **注意**：有本地未提交改动时直接 pull，最容易把现场搞乱。

#### `git push -u origin <branch>`
- **功能**：第一次推新分支时最常用，后续再 push 可省略远端和分支名。
- **常见场景**：新功能分支要发到远端开 PR。
- **示例**：
```bash
git push -u origin feature/payment
```

#### `git tag`
- **功能**：常用于发布版本、里程碑节点或回滚基线。
- **常见场景**：发 v1.2.0、标记上线点。
- **示例**：
```bash
git tag v1.2.0
```

### 救场与修复

#### `git stash`
- **功能**：把工作区和暂存区内容先塞进一个临时栈，等你切别的分支处理完再取回。
- **常见场景**：做到一半突然要紧急切分支修线上 bug。
- **示例**：
```bash
git stash push -m "wip payment refactor"
```

#### `git cherry-pick <commit>`
- **功能**：精确搬运一个提交，而不是整条分支都合进来。
- **常见场景**：线上修复只需要另一条分支里的一个补丁。
- **示例**：
```bash
git cherry-pick a1b2c3d
```

#### `git reset --soft HEAD~1`
- **功能**：把 commit 回退掉，方便你改提交说明或重组本次提交。
- **常见场景**：刚 commit 就发现信息写错，或者还想补点内容再提交。
- **示例**：
```bash
git reset --soft HEAD~1
```

#### `git reset --hard HEAD`
- **功能**：彻底丢弃尚未提交的本地修改。
- **常见场景**：本地改乱了，确认不要了，直接回到干净状态。
- **示例**：
```bash
git reset --hard HEAD
```
- **注意**：危险操作，未提交内容会直接消失。

#### `git revert <commit>`
- **功能**：不是改历史，而是追加一个“反操作”提交，更适合公共分支。
- **常见场景**：某个已推送到远端的提交有问题，但你不想改写共享历史。
- **示例**：
```bash
git revert a1b2c3d
```

#### `git reflog`
- **功能**：记录本地引用移动历史，是误 reset、误 rebase 后的救命绳。
- **常见场景**：分支突然找不到了，提交像是丢了，其实常能在 reflog 里捞回来。
- **示例**：
```bash
git reflog
```
