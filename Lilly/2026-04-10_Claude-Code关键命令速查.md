# Claude Code 关键命令与 Status Line

## Status Line

可自定义状态栏，位于 Claude Code 底部，运行任意 shell 脚本：
- 脚本接收 JSON session 数据在 stdin
- 脚本输出的内容显示在状态栏

典型用途：
- 上下文窗口使用率（最重要的监控指标）
- session 成本追踪
- git branch 和状态
- 多 session 区分

设置方式：
1. /statusline 自然语言命令：Claude Code 自动生成脚本
2. 手动配置：在 settings.json 添加 statusLine 字段

## 关键命令速查

| 命令 | 用途 |
|------|------|
| /model <alias> | 切换模型 |
| /permissions | 管理权限规则 |
| /compact [focus] | 手动压缩上下文，可指定焦点 |
| /statusline | 配置状态栏 |
| /agents | 管理 subagent 配置 |
| /hooks | 查看 hook 配置 |
| /init | 生成起始 CLAUDE.md |
| /cost | 查看 token 使用统计 |

## Compact 命令

/compact 是手动触发上下文压缩的命令：
- 可带焦点参数：/compact focus on API changes
- 保留最近访问文件、活跃 plan
- 重置工作预算到 50K token

## 工程意义

1. **Status Line 是生产级会话管理的必备工具**：上下文使用率实时可见，不会等到快满了才发现
2. **/compact focus 是人工干预压缩质量的手段**：可以指定压缩焦点，让 Claude 保留最相关的内容
3. **监控成本是运营的基本要求**：多 session 并行时，成本追踪是必须有的
