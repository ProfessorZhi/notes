# Claude Code Teammate 架构与 tmux 集成

## 核心发现：Claude Code 使用 tmux 作为多 Agent 执行环境

从 changelog bugfix 中反向推导的架构：

### 1. Teammate 类型

| Type | 说明 | 特点 |
|------|------|------|
| `in_process_teammate` | 进程内 teammate | 低延迟，但不能独立扩展 |
| tmux-spawned teammate | tmux pane 中运行 | 独立进程，隔离性好 |

### 2. tmux 集成细节

Claude Code 在 tmux 环境中运行 subagent 时：
- 使用 tmux pane 作为隔离执行环境
- 通过 `tmux new-window` / `tmux split-window` 创建 teammate pane
- pane count 用于任务管理
- **已知 bug**：tmux window 被 kill 或 renumber 后，subagent spawn 会永久失败（"Could not determine pane count"），2.1.92 修复

tmux 环境下的特殊处理：
- 通知（iTerm2/Kitty/Ghostty popups）需要 `set -g allow-passthrough on` 才能透传
- clipboard 操作在 tmux over SSH 时需要双重机制（直接 terminal write + tmux clipboard integration）
- iTerm2 split-pane teammates 有专用检测逻辑

### 3. Teammate Memory 管理问题（多个 bugfix）

Claude Code 在 teammate memory 管理上有历史问题：

1. **进程内 teammate 内存保留**（v2.1.78+）：
   - 父的完整对话历史被 teammate 生命周期持有
   - `/clear` 或 auto-compact 后无法 GC
   - 原因：parent conversation history pinned for teammate lifetime

2. **AppState 消息保留**（v2.1.78+）：
   - 长运行 teammate 在 AppState 中保留所有消息
   - 即使 conversation compaction 后也不释放

3. **Completed teammate task GC**（v2.1.77+）：
   - completed teammate tasks 从未从 session state 中 GC
   - 长期运行的 team session 内存泄漏

4. **Memory leak in agent teams**（v2.1.76+）：
   - completed teammate tasks 永不 GC

### 4. Teammate 生命周期管理

关键 hook：
- `TeammateIdle` hook - teammate 空闲时触发
- `TaskCompleted` hook - 任务完成时触发
- 两个 hook 都支持 `{"continue": false, "stopReason": "..."}` 停止 teammate

修复的问题：
- Teammate panes 在 leader exit 时未关闭（v2.1.92）
- Nested teammate spawning 防止（通过 Agent tool 的 name 参数）
- Teammate spinners 不遵守自定义 spinnerVerbs

### 5. API Provider Env Vars 传播

当使用 Bedrock、Vertex、Foundry 等第三方 provider 时：
- API provider 环境变量需要传播到 tmux-spawned 进程
- 否则 tmux 中的 teammate 会因为缺少认证而失败

## 工程意义

1. **tmux 是 Claude Code 多 agent 的核心隔离机制**：
   - 不是进程池，不是容器，而是 tmux pane
   - 这解释了为什么 Claude Code 在纯 Windows 环境（无 tmux）下的行为不同

2. **memory management 是 teammate 架构的长期技术债务**：
   - 多个版本持续修复 memory leak
   - 进程内 vs tmux 进程有不同的 GC 语义

3. **对于设计多 agent 系统**：
   - 需要注意 parent-child context 的生命周期管理
   - completed task GC 是必须解决的问题
   - tmux 集成适合类 Unix 环境，Windows 需要替代方案

## 来源

Claude Code changelog（~/.claude/cache/changelog.md）
- v2.1.92, v2.1.78, v2.1.77, v2.1.76 等版本
