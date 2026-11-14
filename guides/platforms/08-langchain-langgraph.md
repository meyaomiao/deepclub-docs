# LangChain & LangGraph 接入深云 DeepClub：多模型节点路由与 Responses/Messages 协议适配

在 **LangChain / LangGraph（Python）** 中使用 **[深云 DeepClub](https://deepclub.net)**，最大的价值在于**同一个 API Key 可以在同一个图（StateGraph）的不同节点之间混用最优模型**——比如规划节点用 `claude-opus-5-5`（原生 Messages）或 `gpt-6-astra`（Responses），批量检索与清洗节点用 `gpt-6-luna` 或 `gemini-4-hydrogen`。

---

## 一、安装依赖

```bash
pip install -U langchain langchain-openai langchain-anthropic langgraph
export DEEPCLUB_API_KEY="sk-你的DeepClub密钥"
```

---

## 二、三大协议初始化对照表

| 模型家族 | LangChain 类 | Base URL 参数 | 协议关键参数 |
|---|---|---|---|
| **Claude 系列**<br>(`claude-opus-5-5`、`claude-sonnet-5-5`、`claude-fable-5-1/5-2`) | `ChatAnthropic` | `base_url="https://deepclub.net"`（**不带 `/v1`**） | 底层复用官方 `anthropic` SDK，走 `POST /v1/messages` |
| **GPT 文本 & Grok**<br>(`gpt-6.1-sol`、`gpt-6-astra`、`gpt-6-luna`、`grok-4.7`) | `ChatOpenAI` | `base_url="https://deepclub.net/v1"` | **`use_responses_api=True`**，`store=False` |
| **DeepSeek / Gemini 文本 / GLM / Kimi / Qwen** | `ChatOpenAI` | `base_url="https://deepclub.net/v1"` | 默认走 `POST /v1/chat/completions`（`use_responses_api=False`） |

---

## 三、完整可运行示例（多模型实例化 + LangGraph 节点分工）

```python
import os
from typing import TypedDict
from langchain_anthropic import ChatAnthropic
from langchain_openai import ChatOpenAI
from langgraph.graph import StateGraph, START, END

API_KEY = os.environ["DEEPCLUB_API_KEY"]

# 1. Claude 原生 Messages（不带 /v1；自适应思考模型不要强传非默认 temperature）
claude_coder = ChatAnthropic(
    model="claude-sonnet-5-5",
    api_key=API_KEY,
    base_url="https://deepclub.net",
    max_tokens=4096,
)

# 2. GPT-6 / Grok Responses API
gpt_architect = ChatOpenAI(
    model="gpt-6.1-sol",
    api_key=API_KEY,
    base_url="https://deepclub.net/v1",
    use_responses_api=True,
    store=False,
)

# 3. 超低成本预处理 / 摘要节点（Chat Completions）
fast_classifier = ChatOpenAI(
    model="gemini-4-hydrogen",
    api_key=API_KEY,
    base_url="https://deepclub.net/v1",
)


class ReviewState(TypedDict):
    diff: str
    risk_level: str
    report: str


def triage_node(state: ReviewState) -> dict:
    resp = fast_classifier.invoke(
        f"判断这段代码改动风险等级（low/medium/high），仅输出单词：\n{state['diff'][:2000]}"
    )
    return {"risk_level": resp.content.strip().lower()}


def deep_review_node(state: ReviewState) -> dict:
    llm = gpt_architect if "high" in state["risk_level"] else claude_coder
    resp = llm.invoke(f"请对以下改动做安全与逻辑审查：\n{state['diff']}")
    # LangChain 中 Responses / Anthropic 可能返回块列表或字符串
    text = resp.text if hasattr(resp, "text") else str(resp.content)
    return {"report": text}


builder = StateGraph(ReviewState)
builder.add_node("triage", triage_node)
builder.add_node("review", deep_review_node)
builder.add_edge(START, "triage")
builder.add_edge("triage", "review")
builder.add_edge("review", END)

graph = builder.compile()
out = graph.invoke({"diff": "def pay(amount): return db.exec(f'UPDATE u SET bal=bal-{amount}')", "risk_level": "", "report": ""})
print(out["risk_level"], "\n", out["report"])
```

---

## 四、LangChain 常见踩坑

1. **`ChatAnthropic` 填了 `/v1` 报 404**：`langchain_anthropic` 中的 `base_url`（`anthropic_api_url`）直接透传给官方 `anthropic` SDK，填 `https://deepclub.net` 即可，末尾不要带 `/v1`。
2. **`claude-opus-5-5` / `claude-sonnet-5-5` / `claude-fable-5-2` 传入 `temperature=0` 报错 400**：这三款自适应思考型号在原生 `/v1/messages` 上不接受非默认 `temperature` / `top_p` / `top_k`。初始化 `ChatAnthropic` 时请省略 `temperature`。
3. **结构化输出（`with_structured_output`）**：在 `use_responses_api=True` 下使用 `gpt-6-astra` / `gpt-6.1-sol` 能更好地利用 Responses 原生 schema 约束；若切换不同模型，建议先用小批量单测核对返回 Pydantic 解析结果。

更多模型适用边界与价格：[https://deepclub.net/pricing](https://deepclub.net/pricing) ｜ [https://deepclub.net/articles](https://deepclub.net/articles)
