# GPT-6 Astra 怎么用？深度推理、科学计算与难题攻关接入教程

> **一句话定位**：优先 OpenAI Responses，SDK Base URL 填 https://deepclub.net/v1，模型名填 gpt-6-astra。  
> **官网原生教程页**：[https://deepclub.net/articles/gpt-6-astra-how-to-use-guide](https://deepclub.net/articles/gpt-6-astra-how-to-use-guide) ｜ **实时定价与规格**：[https://deepclub.net/pricing/gpt-6-astra](https://deepclub.net/pricing/gpt-6-astra)

## 实时定价快照

| 计费项 | 实时参考价格（以官网为准） | 单位 |
|---|---|---|
| 输入 | ¥3.5 | 元 / 百万 Token |
| 输出 | ¥17.5 | 元 / 百万 Token |
| 缓存读取 (`cache_read`) | ¥0.35 | 元 / 百万 Token |

> 实时价格、可用状态与规格详情见官网：[https://deepclub.net/pricing/gpt-6-astra](https://deepclub.net/pricing/gpt-6-astra)

| 参数 | 值 |
| --- | --- |
| 模型 ID | `gpt-6-astra` |
| 类型 | 文本 |
| 调用端点 | `POST /v1/responses`、`POST /v1/chat/completions` |
| 推荐协议 | OpenAI Responses（推荐） |
| 客户端与 SDK | Cherry Studio、Cursor、OpenAI Python SDK |
| 计费方式 | 按百万 Token，实时价格见 [定价页](https://deepclub.net/pricing/gpt-6-astra) |

## GPT-6 Astra 适合做什么，不适合做什么？

适合算法设计、科研推导和难以定位的 Bug 排查。

gpt-6-astra 是在售 GPT 文本型号中单价最高的一款，可用于复杂逻辑、算法设计和科研推导；复杂问题的回答通常更长，Token 消耗也更高。

适合的场景：

- 竞赛题、证明题的逐步推导
- 设计和分析算法的复杂度
- 排查复现困难的逻辑 Bug

不太适合：高频、即时回复的聊天或客服场景，单价更低的 gpt-6-luna 更合适。

## gpt-6-astra 的 Base URL 和 API Key 怎么获取？

本文 OpenAI Python SDK 示例的 Base URL 为 `https://deepclub.net/v1`，Key 在控制台创建。客户端地址拼接规则可能不同，请按下方对应客户端的步骤填写。

1. 登录 [深云 DeepClub](https://deepclub.net/)，进入控制台的 **API 密钥** 页面。
2. 点击 **创建 API 密钥**，复制生成的 `sk-` 开头密钥。
3. 在客户端或代码中填写以下三项：

```text
Base URL: https://deepclub.net/v1
API Key:  sk-你的DeepClub密钥
Model:    gpt-6-astra
```

## gpt-6-astra 推荐哪个协议？

**首选 OpenAI Responses：`POST /v1/responses`。**

OpenAI 推荐新集成使用 Responses，DeepClub 当前也为该模型开放此端点。它使用 input / output 和专门的流式事件；不能把 Chat 的 messages、choices 或 delta 解析直接搬过来。

参考：[OpenAI Responses 迁移指南](https://developers.openai.com/api/docs/guides/migrate-to-responses)。

选择依据是模型官方接口、DeepClub 已发布端点和客户端实际能力。使用缓存、推理或工具调用前，请先用小请求验证；端点可用不代表提供联网搜索等厂商托管工具、保证缓存命中或在服务端保存会话。

## GPT-6 Astra 在 Cherry Studio 里怎么配置？

Cherry Studio v2.2.0 使用端点配置，模型选择 OpenAI Responses。

1. 进入 **设置 → 模型服务**，添加自定义提供商，例如 DeepClub。
2. 为服务配置 **OpenAI Responses** 端点，地址填 `https://deepclub.net/v1`，粘贴 DeepClub 密钥。
3. 添加完整模型 ID `gpt-6-astra`，在该模型的端点设置中仅选择上面的协议。
4. 保存并检查最终请求地址应为 `https://deepclub.net/v1/responses`，然后在对话中选择该模型。

建议按协议分别建服务，避免多端点的优先级让模型走错协议。旧版设置与 v2.2.0 不同，若没有端点选择请升级或使用本文 SDK。

## GPT-6 Astra 在 Cursor 里怎么配置？

覆盖 Base URL 为 deepclub.net/v1，再添加模型名。

> 这是客户端自动选择协议的自定义 OpenAI 入口。gpt-6-astra 首选 OpenAI Responses；若客户端不能同时自定义协议、网关地址和完整模型 ID，建议使用下文 SDK 或支持该协议的客户端。只填写模型名不能保证使用首选协议。

1. 打开 Cursor 的 **Settings → Models**。
2. 在 **OpenAI API Key** 一栏粘贴 DeepClub 密钥。
3. 展开 **Override OpenAI Base URL**，填写 `https://deepclub.net/v1`。
4. 点击 **Add model**，输入 `gpt-6-astra`，然后在 Chat 面板选择它。

Cursor 的 OpenAI 覆盖入口会按版本和模式发送 Chat Completions 或 Responses，不会发送 Anthropic Messages。保存后发一条短消息，在控制台使用日志中确认调用成功；若所选型号不支持该模式使用的接口，请改用本文 SDK 或可以明确选择协议的客户端。Tab 补全等功能使用 Cursor 自己的模型。

## 怎么用 Python 调用 gpt-6-astra（推荐协议）？

安装 `pip install -U openai`，并在本地设置环境变量 `DEEPCLUB_API_KEY`。

```python
import os
from openai import OpenAI

client = OpenAI(
    api_key=os.environ["DEEPCLUB_API_KEY"],
    base_url="https://deepclub.net/v1",
)

response = client.responses.create(
    model="gpt-6-astra",
    input="证明任意三个连续整数之积能被六整除",
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
  -d '{"model":"gpt-6-astra","input":"证明任意三个连续整数之积能被六整除","store":false}'
```

## 用 curl 调用 OpenAI Chat Completions（兼容选项）

仅在现有客户端或集成需要该协议时选用；切换时必须同步修改请求体和响应解析。

```bash
curl https://deepclub.net/v1/chat/completions \
  -H "Authorization: Bearer $DEEPCLUB_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model":"gpt-6-astra","messages":[{"role":"user","content":"证明任意三个连续整数之积能被六整除"}]}'
```

## GPT-6 Astra 适合用在哪些请求上？

用在真正需要推导的任务上，日常问答交给单价更低的型号。

可以用 Responses 的 `reasoning={"effort": "high"}`（Chat 用 `reasoning_effort`）提高推理强度；实际消耗以返回的 usage 和控制台使用日志为准。

复杂问题的回答通常更长、耗时更久，客户端如有请求超时设置，可适当调大。

## gpt-6-astra API 怎么计费？

按百万 Token 计费，输入和输出分别计价。

实时价格见 [定价页](https://deepclub.net/pricing/gpt-6-astra)。

- 计费单位：按百万 Token，价格可能调整，以 [定价页](https://deepclub.net/pricing/gpt-6-astra) 实时显示为准。
- 用量明细：在控制台 **使用日志** 中按请求查看消耗。
- 充值方式：支持信用卡、微信支付、支付宝等，以控制台充值页显示为准。

## 调用 gpt-6-astra 报 401 怎么办？

401 表示密钥无效或缺失，先检查 Key 是否完整。

- 确认 API Key 前后没有空格或换行，且令牌未被禁用、未过期。
- 请求头格式为 Authorization: Bearer sk-...。
- 如果返回 404，按上面的客户端步骤检查最终请求地址，避免重复或漏掉 /v1；模型 ID 必须完整匹配 `gpt-6-astra`。SDK 与桌面客户端的 Base URL 拼接规则不相同。

## 用 gpt-6-astra 时请求内容会被存储吗？

不会，深云 DeepClub 实行零留存（ZDR）政策。

深云 DeepClub 实行 Zero Data Retention (ZDR) 零留存政策：请求的输入和输出内容不做持久化存储，详见 [隐私政策](https://deepclub.net/privacy-policy)。

## 使用 GPT-6 Astra 还有哪些常见问题？

以下问答汇总了调用时最常见的问题。

### 调用 gpt-6-astra 返回 401 Unauthorized 怎么办？

401 表示密钥无效或缺失，先检查 API Key 是否复制完整、前后没有空格。请求头格式为 Authorization: Bearer sk-...。

### gpt-6-astra 用哪个接口和协议调用？

优先用 OpenAI Responses，端点为 POST /v1/responses，OpenAI Python SDK 的 Base URL 为 `https://deepclub.net/v1`。其他已开放端点 POST /v1/chat/completions；只有客户端限制或已有集成需要时才使用兼容路径。

### gpt-6-astra API 怎么计费？

按百万 Token 计费，输入和输出分别计价。实时价格见 https://deepclub.net/pricing/gpt-6-astra。

### GPT-6 Astra 适合用在哪些请求上？

用在真正需要推导的任务上，日常问答交给单价更低的型号。

## 多框架 & 主流平台快捷接入对照

| 工具 / 开发框架 | 选用模式 / Provider | 地址（Base URL / Endpoint） | 模型参数写法与注意事项 |
|---|---|---|---|
| **Codex CLI** | `wire_api = "responses"` | `base_url = "https://deepclub.net/v1"` | 写入 `~/.codex/config.toml`，指定 `model = "gpt-6-astra"` |
| **Vercel AI SDK** | `createOpenAI` | `https://deepclub.net/v1` | `openai('gpt-6-astra')` 或 `openai.responses('gpt-6-astra')`，配 `store: false` |
| **OpenAI Agents SDK** | `AsyncOpenAI` + Responses | `https://deepclub.net/v1` | 默认走 Responses，配 `store=False` 与 `set_tracing_disabled(True)` |
| **LangChain (Py)** | `ChatOpenAI` | `base_url="https://deepclub.net/v1"` | 开启 `use_responses_api=True, store=False` |
| **Dify 官方插件** | `OpenAI` (`1.1.0+`) | `https://deepclub.net/v1` | `API Protocol` 选 `Responses API` |
| **沉浸式翻译** | OpenAI 自定义 | `https://deepclub.net/v1/responses` | **必须填完整端点 URL** |

> **协议主选摘要**：`gpt-6-astra` 推荐优先使用 **OpenAI Responses (`/v1/responses`)**。完整客户端配置指南见 [平台与开发框架适配目录](../../guides/platforms/README.md) 与 [全量排障手册](../../guides/protocol-and-clients/error-troubleshooting-handbook.md)。
