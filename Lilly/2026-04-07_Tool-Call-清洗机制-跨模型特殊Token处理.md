# Tool Call 清洗机制：跨模型特殊 Token 处理

## 背景

在多 Provider 环境中，模型输出的 tool call 格式存在差异，有些 Provider 会在 text content 中泄漏不应该暴露给用户的内容。OpenClaw 通过清洗函数处理这些问题。

## 关键清洗函数

### 1. `stripMinimaxToolCallXml`

**问题**：MiniMax 有时将 tool call 作为 XML 嵌入 text block，而不是正确的结构化 tool call 格式。

```typescript
// 清洗内容
<invoke name="...">...</invoke>  // XML 块
<minimax:tool_call>             // 标签泄漏
</minimax:tool_call>
```

**触发条件**：检测到 `minimax:tool_call` 标识才执行清洗。

### 2. `stripModelSpecialTokens`

**问题**：GLM-5 和 DeepSeek 等模型有时在响应中泄漏内部定界符 token，例如：
- `<|assistant|>`
- `<|tool_call_result_begin|>`
- `<｜begin▁of▁sentence｜>` （全角 pipe 变体）

**清洗规则**：匹配 ASCII `<|...|>` 和全角 `<｜...｜>` 两种变体。

**相关 Issue**：https://github.com/openclaw/openclaw/issues/40020

### 3. `stripDowngradedToolCallText`

**问题**：当向 Gemini 重放历史时，没有 `thought_signature` 的 tool call 会被降级为文本块。

**降级格式**：
```
[Tool Call: name (ID: ...)]
[Tool Result: ...]
[Historical context: ...]
```

## 工程意义

**Provider 差异是 LLM 应用的实际问题**，不能假设所有模型都按标准格式输出 tool call。需要在推理层或后处理层做兼容处理：

1. Provider 特定的 text 清洗
2. 结构化 vs 非结构化 tool call 的降级处理
3. 内部 token 的过滤

这对于在中国区部署多模型 Agent 系统有现实意义——不同模型的输出质量差异需要适配层来处理。
