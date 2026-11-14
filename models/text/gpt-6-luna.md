# GPT-6 Luna 怎么用？即时问答与流式输出配置

> **一句话定位**：优先 OpenAI Responses，SDK Base URL 填 https://deepclub.net/v1，模型名填 gpt-6-luna。  
> **官网原生教程页**：[https://deepclub.net/articles/gpt-6-luna-how-to-use-guide](https://deepclub.net/articles/gpt-6-luna-how-to-use-guide) ｜ **实时定价与规格**：[https://deepclub.net/pricing/gpt-6-luna](https://deepclub.net/pricing/gpt-6-luna)

## 实时定价快照

| 计费项 | 实时参考价格（以官网为准） | 单位 |
|---|---|---|
| 输入 | ¥0.12 | 元 / 百万 Token |
| 输出 | ¥0.6 | 元 / 百万 Token |
| 缓存读取 (`cache_read`) | ¥0.012 | 元 / 百万 Token |

> 实时价格、可用状态与规格详情见官网：[https://deepclub.net/pricing/gpt-6-luna](https://deepclub.net/pricing/gpt-6-luna)

| 参数 | 值 |
| --- | --- |
| 模型 ID | `gpt-6-luna` |
| 类型 | 文本 |
| 调用端点 | `POST /v1/responses`、`POST /v1/chat/completions` |
| 推荐协议 | OpenAI Responses（推荐） |
| 客户端与 SDK | Chatbox、沉浸式翻译、OpenAI Python SDK |
| 计费方式 | 按百万 Token，实时价格见 [定价页](https://deepclub.net/pricing/gpt-6-luna) |

## GPT-6 Luna 适合做什么，不适合做什么？

适合高频短问答、划词翻译和需要流式输出的实时助手。

gpt-6-luna 是在售 GPT 文本型号中单价最低的一款，适合高频短问答、实时互动和流式输出场景。它本身不联网，需要最新信息时请在请求中提供资料。

适合的场景：

- 桌面端随手问答
- 浏览器划词翻译、即时解释
- 需要逐字输出的对话机器人

不太适合：需要长时间深度推理的难题，这类任务选 gpt-6-astra。

## gpt-6-luna 的 Base URL 和 API Key 怎么获取？

本文 OpenAI Python SDK 示例的 Base URL 为 `https://deepclub.net/v1`，Key 在控制台创建。客户端地址拼接规则可能不同，请按下方对应客户端的步骤填写。

1. 登录 [深云 DeepClub](https://deepclub.net/)，进入控制台的 **API 密钥** 页面。
2. 点击 **创建 API 密钥**，复制生成的 `sk-` 开头密钥。
3. 在客户端或代码中填写以下三项：

```text
Base URL: https://deepclub.net/v1
API Key:  sk-你的DeepClub密钥
Model:    gpt-6-luna
```

## gpt-6-luna 推荐哪个协议？

**首选 OpenAI Responses：`POST /v1/responses`。**

OpenAI 推荐新集成使用 Responses，DeepClub 当前也为该模型开放此端点。它使用 input / output 和专门的流式事件；不能把 Chat 的 messages、choices 或 delta 解析直接搬过来。

参考：[OpenAI Responses 迁移指南](https://developers.openai.com/api/docs/guides/migrate-to-responses)。

选择依据是模型官方接口、DeepClub 已发布端点和客户端实际能力。使用缓存、推理或工具调用前，请先用小请求验证；端点可用不代表提供联网搜索等厂商托管工具、保证缓存命中或在服务端保存会话。

## GPT-6 Luna 在 Chatbox 里怎么配置？

使用支持 OpenAI Responses 的新版 Chatbox，显式选择 Responses API 模式。

1. 进入 **设置 → 模型提供方 → 添加自定义提供方**。
2. **API 模式** 选择 **OpenAI Responses API Compatible**，不要选 Chat Completions。
3. **API Host** 填 `https://deepclub.net/v1`，**API Path** 填 `/responses`。
4. 粘贴 DeepClub 密钥，添加完整模型 ID `gpt-6-luna`，保存后选择该模型。

最终地址应为 `https://deepclub.net/v1/responses`。旧版若没有 Responses 模式，可升级；继续用 Chat 模式时，只能视为兼容方案。

## GPT-6 Luna 在 沉浸式翻译 里怎么配置？

使用支持 Responses 的版本，将 OpenAI 服务的自定义接口设为完整 /v1/responses 地址。

1. 使用沉浸式翻译 1.35.2 或更新版本，进入 **设置 → 翻译服务 → OpenAI**。
2. 填写 DeepClub API Key。
3. **自定义 API 接口地址** 填 `https://deepclub.net/v1/responses`，客户端按路径识别 Responses。
4. 模型选 **自定义模型**，填入 `gpt-6-luna`，保存并测试。

不用额外粘贴高级 JSON。旧版若仍按 messages 发送或不能识别 Responses，请升级，或明确改用完整 `/v1/chat/completions` 地址作为兼容方案。

## 怎么用 Python 调用 gpt-6-luna（推荐协议）？

安装 `pip install -U openai`，并在本地设置环境变量 `DEEPCLUB_API_KEY`。

```python
import os
from openai import OpenAI

client = OpenAI(
    api_key=os.environ["DEEPCLUB_API_KEY"],
    base_url="https://deepclub.net/v1",
)

response = client.responses.create(
    model="gpt-6-luna",
    input="用一句话解释什么是 HTTP 缓存",
    store=False,
)
print(response.output_text)
```

多轮对话由客户端保留历史：把上一轮的 input 和返回的 output 项（message、function_call，以及你补上的 function_call_output）一起带回下一次请求，不要只保留 output_text 而丢掉工具调用。本站不支持 previous_response_id 和服务端会话存储，示例中的 store=False 也表示不依赖服务端保存。

## 用 curl 调用 OpenAI Responses（推荐）

```bash
curl https://deepclub.net/v1/responses \
  -H "Authorization: Bearer $DEEPCLUB_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model":"gpt-6-luna","input":"用一句话解释什么是 HTTP 缓存","store":false}'
```

## 用 curl 调用 OpenAI Chat Completions（兼容选项）

仅在现有客户端或集成需要该协议时选用；切换时必须同步修改请求体和响应解析。

```bash
curl https://deepclub.net/v1/chat/completions \
  -H "Authorization: Bearer $DEEPCLUB_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model":"gpt-6-luna","messages":[{"role":"user","content":"用一句话解释什么是 HTTP 缓存"}]}'
```

## GPT-6 Luna 怎么开启流式输出？

请求里加 stream=True，按增量逐段读取回复。

示例：

```python
stream = client.responses.create(
    model="gpt-6-luna",
    input="用一句话解释什么是 HTTP 缓存",
    store=False,
    stream=True,
)
for event in stream:
    if event.type == "response.output_text.delta":
        print(event.delta, end="", flush=True)
    elif event.type in ("response.completed", "response.failed"):
        print()
```

遇到 `response.output_text.delta` 时读取 `event.delta`，并处理完成和失败事件；不要用 Chat 的 choices[0].delta 解析 Responses。

流式输出不改变计费，按实际输入、输出 Token 计算。

## gpt-6-luna API 怎么计费？

按百万 Token 计费，输入和输出分别计价。

实时价格见 [定价页](https://deepclub.net/pricing/gpt-6-luna)。

- 计费单位：按百万 Token，价格可能调整，以 [定价页](https://deepclub.net/pricing/gpt-6-luna) 实时显示为准。
- 用量明细：在控制台 **使用日志** 中按请求查看消耗。
- 充值方式：支持信用卡、微信支付、支付宝等，以控制台充值页显示为准。

## 调用 gpt-6-luna 报 401 怎么办？

401 表示密钥无效或缺失，先检查 Key 是否完整。

- 确认 API Key 前后没有空格或换行，且令牌未被禁用、未过期。
- 请求头格式为 Authorization: Bearer sk-...。
- 如果返回 404，按上面的客户端步骤检查最终请求地址，避免重复或漏掉 /v1；模型 ID 必须完整匹配 `gpt-6-luna`。SDK 与桌面客户端的 Base URL 拼接规则不相同。

## 用 gpt-6-luna 时请求内容会被存储吗？

不会，深云 DeepClub 实行零留存（ZDR）政策。

深云 DeepClub 实行 Zero Data Retention (ZDR) 零留存政策：请求的输入和输出内容不做持久化存储，详见 [隐私政策](https://deepclub.net/privacy-policy)。

## 使用 GPT-6 Luna 还有哪些常见问题？

以下问答汇总了调用时最常见的问题。

### 调用 gpt-6-luna 返回 401 Unauthorized 怎么办？

401 表示密钥无效或缺失，先检查 API Key 是否复制完整、前后没有空格。请求头格式为 Authorization: Bearer sk-...。

### gpt-6-luna 用哪个接口和协议调用？

优先用 OpenAI Responses，端点为 POST /v1/responses，OpenAI Python SDK 的 Base URL 为 `https://deepclub.net/v1`。其他已开放端点 POST /v1/chat/completions；只有客户端限制或已有集成需要时才使用兼容路径。

### gpt-6-luna API 怎么计费？

按百万 Token 计费，输入和输出分别计价。实时价格见 https://deepclub.net/pricing/gpt-6-luna。

### GPT-6 Luna 怎么开启流式输出？

请求里加 stream=True，按增量逐段读取回复。

## 多框架 & 主流平台快捷接入对照

| 工具 / 开发框架 | 选用模式 / Provider | 地址（Base URL / Endpoint） | 模型参数写法与注意事项 |
|---|---|---|---|
| **Codex CLI** | `wire_api = "responses"` | `base_url = "https://deepclub.net/v1"` | 写入 `~/.codex/config.toml`，指定 `model = "gpt-6-luna"` |
| **Vercel AI SDK** | `createOpenAI` | `https://deepclub.net/v1` | `openai('gpt-6-luna')` 或 `openai.responses('gpt-6-luna')`，配 `store: false` |
| **OpenAI Agents SDK** | `AsyncOpenAI` + Responses | `https://deepclub.net/v1` | 默认走 Responses，配 `store=False` 与 `set_tracing_disabled(True)` |
| **LangChain (Py)** | `ChatOpenAI` | `base_url="https://deepclub.net/v1"` | 开启 `use_responses_api=True, store=False` |
| **Dify 官方插件** | `OpenAI` (`1.1.0+`) | `https://deepclub.net/v1` | `API Protocol` 选 `Responses API` |
| **沉浸式翻译** | OpenAI 自定义 | `https://deepclub.net/v1/responses` | **必须填完整端点 URL** |

> **协议主选摘要**：`gpt-6-luna` 推荐优先使用 **OpenAI Responses (`/v1/responses`)**。完整客户端配置指南见 [平台与开发框架适配目录](../../guides/platforms/README.md) 与 [全量排障手册](../../guides/protocol-and-clients/error-troubleshooting-handbook.md)。
