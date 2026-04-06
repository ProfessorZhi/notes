# OpenClaw Session 与 Cron 机制笔记

## Session 路由规则

| 来源 | 行为 |
|------|------|
| DM | 默认共享 session |
| 群聊 | 按群隔离 |
| Cron jobs | 每次运行全新 session |
| Webhooks | 隔离 session |

## 重要含义

Cron job 每次运行是 fresh session：这解释了为什么 cron 触发的任务默认不共享上下文。Lilly 的心跳如果通过 cron 触发，每次都是独立 session，不会继承前一次的变量状态。

DM 默认共享 session：如果多人用同一个 bot，DM 会共享上下文。需要配置 dmScope 来隔离。

## Session 存储位置

- 元数据: ~/.openclaw/agents/<agentId>/sessions/sessions.json
- 完整记录: ~/.openclaw/agents/<agentId>/sessions/<sessionId>.jsonl
- 可用 openclaw sessions list 查看当前 session 列表

## Session 生命周期

- 每日重置: 默认早上 4:00（本地时间）
- 空闲重置: 可选，配置 idleMinutes
- 手动重置: /new 或 /reset

## 来源

/home/dministrator/openclaw-wsl/docs/concepts/session.md
