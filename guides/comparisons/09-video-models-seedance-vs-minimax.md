# 视频生成模型横向对比：`Seedance-2.0-Fast` vs `Seedance-2.0-Mini` vs `MiniMax-H3`

目前 **[深云 DeepClub](https://deepclub.net)** 在视频赛道同时提供了三款热门按次计费模型：`Seedance-2.0-Fast`、`Seedance-2.0-Mini` 与 `MiniMax-H3`。它们共用相同的异步任务接口范式，但画质倾向、动作连贯性、默认时长规格与成本拉开明显梯度。

---

## 一、三款视频模型规格与定价对照

> 实时支持时长、分辨率与单次价格请以 [https://deepclub.net/pricing](https://deepclub.net/pricing) 为准：

| 模型 ID | 端点与提交方式 | 教程参考时长（`seconds`） | 参考按次价格区间 | 最适合的场景 |
|---|---|---|---|---|
| **`Seedance-2.0-Mini`** | `POST /v1/videos` (`multipart`) | `5` | **¥1.5 ~ ¥4.5** / 个 | 分镜快速预演、Prompt 构图试错、批量草稿视频 |
| **`Seedance-2.0-Fast`** | `POST /v1/videos` (`multipart`) | `5` | **¥2.5 ~ ¥7.5** / 个 | 商业广告短片、运镜控制（推拉摇移/环绕）成品主力 |
| **`MiniMax-H3`** | `POST /v1/videos` (`multipart`) | **`10`** | **¥2.0 ~ ¥4.5** / 个 | 较长动作片段、人物肢体连贯运动、雨雪水花等物理动态真实感 |

---

## 二、常见踩坑与最佳实践

1. **不要直接混用 `seconds` 参数**：教程与销售规格验证中，`Seedance-2.0-*` 示例传 `seconds=5`，而 `MiniMax-H3` 示例传 `seconds=10`。切换模型前务必先在定价页点开具体型号核对当前发布的时长与分辨率选项。
2. **用“Mini 试错 → Fast/H3 成片”工作流省下 50% 视频预算**：
   - 先用 `Seedance-2.0-Mini` 测试镜头主体位置、光线氛围和运镜词是否跑偏；
   - Prompt 定稿后再切到 `Seedance-2.0-Fast` 或 `MiniMax-H3` 生成最终成片。
3. **断点恢复防重扣**：无论选哪款模型，提交 `POST /v1/videos` 拿到的 `id` 必须立即存储，查询超时只需复用 `GET /v1/videos/{task_id}`。完整恢复脚本见 [../media/video-async-tasks-recovery.md](../media/video-async-tasks-recovery.md)。

官网独立对照页：[https://deepclub.net/compare/seedance-2-0-vs-minimax-h3](https://deepclub.net/compare/seedance-2-0-vs-minimax-h3)
