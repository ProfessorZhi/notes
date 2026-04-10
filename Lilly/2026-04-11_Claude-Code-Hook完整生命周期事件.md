# Claude Code Hook 系统完整参考

## 信息来源
- https://code.claude.com/docs/en/hooks

## Hook 生命周期事件全图

三类事件节奏：

### 1. Session 级（每 session 一次）
- SessionStart：session 开始或恢复时
- SessionEnd：session 结束时

### 2. Per-turn 级（每轮一次）
- UserPromptSubmit：提交 prompt 后，Claude 处理前
- Stop：Claude 完成响应时
- StopFailure：API 错误导致 turn 结束时

### 3. Agentic loop 内（每次工具调用）
- PreToolUse：工具执行前（可 block）
- PermissionRequest：权限对话框出现时
- PermissionDenied：auto mode 分类器拒绝时
- PostToolUse：工具执行成功后
- PostToolUseFailure：工具执行失败后
- SubagentStart/SubagentStop：subagent 启动/结束时
- TaskCreated/TaskCompleted：task 创建/完成时

### 4. 独立异步事件
- Notification：Claude Code 发送通知时
- ConfigChange：配置文件在 session 中变更时
- InstructionsLoaded：CLAUDE.md 或规则文件加载到上下文时（session 开始 + lazy 加载时）
- CwdChanged：工作目录改变时（如执行 cd 命令后）
- FileChanged：监视的文件在磁盘上变更时
- WorktreeCreate/WorktreeRemove：git worktree 创建/移除时
- TeammateIdle：agent team teammate 即将 idle 时
- Elicitation/ElicitationResult：MCP 工具执行内的征求
- PreCompact/PostCompact：压缩前/后

## 重要新增事件详解

### PermissionDenied 事件
当 auto mode 分类器拒绝工具调用时触发。
可返回 {retry: true} 告诉模型可以重试被拒绝的工具调用。

### InstructionsLoaded 事件
CLAUDE.md 或 .claude/rules/*.md 加载到上下文时触发。
- session 开始时触发
- lazy 加载时也触发（如读取匹配路径规则的文件时）
这意味着可以监视 Claude 加载了什么规则文件。

### CwdChanged 事件
工作目录改变时触发（如执行 cd 命令）。
用途：与 direnv 等工具配合，实现响应式环境管理。

### FileChanged 事件
监视的文件在磁盘变更时触发。
matcher 字段指定监视的文件名。
用途：外部文件变更触发 Claude 重新加载或响应。

### PreCompact/PostCompact
压缩前后触发。
可以监视压缩质量或注入额外上下文。

## Hook 类型

1. **Command hook**：shell 命令，stdin 接收 JSON
2. **HTTP hook**：HTTP 端点，POST body 接收 JSON
3. **Prompt hook**：LLM prompt，用于需要 LLM 判断的场景

## 工程意义

1. **InstructionsLoaded 可用于审计**：监视 CLAUDE.md 和规则何时加载，可以了解 Claude 看到了什么
2. **CwdChanged + direnv 是响应式环境管理**：工作目录改变时自动加载对应环境变量
3. **FileChanged 实现外部触发的重新加载**：外部文件变更触发 Claude 响应，不需要人工干预
4. **PermissionDenied retry 机制**：auto mode 被拒绝的工具调用可以重试，比直接拒绝更灵活
5. **PreCompact/PostCompact 监视压缩质量**：可以在压缩前后注入额外上下文

## 与 OpenClaw Hook 对比

| 事件 | Claude Code | OpenClaw |
|------|------------|---------|
| 工具调用前 | PreToolUse | before_tool_call |
| 工具调用后 | PostToolUse | after_tool_call |
| session 开始 | SessionStart | session_start |
| session 结束 | SessionEnd | session_end |
| 消息发送前 | - | message_sending |
| 消息发送后 | - | message_sent |
| compact 前 | PreCompact | before_compaction |
| 压缩后 | PostCompact | after_compaction |
| 目录变更 | CwdChanged | 无对等 |
| 文件变更 | FileChanged | 无对等 |
| 规则加载 | InstructionsLoaded | 无对等 |
