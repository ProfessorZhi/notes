# OpenClaw 与 Claude Code Agent Loop 对比

## 信息来源

- OpenClaw: `/home/dministrator/openclaw-wsl/docs/concepts/agent-loop.md`
- Claude Code: `code.claude.com/docs/en/how-claude-code-works` + 源码泄露

## Claude Code Agentic Loop

```
用户 prompt → gather context → take action → verify results → 重复直到完成
```

- 三个阶段循环：gather context / take action / verify results
- 用户可随时打断干预方向
- Model + tools 是核心驱动力
- 子 agent 有独立上下文窗口，隔离运行

## OpenClaw Agent Loop

```
Gateway RPC (agent/agent.wait)
  → 验证参数 + 解析 session
  → 持久化 session 元数据
  → 队列化 (per-session lane + global lane)
  → 加载 skills snapshot → 构建系统提示词
  → runEmbeddedPiAgent (pi-agent-core runtime)
  → 事件流: lifecycle / assistant / tool
  → reply shaping → 回复
```

## 关键区别

### 1. Loop 结构

| 维度 | Claude Code | OpenClaw |
|------|------------|---------|
| 循环触发 | 用户 prompt | Gateway RPC |
| 打断机制 | 用户随时可打断 | AbortSignal / timeout |
| 子 agent | 独立上下文窗口 | sessions_spawn |
| 验证机制 | Verification agent | 无内置验证 agent |

### 2. 队列模型

OpenClaw 有 per-session lane + global lane，防止 tool/session race：
- 每个 session 的运行是串行的
- 全局可选择是否共享 lane
- Claude Code 无此设计（本地 terminal 环境，无并发场景）

### 3. Hook 系统

OpenClaw 有非常细粒度的 lifecycle hook：

```
before_model_resolve
before_prompt_build
before_agent_start
agent_end
before_compaction / after_compaction
before_tool_call / after_tool_call
before_install
tool_result_persist
message_received / message_sending / message_sent
session_start / session_end
gateway_start / gateway_stop
```

共 ~15 个 hook 点，覆盖完整生命周期。

Claude Code 的 hook 机制未在泄露源码中看到详细设计。

### 4. 工具执行

Claude Code: 每个工具是独立原语，tool call 是 loop 中的基本单元

OpenClaw: `before_tool_call` / `after_tool_call` 拦截点，可以 block 或修改结果

### 5. 子 agent 模型

Claude Code:
- subagent 独立上下文窗口（核心设计）
- subagent 不能嵌套 spawn
- 返回摘要，不撑爆主上下文

OpenClaw:
- sessions_spawn 创建子 agent
- 消息传回主 session（不是完全隔离）
- 无嵌套限制说明

## OpenClaw 的优势

1. **细粒度 lifecycle hook**：可以在任何阶段拦截、修改、阻止行为
2. **队列并发控制**：防止 race condition，保证 session 历史一致性
3. **Compaction 自动化**：自动压缩长对话，减少 token 消耗
4. **多 channel 支持**：Telegram/Feishu/Discord 等多 channel routing

## Claude Code 的优势

1. **上下文隔离是核心设计**：subagent 不会撑爆主对话
2. **Verification agent**：专职验证 agent，防止"代码看起来对"就结束
3. **用户体验打断机制**：随时可干预方向
4. **Model-tier 节省成本**：Haiku/Sonnet/Opus 按需选择

## 工程意义

OpenClaw 的 hook 系统适合需要监管和合规的企业场景（每个操作都可审计/拦截）。Claude Code 的隔离模型适合长程开发任务（上下文不撑爆）。

在中国区 agent 工程中：
- 需要监管：借鉴 OpenClaw 的 before_tool_call 等 hook
- 需要长程任务：借鉴 Claude Code 的 subagent 隔离模型
- 验证环节：Claude Code 的 Verification agent 模式值得引入

## 来源

- OpenClaw agent-loop: `openclaw-wsl/docs/concepts/agent-loop.md`
- Claude Code how-it-works: `code.claude.com/docs/en/how-claude-code-works`
