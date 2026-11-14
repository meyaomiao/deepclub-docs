# Vercel AI SDK 接入深云 DeepClub：`@ai-sdk/openai` vs `@ai-sdk/anthropic` 与多模型路由

**Vercel AI SDK** 是 Next.js / TypeScript 全栈应用与流式 AI 界面最常用的 SDK 之一。但在对接第三方聚合网关（如 **[深云 DeepClub](https://deepclub.net)**）时，有两个和 Python SDK 完全不同的关键细节：

1. **`@ai-sdk/anthropic` 的 `baseURL` 默认前缀是 `https://api.anthropic.com/v1`（带 `/v1`）** —— 这与官方 Anthropic Python SDK（填站根不带 `/v1`）恰好相反；
2. **`@ai-sdk/openai` 从 AI SDK 5 起默认调用 Responses API**（`deepclub('gpt-6.1-sol')` / `deepclub.responses(...)`），而调用 DeepSeek / Gemini / GLM / Kimi / Qwen 时应改用 `deepclub.chat(...)`。

---

## 一、安装依赖

```bash
npm install ai @ai-sdk/openai @ai-sdk/anthropic
```

环境变量 `.env.local`：

```bash
DEEPCLUB_API_KEY="sk-你的DeepClub密钥"
```

---

## 二、协议与工厂方法对照表（直接抄作业）

| 模型家族 | Provider 工厂与 `baseURL` | 调用方法 | 实际端点 |
|---|---|---|---|
| **GPT 文本 & Grok**<br>(`gpt-6.1-sol`、`gpt-6-astra`、`gpt-6-luna`、`grok-4.7`) | `createOpenAI({ baseURL: 'https://deepclub.net/v1', apiKey })` | `openai('gpt-6.1-sol')` 或 `openai.responses('gpt-6.1-sol')` | `POST /v1/responses` |
| **DeepSeek / Gemini 文本 / GLM / Kimi / Qwen** | 同上 `createOpenAI({ baseURL: 'https://deepclub.net/v1', apiKey })` | **`openai.chat('deepseek-v4.1-flash')`** | `POST /v1/chat/completions` |
| **Claude 系列**<br>(`claude-sonnet-5-5`、`claude-opus-5-5`、`claude-fable-5-1/5-2`) | `createAnthropic({ baseURL: 'https://deepclub.net/v1', apiKey })` | `anthropic('claude-sonnet-5-5')` | `POST /v1/messages` |

---

## 三、多协议路由封装（`lib/ai.ts`）

在一个项目中同时管理三条协议通道，按模型家族精准分发：

```ts
import { createOpenAI } from '@ai-sdk/openai'
import { createAnthropic } from '@ai-sdk/anthropic'

const apiKey = process.env.DEEPCLUB_API_KEY!

// 1) OpenAI Provider（Responses + Chat Completions）
export const dcOpenAI = createOpenAI({
  apiKey,
  baseURL: 'https://deepclub.net/v1',
})

// 2) Anthropic Provider（注意 @ai-sdk/anthropic 的 baseURL 需要包含 /v1）
export const dcAnthropic = createAnthropic({
  apiKey,
  baseURL: 'https://deepclub.net/v1',
})

// 3) 常用模型别名导出
export const models = {
  // Responses API（默认）
  gptSol: dcOpenAI.responses('gpt-6.1-sol'),
  gptAstra: dcOpenAI.responses('gpt-6-astra'),
  gptLuna: dcOpenAI.responses('gpt-6-luna'),
  grok: dcOpenAI.responses('grok-4.7'),

  // Chat Completions API（必须用 .chat()）
  deepseek: dcOpenAI.chat('deepseek-v4.1-flash'),
  geminiHydrogen: dcOpenAI.chat('gemini-4-hydrogen'),
  geminiArgon: dcOpenAI.chat('gemini-4-argon'),
  glm: dcOpenAI.chat('glm-5.3'),
  kimi: dcOpenAI.chat('kimi-k3'),
  qwen: dcOpenAI.chat('qwen3.8-flash'),

  // Anthropic Messages API
  claudeSonnet: dcAnthropic('claude-sonnet-5-5'),
  claudeOpus: dcAnthropic('claude-opus-5-5'),
}
```

---

## 四、Next.js App Router 流式 Route Handler 示例

```ts
// app/api/chat/route.ts
import { streamText, convertToModelMessages } from 'ai'
import { models } from '@/lib/ai'

export async function POST(req: Request) {
  const { messages, modelKey = 'claudeSonnet' } = await req.json()
  const model = models[modelKey as keyof typeof models] ?? models.gptSol

  const result = streamText({
    model,
    messages: await convertToModelMessages(messages),
    // 如果选用 Responses 模型，明确关闭服务端存储以匹配 ZDR
    providerOptions:
      modelKey.startsWith('gpt') || modelKey === 'grok'
        ? { openai: { store: false } }
        : undefined,
  })

  return result.toUIMessageStreamResponse()
}
```

---

## 五、常见踩坑提醒

1. **用 `@ai-sdk/openai` 调 `deepseek-v4.1-flash` 或 `gemini-4-hydrogen` 时报 404 / 不支持**：因为 `dcOpenAI('deepseek-v4.1-flash')` 默认发向 `/v1/responses`，而 DeepSeek / Gemini 在目录中只开放了 `/v1/chat/completions`。改成 **`dcOpenAI.chat('deepseek-v4.1-flash')`** 即可。
2. **`@ai-sdk/anthropic` 的 `baseURL` 少了 `/v1` 报 404**：官方 Python SDK 会自动加 `/v1`，而 `@ai-sdk/anthropic` 默认前缀本身就带 `/v1`，它会直接往 `baseURL` 后拼 `/messages`，所以这里必须填 `https://deepclub.net/v1`。
3. **Claude 5.x 采样参数**：`claude-opus-5-5`、`claude-sonnet-5-5`、`claude-fable-5-2` 在 `/v1/messages` 上不要传入自定义非 1 的 `temperature` / `topP` / `topK`。

完整模型目录与价格：[https://deepclub.net/pricing](https://deepclub.net/pricing) ｜ [https://deepclub.net/docs](https://deepclub.net/docs)
