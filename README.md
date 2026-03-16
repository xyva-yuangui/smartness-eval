<div align="center">

# 🎯 OpenClaw Smartness Eval

**自动化评估 AI Agent 智能度的开源技能**

[![Version](https://img.shields.io/badge/version-0.2.0-blue.svg)](https://github.com/xyva-yuangui/smartness-eval)
[![Python](https://img.shields.io/badge/python-3.10+-green.svg)](https://python.org)
[![License](https://img.shields.io/badge/license-MIT-yellow.svg)](LICENSE)
[![Platform](https://img.shields.io/badge/platform-OpenClaw-purple.svg)](https://github.com/nicepkg/openclaw)

*不再凭"感觉"判断 Agent 是否在进步 —— 用数据说话。*

</div>

---

## 🧠 这是什么

一个为 [OpenClaw](https://github.com/nicepkg/openclaw) 生态设计的 **Agent 自评技能**，通过 28 项自动化测试 + 10+ 真实运行数据源 + 多维度交叉验证，量化评估 Agent 的智能水平。

**不是玩具 benchmark，而是面向生产环境的持续能力监控。**

### 核心特性

- 🔬 **12 维度评估** — 理解、分析、思考、推理、自我迭代、沟通、响应、鲁棒性、泛化、策略遵循、工具可靠性、校准
- 📊 **28 项真实测试** — 不只检查"脚本能跑"，而是验证"能力是否真实"
- 🗃️ **多数据源融合** — 推理知识库 (SQLite)、错误追踪、延迟指标、告警日志、反思报告、真实交互日志
- 📈 **趋势追踪** — 维度级 delta 对比，退化预警，历史记录
- 🎲 **反作弊** — 随机探针测试，防止为提分而优化
- 🔁 **pass@k 可靠性** — deep 模式下重复运行，计算可靠性指标
- 🤖 **LLM Judge** — 可选的 LLM 主观评分，为客观数据补充人性化判断
- 📋 **结构化报告** — JSON + Markdown 双格式输出

---

## ⚡ 快速开始

### 安装

将此仓库放入 OpenClaw workspace 的 `skills/` 目录：

```bash
# 方式一：ClawHub 安装（推荐）
clawhub install openclaw-smartness-eval

# 方式二：手动安装
git clone https://github.com/xyva-yuangui/smartness-eval.git \
  ~/.openclaw/workspace/skills/openclaw-smartness-eval
```

### 运行

```bash
# 快速评估（~30 秒，10 项测试）
python3 skills/openclaw-smartness-eval/scripts/eval.py --mode quick

# 标准评估（推荐，~25 项测试 + 随机探针）
python3 skills/openclaw-smartness-eval/scripts/eval.py --mode standard

# 深度评估（全部测试 x2，pass@k，30 天窗口）
python3 skills/openclaw-smartness-eval/scripts/eval.py --mode deep --compare-last

# Markdown 报告
python3 skills/openclaw-smartness-eval/scripts/eval.py --mode standard --format markdown

# LLM 裁判评分（需设置 DEEPSEEK_API_KEY）
python3 skills/openclaw-smartness-eval/scripts/eval.py --mode standard --llm-judge
```

### 健康检查

```bash
python3 skills/openclaw-smartness-eval/scripts/check.py
```

---

## 📐 评估维度

### 核心维度（权重 70%）

| 维度 | 权重 | 评估内容 | 数据来源 |
|:-----|:----:|:---------|:---------|
| **Understanding** 理解 | 10% | 意图识别、约束捕获、上下文一致性 | message-analyzer 测试 + 交互日志 |
| **Analysis** 分析 | 10% | 问题拆解、依赖识别、方案结构 | 推理库 + 回归指标 + benchmark |
| **Thinking** 思考 | 10% | 风险意识、反面思考、边界检查 | 安全审计 + 告警 + 推理高置信 |
| **Reasoning** 推理 | 15% | 逻辑链完整性、证据支持、根因分析 | 推理库 SQLite + benchmark |
| **Self-iteration** 自我迭代 | 10% | 错误修复率、模式沉淀、重复错误下降 | error-tracker + pattern-library + reflection |
| **Communication** 沟通 | 10% | 表达清晰度、需求覆盖、语气贴合 | 真实交互日志 + benchmark |
| **Responsiveness** 响应 | 5% | P50/P95 时延、超时率 | latency-metrics + regression |

### 扩展维度（权重 30%）

| 维度 | 权重 | 评估内容 | 数据来源 |
|:-----|:----:|:---------|:---------|
| **Robustness** 鲁棒性 | 8% | 异常输入、噪声、长上下文稳定性 | benchmark + 告警 + 重复错误 |
| **Generalization** 泛化 | 5% | 多场景迁移、跨域路由 | 意图分布 + orchestrator 日志 |
| **Policy Adherence** 策略遵循 | 7% | 安全确认、路由约束、操作规范 | cron-governor + 安全审计 |
| **Tool Reliability** 工具可靠 | 5% | 脚本执行、Cron 健康、状态落盘 | benchmark + cron + rule-candidates |
| **Calibration** 校准 | 5% | 置信度准确性、高置信错误控制 | 推理库 + finalize + 高优错误 |

---

## 📊 评分体系

### 等级

| 等级 | 分数 | 含义 |
|:----:|:----:|:-----|
| A+ | ≥92 | 卓越——各维度均衡且优秀 |
| A | ≥88 | 优秀——绝大部分维度表现出色 |
| A- | ≥84 | 良好偏上 |
| B+ | ≥80 | 良好 |
| B | ≥76 | 中等偏上 |
| B- | ≥72 | 中等 |
| C+ | ≥68 | 及格偏上——有明显短板 |
| C | ≥64 | 及格 |
| D | <64 | 不及格——需要立即改进 |

### 每个维度的评分公式独立设计

不同于简单的 `task * 0.8 + benchmark * 0.2` 一刀切，每个维度有独立的评分公式，
融合任务测试分数、运行指标、日志数据等多信号。例如：

- **self_iteration**: 任务分 × 25% + 修复率 × 20% + 验证率 × 15% + 模式推广率 × 15% + 反思报告 × 15% + 重复错误下降 × 10%
- **reasoning**: 任务分 × 40% + benchmark × 15% + 推理高置信比 × 25% + 推理库规模 × 20%
- **responsiveness**: 非线性延迟函数 (P50^1.5 + P95^1.4 + 超时惩罚)

---

## 🔬 工作原理

```
┌─────────────────────────────────────────────┐
│                 eval.py                      │
├──────────────┬──────────────┬───────────────┤
│  Task Suite  │  Metrics     │  Data Sources │
│  (28 tests)  │  Collection  │  Integration  │
├──────────────┼──────────────┼───────────────┤
│ message-     │ latency      │ reasoning.db  │
│ analyzer     │ error-tracker│ reflection    │
│ benchmark    │ pattern-lib  │ alerts.jsonl  │
│ security     │ cron-gov     │ analyzer-log  │
│ reasoning    │ benchmark    │ regression    │
│ + probes     │ orchestrator │ rule-cands    │
├──────────────┴──────────────┴───────────────┤
│          12 独立评分公式                      │
│          加权汇总 → 总分 + 等级               │
│          置信区间 (重复运行方差)               │
│          维度趋势对比                         │
│          → JSON + Markdown 报告              │
└─────────────────────────────────────────────┘
```

---

## 📁 目录结构

```text
smartness-eval/
├── README.md                    # 本文件
├── SKILL.md                     # OpenClaw 技能描述
├── CLAWHUB-UPLOAD-GUIDE.md      # ClawHub 上传指南
├── _meta.json                   # 技能元数据
├── LICENSE                      # MIT 许可证
├── config/
│   ├── config.json              # 评估配置（权重、模式、窗口）
│   ├── rubrics.json             # 12 维度 0-5 量表定义
│   └── task-suite.json          # 28 项测试定义
└── scripts/
    ├── eval.py                  # 核心评估引擎
    └── check.py                 # 健康检查
```

---

## 🔧 配置

### 维度权重 (`config/config.json`)

```json
{
  "weights": {
    "main": {
      "understanding": 10, "analysis": 10, "thinking": 10,
      "reasoning": 15, "self_iteration": 10,
      "dialogue_communication": 10, "responsiveness": 5
    },
    "expanded": {
      "robustness": 8, "generalization": 5, "policy_adherence": 7,
      "tool_reliability": 5, "calibration": 5
    }
  }
}
```

权重总和 = 100。可根据实际优先级调整。

### 评估模式

| 模式 | 测试范围 | 重复次数 | 时间窗口 | 随机探针 |
|:-----|:---------|:--------:|:--------:|:--------:|
| `quick` | core 标签 (~10) | 1 | 3 天 | 1 个 |
| `standard` | core + standard (~25) | 1 | 7 天 | 2 个 |
| `deep` | 全部 (~28) | 2 | 30 天 | 3 个 |

### 量表 (`config/rubrics.json`)

每个维度都有完整的 0-5 分量表定义，用于人工审核和未来 LLM Judge 的锚定。

---

## 🛡️ 反作弊机制

1. **随机探针** — 每次运行注入 1-3 个随机测试输入，防止为固定测试集优化
2. **pass@k** — deep 模式下同一测试运行 2 次，计算组合可靠性
3. **时间窗口过滤** — 错误和指标按时间窗口过滤，防止历史数据稀释当前状态
4. **多数据源交叉** — 单一数据源作弊不影响最终分数

---

## 📈 示例输出

```
# OpenClaw Smartness Eval — 2026-03-16T10:34:39

> **⚠️ Overall: 70.22 (C+)** | CI: [70.22, 70.22] | mode: standard | samples: 30

**最强维度**: analysis (95.39)
**最弱维度**: responsiveness (34.71) ← 优先提升

## 主维度评分
| 维度 | 分数 | 趋势 |
|------|------|------|
| understanding | 62.74 |  (-7.9) |
| analysis | 95.39 |  (+0.0) |
| thinking | 72.03 |  (+2.7) |
| reasoning | 81.23 |  (+0.0) |
| self_iteration | 51.68 |  (+14.5) |
| dialogue_communication | 68.33 |  (+0.0) |
| responsiveness | 34.71 |  (+0.0) |
```

---

## 🤝 贡献

欢迎提交 PR 和 Issue！可以贡献的方向：

- **新测试用例** — 在 `config/task-suite.json` 中添加
- **评分公式优化** — 在 `scripts/eval.py` 的 `merge_scores()` 中调整
- **新数据源接入** — 在 `collect_metrics()` 中扩展
- **新维度** — 在 rubrics + config + eval 三处同步添加

### 开发流程

```bash
# 1. Fork & Clone
git clone https://github.com/YOUR_NAME/smartness-eval.git

# 2. 安装到 OpenClaw workspace
ln -s $(pwd)/smartness-eval ~/.openclaw/workspace/skills/openclaw-smartness-eval

# 3. 修改后验证
python3 skills/openclaw-smartness-eval/scripts/check.py
python3 skills/openclaw-smartness-eval/scripts/eval.py --mode quick --no-probes

# 4. 提交 PR
```

---

## 📄 License

MIT License. See [LICENSE](LICENSE).

---

<div align="center">

**Built with ❤️ for the OpenClaw ecosystem**

[ClawHub](https://github.com/nicepkg/openclaw) · [Report Bug](https://github.com/xyva-yuangui/smartness-eval/issues) · [Request Feature](https://github.com/xyva-yuangui/smartness-eval/issues)

</div>