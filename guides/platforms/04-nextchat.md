# NextChat 接入深云 DeepClub：站根地址、`@Anthropic` 与 `@OpenAI` 模型绑定教程

**NextChat（ChatGPT-Next-Web）** 支持在一个界面里按模型名后缀切换不同提供商。对接 **[深云 DeepClub](https://deepclub.net)** 时最关键的两点是：**接口地址填写站根（不带 `/v1`）**，并在自定义模型名中用 **`@Anthropic`** 或 **`@OpenAI`** 明确绑定协议。

---

## 一、精确配置参数

在 NextChat 左下角 **设置（Settings）**：

1. 勾选 **自定义接口（Custom Endpoint）**。
2. 根据你想调用的模型家族选择 **模型服务商（Model Provider）** 并填入配置：

| 模型家族 | NextChat 服务商 | 接口地址（Endpoint） | 自定义模型名写法（Custom Models） | 实际请求路径 |
|---|---|---|---|---|
| **Claude 系列**<br>(`claude-sonnet-5-5`、`claude-opus-5-5`、`claude-fable-5-1/5-2`) | **Anthropic** | `https://deepclub.net` | `+claude-sonnet-5-5@Anthropic,+claude-opus-5-5@Anthropic` | `/v1/messages` |
| **DeepSeek / Gemini / GLM / Kimi / Qwen / GPT（Chat 兼容）** | **OpenAI** | `https://deepclub.net` | `+deepseek-v4.1-flash@OpenAI,+gemini-4-hydrogen@OpenAI,+glm-5.3@OpenAI,+gpt-6.1-sol@OpenAI` | `/v1/chat/completions` |

> **说明**：  
> 1. 前面的 `+` 表示在模型列表中添加/启用该模型；如果不想展示内置默认列表中的无关旧模型，可以在最前面加 `-all,`（例如 `-all,+claude-sonnet-5-5@Anthropic,+deepseek-v4.1-flash@OpenAI`）。  
> 2. **为什么不带 `/v1`？** NextChat 源码中按固定常量路径拼接 `/v1/chat/completions` 与 `/v1/messages`，填站根 `https://deepclub.net` 才不会重复拼接出 `/v1/v1/...`。  
> 3. **关于 GPT / Grok Responses**：在未核实你所用 NextChat 版本是否内置原生 Responses 支持前，GPT/Grok 通过 `@OpenAI` 接入属于 **Chat Completions 兼容路径**；如果你需要严格使用原生 `/v1/responses`，推荐使用 [Chatbox](./02-chatbox.md)、[Cherry Studio](https://deepclub.net/integrations/cherry-studio) 或直接用 [OpenAI SDK](./06-openai-agents-sdk.md)。

---

## 二、环境变量部署写法（自部署 Docker / Vercel）

如果你自己托管 NextChat 服务端给团队用，可以直接通过环境变量下发：

```bash
OPENAI_API_KEY=sk-你的DeepClub密钥
BASE_URL=https://deepclub.net
CUSTOM_MODELS=-all,+deepseek-v4.1-flash@OpenAI,+gemini-4-hydrogen@OpenAI,+glm-5.3@OpenAI,+gpt-6.1-sol@OpenAI
```

若主要给团队提供 Claude（通过 Anthropic 通道）：

```bash
ANTHROPIC_API_KEY=sk-你的DeepClub密钥
ANTHROPIC_URL=https://deepclub.net
CUSTOM_MODELS=-all,+claude-sonnet-5-5@Anthropic,+claude-opus-5-5@Anthropic
```

*(具体环境变量名以你当前部署的 NextChat Release README 为准；前端设置面板中配置后可直接在聊天框测试。)*

---

## 三、常见问题

- **提示 `404`**：检查接口地址末尾是否不小心多加了 `/v1` 或 `/`；NextChat 中应直接填 `https://deepclub.net`。
- **模型下拉选不到在售新模型**：必须在「自定义模型名」里写完整 ID（可在 [https://deepclub.net/pricing](https://deepclub.net/pricing) 核对），并用 `@Anthropic` 或 `@OpenAI` 显式绑定对应服务商。
- **更多教程**：[LobeChat 与 NextChat 自定义 API 教程](https://deepclub.net/integrations/lobechat-nextchat) ｜ [401/404 报错排查](https://deepclub.net/integrations/api-troubleshooting) ｜ [模型教程](https://deepclub.net/articles)
