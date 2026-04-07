# Claude Code 泄露源码分析

## 背景

2026-03-31，Anthropic 在 npm registry 误发完整源码（512K 行 TypeScript，1906 文件），通过 .map 文件暴露。

## 源码位置

```
/tmp/claude-code-leak/original-source-code/src.zip  → 解压后 /tmp/claude-src/src/
```

## 核心架构

### 1. Tool 系统

- 中心注册表：`src/tools.ts` 导入所有工具
- ~30+ 内置工具：AgentTool, BashTool, FileEditTool, FileReadTool, GlobTool, GrepTool, WebSearchTool, WebFetchTool, TaskCreateTool, TaskListTool, SkillTool 等
- 工具定义：`src/Tool.ts` — 统一的 Tool 接口和 ToolInputJSONSchema
- 支持 MCP 工具（MCP tool）和条件编译工具（feature flags）

### 2. Agent 系统

**内置 Agent 类型**（`tools/AgentTool/built-in/`）：
- `general-purpose`：通用研究 agent，工具权限全开
- `coder`：专门写代码，强调简洁、最小改动
- `reviewer`：代码审查，分 Critical / Warning / Suggestion
- `researcher`：专注于代码库理解，引用文件路径和行号
- `tester`：测试 agent
- `Explore` / `Plan`：一次性任务，不返回继续对话
- `verification`：验证 agent

**Agent 定义结构**：
```typescript
interface AgentDefinition {
  agentType: string
  whenToUse: string          // 何时使用
  tools: string[] | ['*']   // 工具列表，* 表示全部
  source: 'built-in' | 'user' | 'project'
  baseDir: string
  model?: string             // 空则继承父 agent
  system_prompt?: string     // 额外系统提示
  getSystemPrompt?(): string // 动态生成
}
```

**AgentTool**（`tools/AgentTool/AgentTool.tsx`）：spawn 子 agent 的工具，支持同步/异步、isolation 模式

### 3. Task 系统

**任务类型**（`src/Task.ts`）：
- `local_bash` (b*)：本地 shell
- `local_agent` (a*)：本地 agent 任务
- `remote_agent` (r*)：远程 agent
- `in_process_teammate` (t*)：进程内 teammate
- `local_workflow` (w*)：本地工作流
- `monitor_mcp` (m*)：MCP 监控
- `dream`：梦境/探索模式

**任务状态**：`pending → running → completed/failed/killed`

**Task ID 生成**：前缀 + 8位随机字符，36^8 ≈ 2.8 万亿组合

### 4. Coordinator Mode

`src/coordinator/coordinatorMode.ts`：
- 环境变量 `CLAUDE_CODE_COORDINATOR_MODE` 开启
- Coordinator 模式下，worker agents 通过 `AgentTool` spawn
- Worker tools list 通过 `ASYNC_AGENT_ALLOWED_TOOLS` 配置

### 5. 命令系统

`src/commands.ts`：slash commands 注册表，支持 `/btw`、`/config`、`/agent` 等

### 6. 特色设计

**Feature Flags**：
- 使用 `feature('FLAG_NAME')` 做条件编译（DCE）
- 大量环境变量控制功能开关
- 示例：`COORDINATOR_MODE`、`KAIROS`、`AGENT_TRIGGERS`、`MONITOR_TOOL`

**进程分离**：
- CLI 入口：`src/main.tsx`
- 任务执行通过 subprocess / IPC
- Bridge 系统处理进程间通信

## 与 OpenClaw 的对比

| 维度 | Claude Code | OpenClaw |
|---|---|---|
| Agent 定义 | 内置 + 用户定义 + 项目定义 | workspace/记忆 |
| Tool 注册 | 中心注册，30+ 内置 | 插件化工具 |
| Task 生命周期 | 完整状态机 + 输出文件 | 更轻量 |
| Multi-agent | AgentTool spawn，Coordinator Mode | 多 bot 分派 |
| 特色机制 | Feature flags 条件编译，ASCII pets，sprites | heartbeat/cron |

## 值得学习的点

1. **Tool 接口泛化**：所有工具统一 ToolInputJSONSchema，便于扩展
2. **Task 状态机**：用 `isTerminalTaskStatus()` 判断，避免死任务残留
3. **Feature Flag 做 DCE**：不让无用代码进入生产 bundle
4. **Agent 分层**：general-purpose / coder / reviewer / researcher 各司其职
5. **Built-in agent 动态加载**：支持禁用所有内置 agent（SDK 场景）

## 待深入

- [ ] Bridge 系统（进程间通信机制）
- [ ] Coordinator Worker 通信协议
- [ ] 权限系统（permission mode）
- [ ] Memory / sessionHistory 机制

---

## 深度分析：Coordinator 模式

### Coordinator System Prompt（核心摘录）

`src/coordinator/coordinatorMode.ts` 里的 `getCoordinatorSystemPrompt()` 是整个多 agent 编排的核心。Coordinator 被定义为一个**真正的协调者**，不是执行者：

