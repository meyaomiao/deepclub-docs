# 深云 DeepClub 34 款在售模型单独指南与多平台适配（Batch 4 & Batch 5）

> 涵盖当前 [深云 DeepClub](https://deepclub.net) 全量在售 **21 款文本模型** + **13 款生图与视频模型**，每篇均包含官网定价快照、适用边界、多框架/多客户端参数对照与可复制调用代码。

## 一、21 款文本模型（`./text/`）

| 模型 ID（点击查看独立文档） | 开放端点 | 官网直达 |
|---|---|---|
| [`gpt-6.1-terra`](./text/gpt-6.1-terra.md) | `/v1/chat/completions`、`/v1/responses` | [教程](https://deepclub.net/articles/gpt-6-1-terra-how-to-use-guide) · [定价](https://deepclub.net/pricing/gpt-6.1-terra) |
| [`claude-fable-5-2`](./text/claude-fable-5-2.md) | `/v1/messages`、`/v1/chat/completions` | [教程](https://deepclub.net/articles/claude-fable-5-2-how-to-use-guide) · [定价](https://deepclub.net/pricing/claude-fable-5-2) |
| [`gemini-4-carbon`](./text/gemini-4-carbon.md) | `/v1/chat/completions` | [教程](https://deepclub.net/articles/gemini-4-carbon-how-to-use-guide) · [定价](https://deepclub.net/pricing/gemini-4-carbon) |
| [`gemini-4-hydrogen`](./text/gemini-4-hydrogen.md) | `/v1/chat/completions` | [教程](https://deepclub.net/articles/gemini-4-hydrogen-how-to-use-guide) · [定价](https://deepclub.net/pricing/gemini-4-hydrogen) |
| [`claude-opus-5-5`](./text/claude-opus-5-5.md) | `/v1/messages`、`/v1/chat/completions` | [教程](https://deepclub.net/articles/claude-opus-5-5-how-to-use-guide) · [定价](https://deepclub.net/pricing/claude-opus-5-5) |
| [`claude-sonnet-5-5`](./text/claude-sonnet-5-5.md) | `/v1/messages`、`/v1/chat/completions` | [教程](https://deepclub.net/articles/claude-sonnet-5-5-how-to-use-guide) · [定价](https://deepclub.net/pricing/claude-sonnet-5-5) |
| [`claude-fable-5-1`](./text/claude-fable-5-1.md) | `/v1/messages`、`/v1/chat/completions` | [教程](https://deepclub.net/articles/claude-fable-5-1-how-to-use-guide) · [定价](https://deepclub.net/pricing/claude-fable-5-1) |
| [`deepseek-v4.1-flash`](./text/deepseek-v4.1-flash.md) | `/v1/chat/completions` | [教程](https://deepclub.net/articles/deepseek-v4-1-flash-how-to-use-guide) · [定价](https://deepclub.net/pricing/deepseek-v4.1-flash) |
| [`deepseek-v4.1-flash-1017`](./text/deepseek-v4.1-flash-1017.md) | `/v1/chat/completions` | [教程](https://deepclub.net/articles/deepseek-v4-1-flash-1017-how-to-use-guide) · [定价](https://deepclub.net/pricing/deepseek-v4.1-flash-1017) |
| [`gpt-6.1-sol`](./text/gpt-6.1-sol.md) | `/v1/chat/completions`、`/v1/responses` | [教程](https://deepclub.net/articles/gpt-6-1-sol-how-to-use-guide) · [定价](https://deepclub.net/pricing/gpt-6.1-sol) |
| [`gpt-6-astra`](./text/gpt-6-astra.md) | `/v1/chat/completions`、`/v1/responses` | [教程](https://deepclub.net/articles/gpt-6-astra-how-to-use-guide) · [定价](https://deepclub.net/pricing/gpt-6-astra) |
| [`gpt-6-luna`](./text/gpt-6-luna.md) | `/v1/chat/completions`、`/v1/responses` | [教程](https://deepclub.net/articles/gpt-6-luna-how-to-use-guide) · [定价](https://deepclub.net/pricing/gpt-6-luna) |
| [`gpt-5.6-sol`](./text/gpt-5.6-sol.md) | `/v1/chat/completions`、`/v1/responses` | [教程](https://deepclub.net/articles/gpt-5.6-sol-how-to-use-guide) · [定价](https://deepclub.net/pricing/gpt-5.6-sol) |
| [`gemini-4-argon`](./text/gemini-4-argon.md) | `/v1/chat/completions` | [教程](https://deepclub.net/articles/gemini-4-argon-how-to-use-guide) · [定价](https://deepclub.net/pricing/gemini-4-argon) |
| [`gemini-3.8-flash-high`](./text/gemini-3.8-flash-high.md) | `/v1/chat/completions` | [教程](https://deepclub.net/articles/gemini-3.8-flash-high-how-to-use-guide) · [定价](https://deepclub.net/pricing/gemini-3.8-flash-high) |
| [`gemini-3.8-flash`](./text/gemini-3.8-flash.md) | `/v1/chat/completions` | [教程](https://deepclub.net/articles/gemini-3.8-flash-how-to-use-guide) · [定价](https://deepclub.net/pricing/gemini-3.8-flash) |
| [`glm-5.3`](./text/glm-5.3.md) | `/v1/chat/completions`、`/v1/responses` | [教程](https://deepclub.net/articles/glm-5-3-how-to-use-guide) · [定价](https://deepclub.net/pricing/glm-5.3) |
| [`glm-5.3-flash`](./text/glm-5.3-flash.md) | `/v1/chat/completions`、`/v1/responses` | [教程](https://deepclub.net/articles/glm-5-3-flash-how-to-use-guide) · [定价](https://deepclub.net/pricing/glm-5.3-flash) |
| [`kimi-k3`](./text/kimi-k3.md) | `/v1/chat/completions` | [教程](https://deepclub.net/articles/kimi-k3-how-to-use-guide) · [定价](https://deepclub.net/pricing/kimi-k3) |
| [`qwen3.8-flash`](./text/qwen3.8-flash.md) | `/v1/chat/completions` | [教程](https://deepclub.net/articles/qwen3-8-flash-how-to-use-guide) · [定价](https://deepclub.net/pricing/qwen3.8-flash) |
| [`grok-4.7`](./text/grok-4.7.md) | `/v1/chat/completions`、`/v1/responses`、`/v1/messages` | [教程](https://deepclub.net/articles/grok-4-7-how-to-use-guide) · [定价](https://deepclub.net/pricing/grok-4.7) |

## 二、13 款生图与视频模型（`./media/`）

| 模型 ID（点击查看独立文档） | 开放端点 | 官网直达 |
|---|---|---|
| [`Seedance-2.0-Fast`](./media/Seedance-2.0-Fast.md) | `/v1/videos` | [教程](https://deepclub.net/articles/seedance-2-0-fast-how-to-use-guide) · [定价](https://deepclub.net/pricing/Seedance-2.0-Fast) |
| [`Seedance-2.0-Mini`](./media/Seedance-2.0-Mini.md) | `/v1/videos` | [教程](https://deepclub.net/articles/seedance-2-0-mini-how-to-use-guide) · [定价](https://deepclub.net/pricing/Seedance-2.0-Mini) |
| [`MiniMax-H3`](./media/MiniMax-H3.md) | `/v1/videos` | [教程](https://deepclub.net/articles/minimax-h3-how-to-use-guide) · [定价](https://deepclub.net/pricing/MiniMax-H3) |
| [`gemini-3-pro-image`](./media/gemini-3-pro-image.md) | `/v1/images/generations`、`/v1/images/edits` | [教程](https://deepclub.net/articles/gemini-3-pro-image-how-to-use-guide) · [定价](https://deepclub.net/pricing/gemini-3-pro-image) |
| [`gemini-3.1-flash-image`](./media/gemini-3.1-flash-image.md) | `/v1/images/generations`、`/v1/images/edits` | [教程](https://deepclub.net/articles/gemini-3-1-flash-image-how-to-use-guide) · [定价](https://deepclub.net/pricing/gemini-3.1-flash-image) |
| [`gpt-image-2`](./media/gpt-image-2.md) | `/v1/images/generations`、`/v1/images/edits` | [教程](https://deepclub.net/articles/gpt-image-2-how-to-use-guide) · [定价](https://deepclub.net/pricing/gpt-image-2) |
| [`gpt-image-2-adobe`](./media/gpt-image-2-adobe.md) | `/v1/images/generations`、`/v1/images/edits` | [教程](https://deepclub.net/articles/gpt-image-2-adobe-how-to-use-guide) · [定价](https://deepclub.net/pricing/gpt-image-2-adobe) |
| [`gpt-image-2.5-flare`](./media/gpt-image-2.5-flare.md) | `/v1/images/generations`、`/v1/images/edits` | [教程](https://deepclub.net/articles/gpt-image-2-5-flare-how-to-use-guide) · [定价](https://deepclub.net/pricing/gpt-image-2.5-flare) |
| [`gpt-image-2.5-flare-adobe`](./media/gpt-image-2.5-flare-adobe.md) | `/v1/images/generations`、`/v1/images/edits` | [教程](https://deepclub.net/articles/gpt-image-2-5-flare-adobe-how-to-use-guide) · [定价](https://deepclub.net/pricing/gpt-image-2.5-flare-adobe) |
| [`gpt-image-2.5-sunburst`](./media/gpt-image-2.5-sunburst.md) | `/v1/images/generations`、`/v1/images/edits` | [教程](https://deepclub.net/articles/gpt-image-2-5-sunburst-how-to-use-guide) · [定价](https://deepclub.net/pricing/gpt-image-2.5-sunburst) |
| [`gpt-image-2.5-sunburst-adobe`](./media/gpt-image-2.5-sunburst-adobe.md) | `/v1/images/generations`、`/v1/images/edits` | [教程](https://deepclub.net/articles/gpt-image-2-5-sunburst-adobe-how-to-use-guide) · [定价](https://deepclub.net/pricing/gpt-image-2.5-sunburst-adobe) |
| [`nano-banana-2`](./media/nano-banana-2.md) | `/v1/images/generations`、`/v1/images/edits` | [教程](https://deepclub.net/articles/nano-banana-2-how-to-use-guide) · [定价](https://deepclub.net/pricing/nano-banana-2) |
| [`nano-banana-pro`](./media/nano-banana-pro.md) | `/v1/images/generations`、`/v1/images/edits` | [教程](https://deepclub.net/articles/nano-banana-pro-how-to-use-guide) · [定价](https://deepclub.net/pricing/nano-banana-pro) |
