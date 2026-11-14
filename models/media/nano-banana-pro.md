# Nano Banana Pro 怎么用？多视角角色一致性与原画出图教程

> **一句话定位**：nano-banana-pro 走 /v1/images/generations，按次计费。  
> **官网原生教程页**：[https://deepclub.net/articles/nano-banana-pro-how-to-use-guide](https://deepclub.net/articles/nano-banana-pro-how-to-use-guide) ｜ **实时定价与规格**：[https://deepclub.net/pricing/nano-banana-pro](https://deepclub.net/pricing/nano-banana-pro)

## 实时定价快照

| 计费项 | 实时参考价格（以官网为准） | 单位 |
|---|---|---|
| 每次调用 | ¥0.24 | 元 / 张 |

> 实时价格、可用状态与规格详情见官网：[https://deepclub.net/pricing/nano-banana-pro](https://deepclub.net/pricing/nano-banana-pro)

| 参数 | 值 |
| --- | --- |
| 模型 ID | `nano-banana-pro` |
| 类型 | 生图 |
| 调用端点 | `POST /v1/images/generations`、`POST /v1/images/edits` |
| 推荐协议 | OpenAI Images API（生成用 JSON，编辑用 multipart） |
| 客户端与 SDK | OpenAI Python SDK、curl |
| 计费方式 | 按次，实时价格见 [定价页](https://deepclub.net/pricing/nano-banana-pro) |

## Nano Banana Pro 适合做什么，不适合做什么？

适合二次元角色立绘和多视角角色设定图。

nano-banana-pro 是面向二次元、动漫和游戏角色立绘的 Pro 版本，能在角度和姿态变化时保持角色面部与服装特征一致。

适合的场景：

- 角色的正面、侧面、背面三视图
- 同一角色的不同动作立绘
- 游戏、动漫的概念设定

不太适合：简单的扁平图标，nano-banana-2 更轻量。

## nano-banana-pro 的 Base URL 和 API Key 怎么获取？

本文示例的 Base URL 为 `https://deepclub.net/v1`，Key 在控制台创建。

1. 登录 [深云 DeepClub](https://deepclub.net/)，进入控制台的 **API 密钥** 页面。
2. 点击 **创建 API 密钥**，复制生成的 `sk-` 开头密钥。
3. 在客户端或代码中填写以下三项：

```text
Base URL: https://deepclub.net/v1
API Key:  sk-你的DeepClub密钥
Model:    nano-banana-pro
```

## 怎么用 Python 调用 nano-banana-pro 生成图片？

用 client.images.generate，model 填模型 ID。先在本地设置环境变量 `DEEPCLUB_API_KEY`（例如 `export DEEPCLUB_API_KEY=sk-你的DeepClub密钥`），下面的 Python 和 curl 示例都会读取它。

```python
import base64
import os
from openai import OpenAI

client = OpenAI(api_key=os.environ["DEEPCLUB_API_KEY"], base_url="https://deepclub.net/v1")

result = client.images.generate(
    model="nano-banana-pro",
    prompt="银发少女剑士的角色三视图，正面、侧面、背面，白色背景",
    size="1024x1024",
)

image = result.data[0]
if image.b64_json:
    with open("output.png", "wb") as f:
        f.write(base64.b64decode(image.b64_json))
else:
    print(image.url)
```

返回结果可能是 base64 数据（b64_json）或图片链接（url），上面的代码两种都处理了。

## 用 curl 怎么调用 nano-banana-pro？

POST 到 /v1/images/generations，传三个字段。

```bash
curl https://deepclub.net/v1/images/generations \
  -H "Authorization: Bearer $DEEPCLUB_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model": "nano-banana-pro", "prompt": "银发少女剑士的角色三视图，正面、侧面、背面，白色背景", "size": "1024x1024"}'
```

## nano-banana-pro 支持哪些图片尺寸？

已发布 1K / 2K / 4K 规格，通过 `size` 选择（`1024x1024`、`2048x2048`、`4096x4096`），模型 ID 不变；每次请求生成 1 张，不要传 `n` 大于 1。各规格价格以 [定价页](https://deepclub.net/pricing/nano-banana-pro) 为准。

## 怎么用 nano-banana-pro 编辑参考图？

使用 Images 编辑接口，multipart 中上传实际图片文件；文件路径不能放在 JSON 里代替上传。

```bash
curl https://deepclub.net/v1/images/edits \
  -H "Authorization: Bearer $DEEPCLUB_API_KEY" \
  -F model=nano-banana-pro \
  -F image=@reference.png \
  -F prompt="保留主体，将背景改为浅蓝色" \
  -F size=1024x1024
```

把 reference.png 换成本机真实图片路径，让 curl 自动生成 multipart Content-Type。返回仍按 Images 的 data 数组读取，每次 1 张，从 data[0] 取结果；蒙版和其他可选参数请先用小请求验证。

## Nano Banana Pro 怎么保持同一角色多张图一致？

每次都使用同一段详细的角色设定描述。

把角色的发型、发色、瞳色、服装、配饰写成固定的设定段落，每次生成时原样放在提示词开头，再追加角度和动作描述。

需要更高精度的原画时，把 size 改为 `2048x2048` 或 `4096x4096`；需要复用角色时，可通过本文 Images 编辑接口上传参考图。

## nano-banana-pro API 怎么计费？

按次计费，每成功生成一次计一次。

实时价格见 [定价页](https://deepclub.net/pricing/nano-banana-pro)。

- 计费单位：按次，价格可能调整，以 [定价页](https://deepclub.net/pricing/nano-banana-pro) 实时显示为准。
- 用量明细：在控制台 **使用日志** 中按请求查看消耗。
- 充值方式：支持信用卡、微信支付、支付宝等，以控制台充值页显示为准。

## 调用 nano-banana-pro 报 401 怎么办？

401 表示密钥无效或缺失，先检查 Key 是否完整。

- 确认 API Key 前后没有空格或换行，且令牌未被禁用、未过期。
- 请求头格式为 Authorization: Bearer sk-...。
- 如果返回 404，检查请求地址是否与上面示例一致，避免重复或漏掉 /v1；模型 ID 必须完整匹配 `nano-banana-pro`。

## 用 nano-banana-pro 时请求内容会被存储吗？

不会，深云 DeepClub 实行零留存（ZDR）政策。

深云 DeepClub 实行 Zero Data Retention (ZDR) 零留存政策：请求的输入和输出内容不做持久化存储，详见 [隐私政策](https://deepclub.net/privacy-policy)。

## 使用 Nano Banana Pro 还有哪些常见问题？

以下问答汇总了调用时最常见的问题。

### 调用 nano-banana-pro 返回 401 Unauthorized 怎么办？

401 表示密钥无效或缺失，先检查 API Key 是否复制完整、前后没有空格。请求头格式为 Authorization: Bearer sk-...。

### nano-banana-pro 用哪个接口生成图片？

文生图用 POST /v1/images/generations 或 client.images.generate；参考图编辑用 POST /v1/images/edits，以 multipart 上传实际图片。这些在售图像模型不应套用文本 Chat、Responses 或 Gemini 原生配置。

### nano-banana-pro API 怎么计费？

按次计费，每成功生成一次计一次。实时价格见 https://deepclub.net/pricing/nano-banana-pro。

### Nano Banana Pro 怎么保持同一角色多张图一致？

每次都使用同一段详细的角色设定描述。

## 多框架与工程脚本扩展

- **Python / TypeScript OpenAI SDK**：`baseURL` 设为 `https://deepclub.net/v1`，生成调用 `images.generate`（`POST /v1/images/generations`），参考图微调调用 `images.edit`（`POST /v1/images/edits`，`multipart/form-data`）。
- **尺寸与并发建议**：每次请求保持 `n=1`；先使用 `1024x1024`（1K）验证构图与文字，确认后再切换该模型在定价页已发布的更高规格。
- **双返回落盘**：完整生产封装（同时兼容 `url` 与 `b64_json` 返回并自动落盘）见 [图片 API 工程实战手册](../../guides/media/image-api-engineering-handbook.md)。
