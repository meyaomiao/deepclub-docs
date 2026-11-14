# Grok 4.7 怎么用？科技资讯追踪与三种协议接入教程

> **一句话定位**：优先 OpenAI Responses，SDK Base URL 填 https://deepclub.net/v1，模型名填 grok-4.7。  
> **官网原生教程页**：[https://deepclub.net/articles/grok-4-7-how-to-use-guide](https://deepclub.net/articles/grok-4-7-how-to-use-guide) ｜ **实时定价与规格**：[https://deepclub.net/pricing/grok-4.7](https://deepclub.net/pricing/grok-4.7)

## 实时定价快照

| 计费项 | 实时参考价格（以官网为准） | 单位 |
|---|---|---|
| 输入 | ¥0.4 | 元 / 百万 Token |
| 输出 | ¥1.2 | 元 / 百万 Token |
| 缓存读取 (`cache_read`) | ¥0.1 | 元 / 百万 Token |

> 实时价格、可用状态与规格详情见官网：[https://deepclub.net/pricing/grok-4.7](https://deepclub.net/pricing/grok-4.7)

| 参数 | 值 |
| --- | --- |
| 模型 ID | `grok-4.7` |
| 类型 | 文本 |
| 调用端点 | `POST /v1/responses`、`POST /v1/chat/completions`、`POST /v1/messages` |
| 推荐协议 | OpenAI Responses（推荐） |
| 客户端与 SDK | NextChat、Cherry Studio、OpenAI Python SDK |
| 计费方式 | 按百万 Token，实时价格见 [定价页](https://deepclub.net/pricing/grok-4.7) |

## Grok 4.7 适合做什么，不适合做什么？

适合科技资讯解读、开源动态追踪和直接的观点表达。

grok-4.7 是 xAI 的模型，回答风格直接，对科技圈新闻、金融动态和开源社区热点有较好的知识覆盖。

适合的场景：

- 解读科技新闻和开源项目动态
- 对一个观点做多角度事实校验
- 需要直接、不绕弯的回答风格

不太适合：需要严格公文语气的中文写作，选 glm-5.3 更合适。

## grok-4.7 的 Base URL 和 API Key 怎么获取？

本文 OpenAI Python SDK 示例的 Base URL 为 `https://deepclub.net/v1`，Key 在控制台创建。客户端地址拼接规则可能不同，请按下方对应客户端的步骤填写。

1. 登录 [深云 DeepClub](https://deepclub.net/)，进入控制台的 **API 密钥** 页面。
2. 点击 **创建 API 密钥**，复制生成的 `sk-` 开头密钥。
3. 在客户端或代码中填写以下三项：

```text
Base URL: https://deepclub.net/v1
API Key:  sk-你的DeepClub密钥
Model:    grok-4.7
```

## grok-4.7 推荐哪个协议？

**首选 OpenAI Responses：`POST /v1/responses`。**

xAI 推荐新集成使用 Responses，DeepClub 当前也为该模型开放此端点。它使用 input / output 和专门的流式事件；不能把 Chat 的 messages、choices 或 delta 解析直接搬过来。

参考：[xAI 推荐新集成使用 Responses](https://docs.x.ai/developers/rest-api-reference/inference/chat-completions)。

选择依据是模型官方接口、DeepClub 已发布端点和客户端实际能力。使用缓存、推理或工具调用前，请先用小请求验证；端点可用不代表提供联网搜索等厂商托管工具、保证缓存命中或在服务端保存会话。

## Grok 4.7 在 NextChat 里怎么配置？

开启自定义接口，接口地址填 https://deepclub.net。

> 这是该客户端的 Chat Completions 兼容配置。grok-4.7 首选 OpenAI Responses；若客户端不能同时自定义协议、网关地址和完整模型 ID，建议使用下文 SDK 或支持该协议的客户端。只填写模型名不能保证使用首选协议。

1. 打开 NextChat 的 **设置**，勾选 **自定义接口**。
2. 提供商选 **OpenAI**，**接口地址** 填 `https://deepclub.net`，**API Key** 粘贴 DeepClub 密钥。
3. 在 **自定义模型名** 中填入 `+grok-4.7@OpenAI`，明确绑定 Chat 提供商。
4. 回到对话页，在模型选择器中切换到 `grok-4.7`。

## Grok 4.7 在 Cherry Studio 里怎么配置？

Cherry Studio v2.2.0 使用端点配置，模型选择 OpenAI Responses。

1. 进入 **设置 → 模型服务**，添加自定义提供商，例如 DeepClub。
2. 为服务配置 **OpenAI Responses** 端点，地址填 `https://deepclub.net/v1`，粘贴 DeepClub 密钥。
3. 添加完整模型 ID `grok-4.7`，在该模型的端点设置中仅选择上面的协议。
4. 保存并检查最终请求地址应为 `https://deepclub.net/v1/responses`，然后在对话中选择该模型。

建议按协议分别建服务，避免多端点的优先级让模型走错协议。旧版设置与 v2.2.0 不同，若没有端点选择请升级或使用本文 SDK。

## 怎么用 Python 调用 grok-4.7（推荐协议）？

安装 `pip install -U openai`，并在本地设置环境变量 `DEEPCLUB_API_KEY`。

```python
import os
from openai import OpenAI

client = OpenAI(
    api_key=os.environ["DEEPCLUB_API_KEY"],
    base_url="https://deepclub.net/v1",
)

response = client.responses.create(
    model="grok-4.7",
    input="概括最近一周开源大模型领域的重要动态",
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
  -d '{"model":"grok-4.7","input":"概括最近一周开源大模型领域的重要动态","store":false}'
```

## 用 curl 调用 OpenAI Chat Completions（兼容选项）

仅在现有客户端或集成需要该协议时选用；切换时必须同步修改请求体和响应解析。

```bash
curl https://deepclub.net/v1/chat/completions \
  -H "Authorization: Bearer $DEEPCLUB_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model":"grok-4.7","messages":[{"role":"user","content":"概括最近一周开源大模型领域的重要动态"}]}'
```

## 用 curl 调用 Anthropic Messages（兼容选项）

仅在现有客户端或集成需要该协议时选用；切换时必须同步修改请求体和响应解析。

```bash
curl https://deepclub.net/v1/messages \
  -H "x-api-key: $DEEPCLUB_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "Content-Type: application/json" \
  -d '{"model":"grok-4.7","max_tokens":4096,"messages":[{"role":"user","content":"概括最近一周开源大模型领域的重要动态"}]}'
```

## Grok 4.7 能获取实时新闻吗？

以模型自身知识为准，本文的基本 API 请求不自动联网搜索。

通过 API 调用时，模型只基于训练数据和你提供的上下文回答。需要最新信息时，请把新闻原文或链接内容放进提示词，让模型做总结和校验。

涉及事实判断的结论，建议对照原始来源核实。

## grok-4.7 API 怎么计费？

按百万 Token 计费，输入和输出分别计价。

实时价格见 [定价页](https://deepclub.net/pricing/grok-4.7)。

- 计费单位：按百万 Token，价格可能调整，以 [定价页](https://deepclub.net/pricing/grok-4.7) 实时显示为准。
- 用量明细：在控制台 **使用日志** 中按请求查看消耗。
- 充值方式：支持信用卡、微信支付、支付宝等，以控制台充值页显示为准。

## 调用 grok-4.7 报 401 怎么办？

401 表示密钥无效或缺失，先检查 Key 是否完整。

- 确认 API Key 前后没有空格或换行，且令牌未被禁用、未过期。
- Messages 请求用 x-api-key: sk-... 并带 anthropic-version: 2023-06-01；Claude Code 的 ANTHROPIC_AUTH_TOKEN 使用网关支持的 Authorization: Bearer。OpenAI Responses / OpenAI Chat Completions 请求使用 Authorization: Bearer sk-...。
- 如果返回 404，按上面的客户端步骤检查最终请求地址，避免重复或漏掉 /v1；模型 ID 必须完整匹配 `grok-4.7`。SDK 与桌面客户端的 Base URL 拼接规则不相同。

## 用 grok-4.7 时请求内容会被存储吗？

不会，深云 DeepClub 实行零留存（ZDR）政策。

深云 DeepClub 实行 Zero Data Retention (ZDR) 零留存政策：请求的输入和输出内容不做持久化存储，详见 [隐私政策](https://deepclub.net/privacy-policy)。

## 使用 Grok 4.7 还有哪些常见问题？

以下问答汇总了调用时最常见的问题。

### 调用 grok-4.7 返回 401 Unauthorized 怎么办？

401 表示密钥无效或缺失，先检查 API Key 是否复制完整、前后没有空格。Messages 请求用 x-api-key: sk-... 并带 anthropic-version: 2023-06-01；Claude Code 的 ANTHROPIC_AUTH_TOKEN 使用网关支持的 Authorization: Bearer。OpenAI Responses / OpenAI Chat Completions 请求使用 Authorization: Bearer sk-...。

### grok-4.7 用哪个接口和协议调用？

优先用 OpenAI Responses，端点为 POST /v1/responses，OpenAI Python SDK 的 Base URL 为 `https://deepclub.net/v1`。其他已开放端点 POST /v1/chat/completions、POST /v1/messages；只有客户端限制或已有集成需要时才使用兼容路径。

### grok-4.7 API 怎么计费？

按百万 Token 计费，输入和输出分别计价。实时价格见 https://deepclub.net/pricing/grok-4.7。

### Grok 4.7 能获取实时新闻吗？

以模型自身知识为准，本文的基本 API 请求不自动联网搜索。

## 多框架 & 主流平台快捷接入对照

| 工具 / 开发框架 | 选用模式 / Provider | 地址（Base URL / Endpoint） | 模型参数写法与注意事项 |
|---|---|---|---|
| **Codex CLI** | `wire_api = "responses"` | `base_url = "https://deepclub.net/v1"` | 写入 `~/.codex/config.toml`，指定 `model = "grok-4.7"` |
| **Vercel AI SDK** | `createOpenAI` | `https://deepclub.net/v1` | `openai('grok-4.7')` 或 `openai.responses('grok-4.7')`，配 `store: false` |
| **OpenAI Agents SDK** | `AsyncOpenAI` + Responses | `https://deepclub.net/v1` | 默认走 Responses，配 `store=False` 与 `set_tracing_disabled(True)` |
| **LangChain (Py)** | `ChatOpenAI` | `base_url="https://deepclub.net/v1"` | 开启 `use_responses_api=True, store=False` |
| **Dify 官方插件** | `OpenAI` (`1.1.0+`) | `https://deepclub.net/v1` | `API Protocol` 选 `Responses API` |
| **沉浸式翻译** | OpenAI 自定义 | `https://deepclub.net/v1/responses` | **必须填完整端点 URL** |

> **协议主选摘要**：`grok-4.7` 推荐优先使用 **OpenAI Responses (`/v1/responses`)**。完整客户端配置指南见 [平台与开发框架适配目录](../../guides/platforms/README.md) 与 [全量排障手册](../../guides/protocol-and-clients/error-troubleshooting-handbook.md)。
