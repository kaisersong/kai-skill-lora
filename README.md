# kai-skill-lora

Content-generation LoRA adapters fine-tuned so a small model runs two content-layout skills — **slide-creator** and **kai-report-creator** — more reliably.

内容生成 LoRA 适配器：让小模型更稳定地执行 slide-creator 与 kai-report-creator 两个内容布局 Skill。

> **Weights live on Hugging Face — this repository contains no model files.**
> **模型权重在 Hugging Face，本仓库不含任何权重文件。**
> Download / 下载：https://huggingface.co/Songkai/kai-skill-lora

## What it is / 这是什么

- Base model / 基座：Qwen3-4B-Instruct-2507
- Method / 方法：QLoRA fine-tune, released version **fix3**
- One repo, two adapters / 单仓双适配器：
  - `content/` — generates the content / IR for the skills
  - `router/` — routes a request to the right skill and output format

## Key results / 关键指标

| Metric / 指标 | Result / 结果 |
|---|---|
| Routing eval / 路由评测 | 60/60 (within the 60-case set / 60 例集内) |
| IR first-pass / IR 一次通过 | 46/52 |
| Latency / 时延 | routing + IR generation median ≈ 77.7 s per case (RTX 4090, 4-bit, single-stream; excludes full rendering and delivery) / 路由＋IR 生成中位约 77.7 秒/例（4090、4bit、单流；不含完整渲染交付） |

Against GLM-5.3 given the full Skill contract, on the same set and the same validators: first-pass 46/52 vs 45/52 — observed values are close, and statistical equivalence or a lead is not claimed. GLM-5.3 reached 52/52 after one repair round; the local model's repair effect has not been measured.
与拿到完整 Skill 契约的 GLM-5.3 同集同验证器对比：首轮 46/52 对 45/52，观察值接近，不宣称统计等效或领先；GLM 一轮修复达 52/52，本地模型修复效果尚未测。

Evaluation details (including the separate held-out set used for the release decision) are on the Hugging Face model card.
评测口径细节（包括发布判定用的独立保密集）以 Hugging Face 模型卡为准。

## Download & use / 下载与使用

```python
from huggingface_hub import snapshot_download

path = snapshot_download(repo_id="Songkai/kai-skill-lora")
# path/content = content adapter, path/router = router adapter
# Load either adapter on top of the base model Qwen3-4B-Instruct-2507.
```

Full usage instructions, adapter configuration and evaluation notes are on the Hugging Face model card — this GitHub repo is a pointer, the card is the source of truth.
完整使用方式、适配器配置与评测说明见 Hugging Face 模型卡；本 GitHub 仓库只是索引，细节以模型卡为准。

## Known limitations / 已知短板

- Adapters only — they do nothing without the base model. 仅适配器，必须配合基座模型使用。
- Known failure cases are documented on the Hugging Face model card. 已知失败例以 Hugging Face 模型卡的记录为准。

## Roadmap (summary) / 下一步计划（摘要）

Full roadmap on the Hugging Face model card. 完整版见 Hugging Face 模型卡。

- **Phase 1 / 第一期**：segmented latency profile (prefill/decode, token counts, adapter switching, validation, rendering) and production-contract alignment with a non-author trial run. 时延分段 profile、生产契约对齐与非作者试跑。
- **Phase 2 / 第二期**：E1 content-loop experiment (chart safety constraints + one budgeted local validation-repair pass; KPI blocks stay explicitly missing when no real data exists — numbers are never fabricated) and E2 inference-stack comparison. E1 内容闭环（约束＋一次本地修复对照，KPI 无真数据不编）、E2 推理栈比较。
- **Phase 3 / 第三期**：targeted top-up training on the remaining semantic errors. 按剩余语义错误定向补训。

Acceptance criteria are set before each experiment, from actual costs and budget — no pre-committed hard gates. 验收标准在实验前按实际成本与预算定，不预设硬门槛。
