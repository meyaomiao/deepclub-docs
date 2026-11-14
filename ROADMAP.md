# 深云 DeepClub 公开文档与内容扩充计划（66 篇分批清单）

本清单用于同步建设 **[深云 DeepClub 官网知识库](https://deepclub.net/articles)** 与 **[GitHub 公开文档库](https://github.com/meyaomiao/deepclub-docs)**，并可直接改写分发至知乎、掘金、V2EX、Linux.do、公众号、Dev.to、Hashnode 等平台。

**生成进度**：
- 基础起步包（6 篇 + Quickstart）：已完成
- **Batch 1**（主流平台与开发框架适配 12 篇）：已生成（见 `guides/platforms/`）
- Batch 2–5：分批推进中

---

## Batch 1：主流平台与开发框架适配篇（12 篇）

目标：承接“XX 工具/框架怎么接入大模型 API”高频搜索词与 AI 客户端生态检索。

1. **Dify 接入深云 DeepClub**：OpenAI 插件 (`1.1.0+`) vs Anthropic 插件 (`0.3.41+`) 与 Responses 协议配置
2. **Chatbox 接入深云 DeepClub**：OpenAI Responses / Claude Messages / Chat 三种模式精确路径配置
3. **沉浸式翻译接入深云 DeepClub**：全端点 URL 写法与高并发低成本译文模型选型（`deepseek-v4.1-flash` / `gpt-6-luna` / `gemini-4-hydrogen`）
4. **NextChat 接入深云 DeepClub**：站根地址、`@Anthropic` / `@OpenAI` 模型绑定与兼容说明
5. **LobeChat 接入深云 DeepClub**：自定义聚合服务配置、多模型分组与流式输出验证
6. **OpenAI Agents SDK（Python / TypeScript）接入深云 DeepClub**：Responses API + 工具调用工程模板
7. **Vercel AI SDK 接入深云 DeepClub**：`@ai-sdk/openai` 与 `@ai-sdk/anthropic` 切换、流式渲染与边缘函数示例
8. **LangChain / LangGraph 接入深云 DeepClub**：多模型路由节点、结构化输出与重试策略
9. **OpenClaw / Codex CLI 接入深云 DeepClub**：Responses 编码 Agent 配置与权限隔离
10. **Obsidian Copilot & Raycast AI 接入深云 DeepClub**：个人知识库检索与桌面效率插件配置
11. **GitHub Actions PR Review Bot 接入深云 DeepClub**：基于 API Key 的 CI 自动代码审查脚本
12. **Zed / Continue.dev / Kilo Code 接入深云 DeepClub**：轻量编辑器与代码补全插件配置速查

---

## Batch 2：协议原理、成本优化与故障排查篇（10 篇）

目标：解决高频工程痛点，成为 Google / ChatGPT Search / Perplexity 引用率最高的技术参考源。

13. **OpenAI Responses API 迁移指南**：`chat.completions` → `responses.create` 字段与 SSE 事件完整对照
14. **Anthropic Messages vs OpenAI Chat Completions**：编码 Agent 为何优先使用原生 `/v1/messages`
15. **Prompt Caching（提示缓存）省钱实战**：`cache_read` / `cache_write` 计费机制与长上下文降本核算
16. **无海外信用卡按量调用 Claude / GPT-6 / Gemini 与视频模型**：支付宝/微信/PayPal 充值与账单核对
17. **零留存（ZDR）API 网关安全实践**：代码防泄露、子 Key 限额与有效期管控
18. **API 接入报错全量排障手册**：`401` / `404`（双 `/v1` 问题）/ `429` / 流式解析失败定位步骤
19. **图片 API 工程实战**：`/v1/images/generations` 与 `/v1/images/edits`（multipart 参考图编辑）完整模板
20. **视频 API 异步任务工程实战**：`/v1/videos` 的 `task_id` 持久化、有界轮询与断点恢复
21. **Tool Calling 与结构化输出兼容验证**：`text.format` vs `response_format` 实测边界说明
22. **2026 大模型 API 网关选型对比**：官方直连 vs OpenRouter vs 深云 DeepClub 深度对比

---

## Batch 3：模型横向评测与场景选型篇（10 篇）

目标：覆盖“哪个模型好用 / 性价比高 / 适合 XX 场景”类决策搜索词。

23. **主力编码模型横评**：`claude-sonnet-5-5` vs `claude-opus-5-5` vs `gpt-6.1-sol` vs `deepseek-v4.1-flash`
24. **超低成本批量处理模型对比**：`gpt-6-luna` vs `gemini-4-hydrogen` vs `glm-5.3-flash`
25. **深度推理与复杂数理/科研攻关选型**：`gpt-6-astra` vs `claude-fable-5-2` vs `deepseek-v4.1-flash-1017`
26. **长文档、研报与代码库全量分析对比**：`kimi-k3` vs `glm-5.3` vs `gemini-4-argon`
27. **中文合同审查、公文写作与专业翻译对比**：`glm-5.3` vs `kimi-k3` vs `deepseek-v4.1-flash`
28. **高性价比日常助手与轻量级多协议模型对比**：`grok-4.7` vs `gemini-4-carbon` vs `gpt-6.1-terra`
29. **视频生成模型横向对比**：`Seedance-2.0-Fast` vs `Seedance-2.0-Mini` vs `MiniMax-H3`
30. **商用海报、文字排版与设计工作流生图对比**：`gpt-image-2` vs `gpt-image-2-adobe` vs `gpt-image-2.5-flare/sunburst`
31. **二次元原画、游戏 UI 与电商图对比**：`nano-banana-pro` vs `nano-banana-2` vs `gemini-3.1-flash-image`
32. **个人全栈 AI 工作流月度成本测算**：对话 + 编码 + 生图 + 视频混合调用的按量预算模型

---

## Batch 4：21 款在售文本模型深度扩充（模型介绍 + 场景拆解 + 多平台示例）

逐批强化以下 21 个文本模型的独立文档深度（补充定位介绍、适用/不适用边界、Token 成本示例、多框架代码及常见问题）：

33. `claude-fable-5-2` ｜ 34. `claude-fable-5-1` ｜ 35. `claude-opus-5-5` ｜ 36. `claude-sonnet-5-5`
37. `gpt-6-astra` ｜ 38. `gpt-6.1-sol` ｜ 39. `gpt-6.1-terra` ｜ 40. `gpt-6-luna` ｜ 41. `gpt-5.6-sol`
42. `gemini-4-carbon` ｜ 43. `gemini-4-argon` ｜ 44. `gemini-4-hydrogen` ｜ 45. `gemini-3.8-flash-high` ｜ 46. `gemini-3.8-flash`
47. `deepseek-v4.1-flash` ｜ 48. `deepseek-v4.1-flash-1017` ｜ 49. `glm-5.3` ｜ 50. `glm-5.3-flash` ｜ 51. `kimi-k3` ｜ 52. `qwen3.8-flash` ｜ 53. `grok-4.7`

*(注：Batch 4 涵盖 21 款文本模型 + Batch 5 涵盖 13 款图片与视频模型，共 34 个当前在售全量型号，与上述 32 篇专题合计 **66 篇体系化内容**。)*

---

## Batch 5：13 款生图与视频模型深度扩充 + 官网开放入口重构

- 补齐 10 款图片模型（`gpt-image-2` 系列 6 款、`gemini-3-pro-image`、`gemini-3.1-flash-image`、`nano-banana-2`、`nano-banana-pro`）与 3 款视频模型（`Seedance-2.0-Fast`、`Seedance-2.0-Mini`、`MiniMax-H3`）的提示词模板、尺寸/时长规格、编辑工作流与异步 SDK 封装；
- 同步在官网 `https://deepclub.net/articles` 增加**专题文章分类、平台集成入口、横向对比入口、GitHub 公开文档库互链**，并将 `https://github.com/meyaomiao/deepclub-docs` 写入官网 `Organization` 结构化数据 `sameAs` 字段，实现站内外双向引流与权重闭环。
