# OpenAI Agents SDK（Python / TypeScript）接入深云 DeepClub：多 Agent 路由与 Responses 模板

**OpenAI Agents SDK**（`openai-agents`）默认采用 **OpenAI Responses API**，非常适合多 Agent Handoffs、工具调用与结构化输出。

对接 **[深云 DeepClub](https://deepclub.net)** 时，只要通过 `AsyncOpenAI(base_url="https://deepclub.net/v1", ...)` 注入默认客户端，即可无缝使用 `gpt-6-astra`、`gpt-6.1-sol`、`gpt-6.1-terra`、`gpt-6-luna`、`grok-4.7` 等原生 Responses 模型；若某些 Agent 节点需要切到 DeepSeek / Gemini / GLM / Kimi / Qwen，也可以按模型切换成 Chat Completions 模式。

---

## 一、安装与环境变量

```bash
pip install -U openai-agents openai
export DEEPCLUB_API_KEY="sk-你的DeepClub密钥"
```

*(也可以直接使用环境变量 `OPENAI_API_KEY` + `OPENAI_BASE_URL="https://deepclub.net/v1"`；下面的代码演示显式注入 `AsyncOpenAI`，更方便管理多环境变量与无状态调用。)*

---

## 二、Python 模板 1：全局使用深云 DeepClub + Responses API（GPT-6 / Grok）

```python
import os
from openai import AsyncOpenAI
from agents import (
    Agent,
    Runner,
    ModelSettings,
    function_tool,
    set_default_openai_client,
    set_tracing_disabled,
)

# 1. 将 Agents SDK 默认客户端指向深云 DeepClub (/v1)
client = AsyncOpenAI(
    api_key=os.environ["DEEPCLUB_API_KEY"],
    base_url="https://deepclub.net/v1",
)
set_default_openai_client(client)

# 不向第三方链路上传 trace 数据，配合 DeepClub ZDR 零留存政策
set_tracing_disabled(True)


@function_tool
def get_service_status(service_name: str) -> str:
    """Query internal health status of a microservice."""
    return f"{service_name}: healthy (p99=24ms)"


agent = Agent(
    name="SRE Triage Agent",
    instructions="你是值班 SRE 助手。先调用工具查询状态，再给出简洁中文结论。",
    model="gpt-6.1-sol",
    model_settings=ModelSettings(store=False),
    tools=[get_service_status],
)

result = Runner.run_sync(agent, "帮我检查 billing-gateway 的当前健康状态")
print(result.final_output)
```

> **重要工程说明**：  
> 1. DeepClub 不支持服务端会话存储与 `previous_response_id`，因此模板中显式设置了 `store=False`（以及关闭 Agents SDK 默认上传到 OpenAI 官方后台的 tracing：`set_tracing_disabled(True)`），全程对齐 **Zero Data Retention (ZDR)**。  
> 2. 注意遵循 `openai-agents` 官方客户端所有权约束：把 `client` 传给 `set_default_openai_client` 或 `OpenAIProvider(openai_client=client)` 时，不要再同时传入 `api_key=` / `base_url=` 参数给 `OpenAIProvider`。

---

## 三、Python 模板 2：混合使用 DeepSeek / Gemini / Qwen（`OpenAIChatCompletionsModel`）

如果你的主调度 Agent 用 `gpt-6-astra`（Responses），而子工具/总结 Agent 想用更低成本且仅开放 `/v1/chat/completions` 的模型（如 `deepseek-v4.1-flash`、`gemini-4-hydrogen`、`qwen3.8-flash`），**不要全局切掉 Responses**，直接给该 Agent 指定 `OpenAIChatCompletionsModel` 即可：

```python
import os
from openai import AsyncOpenAI
from agents import (
    Agent,
    OpenAIChatCompletionsModel,
    OpenAIResponsesModel,
    Runner,
    set_tracing_disabled,
)

set_tracing_disabled(True)

dc_client = AsyncOpenAI(
    api_key=os.environ["DEEPCLUB_API_KEY"],
    base_url="https://deepclub.net/v1",
)

# 路由 Agent：走 /v1/responses
planner = Agent(
    name="Planner",
    instructions="负责拆解任务，需要高速总结时交给 Summarizer。",
    model=OpenAIResponsesModel(model="gpt-6.1-sol", openai_client=dc_client),
)

# 子 Agent：走 /v1/chat/completions
summarizer = Agent(
    name="Summarizer",
    instructions="负责把长文本浓缩成 3 条要点。",
    model=OpenAIChatCompletionsModel(
        model="gemini-4-hydrogen", openai_client=dc_client
    ),
)

planner.handoffs = [summarizer]

result = Runner.run_sync(planner, "请总结一下 Responses API 和 Chat Completions 的区别")
print(result.final_output)
```

*(如果整个项目都只用 Chat Completions 模型，也可以直接调用 `from agents import set_default_openai_api; set_default_openai_api("chat_completions")`。）*

---

## 四、TypeScript / Node.js（`@openai/agents`）简明接入

在 TypeScript 版本中同样是把默认 OpenAI 客户端指向 `https://deepclub.net/v1`：

```ts
import OpenAI from 'openai'
import { Agent, run, setDefaultOpenAIClient, setTracingDisabled } from '@openai/agents'

setDefaultOpenAIClient(
  new OpenAI({
    apiKey: process.env.DEEPCLUB_API_KEY,
    baseURL: 'https://deepclub.net/v1',
  })
)
setTracingDisabled(true)

const agent = new Agent({
  name: 'Code Reviewer',
  model: 'gpt-6.1-sol',
  instructions: '审查输入的代码片段，按严重级别输出问题列表。',
})

const result = await run(agent, '请审查这段 Go HTTP handler 是否遗漏 context 取消检查')
console.log(result.finalOutput)
```

---

## 五、常见问题排查

- **开启了 tracing 报错或提示缺少 `OPENAI_API_KEY`**：`openai-agents` 默认会把 trace 发往 OpenAI 官方服务器。如果环境变量里没有官方 Key 而只有 `DEEPCLUB_API_KEY`，务必加上 `set_tracing_disabled(True)`（TS：`setTracingDisabled(true)`）。
- **在 Chat 模型上忘记切协议报错**：调用 `deepseek-v4.1-flash`、`gemini-4-hydrogen`、`kimi-k3`、`qwen3.8-flash` 时，它们仅开放 `/v1/chat/completions`，请使用 `OpenAIChatCompletionsModel` 或 `set_default_openai_api("chat_completions")`。

完整目录与端点对照：[https://deepclub.net/pricing](https://deepclub.net/pricing) ｜ [https://deepclub.net/docs](https://deepclub.net/docs)
