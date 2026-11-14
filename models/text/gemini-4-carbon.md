# Gemini 4 Carbon 怎么接入？Chat Completions 配置教程

> **一句话定位**：优先 OpenAI Chat Completions，SDK Base URL 填 https://deepclub.net/v1，模型名填 gemini-4-carbon。  
> **官网原生教程页**：[https://deepclub.net/articles/gemini-4-carbon-how-to-use-guide](https://deepclub.net/articles/gemini-4-carbon-how-to-use-guide) ｜ **实时定价与规格**：[https://deepclub.net/pricing/gemini-4-carbon](https://deepclub.net/pricing/gemini-4-carbon)

## 实时定价快照

| 计费项 | 实时参考价格（以官网为准） | 单位 |
|---|---|---|
| 输入 | ¥1.32 | 元 / 百万 Token |
| 输出 | ¥6.6 | 元 / 百万 Token |
| 缓存读取 (`cache_read`) | ¥0.066 | 元 / 百万 Token |

> 实时价格、可用状态与规格详情见官网：[https://deepclub.net/pricing/gemini-4-carbon](https://deepclub.net/pricing/gemini-4-carbon)

| 参数 | 值 |
| --- | --- |
| 模型 ID | `gemini-4-carbon` |
| 类型 | 文本 |
| 调用端点 | `POST /v1/chat/completions` |
| 推荐协议 | OpenAI Chat Completions（推荐） |
| 客户端与 SDK | Cherry Studio、Cline、OpenAI Python SDK |
| 计费方式 | 按百万 Token，实时价格见 [定价页](https://deepclub.net/pricing/gemini-4-carbon) |

## Gemini 4 Carbon 适合做什么，不适合做什么？

可用于文本分析、总结和代码问答，先验证目标任务。

gemini-4-carbon 已在 DeepClub 的文本模型目录上架。当前对外开放 Chat Completions，不能依据 Gemini 名称直接填写未发布的 Google 原生端点。

适合的场景：

- 文本总结与分析
- 代码问答
- 已有 OpenAI Chat 客户端中的多模型比较

不太适合：仅因名称相近就用于图片生成；需要生图时请选择专用图像型号。

## gemini-4-carbon 的 Base URL 和 API Key 怎么获取？

本文 OpenAI Python SDK 示例的 Base URL 为 `https://deepclub.net/v1`，Key 在控制台创建。客户端地址拼接规则可能不同，请按下方对应客户端的步骤填写。

1. 登录 [深云 DeepClub](https://deepclub.net/)，进入控制台的 **API 密钥** 页面。
2. 点击 **创建 API 密钥**，复制生成的 `sk-` 开头密钥。
3. 在客户端或代码中填写以下三项：

```text
Base URL: https://deepclub.net/v1
API Key:  sk-你的DeepClub密钥
Model:    gemini-4-carbon
```

## gemini-4-carbon 推荐哪个协议？

**首选 OpenAI Chat Completions：`POST /v1/chat/completions`。**

Google 原生 Gemini API 对原生功能有更完整的表达，但 DeepClub 当前此模型的销售目录只开放 Chat Completions。因此这里推荐实际可用的 Chat 接入，不要把提供商改为 Google 后调用未开放的 generateContent。

参考：[Google 的 OpenAI 兼容说明](https://ai.google.dev/gemini-api/docs/openai)。Google 原生建议不代表本站已开放同样的端点。

选择依据是模型官方接口、DeepClub 已发布端点和客户端实际能力。使用缓存、推理或工具调用前，请先用小请求验证；端点可用不代表提供联网搜索等厂商托管工具、保证缓存命中或在服务端保存会话。

## Gemini 4 Carbon 在 Cherry Studio 里怎么配置？

Cherry Studio v2.2.0 使用端点配置，模型选择 OpenAI Chat Completions。

1. 进入 **设置 → 模型服务**，添加自定义提供商，例如 DeepClub。
2. 为服务配置 **OpenAI Chat Completions** 端点，地址填 `https://deepclub.net/v1`，粘贴 DeepClub 密钥。
3. 添加完整模型 ID `gemini-4-carbon`，在该模型的端点设置中仅选择上面的协议。
4. 保存并检查最终请求地址应为 `https://deepclub.net/v1/chat/completions`，然后在对话中选择该模型。

建议按协议分别建服务，避免多端点的优先级让模型走错协议。旧版设置与 v2.2.0 不同，若没有端点选择请升级或使用本文 SDK。

## Gemini 4 Carbon 在 Cline 里怎么配置？

Provider 选 OpenAI Compatible，Base URL 带 /v1。

1. 在 VS Code 侧边栏打开 Cline，点击齿轮进入设置。
2. **API Provider** 选择 **OpenAI Compatible**。
3. **Base URL** 填 `https://deepclub.net/v1`，**API Key** 粘贴 DeepClub 密钥。
4. **Model ID** 填 `gemini-4-carbon`，保存后即可在任务中使用。

## 怎么用 Python 调用 gemini-4-carbon（推荐协议）？

安装 `pip install -U openai`，并在本地设置环境变量 `DEEPCLUB_API_KEY`。

```python
import os
from openai import OpenAI

client = OpenAI(
    api_key=os.environ["DEEPCLUB_API_KEY"],
    base_url="https://deepclub.net/v1",
)

response = client.chat.completions.create(
    model="gemini-4-carbon",
    messages=[{"role": "user", "content": "总结这份需求文档中的约束条件和待确认问题"}],
)
print(response.choices[0].message.content)
```

## 用 curl 调用 OpenAI Chat Completions（推荐）

```bash
curl https://deepclub.net/v1/chat/completions \
  -H "Authorization: Bearer $DEEPCLUB_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model":"gemini-4-carbon","messages":[{"role":"user","content":"总结这份需求文档中的约束条件和待确认问题"}]}'
```

## Gemini 4 Carbon 能在这里直接用 Google SDK 吗？

当前发布目录只有 Chat Completions，请使用对应的 OpenAI 兼容客户端。

将 Base URL 设为 `https://deepclub.net/v1`，模型名填写 `gemini-4-carbon`。

Google 原生功能和本站开放接口不是同一个范围。需要特定原生能力时，应先确认本站目录是否开放，不要只替换 URL。

## gemini-4-carbon API 怎么计费？

按百万 Token 计费，输入和输出分别计价。

实时价格见 [定价页](https://deepclub.net/pricing/gemini-4-carbon)。

- 计费单位：按百万 Token，价格可能调整，以 [定价页](https://deepclub.net/pricing/gemini-4-carbon) 实时显示为准。
- 用量明细：在控制台 **使用日志** 中按请求查看消耗。
- 充值方式：支持信用卡、微信支付、支付宝等，以控制台充值页显示为准。

## 调用 gemini-4-carbon 报 401 怎么办？

401 表示密钥无效或缺失，先检查 Key 是否完整。

- 确认 API Key 前后没有空格或换行，且令牌未被禁用、未过期。
- 请求头格式为 Authorization: Bearer sk-...。
- 如果返回 404，按上面的客户端步骤检查最终请求地址，避免重复或漏掉 /v1；模型 ID 必须完整匹配 `gemini-4-carbon`。SDK 与桌面客户端的 Base URL 拼接规则不相同。

## 用 gemini-4-carbon 时请求内容会被存储吗？

不会，深云 DeepClub 实行零留存（ZDR）政策。

深云 DeepClub 实行 Zero Data Retention (ZDR) 零留存政策：请求的输入和输出内容不做持久化存储，详见 [隐私政策](https://deepclub.net/privacy-policy)。

## 使用 Gemini 4 Carbon 还有哪些常见问题？

以下问答汇总了调用时最常见的问题。

### 调用 gemini-4-carbon 返回 401 Unauthorized 怎么办？

401 表示密钥无效或缺失，先检查 API Key 是否复制完整、前后没有空格。请求头格式为 Authorization: Bearer sk-...。

### gemini-4-carbon 用哪个接口和协议调用？

优先用 OpenAI Chat Completions，端点为 POST /v1/chat/completions，OpenAI Python SDK 的 Base URL 为 `https://deepclub.net/v1`。其他已开放端点 无；只有客户端限制或已有集成需要时才使用兼容路径。

### gemini-4-carbon API 怎么计费？

按百万 Token 计费，输入和输出分别计价。实时价格见 https://deepclub.net/pricing/gemini-4-carbon。

### Gemini 4 Carbon 能在这里直接用 Google SDK 吗？

当前发布目录只有 Chat Completions，请使用对应的 OpenAI 兼容客户端。

## 多框架 & 主流平台快捷接入对照

| 工具 / 开发框架 | 选用模式 / Provider | 地址（Base URL / Endpoint） | 模型参数写法与注意事项 |
|---|---|---|---|
| **Cherry Studio (`2.2.0+`)** | OpenAI Chat Completions | `https://deepclub.net/v1` | 仅勾选 Chat 端点（勿错选 Responses 或未开放的原生 Gemini 端点） |
| **Vercel AI SDK** | `createOpenAI` | `https://deepclub.net/v1` | 务必用 **`openai.chat('gemini-4-carbon')`**，不要用默认 `openai('gemini-4-carbon')` |
| **OpenAI Agents SDK** | `OpenAIChatCompletionsModel` | `https://deepclub.net/v1` | 不要默认走 `/v1/responses` |
| **LangChain (Py)** | `ChatOpenAI` | `base_url="https://deepclub.net/v1"` | 默认 Chat Completions 即生效 |
| **Dify 兼容插件** | `OpenAI-API-compatible` | `https://deepclub.net/v1` | `Completion mode` 选 `Chat` |
| **沉浸式翻译** | OpenAI 自定义 | `https://deepclub.net/v1/chat/completions` | **必须填完整端点 URL** |

> **协议主选摘要**：`gemini-4-carbon` 推荐优先使用 **OpenAI Chat Completions (`/v1/chat/completions`)**。完整客户端配置指南见 [平台与开发框架适配目录](../../guides/platforms/README.md) 与 [全量排障手册](../../guides/protocol-and-clients/error-troubleshooting-handbook.md)。
