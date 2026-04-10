# OpenClaw Agent Loop vs Claude Code 架构对比

## 数据来源

- OpenClaw: `/home/dministrator/openclaw-wsl/docs/concepts/Agent-loop.md` + `Multi-agent.md`
- Claude Code: 上次笔记（`/mnt/f/agent_project/code_agent/claw-code/rust/crates/runtime/src/`）

## 入口模型对比

| | Claude Code | OpenClaw |
|--|------------|----------|
| 入口 | BootstrapPlan 分阶段 fast path | `agent` RPC → `runEmbeddedPiAgent` |
| 阶段数量 | 12 个预定义阶段 | 无固定阶段概念，更扁平 |
| 短路机制 | 每个阶段可独立短路（fast path）| 超时 abort |

Claude Code 的 BootstrapPlan 去重 + 阶段化启动是一个相对精致的初始化框架。OpenClaw 的入口更直接：RPC 进来 → 验证 → 塞队列 → 执行。

## Hook 系统对比

| | Claude Code | OpenClaw |
|--|------------|----------|
| Hook 数量 | 3 个（PreToolUse / PostToolUse / PostToolUseFailure）| 17+ 个（分层） |
| 粒度 | 工具级 | 贯穿整个生命周期 |

OpenClaw 的 hook 覆盖更广：
- `before_model_resolve` — 模型选择前
- `before_prompt_build` — prompt 装配前（可注入选言上下文）
- `before_agent_start` — agent 启动前
- `agent_end` — agent 结束后可检查消息列表
- `before/after_tool_call` — 工具调用前后
- `before/after_compaction` — 压缩前后
- `message_received/sending/sent` — 消息生命周期
- `session_start/end` — 会话边界
- `gateway_start/stop` — 网关生命周期

Claude Code 的三级 Hook 设计更简洁，但 OpenClaw 的分层 hook 更适合生产级工作流。

## Multi-Agent 对比

| | Claude Code | OpenClaw |
|--|------------|----------|
| 多 agent 支持 | 无 | 完整支持 |
| 隔离单位 | 无 | agentId = workspace + agentDir + sessions + auth |
| 路由 | 无 | 绑定系统（channel / accountId / peer / guildId）|
| 跨 agent 搜索 | 无 | QMD extraCollections |

Claude Code 是单 agent 设计。OpenClaw 是完整的多租户系统，支持：
- 每个 agent 有独立 workspace、auth、session store
- 跨 channel 路由（Discord / Telegram / WhatsApp 等）
- 消息级别路由规则（peer kind + id）
- per-agent sandbox 和 tool 限制

## Queue 模型

两者都使用 lane 序列化：
- Claude Code: per-session lane + optional global lane
- OpenClaw: per-session + global queue，防止 session race 和历史不一致

## Streaming 模型

| | Claude Code | OpenClaw |
|--|------------|----------|
| Stream 类型 | 未详细看 | lifecycle / assistant / tool |
| 工具事件 | tool start/update/end | tool 事件 |
| 生命周期的 end/error | 有 | 有 |

OpenClaw 的 stream 分类更清晰，lifecycle 事件（start/end/error）是独立 stream。

## 工程价值总结

1. **OpenClaw 的优势**：多 agent 隔离、丰富的 hook 生命周期、完整的 channel 路由体系，适合做平台级 agent 系统
2. **Claude Code 的优势**：BootstrapPlan 的阶段化设计更干净，Hook 虽然少但足够，Session 压缩机制更主动
3. **关键差异**：OpenClaw 是设计来托管多个 agent 的平台；Claude Code 是单 agent CLI tool，架构更内聚但扩展性有限

