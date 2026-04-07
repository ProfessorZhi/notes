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
