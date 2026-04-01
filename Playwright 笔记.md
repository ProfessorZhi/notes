## 1. 定位

**Playwright 不是单一概念。**  
在实际语境里，`Playwright` 可能同时指：

- 一个 **浏览器自动化库**：直接用代码启动浏览器、开页面、点击、输入、截图。
    
- 一个 **端到端测试框架**：官方叫 **Playwright Test**，提供 test runner、断言、隔离、并行、报告等能力。
    
- 一套 **命令行工具**：`npx playwright ...`
    
- 一套 **给 AI/Agent 用的 MCP 服务**：`@playwright/mcp`
    

官方文档把 **Playwright Library** 和 **Playwright Test** 明确区分开：前者是统一的浏览器控制 API，后者是在其之上的完整测试框架。对 E2E 测试，官方通常更推荐 `@playwright/test`，而不是直接只用 `playwright`。([Playwright](https://playwright.dev/docs/next/library?utm_source=chatgpt.com "Library | Playwright"))

---

## 2. 可以把它理解成什么

### Playwright Library

代码层面的浏览器自动化库。  
直接提供 `chromium.launch()`、`browser.newContext()`、`page.goto()`、`page.click()`、`page.screenshot()` 这类 API。([Playwright](https://playwright.dev/docs/next/library?utm_source=chatgpt.com "Library | Playwright"))

### Playwright Test

建立在 Library 之上的测试框架。  
除了浏览器控制，还附带：

- test runner
    
- `expect`
    
- fixture
    
- trace / report
    
- 并行与隔离
    

也就是：  
**Library = 只负责“控浏览器”**  
**Test = 把“控浏览器”包装成完整测试系统**。([Playwright](https://playwright.dev/docs/next/library?utm_source=chatgpt.com "Library | Playwright"))

### Playwright CLI

命令行入口。  
典型命令：

- `npx playwright --version`
    
- `npx playwright install`
    
- `npx playwright test`
    
- `npx playwright show-trace`
    

CLI 本质上是 Playwright 的命令入口，不是另一个独立引擎。([Playwright](https://playwright.dev/docs/test-cli?utm_source=chatgpt.com "Command line | Playwright"))

### Playwright MCP

给 AI 客户端暴露浏览器自动化能力的 **MCP server**。  
它通过 Model Context Protocol 把“打开网页、读取页面结构、点击、输入、截图、切换标签页”等能力作为工具暴露给 VS Code、Cursor、Claude Desktop、Codex 一类 MCP 客户端。官方文档把它定义为“通过 MCP 提供浏览器自动化能力”的 server。([Playwright](https://playwright.dev/docs/next/getting-started-mcp?utm_source=chatgpt.com "Playwright MCP | Playwright"))

---

## 3. 一句话区分

- **`playwright`**：库
    
- **`@playwright/test`**：测试框架
    
- **`npx playwright ...`**：CLI 入口
    
- **`@playwright/mcp`**：MCP server 包
    

所以：

**Playwright 不是 MCP。**  
MCP 只是 Playwright 生态里的一种使用形式。([Playwright](https://playwright.dev/docs/next/library?utm_source=chatgpt.com "Library | Playwright"))

---

## 4. CLI 形式 vs MCP 形式

## CLI 形式

更像“人或脚本直接下命令”。

特点：

- 一次性执行很顺手
    
- 适合装浏览器、跑测试、看 trace、截图
    
- 调用路径短
    
- 不强调长会话
    

官方文档里，CLI 被定义为运行测试、生成代码、调试、安装浏览器等的命令行接口；Playwright 还专门提供了面向 coding agents 的 `playwright-cli` 方案，并明确对比了 CLI 与 MCP 的适用场景。CLI 更偏 token-efficient、命令式工作流。([Playwright](https://playwright.dev/docs/test-cli?utm_source=chatgpt.com "Command line | Playwright"))

## MCP 形式

更像“AI 通过协议调浏览器”。

特点：

- 适合 Agent 连续操作网页
    
- 更适合多轮、持续、带状态的自动化
    
- 有长期存在的 browser / context / page 概念
    
- 会出现“会话死掉但环境没坏”的问题
    

官方文档把 MCP 描述成：通过结构化 accessibility snapshots 让 LLM 与网页交互，不要求视觉模型；并明确说它适合与 MCP client 一起工作。([Playwright](https://playwright.dev/docs/next/getting-started-mcp?utm_source=chatgpt.com "Playwright MCP | Playwright"))

---

## 5. Chromium 和 Playwright 的关系

**Chromium 不是 Playwright 本身。**  
Chromium 是 **被 Playwright 控制的浏览器二进制** 之一。

Playwright 官方支持控制：

- Chromium
    
- WebKit
    
- Firefox
    

也支持 branded browsers，比如 Chrome、Edge。([Playwright](https://playwright.dev/docs/browsers?utm_source=chatgpt.com "Browsers | Playwright"))

所以关系不是：

> Playwright = Chromium

而是：

> Playwright 控制 Chromium / Firefox / WebKit

---

## 6. “Playwright 里面可以装 Chromium” 这句话怎么理解

更准确的说法是：

**Playwright 会下载并管理它需要的浏览器二进制。**

官方文档写得很明确：

- 每个版本的 Playwright 都需要匹配的 browser binaries
    
- 需要通过 Playwright CLI 去安装这些浏览器
    
- 升级 Playwright 后，往往还要重新执行安装命令
    

典型命令：

```bash
npx playwright install
npx playwright install chromium
```

这说明它不是“只 npm 安装一个 JS 包，然后自动依赖系统里现成浏览器”这么简单；而是 **Playwright 自己会下载并维护一套匹配版本的浏览器二进制**。([Playwright](https://playwright.dev/docs/browsers?utm_source=chatgpt.com "Browsers | Playwright"))

---

## 7. npm 安装和浏览器安装，不是一回事

### 第一层：安装 Node 包

这是装代码、装 API、装 CLI。

例如：

```bash
npm install -D @playwright/test
npm install playwright
npm install @playwright/mcp
```

### 第二层：安装浏览器二进制

这是装真正会被启动的浏览器内核。

例如：

```bash
npx playwright install
npx playwright install chromium
```

官方安装文档明确写到，初始化 Playwright 时会“download required browser binaries”；浏览器安装文档也明确写到，浏览器二进制需要通过 CLI 安装。([Playwright](https://playwright.dev/docs/intro?utm_source=chatgpt.com "Installation | Playwright"))

---

## 8. 所以它到底是“依赖 Chromium”还是“安装 Chromium”

两句话同时成立，但语义不同：

### 从运行原理上说

Playwright **依赖浏览器二进制** 才能真正控制浏览器。  
没有浏览器二进制，很多操作跑不起来。([Playwright](https://playwright.dev/docs/browsers?utm_source=chatgpt.com "Browsers | Playwright"))

### 从安装行为上说

Playwright **可以主动下载并安装** 这些浏览器二进制。  
不是单纯指望系统提前装好。([Playwright](https://playwright.dev/docs/browsers?utm_source=chatgpt.com "Browsers | Playwright"))

所以最准确的笔记表达是：

> Playwright 依赖浏览器二进制运行；官方 CLI 负责把这些二进制下载并安装到本地。

---

## 9. MCP 和 Chromium 的关系

MCP 不直接等于浏览器。  
MCP 是 **协议层 / 服务层**，底下还是 Playwright 在控浏览器。

结构可以这样记：

```text
AI / 编辑器 / Agent
        ↓
   Playwright MCP
        ↓
    Playwright
        ↓
Chromium / Firefox / WebKit
```

官方 MCP 文档说明它是 “browser automation capabilities through the Model Context Protocol”；npm 页也说明它底层是 using Playwright。([Playwright](https://playwright.dev/docs/next/getting-started-mcp?utm_source=chatgpt.com "Playwright MCP | Playwright"))

---

## 10. CLI 和 MCP 会不会共用同一个浏览器

**可能共用同一类浏览器内核，但不一定共用同一个会话。**

要区分两层：

### 同一类浏览器

CLI 和 MCP 都可能使用 Chromium。  
这一层可以相同。([Playwright](https://playwright.dev/docs/next/getting-started-mcp?utm_source=chatgpt.com "Playwright MCP | Playwright"))

### 同一个会话

不一定。  
CLI 常常是临时起一个浏览器进程，做完就退出。  
MCP 更偏持续会话，强调 profile、persistent / isolated 模式。官方 MCP 文档也明确列了 persistent 与 isolated 两类 profile。([Playwright](https://playwright.dev/docs/next/getting-started-mcp?utm_source=chatgpt.com "Playwright MCP | Playwright"))

所以：

> 同一个 Chromium，不等于同一个 browser context / page / session。

---

## 11. 这类命令分别在干什么

### 只检查 CLI 在不在

```bash
npx playwright --version
```

检查命令行工具是否可调用。([Playwright](https://playwright.dev/docs/browsers?utm_source=chatgpt.com "Browsers | Playwright"))

### 安装 Playwright 需要的浏览器

```bash
npx playwright install
npx playwright install chromium
```

下载 Playwright 对应版本需要的浏览器二进制。([Playwright](https://playwright.dev/docs/browsers?utm_source=chatgpt.com "Browsers | Playwright"))

### 跑测试

```bash
npx playwright test
```

调用 Playwright Test 运行测试。([Playwright](https://playwright.dev/docs/intro?utm_source=chatgpt.com "Installation | Playwright"))

### 启动 MCP

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["@playwright/mcp@latest"]
    }
  }
}
```

这是让 MCP client 通过 `npx` 启动 Playwright MCP server。([Playwright](https://playwright.dev/docs/next/getting-started-mcp?utm_source=chatgpt.com "Playwright MCP | Playwright"))

---

## 12. 最终脑图

```text
Playwright
├─ Library（代码 API）
├─ Test（测试框架，包名常见为 @playwright/test）
├─ CLI（npx playwright ...）
└─ MCP（@playwright/mcp，给 AI/Agent 用）
```

```text
Playwright 本体 ≠ Chromium
Playwright 是控制层
Chromium / Firefox / WebKit 是被控制的浏览器
```

```text
npm install 某个包
    ≠
浏览器二进制已经装好

Node 包负责“代码与命令”
playwright install 负责“浏览器内核”
```

---

## 13. 记忆版结论

- Playwright 首先是 **浏览器自动化生态**，不是单一包名。
    
- `playwright` 偏 **库**。
    
- `@playwright/test` 偏 **测试框架**。
    
- `npx playwright` 是 **CLI 入口**。
    
- `@playwright/mcp` 是 **MCP server**，不是 Playwright 的同义词。([Playwright](https://playwright.dev/docs/next/library?utm_source=chatgpt.com "Library | Playwright"))
    
- Chromium 不是被 npm“顺手依赖一下”就完事；Playwright 官方会要求并提供 **专门的浏览器二进制安装**。([Playwright](https://playwright.dev/docs/browsers?utm_source=chatgpt.com "Browsers | Playwright"))
    
- CLI 和 MCP 都能用 Chromium，但它们不必共享同一个浏览器会话。([Playwright](https://playwright.dev/docs/next/getting-started-mcp?utm_source=chatgpt.com "Playwright MCP | Playwright"))
    

如果你要，我可以下一条把这份笔记再整理成更适合 Obsidian 的版本。