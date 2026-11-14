# 沉浸式翻译（Immersive Translate）接入深云 DeepClub：完整端点 URL 与低成本模型选型

**沉浸式翻译（`v1.35.2+`）**在配置自定义大模型时，有一个非常容易踩坑的细节：**客户端根据你填写的完整 API URL 路径（例如结尾是 `/v1/responses` 还是 `/v1/chat/completions`）来判断实际发送哪套协议**。如果只改动高级设置里的下拉选项而保留了旧路径，请求依然会按错的协议发送。

本文给出对接 **[深云 DeepClub](https://deepclub.net)** 的精确 URL 填法与适合网页对照/PDF 翻译的高性价比模型清单。

---

## 一、三种协议的完整接口地址（直接复制）

在沉浸式翻译中打开 **设置 → 翻译服务 → OpenAI**（或其他对应自定义入口），按你选择的模型填写 **完整接口地址（自定义 API 接口地址）**：

| 所选模型 | 推荐协议 | 完整自定义 API 接口地址（必须包含完整路径） |
|---|---|---|
| `deepseek-v4.1-flash`、`gemini-4-hydrogen`、`glm-5.3-flash`、`qwen3.8-flash` | Chat Completions | `https://deepclub.net/v1/chat/completions` |
| `gpt-6-luna`、`gpt-6.1-terra`、`gpt-6.1-sol`、`grok-4.7` | Responses API | `https://deepclub.net/v1/responses` |
| `claude-sonnet-5-5`、`claude-opus-5-5` | Anthropic Messages | `https://deepclub.net/v1/messages` |

> **提示**：不要只填 `https://deepclub.net/v1`。沉浸式翻译要求填写完整端点路径；如果旧版本还不支持 `/v1/responses`，建议升级至 `1.35.2+`，或选择同样开放了 `/v1/chat/completions` 兼容端点的模型并填 `https://deepclub.net/v1/chat/completions`。

---

## 二、图形配置步骤

1. 登录 [深云 DeepClub](https://deepclub.net) → **API 密钥**，新建一个 Key（`sk-...`）。建议给翻译插件单独建一个子 Key 并设置预算上限，用多少一目了然。
2. 打开浏览器扩展 **沉浸式翻译 → 设置 → 翻译服务 → OpenAI**。
3. 粘贴 DeepClub `API Key`。
4. 在 **模型** 一栏选择 **自定义模型**，输入目录里的精确型号（如 `gpt-6-luna`、`gemini-4-hydrogen` 或 `deepseek-v4.1-flash`）。
5. 展开 **更多设置 → 自定义 API 接口地址**，对照上表填入完整 URL（例如 `https://deepclub.net/v1/responses` 或 `https://deepclub.net/v1/chat/completions`）。
6. 如有 **每秒最大请求数 / 最大段落长度** 设置，按需保持默认或适度放宽并发，保存后点「测试」验证返回正常即可。

---

## 三、双语网页 / PDF 翻译高性价比模型推荐

翻译长网页和论文时每页要发几十个并发段落，选对单价与风格模型可以省下 80%+ 费用（以下单价请以 [实时定价页](https://deepclub.net/pricing) 为准）：

| 模型 ID | 参考价格（每百万 Token） | 适用翻译场景 | 填写的完整接口地址 |
|---|---|---|---|
| **`gpt-6-luna`** | 输入 ¥0.12 / 输出 ¥0.6 | **极速日常双语阅读**，成本极低，适合资讯流与长网页全文扫描 | `https://deepclub.net/v1/responses` |
| **`glm-5.3-flash`** | 输入 ¥0.44 / 输出 ¥1.54 | **中文地道润色与长句拆解**，英译中表达自然 | `https://deepclub.net/v1/chat/completions` |
| **`gemini-4-hydrogen`** | 输入 ¥0.6 / 输出 ¥3 | **高频轻量外文浏览、技术文档**，响应快且语序稳 | `https://deepclub.net/v1/chat/completions` |
| **`deepseek-v4.1-flash`** | 输入 ¥1.1 / 输出 ¥4.4 | **专业计算机 / 技术博客 / GitHub README**，术语准确度高 | `https://deepclub.net/v1/chat/completions` |
| **`glm-5.3` / `claude-sonnet-5-5`** | 见 [pricing](https://deepclub.net/pricing) | **英文 PDF 论文精读、法律合同、财务研报高精度翻译** | GLM：`.../v1/chat/completions`<br>Claude：`.../v1/messages` |

---

## 四、常见报错排查

- **`404 Not Found`**：十有八九是接口地址只填了 `https://deepclub.net/v1` 或末尾多了一个斜杠/路径拼接。请核对必须为 `https://deepclub.net/v1/chat/completions`、`https://deepclub.net/v1/responses` 或 `https://deepclub.net/v1/messages`。
- **`400` 参数不兼容**：若切了模型后报字段错误，先确认完整接口地址是否与该模型对应的协议一致，不要把 `/v1/responses` 填给仅开放 Chat 的模型（如 `deepseek-v4.1-flash`、`gemini-4-hydrogen`）。
- **数据安全**：深云 DeepClub 全平台实行 **Zero Data Retention (ZDR)** 零留存政策，不会保存你翻译的网页或 PDF 原文，详见 [https://deepclub.net/privacy-policy](https://deepclub.net/privacy-policy)。
