# Claude Code 架构：核心系统分析

> 来源：dev.to 文章（基于 Rust rewrite 分析）+ Claude Code 官方 agent-loop 文档
> 时间：2026-04-10

## 架构概览：6 模块结构

```
CLI / REPL (用户交互层)
──────────────────────────────────
MCP Protocol · Sub-agents (扩展层)
──────────────────────────────────
API Client · Session Management (通信层)
──────────────────────────────────
System Prompt · Config (上下文层)
──────────────────────────────────
Agent Loop · Tools · Permissions (核心层)
```

Rust 实现模块：

- `runtime/`：核心运行时：loop、permissions、config、session、prompt
- `api/`：LLM 客户端、SSE 流、OAuth
- `tools/`：工具注册和执行
- `commands/`：斜杠命令（/help、/cost）
- `compat-harness/`：TS → Rust 兼容性层
- `rusty-claude-cli/`：CLI、REPL、终端渲染

## 核心发现：88 行实现整个 Agent Loop

```rust
AgentRuntime {
    session          // message 数组（唯一状态）
    api_client       // LLM 接口
    tool_executor    // 工具执行
    permission_policy // 访问控制
    system_prompt
    max_iterations
    usage_tracker
}
```

**令人惊讶的结论：唯一状态就是 message 数组。**
没有显式状态机，没有工作流图。

### run_turn() 核心循环

```python
def run_turn(user_input):
    session.messages.append(UserMessage(user_input))

    while True:
        if iterations > max_iterations:
            raise Error("Max iterations exceeded")

        response = api_client.stream(system_prompt, session.messages)
        assistant_message = parse_response(response)
        session.messages.append(assistant_message)

        tool_calls = extract_tool_uses(assistant_message)

        if not tool_calls:
            break  # LLM 决定停止

        for tool_name, input in tool_calls:
            permission = authorize(tool_name, input)
            if permission == Allow:
                result = tool_executor.execute(tool_name, input)
                session.messages.append(ToolResult(result))
            else:
                session.messages.append(ToolResult(deny_reason, is_error=True))
```

### 关键设计点 1：消息 = 状态

- 系统把所有内容存为消息
- 全部状态可从历史中重建

好处：持久化简单 / 重放简单 / 压缩简单
> 一个 append-only 结构解决多个问题。

### 关键设计点 2：错误是反馈

工具被拒绝时：系统不崩溃，而是把错误作为 `ToolResult` 返回给模型。
模型收到后：适应并选择替代策略。
> 失败成为推理循环的一部分。

## 工具系统：18 个工具，一种模式

三层结构：

1. **Tool Registry**：定义 schema 和 permissions
2. **Dispatcher**：路由工具调用
3. **Implementation**：执行逻辑

### 工具规范（JSON Schema）

```json
{
  "name": "bash",
  "description": "Execute shell commands",
  "input_schema": {
    "command": "string",
    "timeout": "number?"
  },
  "required_permission": "DangerFullAccess"
}
```

Schema 是 LLM 和实现之间的契约，解耦语言相关细节。

## 子 Agent 设计

子 Agent 复用同一运行时，但使用受限工具集和更高权限：

```python
runtime = AgentRuntime(
    session = new_session,
    tool_executor = restricted_tools,
    permission = high,
    prompter = None
)
```

**安全约束：子 Agent 不能生成子 Agent**，防止递归循环。

## 权限系统：5 级 + 渐进升级

权限级别：**ReadOnly < WorkspaceWrite < DangerFullAccess < Prompt < Allow**

核心逻辑：

```python
if current >= required:
    allow
elif one_level_gap:
    ask_user  # 差一级询问用户
else:
    deny
```

### 子 Agent 安全模型

- 子 Agent 有高权限
- 但没有用户 prompt 接口
- 在范围内允许，超出自动阻止

两个机制组合成精确控制。

## 与 OpenClaw 的关键区别

| 维度 | Claude Code | OpenClaw |
|------|------------|----------|
| 状态管理 | 唯一状态 = message 数组 | session + 多种状态 |
| 终止条件 | LLM 自己决定何时停止 | 显式 max_turns / max_budget |
| 错误处理 | ToolResult 错误反馈给 LLM 自适应 | hook 拦截可 block |
| 工具 schema | JSON Schema 作为 LLM 契约 | tool schema 由框架定义 |
| 权限模型 | 5 级渐进升级 | hook 级别的 block/allow |
| 子 Agent | 同一运行时 + 受限工具 | 多 workspace 隔离 |

## Claude Code 官方 Agent Loop 文档要点

（来源：code.claude.com/docs/en/agent-sdk/agent-loop）

### 消息类型

- **SystemMessage**：session 生命周期（init / compact_boundary）
- **AssistantMessage**：每次 Claude 响应后发出，含 text + tool calls
- **UserMessage**：每次工具执行后发出，含工具结果
- **StreamEvent**：部分消息（streaming 时）
- **ResultMessage**：最后一条，含最终文本 + token 使用量 + 费用 + session ID

### Turn 定义

一个 turn = Claude 产生 tool calls + SDK 执行 + 结果反馈给 Claude。
循环直到 Claude 产生不带 tool calls 的输出为止。

### 限制参数

- `max_turns`：只计算工具调用轮次
- `max_budget_usd`：按花费阈值停止

## 核心原则总结

1. **消息是唯一状态**：append-only 结构，状态可重建
2. **LLM 决定何时停止**：终止条件由模型判断而非框架
3. **工具由 schema 驱动**：LLM 通过契约调用实现
4. **错误是反馈的一部分**：模型在错误中自适应
5. **权限是渐进式的**：小差距询问用户，大差距拒绝

## 下一步

- Part 2: Context Engineering and Design Patterns（prompt 构建、config 合并、context compression）
- 参考：https://claw-code.codes/（Rust rewrite 项目）
