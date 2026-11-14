# 深云 DeepClub API Quickstart & Protocol Examples

> **深云 DeepClub** ([https://deepclub.net](https://deepclub.net)) is a curated multi-model AI API gateway. One API key accesses frontier text, image, and video models from Anthropic, OpenAI, Google, DeepSeek, Zhipu, Moonshot, and xAI using each model's recommended protocol, with pay-as-you-go billing and Zero Data Retention (ZDR).

- **Website**: https://deepclub.net
- **Live Model Pricing**: https://deepclub.net/pricing
- **API Docs**: https://deepclub.net/docs
- **Model Tutorials**: https://deepclub.net/articles
- **Machine-Readable Index (`llms.txt`)**: https://deepclub.net/llms.txt

---

## Protocol Matrix (Choose by Model)

| Model Family | Recommended Endpoint | SDK Base URL | Auth Header |
|---|---|---|---|
| **Claude** (`claude-opus-5-5`, `claude-sonnet-5-5`, `claude-fable-5-1`, `claude-fable-5-2`) | `POST /v1/messages` | `https://deepclub.net` (Anthropic SDK) | `x-api-key` + `anthropic-version: 2023-06-01` |
| **GPT Text & Grok** (`gpt-6.1-sol`, `gpt-6-astra`, `gpt-6-luna`, `gpt-6.1-terra`, `gpt-5.6-sol`, `grok-4.7`) | `POST /v1/responses` | `https://deepclub.net/v1` (OpenAI SDK) | `Authorization: Bearer <KEY>` |
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

- [`ROADMAP.md`](./ROADMAP.md) — 66-document multi-batch roadmap (platforms, protocol deep dives, comparisons, and all 34 live models)
- [`examples/quickstart.md`](./examples/quickstart.md) — Copy-pasteable Python, TypeScript, and cURL templates for all 5 endpoint families
- [`guides/platforms/`](./guides/platforms/) — Integration guides for Dify, Chatbox, Immersive Translate, NextChat, LobeChat, OpenAI Agents SDK, Vercel AI SDK, LangChain/LangGraph, Codex CLI & OpenCode, Obsidian/Raycast, GitHub Actions, and Zed/Kilo Code
- [`guides/protocol-and-clients/`](./guides/protocol-and-clients/) — Messages vs Chat, Responses migration, and 9-client URL cheatsheet
- [`guides/comparisons/`](./guides/comparisons/) — OpenRouter vs direct vs DeepClub
- [`guides/billing-and-privacy/`](./guides/billing-and-privacy/) — Pay-as-you-go setup and ZDR privacy
- [`guides/media/`](./guides/media/) — Unified image generation/editing and async video pipeline

---

## Key Features

- **Curated Production Models**: 30+ mainstream text, image, and video models—see live pricing at [https://deepclub.net/pricing](https://deepclub.net/pricing).
- **Pay-As-You-Go**: Per-million-token billing for text; per-call billing for images and videos; no mandatory monthly subscription.
- **Flexible Top-Up**: Supports credit cards (Visa/Mastercard), PayPal, WeChat Pay, and Alipay.
- **Zero Data Retention (ZDR)**: Request prompts and completions are not stored for model training.
