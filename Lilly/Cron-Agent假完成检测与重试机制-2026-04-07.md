# Cron Agent 的"假完成"检测与重试机制

## 核心问题

当一个 cron/heartbeat agent 被触发后，如果它第一轮只回复了"收到"/"正在处理"这类确认消息，而没有真正完成任务，会导致 cron run 返回一个空洞的结果。

更隐蔽的bug是：agent 在第一轮调用了 `sessions_spawn` 派生子 agent，但子 agent 还没开始跑，第一轮就结束了，导致 cron run 误以为任务完成。

## OpenClaw 的解决方案

`openclaw-wsl/src/cron/isolated-agent/run.ts` 中的 `shouldRetryInterimAck` 逻辑：

```typescript
const shouldRetryInterimAck =
  !interimRunResult.meta?.error &&                          // 没有报错
  !interimRunResult.didSendViaMessagingTool &&            // 没有通过消息工具发送
  !interimPayloadHasStructuredContent &&                   // 没有结构化输出
  !interimPayloads.some((payload) => payload?.isError === true) && // 没有错误payload
  countActiveDescendantRuns(agentSessionKey) === 0 &&     // 没有活跃子agent
  !hasDescendantsSinceRunStart &&                          // 本轮没有新的子agent被派发
  isLikelyInterimCronMessage(interimText);                 // 文本像是"确认"而非"完成"
```

如果满足上述条件，强制执行第二轮：

```typescript
const continuationPrompt = [
  "Your previous response was only an acknowledgement and did not complete this cron task.",
  "Complete the original task now.",
  "Do not send a status update like 'on it'.",
  "Use tools when needed, including sessions_spawn for parallel subtasks, " +
    "wait for spawned subagents to finish, then return only the final summary.",
].join(" ");
await runPrompt(continuationPrompt);
```

## 工程判断：这个设计好在哪里

1. **解决了"确认≠完成"的问题**：agent 说"收到"不代表任务完成了
2. **解决了子 agent 异步派发的时序问题**：如果第一轮刚派发完子 agent 就结束，不会误判为完成
3. **给 agent 第二次机会**：而不是直接判失败，降低了误判率
4. **有结构性保障**：检查 `didSendViaMessagingTool` 和 `hasDescendantsSinceRunStart`，区分"发了消息"和"只是嘴上说在做"

## 对未来 agent 设计的启发

在设计任何 autonomous cron/scheduled agent 时，必须考虑：
- 如何区分"确认响应"和"完成响应"
- 如果 agent 依赖派生子 agent 来完成任务，第一轮结束后的状态判断必须等子 agent 完成
- 或者：直接像 OpenClaw 这样做二次确认轮

## 相关文件

- `openclaw-wsl/src/cron/isolated-agent/run.ts` — `shouldRetryInterimAck` 和 `runCronIsolatedAgentTurn`
