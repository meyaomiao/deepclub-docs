# Obsidian Copilot & 桌面效率插件接入深云 DeepClub 配置参考

知识库插件（如 **Obsidian Copilot**、**Obsidian Smart Connections**）以及桌面启动器/剪贴板助手（支持自定义 OpenAI 兼容端点的效率工具）通常提供通用的 **Custom OpenAI** 配置项：填对 **Base URL**、**API Key** 和 **精确 Model ID** 即可按量使用。

---

## 一、通用 Custom OpenAI 接入参数（三件套）

在所有支持 OpenAI Compatible（Chat Completions）的桌面或笔记插件中，核心参数统一如下：

| 配置项 | 填写内容 | 说明 |
|---|---|---|
| **Base URL / Endpoint** | `https://deepclub.net/v1` | 如果插件不会自动拼 `/chat/completions`，则填完整地址 `https://deepclub.net/v1/chat/completions` |
| **API Key** | `sk-你的DeepClub密钥` | 建议为 Obsidian / 桌面插件单独创建一个子 Key 并设置月度上限 |
| **Model ID** | 目录中完整精确 ID | 例如：`deepseek-v4.1-flash`、`gemini-4-hydrogen`、`gpt-6-luna`、`gpt-6.1-sol`、`glm-5.3` |

---

## 二、Obsidian Copilot 配置步骤（Chat / Custom Model）

1. 在 Obsidian 设置 → 第三方插件中安装并启用 **Copilot**。
2. 打开 **Copilot 插件设置 → Model（模型管理）→ Add Custom Model（添加自定义模型）**：
   - **Model Name**：填精确 ID，例如 `gemini-4-hydrogen` 或 `deepseek-v4.1-flash`
   - **Provider**：选择 `3rd party (OpenAI-format)` / `OpenAI Compatible`
   - **Base URL**：`https://deepclub.net/v1`
   - **API Key**：粘贴 DeepClub `sk-...` 密钥
3. 保存后回到 Copilot 侧边栏，在模型下拉框中选中刚才添加的模型，选中一段笔记测试总结或问答。

> **关于笔记库隐私（ZDR）**：  
> 个人笔记和工作周报经常包含敏感项目资料。深云 DeepClub 全平台实行 **Zero Data Retention (ZDR)** 零留存政策，请求的笔记上下文与输出内容不做持久化存储、不用于模型训练，详见 [https://deepclub.net/privacy-policy](https://deepclub.net/privacy-policy)。

---

## 三、桌面知识问答与翻译快捷插件选型建议

笔记润色、长文摘要、剪贴板翻译属于**高频、中等长度输入、低延迟**场景，建议优先挑选性价比高的模型：

| 使用场景 | 推荐模型 ID | 协议路径（兼容 Chat） | 参考单价（实时见 [pricing](https://deepclub.net/pricing)） |
|---|---|---|---|
| **超低成本日常润色 / 周报整理 / 摘录** | `gpt-6-luna` | `https://deepclub.net/v1` | 输入 ¥0.12 / 输出 ¥0.6 每百万 Token |
| **快速外文笔记翻译 / 结构化大纲** | `gemini-4-hydrogen` | `https://deepclub.net/v1` | 输入 ¥0.6 / 输出 ¥3 每百万 Token |
| **技术笔记整理 / 代码片段解读** | `deepseek-v4.1-flash` | `https://deepclub.net/v1` | 输入 ¥1.1 / 输出 ¥4.4 每百万 Token |
| **长篇论文 / 年度复盘 / 万字笔记深度综合** | `gemini-4-argon`、`glm-5.3`、`gpt-6.1-sol` | `https://deepclub.net/v1` | 详见定价目录 |

---

## 四、常见问题排查

1. **填了 `https://deepclub.net/v1` 仍然报 404**：检查该插件文档是否要求填写完整端点（即需要带 `/chat/completions` 后缀）。若只填 `/v1` 报 404，可尝试改为 `https://deepclub.net/v1/chat/completions`。
2. **模型不支持向量（Embeddings）怎么办？**：当前 DeepClub 在售目录集中在文本对话、生图和视频生成模型（实时列表见 [https://deepclub.net/pricing](https://deepclub.net/pricing)），如果 Obsidian 插件需要本地 RAG 建库向量化，Embedding 模型建议使用本地 Ollama（如 `bge-m3` / `nomic-embed-text`）配合 DeepClub 的对话大模型做最终推理与问答。
