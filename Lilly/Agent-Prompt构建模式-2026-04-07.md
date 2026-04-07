# Agent Prompt 构建模式：Conversation Entry 处理

## 核心问题

在多轮对话中，Agent 的"当前消息"应该是哪一条？
是最新的一条 assistant 消息，还是最新的 user/tool 消息？

## OpenClaw 的设计选择

`src/gateway/agent-prompt.ts` 中的 `buildAgentMessageFromConversationEntries` 给出答案：

```typescript
// 找最后一个 user 或 tool entry 作为"当前消息"
for (let i = entries.length - 1; i >= 0; i -= 1) {
  const role = entries[i]?.role;
  if (role === "user" || role === "tool") {
    currentIndex = i;
    break;
  }
}
```

理由：Agent 应该对最新的用户输入或工具输出做出反应，
而不是对 Assistant 自己刚说的话做回应。

## 工程判断

这个选择是合理的，原因：

1. **工具调用场景**：当 Agent 调用工具后，它需要看到工具结果才能决定下一步，而不是重复自己刚输出的内容
2. **多 Agent 协作**：如果一个 Agent 的输出直接传给下一个 Agent，下一个 Agent 应该处理信息内容，而不是复述上一个 Agent 的话
3. **历史记录友好**：历史部分仍然保留完整的对话轨迹，但 currentMessage 是最新的有效输入

## 反例风险

如果用 `entries[length-1]`（通常是最新的 assistant 消息）作为 currentMessage，
会导致：
- Agent 倾向于复述而非行动
- 工具结果被忽略
- 多轮状态机混乱

## 相关文件

- `openclaw-wsl/src/gateway/agent-prompt.ts`
- `openclaw-wsl/src/agents/acp-spawn.ts`（spawn 机制）
