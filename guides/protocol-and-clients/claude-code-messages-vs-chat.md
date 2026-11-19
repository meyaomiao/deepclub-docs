# Claude Code / Cline / Roo Code 接入第三方 API 的协议避坑指南：为什么不能全填 `/v1/chat/completions`？

很多开发者在给 **Claude Code**、**Cline**、**Roo Code** 或 **Cherry Studio** 配第三方 API 网关时，习惯性地把所有客户端的 Base URL 都填成 `https://xxx/v1` 然后一路走 OpenAI Chat Completions。

结果往往遇到各种诡异报错：`404 Not Found`、双 `/v1/v1/messages`、工具调用参数错乱、或者提示缓存（Prompt Caching）和长思考结构体丢失。

本文结合主流编码 Agent 的实际实现路径，梳理一套**不踩坑的多协议配置原则**（文中示例基于支持原生多协议转发的 **[深云 DeepClub](https://deepclub.net)**）。

---

## 一、先分清两种 Base URL：站根 vs `/v1`

这是 90% 的 `404` 报错根源。Anthropic SDK 与 OpenAI SDK 对 `base_url` 的拼接方式完全不同：

| SDK / 客户端类型 | 内部自动拼接路径 | Base URL 应该填什么（以深云 DeepClub 为例） |
|---|---|---|
| **Anthropic 官方 SDK / Claude Code** | `<base_url>/v1/messages` | 填站根：`https://deepclub.net`（**不带 `/v1`**） |
| **OpenAI SDK / OpenAI Compatible 客户端** | `<base_url>/chat/completions` 或 `<base_url>/responses` | 填：`https://deepclub.net/v1`（**带 `/v1`**） |
| **部分第三方 Claude 兼容插件（如 Cline 4.1.26 的新 AI SDK 路径）** | `<base_url>/messages` | 视插件代码拼接规则而定，若拼 `/messages` 则需要带 `/v1` |
| **沉浸式翻译 1.35.2 等直接写完整端点 URL 的工具** | 不自动拼接 | 填完整路径：`https://deepclub.net/v1/messages` 或 `/v1/responses` |

如果把 Anthropic SDK 的 `base_url` 误填成 `https://deepclub.net/v1`，最终发出去的请求会变成 `/v1/v1/messages`，必然 404。

---

## 二、为什么 Claude 系列强烈建议走原生 Messages，而不是 Chat 转换？

OpenAI Chat Completions 是围绕 `messages: [{role, content}]` + `choices[0].message.tool_calls` 设计的；而 Anthropic Messages API（`POST /v1/messages`）拥有独立的 `content` 块数组结构（`thinking`、`text`、`tool_use`、`tool_result`）以及 `cache_control` 提示缓存控制字段。

在编码 Agent（Claude Code、Cline、Roo Code）场景下：
1. 工具调用轮次极多，结构化 `tool_use` / `tool_result` 块比平铺转换更稳定；
2. 代码库上下文长，原生 Messages 协议配合 `cache_read` / `cache_write` 能显著降低长会话成本；
3. 如果只走 Chat Completions 转换层，部分版本客户端或高级特性可能无法与原生行为完全等价。

---

## 三、Claude Code 接入实操步骤

以在 **[深云 DeepClub](https://deepclub.net/integrations/claude-code)** 使用 `claude-sonnet-5-5` 或 `claude-opus-5-5` 为例（如需使用 `claude-fable-5-1` / `claude-fable-5-2`，需确保 Claude Code 版本分别在 `2.1.257+` / `2.1.321+`）：

1. 在控制台「API 令牌」页创建个人 Key（`sk-...`）。
2. 在终端设置环境变量（注意 `ANTHROPIC_BASE_URL` **不带 `/v1`**）：

```bash
export ANTHROPIC_BASE_URL="https://deepclub.net"
export ANTHROPIC_API_KEY="你的_DEEPCLUB_API_KEY"
```

3. 验证鉴权与最小请求（支持 `x-api-key` 与 `anthropic-version: 2023-06-01`）：

```bash
curl https://deepclub.net/v1/messages \
  -H "x-api-key: $DEEPCLUB_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "claude-sonnet-5-5",
    "max_tokens": 1024,
    "messages": [{"role": "user", "content": "用一句话解释什么是零留存 ZDR"}]
  }'
```

4. 在项目内运行 Claude Code，通过 `/models` 或配置指定目录中的精确模型 ID（如 `claude-sonnet-5-5`、`claude-opus-5-5`）。

---

## 四、如果客户端确实只支持 OpenAI Compatible 怎么办？

部分工具或旧版本插件仅支持填写 OpenAI Compatible 端点（例如某些版本的 Cursor 自定义模型入口、旧版 Chat 客户端）。这种情况下，可以选用在深云 DeepClub 目录中同时开放了 `/v1/chat/completions` 兼容入口的型号（可在 [https://deepclub.net/pricing](https://deepclub.net/pricing) 查看每个模型具体开放的端点列表），将 Base URL 设为 `https://deepclub.net/v1`，填入完整模型 ID 即可兼容运行。

**小结**：接入多模型 API 网关时，记住一句口诀——**Claude 优先走站根 + `/v1/messages`，GPT 文本与 Grok 优先走 `/v1/responses`，DeepSeek / Gemini 文本 / GLM / Kimi / Qwen 走 `/v1/chat/completions`**，就能避开绝大多数协议与路径坑。更多客户端的详细配置可参考 [Claude API 国内接入全指南](https://deepclub.net/compare/claude-api-guide)、[Claude Code 国内配置教程](https://deepclub.net/integrations/claude-code)（English: [Claude API Gateway](https://deepclub.net/en/compare/claude-api-proxy)）。
