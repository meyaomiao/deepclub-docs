# Claude Opus 5.5 怎么用？Claude Code、Python 与 Cursor 接入教程

> **一句话定位**：优先 Anthropic Messages，SDK Base URL 填 https://deepclub.net，模型名填 claude-opus-5-5。  
> **官网原生教程页**：[https://deepclub.net/articles/claude-opus-5-5-how-to-use-guide](https://deepclub.net/articles/claude-opus-5-5-how-to-use-guide) ｜ **实时定价与规格**：[https://deepclub.net/pricing/claude-opus-5-5](https://deepclub.net/pricing/claude-opus-5-5)

## 实时定价快照

| 计费项 | 实时参考价格（以官网为准） | 单位 |
|---|---|---|
| 输入 | ¥2 | 元 / 百万 Token |
| 输出 | ¥10 | 元 / 百万 Token |
| 缓存读取 (`cache_read`) | ¥0.1 | 元 / 百万 Token |
| 缓存写入 5m (`cache_write`) | ¥2.5 | 元 / 百万 Token |
| 缓存写入 1h (`cache_write_1h`) | ¥4 | 元 / 百万 Token |

> 实时价格、可用状态与规格详情见官网：[https://deepclub.net/pricing/claude-opus-5-5](https://deepclub.net/pricing/claude-opus-5-5)

| 参数 | 值 |
| --- | --- |
| 模型 ID | `claude-opus-5-5` |
| 类型 | 文本 |
| 调用端点 | `POST /v1/messages`、`POST /v1/chat/completions` |
| 推荐协议 | Anthropic Messages（推荐） |
| 客户端与 SDK | Claude Code、Cline、Cursor、Anthropic Python SDK |
| 计费方式 | 按百万 Token，实时价格见 [定价页](https://deepclub.net/pricing/claude-opus-5-5) |

## Claude Opus 5.5 适合做什么，不适合做什么？

适合跨文件代码重构、数学推导和长篇技术分析。

claude-opus-5-5 是 Claude 系列的高能力型号，适合需要深度推理的任务，例如跨工程的代码重构、数学证明和高精度技术分析；需要长时间自主执行的 Agent 任务，可另行评估 Fable 系列。

适合的场景：

- 在 Cursor 或 Claude Code 中做多文件重构、架构调整
- 推导复杂算法、审查数学证明步骤
- 阅读长篇论文或技术文档后给出结构化结论

不太适合：高频、低成本的简单问答或批量翻译。这类任务用 claude-sonnet-5-5 或 deepseek-v4.1-flash 成本更低。

## claude-opus-5-5 的 Base URL 和 API Key 怎么获取？

本文 Anthropic Python SDK 示例的 Base URL 为 `https://deepclub.net`，Key 在控制台创建。客户端地址拼接规则可能不同，请按下方对应客户端的步骤填写。

1. 登录 [深云 DeepClub](https://deepclub.net/)，进入控制台的 **API 密钥** 页面。
2. 点击 **创建 API 密钥**，复制生成的 `sk-` 开头密钥。
3. 在 Anthropic SDK 或 Claude Code 中填写以下三项（其他客户端按下文对应步骤填写，多为 `https://deepclub.net/v1`）：

```text
Base URL: https://deepclub.net
API Key:  sk-你的DeepClub密钥
Model:    claude-opus-5-5
```

## claude-opus-5-5 推荐哪个协议？

**首选 Anthropic Messages：`POST /v1/messages`。**

Claude 优先使用 Anthropic Messages，直接保留 content blocks、tool_use / tool_result、思考（thinking）和 cache_control 字段的原生结构；各型号接受的参数不同，见下文。OpenAI Chat 兼容接口适合已有 Chat 客户端，不应作为新集成的首选。

参考：[Anthropic 原生 API 与 OpenAI 兼容层的差异](https://platform.claude.com/docs/en/cli-sdks-libraries/libraries/openai-sdk)。

选择依据是模型官方接口、DeepClub 已发布端点和客户端实际能力。使用缓存、推理或工具调用前，请先用小请求验证；端点可用不代表提供联网搜索等厂商托管工具、保证缓存命中或在服务端保存会话。

## Claude Opus 5.5 在 Claude Code 里怎么配置？

ANTHROPIC_BASE_URL 设为 deepclub.net，不带 /v1。

1. 在终端设置环境变量：`export ANTHROPIC_BASE_URL=https://deepclub.net`。
2. 设置密钥：`export ANTHROPIC_AUTH_TOKEN=sk-你的DeepClub密钥`。
3. 指定模型：`export ANTHROPIC_MODEL=claude-opus-5-5`。
4. 在项目目录运行 `claude`，请求会走 `/v1/messages`。

Claude Code 会自己拼接 `/v1/messages`，所以 Base URL 不要带 `/v1`。请使用支持该型号的 Claude Code 版本（见 [Claude Code 接入页](https://deepclub.net/integrations/claude-code) 的版本表）。

## Claude Opus 5.5 在 Cline 里怎么配置？

用 OpenAI Compatible 配置，Base URL 带 /v1，Model ID 填完整型号。

> 这是该客户端的 Chat Completions 兼容配置。claude-opus-5-5 首选 Anthropic Messages；若客户端不能同时自定义协议、网关地址和完整模型 ID，建议使用下文 SDK 或支持该协议的客户端。只填写模型名不能保证使用首选协议。

1. 在 VS Code 侧边栏打开 Cline，点击齿轮进入设置。
2. **API Provider** 选择 **OpenAI Compatible**。
3. **Base URL** 填 `https://deepclub.net/v1`，**API Key** 粘贴 DeepClub 密钥。
4. **Model ID** 填 `claude-opus-5-5`，保存后即可在任务中使用。

这里使用 OpenAI Compatible 的 Chat 兼容配置，可以准确填写完整模型 ID；网关会为 Claude 5.x 忽略模型不支持的温度、Top P、Top K，并把推理强度转换为自适应思考。需要原生 Messages 时请使用 Claude Code 或本文 Anthropic SDK。

## Claude Opus 5.5 在 Cursor 里怎么配置？

覆盖 Base URL 为 deepclub.net/v1，再添加模型名。

> 这是客户端自动选择协议的自定义 OpenAI 入口。claude-opus-5-5 首选 Anthropic Messages；若客户端不能同时自定义协议、网关地址和完整模型 ID，建议使用下文 SDK 或支持该协议的客户端。只填写模型名不能保证使用首选协议。

1. 打开 Cursor 的 **Settings → Models**。
2. 在 **OpenAI API Key** 一栏粘贴 DeepClub 密钥。
3. 展开 **Override OpenAI Base URL**，填写 `https://deepclub.net/v1`。
4. 点击 **Add model**，输入 `claude-opus-5-5`，然后在 Chat 面板选择它。

Cursor 的 OpenAI 覆盖入口会按版本和模式发送 Chat Completions 或 Responses，不会发送 Anthropic Messages。保存后发一条短消息，在控制台使用日志中确认调用成功；若所选型号不支持该模式使用的接口，请改用本文 SDK 或可以明确选择协议的客户端。Tab 补全等功能使用 Cursor 自己的模型。

## 怎么用 Python 调用 claude-opus-5-5（推荐协议）？

安装 `pip install -U anthropic`，并在本地设置环境变量 `DEEPCLUB_API_KEY`。

```python
import os
from anthropic import Anthropic

client = Anthropic(
    api_key=os.environ["DEEPCLUB_API_KEY"],
    base_url="https://deepclub.net",
)

response = client.messages.create(
    model="claude-opus-5-5",
    max_tokens=4096,
    messages=[{"role": "user", "content": "请审查这段函数的边界条件并给出重构建议"}],
)
print("".join(block.text for block in response.content if block.type == "text"))
```

## 用 curl 调用 Anthropic Messages（推荐）

```bash
curl https://deepclub.net/v1/messages \
  -H "x-api-key: $DEEPCLUB_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "Content-Type: application/json" \
  -d '{"model":"claude-opus-5-5","max_tokens":4096,"messages":[{"role":"user","content":"请审查这段函数的边界条件并给出重构建议"}]}'
```

## 用 curl 调用 OpenAI Chat Completions（兼容选项）

仅在现有客户端或集成需要该协议时选用；切换时必须同步修改请求体和响应解析。

```bash
curl https://deepclub.net/v1/chat/completions \
  -H "Authorization: Bearer $DEEPCLUB_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model":"claude-opus-5-5","messages":[{"role":"user","content":"请审查这段函数的边界条件并给出重构建议"}]}'
```

## claude-opus-5-5 怎么开启思考？哪些参数不能用？

claude-opus-5-5 在 `/v1/messages` 上不接受非 1 的 `temperature`、`top_p`、`top_k`，也不接受 `thinking: {"type": "enabled", "budget_tokens": ...}`，会返回 400。需要思考时使用自适应思考，用 `output_config.effort` 控制强度（low / medium / high）：

```json
{
  "model": "claude-opus-5-5",
  "max_tokens": 4096,
  "thinking": {
    "type": "adaptive",
    "display": "summarized"
  },
  "output_config": {
    "effort": "medium"
  },
  "messages": [
    {
      "role": "user",
      "content": "请审查这段函数的边界条件并给出重构建议"
    }
  ]
}
```

如果客户端会在 Messages 请求里自动带温度、Top P 或思考预算，请先在客户端里关闭这些设置。走 `/v1/chat/completions` 时，网关会忽略这些采样参数，并把 `reasoning_effort` 转换为自适应思考。

## Claude Opus 5.5 和 Claude Sonnet 5.5 怎么选？

难题和大型重构选 Opus，日常编码和 Agent 循环选 Sonnet。

claude-opus-5-5 单次推理更深入，适合一次性解决复杂问题；claude-sonnet-5-5 在工具调用和多轮 Agent 任务中更均衡，单价也更低。

两者调用方式完全相同，切换只需修改 `model` 参数，可以先用 Sonnet 处理常规任务，遇到卡住的问题再换 Opus。

## claude-opus-5-5 API 怎么计费？

按百万 Token 计费，输入和输出分别计价。

实时价格见 [定价页](https://deepclub.net/pricing/claude-opus-5-5)。

- 计费单位：按百万 Token，价格可能调整，以 [定价页](https://deepclub.net/pricing/claude-opus-5-5) 实时显示为准。
- 用量明细：在控制台 **使用日志** 中按请求查看消耗。
- 充值方式：支持信用卡、微信支付、支付宝等，以控制台充值页显示为准。

## 调用 claude-opus-5-5 报 401 怎么办？

401 表示密钥无效或缺失，先检查 Key 是否完整。

- 确认 API Key 前后没有空格或换行，且令牌未被禁用、未过期。
- Messages 请求用 x-api-key: sk-... 并带 anthropic-version: 2023-06-01；Claude Code 的 ANTHROPIC_AUTH_TOKEN 使用网关支持的 Authorization: Bearer。OpenAI Chat Completions 请求使用 Authorization: Bearer sk-...。
- 如果返回 404，按上面的客户端步骤检查最终请求地址，避免重复或漏掉 /v1；模型 ID 必须完整匹配 `claude-opus-5-5`。Anthropic SDK、Claude Code 与桌面客户端的 Base URL 拼接规则不相同。

## 用 claude-opus-5-5 时请求内容会被存储吗？

不会，深云 DeepClub 实行零留存（ZDR）政策。

深云 DeepClub 实行 Zero Data Retention (ZDR) 零留存政策：请求的输入和输出内容不做持久化存储，详见 [隐私政策](https://deepclub.net/privacy-policy)。

## 使用 Claude Opus 5.5 还有哪些常见问题？

以下问答汇总了调用时最常见的问题。

### 调用 claude-opus-5-5 返回 401 Unauthorized 怎么办？

401 表示密钥无效或缺失，先检查 API Key 是否复制完整、前后没有空格。Messages 请求用 x-api-key: sk-... 并带 anthropic-version: 2023-06-01；Claude Code 的 ANTHROPIC_AUTH_TOKEN 使用网关支持的 Authorization: Bearer。OpenAI Chat Completions 请求使用 Authorization: Bearer sk-...。

### claude-opus-5-5 用哪个接口和协议调用？

优先用 Anthropic Messages，端点为 POST /v1/messages，Anthropic Python SDK 的 Base URL 为 https://deepclub.net。其他已开放端点 POST /v1/chat/completions；只有客户端限制或已有集成需要时才使用兼容路径。

### claude-opus-5-5 API 怎么计费？

按百万 Token 计费，输入和输出分别计价。实时价格见 https://deepclub.net/pricing/claude-opus-5-5。

### Claude Opus 5.5 和 Claude Sonnet 5.5 怎么选？

难题和大型重构选 Opus，日常编码和 Agent 循环选 Sonnet。

## 多框架 & 主流平台快捷接入对照

| 工具 / 开发框架 | 选用模式 / Provider | 地址（Base URL / Endpoint） | 模型参数写法与注意事项 |
|---|---|---|---|
| **Claude Code** | Anthropic 原生 | `ANTHROPIC_BASE_URL=https://deepclub.net`（**不带 `/v1`**） | 填完整 `claude-opus-5-5`；配合 `cache_control` 降本 |
| **Vercel AI SDK** | `createAnthropic` | `https://deepclub.net/v1`（**需带 `/v1`**） | `anthropic('claude-opus-5-5')`；`@ai-sdk/anthropic` 默认只追加 `/messages` |
| **LangChain (Py)** | `ChatAnthropic` | `base_url="https://deepclub.net"`（不带 `/v1`） | 不要传入非 1 的 `temperature` / `top_p` / `top_k` |
| **Dify 插件** | 官方 `Anthropic` (`0.3.41+`) | `https://deepclub.net` | 添加自定义模型 `claude-opus-5-5` |
| **Chatbox (`1.24.3+`)** | Claude API Compatible | `API Host: https://deepclub.net/v1` | Host 勿漏 `/v1`；保持默认采样参数 |
| **Zed Editor** | `anthropic_compatible` | `api_url: "https://deepclub.net"` | 支持 tools 与 prompt caching |

> **协议主选摘要**：`claude-opus-5-5` 推荐优先使用 **Anthropic Messages (`/v1/messages`)**。完整客户端配置指南见 [平台与开发框架适配目录](../../guides/platforms/README.md) 与 [全量排障手册](../../guides/protocol-and-clients/error-troubleshooting-handbook.md)。
