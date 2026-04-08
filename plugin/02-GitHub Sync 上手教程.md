# GitHub Sync 上手教程

## 这个插件到底是干什么的

`GitHub Sync` 的本质不是“上传文件到 GitHub”，而是：

- 调本机 `git`
- 先提交本地改动
- 再和远端仓库同步

所以它更接近一个“简化过的 Git 同步按钮”。

## 原理

按你现在安装的这个插件版本，它的大致流程是：

1. 检查本地有没有改动
2. 有的话自动提交
3. 配置远程仓库
4. `fetch`
5. `pull`
6. 再 `push`

所以它不是单向推送。

### 这意味着什么

- 本地和远端都变了时，它会尝试合并
- 改的是同一处，就可能冲突
- 所以它不是“永远以本地为准”

## 你的当前配置

你本地现在的设置是：

- `Remote URL`：已填 GitHub 仓库地址
- `git binary location`：空
- `syncinterval`：`0`
- `Auto sync on startup`：关
- `Check status on startup`：开

## 配置页面逐项解释

### `Remote URL`

作用：

- 指定同步到哪个 GitHub 仓库

怎么配：

- 填完整仓库地址

推荐格式：

`https://github.com/ProfessorZhi/notes.git`

为什么：

- 用最标准格式，减少插件识别异常

### `git binary location`

作用：

- 告诉插件本机 `git` 可执行文件在哪

什么时候填：

- 只有 Obsidian 找不到 git 时才填

你现在怎么配：

- 留空是对的

### `Check status on startup`

作用：

- 打开 Obsidian 时检查本地是不是落后于远端

优点：

- 你能更早发现远端有更新

缺点：

- 每次启动都多一次检查
- 容易把“提醒”误当成“必须立刻同步”

推荐值：

- 如果你主要在一台电脑上写，关
- 如果你确实会被别的设备改远端，开

对你当前场景，我更建议：

- 关

### `Auto sync on startup`

作用：

- 启动 Obsidian 时自动同步

问题：

- 同步是会 `pull` 的
- 自动跑这一步，容易在你没注意时引发冲突

推荐值：

- 关

### `Auto sync at interval`

作用：

- 每隔几分钟自动同步一次

问题：

- 自动同步会让冲突更隐蔽
- `0` 没有实际意义

推荐值：

- 留空

不建议：

- 填 `0`

## 推荐你怎么配置

### 最稳配置

- `Remote URL`：`https://github.com/ProfessorZhi/notes.git`
- `git binary location`：留空
- `Check status on startup`：关
- `Auto sync on startup`：关
- `Auto sync at interval`：留空

为什么这么配：

- 先把自动行为都拿掉
- 同步动作由你手动触发
- 最容易理解，也最不容易悄悄出问题

## 这个插件适合什么场景

适合：

- 桌面端单人使用
- 以备份为主
- 主要在一台电脑编辑

不适合：

- 安卓和桌面频繁双向编辑
- 多设备同时改同一 vault
- 想要稳定、完整的 Git 工作流

## 最容易冲突的文件

这些文件最容易在不同设备上产生冲突：

- `.obsidian/workspace.json`
- `.obsidian/workspace-mobile.json`
- 同一篇正在编辑的笔记

## 推荐 `.gitignore`

如果你继续用这个插件，至少建议忽略：

```gitignore
.obsidian/workspace.json
.obsidian/workspace-mobile.json
```

原因：

- 这些文件主要记录界面状态
- 价值低
- 冲突概率高

## 你应该怎么用它

### 推荐工作流

1. 先在电脑端正常写笔记
2. 写完后手动点一次 `sync`
3. 看到成功提示就结束

不要：

- 启动就自动同步
- 多设备同时修改再同步
- 把它当成实时协作工具

## 遇到冲突时怎么理解

如果插件提示 `merge conflict`，意思不是“插件坏了”，而是：

- 本地和远端都改了
- Git 无法自动决定保留哪边

这时真正的问题不是“再点一次”，而是：

- 先判断以本地为准，还是以远端为准

## 一句话建议

把 `GitHub Sync` 当成“桌面端手动备份按钮”，不要当成“全平台自动同步方案”。