**核心职责分层**：
1. **Research**：Workers 并行研究
2. **Synthesis**：Coordinator 本人读取结果，理解问题，写实现规范
3. **Implementation**：Workers 按规范执行
4. **Verification**：Workers 独立验证

**并行是核心能力**：
- 只读任务（research）自由并行
- 写任务（implementation）每次一个
- Verification 可以和 implementation 在不同文件区域并行

**Continue vs Spawn 的判断矩阵**：

| 情况 | 机制 | 原因 |
|------|------|------|
| 研究刚好覆盖要改的文件 | **Continue** | Worker 已有文件上下文 + 清晰计划 |
| 研究广但实现窄 | **Spawn fresh** | 避免引入探索噪声 |
| 纠正失败 | **Continue** | Worker 有错误上下文 |
| 验证其他 Worker 刚写的代码 | **Spawn fresh** | 验证者应该用全新视角 |
| 第一次尝试方向完全错误 | **Spawn fresh** | 错误上下文会污染重试 |

**最重要的规则**：Coordinator 必须自己先理解研究结果，才能写出好的实现规范。永远不要写"基于你的发现"这种话——要自己综合后给出具体的文件路径、行号和改动描述。

---

## 深度分析：Fork Subagent 机制

`tools/AgentTool/forkSubagent.ts` 实现了一种隐式 fork：

**触发条件**：不传 `subagent_type` 时触发 fork 实验

**关键设计**：
- Fork child 继承父 agent 的**完整对话上下文**
- 所有 fork child 的 API 请求前缀是**字节级相同**的（用于 prompt cache 共享）
- 方法：收集父 assistant message 的所有 tool_use blocks，用相同的占位符文本构建 tool_result blocks，然后附加每个 child 独有的 directive text block
- `FORK_PLACEHOLDER_RESULT = 'Fork started — processing in background'` 所有 child 一致

**禁止递归**：通过检测消息中的 `<FORK_BOILERPLATE_TAG>` 防止递归 fork

---

## 深度分析：Agent 生命周期与内存管理

`tasks/LocalAgentTask/LocalAgentTask.tsx` 中的 `LocalAgentTaskState` 暴露了几个关键设计：

**状态字段**：
```typescript
type LocalAgentTaskState = {
  type: 'local_agent'
  agentId: string
  prompt: string
  selectedAgent?: AgentDefinition
  agentType: string
  isBackgrounded: boolean        // 是否后台运行
  pendingMessages: string[]       // SendMessage 队列
  retain: boolean                // UI 持有标记
  diskLoaded: boolean            // 是否已从磁盘加载侧链
  messages?: Message[]           // 完整消息历史
}
```

**内存清理（runAgent.ts finally 块）**：
- 清理 agent 专用的 MCP servers
- 清理 session hooks
- 清理 prompt cache tracking
- 释放克隆的 file state cache
- 释放 fork context messages
- 从 perfetto 注册表移除
- 从 todos AppState 移除（防止泄漏：每个 subagent 调用 TodoWrite 都会留下 key）
- Kill 所有该 agent 启动的后台 bash 任务（防止 zombie 进程）
- Kill 所有该 agent 的 monitor 任务

---

## Feature Flag 系统

广泛使用 `feature('FLAG_NAME')` 做条件编译DCE：

```typescript
// Coordinator Mode
if (feature('COORDINATOR_MODE')) {
  return isEnvTruthy(process.env.CLAUDE_CODE_COORDINATOR_MODE)
}

// Fork Subagent（与 Coordinator 互斥）
if (feature('FORK_SUBAGENT')) {
  if (isCoordinatorMode()) return false
  return true
}

// 条件工具
const cronTools = feature('AGENT_TRIGGERS')
  ? [CronCreateTool, CronDeleteTool, CronListTool]
  : []

// 条件模块
const coordinatorModeModule = feature('COORDINATOR_MODE')
  ? require('./coordinator/coordinatorMode.js')
  : null
```

环境变量 + GrowthBook 双通道控制，支持 session 级动态切换。

---

## MCP 集成

每个 agent 可以定义自己的 MCP servers（加到父级的 MCP clients 上）：

```typescript
async function initializeAgentMcpServers(
  agentDefinition: AgentDefinition,
  parentClients: MCPServerConnection[],
): Promise<{
  clients: MCPServerConnection[]
  tools: Tools
  mcpCleanup: () => Promise<void>  // agent 结束时清理
}>
```

---

## 与 OpenClaw 的关键差异

| 维度 | Claude Code | OpenClaw |
|------|------------|----------|
| **编排模式** | Coordinator 大循环 + Workers 并行 | 多 bot 分派 + 主管收口 |
| **Continue 语义** | 同一 worker 继续，有完整上下文 | 改派/返工，原 bot 或换 bot |
| **任务标识** | 完整状态机 + 磁盘输出 | 轻量任务记录 |
| **内存管理** | AppState todos 清理 + zombie kill | workspace 隔离 |
| **Feature Flag** | 编译期 DCE + session 动态切换 | 配置驱动 |
