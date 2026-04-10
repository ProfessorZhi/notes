# Claude Code 官方文档笔记：Subagent 系统

## 信息来源

- https://code.claude.com/docs/en/how-claude-code-works
- https://code.claude.com/docs/en/sub-agents

## 核心架构：Agentic Loop

```
用户 prompt → gather context → take action → verify results → 重复直到完成
```

Claude Code 是围绕 Claude 的 **agentic harness**：
- 提供工具、上下文管理、执行环境
- 模型负责推理，工具负责行动
- 用户可随时打断干预方向

## Subagent 隔离模型

**关键设计：subagent 有独立上下文窗口**

- 主会话和 subagent 的上下文完全隔离
- subagent 返回时只返回摘要，不撑爆主上下文
- subagent 不能 spawn 其他 subagent（防止无限嵌套）

**权限继承 + 限制**：
- 每个 subagent 继承父会话的权限，但可以有额外的工具限制
- 读 only subagent 可以 deny 掉 Write/Edit 工具，其余工具正常可用

## 上下文管理

**会话存储**：`~/.claude/projects/<project-id>/sessions/`

- 每次新会话从空白上下文开始
- 对话历史、工具使用、结果都写入 plaintext JSONL 文件
- 支持 resume / fork / rewind

**上下文填满时的处理**：
1. 先清除旧的工具输出
2. 需要时 summarizing 对话
3. 关键请求和代码片段优先保留
4. **持久规则写入 CLAUDE.md**，不依赖对话历史

**CLAUDE.md vs MEMORY.md**：
- CLAUDE.md：项目级持久规则和约定，每次会话加载
- Auto memory：Claude 自动保存的学习内容，前 200 行或 25KB 优先加载

## Subagent 类型

| Subagent | 模型策略 | 工具策略 | 用途 |
|----------|---------|---------|------|
| Explore | Haiku（外部）/ inherit（内部） | deny Write/Edit | 快速只读搜索 |
| Plan | inherit | deny Write/Edit | 规划前研究 |
| General Purpose | inherit | 全部工具 | 复杂多步任务 |
| Claude Code Guide | Haiku | ？ | 回答 Claude Code 功能问题 |

## Subagent 定义格式

Markdown + YAML frontmatter：

```yaml
---
identifier: code-reviewer
description: 代码审查 agent，扫描文件并提出改进建议
model: sonnet
tools:
  - read
  - grep
  - glob
  # 或者 "all" 表示全部工具
memory: user    # 可选：持久化记忆目录
---
# System prompt（可选，覆盖 frontmatter 中的 description）
```

## Subagent 存储位置（优先级从高到低）

| 位置 | 范围 | 创建方式 |
|------|------|---------|
| Managed settings | 组织级 | 部署配置 |
| `--agents` CLI flag | 当前会话 | 启动时传入 JSON |
| `.claude/agents/` | 当前项目 | 交互或手动 |
| `~/.claude/agents/` | 用户级 | 交互或手动 |

**同名称时，高优先级覆盖低优先级**

## Subagent 记忆（可选）

设置 `memory: user` 时，subagent 有持久化记忆目录：
- `~/.claude/agent-memory/<agent-identifier>/`
- subagent 跨会话积累知识（如代码模式、重复问题）

## 工程意义

1. **隔离是上下文管理的核心**：subagent 独立上下文窗口 → 主会话不会被研究过程撑爆 → 对话可以持续很久
2. **deny-list 而非 allow-list**：Explore/Plan 只 deny 4 个编辑工具，其余可用
3. **模型分层节省成本**：Haiku 用于快速搜索， Sonnet 用于复杂推理，按需升级
4. **记忆持久化**：subagent 级别的记忆目录让 agent 能跨会话积累知识
5. **CLI 可编程**：所有配置都可以通过 CLI 或文件管理，便于版本控制

## 与 OpenClaw 对比

| 维度 | Claude Code | OpenClaw |
|------|------------|---------|
| subagent 存储 | Markdown + YAML | workspace 文件 |
| 上下文隔离 | 强制隔离（核心设计） | session 隔离但消息传回 |
| 工具限制 | deny-list | 运行时配置 |
| 记忆持久化 | agent-memory dir | memory-builtin |
| 嵌套限制 | 禁止嵌套 | 无明确限制 |
