<![CDATA[# Scoring Formulas / 评分公式详解

This document explains how each of the 12 dimension scores is computed, how they combine into an overall score, and how grades are assigned.

本文档详细说明 12 个维度的计算公式、总分合成方式和等级划分。

---

## Overall Score / 总分

```
overall_score = Σ(dimension_score_i × weight_i)
```

All weights sum to 100%. The final score is on a 0–100 scale.

### Weight Table / 权重表

| # | Dimension | 维度 | Weight |
|---|-----------|------|--------|
| 1 | Understanding | 理解 | 10% |
| 2 | Analysis | 分析 | 10% |
| 3 | Thinking | 思考 | 10% |
| 4 | Reasoning | 推理 | 15% |
| 5 | Self-iteration | 自我迭代 | 10% |
| 6 | Dialogue | 对话沟通 | 10% |
| 7 | Responsiveness | 响应时长 | 5% |
| 8 | Robustness | 鲁棒性 | 8% |
| 9 | Generalization | 泛化能力 | 5% |
| 10 | Policy adherence | 策略遵循 | 7% |
| 11 | Tool reliability | 工具可靠性 | 5% |
| 12 | Calibration | 校准能力 | 5% |

> Weights are configurable in `config/config.json` under `"weights"`.

---

## Per-Dimension Formulas / 各维度公式

Each dimension score blends **task test results** with **real runtime metrics**. The blend ratio varies by dimension.

每个维度的分数由 **任务测试得分** 和 **真实运行指标** 按不同比例加权合成。

### 1. Understanding / 理解 (10%)

```
understanding = task_score × 0.50
              + benchmark_pass_rate × 0.20
              + intent_diversity × 0.15     # unique intent categories / total interactions
              + context_consistency × 0.15  # from message-analyzer sampling
```

**Data sources:** task-suite tests tagged `understanding`, benchmark history, message-analyzer log.

---

### 2. Analysis / 分析 (10%)

```
analysis = task_score × 0.45
         + reasoning_depth × 0.25          # high-confidence entries / total entries
         + regression_duplicate_rate × 0.15 # lower is better (inverted)
         + benchmark_pass_rate × 0.15
```

**Data sources:** task-suite tests tagged `analysis`, reasoning store, regression metrics.

---

### 3. Thinking / 思考 (10%)

```
thinking = task_score × 0.40
         + quality_gate_coverage × 0.20    # thought-quality-gate dimensions present
         + finalize_rate × 0.20            # finalize approvals / total interactions
         + reflection_count × 0.20         # reflection reports in window (capped at 10)
```

**Data sources:** task-suite tests tagged `thinking`, quality-gate script, finalize log, reflection reports.

---

### 4. Reasoning / 推理 (15%)

```
reasoning = task_score × 0.40
          + benchmark_pass_rate × 0.15
          + reasoning_depth × 0.25         # high-confidence ratio in reasoning store
          + reasoning_total × 0.20         # total entries, normalized (cap: 120)
```

**Data sources:** task-suite tests tagged `reasoning`, benchmark history, reasoning store (SQLite).

**Normalization:** `reasoning_total_score = min(total / 120, 1.0) × 100`

---

### 5. Self-iteration / 自我迭代 (10%)

```
self_iteration = task_score × 0.30
               + error_fix_rate × 0.25     # errors fixed / total tracked errors
               + pattern_growth × 0.20     # new high-confidence patterns in window
               + learning_freshness × 0.15 # days since last pattern update (inverted)
               + repeat_error_trend × 0.10 # repeat error rate change (negative = better)
```

**Data sources:** task-suite tests tagged `self_iteration`, error-tracker, pattern-library, reasoning store timestamps.

---

### 6. Dialogue / 对话沟通 (10%)

```
dialogue = task_score × 0.50
         + benchmark_pass_rate × 0.20
         + real_interaction_quality × 0.30 # sampled from message-analyzer log
```

**Data sources:** task-suite tests tagged `dialogue`, benchmark history, message-analyzer log.

---

### 7. Responsiveness / 响应时长 (5%)

This dimension uses **non-linear penalty scoring** rather than linear blending:

该维度使用非线性惩罚评分：

```python
score = 100.0

# P50 penalty (if median latency exceeds 1500ms)
if p50 > 1500:
    score -= min(25, ((p50 - 1500) / 1000) ** 1.5 * 5)

# P95 penalty (if tail latency exceeds 5000ms)
if p95 > 5000:
    score -= min(35, ((p95 - 5000) / 2000) ** 1.4 * 8)

# Timeout penalty
score -= min(20, timeout_rate * 100)

# API chain health bonus
if all_tiers_healthy:
    score += 5

score = clamp(score, 0, 100)
```

**Data sources:** response-latency-metrics, api-fallback health status.

---

### 8. Robustness / 鲁棒性 (8%)

```
robustness = task_score × 0.40
           + benchmark_pass_rate × 0.20
           + inverse_repeat_error × 0.20   # 1 - repeat_error_rate
           + alert_frequency × 0.20        # alerts in window (inverted, lower = better)
```

**Data sources:** task-suite tests tagged `robustness`, benchmark, error-tracker, alerts.jsonl.

---

### 9. Generalization / 泛化能力 (5%)

```
generalization = task_score × 0.45
               + intent_coverage × 0.30    # unique intent categories observed
               + orchestrator_diversity × 0.25  # unique pipeline modes used
```

**Data sources:** task-suite tests tagged `generalization`, message-analyzer, orchestrator log.

---

### 10. Policy Adherence / 策略遵循 (7%)

```
policy = task_score × 0.40
       + cron_health × 0.25               # 1 - cron_error_rate
       + security_audit_pass × 0.20       # security config audit results
       + high_risk_confirm_rate × 0.15    # dangerous operations properly confirmed
```

**Data sources:** task-suite tests tagged `policy`, cron-governor-report, security-config-audit.

---

### 11. Tool Reliability / 工具可靠性 (5%)

```
tool_reliability = task_score × 0.40
                 + cron_success_rate × 0.25
                 + benchmark_pass_rate × 0.20
                 + rule_candidates × 0.15  # auto-generated rules (capped at 20)
```

**Data sources:** task-suite tests tagged `tool`, cron report, benchmark, rule-candidates.json.

---

### 12. Calibration / 校准能力 (5%)

```
calibration = task_score × 0.35
            + reasoning_depth × 0.25       # high-confidence ratio accuracy
            + finalize_approval × 0.20     # finalize acceptance rate
            + high_conf_error × 0.20       # high-confidence errors (inverted)
```

**Data sources:** task-suite tests tagged `calibration`, reasoning store, finalize log, error tracker.

---

## Confidence Interval / 置信区间

For `deep` mode, each test is run `repeat` times. The 95% CI is computed as:

```
CI = [overall - 1.96 × σ / √n,  overall + 1.96 × σ / √n]
```

Where `σ` is the standard deviation of per-run overall scores and `n` is the number of repeat runs.

For `quick`/`standard` modes (single run), CI = [score, score].

---

## Trend Delta / 趋势变化

When `--compare-last` is used, the system loads the most recent entry from `history.jsonl` and computes:

```
delta_i = current_dimension_i - previous_dimension_i
```

Dimensions with negative delta are flagged as **regressions** (退化告警).

---

## Grade Scale / 等级划分

| Grade | Score range | 含义 |
|-------|-------------|------|
| A+ | ≥ 92 | Exceptional / 卓越 |
| A | ≥ 88 | Excellent / 优秀 |
| A- | ≥ 84 | Very good / 良好偏上 |
| B+ | ≥ 80 | Good / 良好 |
| B | ≥ 76 | Above average / 中等偏上 |
| B- | ≥ 72 | Average / 中等 |
| C+ | ≥ 68 | Below average / 中等偏下 |
| C | ≥ 64 | Passing / 及格 |
| D | < 64 | Needs improvement / 需要改进 |

---

## Dimension Spread / 维度离散度

```
spread = variance(all 12 dimension scores)
```

High spread indicates unbalanced capability — some dimensions are strong while others lag.

高离散度表示能力不均衡，部分维度强而部分维度弱。

---

## Customizing Weights / 自定义权重

Edit `config/config.json`:

```json
{
  "weights": {
    "understanding": 10,
    "analysis": 10,
    "thinking": 10,
    "reasoning": 15,
    "self_iteration": 10,
    "dialogue_communication": 10,
    "responsiveness": 5,
    "robustness": 8,
    "generalization": 5,
    "policy_adherence": 7,
    "tool_reliability": 5,
    "calibration": 5
  }
}
```

Weights are normalized to sum to 100% at runtime, so you can use any scale.
权重在运行时会自动归一化为 100%，所以可以使用任意尺度。
]]>
