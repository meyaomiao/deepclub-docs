# Claude Fable 5.1 怎么用？Claude Code 长任务与 Messages 接入教程

> **一句话定位**：优先 Anthropic Messages，SDK Base URL 填 https://deepclub.net，模型名填 claude-fable-5-1。  
> **官网原生教程页**：[https://deepclub.net/articles/claude-fable-5-1-how-to-use-guide](https://deepclub.net/articles/claude-fable-5-1-how-to-use-guide) ｜ **实时定价与规格**：[https://deepclub.net/pricing/claude-fable-5-1](https://deepclub.net/pricing/claude-fable-5-1)

## 实时定价快照

| 计费项 | 实时参考价格（以官网为准） | 单位 |
|---|---|---|
| 输入 | ¥8.5 | 元 / 百万 Token |
| 输出 | ¥42.5 | 元 / 百万 Token |
| 缓存读取 (`cache_read`) | ¥0.2125 | 元 / 百万 Token |
| 缓存写入 5m (`cache_write`) | ¥10.625 | 元 / 百万 Token |
| 缓存写入 1h (`cache_write_1h`) | ¥17 | 元 / 百万 Token |

> 实时价格、可用状态与规格详情见官网：[https://deepclub.net/pricing/claude-fable-5-1](https://deepclub.net/pricing/claude-fable-5-1)

| 参数 | 值 |
| --- | --- |
| 模型 ID | `claude-fable-5-1` |
| 类型 | 文本 |
| 调用端点 | `POST /v1/messages`、`POST /v1/chat/completions` |
| 推荐协议 | Anthropic Messages（推荐） |
| 客户端与 SDK | Claude Code、Anthropic Python SDK |
| 计费方式 | 按百万 Token，实时价格见 [定价页](https://deepclub.net/pricing/claude-fable-5-1) |

## Claude Fable 5.1 适合做什么，不适合做什么？

适合复杂编码、根因调查和持续多轮的 Agent 任务。

Anthropic 的 Claude Code 文档把 Fable 定位为处理复杂、长时间自主任务的模型。接入时优先使用 Messages，保留原生工具和内容块结构。

适合的场景：

- 跨文件代码分析与修改
- 故障根因调查与多轮验证
- 需要长时间保持任务上下文的 Agent 工作

不太适合：只需要短回复的高频简单任务；应按真实任务和实时价格比较是否值得使用。

## claude-fable-5-1 的 Base URL 和 API Key 怎么获取？

本文 Anthropic Python SDK 示例的 Base URL 为 `https://deepclub.net`，Key 在控制台创建。客户端地址拼接规则可能不同，请按下方对应客户端的步骤填写。

1. 登录 [深云 DeepClub](https://deepclub.net/)，进入控制台的 **API 密钥** 页面。
2. 点击 **创建 API 密钥**，复制生成的 `sk-` 开头密钥。
3. 在 Anthropic SDK 或 Claude Code 中填写以下三项（其他客户端按下文对应步骤填写，多为 `https://deepclub.net/v1`）：

```text
Base URL: https://deepclub.net
API Key:  sk-你的DeepClub密钥
Model:    claude-fable-5-1
```

## claude-fable-5-1 推荐哪个协议？

**首选 Anthropic Messages：`POST /v1/messages`。**

Claude 优先使用 Anthropic Messages，直接保留 content blocks、tool_use / tool_result、思考（thinking）和 cache_control 字段的原生结构；各型号接受的参数不同，见下文。OpenAI Chat 兼容接口适合已有 Chat 客户端，不应作为新集成的首选。

参考：[Anthropic 原生 API 与 OpenAI 兼容层的差异](https://platform.claude.com/docs/en/cli-sdks-libraries/libraries/openai-sdk)。

选择依据是模型官方接口、DeepClub 已发布端点和客户端实际能力。使用缓存、推理或工具调用前，请先用小请求验证；端点可用不代表提供联网搜索等厂商托管工具、保证缓存命中或在服务端保存会话。

## Claude Fable 5.1 在 Claude Code 里怎么配置？

ANTHROPIC_BASE_URL 设为 deepclub.net，不带 /v1。

1. 在终端设置环境变量：`export ANTHROPIC_BASE_URL=https://deepclub.net`。
2. 设置密钥：`export ANTHROPIC_AUTH_TOKEN=sk-你的DeepClub密钥`。
3. 指定模型：`export ANTHROPIC_MODEL=claude-fable-5-1`。
4. 在项目目录运行 `claude`，请求会走 `/v1/messages`。

Claude Code 会自己拼接 `/v1/messages`，所以 Base URL 不要带 `/v1`。请使用支持该型号的 Claude Code 版本（见 [Claude Code 接入页](https://deepclub.net/integrations/claude-code) 的版本表）。

## 怎么用 Python 调用 claude-fable-5-1（推荐协议）？

安装 `pip install -U anthropic`，并在本地设置环境变量 `DEEPCLUB_API_KEY`。

```python
import os
from anthropic import Anthropic

client = Anthropic(
    api_key=os.environ["DEEPCLUB_API_KEY"],
    base_url="https://deepclub.net",
)

response = client.messages.create(
    model="claude-fable-5-1",
    max_tokens=4096,
    messages=[{"role": "user", "content": "分析这个项目的错误处理流程，指出最需要优先验证的边界条件"}],
)
print("".join(block.text for block in response.content if block.type == "text"))
```

## 用 curl 调用 Anthropic Messages（推荐）

```bash
curl https://deepclub.net/v1/messages \
  -H "x-api-key: $DEEPCLUB_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "Content-Type: application/json" \
  -d '{"model":"claude-fable-5-1","max_tokens":4096,"messages":[{"role":"user","content":"分析这个项目的错误处理流程，指出最需要优先验证的边界条件"}]}'
```

## 用 curl 调用 OpenAI Chat Completions（兼容选项）

仅在现有客户端或集成需要该协议时选用；切换时必须同步修改请求体和响应解析。

```bash
curl https://deepclub.net/v1/chat/completions \
  -H "Authorization: Bearer $DEEPCLUB_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model":"claude-fable-5-1","messages":[{"role":"user","content":"分析这个项目的错误处理流程，指出最需要优先验证的边界条件"}]}'
```

claude-fable-5-1 接受 `temperature`、`top_p`、`top_k` 和 `thinking.budget_tokens`；切换到 claude-opus-5-5、claude-sonnet-5-5 或 claude-fable-5-2 前，请删掉这些参数，改用自适应思考。

## Claude Code 怎么固定使用 Fable 5.1？

使用完整模型 ID claude-fable-5-1，避免 fable 别名自动指向新版本。

Claude Code 需要 2.1.257 或更新版本。使用 `ANTHROPIC_MODEL=claude-fable-5-1` 或启动参数 `claude --model claude-fable-5-1`。

完整型号和版本要求见 [Claude Code 模型配置](https://code.claude.com/docs/en/model-config#work-with-fable)。网关支持的上下文、价格和可用状态仍以 DeepClub 当前目录为准。

## claude-fable-5-1 API 怎么计费？

按百万 Token 计费，输入和输出分别计价。

实时价格见 [定价页](https://deepclub.net/pricing/claude-fable-5-1)。

- 计费单位：按百万 Token，价格可能调整，以 [定价页](https://deepclub.net/pricing/claude-fable-5-1) 实时显示为准。
- 用量明细：在控制台 **使用日志** 中按请求查看消耗。
- 充值方式：支持信用卡、微信支付、支付宝等，以控制台充值页显示为准。

## 调用 claude-fable-5-1 报 401 怎么办？

401 表示密钥无效或缺失，先检查 Key 是否完整。

- 确认 API Key 前后没有空格或换行，且令牌未被禁用、未过期。
- Messages 请求用 x-api-key: sk-... 并带 anthropic-version: 2023-06-01；Claude Code 的 ANTHROPIC_AUTH_TOKEN 使用网关支持的 Authorization: Bearer。OpenAI Chat Completions 请求使用 Authorization: Bearer sk-...。
- 如果返回 404，按上面的客户端步骤检查最终请求地址，避免重复或漏掉 /v1；模型 ID 必须完整匹配 `claude-fable-5-1`。Anthropic SDK、Claude Code 与桌面客户端的 Base URL 拼接规则不相同。

## 用 claude-fable-5-1 时请求内容会被存储吗？

不会，深云 DeepClub 实行零留存（ZDR）政策。

深云 DeepClub 实行 Zero Data Retention (ZDR) 零留存政策：请求的输入和输出内容不做持久化存储，详见 [隐私政策](https://deepclub.net/privacy-policy)。

## 使用 Claude Fable 5.1 还有哪些常见问题？

以下问答汇总了调用时最常见的问题。

### 调用 claude-fable-5-1 返回 401 Unauthorized 怎么办？

401 表示密钥无效或缺失，先检查 API Key 是否复制完整、前后没有空格。Messages 请求用 x-api-key: sk-... 并带 anthropic-version: 2023-06-01；Claude Code 的 ANTHROPIC_AUTH_TOKEN 使用网关支持的 Authorization: Bearer。OpenAI Chat Completions 请求使用 Authorization: Bearer sk-...。

### claude-fable-5-1 用哪个接口和协议调用？

优先用 Anthropic Messages，端点为 POST /v1/messages，Anthropic Python SDK 的 Base URL 为 https://deepclub.net。其他已开放端点 POST /v1/chat/completions；只有客户端限制或已有集成需要时才使用兼容路径。

### claude-fable-5-1 API 怎么计费？

按百万 Token 计费，输入和输出分别计价。实时价格见 https://deepclub.net/pricing/claude-fable-5-1。

### Claude Code 怎么固定使用 Fable 5.1？

使用完整模型 ID claude-fable-5-1，避免 fable 别名自动指向新版本。

## 多框架 & 主流平台快捷接入对照

| 工具 / 开发框架 | 选用模式 / Provider | 地址（Base URL / Endpoint） | 模型参数写法与注意事项 |
|---|---|---|---|
| **Claude Code** | Anthropic 原生 | `ANTHROPIC_BASE_URL=https://deepclub.net`（**不带 `/v1`**） | 填完整 `claude-fable-5-1`；需 Claude Code `2.1.257+` |
| **Vercel AI SDK** | `createAnthropic` | `https://deepclub.net/v1`（**需带 `/v1`**） | `anthropic('claude-fable-5-1')`；`@ai-sdk/anthropic` 默认只追加 `/messages` |
| **LangChain (Py)** | `ChatAnthropic` | `base_url="https://deepclub.net"`（不带 `/v1`） | 标准 Messages 参数 |
| **Dify 插件** | 官方 `Anthropic` (`0.3.41+`) | `https://deepclub.net` | 添加自定义模型 `claude-fable-5-1` |
| **Chatbox (`1.24.3+`)** | Claude API Compatible | `API Host: https://deepclub.net/v1` | Host 勿漏 `/v1`；保持默认采样参数 |
| **Zed Editor** | `anthropic_compatible` | `api_url: "https://deepclub.net"` | 支持 tools 与 prompt caching |

> **协议主选摘要**：`claude-fable-5-1` 推荐优先使用 **Anthropic Messages (`/v1/messages`)**。完整客户端配置指南见 [平台与开发框架适配目录](../../guides/platforms/README.md) 与 [全量排障手册](../../guides/protocol-and-clients/error-troubleshooting-handbook.md)。
