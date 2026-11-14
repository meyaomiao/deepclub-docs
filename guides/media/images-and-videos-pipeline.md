# 用一个 API Key 串起文生图与视频生成：Images (`/v1/images/*`) 与异步 Videos (`/v1/videos`) 工程实战

很多同学在接了多模型网关后，容易把名字里带 `gpt` 或 `gemini` 的生图模型也往 `/v1/chat/completions` 里塞，结果收到 `400/404` 报错；或者在接视频生成时把异步任务接口当成同步请求去等，导致 HTTP 超时。

本文结合 **[深云 DeepClub](https://deepclub.net)** 当前发布的媒体接口规范，整理一套可直接放进生产代码的**图片生成 + 参考图编辑 + 异步视频生成**工程模板。

---

## 一、媒体接口端点总览（不要混用文本接口）

| 能力 | HTTP 端点 | 请求格式 | 计费方式 | 在售代表型号 |
|---|---|---|---|---|
| **文生图** | `POST /v1/images/generations` | JSON | 按张计费（¥0.16 起） | `gpt-image-2`、`gpt-image-2.5-flare/sunburst`、`gemini-3-pro-image`、`gemini-3.1-flash-image`、`nano-banana-2/pro` |
| **参考图编辑** | `POST /v1/images/edits` | `multipart/form-data` | 按张计费 | 同上 |
| **视频生成** | `POST /v1/videos` → `GET /v1/videos/{id}` → `GET /v1/videos/{id}/content` | `multipart/form-data` 提交 + 异步轮询 | 按个计费（¥1.5 起） | `Seedance-2.0-Mini`、`Seedance-2.0-Fast`、`MiniMax-H3` |

---

## 二、图片生成与参考图编辑（Python + OpenAI SDK）

无论底层是 GPT Image 系列还是 Gemini Image / Nano Banana 系列，在深云 DeepClub 上统一使用 OpenAI Images 接口，代码完全通用：

```python
import os
from openai import OpenAI

client = OpenAI(
    api_key=os.environ["DEEPCLUB_API_KEY"],
    base_url="https://deepclub.net/v1",
)

# 1. 文生图（size 需匹配模型在定价页公布的规格）
img = client.images.generate(
    model="gpt-image-2.5-flare",
    prompt="Studio product photo of matte black wireless headphones, subtle rim light",
    size="1024x1024",
)
print("Generated:", img.data[0].url or img.data[0].b64_json[:64])

# 2. 基于原图做编辑（/v1/images/edits）
with open("source.png", "rb") as f:
    edited = client.images.edit(
        model="nano-banana-pro",
        image=f,
        prompt="Keep the character design and pose, change outfit to cyberpunk rain jacket",
    )
print("Edited:", edited.data[0].url or edited.data[0].b64_json[:64])
```

> **提示**：各生图模型支持的分辨率/比例（如 `1024x1024`、`1K`/`2K`/`4K`）与对应单价可在 [https://deepclub.net/pricing](https://deepclub.net/pricing) 点开对应模型详情核对。

---

## 三、视频生成异步任务工程实践（Python）

视频生成属于耗时长的异步任务，正确的工程实践应包含：
1. **记录 `task_id`**：提交后第一时间持久化 `id`，避免进程重启后丢失任务导致重复提交；
2. **有界轮询 + HTTP 状态检查**：每隔 5~10 秒查一次 `GET /v1/videos/{task_id}`，设置最大超时时间；
3. **仅在 `completed` 时下载成片**：请求 `GET /v1/videos/{task_id}/content` 落盘。

```python
import os
import time
import requests

BASE_URL = "https://deepclub.net/v1"
HEADERS = {"Authorization": f"Bearer {os.environ['DEEPCLUB_API_KEY']}"}

# 1) 创建视频任务（Seedance 示例 5 秒，MiniMax-H3 按目录规格可用 10 秒）
submit_resp = requests.post(
    f"{BASE_URL}/videos",
    headers=HEADERS,
    data={
        "model": "Seedance-2.0-Fast",
        "prompt": "Slow tracking shot through a neon-lit rainy alley in Tokyo, 35mm film grain",
        "seconds": "5",
    },
    timeout=30,
)
submit_resp.raise_for_status()
task_id = submit_resp.json()["id"]
print("Created task:", task_id)

# 2) 有界轮询任务状态
deadline = time.time() + 600  # 最多等待 10 分钟
while time.time() < deadline:
    poll_resp = requests.get(f"{BASE_URL}/videos/{task_id}", headers=HEADERS, timeout=20)
    poll_resp.raise_for_status()
    data = poll_resp.json()
    status = data.get("status")
    print("Status:", status)
    if status == "completed":
        break
    if status == "failed":
        raise RuntimeError(f"Video generation failed: {data}")
    time.sleep(5)
else:
    raise TimeoutError(f"Task {task_id} did not finish in time; keep task_id to resume polling.")

# 3) 下载成片 MP4
content_resp = requests.get(f"{BASE_URL}/videos/{task_id}/content", headers=HEADERS, timeout=120)
content_resp.raise_for_status()
with open("output.mp4", "wb") as out:
    out.write(content_resp.content)
print("Saved output.mp4")
```

更多关于图片规格与视频模型（`Seedance-2.0-Fast`、`Seedance-2.0-Mini`、`MiniMax-H3`）的详细教程可参考：[https://deepclub.net/articles](https://deepclub.net/articles)。
