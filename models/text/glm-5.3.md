# GLM-5.3 怎么用？中文合同审查与财报分析接入教程

> **一句话定位**：优先 OpenAI Chat Completions，SDK Base URL 填 https://deepclub.net/v1，模型名填 glm-5.3。  
> **官网原生教程页**：[https://deepclub.net/articles/glm-5-3-how-to-use-guide](https://deepclub.net/articles/glm-5-3-how-to-use-guide) ｜ **实时定价与规格**：[https://deepclub.net/pricing/glm-5.3](https://deepclub.net/pricing/glm-5.3)

## 实时定价快照

| 计费项 | 实时参考价格（以官网为准） | 单位 |
|---|---|---|
| 输入 | ¥4.4 | 元 / 百万 Token |
| 输出 | ¥15.4 | 元 / 百万 Token |
| 缓存读取 (`cache_read`) | ¥1.1 | 元 / 百万 Token |

> 实时价格、可用状态与规格详情见官网：[https://deepclub.net/pricing/glm-5.3](https://deepclub.net/pricing/glm-5.3)

| 参数 | 值 |
| --- | --- |
| 模型 ID | `glm-5.3` |
| 类型 | 文本 |
| 调用端点 | `POST /v1/chat/completions`、`POST /v1/responses` |
| 推荐协议 | OpenAI Chat Completions（推荐） |
| 客户端与 SDK | Cherry Studio、Dify、OpenAI Python SDK |
| 计费方式 | 按百万 Token，实时价格见 [定价页](https://deepclub.net/pricing/glm-5.3) |

## GLM-5.3 适合做什么，不适合做什么？

适合中文合同审查、合规文本和公文写作。

glm-5.3 是智谱 AI 的旗舰模型，针对中国本土商业环境、法律合规文本和公文格式做了大规模强化。

适合的场景：

- 识别中文合同中的违约、对赌等风险条款
- 按公文格式起草通知、报告
- 财报中的关键数据提取与解读

不太适合：高并发、低预算的批量任务，可选 glm-5.3-flash。

## glm-5.3 的 Base URL 和 API Key 怎么获取？

本文 OpenAI Python SDK 示例的 Base URL 为 `https://deepclub.net/v1`，Key 在控制台创建。客户端地址拼接规则可能不同，请按下方对应客户端的步骤填写。

1. 登录 [深云 DeepClub](https://deepclub.net/)，进入控制台的 **API 密钥** 页面。
2. 点击 **创建 API 密钥**，复制生成的 `sk-` 开头密钥。
3. 在客户端或代码中填写以下三项：

```text
Base URL: https://deepclub.net/v1
API Key:  sk-你的DeepClub密钥
Model:    glm-5.3
```

## glm-5.3 推荐哪个协议？

**首选 OpenAI Chat Completions：`POST /v1/chat/completions`。**

GLM 的标准接入与模型专有字段以 Chat Completions 为基础，本文优先沿用这条路径。当前也开放 Responses，已有 Responses 客户端可选用；支持该端点不等于所有原生特性都完全相同。

选择依据是模型官方接口、DeepClub 已发布端点和客户端实际能力。使用缓存、推理或工具调用前，请先用小请求验证；端点可用不代表提供联网搜索等厂商托管工具、保证缓存命中或在服务端保存会话。

## GLM-5.3 在 Cherry Studio 里怎么配置？

Cherry Studio v2.2.0 使用端点配置，模型选择 OpenAI Chat Completions。

1. 进入 **设置 → 模型服务**，添加自定义提供商，例如 DeepClub。
2. 为服务配置 **OpenAI Chat Completions** 端点，地址填 `https://deepclub.net/v1`，粘贴 DeepClub 密钥。
3. 添加完整模型 ID `glm-5.3`，在该模型的端点设置中仅选择上面的协议。
4. 保存并检查最终请求地址应为 `https://deepclub.net/v1/chat/completions`，然后在对话中选择该模型。

建议按协议分别建服务，避免多端点的优先级让模型走错协议。旧版设置与 v2.2.0 不同，若没有端点选择请升级或使用本文 SDK。

## GLM-5.3 在 Dify 里怎么配置？

添加 OpenAI-API-compatible，端点填 deepclub.net/v1。

1. 在 Dify 中进入 **设置 → 模型供应商**，添加 **OpenAI-API-compatible**。
2. **模型类型** 选 LLM，**模型名称** 填 `glm-5.3`，**Completion mode** 选 Chat。
3. **API Key** 粘贴 DeepClub 密钥，**API endpoint URL** 填 `https://deepclub.net/v1`，若有 **API Type** 选项，选 **Chat Completions API**。
4. 保存后，在工作流的 LLM 节点中选择该模型。

## 怎么用 Python 调用 glm-5.3（推荐协议）？

安装 `pip install -U openai`，并在本地设置环境变量 `DEEPCLUB_API_KEY`。

```python
import os
from openai import OpenAI

client = OpenAI(
    api_key=os.environ["DEEPCLUB_API_KEY"],
    base_url="https://deepclub.net/v1",
)

response = client.chat.completions.create(
    model="glm-5.3",
    messages=[{"role": "user", "content": "列出这份合同中对乙方不利的条款"}],
)
print(response.choices[0].message.content)
```

## 用 curl 调用 OpenAI Chat Completions（推荐）

```bash
curl https://deepclub.net/v1/chat/completions \
  -H "Authorization: Bearer $DEEPCLUB_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model":"glm-5.3","messages":[{"role":"user","content":"列出这份合同中对乙方不利的条款"}]}'
```

## 用 curl 调用 OpenAI Responses（兼容选项）

仅在现有客户端或集成需要该协议时选用；切换时必须同步修改请求体和响应解析。

```bash
curl https://deepclub.net/v1/responses \
  -H "Authorization: Bearer $DEEPCLUB_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model":"glm-5.3","input":"列出这份合同中对乙方不利的条款","store":false}'
```

## GLM-5.3 审查合同时怎么写提示词？

先给角色和审查维度，再贴合同原文。

可以这样写：你是企业法务，请从付款条件、违约责任、解除条件、争议解决四个方面审查以下合同，逐条列出风险和修改建议。然后附上合同全文。

模型输出仅供参考，涉及重大法律决策时仍需专业律师复核。

## glm-5.3 API 怎么计费？

按百万 Token 计费，输入和输出分别计价。

实时价格见 [定价页](https://deepclub.net/pricing/glm-5.3)。

- 计费单位：按百万 Token，价格可能调整，以 [定价页](https://deepclub.net/pricing/glm-5.3) 实时显示为准。
- 用量明细：在控制台 **使用日志** 中按请求查看消耗。
- 充值方式：支持信用卡、微信支付、支付宝等，以控制台充值页显示为准。

## 调用 glm-5.3 报 401 怎么办？

401 表示密钥无效或缺失，先检查 Key 是否完整。

- 确认 API Key 前后没有空格或换行，且令牌未被禁用、未过期。
- 请求头格式为 Authorization: Bearer sk-...。
- 如果返回 404，按上面的客户端步骤检查最终请求地址，避免重复或漏掉 /v1；模型 ID 必须完整匹配 `glm-5.3`。SDK 与桌面客户端的 Base URL 拼接规则不相同。

## 用 glm-5.3 时请求内容会被存储吗？

不会，深云 DeepClub 实行零留存（ZDR）政策。

深云 DeepClub 实行 Zero Data Retention (ZDR) 零留存政策：请求的输入和输出内容不做持久化存储，详见 [隐私政策](https://deepclub.net/privacy-policy)。

## 使用 GLM-5.3 还有哪些常见问题？

以下问答汇总了调用时最常见的问题。

### 调用 glm-5.3 返回 401 Unauthorized 怎么办？

401 表示密钥无效或缺失，先检查 API Key 是否复制完整、前后没有空格。请求头格式为 Authorization: Bearer sk-...。

### glm-5.3 用哪个接口和协议调用？

优先用 OpenAI Chat Completions，端点为 POST /v1/chat/completions，OpenAI Python SDK 的 Base URL 为 `https://deepclub.net/v1`。其他已开放端点 POST /v1/responses；只有客户端限制或已有集成需要时才使用兼容路径。

### glm-5.3 API 怎么计费？

按百万 Token 计费，输入和输出分别计价。实时价格见 https://deepclub.net/pricing/glm-5.3。

### GLM-5.3 审查合同时怎么写提示词？

先给角色和审查维度，再贴合同原文。

## 多框架 & 主流平台快捷接入对照

| 工具 / 开发框架 | 选用模式 / Provider | 地址（Base URL / Endpoint） | 模型参数写法与注意事项 |
|---|---|---|---|
| **Cherry Studio (`2.2.0+`)** | OpenAI Chat Completions | `https://deepclub.net/v1` | 仅勾选 Chat 端点（GLM 亦可按需测试 `/v1/responses`） |
| **Vercel AI SDK** | `createOpenAI` | `https://deepclub.net/v1` | 务必用 **`openai.chat('glm-5.3')`**，不要用默认 `openai('glm-5.3')` |
| **OpenAI Agents SDK** | `OpenAIChatCompletionsModel` | `https://deepclub.net/v1` | 不要默认走 `/v1/responses` |
| **LangChain (Py)** | `ChatOpenAI` | `base_url="https://deepclub.net/v1"` | 默认 Chat Completions 即生效 |
| **Dify 兼容插件** | `OpenAI-API-compatible` | `https://deepclub.net/v1` | `Completion mode` 选 `Chat` |
| **沉浸式翻译** | OpenAI 自定义 | `https://deepclub.net/v1/chat/completions` | **必须填完整端点 URL** |

> **协议主选摘要**：`glm-5.3` 推荐优先使用 **OpenAI Chat Completions (`/v1/chat/completions`)**。完整客户端配置指南见 [平台与开发框架适配目录](../../guides/platforms/README.md) 与 [全量排障手册](../../guides/protocol-and-clients/error-troubleshooting-handbook.md)。
