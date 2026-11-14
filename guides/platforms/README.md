# 主流平台与开发框架接入指南（Batch 1）

这里汇总了 12 类主流 AI 客户端、笔记插件、IDE 编辑器与开发框架对接 **[深云 DeepClub](https://deepclub.net)**（[实时价格与端点目录](https://deepclub.net/pricing)）的精确配置与代码示例：

| 文档 | 适用对象 | 关键协议与地址提醒 |
|---|---|---|
| [01-dify.md](./01-dify.md) | Dify（Workflow / Agent） | 区分官方 OpenAI 插件 `1.1.0+`、Anthropic 插件 `0.3.41+` 与兼容插件 |
| [02-chatbox.md](./02-chatbox.md) | Chatbox (`1.24.3+`) | Responses / Claude / Chat 三模式精确 Host 与 Path |
| [03-immersive-translate.md](./03-immersive-translate.md) | 沉浸式翻译 (`1.35.2+`) | 按完整 URL 识别协议（`/v1/responses` / `/messages` / `/chat/completions`） |
| [04-nextchat.md](./04-nextchat.md) | NextChat | 接口地址填站根 `https://deepclub.net`，配合 `@Anthropic` / `@OpenAI` |
| [05-lobechat.md](./05-lobechat.md) | LobeChat | 按 Anthropic 与 OpenAI 兼容入口拆分，校验用精确模型 ID |
| [06-openai-agents-sdk.md](./06-openai-agents-sdk.md) | OpenAI Agents SDK (Py / TS) | `AsyncOpenAI` 注入、`store=False` 与 `OpenAIChatCompletionsModel` 混用 |
| [07-vercel-ai-sdk.md](./07-vercel-ai-sdk.md) | Vercel AI SDK (`ai`) | 注意 `@ai-sdk/anthropic` 需 `/v1`，`@ai-sdk/openai` 用 `.responses` / `.chat` |
| [08-langchain-langgraph.md](./08-langchain-langgraph.md) | LangChain & LangGraph | `ChatAnthropic(base_url=站根)` 与 `ChatOpenAI(use_responses_api=True)` |
| [09-codex-cli-and-opencode.md](./09-codex-cli-and-opencode.md) | Codex CLI / Codex++ / OpenCode | `~/.codex/config.toml` (`wire_api="responses"`) 与 `opencode.json` |
| [10-obsidian-copilot-and-raycast.md](./10-obsidian-copilot-and-raycast.md) | Obsidian Copilot & 桌面插件 | Custom OpenAI 写法、ZDR 零留存保护与低成本总结模型推荐 |
| [11-github-actions-pr-review.md](./11-github-actions-pr-review.md) | GitHub Actions PR Review Bot | CI 工作流 YAML、diff 截断与 Secret 权限隔离 |
| [12-zed-and-kilo-code.md](./12-zed-and-kilo-code.md) | Zed / Kilo Code / DeepSeek Harness | Zed `settings.json` 双协议与 `chat_completions: false` 切换 Responses |

更多客户端指南与协议原理见仓库根目录 [README.md](../../README.md) 与 [ROADMAP.md](../../ROADMAP.md)。
