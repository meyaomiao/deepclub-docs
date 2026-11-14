# GitHub Actions 自动 PR Code Review 机器人（接入深云 DeepClub）

在 CI 流水线里挂一个轻量 PR 审查脚本，每次提交 Pull Request 时自动对核心 diff 做安全漏洞、边界条件和并发隐患检查，能帮团队挡掉不少低级 Bug。

借助 **[深云 DeepClub](https://deepclub.net)** 的按量计费和 ZDR 零留存政策，你可以直接用 GitHub Repository Secret 挂载一个 `DEEPCLUB_API_KEY`，无需为每个仓库维护单独的海外供应商账单。

---

## 一、推荐模型选择

| 仓库规模 / 预算偏好 | 推荐模型 | 协议端点 | 特点 |
|---|---|---|---|
| **高频日常 PR（高性价比主力）** | `gpt-6.1-sol` | `POST /v1/responses` | 响应快、结构化审查结论清晰 |
| **复杂核心后端 / 安全审查** | `claude-sonnet-5-5` | `POST /v1/messages` | 代码上下文理解深、边界检查细致 |
| **开源大库低成本预筛** | `deepseek-v4.1-flash` | `POST /v1/chat/completions` | 极低单价适合大批量 commit 初步扫描 |

---

## 二、准备 GitHub Secret

1. 在 [深云 DeepClub](https://deepclub.net) 控制台 **API 密钥** 新建一个**CI 专用子 Key**，并设置较低的额度上限与过期时间（例如 90 天轮换一次）。
2. 在 GitHub 仓库进入 **Settings → Secrets and variables → Actions → New repository secret**：
   - Name: `DEEPCLUB_API_KEY`
   - Secret: 粘贴刚才创建的 `sk-...` 密钥

---

## 三、Workflow 配置（`.github/workflows/ai-pr-review.yml`）

下面的 workflow 在非 Fork PR 触发时提取变更 diff（截断过长文本以控成本），通过 OpenAI Python SDK 调用 `https://deepclub.net/v1` 的 Responses API（`store=False`）并自动把审查意见写到 PR 评论里：

```yaml
name: AI PR Review (DeepClub)

on:
  pull_request:
    types: [opened, synchronize]

permissions:
  contents: read
  pull-requests: write

jobs:
  review:
    # 避免外部 Fork PR 无法/不应读取仓库 Secrets
    if: github.event.pull_request.head.repo.full_name == github.repository
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.12'

      - name: Install OpenAI SDK
        run: pip install openai

      - name: Run Review via DeepClub Responses API
        env:
          DEEPCLUB_API_KEY: ${{ secrets.DEEPCLUB_API_KEY }}
          BASE_SHA: ${{ github.event.pull_request.base.sha }}
          HEAD_SHA: ${{ github.event.pull_request.head.sha }}
        run: |
          git diff "$BASE_SHA" "$HEAD_SHA" -- ':!*.lock' ':!dist/*' > pr.diff
          python3 - <<'PY'
          import os
          from openai import OpenAI

          with open("pr.diff", "r", encoding="utf-8", errors="ignore") as f:
              diff_text = f.read()[:24000]

          if not diff_text.strip():
              with open("review.md", "w") as out:
                  out.write("No reviewable source diff found.")
              raise SystemExit(0)

          client = OpenAI(
              api_key=os.environ["DEEPCLUB_API_KEY"],
              base_url="https://deepclub.net/v1",
          )

          resp = client.responses.create(
              model="gpt-6.1-sol",
              store=False,
              input=(
                  "你是资深代码审查工程师。仅针对真正存在的 Bug、安全漏洞、并发竞态、资源泄漏或计费边界问题输出中文要点；"
                  "如果未发现高风险问题，请简短说明 LGTM。\n\n```diff\n"
                  + diff_text
                  + "\n```"
              ),
          )
          with open("review.md", "w", encoding="utf-8") as out:
              out.write("### AI PR Review (gpt-6.1-sol via DeepClub)\n\n" + resp.output_text)
          PY

      - name: Comment on PR
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs')
            const body = fs.readFileSync('review.md', 'utf8')
            await github.rest.issues.createComment({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: context.issue.number,
              body,
            })
```

---

## 四、改用 Claude 原生 `/v1/messages`（可选）

如果你想在安全关键仓库使用 `claude-sonnet-5-5` 或 `claude-opus-5-5`，把上面的 `pip install openai` 换成 `pip install anthropic`，并改为：

```python
from anthropic import Anthropic

client = Anthropic(
    api_key=os.environ["DEEPCLUB_API_KEY"],
    base_url="https://deepclub.net",  # 注意 Anthropic SDK 不带 /v1
)
msg = client.messages.create(
    model="claude-sonnet-5-5",
    max_tokens=2048,
    messages=[{"role": "user", "content": prompt}],
)
text = "".join(b.text for b in msg.content if b.type == "text")
```

> **安全最佳实践**：
> 1. Workflow 中已加 `head.repo.full_name == github.repository` 防止恶意 Fork 读取 Secret，或触发未授权请求；
> 2. 始终对 diff 做长度截断（示例限制 `24000` 字符），避免自动生成的大文件 lockfile 刷爆 Token 费用；
> 3. 深云 DeepClub 实行 ZDR 零留存，CI 发送的代码片段不会被持久化保存。

定价与子 Key 管控说明：[https://deepclub.net/pricing](https://deepclub.net/pricing)
