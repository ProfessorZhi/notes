# Claude Code 工具并发执行模式

## 核心发现

Claude Code 的工具执行使用 **batch + concurrency-safe partition** 模式：

1. **并发分区**：所有工具调用按 `isConcurrencySafe` 分组
   - `true` → 同一批次内并行执行
   - `false` → 必须串行执行

2. **执行策略**（`toolOrchestration.ts`）：
   - `runToolsConcurrently`：使用 `all()` 生成器并行跑多个工具
   - `runToolsSerially`：逐个执行，中间结果累积到 context

3. **`isConcurrencySafe` 设计**：
   - 每个工具自行判断（方法，接收 parsed input）
   - 默认返回 `false`（保守）
   - 读工具（FileRead/Glob/Grep/WebSearch/WebFetch）→ `true`
   - 写工具通常 `false`，但 TaskUpdateTool 例外（也是 `true`）

4. **工程意义**：
   - 读操作天然可并行（如同时读多个文件）
   - 写操作需保证顺序以避免竞态
   - 并发安全由工具自己声明，不依赖中央判断

## 对多 Agent 系统的参考价值

- 设计工具时，显式声明并发安全性比隐式串行更灵活
- 批量并发 + 串行写门的组合是实用的默认策略
- Agent Supervisor 拦截工具调用时可以参考 `isConcurrencySafe` 分区思路

## 来源

`/mnt/f/agent_project/codingagent/claudecodes/claudecodets/claudecodets/src/services/tools/toolOrchestration.ts`
