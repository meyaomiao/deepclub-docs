# 深云 DeepClub 完整接入代码示例（5 大协议模板）

官网：[https://deepclub.net](https://deepclub.net) ｜ 实时价格：[https://deepclub.net/pricing](https://deepclub.net/pricing) ｜ 官方文档：[https://deepclub.net/docs](https://deepclub.net/docs)

---

## 1. Anthropic Messages 流式输出（Claude 系列）

> **注意**：Anthropic SDK 的 `base_url` 填站点根地址 `https://deepclub.net`（**不要**带 `/v1`）；直接发 HTTP 请求时完整地址为 `https://deepclub.net/v1/messages`。

```python
import os
from anthropic import Anthropic

client = Anthropic(
    api_key=os.environ["DEEPCLUB_API_KEY"],
    base_url="https://deepclub.net",
)

with client.messages.stream(
    model="claude-opus-5-5",
    max_tokens=1024,
    messages=[{"role": "user", "content": "请给出 Go HTTP 服务优雅退出的关键点"}],
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)
```

---

## 2. OpenAI Responses 流式输出（GPT 文本 / Grok 系列）

> **注意**：Responses 的流式文本增量事件是 `response.output_text.delta`，不要套用 Chat Completions 的 `choices[0].delta`。

```python
import os
from openai import OpenAI

client = OpenAI(
    api_key=os.environ["DEEPCLUB_API_KEY"],
    base_url="https://deepclub.net/v1",
)

stream = client.responses.create(
    model="gpt-6.1-sol",
    input="列出三个代码审阅要点",
    store=False,
    stream=True,
)
for event in stream:
    if event.type == "response.output_text.delta":
        print(event.delta, end="", flush=True)
```

Node.js / TypeScript 版本：

```ts
import OpenAI from 'openai'

const client = new OpenAI({
  apiKey: process.env.DEEPCLUB_API_KEY,
  baseURL: 'https://deepclub.net/v1',
})

const response = await client.responses.create({
  model: 'gpt-6.1-terra',
  input: 'Summarize HTTP/3 benefits in 3 bullets.',
  store: false,
})
console.log(response.output_text)
```

---

## 3. OpenAI Chat Completions（DeepSeek / Gemini 文本 / GLM / Kimi / Qwen）

> **注意**：深云 DeepClub 当前目录中的 Gemini 文本模型（`gemini-4-argon`、`gemini-4-carbon`、`gemini-4-hydrogen`、`gemini-3.8-flash-high`、`gemini-3.8-flash`）均通过 `/v1/chat/completions` 调用。

```bash
curl https://deepclub.net/v1/chat/completions \
  -H "Authorization: Bearer $DEEPCLUB_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gemini-4-hydrogen",
    "messages": [{"role": "user", "content": "Hello"}]
  }'
```

---

## 4. 图片生成与参考图编辑（`/v1/images/generations` & `/v1/images/edits`）

```python
import os
from openai import OpenAI

client = OpenAI(
    api_key=os.environ["DEEPCLUB_API_KEY"],
    base_url="https://deepclub.net/v1",
)

# 1) 文生图
gen = client.images.generate(
    model="gpt-image-2",
    prompt="A clean ceramic coffee cup on a minimalist oak desk, soft daylight",
    size="1024x1024",
)
print(gen.data[0].url or gen.data[0].b64_json[:64])

# 2) 参考图编辑（multipart）
with open("input.png", "rb") as image_file:
    edited = client.images.edit(
        model="gemini-3-pro-image",
        image=image_file,
        prompt="Change the desk background to dark slate stone while keeping the cup",
    )
print(edited.data[0].url or edited.data[0].b64_json[:64])
```

---

## 5. 视频异步生成与轮询下载（`/v1/videos`）

```bash
# 1) 提交任务（Seedance 示例 5 秒；MiniMax-H3 按目录规格可使用 10 秒）
TASK_JSON=$(curl -sS https://deepclub.net/v1/videos \
  -H "Authorization: Bearer $DEEPCLUB_API_KEY" \
  -F model=Seedance-2.0-Fast \
  -F prompt="A slow dolly shot across a quiet alpine lake at sunrise" \
  -F seconds=5)
echo "$TASK_JSON"

TASK_ID=$(python3 -c 'import json,sys; print(json.load(sys.stdin)["id"])' <<<"$TASK_JSON")

# 2) 轮询状态：queued / in_progress / completed / failed
while true; do
  STATUS=$(curl -fsS "https://deepclub.net/v1/videos/$TASK_ID" \
    -H "Authorization: Bearer $DEEPCLUB_API_KEY" | python3 -c 'import json,sys; print(json.load(sys.stdin)["status"])')
  echo "status=$STATUS"
  [[ "$STATUS" == "completed" ]] && break
  [[ "$STATUS" == "failed" ]] && exit 1
  sleep 5
done

# 3) 下载成片
curl -fsSL -o output.mp4 "https://deepclub.net/v1/videos/$TASK_ID/content" \
  -H "Authorization: Bearer $DEEPCLUB_API_KEY"
```
