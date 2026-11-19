# 深云 DeepClub API Quickstart & Protocol Examples

> **深云 DeepClub** ([https://deepclub.net](https://deepclub.net)) is a curated multi-model AI API gateway. One API key accesses frontier text, image, and video models from Anthropic, OpenAI, Google, DeepSeek, Zhipu, Moonshot, and xAI using each model's recommended protocol, with pay-as-you-go billing and Zero Data Retention (ZDR).

- **Website (中文)**: https://deepclub.net
- **English Portal**: https://deepclub.net/en
- **Live Model Pricing**: https://deepclub.net/pricing ([English Pricing](https://deepclub.net/en/pricing))
- **API Docs**: https://deepclub.net/docs
- **Model Tutorials & Guides**: https://deepclub.net/articles
- **Desktop & Mobile Clients**: https://deepclub.net/download/desktop · https://deepclub.net/download/mobile
- **Machine-Readable Index (`llms.txt`)**: https://deepclub.net/llms.txt

---

## Protocol Matrix (Choose by Model)

| Model Family | Recommended Endpoint | SDK Base URL | Auth Header |
|---|---|---|---|
| **Claude** (`claude-opus-5-5`, `claude-sonnet-5-5`, `claude-fable-5-1`, `claude-fable-5-2`) | `POST /v1/messages` | `https://deepclub.net` (Anthropic SDK) | `x-api-key` + `anthropic-version: 2023-06-01` |
| **GPT Text & Grok** (`gpt-6.2-sol`, `gpt-6.2-astra`, `gpt-6.1-sol`, `gpt-6-astra`, `gpt-6-luna`, `gpt-6.1-terra`, `gpt-5.6-sol`, `grok-4.7`) | `POST /v1/responses` | `https://deepclub.net/v1` (OpenAI SDK) | `Authorization: Bearer <KEY>` |
| **DeepSeek / Gemini Text / GLM / Kimi / Qwen** | `POST /v1/chat/completions` | `https://deepclub.net/v1` (OpenAI SDK) | `Authorization: Bearer <KEY>` |
| **Image Generation & Editing** (`gpt-image-2*`, `gemini-3-pro-image`, `gemini-3.1-flash-image`, `nano-banana-*`) | `POST /v1/images/generations`<br>`POST /v1/images/edits` | `https://deepclub.net/v1` (OpenAI SDK) | `Authorization: Bearer <KEY>` |
| **Video Generation** (`Seedance-2.0-Fast`, `Seedance-2.0-Mini`, `MiniMax-H3`) | `POST /v1/videos`<br>`GET /v1/videos/{task_id}` | `https://deepclub.net/v1` | `Authorization: Bearer <KEY>` |

> Compatibility note: Listed Claude, GPT text, and Grok models also expose `/v1/chat/completions` for clients that only support Chat Completions. GLM also exposes `/v1/responses`. Check [https://deepclub.net/pricing](https://deepclub.net/pricing) for live endpoints per model.

---

## Quickstart Snippets

### 1. Claude (Anthropic SDK — `https://deepclub.net`)

```python
import os
from anthropic import Anthropic

client = Anthropic(
    api_key=os.environ["DEEPCLUB_API_KEY"],
    base_url="https://deepclub.net",
)

message = client.messages.create(
    model="claude-sonnet-5-5",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Summarize zero data retention in one sentence."}],
)
print(message.content[0].text)
```

### 2. GPT-6 / Grok (OpenAI Responses API — `https://deepclub.net/v1`)

```python
import os
from openai import OpenAI

client = OpenAI(
    api_key=os.environ["DEEPCLUB_API_KEY"],
    base_url="https://deepclub.net/v1",
)

response = client.responses.create(
    model="gpt-6.1-sol",
    input="List 3 differences between Responses API and Chat Completions.",
    store=False,
)
print(response.output_text)
```

### 3. DeepSeek / Gemini / GLM / Kimi / Qwen (OpenAI Chat Completions)

```python
import os
from openai import OpenAI

client = OpenAI(
    api_key=os.environ["DEEPCLUB_API_KEY"],
    base_url="https://deepclub.net/v1",
)

completion = client.chat.completions.create(
    model="deepseek-v4.1-flash",
    messages=[{"role": "user", "content": "Write a minimal healthcheck in Go."}],
)
print(completion.choices[0].message.content)
```

More examples (streaming, image generation/editing, async video polling, and client guides for Cursor, Claude Code, Cline, Cherry Studio, Roo Code) are available in [`examples/quickstart.md`](./examples/quickstart.md), [the platform integration guides](./guides/platforms/), and [https://deepclub.net/docs](https://deepclub.net/docs).

## Repository Index

- [`ROADMAP.md`](./ROADMAP.md) — 66-document multi-batch roadmap (platforms, protocol deep dives, comparisons, and all 34 live models — **all batches complete**)
- [`examples/quickstart.md`](./examples/quickstart.md) — Copy-pasteable Python, TypeScript, and cURL templates for all 5 endpoint families
- [`guides/platforms/`](./guides/platforms/) — Batch 1: 12 integration guides (Dify, Chatbox, Immersive Translate, NextChat, LobeChat, OpenAI Agents SDK, Vercel AI SDK, LangChain/LangGraph, Codex CLI & OpenCode, Obsidian/Raycast, GitHub Actions, Zed/Kilo Code)
- [`guides/protocol-and-clients/`](./guides/protocol-and-clients/) — Batch 2: Messages vs Chat, Responses migration, Prompt Caching cost guide, Tool Calling & Structured Outputs, 401/404/429/SSE troubleshooting, and 9-client URL cheatsheet
- [`guides/billing-and-privacy/`](./guides/billing-and-privacy/) — Batch 2: Pay-as-you-go without overseas cards, ZDR security & sub-key governance, and billing reconciliation
- [`guides/media/`](./guides/media/) — Batch 2: Image generation/edits handbook and resumable async video polling
- [`guides/comparisons/`](./guides/comparisons/) — Batch 3: 11 comparison & selection guides (OpenRouter vs Direct vs DeepClub, 2026 gateway selection, coding/batch/reasoning/long-context/Chinese/lightweight/video/poster/UI models, and monthly full-stack budget playbook)
- [`models/`](./models/README.md) — Batch 4 & 5: All 34 live models (21 text + 13 image & video) with live pricing snapshots, multi-framework adapters, and runnable examples

---

## Key Features

- **Curated Production Models**: 36 live mainstream text, image, and video models—see live pricing at [https://deepclub.net/pricing](https://deepclub.net/pricing).
- **Pay-As-You-Go**: Per-million-token billing for text; per-call billing for images and videos; no mandatory monthly subscription.
- **Flexible Top-Up**: Supports credit cards (Visa/Mastercard), WeChat Pay, and Alipay (see the console top-up page for current options).
- **Zero Data Retention (ZDR)**: Request prompts and completions are not stored for model training.

---

## Topic Hubs & Guides on deepclub.net

- **Model Family Hubs (中文)**: [Claude API 国内接入与原生 Messages 指南](https://deepclub.net/compare/claude-api-guide) · [GPT / OpenAI Responses 接入指南](https://deepclub.net/compare/gpt-api-guide) · [DeepSeek & Gemini API 聚合指南](https://deepclub.net/compare/deepseek-gemini-api-guide) · [AI 生图与视频生成 API 汇总](https://deepclub.net/compare/ai-image-video-api-pricing)
- **Gateway, Billing & Troubleshooting (中文)**: [大模型 API 聚合网关选型](https://deepclub.net/compare/ai-api-gateway-guide) · [OpenRouter 国内替代方案](https://deepclub.net/compare/openrouter-alternative) · [免海外卡国内充值指南](https://deepclub.net/compare/claude-gpt-api-china-payment) · [Responses vs Chat 迁移](https://deepclub.net/compare/openai-responses-vs-chat-completions) · [Prompt Caching 省钱指南](https://deepclub.net/compare/prompt-caching-guide) · [401/404/429 报错排查](https://deepclub.net/integrations/api-troubleshooting)
- **Client Guides (中文)**: [Claude Code](https://deepclub.net/integrations/claude-code) · [Cursor](https://deepclub.net/integrations/cursor) · [Cline](https://deepclub.net/integrations/cline) · [Continue](https://deepclub.net/integrations/continue) · [Open WebUI](https://deepclub.net/integrations/open-webui) · [LobeChat & NextChat](https://deepclub.net/integrations/lobechat-nextchat) · [Dify](https://deepclub.net/integrations/dify)
- **English Guides**: [Unified AI API Gateway](https://deepclub.net/en) · [OpenRouter Alternative](https://deepclub.net/en/compare/openrouter-alternative) · [Claude API Proxy](https://deepclub.net/en/compare/claude-api-proxy) · [GPT & Responses Gateway](https://deepclub.net/en/compare/gpt-api-gateway) · [DeepSeek & Gemini API](https://deepclub.net/en/compare/deepseek-gemini-api) · [AI Video & Image API](https://deepclub.net/en/compare/ai-video-image-api) · [Prompt Caching Guide](https://deepclub.net/en/compare/prompt-caching-llm-api)
