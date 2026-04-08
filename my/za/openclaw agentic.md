你现在遇到的不是“Feishu 接入失败”，而是 **触发模型没配对**。

**结论先说：**  
把 agent 接到飞书，默认只代表它**能收消息并回复**；**不代表它会自己在后台主动跑**。  
要让它“自己运行”，你还得额外启用下面至少一种机制：

1. **Heartbeat**：定时唤醒 agent 自己检查和行动
    
2. **Cron / Scheduled Tasks**：按时间计划执行任务
    
3. **Webhook / 外部事件**：有事件就主动触发
    
4. **特定事件源**：比如 Feishu Drive 评论事件，不是只有聊天消息 ([OpenClaw](https://docs.openclaw.ai/gateway/configuration-reference "Configuration Reference - OpenClaw"))
    

---

## 你现在为什么“不会自己跑”

因为 Feishu 这条通道，本质上默认是**消息驱动**的。  
官方文档里 Feishu 重点支持的是接收消息、线程回复，以及文档评论触发；不是“接上就自动 24/7 自治”。([OpenClaw](https://docs.openclaw.ai/channels/feishu "Feishu - OpenClaw"))

而 OpenClaw 的后台主动执行，是靠 **Gateway 内置调度器和 heartbeat** 完成的：

- Cron 是 Gateway 内置 scheduler，会在到点时唤醒 agent，并把结果投递到聊天通道或 webhook。([OpenClaw](https://docs.openclaw.ai/automation/cron-jobs "Scheduled Tasks - OpenClaw"))
    
- Heartbeat 是 agent 的周期性运行配置，默认示例是每 30 分钟一次。([OpenClaw](https://docs.openclaw.ai/gateway/configuration-reference "Configuration Reference - OpenClaw"))
    

所以你现在的现象其实很正常：

- **飞书负责“入口/出口”**
    
- **heartbeat / cron / hooks 负责“主动运行”**
    

---

## 最短路径：怎么让它真的后台跑起来

### 方案 A：开 heartbeat

这是最像“自己背后运行”的。

配置里有 `agents.defaults.heartbeat`，可以设：

- `every`: 多久跑一次
    
- `prompt`: 每次心跳时它要做什么
    
- `target` / `to`: 结果发到哪里
    
- `isolatedSession`: 是否每次独立上下文，减少 token 消耗
    

官方配置参考里明确写了：

- `every: "30m"` 这类周期运行
    
- `0m` 表示禁用
    
- heartbeat 跑的是**完整 agent turn**
    
- 间隔越短，token 消耗越大 ([OpenClaw](https://docs.openclaw.ai/gateway/configuration-reference "Configuration Reference - OpenClaw"))
    

一个最小思路大概像这样：

```json
{
  "agents": {
    "defaults": {
      "heartbeat": {
        "every": "30m",
        "session": "main",
        "isolatedSession": true,
        "target": "feishu",
        "to": "你的飞书目标ID",
        "prompt": "检查待办、日历、新事件；如果有需要处理的事情就执行并汇报，没有则简短汇报。"
      }
    }
  }
}
```

注意两点：

- `to` 必须是**可投递目标**
    
- Gateway 进程必须**一直活着**，否则 heartbeat 不会跑
    

---

### 方案 B：用 cron 明确建任务

这适合“每天早上 9 点发日报”“每 10 分钟扫一次某个来源”。

官方文档已经给了 CLI：

- `openclaw cron add`
    
- 可以 `--at` 一次性
    
- 可以 `--every` 固定间隔
    
- 可以 `--cron` 用 cron 表达式
    
- job 会持久化到 `~/.openclaw/cron/jobs.json`，重启后不会丢 ([OpenClaw](https://docs.openclaw.ai/automation/cron-jobs "Scheduled Tasks - OpenClaw"))
    

例如定时任务：

```bash
openclaw cron add \
  --name "Morning brief" \
  --cron "0 9 * * *" \
  --tz "Asia/Shanghai" \
  --session isolated \
  --message "检查今天需要处理的事项，生成简报并执行必要操作。" \
  --announce \
  --channel feishu \
  --to "你的目标ID"
```

关键点：

- `cron` 真正负责“到点就跑”
    
- `--announce --channel feishu --to ...` 负责把结果发回飞书
    
- `isolated` 适合后台任务，比较稳
    

---

### 方案 C：开 webhook，让外部系统叫醒它

如果你不是想“纯定时”，而是想：

- 收到某系统事件就跑
    
- 某个服务回调就跑
    
- 某个自动化平台触发就跑
    

那要开 hooks。官方文档里写了：

```json
{
  "hooks": {
    "enabled": true,
    "token": "shared-secret",
    "path": "/hooks"
  }
}
```

然后用带 token 的 HTTP 请求触发。([OpenClaw](https://docs.openclaw.ai/automation/cron-jobs "Scheduled Tasks - OpenClaw"))

这个模式适合：

- 飞书以外的系统触发
    
- 监控告警触发
    
- CI/CD、Gmail PubSub 等外部事件源触发 ([OpenClaw](https://docs.openclaw.ai/gateway/configuration-reference "Configuration Reference - OpenClaw"))
    

---

## Feishu 里你还要确认的事

### 1. 不要把“聊天触发模式”误当成“后台运行”

文档里不同 channel 有 chat mode 概念，典型包括：

- `oncall`：@它才回
    
- `onmessage`：每条都回
    
- `onchar`：特定前缀才回 ([OpenClaw](https://docs.openclaw.ai/gateway/configuration-reference "Configuration Reference - OpenClaw"))
    

这类配置只影响**消息来了怎么响应**，不等于“自己主动定时跑”。

---

### 2. Feishu 默认还可能有 mention / 群权限门槛

Feishu 配置里有：

- `requireMention`
    
- `groupPolicy`
    
- `groupAllowFrom`
    
- `dmPolicy` 等 ([OpenClaw](https://docs.openclaw.ai/llms-full.txt?utm_source=chatgpt.com "llms-full.txt"))
    

这些会影响：

- 群里不 @ 它时，它是不是根本不处理
    
- 哪些群、哪些人能触发它
    

但这仍然只是**被动触发权限**，不是后台自治。

---

### 3. Feishu 目前有一个“主动事件”例外：Drive 评论

如果你订阅了 `drive.notice.comment_add_v1`，用户在飞书文档/表格里加评论，也能触发 agent。  
这算“事件驱动主动触发”，但本质上仍然是**有事件才跑**，不是自己无限巡航。([OpenClaw](https://docs.openclaw.ai/channels/feishu "Feishu - OpenClaw"))

---

## 你大概率缺的不是配置细节，而是这两个前提

### 前提 1：Gateway 必须常驻

OpenClaw 的 cron 明确写了：  
**Cron runs inside the Gateway process**。也就是调度器就在 Gateway 里。Gateway 不在，cron 就不会 firing。([OpenClaw](https://docs.openclaw.ai/automation/cron-jobs "Scheduled Tasks - OpenClaw"))

所以如果你现在是：

- 手动开一下
    
- 只在你交互时进程才存在
    
- 没有 systemd / pm2 / docker restart policy
    

那它当然不会“背后运行”。

你需要把 Gateway 作为**常驻服务**跑起来。

---

### 前提 2：你要真的配置 heartbeat 或 cron

只接 Feishu，不会自动长出后台任务。  
要主动，就必须显式给它：

- `heartbeat`
    
- 或 `cron`
    
- 或 `hooks`
    

---

## 最实用的排查顺序

按这个顺序查，最快：

**第一步：确认 Gateway 是常驻的**

- 不是临时命令跑一下就退出
    
- 重启机器后它还能起来
    

**第二步：确认你有没有 heartbeat 配置**

- 看 `agents.defaults.heartbeat`
    
- 或 `agents.list[].heartbeat`
    
- 注意文档说：如果某些 agent 单独定义了 heartbeat，那么只会跑那些定义过的 agent。([OpenClaw](https://docs.openclaw.ai/gateway/configuration-reference "Configuration Reference - OpenClaw"))
    

**第三步：先人工建一个 cron 测试**  
比如 5 分钟后发一条到飞书：

```bash
openclaw cron add \
  --name "test-feishu" \
  --at "5m" \
  --session isolated \
  --message "这是后台任务测试，请回复当前时间并说明任务已执行。" \
  --announce \
  --channel feishu \
  --to "你的目标ID"
```

如果这都不触发，问题通常在：

- Gateway 没常驻
    
- Feishu 目标 ID 不对
    
- delivery routing 不对
    

**第四步：再上 heartbeat**  
因为 heartbeat 比 cron 更抽象，也更容易因为 prompt、session、投递目标等配置不清楚而看起来“没动”。

---

## 我对你的直接建议

**先别追求“完全自治 agent”。**  
先做一个最小闭环：

1. Gateway 常驻
    
2. 加一个 cron
    
3. 成功把结果投递到飞书
    
4. 再加 heartbeat
    
5. 最后再接外部事件
    

这是最短路径。  
你现在的问题，本质上不是 Feishu，而是 **“通道已经通了，但调度层没开”**。

如果你把你现在的 OpenClaw 配置文件里这几段贴出来：

- `channels.feishu`
    
- `agents.defaults`
    
- `agents.list`
    
- `hooks`
    

我可以直接帮你指出到底差哪一行。