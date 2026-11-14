# 视频生成异步任务工程实战（`/v1/videos` 持久化、断点恢复与防重复扣费）

视频生成属于**高耗时、按次计费**的异步任务。如果在生产系统里直接像调聊天接口那样傻等，一旦触发反代/客户端超时并盲目重试，就会造成**同一段视频被重复创建并多次扣费**。

本文结合 **[深云 DeepClub](https://deepclub.net)** 视频端点规范（`Seedance-2.0-Fast`、`Seedance-2.0-Mini`、`MiniMax-H3`），给出一套带 `task_id` 持久化、有界退避轮询和断点恢复的 Python 工程参考实现。

---

## 一、三款在售视频模型核心参数

> 实时规格、价格与开放时长请以 [https://deepclub.net/pricing](https://deepclub.net/pricing) 为准：

| 模型 ID | 提交端点（`multipart/form-data`） | 示例 `seconds` | 参考单价 | 核心定位 |
|---|---|---|---|---|
| `Seedance-2.0-Mini` | `POST /v1/videos` | `5` | ¥1.5 ~ ¥4.5 / 个 | 分镜快速预演、低成本大批量草样验证 |
| `Seedance-2.0-Fast` | `POST /v1/videos` | `5` | ¥2.5 ~ ¥7.5 / 个 | 运镜控制、商业短片、广告镜头主力 |
| `MiniMax-H3` | `POST /v1/videos` | `10` | ¥2.0 ~ ¥4.5 / 个 | 雨雪流体物理真实感、连贯人物动作片段 |

> **关键细节**：`MiniMax-H3` 与 `Seedance-2.0-*` 发布的合法 `seconds` 和分辨率规格不同，切勿把某一款的参数直接硬套给另一款。

---

## 二、异步任务三步流与状态机

```text
POST /v1/videos (multipart)
   │ 立即持久化返回的 id (task_id)
   ▼
GET /v1/videos/{task_id}  ──▶ queued / in_progress (继续等待)
   │
   ├──▶ failed     (停止轮询，记录失败原因，不再下载)
   └──▶ completed  ──▶ GET /v1/videos/{task_id}/content (流式下载 MP4)
```

---

## 三、带状态持久化与断点恢复的 Python 工程脚本

这段脚本把 `task_id` 保存到本地 `video_task_state.json`。即使在轮询阶段按了 `Ctrl+C` 或进程重启，再次运行同一命令时传入 `--resume`，就会直接续查已有 `task_id` 而不重复扣费：

```python
import json
import os
import sys
import time
from pathlib import Path
import requests

BASE_URL = "https://deepclub.net/v1"
HEADERS = {"Authorization": f"Bearer {os.environ['DEEPCLUB_API_KEY']}"}
STATE_FILE = Path("video_task_state.json")


def submit_video_task(model: str, prompt: str, seconds: str) -> str:
    resp = requests.post(
        f"{BASE_URL}/videos",
        headers=HEADERS,
        data={"model": model, "prompt": prompt, "seconds": seconds},
        timeout=30,
    )
    resp.raise_for_status()
    payload = resp.json()
    task_id = payload["id"]
    # 收到 task_id 第一时间落盘
    STATE_FILE.write_text(
        json.dumps({"task_id": task_id, "model": model, "status": payload.get("status", "queued")}, indent=2),
        encoding="utf-8",
    )
    print(f"[CREATED] task_id={task_id}")
    return task_id


def wait_for_completion(task_id: str, max_wait_seconds: int = 900) -> dict:
    deadline = time.time() + max_wait_seconds
    interval = 5
    while time.time() < deadline:
        r = requests.get(f"{BASE_URL}/videos/{task_id}", headers=HEADERS, timeout=20)
        r.raise_for_status()
        data = r.json()
        status = data.get("status")
        print(f"[POLL] task_id={task_id} status={status}")
        if status == "completed":
            return data
        if status == "failed":
            raise RuntimeError(f"Video task {task_id} failed: {json.dumps(data, ensure_ascii=False)}")
        time.sleep(interval)
        # 温和退避到 15 秒上限，避免高频无意义轮询
        interval = min(interval + 2, 15)
    raise TimeoutError(f"Timed out after {max_wait_seconds}s; rerun with --resume for task_id={task_id}")


def download_video(task_id: str, out_file: str = "output.mp4") -> None:
    with requests.get(f"{BASE_URL}/videos/{task_id}/content", headers=HEADERS, stream=True, timeout=180) as r:
        r.raise_for_status()
        with open(out_file, "wb") as f:
            for chunk in r.iter_content(chunk_size=1024 * 256):
                if chunk:
                    f.write(chunk)
    print(f"[DOWNLOADED] {out_file}")


if __name__ == "__main__":
    if "--resume" in sys.argv and STATE_FILE.exists():
        saved = json.loads(STATE_FILE.read_text(encoding="utf-8"))
        tid = saved["task_id"]
        print(f"[RESUME] Continuing saved task_id={tid}")
    else:
        tid = submit_video_task(
            model="Seedance-2.0-Mini",
            prompt="Slow drone push-in over misty bamboo forest at golden sunrise, cinematic 24fps",
            seconds="5",
        )

    wait_for_completion(tid)
    download_video(tid, out_file=f"{tid}.mp4")
```

---

## 四、重要避坑检查点

1. **必须检查 HTTP 状态码**：轮询时务必调用 `r.raise_for_status()`（或在 curl 中使用 `curl -fsS`），避免网络抖动返回 HTML 502 时脚本误判状态或静默循环。
2. **先测试再批跑**：先用 `Seedance-2.0-Mini`（¥1.5 起）验证提示词和运动描述是否符合预期，再切换到 `Seedance-2.0-Fast` 或 `MiniMax-H3` 生成高质终版。
3. **绝对不要把“连接中断”当成任务失败**：只要 `POST /v1/videos` 已经提交成功并返回了 `id`，后端仍会继续渲染；断线后重连查询 `GET /v1/videos/{task_id}` 即可。
