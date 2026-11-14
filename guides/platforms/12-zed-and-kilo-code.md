# Zed Editor、Kilo Code 与 DeepSeek Harness 接入深云 DeepClub 配置指南

除了 Cursor、Cline 和 Roo Code 之外，很多开发者还会使用原生性能极高的 **Zed** 编辑器，以及 **Kilo Code**、**DeepSeek Harness** 等新兴 AI 编码工具。

本文对照各工具官方最新配置规范，提供对接 **[深云 DeepClub](https://deepclub.net)** 的精确配置模板。

---

## 一、Zed Editor（`settings.json` 原生双协议配置）

Zed 官网文档明确把自定义提供商分为 **`anthropic_compatible`** 与 **`openai_compatible`** 两类，而且支持通过 `capabilities.chat_completions: false` 让 `openai_compatible` 改走 **Responses API**。

### 1. API Key 注入方式（不要把密钥写进 `settings.json`）

- 方式 A：通过环境变量注入（例如在 shell profile 中设置 `ANTHROPIC_API_KEY` 或以 `OPENROUTER` / 自定义 provider 名称大写下划线加 `_API_KEY` 命名，如 `DEEPCLUB_API_KEY`）；
- 方式 B：在 Zed 中打开 **Settings → AI → LLM Providers → Add Provider**，把密钥安全保存到系统钥匙串（Keychain）。

### 2. 配置 `settings.json`（Claude 走 Messages，GPT 走 Responses，DeepSeek/GLM 走 Chat）

```json
{
  "language_models": {
    "anthropic_compatible": {
      "DeepClub Anthropic": {
        "api_url": "https://deepclub.net",
        "available_models": [
          {
            "name": "claude-sonnet-5-5",
            "display_name": "Claude Sonnet 5.5 (DeepClub)",
            "max_tokens": 200000,
            "max_output_tokens": 16000,
            "capabilities": {
              "tools": true,
              "images": true,
              "prompt_caching": true
            }
          },
          {
            "name": "claude-opus-5-5",
            "display_name": "Claude Opus 5.5 (DeepClub)",
            "max_tokens": 200000,
            "max_output_tokens": 16000,
            "capabilities": {
              "tools": true,
              "images": true,
              "prompt_caching": true
            }
          }
        ]
      }
    },
    "openai_compatible": {
      "deepclub": {
        "api_url": "https://deepclub.net/v1",
        "available_models": [
          {
            "name": "gpt-6.1-sol",
            "display_name": "GPT-6.1 Sol (Responses)",
            "max_tokens": 128000,
            "capabilities": {
              "chat_completions": false,
              "tools": true
            }
          },
          {
            "name": "deepseek-v4.1-flash",
            "display_name": "DeepSeek V4.1 Flash",
            "max_tokens": 128000,
            "capabilities": {
              "chat_completions": true,
              "tools": true
            }
          },
          {
            "name": "glm-5.3",
            "display_name": "GLM-5.3",
            "max_tokens": 128000,
            "capabilities": {
              "chat_completions": true,
              "tools": true
            }
          }
        ]
      }
    }
  }
}
```

> **注意**：`anthropic_compatible` 中的 `api_url` 填站点根地址 **`https://deepclub.net`**（Zed 会拼 `/v1/messages`）；`openai_compatible` 中的 `api_url` 填 **`https://deepclub.net/v1`**。对 `gpt-6.1-sol` 设置 `"chat_completions": false` 后，Zed 会自动请求 `/v1/responses`。

---

## 二、Kilo Code（VS Code / JetBrains 插件）

1. 点击 Kilo Code 面板右上角齿轮进入 **Settings → Providers → Custom provider**。
2. **Provider ID** 填 `deepclub`，显示名称填 `DeepClub`。
3. **Provider API** 选择 **OpenAI Compatible**。
4. **Base URL** 填 `https://deepclub.net/v1`，本地粘贴 DeepClub API Key。
5. 添加目录中的精确 Chat 兼容模型 ID（例如 `deepseek-v4.1-flash`、`glm-5.3`、`qwen3.8-flash` 或兼容接入的 `gpt-6.1-sol`），保存后在项目目录新建会话并测试文件工具。

---

## 三、DeepSeek Harness

1. 打开 **Settings → Models → Add Model Provider**，选择 **Custom Model API**（不要错选无关的内置供应商）。
2. **Provider ID** 填 `deepclub`，显示名称填 `DeepClub`。
3. 协议选择 **OpenAI Chat Completions (`openai-completions`)**，Base URL 填 **`https://deepclub.net/v1`**，填入个人 DeepClub API Key。
4. 拉取或手动添加精确模型 ID（如 `deepseek-v4.1-flash`、`deepseek-v4.1-flash-1017`、`glm-5.3`），同时核对模型的 `contextWindow` 与 `maxTokens` 参数，避免直接粘贴旧 YAML 覆盖完整配置。

---

## 四、扩展阅读

- [9 款主流客户端协议与 Base URL 速查表](../protocol-and-clients/client-setup-cheatsheet.md)
- [深云 DeepClub 定价与开放端点列表](https://deepclub.net/pricing)
- [深云 DeepClub API 文档](https://deepclub.net/docs)
