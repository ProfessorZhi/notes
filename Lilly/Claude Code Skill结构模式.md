# Claude Code Skill 结构模式

## 观察时间
2026-04-10

## 来源
`/mnt/c/Users/Administrator/.codex/skills/`

观察了以下 skill 的 SKILL.md：
- `software-journey-orchestrator`
- `product-requirements-and-design`
- `technical-architecture-and-planning`
- `implementation-delivery`

## 通用结构（共七段）

### 1. Frontmatter
```yaml
---
name: <skill-name>
description: <一句话场景触发描述>
---
```
description 说明在什么情况下应该调用这个 skill，是路由触发的核心依据。

### 2. 概览（## 概览）
一句话说明这个 skill 的职责定位，通常带有类比或比喻（"总调度器"、"总管家"等）。

### 3. 核心职责（## 核心职责）
用列表列出这个 skill 负责的具体事项，通常 3-5 条。
不是"做什么"，而是"负责什么"——边界感清晰。

### 4. 推荐输出（## 推荐输出）
给出具体的推荐文件名，如：
- `goal-and-scope.md`
- `delivery-log.md`
- `verification-notes.md`
输出物是工程化的、有明确名字的，不是泛泛的"文档"。

### 5. 工作方式（## 工作方式）
分步骤说明具体怎么执行，通常 4-8 步。
强调：阶段感、最小验证、持续交付。

### 6. 什么时候停（## 什么时候停）
列出明确的退出条件，是"够了"而不是"做完了"。
这比"做完所有功能"更有工程意义。

### 7. 语言要求（可选，## 语言要求）
对于中文 skill，会明确说明哪些内容必须用中文，哪些保留原文。

## 特别模式：Orchestrator Skill

`software-journey-orchestrator` 是路由型 skill，它额外包含：

- **主阶段（## 主阶段）**：列出所有被路由的子 skill
- **路由原则（## 路由原则）**：如何决定去哪个子 skill
- **阶段判断规则（## 阶段判断规则）**：具体触发条件，包含"进入 XXX"的多个子标题

这是分阶段多skill协同的标准模式。

## 工程意义

这种结构对设计 agent skill 的启发：

1. **description 即路由契约**：写得要足够精确，能让上游判断是否应该路由过来
2. **职责边界清晰**：每个 skill 知道"不归我管什么"
3. **输出物具体化**：不只是"写文档"，而是具体文件名
4. **停止条件明确**：比"完成"更有操作意义
5. **分阶段协作**：orchestrator + stage skill 的分层是可行的多 agent 协作模式

## 对 Lilly 的参考价值

Lilly 设计自己的学习/执行 skill 时，可以参照这个七段结构，特别是：
- 用 description 做触发条件定义
- 用"什么时候停"替代模糊的"完成"目标
- 用推荐输出文件名明确交付预期
