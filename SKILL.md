---
name: openclaw-smartness-eval
description: OpenClaw 智能度综合评伌技能。围绕 14 个维度（含规划能力、幻觉控制）输出综合评分、证据、风险与趋势。对齐 CLEAR/T-Eval/Anthropic 行业标准。
triggers:
version: "0.3.0"
status: beta
updated: 2026-03-16
provides: ["smartness-evaluation", "capability-audit", "self-eval", "benchmark-aggregation", "trend-analysis"]
os: ["darwin", "linux"]
clawdbot: {"emoji": "🎯", "category": "evaluation", "priority": "high"}
---

# OpenClaw Smartness Eval

用于评估 OpenClaw 是否真的“更聪明”，而不是只看单次回答是否看起来不错。

## 适用场景

- **版本升级后回归**：确认能力是否真的提升
- **每周 / 每月自评**：输出结构化能力报告
- **发现退化**：查看哪个维度下降最快
- **准备对外展示**：生成统一口径的能力评估结果

## 命令

### 1) 标准评估

```bash
python3 skills/openclaw-smartness-eval/scripts/eval.py --mode standard
```

### 2) 快速评估

```bash
python3 skills/openclaw-smartness-eval/scripts/eval.py --mode quick
```

### 3) 深度评估

```bash
python3 skills/openclaw-smartness-eval/scripts/eval.py --mode deep --compare-last
```

### 4) 只输出 Markdown

```bash
python3 skills/openclaw-smartness-eval/scripts/eval.py --mode standard --format markdown
```

### 5) 健康检查

```bash
python3 skills/openclaw-smartness-eval/scripts/check.py
```

## 输出内容

评估结果将写入：

- `state/smartness-eval/runs/<timestamp>.json`
- `state/smartness-eval/reports/<date>.md`
- `state/smartness-eval/history.jsonl`

输出结果包含：

- `overall_score`
- `grade`
- `dimension_scores`
- `expanded_scores`
- `evidence`
- `risk_flags`
- `upgrade_recommendations`
- `trend_vs_last`

### 6) LLM Judge 主观评分

```bash
python3 skills/openclaw-smartness-eval/scripts/eval.py --mode standard --llm-judge
```

需设置 `DEEPSEEK_API_KEY` 或 `OPENAI_API_KEY` 环境变量。
该功能会发起外部 API 请求，默认不开启，仅在显式传入 `--llm-judge` 时启用。

## 输出新增字段 (v0.2)

- `dimension_spread` — 维度间离散度
- `trend_vs_last.dimension_deltas` — 各维度分数变化
- `trend_vs_last.degradation_alert` — 退化超过 5 分的维度
- `pass_at_k` — deep 模式下各测试的 pass@k 可靠性
- `llm_judge` — LLM 裁判主观评分和评语

## 数据来源

- `state/response-latency-metrics.json`
- `state/error-tracker.json` (时间窗口过滤)
- `state/pattern-library.json`
- `state/cron-governor-report.json`
- `state/benchmark-results/history.jsonl`
- `state/v5-orchestrator-log.json`
- `state/v5-finalize-log.json`
- `state/message-analyzer-log.json` (真实日志抽样)
- `state/reflection-reports/` (反思报告)
- `state/alerts.jsonl` (告警日志)
- `state/rule-candidates.json`
- `.reasoning/reasoning-store.sqlite` (推理知识库)
- `scripts/regression-metrics-report.py` (回归指标)
- 任务集中的 34 项规则测试命令
- 随机探针测试 (反作弊)

## 模式说明

- `quick` — 小样本 + 关键日志，~10 个测试
- `standard` — 默认周度评估，~25 个测试 + 2 个随机探针
- `deep` — 全部测试 x2 重复运行 + pass@k + 30天窗口 + 趋势对比

## 文件结构

```text
openclaw-smartness-eval/
├── SKILL.md
├── _meta.json
├── config/
│   ├── config.json
│   ├── rubrics.json
│   └── task-suite.json
└── scripts/
    ├── eval.py
    └── check.py
```
