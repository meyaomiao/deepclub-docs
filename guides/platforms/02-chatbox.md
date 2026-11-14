# Chatbox 接入深云 DeepClub 完整指南（Responses / Claude / Chat 三模式精确路径）

**Chatbox**（`v1.24.3+`）在自定义提供方（Custom Provider）中把协议拆成了三种独立模式，并在配置界面同时提供了 **API Host** 与 **API Path** 两个输入框。如果沿用旧教程乱拼 `/v1`，很容易拼出 `/v1/v1/...` 导致 `404`。

本文对照 Chatbox 自定义提供方实际路径拼接规则，整理出 **[深云 DeepClub](https://deepclub.net)** 全模型族的精确配置方法。

---

## 一、三类模型对应的精确配置

| 模型家族 | API 模式（API Mode） | API Host | API Path | 最终请求地址 |
|---|---|---|---|---|
| **GPT 文本 & Grok**<br>(`gpt-6.1-sol`、`gpt-6-astra`、`gpt-6-luna`、`gpt-6.1-terra`、`gpt-5.6-sol`、`grok-4.7`) | **OpenAI Responses API Compatible** | `https://deepclub.net/v1` | `/responses` | `https://deepclub.net/v1/responses` |
| **Claude 系列**<br>(`claude-opus-5-5`、`claude-sonnet-5-5`、`claude-fable-5-1`、`claude-fable-5-2`) | **Claude API Compatible** | `https://deepclub.net/v1`（注：勿漏 `/v1`） | `/messages`（默认） | `https://deepclub.net/v1/messages` |
| **DeepSeek / Gemini 文本 / GLM / Kimi / Qwen** | **OpenAI API Compatible** | `https://deepclub.net/v1` | `/chat/completions` | `https://deepclub.net/v1/chat/completions` |

> **为什么 Claude 在 Chatbox 里 Host 要写 `/v1`，而在 Anthropic Python SDK / Claude Code 里写站根？**  
> 因为 Chatbox 内部的 `custom-claude` 定义直接把 Host 与 `/messages` 拼接，**不会**自己补 `/v1`；而官方 Anthropic SDK 会自己补 `/v1/messages`。这是不同客户端源码实现差异，填好后可以在发一条测试消息时检查 DeepClub 控制台日志或报错 URL 来验证最终地址只包含一次 `/v1`。

---

## 二、图形界面操作步骤

1. 打开 [深云 DeepClub](https://deepclub.net) 控制台 → **API 密钥** 页面，生成一个密钥（`sk-...`）。
2. 打开 Chatbox，进入 **设置 → 模型提供方**，点击底部 **添加自定义提供方（Add Custom Provider）**。
3. 如果你同时使用 GPT-6（Responses）、Claude（Messages）和 DeepSeek/Gemini（Chat），**建议分别添加 2~3 个自定义提供方**（例如 `DeepClub Responses`、`DeepClub Claude`、`DeepClub Chat`），避免在同一个提供方下混用端点不同的模型。
4. 在对应提供方中添加精确的模型 ID（可直接在 [https://deepclub.net/pricing](https://deepclub.net/pricing) 复制）：
   - `DeepClub Responses` 添加：`gpt-6.1-sol`、`gpt-6-astra`、`gpt-6-luna`、`gpt-6.1-terra`、`grok-4.7`
   - `DeepClub Claude` 添加：`claude-sonnet-5-5`、`claude-opus-5-5`、`claude-fable-5-1`、`claude-fable-5-2`
   - `DeepClub Chat` 添加：`deepseek-v4.1-flash`、`gemini-4-hydrogen`、`gemini-4-argon`、`glm-5.3`、`kimi-k3`、`qwen3.8-flash`
5. 保存后新建对话，在右上角选择刚添加的模型测试。

---

## 三、常见问题排查

- **报 `404 Not Found`**：检查 `API Host` + `API Path` 合起来是否出现了 `/v1/v1/` 或完全少了 `/v1/`。正确结果只能是 `/v1/responses`、`/v1/messages` 或 `/v1/chat/completions`。
- **Claude 5.x 报 400 参数错误**：在 Chatbox 对话设置中关闭自定义的温度（Temperature）或 Top P 覆盖，保持默认（`claude-opus-5-5`、`claude-sonnet-5-5`、`claude-fable-5-2` 在原生 Messages 下仅接受默认采样）。
- **旧版 Chatbox 没有 Responses 模式怎么办？**：建议升级到 `1.24.3+`；若暂不升级，可在 `DeepClub Chat`（`OpenAI API Compatible` + `/chat/completions`）下以兼容模式调用 `gpt-6.1-sol` 等同样开放了 Chat 端点的模型。

更多模型介绍与多客户端教程：
- 官网模型教程：[https://deepclub.net/articles](https://deepclub.net/articles)
- 官方定价目录：[https://deepclub.net/pricing](https://deepclub.net/pricing)
- API 文档中心：[https://deepclub.net/docs](https://deepclub.net/docs)
