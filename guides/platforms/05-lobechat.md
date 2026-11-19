# LobeChat 接入深云 DeepClub：多协议模型分组、Base URL 与流式输出验证

**LobeChat** 支持在「语言模型（Language Model）」设置中分别配置 OpenAI 与 Anthropic 的代理地址（Base URL / Proxy），也可以通过自定义聚合或环境变量一键挂载多个在售模型。

本文整理对接 **[深云 DeepClub](https://deepclub.net)** 的推荐配置方案，按协议把 Claude 与其他大模型分开挂载，避免路径和接口不匹配。

---

## 一、按协议分开填（最稳做法）

LobeChat 对 **Anthropic** 与 **OpenAI** 使用不同的 SDK 路径规则：

| LobeChat 模型服务商入口 | API Proxy / Base URL 填法 | API Key | 建议添加的模型 ID（示例） |
|---|---|---|---|
| **Anthropic (Claude)** | `https://deepclub.net`（或按你所用版本提示确认是否带 `/v1`；以实际版本面板说明为准，报错 404 时互换测试） | DeepClub `sk-...` | `claude-sonnet-5-5`、`claude-opus-5-5`、`claude-fable-5-1`、`claude-fable-5-2` |
| **OpenAI / OpenAI Compatible** | `https://deepclub.net/v1` | DeepClub `sk-...` | `deepseek-v4.1-flash`、`gemini-4-hydrogen`、`gemini-4-argon`、`glm-5.3`、`kimi-k3`、`qwen3.8-flash`、`gpt-6.1-sol`、`gpt-6-luna`、`grok-4.7` |

> **如何确认当前版本 LobeChat 是否需要 Responses？**  
> 在 DeepClub 目录中，`gpt-6.1-sol`、`gpt-6-astra`、`gpt-6-luna`、`gpt-6.1-terra`、`gpt-5.6-sol` 与 `grok-4.7` **同时开放了 `/v1/responses`（首选）与 `/v1/chat/completions`（兼容）**。如果 LobeChat 当前服务商面板默认发 Chat Completions，这些模型也能正常对话和流式输出；若新版 LobeChat 的 OpenAI 服务商开启了 Responses API 开关，则可直接走原生 `/v1/responses`。

---

## 二、图形界面操作步骤

1. 在 [深云 DeepClub](https://deepclub.net) 控制台 **API 密钥** 中创建一个 `sk-...` 密钥。
2. 打开 LobeChat → 左上角头像 / 左下角设置 → **应用设置 → 语言模型**。
3. **配置 OpenAI 兼容入口**：
   - 打开 **OpenAI**（或自定义的 OpenAI-compatible 服务商）；
   - **API Key**：填入 DeepClub 密钥；
   - **API 代理地址（Base URL）**：`https://deepclub.net/v1`；
   - 在自定义模型输入框填入完整模型 ID（用英文逗号或按面板提示分隔）：  
     `deepseek-v4.1-flash,gemini-4-hydrogen,gemini-4-argon,glm-5.3,glm-5.3-flash,kimi-k3,qwen3.8-flash,gpt-6.1-sol,gpt-6-luna,gpt-6-astra,grok-4.7`
   - 点击面板自带的 **检查（Check）** 按钮时，把检查用模型改成你刚填入的一个在售精确 ID（如 `gpt-6-luna` 或 `deepseek-v4.1-flash`），不要用未上架的旧默认测试名。
4. **配置 Anthropic 入口（走原生 `/v1/messages`）**：
   - 打开 **Anthropic** 服务商；
   - 填入同一把 DeepClub API Key；
   - **API 代理地址**：先填 `https://deepclub.net`（若提示 404 可检查 LobeChat 当前版本是否要求加 `/v1`）；
   - 添加 `claude-sonnet-5-5,claude-opus-5-5,claude-fable-5-1,claude-fable-5-2` 并用其中一个精确 ID 测试连通性。

---

## 三、常见问题排查

1. **连通性检查报“模型不存在”**：LobeChat 的“检查”按钮默认可能测试官方写死的模型 ID，请先在检查模型选择框里切换为你刚刚添加的精确模型 ID（例如 `gpt-6-luna`），再点测试。
2. **返回 404**：检查是 `/v1` 多拼还是少了 `/v1`。可在 [深云 DeepClub 控制台使用日志](https://deepclub.net) 对照实际收到的请求路径；未进日志说明请求在路由前被路径 404 拦截。
3. **生图与视频模型不要放进文本对话列表**：`gpt-image-2`、`gemini-3-pro-image`、`nano-banana-pro`、`Seedance-2.0-Fast` 等走独立的 `/v1/images/*` 和 `/v1/videos` 端点，不要加到聊天文本列表里。如需生图/视频模板，见 [图像与视频 API 实战](../media/images-and-videos-pipeline.md)，或在官网查看 [LobeChat 与 NextChat 配置](https://deepclub.net/integrations/lobechat-nextchat) 与 [API 报错排查](https://deepclub.net/integrations/api-troubleshooting)。
