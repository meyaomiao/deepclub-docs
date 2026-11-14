# 零留存（ZDR）API 网关安全实践与子 Key 权限治理

把生产源码、内部合同、用户工单或科研数据接入第三方 AI API 时，安全团队最关心两件事：**数据会不会被留存用来训练模型？一旦某个端侧 Key 泄露，如何把爆炸半径控制在最小？**

本文基于 **[深云 DeepClub](https://deepclub.net)** 的零留存架构与密钥治理能力，梳理一套适合个人开发者与小团队的安全基线。

---

## 一、什么是 Zero Data Retention（ZDR）零留存？

根据 [深云 DeepClub 隐私政策](https://deepclub.net/privacy-policy)：

- **请求输入（Prompts）与模型输出（Completions）在网关不做持久化内容存储**；
- 控制台「使用日志」仅记录用于对账的元数据（模型 ID、时间戳、输入/输出/缓存 Token 数与扣费金额），不留存正文；
- 客户端若配合使用 OpenAI Responses API 的 **`store=False`**，也不依赖或触发服务端状态持久化。

> **安全边界提醒**：虽然网关实行 ZDR，但**仍建议在发送前做最基本的密钥与敏感凭据脱敏**（不要把数据库密码、私钥、AccessKey 直接写进 Prompt 发到外部或复制进公共示例）。

---

## 二、用“子 Key 隔离”替代“一把万能 Key 走天下”

很多开发者习惯只生成一个 `sk-...` Key，然后同时填在 Cursor、Claude Code、沉浸式翻译、GitHub Actions 和同事电脑里——一旦其中一个环境泄露，所有地方都要换，也根本看不出是哪个工具刷了费用。

**推荐的子 Key 治理策略**：

| 使用场景 | 建议做法 | 风险隔离收益 |
|---|---|---|
| **本地 IDE（Cursor / Claude Code / Cline）** | 单独建开发 Key，设适中额度上限 | 换电脑或某插件出异常时一键禁用，不影响线上服务 |
| **浏览器插件（沉浸式翻译）与笔记工具** | 建低额度 Key，只开放 `gpt-6-luna` / `gemini-4-hydrogen` / `deepseek-v4.1-flash` | 防止误全选网页翻译或无限并发跑光高价模型余额 |
| **CI/CD（GitHub Actions PR Review）** | 建专用 CI Key + 较低预算上限 + 定期（如 90 天）轮换 | 防止大体积 diff 或外部 PR 异常消耗高额配额 |
| **生产应用 / 团队内部工具** | 按业务环境（`staging` / `prod`）拆分独立 Key，开启模型白名单 | 出现异常调用可直接按 Key 维度统计与止损 |

---

## 三、开发期 4 条硬性安全规范

1. **禁止硬编码 API Key**：源码中一律读取 `os.environ["DEEPCLUB_API_KEY"]` 或 `process.env.DEEPCLUB_API_KEY`；在客户端（Zed、OpenCode、Claude Code）尽量使用环境变量或系统钥匙串，不要把 Key 提交进 Git。
2. **小心客户端默认遥测 / Tracing**：
   - 使用 **OpenAI Agents SDK** 时，默认会把 Trace 发给 OpenAI 官方，请显式调用 `set_tracing_disabled(True)`；
   - 使用 **LangChain** 时，如无需上传 LangSmith，确保未误开 `LANGCHAIN_TRACING_V2=true`；
   - 这些第三方遥测通道不属于 DeepClub 网关链路，需按各自数据策略独立控制。
3. **GitHub Actions 防 Fork 泄露**：AI Review Workflow 务必加上 `if: github.event.pull_request.head.repo.full_name == github.repository`，并对 `git diff` 截断字符上限与忽略 `*.lock` / 构建产物。
4. **给客户端写 Agent 提示词时的安全规则**：涉及自动化修改 `~/.codex/config.toml` 或 `~/.claude/settings.json` 时，要求代理先备份原文件、仅在本地替换占位符、绝不打印环境变量完整密钥值。

隐私条款与安全说明：[https://deepclub.net/privacy-policy](https://deepclub.net/privacy-policy)
