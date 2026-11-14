# 主流 AI 客户端接入深云 DeepClub 协议与 Base URL 速查表（防 404 / 双 `/v1` 踩坑）

很多客户端虽然都叫“自定义 API”，但不同客户端（甚至同一客户端不同版本）拼接 URL 的方式完全不一样。

下面整理一份基于 **[深云 DeepClub](https://deepclub.net)**（[定价与开放端点表](https://deepclub.net/pricing)）的常用客户端精确配置速查，核对过源码/发布包逻辑，直接抄作业即可：

---

## 一、9 款常用工具精确填法

| 客户端 | 选用的模式 / Provider | Base URL / Host 怎么填 | 建议模型示例 | 注意事项 |
|---|---|---|---|---|
| **Claude Code** | Anthropic 原生 | `ANTHROPIC_BASE_URL=https://deepclub.net`（**不带 `/v1`**） | `claude-sonnet-5-5`、`claude-opus-5-5`、`claude-fable-5-2` | Fable 5.1 需 2.1.257+，Fable 5.2 需 2.1.321+；Auth 可用 `ANTHROPIC_API_KEY` |
| **Cursor** | OpenAI API Key + Override Base URL | `https://deepclub.net/v1` | `claude-sonnet-5-5`、`gpt-6.1-sol`、`qwen3.8-flash` | 需在 Models 里手动 Add model 填精确 ID 后 Verify；Tab 补全走 Cursor 自家模型不走网关 |
| **Cline (4.1.26+)** | 1) OpenAI Compatible<br>2) Anthropic（仅当选择器支持自定义精确型号时） | 1) `https://deepclub.net/v1`<br>2) 新 AI SDK 路径填 `https://deepclub.net/v1`（它会拼 `/messages`） | `claude-sonnet-5-5`、`deepseek-v4.1-flash` | 常规 OpenAI UI 无通用自定义 Responses 开关，走 OpenAI Compatible 即为 Chat |
| **Roo Code (3.54.0+)** | 1) OpenAI Compatible<br>2) Anthropic 原生 | 1) `https://deepclub.net/v1`<br>2) `https://deepclub.net`（站根） | `claude-sonnet-5-5`、`gpt-6.1-sol` | 原生 Anthropic 若列表回落默认型号需核对精确 ID；原生 OpenAI 备用 fetch 有双 `/v1` 风险，更推荐 OpenAI Compatible |
| **Cherry Studio (2.2.0+)** | 自定义服务（建议按协议拆分） | `https://deepclub.net/v1` | Claude 选 Messages；GPT/Grok 选 Responses；DeepSeek/Gemini/GLM 选 Chat | 强烈建议把 Responses 和 Chat 模型拆开配置或逐模型指定端点，避免多端点优先级选错 |
| **Chatbox (1.24.3+)** | 分别选：<br>`OpenAI Responses API Compatible` / `Claude API Compatible` / `OpenAI API Compatible` | Host：`https://deepclub.net/v1`<br>Responses 路径：`/responses`<br>Chat 路径：`/chat/completions` | `gpt-6.1-sol`、`claude-sonnet-5-5`、`gemini-4-hydrogen` | 第三方 Claude Host 注意不要漏掉 `/v1` |
| **沉浸式翻译 (1.35.2+)** | 自定义 API（按完整 URL 识别协议） | Responses：`https://deepclub.net/v1/responses`<br>Claude：`https://deepclub.net/v1/messages`<br>Chat：`https://deepclub.net/v1/chat/completions` | `deepseek-v4.1-flash`、`gpt-6-luna`、`gemini-4-hydrogen` | **必须填完整端点 URL**，不要只改协议下拉框而留着旧 path |
| **NextChat** | `@Anthropic` 或 `@OpenAI` | 站根：`https://deepclub.net` | `@Anthropic` 走 Messages；`@OpenAI` 走 Chat | NextChat 对 OpenAI 默认走 Chat，GPT 模型需按兼容方式使用 |
| **Dify 官方插件** | OpenAI 插件 (`1.1.0+`) / Anthropic 插件 (`0.3.41+`) | OpenAI 插件：`https://deepclub.net/v1`（可选 `API Protocol=Responses`）<br>Anthropic 插件：`https://deepclub.net` | `gpt-6.1-sol`、`claude-sonnet-5-5`、`glm-5.3` | 注意区分“OpenAI 插件版本”与“Dify 主程序版本”；`openai_api_compatible` 插件校验凭据时仍可能发 Chat |

---

## 二、3 条最常见的 404 / 401 排查法

1. **报 `404 Not Found`**：90% 是 `/v1` 多填了或漏填了。打开控制台前先确认客户端源码到底往你填的 URL 后面拼了 `/v1/messages`、`/messages` 还是 `/chat/completions`。
2. **报 `401 Unauthorized`**：Messages 协议用 `x-api-key` + `anthropic-version: 2023-06-01`（或网关兼容的 Bearer），Responses/Chat/图片/视频用 `Authorization: Bearer <KEY>`；同时检查 Key 是否前后多复制了空格。
3. **模型不存在**：必须填写 [深云 DeepClub 定价表](https://deepclub.net/pricing) 里的**完整精确 ID**（例如 `gpt-6.1-sol`、`deepseek-v4.1-flash`、`gemini-4-hydrogen`），不要自造别名。

全套客户端分页指南与常见问题见：[https://deepclub.net/docs](https://deepclub.net/docs) 与 [https://deepclub.net/articles](https://deepclub.net/articles)。
