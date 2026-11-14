# OpenAI Responses API 实战迁移指南：从 `chat.completions` 切到 `responses.create` 到底改了什么？

OpenAI 已在官方迁移文档中明确将 **Responses API（`POST /v1/responses`）** 作为新项目和新一代推理/工具调用模型的首选入口；xAI 对 Grok 新集成同样推荐使用 Responses。

但在实际接入时，很多开发者踩到的第一个坑就是：**只把接口路径或方法名改了，请求参数和流式 SSE 解析却还在套用 `chat.completions` 的旧写法**，结果拿不到 `output_text` 或流式直接报错。

下面整理一份可直接落地的迁移对照清单，示例网关使用同时开放 `/v1/responses` 与 `/v1/chat/completions` 的 **[深云 DeepClub](https://deepclub.net)**。

---

## 一、核心差异速查表

| 对比项 | 旧 Chat Completions (`/v1/chat/completions`) | 新 Responses API (`/v1/responses`) |
|---|---|---|
| **Python SDK 方法** | `client.chat.completions.create(...)` | `client.responses.create(...)` |
| **输入字段** | `messages=[{"role": "user", "content": "..."}]` | `input="..."`（或结构化输入数组） |
| **状态保留开关** | 无默认服务端状态 | 如不需要服务端保存历史，显式传 `store=False` |
| **非流式文本读取** | `resp.choices[0].message.content` | `resp.output_text` |
| **流式文本增量事件** | `chunk.choices[0].delta.content` | `event.type == "response.output_text.delta"` 时读 `event.delta` |
| **结构化输出字段** | `response_format` | `text={"format": ...}` |

---

## 二、Python 迁移对比示例

### 旧写法（Chat Completions）

```python
import os
from openai import OpenAI

client = OpenAI(
    api_key=os.environ["DEEPCLUB_API_KEY"],
    base_url="https://deepclub.net/v1",
)

resp = client.chat.completions.create(
    model="gpt-6.1-sol",
    messages=[{"role": "user", "content": "给出 3 条 HTTP 缓存最佳实践"}],
)
print(resp.choices[0].message.content)
```

### 新写法（Responses API，推荐用于 GPT 文本与 Grok）

```python
import os
from openai import OpenAI

client = OpenAI(
    api_key=os.environ["DEEPCLUB_API_KEY"],
    base_url="https://deepclub.net/v1",
)

# 非流式
resp = client.responses.create(
    model="gpt-6.1-sol",
    input="给出 3 条 HTTP 缓存最佳实践",
    store=False,
)
print(resp.output_text)

# 流式读取
stream = client.responses.create(
    model="gpt-6.1-sol",
    input="给出 3 条 HTTP 缓存最佳实践",
    store=False,
    stream=True,
)
for event in stream:
    if event.type == "response.output_text.delta":
        print(event.delta, end="", flush=True)
```

---

## 三、常见报错与排查要点

1. **流式解析报 `AttributeError: ... has no attribute 'choices'`**  
   原因：仍然按 `chunk.choices[0].delta.content` 解析 Responses SSE 流。请改为判断 `event.type == "response.output_text.delta"` 并读取 `event.delta`。
2. **结构化输出报错**  
   原因：Responses API 不使用顶层 `response_format`，需要改为 `text.format`。
3. **跨客户端配置（如 Cherry Studio / Chatbox / 沉浸式翻译 / Dify）**  
   - **Cherry Studio 2.2.0+**：可在自定义服务中明确选 OpenAI Responses 端点；如果同时有只支持 Chat 的模型（如 `deepseek-v4.1-flash`、`gemini-4-hydrogen`），建议按协议拆成不同服务避免端点优先级混淆。
   - **Chatbox 1.24.3+**：可选择 `OpenAI Responses API Compatible`，Host 填 `https://deepclub.net/v1`，路径填 `/responses`。
   - **沉浸式翻译 1.35.2+**：依据完整 URL `/v1/responses` 识别协议，填写完整端点地址 `https://deepclub.net/v1/responses`。

---

## 四、哪些模型适合切到 Responses？

在 **[深云 DeepClub 定价目录](https://deepclub.net/pricing)** 中，以下主力模型均已开放 `/v1/responses` 并推荐优先使用（同时保留 `/v1/chat/completions` 作为兼容备选）：

- **`gpt-6-astra`**：深度推理与复杂工程（输入 ¥3.5 / 输出 ¥17.5 每百万 Token）
- **`gpt-6.1-sol`**：通用高性价比主力（输入 ¥0.7 / 输出 ¥3.5 每百万 Token）
- **`gpt-6.1-terra`**：轻量快速响应（输入 ¥0.35 / 输出 ¥1.75 每百万 Token）
- **`gpt-6-luna`**：高频低成本对话（输入 ¥0.12 / 输出 ¥0.6 每百万 Token）
- **`grok-4.7`**：实时资讯与多协议兼容（输入 ¥0.4 / 输出 ¥1.2 每百万 Token）

更多 SDK 示例（Node.js / cURL）与各模型实时价格，可直接查看 [https://deepclub.net/docs](https://deepclub.net/docs)。
