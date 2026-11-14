# Prompt Caching（提示缓存）省钱实战：计费规则、命中条件与工程模板

在编码 Agent（Claude Code、Cline、Roo Code）、长文档 RAG、或者带大量 System Prompt / Tool Schema 的生产工作流里，**输入 Token 往往占总 Token 消耗的 80%~95%**。如果每次对话都要把 50K Token 的项目上下文从头按标准输入单价计费，成本会迅速失控。

本文结合 **[深云 DeepClub](https://deepclub.net)** 在售目录中真实公布的缓存单价（实时见 [pricing](https://deepclub.net/pricing)），拆解 **Anthropic 显式缓存（`cache_control`）** 与 **OpenAI / DeepSeek / Google / xAI / Zhipu 自动前缀缓存** 的计费差异与工程写法。

---

## 一、各主力模型实盘缓存单价一览（元 / 百万 Token）

> 以下价格摘自深云 DeepClub 当前公开定价目录，调用前请以 [https://deepclub.net/pricing](https://deepclub.net/pricing) 实时展示为准：

| 模型 ID | 标准输入 | 缓存读取（`cache_read`） | 缓存写入（5m / 1h） | 降本幅度（读取 vs 标准输入） |
|---|---|---|---|---|
| `claude-opus-5-5` | ¥2.0 | **¥0.10** | ¥2.5 / ¥4.0 | **降 95%**（低至 1/20） |
| `claude-sonnet-5-5` | ¥1.0 | **¥0.10** | ¥1.25 / ¥2.0 | **降 90%**（1/10） |
| `claude-fable-5-1` | ¥8.5 | **¥0.2125** | ¥10.625 / ¥17.0 | **降 97.5%**（1/40） |
| `claude-fable-5-2` | ¥18.4 | **¥0.46** | ¥23.0 / ¥36.8 | **降 97.5%**（1/40） |
| `gpt-6-astra` | ¥3.5 | **¥0.35** | 自动触发 | **降 90%** |
| `gpt-6.1-sol` | ¥0.7 | **¥0.035** | 自动触发 | **降 95%** |
| `gpt-6-luna` | ¥0.12 | **¥0.012** | 自动触发 | **降 90%** |
| `deepseek-v4.1-flash` | ¥1.1 | **¥0.022** | 自动触发 | **降 98%**（1/50） |
| `gemini-4-argon` | ¥1.1 | **¥0.055** | 自动触发 | **降 95%** |
| `gemini-4-hydrogen` | ¥0.6 | **¥0.03** | 自动触发 | **降 95%** |
| `glm-5.3` | ¥4.4 | **¥1.10** | 自动触发 | **降 75%** |
| `kimi-k3` | ¥12.0 | **¥1.20** | 自动触发 | **降 90%** |

*(注：`qwen3.8-flash` 当前目录未单独发布 `cache_read` 维度。)*

---

## 二、为什么需要区分“显式缓存”与“自动前缀缓存”？

### 1. Claude 系列：走原生 `/v1/messages` + `cache_control` 断点

Claude 需要通过请求体中的 `cache_control: {"type": "ephemeral"}` 显式标记缓存断点。常见放置顺序：
1. **System Prompt 末尾**（长指令 / 架构规范 / 代码规范）
2. **Tools 定义列表末尾**（大量函数 schema）
3. **倒数第 1~2 轮固定静态上下文块末尾**

```python
import os
from anthropic import Anthropic

client = Anthropic(
    api_key=os.environ["DEEPCLUB_API_KEY"],
    base_url="https://deepclub.net",
)

response = client.messages.create(
    model="claude-sonnet-5-5",
    max_tokens=1024,
    system=[
        {
            "type": "text",
            "text": "这里是长达数千字的企业代码库规范与数据库 Schema 说明……" * 80,
            "cache_control": {"type": "ephemeral"},
        }
    ],
    messages=[{"role": "user", "content": "请检查新增订单表索引是否缺少联合约束"}],
)
print(response.usage)
```

> **注意**：不要把 Claude 强行放到第三方 Chat Completions 转换层并指望缓存命中行为完全等价；需要稳定利用 `cache_write` / `cache_read` 时，优先使用原生 `/v1/messages`（如 Anthropic SDK 或 Claude Code），并在使用日志中核对 `cache_read` 与 `cache_write` Token 数。

### 2. GPT-6 / DeepSeek / Gemini / Grok / GLM / Kimi：保持前缀字节级稳定

这类模型的缓存通常是**前缀匹配自动触发**：只有当连续多次请求的前缀部分（`tools`、`instructions` / `system`、前置消息历史）**完全逐字一致且达到最小缓存阈值**时才会命中 `cache_read`。

**最常见的缓存失效元凶（90% 开发者会踩的坑）**：
- ❌ 把 `当前时间：2026-11-14 10:22:05` 或随机 `request_id` 放进 System Prompt 最开头 → 导致整段前缀失效，每次都缓存未命中！
- ✅ **正确做法**：把完全不变的指令、Schema、知识库放最前面；把时间戳、用户当前状态、动态问题放到最末尾一条 User Message 里。

---

## 三、如何核算缓存到底有没有帮你省钱？

以 `claude-opus-5-5` 为例（假设静态前缀 40,000 Tokens，单轮新增问题 500 Tokens、输出 500 Tokens）：

- **未命中缓存（10 轮）**：`10 × (40,500 × ¥2.0 + 500 × ¥10.0) / 1e6 = ¥0.86`
- **命中 5 分钟缓存（第 1 轮写，第 2~10 轮读）**：
  - 第 1 轮写入：`40,000 × ¥2.5 / 1e6 = ¥0.10`
  - 第 2~10 轮读取：`9 × 40,000 × ¥0.10 / 1e6 = ¥0.036`
  - 加上每轮新输入与输出：`¥0.01 + ¥0.05 = ¥0.06`
  - **10 轮合计：仅约 ¥0.196（省下 77% 费用）**！如果是长会话 30~50 轮，降幅会进一步逼近 **90%+**。

> **提醒**：不要在只有**单次、不复用的短请求**上加 `cache_control`，否则只会多付 `cache_write` 单价而没有后续读取收益。**至少在短时间内会被复用 2 次以上的长前缀才建议写入缓存**。

---

## 四、账单核对

在 [深云 DeepClub](https://deepclub.net) 控制台 → **使用日志（Usage Logs）** 中，可以直接看到每次请求对应的 **缓存读取**、**缓存写入**、**标准输入/输出** 明细与实际扣费，建议上线新 Agent 前先跑 3~5 轮会话核对一次日志。
