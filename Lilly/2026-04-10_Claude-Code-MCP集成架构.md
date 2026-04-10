# Claude Code MCP 集成架构

## 信息来源
- https://code.claude.com/docs/en/mcp

## MCP 是什么

Model Context Protocol：AI 与外部工具集成的开放标准。
MCP server = Claude Code 连接到工具/数据库/API 的桥梁。

## MCP 的典型用途

- 从 issue tracker 读取任务并创建 PR
- 查询数据库（PostgreSQL 等）
- 集成 Figma 设计
- 自动化邮件工作流
- 作为 channel 接收外部事件（webhook/Telegram/Discord）

## 三种传输方式

| 传输方式 | 场景 | 状态 |
|---------|------|------|
| HTTP | 云服务，远程 MCP server | 推荐 |
| SSE | 远程 MCP server | 已废弃 |
| stdio | npm 包，本地运行 | 可用 |

配置示例：
claude mcp add --transport http notion https://mcp.notion.com/mcp
claude mcp add --transport http secure-api https://api.example.com/mcp --header "Authorization: Bearer your-token"

## 安全风险（重要）

文档明确警告：
1. 第三方 MCP server 未经过 Anthropic 验证
2. 可能暴露 prompt injection 风险
3. fetch 不受信内容的 MCP server 风险最高

这与中国区安全要求一致：引入外部 MCP 需要自己评估风险。

## 工具 schema 延迟加载

MCP tool schema 默认延迟加载：
- 只在需要时通过 tool search 加载
- 启动时只占 120 token（只有名称）
- 可配置 ENABLE_TOOL_SEARCH=auto 在上下文 10% 以内时全部加载

## MCP 作为 Channel

MCP server 可以作为 channel 向 session 推送消息：
- Claude 响应 Telegram 消息
- 响应 Discord 聊天
- 响应 webhook 事件
- 这意味着 MCP server 不只是工具，还可以是事件源

## MCP Registry

Anthropic 维护 MCP registry（api.anthropic.com/mcp-registry），
提供经过验证的 MCP server 列表，按平台过滤。

## 与 OpenClaw MCP 对比

| 维度 | Claude Code | OpenClaw |
|------|------------|---------|
| MCP 用途 | 外部工具集成 | 外部工具集成 |
| channel 推送 | 支持（MCP server 推送事件） | 需要 channel 插件 |
| schema 加载 | 延迟加载（默认） | 未明确 |
| 安全评估 | 文档明确警告用户自评 | 类似 |
