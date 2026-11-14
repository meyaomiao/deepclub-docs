# Claude Sonnet 5.5 怎么用？Cline、Roo Code、Claude Code 配置教程

> **一句话定位**：优先 Anthropic Messages，SDK Base URL 填 https://deepclub.net，模型名填 claude-sonnet-5-5。  
> **官网原生教程页**：[https://deepclub.net/articles/claude-sonnet-5-5-how-to-use-guide](https://deepclub.net/articles/claude-sonnet-5-5-how-to-use-guide) ｜ **实时定价与规格**：[https://deepclub.net/pricing/claude-sonnet-5-5](https://deepclub.net/pricing/claude-sonnet-5-5)

## 实时定价快照

| 计费项 | 实时参考价格（以官网为准） | 单位 |
|---|---|---|
| 输入 | ¥1 | 元 / 百万 Token |
| 输出 | ¥5 | 元 / 百万 Token |
| 缓存读取 (`cache_read`) | ¥0.1 | 元 / 百万 Token |
| 缓存写入 5m (`cache_write`) | ¥1.25 | 元 / 百万 Token |
| 缓存写入 1h (`cache_write_1h`) | ¥2 | 元 / 百万 Token |

> 实时价格、可用状态与规格详情见官网：[https://deepclub.net/pricing/claude-sonnet-5-5](https://deepclub.net/pricing/claude-sonnet-5-5)

| 参数 | 值 |
| --- | --- |
| 模型 ID | `claude-sonnet-5-5` |
| 类型 | 文本 |
| 调用端点 | `POST /v1/messages`、`POST /v1/chat/completions` |
| 推荐协议 | Anthropic Messages（推荐） |
| 客户端与 SDK | Cline、Roo Code、Claude Code、Anthropic Python SDK |
| 计费方式 | 按百万 Token，实时价格见 [定价页](https://deepclub.net/pricing/claude-sonnet-5-5) |

## Claude Sonnet 5.5 适合做什么，不适合做什么？

适合编程 Agent、工具调用和多步骤终端调试。

claude-sonnet-5-5 在代码生成能力与成本之间取得平衡，擅长函数调用（Tool Calling）和多步骤终端调试，适合长时间连续运行的编程 Agent。

适合的场景：

- 在 Cline、Roo Code 中让 Agent 自主读写文件、运行命令
- 通过 Function Calling 串联外部工具和 API
- 日常写代码、修 Bug、补测试

不太适合：极难的数学证明或需要最深推理的架构决策，这类任务可换 claude-opus-5-5。

## claude-sonnet-5-5 的 Base URL 和 API Key 怎么获取？

本文 Anthropic Python SDK 示例的 Base URL 为 `https://deepclub.net`，Key 在控制台创建。客户端地址拼接规则可能不同，请按下方对应客户端的步骤填写。

1. 登录 [深云 DeepClub](https://deepclub.net/)，进入控制台的 **API 密钥** 页面。
2. 点击 **创建 API 密钥**，复制生成的 `sk-` 开头密钥。
3. 在 Anthropic SDK 或 Claude Code 中填写以下三项（其他客户端按下文对应步骤填写，多为 `https://deepclub.net/v1`）：

```text
Base URL: https://deepclub.net
API Key:  sk-你的DeepClub密钥
Model:    claude-sonnet-5-5
```

## claude-sonnet-5-5 推荐哪个协议？

**首选 Anthropic Messages：`POST /v1/messages`。**

Claude 优先使用 Anthropic Messages，直接保留 content blocks、tool_use / tool_result、思考（thinking）和 cache_control 字段的原生结构；各型号接受的参数不同，见下文。OpenAI Chat 兼容接口适合已有 Chat 客户端，不应作为新集成的首选。

参考：[Anthropic 原生 API 与 OpenAI 兼容层的差异](https://platform.claude.com/docs/en/cli-sdks-libraries/libraries/openai-sdk)。

选择依据是模型官方接口、DeepClub 已发布端点和客户端实际能力。使用缓存、推理或工具调用前，请先用小请求验证；端点可用不代表提供联网搜索等厂商托管工具、保证缓存命中或在服务端保存会话。

## Claude Sonnet 5.5 在 Cline 里怎么配置？

用 OpenAI Compatible 配置，Base URL 带 /v1，Model ID 填完整型号。

> 这是该客户端的 Chat Completions 兼容配置。claude-sonnet-5-5 首选 Anthropic Messages；若客户端不能同时自定义协议、网关地址和完整模型 ID，建议使用下文 SDK 或支持该协议的客户端。只填写模型名不能保证使用首选协议。

1. 在 VS Code 侧边栏打开 Cline，点击齿轮进入设置。
2. **API Provider** 选择 **OpenAI Compatible**。
3. **Base URL** 填 `https://deepclub.net/v1`，**API Key** 粘贴 DeepClub 密钥。
4. **Model ID** 填 `claude-sonnet-5-5`，保存后即可在任务中使用。

这里使用 OpenAI Compatible 的 Chat 兼容配置，可以准确填写完整模型 ID；网关会为 Claude 5.x 忽略模型不支持的温度、Top P、Top K，并把推理强度转换为自适应思考。需要原生 Messages 时请使用 Claude Code 或本文 Anthropic SDK。

## Claude Sonnet 5.5 在 Roo Code 里怎么配置？

用 OpenAI Compatible 配置，Base URL 带 /v1，Model ID 填完整型号。

> 这是该客户端的 Chat Completions 兼容配置。claude-sonnet-5-5 首选 Anthropic Messages；若客户端不能同时自定义协议、网关地址和完整模型 ID，建议使用下文 SDK 或支持该协议的客户端。只填写模型名不能保证使用首选协议。

1. 在 VS Code 中打开 Roo Code，进入 **Settings → Providers**。
2. **API Provider** 选择 **OpenAI Compatible**。
3. **Base URL** 填 `https://deepclub.net/v1`，粘贴 DeepClub 密钥。
4. **Model** 填 `claude-sonnet-5-5`，保存配置。

这里使用 OpenAI Compatible 的 Chat 兼容配置，可以准确填写完整模型 ID；网关会为 Claude 5.x 忽略模型不支持的温度、Top P、Top K，并把推理强度转换为自适应思考。需要原生 Messages 时请使用 Claude Code 或本文 Anthropic SDK。

## Claude Sonnet 5.5 在 Claude Code 里怎么配置？

ANTHROPIC_BASE_URL 设为 deepclub.net，不带 /v1。

1. 在终端设置环境变量：`export ANTHROPIC_BASE_URL=https://deepclub.net`。
2. 设置密钥：`export ANTHROPIC_AUTH_TOKEN=sk-你的DeepClub密钥`。
3. 指定模型：`export ANTHROPIC_MODEL=claude-sonnet-5-5`。
4. 在项目目录运行 `claude`，请求会走 `/v1/messages`。

Claude Code 会自己拼接 `/v1/messages`，所以 Base URL 不要带 `/v1`。请使用支持该型号的 Claude Code 版本（见 [Claude Code 接入页](https://deepclub.net/integrations/claude-code) 的版本表）。

## 怎么用 Python 调用 claude-sonnet-5-5（推荐协议）？

安装 `pip install -U anthropic`，并在本地设置环境变量 `DEEPCLUB_API_KEY`。

```python
import os
from anthropic import Anthropic

client = Anthropic(
    api_key=os.environ["DEEPCLUB_API_KEY"],
    base_url="https://deepclub.net",
)

response = client.messages.create(
    model="claude-sonnet-5-5",
    max_tokens=4096,
    messages=[{"role": "user", "content": "为这个函数补充单元测试并说明覆盖了哪些边界"}],
)
print("".join(block.text for block in response.content if block.type == "text"))
```

## 用 curl 调用 Anthropic Messages（推荐）

```bash
curl https://deepclub.net/v1/messages \
  -H "x-api-key: $DEEPCLUB_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "Content-Type: application/json" \
  -d '{"model":"claude-sonnet-5-5","max_tokens":4096,"messages":[{"role":"user","content":"为这个函数补充单元测试并说明覆盖了哪些边界"}]}'
```

## 用 curl 调用 OpenAI Chat Completions（兼容选项）

仅在现有客户端或集成需要该协议时选用；切换时必须同步修改请求体和响应解析。

```bash
curl https://deepclub.net/v1/chat/completions \
  -H "Authorization: Bearer $DEEPCLUB_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model":"claude-sonnet-5-5","messages":[{"role":"user","content":"为这个函数补充单元测试并说明覆盖了哪些边界"}]}'
```

## claude-sonnet-5-5 怎么开启思考？哪些参数不能用？

claude-sonnet-5-5 在 `/v1/messages` 上不接受非 1 的 `temperature`、`top_p`、`top_k`，也不接受 `thinking: {"type": "enabled", "budget_tokens": ...}`，会返回 400。需要思考时使用自适应思考，用 `output_config.effort` 控制强度（low / medium / high）：

```json
{
  "model": "claude-sonnet-5-5",
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
      "content": "为这个函数补充单元测试并说明覆盖了哪些边界"
    }
  ]
}
```

如果客户端会在 Messages 请求里自动带温度、Top P 或思考预算，请先在客户端里关闭这些设置。走 `/v1/chat/completions` 时，网关会忽略这些采样参数，并把 `reasoning_effort` 转换为自适应思考。

## Claude Sonnet 5.5 在 Cline 里能用工具调用吗？

可以尝试，先用一个能检查结果的小任务验证所用 Cline 版本和配置。

Cline 负责执行工具，模型通过所选协议提出调用：原生 Messages 用 tool_use / tool_result，OpenAI Compatible 用 tool_calls。按上文 OpenAI Compatible 配置时，网关会忽略 Claude 5.x 不支持的采样参数。

Agent 任务通常会多轮调用、上下文较长，Token 消耗比普通对话高，可以在控制台使用日志中查看每轮消耗。

## claude-sonnet-5-5 API 怎么计费？

按百万 Token 计费，输入和输出分别计价。

实时价格见 [定价页](https://deepclub.net/pricing/claude-sonnet-5-5)。

- 计费单位：按百万 Token，价格可能调整，以 [定价页](https://deepclub.net/pricing/claude-sonnet-5-5) 实时显示为准。
- 用量明细：在控制台 **使用日志** 中按请求查看消耗。
- 充值方式：支持信用卡、微信支付、支付宝等，以控制台充值页显示为准。

## 调用 claude-sonnet-5-5 报 401 怎么办？

401 表示密钥无效或缺失，先检查 Key 是否完整。

- 确认 API Key 前后没有空格或换行，且令牌未被禁用、未过期。
- Messages 请求用 x-api-key: sk-... 并带 anthropic-version: 2023-06-01；Claude Code 的 ANTHROPIC_AUTH_TOKEN 使用网关支持的 Authorization: Bearer。OpenAI Chat Completions 请求使用 Authorization: Bearer sk-...。
- 如果返回 404，按上面的客户端步骤检查最终请求地址，避免重复或漏掉 /v1；模型 ID 必须完整匹配 `claude-sonnet-5-5`。Anthropic SDK、Claude Code 与桌面客户端的 Base URL 拼接规则不相同。

## 用 claude-sonnet-5-5 时请求内容会被存储吗？

不会，深云 DeepClub 实行零留存（ZDR）政策。

深云 DeepClub 实行 Zero Data Retention (ZDR) 零留存政策：请求的输入和输出内容不做持久化存储，详见 [隐私政策](https://deepclub.net/privacy-policy)。

## 使用 Claude Sonnet 5.5 还有哪些常见问题？

以下问答汇总了调用时最常见的问题。

### 调用 claude-sonnet-5-5 返回 401 Unauthorized 怎么办？

401 表示密钥无效或缺失，先检查 API Key 是否复制完整、前后没有空格。Messages 请求用 x-api-key: sk-... 并带 anthropic-version: 2023-06-01；Claude Code 的 ANTHROPIC_AUTH_TOKEN 使用网关支持的 Authorization: Bearer。OpenAI Chat Completions 请求使用 Authorization: Bearer sk-...。

### claude-sonnet-5-5 用哪个接口和协议调用？

优先用 Anthropic Messages，端点为 POST /v1/messages，Anthropic Python SDK 的 Base URL 为 https://deepclub.net。其他已开放端点 POST /v1/chat/completions；只有客户端限制或已有集成需要时才使用兼容路径。

### claude-sonnet-5-5 API 怎么计费？

按百万 Token 计费，输入和输出分别计价。实时价格见 https://deepclub.net/pricing/claude-sonnet-5-5。

### Claude Sonnet 5.5 在 Cline 里能用工具调用吗？

可以尝试，先用一个能检查结果的小任务验证所用 Cline 版本和配置。

## 多框架 & 主流平台快捷接入对照

| 工具 / 开发框架 | 选用模式 / Provider | 地址（Base URL / Endpoint） | 模型参数写法与注意事项 |
|---|---|---|---|
| **Claude Code** | Anthropic 原生 | `ANTHROPIC_BASE_URL=https://deepclub.net`（**不带 `/v1`**） | 填完整 `claude-sonnet-5-5`；配合 `cache_control` 降本 |
| **Vercel AI SDK** | `createAnthropic` | `https://deepclub.net/v1`（**需带 `/v1`**） | `anthropic('claude-sonnet-5-5')`；`@ai-sdk/anthropic` 默认只追加 `/messages` |
| **LangChain (Py)** | `ChatAnthropic` | `base_url="https://deepclub.net"`（不带 `/v1`） | 不要传入非 1 的 `temperature` / `top_p` / `top_k` |
| **Dify 插件** | 官方 `Anthropic` (`0.3.41+`) | `https://deepclub.net` | 添加自定义模型 `claude-sonnet-5-5` |
| **Chatbox (`1.24.3+`)** | Claude API Compatible | `API Host: https://deepclub.net/v1` | Host 勿漏 `/v1`；保持默认采样参数 |
| **Zed Editor** | `anthropic_compatible` | `api_url: "https://deepclub.net"` | 支持 tools 与 prompt caching |

> **协议主选摘要**：`claude-sonnet-5-5` 推荐优先使用 **Anthropic Messages (`/v1/messages`)**。完整客户端配置指南见 [平台与开发框架适配目录](../../guides/platforms/README.md) 与 [全量排障手册](../../guides/protocol-and-clients/error-troubleshooting-handbook.md)。
