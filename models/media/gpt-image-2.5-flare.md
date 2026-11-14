# GPT Image 2.5 Flare 怎么用？镜头光晕与电影质感出图技巧

> **一句话定位**：gpt-image-2.5-flare 走 /v1/images/generations，按次计费。  
> **官网原生教程页**：[https://deepclub.net/articles/gpt-image-2-5-flare-how-to-use-guide](https://deepclub.net/articles/gpt-image-2-5-flare-how-to-use-guide) ｜ **实时定价与规格**：[https://deepclub.net/pricing/gpt-image-2.5-flare](https://deepclub.net/pricing/gpt-image-2.5-flare)

## 实时定价快照

| 计费项 | 实时参考价格（以官网为准） | 单位 |
|---|---|---|
| 每次调用 | ¥0.16 | 元 / 张 |

> 实时价格、可用状态与规格详情见官网：[https://deepclub.net/pricing/gpt-image-2.5-flare](https://deepclub.net/pricing/gpt-image-2.5-flare)

| 参数 | 值 |
| --- | --- |
| 模型 ID | `gpt-image-2.5-flare` |
| 类型 | 生图 |
| 调用端点 | `POST /v1/images/generations`、`POST /v1/images/edits` |
| 推荐协议 | OpenAI Images API（生成用 JSON，编辑用 multipart） |
| 客户端与 SDK | OpenAI Python SDK、curl |
| 计费方式 | 按次，实时价格见 [定价页](https://deepclub.net/pricing/gpt-image-2.5-flare) |

## GPT Image 2.5 Flare 适合做什么，不适合做什么？

可用于科幻、奇幻等需要光效的画面，效果以实测为准。

gpt-image-2.5-flare 是在售的 GPT Image 2.5 系列型号。本文以逆光、镜头光晕（Lens Flare）和晨雾光束为例说明提示词写法；不要仅凭型号名认定其擅长某种光效，请用相同提示词实测比较。

适合的场景：

- 科幻、奇幻题材的概念图
- 电影感海报
- 游戏场景氛围图

不太适合：尚未实测就直接用于批量生产；先用少量请求比较不同型号的效果。

## gpt-image-2.5-flare 的 Base URL 和 API Key 怎么获取？

本文示例的 Base URL 为 `https://deepclub.net/v1`，Key 在控制台创建。

1. 登录 [深云 DeepClub](https://deepclub.net/)，进入控制台的 **API 密钥** 页面。
2. 点击 **创建 API 密钥**，复制生成的 `sk-` 开头密钥。
3. 在客户端或代码中填写以下三项：

```text
Base URL: https://deepclub.net/v1
API Key:  sk-你的DeepClub密钥
Model:    gpt-image-2.5-flare
```

## 怎么用 Python 调用 gpt-image-2.5-flare 生成图片？

用 client.images.generate，model 填模型 ID。先在本地设置环境变量 `DEEPCLUB_API_KEY`（例如 `export DEEPCLUB_API_KEY=sk-你的DeepClub密钥`），下面的 Python 和 curl 示例都会读取它。

```python
import base64
import os
from openai import OpenAI

client = OpenAI(api_key=os.environ["DEEPCLUB_API_KEY"], base_url="https://deepclub.net/v1")

result = client.images.generate(
    model="gpt-image-2.5-flare",
    prompt="宇航员站在外星山脊上，身后恒星逆光，强烈的镜头光晕",
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

## 用 curl 怎么调用 gpt-image-2.5-flare？

POST 到 /v1/images/generations，传三个字段。

```bash
curl https://deepclub.net/v1/images/generations \
  -H "Authorization: Bearer $DEEPCLUB_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model": "gpt-image-2.5-flare", "prompt": "宇航员站在外星山脊上，身后恒星逆光，强烈的镜头光晕", "size": "1024x1024"}'
```

## gpt-image-2.5-flare 支持哪些图片尺寸？

当前发布规格为 1K，示例使用 `size="1024x1024"`，每次请求生成 1 张；不要传 `n` 大于 1 或其他尺寸。规格和价格以 [定价页](https://deepclub.net/pricing/gpt-image-2.5-flare) 为准。

## 怎么用 gpt-image-2.5-flare 编辑参考图？

使用 Images 编辑接口，multipart 中上传实际图片文件；文件路径不能放在 JSON 里代替上传。

```bash
curl https://deepclub.net/v1/images/edits \
  -H "Authorization: Bearer $DEEPCLUB_API_KEY" \
  -F model=gpt-image-2.5-flare \
  -F image=@reference.png \
  -F prompt="保留主体，将背景改为浅蓝色" \
  -F size=1024x1024
```

把 reference.png 换成本机真实图片路径，让 curl 自动生成 multipart Content-Type。返回仍按 Images 的 data 数组读取，每次 1 张，从 data[0] 取结果；蒙版和其他可选参数请先用小请求验证。

## GPT Image 2.5 Flare 怎么写出丁达尔光束效果？

写明光源方向、遮挡物和空气介质，如雾气或尘埃。

例如：清晨阳光从左上方穿过森林树冠缝隙，薄雾中形成清晰的光束，地面有斑驳光影。光源、遮挡物、介质三者缺一，光束效果会变弱。

想要镜头光晕，可以加上“逆光拍摄，镜头正对光源，出现六边形光斑”。

## gpt-image-2.5-flare API 怎么计费？

按次计费，每成功生成一次计一次。

实时价格见 [定价页](https://deepclub.net/pricing/gpt-image-2.5-flare)。

- 计费单位：按次，价格可能调整，以 [定价页](https://deepclub.net/pricing/gpt-image-2.5-flare) 实时显示为准。
- 用量明细：在控制台 **使用日志** 中按请求查看消耗。
- 充值方式：支持信用卡、微信支付、支付宝等，以控制台充值页显示为准。

## 调用 gpt-image-2.5-flare 报 401 怎么办？

401 表示密钥无效或缺失，先检查 Key 是否完整。

- 确认 API Key 前后没有空格或换行，且令牌未被禁用、未过期。
- 请求头格式为 Authorization: Bearer sk-...。
- 如果返回 404，检查请求地址是否与上面示例一致，避免重复或漏掉 /v1；模型 ID 必须完整匹配 `gpt-image-2.5-flare`。

## 用 gpt-image-2.5-flare 时请求内容会被存储吗？

不会，深云 DeepClub 实行零留存（ZDR）政策。

深云 DeepClub 实行 Zero Data Retention (ZDR) 零留存政策：请求的输入和输出内容不做持久化存储，详见 [隐私政策](https://deepclub.net/privacy-policy)。

## 使用 GPT Image 2.5 Flare 还有哪些常见问题？

以下问答汇总了调用时最常见的问题。

### 调用 gpt-image-2.5-flare 返回 401 Unauthorized 怎么办？

401 表示密钥无效或缺失，先检查 API Key 是否复制完整、前后没有空格。请求头格式为 Authorization: Bearer sk-...。

### gpt-image-2.5-flare 用哪个接口生成图片？

文生图用 POST /v1/images/generations 或 client.images.generate；参考图编辑用 POST /v1/images/edits，以 multipart 上传实际图片。这些在售图像模型不应套用文本 Chat、Responses 或 Gemini 原生配置。

### gpt-image-2.5-flare API 怎么计费？

按次计费，每成功生成一次计一次。实时价格见 https://deepclub.net/pricing/gpt-image-2.5-flare。

### GPT Image 2.5 Flare 怎么写出丁达尔光束效果？

写明光源方向、遮挡物和空气介质，如雾气或尘埃。

## 多框架与工程脚本扩展

- **Python / TypeScript OpenAI SDK**：`baseURL` 设为 `https://deepclub.net/v1`，生成调用 `images.generate`（`POST /v1/images/generations`），参考图微调调用 `images.edit`（`POST /v1/images/edits`，`multipart/form-data`）。
- **尺寸与并发建议**：每次请求保持 `n=1`；先使用 `1024x1024`（1K）验证构图与文字，确认后再切换该模型在定价页已发布的更高规格。
- **双返回落盘**：完整生产封装（同时兼容 `url` 与 `b64_json` 返回并自动落盘）见 [图片 API 工程实战手册](../../guides/media/image-api-engineering-handbook.md)。
