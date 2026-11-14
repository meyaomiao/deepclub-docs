# Codex CLI、Codex++ 与 OpenCode 接入深云 DeepClub 配置模板

命令行与桌面端 AI 编码代理（**Codex CLI**、**Codex++**、**OpenCode**）是高频消耗 Token 的主力工具。接入 **[深云 DeepClub](https://deepclub.net)** 后，你可以按量计费使用 `gpt-6.1-sol`、`gpt-6-astra`、`deepseek-v4.1-flash`、`glm-5.3` 等模型，无需绑定海外信用卡或购买固定月费。

---

## 一、Codex CLI（原生 Responses API）

Codex CLI 原生使用 **OpenAI Responses API**。先备份 `~/.codex/config.toml`（Windows 为 `%USERPROFILE%\.codex\config.toml`），再合并以下配置（注意顶层的 `model_provider` 和 `model` 必须放在 `[model_providers.deepclub]` 段之前）：

```toml
model_provider = "deepclub"
model = "gpt-6.1-sol"

[model_providers.deepclub]
name = "DeepClub"
base_url = "https://deepclub.net/v1"
wire_api = "responses"
env_key = "DEEPCLUB_API_KEY"
```

在终端设置环境变量后启动 Codex：

```bash
export DEEPCLUB_API_KEY="sk-你的DeepClub密钥"
codex
```

**注意事项**：
- `model` 必须填写支持 `/v1/responses` 的在售型号，例如 `gpt-6.1-sol`、`gpt-6-astra`、`gpt-6.1-terra`、`gpt-5.6-sol`、`grok-4.7`、`glm-5.3`；不要填仅开放 `/v1/chat/completions` 的模型。
- 如果同时保留了原配置中的其他推理参数（如 `model_reasoning_effort`），合并时保留已有配置并做小范围测试。

---

## 二、Codex++（桌面端 Provider 管理器）

如果你在 macOS / Windows 上使用官方 Codex 桌面应用 + 第三方管理端 **Codex++**：

1. 先分别安装官方 Codex 桌面端与对应系统架构的 Codex++（确认 Codex++ 能识别到本地 Codex）。
2. 在 Codex++ 中打开 **供应商配置（Provider）** → 启用切换 → 新增 `DeepClub`。
3. 选择 **纯 API 模式**：
   - **API 协议**：**Responses API**（不要直接套用本地 Chat 代理适配器，除非你明确需要）
   - **Base URL**：`https://deepclub.net/v1`
   - **API Key**：粘贴 DeepClub `sk-...` 密钥
   - **模型 ID**：从目录添加精确 ID，如 `gpt-6.1-sol` 或 `gpt-6-astra`
4. 保存并激活 DeepClub，通过 Codex++ 重启 Codex 后，先执行一个只读检查任务（例如列出 README 要点）确认连通和控制台消费明细。

> 注意：如果使用 Codex++ 托管 `~/.codex/config.toml`，请统一从 Codex++ 界面修改，避免同时手动覆盖同一份配置文件。

---

## 三、OpenCode（`opencode.json` + `/connect`）

OpenCode 支持通过 `@ai-sdk/openai-compatible` 挂载自定义供应商，把 Key 存在本地凭据库而不是明文写进项目目录：

1. 在项目根目录新建或合并 `opencode.json`：

```json
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "deepclub": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "DeepClub",
      "options": {
        "baseURL": "https://deepclub.net/v1"
      },
      "models": {
        "deepseek-v4.1-flash": { "name": "deepseek-v4.1-flash" },
        "glm-5.3": { "name": "glm-5.3" },
        "gpt-6.1-sol": { "name": "gpt-6.1-sol" }
      }
    }
  }
}
```

2. 启动 `opencode`，输入 `/connect` → 选择 **Other** → Provider ID 填 **`deepclub`**（必须与 `opencode.json` 中的 key 完全一致）→ 粘贴 DeepClub API Key。
3. 运行 `/models` 选择 `DeepClub` 下的精确模型 ID，新建会话验证文件读取与编辑工具是否正常。

---

## 四、模型选型与安全建议

- **高难度重构 / 算法架构**：`gpt-6-astra`、`claude-opus-5-5`（Claude 建议使用 [Claude Code](../protocol-and-clients/claude-code-messages-vs-chat.md) 走原生 Messages）
- **日常编码主力**：`gpt-6.1-sol`（Responses）、`deepseek-v4.1-flash`、`glm-5.3`
- **子 Key 隔离**：在 [深云 DeepClub](https://deepclub.net) 控制台为每台开发机或每个 CLI 单独创建带额度上限的 Key，随时可在控制台禁用或重置。

更多客户端教程：[https://deepclub.net/docs](https://deepclub.net/docs) ｜ 实时价格：[https://deepclub.net/pricing](https://deepclub.net/pricing)
