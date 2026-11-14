# 深云 DeepClub API 全量报错排障手册（401 / 404 双 `/v1` / 400 参数 / 429 / SSE 解析失败）

对接多模型网关时，90% 的接入报错其实都能在 2 分钟内定位。本文按 HTTP 状态码与常见 SDK 异常，整理对照 **[深云 DeepClub](https://deepclub.net)**（[实时目录](https://deepclub.net/pricing)）的排查流程。

---

## 快速自检矩阵

| 现象 / 状态码 | 最常见原因 | 对应修复动作 |
|---|---|---|
| **`401 Unauthorized`** | 密钥复制有空格/过期/配错鉴权头 | Messages 用 `x-api-key` + `anthropic-version`（或 Bearer）；OpenAI/媒体用 `Authorization: Bearer sk-...` |
| **`404 Not Found`** | **双 `/v1`（`/v1/v1/...`）** 或 **少了 `/v1`** | 核对客户端规则：Anthropic SDK 填站根，OpenAI SDK 填 `/v1`，沉浸式翻译填完整端点 URL |
| **`400 Bad Request`** | 参数与协议/模型不兼容 | Claude 自适应思考模型禁用非默认 `temperature`/`top_p`；Responses 勿传 Chat 的 `messages`；视频秒数按目录规格 |
| **模型不存在 / 无权限** | 自造别名或选了已下架型号 | 到 [pricing](https://deepclub.net/pricing) 复制完整精确 ID（含版本号后缀）并确认子 Key 权限列表 |
| **流式输出取不到文本 / 报 `AttributeError`** | 混用了三种协议的 SSE 事件结构 | Messages 看 `text_delta`/`text_stream`；Responses 看 `response.output_text.delta`；Chat 看 `choices[0].delta.content` |
| **`429 Too Many Requests`** | 触发单 Key 限流或上游短暂并发限制 | 指数退避重试（Exponential Backoff + Jitter），区分入口页限频与 API 请求限流 |

---

## 一、`401 Unauthorized` 排查清单

1. **检查 Key 是否被截断或带了前后空格/换行**：尤其是从 `.env` 或 CI Secret 复制时，建议 `curl` 打印前先确认长度及是否误把占位符 `sk-...` 直接传进去了。
2. **检查子 Key 状态、余额与允许模型范围**：登录 [深云 DeepClub](https://deepclub.net) → **API 密钥**，确认该 Key 未过期、额度未耗尽，且如果开启了“模型限制”，勾选了你正在请求的精确模型 ID。
3. **检查鉴权请求头是否匹配协议**：
   - `/v1/messages`（直接 HTTP 调用）：建议按 Anthropic 规范传 `-H "x-api-key: $DEEPCLUB_API_KEY"` 和 `-H "anthropic-version: 2023-06-01"`（网关对 Claude Code 的 `ANTHROPIC_AUTH_TOKEN` Bearer 头也提供兼容支持）。
   - `/v1/responses`、`/v1/chat/completions`、`/v1/images/*`、`/v1/videos`：传 `-H "Authorization: Bearer $DEEPCLUB_API_KEY"`。

---

## 二、`404 Not Found` 排查清单（高频重灾区）

90% 的 `404` 是 **URL 路径拼错**，或者**对某模型发了它未开放的端点**：

1. **检查双 `/v1`（`/v1/v1/messages` 等）**：
   - Anthropic Python/TS SDK、`Claude Code`、`NextChat`、`Dify Anthropic 插件`、`Zed (anthropic_compatible)` 会自己拼 `/v1/...` → Base URL **必须填站根 `https://deepclub.net`**。
   - 唯一例外：**`Chatbox` 自定义 Claude (`API Host`)** 与 **Vercel `@ai-sdk/anthropic` (`baseURL`)** 直接拼 `/messages` → 它们**必须填 `https://deepclub.net/v1`**。
2. **检查漏掉 `/v1`**：
   - `OpenAI SDK`、`Cursor`、`Cline`、`Roo Code`、`Cherry Studio`、`Codex CLI`、`OpenCode`、`LangChain ChatOpenAI` → Base URL 必须带 **`https://deepclub.net/v1`**。
   - `沉浸式翻译 (1.35.2+)` → 必须填完整端点 URL（如 `https://deepclub.net/v1/chat/completions` 或 `https://deepclub.net/v1/responses`）。
3. **检查目标模型是否真的开放了该端点**：
   - 例如 Gemini 文本模型（`gemini-4-argon`、`gemini-4-carbon`、`gemini-4-hydrogen`、`gemini-3.8-flash*`）当前在 DeepClub **仅开放 `/v1/chat/completions`**，不要把它们塞进 Google 原生 `:generateContent` 或 `/v1/responses`。
   - 生图模型（哪怕名字叫 `gpt-image-2` 或 `gemini-3-pro-image`）必须走 `/v1/images/generations` 或 `/v1/images/edits`，不要走 `/v1/chat/completions`。

---

## 三、`400 Bad Request` 高频踩坑点

1. **Claude 自适应思考模型传入冲突采样参数（`400`）**：
   - `claude-opus-5-5`、`claude-sonnet-5-5`、`claude-fable-5-2` 在 `/v1/messages` 上**不接受非 `1` 的 `temperature`、`top_p`、`top_k`**，也不接受旧式 `thinking: {"type": "enabled", "budget_tokens": ...}`。
   - **修复**：去掉自定义采样覆盖，改用自适应思考 + `output_config.effort`（`low` / `medium` / `high`）：
     ```json
     {
       "model": "claude-sonnet-5-5",
       "max_tokens": 4096,
       "thinking": { "type": "adaptive", "display": "summarized" },
       "output_config": { "effort": "medium" },
       "messages": [{ "role": "user", "content": "解释一下幂等键设计" }]
     }
     ```
2. **Responses API 误传 Chat 字段**：
   - 调用 `/v1/responses` 时，输入字段是 **`input`** 而不是 `messages`；结构化输出字段是 **`text.format`** 而不是 `response_format`；若无需服务端保存状态请传 `store=False`。
   - 同时 DeepClub **不支持 `previous_response_id` 服务端状态存储**，多轮对话请由客户端把历史带回下一轮。
3. **图片与视频参数超范围**：
   - 图片接口每次请求建议 `n=1`，且 `size` 需匹配模型发布规格（部分型号仅支持 `1024x1024`，见各自 [pricing 页面](https://deepclub.net/pricing)）。
   - 视频接口 `MiniMax-H3` 示例为 `10` 秒、`Seedance-2.0-*` 示例为 `5` 秒，且必须使用 **`multipart/form-data`** 提交，不要发成 `application/json`。

---

## 四、流式（Streaming）返回空或报错 `KeyError` / `AttributeError`

| 协议端点 | Python / JS 流式文本解析写法 | 常见错误现象 |
|---|---|---|
| `/v1/messages` | `with client.messages.stream(...) as s: for text in s.text_stream:` 或读 `content_block_delta` 中的 `text_delta` | 按 OpenAI `.choices` 解析直接报错 |
| `/v1/responses` | `if event.type == "response.output_text.delta": print(event.delta)` | 按 Chat 的 `chunk.choices[0].delta.content` 读取导致 `AttributeError` |
| `/v1/chat/completions` | `chunk.choices[0].delta.content`（需判 `if chunk.choices` 与非 `None`） | 直接取 `[0]` 未判空（最后一个 usage chunk 的 `choices` 常为空数组 `[]`） |

---

## 五、最小隔离定位法（5 秒区分客户端问题还是 Key 问题）

如果客户端怎么点都报错，先脱离客户端，直接在终端用最原生的 `curl` 发一条最小请求：

```bash
curl -i https://deepclub.net/v1/chat/completions \
  -H "Authorization: Bearer $DEEPCLUB_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model":"gpt-6-luna","messages":[{"role":"user","content":"ping"}]}'
```

- 如果终端 `curl` 返回 `200 OK`，说明账号、余额和 Key 完全正常，100% 是客户端的 URL、路径模式或自定义采样参数配错了；
- 同时可在控制台 **使用日志** 查看请求是否到达网关、最终收到的模型名和响应状态码。
