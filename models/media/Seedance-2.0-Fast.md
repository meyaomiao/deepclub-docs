# Seedance 2.0 Fast 怎么用？运镜提示词与 /v1/videos 异步任务接入

> **一句话定位**：Seedance-2.0-Fast 走 POST /v1/videos 异步生成视频。  
> **官网原生教程页**：[https://deepclub.net/articles/seedance-2-0-fast-how-to-use-guide](https://deepclub.net/articles/seedance-2-0-fast-how-to-use-guide) ｜ **实时定价与规格**：[https://deepclub.net/pricing/Seedance-2.0-Fast](https://deepclub.net/pricing/Seedance-2.0-Fast)

## 实时定价快照

| 计费项 | 实时参考价格（以官网为准） | 单位 |
|---|---|---|
| 每次调用 | ¥2.5 ~ ¥7.5 | 元 / 个 |

> 实时价格、可用状态与规格详情见官网：[https://deepclub.net/pricing/Seedance-2.0-Fast](https://deepclub.net/pricing/Seedance-2.0-Fast)

| 参数 | 值 |
| --- | --- |
| 模型 ID | `Seedance-2.0-Fast` |
| 类型 | 视频 |
| 调用端点 | `POST /v1/videos`，轮询 `GET /v1/videos/{task_id}` |
| 推荐协议 | OpenAI Videos API（multipart 表单） |
| 客户端与 SDK | curl、Python requests |
| 计费方式 | 按次，实时价格见 [定价页](https://deepclub.net/pricing/Seedance-2.0-Fast) |

## Seedance 2.0 Fast 适合做什么，不适合做什么？

适合需要运镜控制的高清短视频和商业成片。

Seedance-2.0-Fast 是支持电影级运镜控制（Pan / Tilt / Zoom / Crane）的高清视频生成模型，动作流畅，光影质感接近商业成片。

适合的场景：

- 广告、产品展示短片
- 带推拉摇移运镜的叙事镜头
- 社交平台的高质量短视频

不太适合：需要大量试错的分镜草稿，用 Seedance-2.0-Mini 成本更低。

## Seedance-2.0-Fast 的 Base URL 和 API Key 怎么获取？

本文示例的 Base URL 为 `https://deepclub.net/v1`，Key 在控制台创建。

1. 登录 [深云 DeepClub](https://deepclub.net/)，进入控制台的 **API 密钥** 页面。
2. 点击 **创建 API 密钥**，复制生成的 `sk-` 开头密钥。
3. 在客户端或代码中填写以下三项：

```text
Base URL: https://deepclub.net/v1
API Key:  sk-你的DeepClub密钥
Model:    Seedance-2.0-Fast
```

## 怎么用 API 生成 Seedance-2.0-Fast 视频？

先 POST /v1/videos 提交任务，再轮询状态，完成后下载。

第一步，用 multipart 表单提交任务，字段为 `model`、`prompt`、`seconds`：

```bash
curl https://deepclub.net/v1/videos \
  -H "Authorization: Bearer $DEEPCLUB_API_KEY" \
  -F model=Seedance-2.0-Fast \
  -F prompt="镜头缓慢推近，清晨海边的灯塔，海浪拍打礁石" \
  -F seconds=5
```

返回 JSON 中的 `id` 就是任务 ID，初始 `status` 为 `queued`。

第二步，把首次响应中的 `id` 复制到 TASK_ID，再轮询状态，直到 `status` 变为 `completed` 或 `failed`。重查状态时沿用这个 ID，不要重复提交生成任务：

```bash
TASK_ID="替换为首次响应中的id"
curl https://deepclub.net/v1/videos/$TASK_ID \
  -H "Authorization: Bearer $DEEPCLUB_API_KEY"
```

第三步，任务完成后下载视频文件：

```bash
curl -L https://deepclub.net/v1/videos/$TASK_ID/content \
  -H "Authorization: Bearer $DEEPCLUB_API_KEY" \
  -o output.mp4
```

`seconds` 是视频时长（秒），本模型示例使用 5 秒。其他时长和分辨率必须匹配 [定价页](https://deepclub.net/pricing/Seedance-2.0-Fast) 已发布的规格，否则可能返回 400。

## 用 Python 怎么轮询 Seedance-2.0-Fast 视频任务？

用 requests 提交表单，每隔数秒查询一次状态。

```python
import time
import requests

BASE = "https://deepclub.net/v1"
headers = {"Authorization": "Bearer sk-你的DeepClub密钥"}

resp = requests.post(
    f"{BASE}/videos",
    headers=headers,
    files={
        "model": (None, "Seedance-2.0-Fast"),
        "prompt": (None, "镜头缓慢推近，清晨海边的灯塔，海浪拍打礁石"),
        "seconds": (None, "5"),
    },
    timeout=120,
)
resp.raise_for_status()
task_id = resp.json()["id"]

print("保存任务 ID，后续查询使用：", task_id)
deadline = time.monotonic() + 600
while time.monotonic() < deadline:
    poll = requests.get(f"{BASE}/videos/{task_id}", headers=headers, timeout=30)
    poll.raise_for_status()
    task = poll.json()
    if task["status"] in ("completed", "failed"):
        break
    time.sleep(10)
else:
    raise TimeoutError(f"稍后用同一任务 ID 继续查询，不要重复提交：{task_id}")

if task["status"] == "completed":
    video = requests.get(f"{BASE}/videos/{task_id}/content", headers=headers, timeout=120)
    video.raise_for_status()
    with open("output.mp4", "wb") as f:
        f.write(video.content)
else:
    print(task.get("error"))
```

## Seedance 2.0 Fast 的运镜提示词怎么写？

按“镜头运动 + 主体 + 环境 + 光线”的顺序描述。

例如：镜头从低角度缓慢上摇（Tilt Up），一位穿红色风衣的女孩站在雨夜街头，霓虹灯倒映在积水中，侧逆光。一个提示词只写一种主要运镜，效果更稳定。

常用运镜词：推近（Zoom In / Dolly In）、拉远（Zoom Out）、横摇（Pan）、上下摇（Tilt）、升降（Crane）、环绕（Orbit）。

## Seedance-2.0-Fast API 怎么计费？

按次计费，每成功生成一次计一次。

实时价格见 [定价页](https://deepclub.net/pricing/Seedance-2.0-Fast)。

- 计费单位：按次，价格可能调整，以 [定价页](https://deepclub.net/pricing/Seedance-2.0-Fast) 实时显示为准。
- 用量明细：在控制台 **使用日志** 中按请求查看消耗。
- 充值方式：支持信用卡、微信支付、支付宝等，以控制台充值页显示为准。

## 调用 Seedance-2.0-Fast 报 401 怎么办？

401 表示密钥无效或缺失，先检查 Key 是否完整。

- 确认 API Key 前后没有空格或换行，且令牌未被禁用、未过期。
- 请求头格式为 Authorization: Bearer sk-...。
- 如果返回 404，检查请求地址是否与上面示例一致，避免重复或漏掉 /v1；模型 ID 必须完整匹配 `Seedance-2.0-Fast`。

## 用 Seedance-2.0-Fast 时请求内容会被存储吗？

不会，深云 DeepClub 实行零留存（ZDR）政策。

深云 DeepClub 实行 Zero Data Retention (ZDR) 零留存政策：请求的输入和输出内容不做持久化存储，详见 [隐私政策](https://deepclub.net/privacy-policy)。

## 使用 Seedance 2.0 Fast 还有哪些常见问题？

以下问答汇总了调用时最常见的问题。

### 调用 Seedance-2.0-Fast 返回 401 Unauthorized 怎么办？

401 表示密钥无效或缺失，先检查 API Key 是否复制完整、前后没有空格。请求头格式为 Authorization: Bearer sk-...。

### Seedance-2.0-Fast 用哪个接口生成视频？

先用 POST /v1/videos 以 multipart 表单提交 model、prompt、seconds，再轮询 GET /v1/videos/{task_id}，完成后从 GET /v1/videos/{task_id}/content 下载。

### Seedance-2.0-Fast API 怎么计费？

按次计费，每成功生成一次计一次。实时价格见 https://deepclub.net/pricing/Seedance-2.0-Fast。

### Seedance 2.0 Fast 的运镜提示词怎么写？

按“镜头运动 + 主体 + 环境 + 光线”的顺序描述。

## 异步任务恢复与工程扩展

- **推荐参考时长**：当前目录参考规格示例为 `seconds=5`，请先到 [定价详情页](https://deepclub.net/pricing/Seedance-2.0-Fast) 确认支持时长与分辨率。
- **防重复扣费铁律**：`POST /v1/videos` 收到响应后立即持久化 `id`；如遇网络中断，仅查询 `GET /v1/videos/{task_id}` 与下载 `/content`，切勿直接重发提交请求。
- **可断点恢复脚本**：完整带状态落盘、退避轮询与 `--resume` 的 Python 工程模板见 [视频异步任务工程实战](../../guides/media/video-async-tasks-recovery.md)。
