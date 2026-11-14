# Dify 接入深云 DeepClub 完整指南（OpenAI 插件 `1.1.0+`、Anthropic 插件 `0.3.41+` 与兼容插件配置）

在 **Dify** 中接入 **[深云 DeepClub](https://deepclub.net)**，最常见的报错集中在两点：把**Dify 主程序版本**当成了**模型供应商插件版本**，或者混淆了**站根地址**与 **`/v1` 地址**。

本文按照 Dify 官方插件的实际 schema 拆解三种接入方式，适用于当前在售的 Claude、GPT-6、Gemini、DeepSeek、GLM、Kimi 与 Qwen 文本模型。实时开放端点与价格见 [深云 DeepClub 定价表](https://deepclub.net/pricing)。

---

## 一、先选对插件与地址（别混用）

| 场景 / 模型家族 | 推荐 Dify 插件与版本 | Base URL / Endpoint 怎么填 | 实际发送协议 |
|---|---|---|---|
| **GPT 文本 & Grok** (`gpt-6.1-sol`、`gpt-6-astra`、`gpt-6-luna`、`grok-4.7`) | 官方 **OpenAI** 插件 (`1.1.0+`) | `https://deepclub.net/v1` | `POST /v1/responses`（设 `API Protocol=Responses API`）或 Chat |
| **Claude 系列** (`claude-sonnet-5-5`、`claude-opus-5-5`、`claude-fable-5-1/5-2`) | 官方 **Anthropic** 插件 (`0.3.41+`) | `https://deepclub.net`（**不带 `/v1`**） | `POST /v1/messages` |
| **DeepSeek / Gemini / GLM / Kimi / Qwen** | **OpenAI-API-compatible** (`0.0.83+`) 或官方 **OpenAI** 插件 | `https://deepclub.net/v1` | `POST /v1/chat/completions` |

> **关键提醒**：Dify 自带插件市场中的插件版本号与 Dify 主程序版本号是分开的。若界面里没有出现 `API Protocol` 或自定义模型表单，请先在 **插件市场** 升级对应的模型供应商插件。

---

## 二、方案 A：官方 OpenAI 插件配置 Responses API（推荐用于 GPT-6 / Grok）

1. 在 Dify 右上角进入 **设置 → 模型供应商**（或 **插件** 页面），确认官方 **OpenAI** 插件版本为 `1.1.0` 或更高。
2. 选择 **添加自定义 LLM 模型**（Custom Model）：
   - **Model Name**：填目录中的精确模型 ID，例如 `gpt-6.1-sol` 或 `gpt-6-astra`
   - **API Key**：粘贴 DeepClub 控制台创建的 `sk-...` 密钥
   - **API Base URL**：`https://deepclub.net/v1`
   - **API Protocol**：选择 **Responses API**（对应 schema 中的 `api_protocol=responses`）
3. 保存后在 **工作室 → Workflow / Chatflow → LLM 节点** 中选择刚添加的模型，发一条短消息测试流式输出与工具调用。

---

## 三、方案 B：官方 Anthropic 插件配置原生 Messages（推荐用于 Claude）

1. 在 Dify **模型供应商** 中安装或更新官方 **Anthropic** 插件（核对版本 `0.3.41+`）。
2. 填写凭据：
   - **API Key**：你的 DeepClub `sk-...` 密钥
   - **API Base URL**：填站点根地址 **`https://deepclub.net`**（**不要**带 `/v1`，Anthropic SDK 内部会自动拼 `/v1/messages`）
3. 若需要在同一套工作流里使用精准的在售型号 ID（如 `claude-sonnet-5-5`、`claude-opus-5-5`、`claude-fable-5-2`），请以自定义模型名称方式加入并发送一条验证请求。
4. 注意：`claude-opus-5-5`、`claude-sonnet-5-5`、`claude-fable-5-2` 在 `/v1/messages` 上不接受非 1 的 `temperature` / `top_p` / `top_k`，在 Dify LLM 节点参数面板中请不要强行开启冲突的采样滑块。

---

## 四、方案 C：`OpenAI-API-compatible` 插件接入 DeepSeek / Gemini / GLM / Kimi / Qwen

1. 在 Dify **模型供应商** 中添加 **OpenAI-API-compatible**：
   - **Model Type**：`LLM`
   - **Model Name**：填精确 ID，例如 `deepseek-v4.1-flash`、`gemini-4-hydrogen`、`glm-5.3`、`kimi-k3`、`qwen3.8-flash`
   - **Completion mode**：`Chat`
   - **API endpoint URL**：`https://deepclub.net/v1`
   - **API Key**：DeepClub `sk-...` 密钥
2. 关于插件 `0.0.83` 中的 **API Type**：虽然新版本开放了可选 `Responses API`，但部分版本的凭据校验请求仍可能发送至 `/v1/chat/completions`。因此：
   - 仅开放 Chat 的模型（DeepSeek / Gemini / Kimi / Qwen）选 Chat Completions 最稳；
   - 若要用 `gpt-6.1-sol` 且需要原生 Responses，优先使用 **方案 A（官方 OpenAI 插件 1.1.0+）**。

---

## 五、工作流低成本模型搭配建议

在 Dify Agent / Workflow 中，不同节点对模型能力与预算的要求差异很大，推荐按节点拆分：

| Dify 工作流节点 | 推荐模型 | 说明与计费查看 |
|---|---|---|
| 意图识别 / 路由分类 / 文本清洗 | `gpt-6-luna`、`glm-5.3-flash`、`gemini-4-hydrogen` | 低单价、低延迟，适合高频入口节点 |
| 知识库 RAG 问答 / 多文档总结 | `deepseek-v4.1-flash`、`gpt-6.1-sol`、`gemini-4-argon` | 性价比与长文本综合能力均衡 |
| 复杂代码 / 合同审查 / 多步推理 Agent | `claude-sonnet-5-5`、`claude-opus-5-5`、`gpt-6-astra`、`glm-5.3` | 逻辑严谨、工具调用准确率高 |

实时单价、缓存读/写费用及 ZDR 零留存说明见：
- 实时定价表：[https://deepclub.net/pricing](https://deepclub.net/pricing)
- 完整 API 与客户指南：[https://deepclub.net/docs](https://deepclub.net/docs) ｜ [https://deepclub.net/articles](https://deepclub.net/articles)
