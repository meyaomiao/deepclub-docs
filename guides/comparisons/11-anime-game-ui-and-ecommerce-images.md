# 二次元原画、游戏 UI 与电商商品图对比：`nano-banana-pro` vs `nano-banana-2` vs `gemini-3.1-flash-image`

做独立游戏美术、表情包套系、动漫角色设定，或者批量生产电商白底/场景主图时，用同一套 OpenAI Images 接口就可以在 `nano-banana-pro`、`nano-banana-2` 和 `gemini-3.1-flash-image` 之间自由切换——且**三款都支持 1K (`1024x1024`)、2K (`2048x2048`)、4K (`4096x4096`) 全规格**，统一按次计费（¥0.24/张，实时以 [定价页](https://deepclub.net/pricing) 为准）。

---

## 一、三款模型场景定位对比

| 模型 ID | 端点（生成 & 编辑） | 支持尺寸 | 参考单价 | 核心擅长场景 |
|---|---|---|---|---|
| **`nano-banana-pro`** | `/v1/images/generations`<br>`/v1/images/edits` | 1K / 2K / 4K | ¥0.24 / 张 | **二次元角色立绘、三视图（正/侧/背）与多姿态角色特征保持一致** |
| **`nano-banana-2`** | `/v1/images/generations`<br>`/v1/images/edits` | 1K / 2K / 4K | ¥0.24 / 张 | **2D 扁平插画（Flat Design）、游戏 Icon、App 空状态插画与可爱表情包** |
| **`gemini-3.1-flash-image`** | `/v1/images/generations`<br>`/v1/images/edits` | 1K / 2K / 4K | ¥0.24 / 张 | **高并发电商商品图、快速场景换背景（配合 `/v1/images/edits`）与社媒配图** |

---

## 二、角色一致性 & 电商换背景实战写法

### 1. 用 `nano-banana-pro` 保持角色一致性（生成 + 参考图微调组合）
- 第一步：写一段**固定角色锚点前缀**（发型、发色、瞳色、服装配色、配饰），用 `/v1/images/generations` 生成角色三视图；
- 第二步：把定稿三视图或半身立绘作为 `image` 上传到 **`POST /v1/images/edits`**，在提示词中写清“保持角色脸部特征与服装完全不变，只调整姿势为手持法杖、仰视视角”。

### 2. 用 `gemini-3.1-flash-image` 批量产出电商场景图
- 把产品白底实拍图传给 **`POST /v1/images/edits`**，依次替换背景描述（例如：“保持保温杯外观和 Logo 不变，放置在清晨露营木桌上，背景虚化森林” / “放置在极简大理石厨房台面，柔和侧光”），每张可按需选 `1024x1024`（快审）或 `2048x2048`（上架详情页）。

工程模板见：[../media/image-api-engineering-handbook.md](../media/image-api-engineering-handbook.md)。
