# Claude Code 核心架构：Rust 重写精简分析（Part 1）

## 信息来源

dev.to: "Claude Code Architecture Explained: Agent Loop, Tool System, and Permission Model (Rust Rewrite Analysis)"
https://dev.to/brooks_wilson_36fbefbbae4/claude-code-architecture-explained-agent-loop-tool-system-and-permission-model-rust-rewrite-41b2

## 核心发现：88 行实现核心 Agent Loop

Claude Code 的 Rust 重写版本（claw-code）将核心功能压缩到约 20K 行。
关键文件 `conversation.rs` 的核心逻辑只有约 **88 行**。

## 架构六模块

```
CLI / REPL (用户交互层)
─────────────────────────────
MCP Protocol · Sub-agents (扩展层)
─────────────────────────────
API Client · Session Management (通信层)
─────────────────────────────
System Prompt · Config (上下文层)
─────────────────────────────
Agent Loop · Tools · Permissions (核心层)
```

## AgentRuntime 的全部状态

```rust
struct AgentRuntime {
    session: MessageArray,      // 唯一状态：消息数组
    api_client: LLMInterface,
    tool_executor: ToolExecutor,
    permission_policy: AccessControl,
    system_prompt,
    max_iterations,
    usage_tracker,
}
```

**惊人事实：唯一的显式状态就是消息数组。** 没有状态机，没有工作流图。

## 核心循环

```python
def run_turn(user_input):
    session.messages.append(UserMessage(user_input))
    while True:
        if iterations > max_iterations:
            raise Error("Max iterations exceeded")
        response = api_client.stream(system_prompt, session.messages)
        assistant = parse_response(response)
        session.messages.append(assistant)
        tool_calls = extract_tool_calls(assistant)
        if not tool_calls:
            break  # LLM 决定停止 → 循环退出
        for name, input in tool_calls:
            permission = authorize(name, input)
            if permission == Allow:
                result = tool_executor.execute(name, input)
                session.messages.append(ToolResult(result))
            else:
                session.messages.append(ToolResult(deny_reason, is_error=True))
```

**终止条件：LLM 决定不再调用工具。** 不是预定义条件，是 LLM 的自主决策。

## 关键设计洞察

### 1. 消息 = 状态

- 所有状态都存储为消息
- 完整状态可从历史重建
- 好处：易持久化、易回放、易压缩

### 2. 错误即反馈

- 工具被拒绝 → 返回 `ToolResult(is_error=True)`
- LLM 收到错误后自适应，选择替代策略
- **失败成为推理循环的一部分**

### 3. JSON Schema 作为契约

```json
{
  "name": "bash",
  "description": "Execute shell commands",
  "input_schema": { "command": "string", "timeout": "number?" },
  "required_permission": "DangerFullAccess"
}
```

Schema 直接传给 LLM，解耦 LLM 与实现。

## 权限系统：5 级递进

| 级别 | 说明 |
|------|------|
| ReadOnly | 只读 |
| WorkspaceWrite | 写工作区 |
| DangerFullAccess | 危险操作 |
| Prompt | 提示注入风险 |
| Allow | 允许一切 |

核心逻辑：
```python
if current >= required:
    allow
elif one_level_gap:
    ask_user  # 差一级 → 询问用户
else:
    deny      # 差太多 → 直接拒绝
```

**设计原则：渐进式权限升级，而非全有或全无。**

## Subagent 安全模型

- Sub-agent 重用相同 runtime，但配置受限工具集
- **关键约束：Sub-agent 不能 spawn sub-agent** → 防止递归循环
- 有高权限，但没有用户 prompt 接口 → 精确控制边界

## 工程意义

| 模式 | 可迁移到 OpenClaw/自建 Agent 系统 |
|------|--------------------------------|
| 消息即状态 | OpenClaw session 消息数组 |
| 错误即反馈 | tool_call 失败返回给 LLM 而非崩溃 |
| Schema 契约 | 工具 schema 驱动 LLM 工具选择 |
| 渐进权限 | before_tool_call hook 做分级审批 |
| Subagent 禁止递归 | sessions_spawn 禁止二级 spawn |

## 核心教训

> 500K 行代码剥离后，剩下的是一个循环、一个工具接口、一个权限系统。
> 这是构建功能型 Agent 的最小集。
> 但让它稳健、可扩展、安全——那才是真正复杂的地方。

Part 2 预告：Context Engineering and Design Patterns（上下文工程与设计模式）。
