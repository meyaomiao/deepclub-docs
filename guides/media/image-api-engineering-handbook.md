# 图片生成与参考图编辑 API 工程实战手册（`/v1/images/generations` & `/v1/images/edits`）

**[深云 DeepClub](https://deepclub.net)** 在售的 10 款图片模型（`gpt-image-2*`、`gemini-3-pro-image`、`gemini-3.1-flash-image`、`nano-banana-2`、`nano-banana-pro`）全部统一挂载在 OpenAI Images 规范端点下。

**核心铁律**：哪怕模型名字里包含 `gpt` 或 `gemini`，**生图与修图也绝不能发往 `/v1/chat/completions`、`/v1/responses` 或 Gemini 原生 `:generateContent`**。

---

## 一、10 款在售图片模型规格与端点一览

> 价格按次（每张图）计费，实时单价以 [https://deepclub.net/pricing](https://deepclub.net/pricing) 为准：

| 模型 ID | 发布支持尺寸（`size`） | 适用特点示例 | 参考单价 |
|---|---|---|---|
| `gpt-image-2` | `1024x1024`、`4096x4096` | 通用海报排版、文字元素与设计草图 | ¥0.16 ~ ¥0.20 / 张 |
| `gpt-image-2-adobe` | `1024x1024` | 包装、广告物料与设计流程草图（需在后期转 CMYK） | ¥0.24 / 张 |
| `gpt-image-2.5-flare` | `1024x1024` | 逆光光晕、丁达尔光束、科幻与概念视觉 | ¥0.16 / 张 |
| `gpt-image-2.5-flare-adobe` | `1024x1024` | 黑底光效素材、方便在 PS 中滤色叠加 | ¥0.24 / 张 |
| `gpt-image-2.5-sunburst` | `1024x1024` | 黄金时刻自然光人像与风光 | ¥0.16 / 张 |
| `gpt-image-2.5-sunburst-adobe` | `1024x1024` | 杂志封面预留标题构图、文旅视觉 | ¥0.24 / 张 |
| `gemini-3-pro-image` | `1024x1024`、`2048x2048`、`4096x4096` | 精细材质光影、商业产品静物与高分辨率海报 | ¥0.30 / 张 |
| `gemini-3.1-flash-image` | `1024x1024`、`2048x2048`、`4096x4096` | 电商主图高并发生成、白底图换背景 | ¥0.24 / 张 |
| `nano-banana-2` | `1024x1024`、`2048x2048`、`4096x4096` | 2D 扁平插画、App 引导页、Icon 与表情包 | ¥0.24 / 张 |
| `nano-banana-pro` | `1024x1024`、`2048x2048`、`4096x4096` | 二次元立绘、角色三视图与跨图一致性设定 | ¥0.24 / 张 |

> **重要规格约束**：每次请求生成 **`n=1`**（不要传 `n > 1`）；部分仅发布 1K 的型号传 `2048x2048` / `4096x4096` 会被拒绝或按当前目录规格限制。具体可支持规格以实时定价页点开详情为准。

---

## 二、生产级 Python 封装（兼容 `url` 与 `b64_json` 双返回 + 自动落盘）

不同后端或型号在响应 `data[0]` 中可能返回 `url` 或 `b64_json`，工程代码应同时兼容两种结构：

```python
import base64
import os
from pathlib import Path
import requests
from openai import OpenAI

client = OpenAI(
    api_key=os.environ["DEEPCLUB_API_KEY"],
    base_url="https://deepclub.net/v1",
)


def save_image_result(data_item, out_path: str) -> Path:
    dest = Path(out_path)
    dest.parent.mkdir(parents=True, exist_ok=True)
    if getattr(data_item, "b64_json", None):
        dest.write_bytes(base64.b64decode(data_item.b64_json))
        return dest
    if getattr(data_item, "url", None):
        r = requests.get(data_item.url, timeout=60)
        r.raise_for_status()
        dest.write_bytes(r.content)
        return dest
    raise ValueError("Image response contained neither b64_json nor url")


# 1) 文生图（JSON 请求 -> POST /v1/images/generations）
gen = client.images.generate(
    model="gemini-3-pro-image",
    prompt="Studio macro photo of brushed titanium mechanical watch on dark slate stone, dramatic rim light",
    size="2048x2048",
    n=1,
)
print("Saved generated image to:", save_image_result(gen.data[0], "outputs/watch_2k.png"))

# 2) 参考图编辑（multipart 上传 -> POST /v1/images/edits）
with open("outputs/watch_2k.png", "rb") as src:
    edited = client.images.edit(
        model="gemini-3-pro-image",
        image=src,
        prompt="Keep the watch untouched, replace dark slate with warm minimalist oak tabletop in morning sunlight",
        size="2048x2048",
        n=1,
    )
print("Saved edited image to:", save_image_result(edited.data[0], "outputs/watch_oak_2k.png"))
```

---

## 三、`curl` 直接测试（文生图 vs `multipart` 改图）

```bash
# 1. 文生图
curl https://deepclub.net/v1/images/generations \
  -H "Authorization: Bearer $DEEPCLUB_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-image-2",
    "prompt": "Minimalist specialty coffee poster, clean kraft paper texture, subtle botanical sketch",
    "size": "1024x1024",
    "n": 1
  }'

# 2. 基于参考图编辑（-F 自动走 multipart/form-data）
curl https://deepclub.net/v1/images/edits \
  -H "Authorization: Bearer $DEEPCLUB_API_KEY" \
  -F model=nano-banana-pro \
  -F image=@character_sheet.png \
  -F prompt="Keep the silver-haired character design and outfit, switch pose to waving happily" \
  -F n=1
```

---

## 四、实用工程技巧

1. **印刷与后期工作流**：带 `-adobe` 后缀并不意味着 API 会直接吐出 CMYK 印刷色域；无论使用哪款模型，出图后均应在 Photoshop / Illustrator 中转 CMYK、检查分辨率并打样后再正式付印。
2. **系列一致性**：使用 `nano-banana-2` 做套系图标或 `nano-banana-pro` 做角色多角度时，把风格/特征段落（色号、描边粗细、发色瞳色、服装细节）写死为模板前缀，每次仅替换动作或视角，效果稳定且可配合 `/v1/images/edits` 迭代微调。
3. **超时设置**：`2K`/`4K` 生图耗时明显高于文本接口，HTTP 客户端建议将请求超时设为 `60s~120s`。
