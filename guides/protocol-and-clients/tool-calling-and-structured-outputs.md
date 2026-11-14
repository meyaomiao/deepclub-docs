# Tool Calling 与结构化输出（Structured Outputs）兼容实战手册

搭建 Agent、数据库查询助手或表单抽取流水线时，最容易出故障的环节就是 **Tool Calling（多轮工具调用回传）** 与 **JSON 结构化输出**——而且 **Messages、Responses、Chat Completions 三套协议的字段和回传格式完全不一样**！

本文给出三套协议在 **[深云 DeepClub](https://deepclub.net)** 上的标准写法与多轮上下文衔接边界说明。

---

## 一、三套协议的关键字段对照

| 能力维度 | Anthropic Messages (`/v1/messages`) | OpenAI Responses (`/v1/responses`) | OpenAI Chat (`/v1/chat/completions`) |
|---|---|---|---|
| **工具定义** | `tools: [{name, description, input_schema}]` | `tools: [{type: "function", name, parameters}]` | `tools: [{type: "function", function: {name, parameters}}]` |
| **模型返回工具调用** | `response.content` 中 `block.type == "tool_use"` (`block.id`, `block.name`, `block.input`) | `response.output` 中 `item.type == "function_call"` (`item.call_id`, `item.name`, `item.arguments`) | `msg.tool_calls` (`tc.id`, `tc.function.name`, `tc.function.arguments`) |
| **客户端回传工具结果** | `role: "user"`，content 为 `[{"type": "tool_result", "tool_use_id": id, "content": "..."}]` | `input` 数组带回原 `output` 项，再追加 `{"type": "function_call_output", "call_id": id, "output": "..."}` | 带回含 `tool_calls` 的 `assistant` 消息，再追加 `{"role": "tool", "tool_call_id": id, "content": "..."}` |
| **结构化输出字段** | 通过 tool schema 约束或提示词 + 校验 | `text={"format": {"type": "json_schema", ...}}` | `response_format={"type": "json_object"}` 或 `json_schema` |

---

## 二、协议 1：Anthropic Messages 原生 Tool Calling（`claude-sonnet-5-5` / `claude-opus-5-5`）

```python
import json, os
from anthropic import Anthropic

client = Anthropic(
    api_key=os.environ["DEEPCLUB_API_KEY"],
    base_url="https://deepclub.net",
)

tools = [
    {
        "name": "get_stock_price",
        "description": "查询股票最新价格（模拟）",
        "input_schema": {
            "type": "object",
            "properties": {"ticker": {"type": "string"}},
            "required": ["ticker"],
        },
    }
]

messages = [{"role": "user", "content": "查一下 NVDA 当前价格并分析"}]

# 第一轮
resp1 = client.messages.create(
    model="claude-sonnet-5-5",
    max_tokens=1024,
    tools=tools,
    messages=messages,
)

# 必须把完整的 assistant content blocks 保留进历史
messages.append({"role": "assistant", "content": resp1.content})

tool_results = []
for block in resp1.content:
    if block.type == "tool_use":
        result_str = json.dumps({"ticker": block.input["ticker"], "price": 142.5})
        tool_results.append({
            "type": "tool_result",
            "tool_use_id": block.id,
            "content": result_str,
        })

if tool_results:
    messages.append({"role": "user", "content": tool_results})
    resp2 = client.messages.create(
        model="claude-sonnet-5-5",
        max_tokens=1024,
        tools=tools,
        messages=messages,
    )
    print("".join(b.text for b in resp2.content if b.type == "text"))
```

---

## 三、协议 2：OpenAI Responses 多轮工具回传与 `text.format`（`gpt-6.1-sol` / `gpt-6-astra` / `grok-4.7`）

> **关键注意点**：DeepClub **不支持 `previous_response_id` 与服务端会话保存**（示例均传 `store=False`）。因此 Responses 多轮工具调用必须把**上一轮的 `input` + 模型返回的全部 `response.output` 项 + 新产生的 `function_call_output`** 一起传回下一轮请求，绝不能只传 `output_text`！

```python
import json, os
from openai import OpenAI

client = OpenAI(
    api_key=os.environ["DEEPCLUB_API_KEY"],
    base_url="https://deepclub.net/v1",
)

tools = [
    {
        "type": "function",
        "name": "get_order_status",
        "description": "查询订单状态",
        "parameters": {
            "type": "object",
            "properties": {"order_id": {"type": "string"}},
            "required": ["order_id"],
            "additionalProperties": False,
        },
    }
]

history = [{"role": "user", "content": "帮我查一下订单 DC-2026-001 的状态"}]

resp1 = client.responses.create(
    model="gpt-6.1-sol",
    input=history,
    tools=tools,
    store=False,
)

# 将返回的输出项（包含 function_call）追加回本地历史
history.extend(resp1.output)

for item in resp1.output:
    if item.type == "function_call":
        args = json.loads(item.arguments)
        history.append({
            "type": "function_call_output",
            "call_id": item.call_id,
            "output": json.dumps({"order_id": args["order_id"], "status": "shipped"}),
        })

resp2 = client.responses.create(
    model="gpt-6.1-sol",
    input=history,
    tools=tools,
    store=False,
)
print(resp2.output_text)
```

### Responses 结构化输出写法（用 `text.format` 而非 `response_format`）

```python
resp = client.responses.create(
    model="gpt-6.1-sol",
    input="从这句话提取姓名与城市：张伟昨天去了深圳出差。",
    store=False,
    text={
        "format": {
            "type": "json_schema",
            "name": "trip_entity",
            "strict": True,
            "schema": {
                "type": "object",
                "properties": {
                    "name": {"type": "string"},
                    "city": {"type": "string"},
                },
                "required": ["name", "city"],
                "additionalProperties": False,
            },
        }
    },
)
print(resp.output_text)
```

---

## 四、协议 3：Chat Completions（`deepseek-v4.1-flash` / `gemini-3.8-flash-high` / `glm-5.3` / 兼容模式）

1. **务必先将完整 `assistant` 消息（含 `tool_calls` 数组）放回 `messages`**，再逐个追加 `role: "tool"` 且匹配 `tool_call_id`；全部工具结果回传完再发起下一轮请求，不要只发孤立的 `tool` 消息。
2. 网关对开放 Chat 兼容的 Claude / GPT 也会做转换，但**不同模型在 Chat 转换下的复杂嵌套工具与思考块等价性需要按具体业务路径先用小测试验证**；新集成仍建议 Claude 走 Messages、GPT/Grok 走 Responses。
3. **业务兜底规则**：即使使用了 `json_schema` 或 `json_object`，在生产代码反序列化前仍建议加 `try/except json.JSONDecodeError` 与 schema 校验，异常时做有界重试。

完整端点数与推荐接入：[https://deepclub.net/docs](https://deepclub.net/docs) ｜ [https://deepclub.net/pricing](https://deepclub.net/pricing)
