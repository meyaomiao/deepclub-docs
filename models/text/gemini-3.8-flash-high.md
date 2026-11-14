# Gemini 3.8 Flash High 怎么用？工具调用与 Agent 工作流配置

> **一句话定位**：优先 OpenAI Chat Completions，SDK Base URL 填 https://deepclub.net/v1，模型名填 gemini-3.8-flash-high。  
> **官网原生教程页**：[https://deepclub.net/articles/gemini-3.8-flash-high-how-to-use-guide](https://deepclub.net/articles/gemini-3.8-flash-high-how-to-use-guide) ｜ **实时定价与规格**：[https://deepclub.net/pricing/gemini-3.8-flash-high](https://deepclub.net/pricing/gemini-3.8-flash-high)

## 实时定价快照

| 计费项 | 实时参考价格（以官网为准） | 单位 |
|---|---|---|
| 输入 | ¥0.36 | 元 / 百万 Token |
| 输出 | ¥1.8 | 元 / 百万 Token |
| 缓存读取 (`cache_read`) | ¥0.036 | 元 / 百万 Token |

> 实时价格、可用状态与规格详情见官网：[https://deepclub.net/pricing/gemini-3.8-flash-high](https://deepclub.net/pricing/gemini-3.8-flash-high)

| 参数 | 值 |
| --- | --- |
| 模型 ID | `gemini-3.8-flash-high` |
| 类型 | 文本 |
| 调用端点 | `POST /v1/chat/completions` |
| 推荐协议 | OpenAI Chat Completions（推荐） |
| 客户端与 SDK | Dify、Cherry Studio、OpenAI Python SDK |
| 计费方式 | 按百万 Token，实时价格见 [定价页](https://deepclub.net/pricing/gemini-3.8-flash-high) |

## Gemini 3.8 Flash High 适合做什么，不适合做什么？

适合高频工具调用、数据库检索和个人 AI 助手。

gemini-3.8-flash-high 是 Gemini Flash 系列的增强版，针对高频工具调用（Function Calling）与外部数据检索做了专项优化，适合作为 Agent 和个人助手的底座。

适合的场景：

- Dify 中带工具节点的 Agent 工作流
- 调用搜索、数据库等外部工具
- 需要较快响应的个人助手

不太适合：只做纯文本闲聊时，可与 gemini-3.8-flash 比较效果；具体成本以实时定价为准。

## gemini-3.8-flash-high 的 Base URL 和 API Key 怎么获取？

本文 OpenAI Python SDK 示例的 Base URL 为 `https://deepclub.net/v1`，Key 在控制台创建。客户端地址拼接规则可能不同，请按下方对应客户端的步骤填写。

1. 登录 [深云 DeepClub](https://deepclub.net/)，进入控制台的 **API 密钥** 页面。
2. 点击 **创建 API 密钥**，复制生成的 `sk-` 开头密钥。
3. 在客户端或代码中填写以下三项：

```text
Base URL: https://deepclub.net/v1
API Key:  sk-你的DeepClub密钥
Model:    gemini-3.8-flash-high
```

## gemini-3.8-flash-high 推荐哪个协议？

**首选 OpenAI Chat Completions：`POST /v1/chat/completions`。**

Google 原生 Gemini API 对原生功能有更完整的表达，但 DeepClub 当前此模型的销售目录只开放 Chat Completions。因此这里推荐实际可用的 Chat 接入，不要把提供商改为 Google 后调用未开放的 generateContent。

参考：[Google 的 OpenAI 兼容说明](https://ai.google.dev/gemini-api/docs/openai)。Google 原生建议不代表本站已开放同样的端点。

选择依据是模型官方接口、DeepClub 已发布端点和客户端实际能力。使用缓存、推理或工具调用前，请先用小请求验证；端点可用不代表提供联网搜索等厂商托管工具、保证缓存命中或在服务端保存会话。

## Gemini 3.8 Flash High 在 Dify 里怎么配置？

添加 OpenAI-API-compatible，端点填 deepclub.net/v1。

1. 在 Dify 中进入 **设置 → 模型供应商**，添加 **OpenAI-API-compatible**。
2. **模型类型** 选 LLM，**模型名称** 填 `gemini-3.8-flash-high`，**Completion mode** 选 Chat。
3. **API Key** 粘贴 DeepClub 密钥，**API endpoint URL** 填 `https://deepclub.net/v1`，若有 **API Type** 选项，选 **Chat Completions API**。
4. 保存后，在工作流的 LLM 节点中选择该模型。

## Gemini 3.8 Flash High 在 Cherry Studio 里怎么配置？

Cherry Studio v2.2.0 使用端点配置，模型选择 OpenAI Chat Completions。

1. 进入 **设置 → 模型服务**，添加自定义提供商，例如 DeepClub。
2. 为服务配置 **OpenAI Chat Completions** 端点，地址填 `https://deepclub.net/v1`，粘贴 DeepClub 密钥。
3. 添加完整模型 ID `gemini-3.8-flash-high`，在该模型的端点设置中仅选择上面的协议。
4. 保存并检查最终请求地址应为 `https://deepclub.net/v1/chat/completions`，然后在对话中选择该模型。

建议按协议分别建服务，避免多端点的优先级让模型走错协议。旧版设置与 v2.2.0 不同，若没有端点选择请升级或使用本文 SDK。

## 怎么用 Python 调用 gemini-3.8-flash-high（推荐协议）？

安装 `pip install -U openai`，并在本地设置环境变量 `DEEPCLUB_API_KEY`。

```python
import os
from openai import OpenAI

client = OpenAI(
    api_key=os.environ["DEEPCLUB_API_KEY"],
    base_url="https://deepclub.net/v1",
)

response = client.chat.completions.create(
    model="gemini-3.8-flash-high",
    messages=[{"role": "user", "content": "查询北京明天的天气并给出穿衣建议"}],
)
print(response.choices[0].message.content)
```

## 用 curl 调用 OpenAI Chat Completions（推荐）

```bash
curl https://deepclub.net/v1/chat/completions \
  -H "Authorization: Bearer $DEEPCLUB_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model":"gemini-3.8-flash-high","messages":[{"role":"user","content":"查询北京明天的天气并给出穿衣建议"}]}'
```

## Gemini 3.8 Flash High 怎么用 Function Calling？

在请求中传 tools 定义，模型返回 tool_calls。

在 `/v1/chat/completions` 请求中传入 `tools` 数组，每项使用 `type: "function"`，把 `name`、`description` 和 `parameters` JSON Schema 放在 `function` 对象内。模型需要调用工具时，会在 `message.tool_calls` 中返回函数名和参数。

先将完整 assistant 消息（含 tool_calls）保留在 messages 中。执行每个工具后，追加 `role: "tool"` 消息，用 `tool_call_id` 对应原调用的 `id`，把结果放入 content；全部结果回传后再请求下一轮，不能只发送孤立的工具结果。

## gemini-3.8-flash-high API 怎么计费？

按百万 Token 计费，输入和输出分别计价。

实时价格见 [定价页](https://deepclub.net/pricing/gemini-3.8-flash-high)。

- 计费单位：按百万 Token，价格可能调整，以 [定价页](https://deepclub.net/pricing/gemini-3.8-flash-high) 实时显示为准。
- 用量明细：在控制台 **使用日志** 中按请求查看消耗。
- 充值方式：支持信用卡、微信支付、支付宝等，以控制台充值页显示为准。

## 调用 gemini-3.8-flash-high 报 401 怎么办？

401 表示密钥无效或缺失，先检查 Key 是否完整。

- 确认 API Key 前后没有空格或换行，且令牌未被禁用、未过期。
- 请求头格式为 Authorization: Bearer sk-...。
- 如果返回 404，按上面的客户端步骤检查最终请求地址，避免重复或漏掉 /v1；模型 ID 必须完整匹配 `gemini-3.8-flash-high`。SDK 与桌面客户端的 Base URL 拼接规则不相同。

## 用 gemini-3.8-flash-high 时请求内容会被存储吗？

不会，深云 DeepClub 实行零留存（ZDR）政策。

深云 DeepClub 实行 Zero Data Retention (ZDR) 零留存政策：请求的输入和输出内容不做持久化存储，详见 [隐私政策](https://deepclub.net/privacy-policy)。

## 使用 Gemini 3.8 Flash High 还有哪些常见问题？

以下问答汇总了调用时最常见的问题。

### 调用 gemini-3.8-flash-high 返回 401 Unauthorized 怎么办？

401 表示密钥无效或缺失，先检查 API Key 是否复制完整、前后没有空格。请求头格式为 Authorization: Bearer sk-...。

### gemini-3.8-flash-high 用哪个接口和协议调用？

优先用 OpenAI Chat Completions，端点为 POST /v1/chat/completions，OpenAI Python SDK 的 Base URL 为 `https://deepclub.net/v1`。其他已开放端点 无；只有客户端限制或已有集成需要时才使用兼容路径。

### gemini-3.8-flash-high API 怎么计费？

按百万 Token 计费，输入和输出分别计价。实时价格见 https://deepclub.net/pricing/gemini-3.8-flash-high。

### Gemini 3.8 Flash High 怎么用 Function Calling？

在请求中传 tools 定义，模型返回 tool_calls。

## 多框架 & 主流平台快捷接入对照

| 工具 / 开发框架 | 选用模式 / Provider | 地址（Base URL / Endpoint） | 模型参数写法与注意事项 |
|---|---|---|---|
| **Cherry Studio (`2.2.0+`)** | OpenAI Chat Completions | `https://deepclub.net/v1` | 仅勾选 Chat 端点（勿错选 Responses 或未开放的原生 Gemini 端点） |
| **Vercel AI SDK** | `createOpenAI` | `https://deepclub.net/v1` | 务必用 **`openai.chat('gemini-3.8-flash-high')`**，不要用默认 `openai('gemini-3.8-flash-high')` |
| **OpenAI Agents SDK** | `OpenAIChatCompletionsModel` | `https://deepclub.net/v1` | 不要默认走 `/v1/responses` |
| **LangChain (Py)** | `ChatOpenAI` | `base_url="https://deepclub.net/v1"` | 默认 Chat Completions 即生效 |
| **Dify 兼容插件** | `OpenAI-API-compatible` | `https://deepclub.net/v1` | `Completion mode` 选 `Chat` |
| **沉浸式翻译** | OpenAI 自定义 | `https://deepclub.net/v1/chat/completions` | **必须填完整端点 URL** |

> **协议主选摘要**：`gemini-3.8-flash-high` 推荐优先使用 **OpenAI Chat Completions (`/v1/chat/completions`)**。完整客户端配置指南见 [平台与开发框架适配目录](../../guides/platforms/README.md) 与 [全量排障手册](../../guides/protocol-and-clients/error-troubleshooting-handbook.md)。
