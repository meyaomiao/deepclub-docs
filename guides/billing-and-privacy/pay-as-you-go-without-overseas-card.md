# 没有海外信用卡怎么按量调用 Claude、GPT-6、Gemini 与视频生成模型？

对于国内个人开发者、独立产品人和小团队来说，使用海外大模型 API 最大的拦路虎往往不是写代码，而是两件事：
1. **绑卡与风控**：很多官方平台要求海外 Visa/Mastercard 或当地手机号，动辄触发账单地址风控或封号；
2. **月度订阅浪费**：官网 Plus / Pro 订阅一个月动辄 20~200 美元，用不完也退不了，而且在 Cursor / Cline / 自动化脚本里超额还是会被限频。

如果你只是想**按实际用了多少付多少钱**，使用支持国内常用支付方式的多模型聚合 API 网关是最省心的解法。下面以 **[深云 DeepClub](https://deepclub.net)** 为例，梳理从注册到首次调用的完整路径。

---

## 一、核心优势：一个账号 + 支付宝/微信/PayPal/信用卡 + 按量计费

1. **无需海外信用卡**：控制台支持支付宝、微信支付、PayPal 以及 Visa/Mastercard 等充值方式（以充值页当前显示为准），充多少用多少。
2. **无月度订阅捆绑**：文本按每百万 Token 计价，生图（`¥0.16` 起/张）和视频生成（`¥1.5` 起/个）按成功次数计价，没有最低消费。
3. **一个 Key 调齐主力模型**：不用分别注册 Anthropic、OpenAI、Google、xAI、DeepSeek、智谱、Moonshot 账号，在后台创建一个 `sk-` 密钥就能按各自最优协议调用。
4. **零留存（ZDR）**：请求内容与回复不做持久化存储，不用于模型训练。

---

## 二、常见模型真实参考价格一览（实时以官网定价页为准）

- **轻量高频对话**：`gpt-6-luna`（输入 ¥0.12 / 输出 ¥0.6 每百万 Token）、`gemini-4-hydrogen`（输入 ¥0.6 / 输出 ¥3）、`glm-5.3-flash`（输入 ¥0.44 / 输出 ¥1.54）
- **日常编码与通用主力**：`claude-sonnet-5-5`（输入 ¥1 / 输出 ¥5）、`gpt-6.1-sol`（输入 ¥0.7 / 输出 ¥3.5）、`deepseek-v4.1-flash-1017`（输入 ¥1 / 输出 ¥4）
- **复杂推理与架构重构**：`claude-opus-5-5`（输入 ¥2 / 输出 ¥10）、`gpt-6-astra`（输入 ¥3.5 / 输出 ¥17.5）、`kimi-k3`（输入 ¥12 / 输出 ¥60）
- **生图与视频**：`gpt-image-2`（¥0.16 起/张）、`gemini-3-pro-image`（¥0.3 起/张）、`Seedance-2.0-Mini`（¥1.5 起/个）、`MiniMax-H3`（¥2 起/个）

完整列表与缓存读写价格见：[https://deepclub.net/pricing](https://deepclub.net/pricing)。

---

## 三、3 步完成首次接入（不写代码也能用）

1. **创建密钥**：登录 [https://deepclub.net](https://deepclub.net)，在「API 令牌」页面生成一个 Key（可设置单个 Key 的额度上限和有效期，更安全）。
2. **选对客户端配置**（如果不想写代码，直接用现成桌面/IDE 客户端）：
   - **Claude Code**：设置 `ANTHROPIC_BASE_URL=https://deepclub.net` 与 `ANTHROPIC_API_KEY=你的Key`；
   - **Cursor / Cline / Roo Code / Cherry Studio**：在 OpenAI Compatible（或对应协议设置）里把 Base URL 设为 `https://deepclub.net/v1`，填入 Key 和模型 ID（如 `claude-sonnet-5-5`、`gpt-6.1-sol`、`deepseek-v4.1-flash`）。
3. **控制台核对账单**：每次请求消耗多少输入/输出 Token（含是否命中提示缓存）、花了多少钱，都可以在使用日志里逐笔查看。

更多客户端的图文级配置说明：[https://deepclub.net/articles](https://deepclub.net/articles) 与 [https://deepclub.net/docs](https://deepclub.net/docs)。
