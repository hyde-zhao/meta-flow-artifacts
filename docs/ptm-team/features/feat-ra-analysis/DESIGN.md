---
status: draft
version: "1.0"
source_cr: "CR-030"
source_hld: "docs/design/HLD.md (v1.2, REV-03)"
feature_id: "FEAT-RA-ANALYSIS"
agent: "ptm-tse"
created_by: "meta-se"
created_at: "2026-07-16"
---

# FEAT-RA-ANALYSIS: 逆向问题分析 — Feature 设计

> 对应 HLD REV-03 的 `reverse-analysis` Skill 边界。

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-16 | meta-se | 初始 Feature 设计。 |

---

## 1. Feature 概述

### 目标

让测试架构师基于 SQLite 中的版本化问题单数据，执行六维逆向分析，产出区分事实、假设、证据支撑和人工确认状态的审计级分析报告，并支持 S1（新增分析）和 S2（增量更新分析）两种分析模式。

### 成功标准

| 指标 | 度量方式 | 目标值 |
|---|---|---|
| 根因可信 | 根因四层状态机不可自动跃迁（raw→candidate→evidence-backed→confirmed） | fixture 证明 |
| 结论可审计 | 100% 结论可追溯到 evidence_ref 或 reviewer 确认 | 报告 schema 审查 |
| 六维覆盖 | 每份报告覆盖根因/产品质量/流出/漏测/改进/环比同比 | 100% |
| 无分母不称密度 | 无从信分母时输出数量/占比/Pareto/趋势 | fixture 证明 |
| S2 差异可解释 | 增量重算差异报告 + 环比同比窗口/N-A 标记 | 100% |

---

## 2. 关键接口

### 2.1 reverse-analysis Skill 接口

| 属性 | 值 |
|---|---|
| 输入（S1） | batch_id 或 ticket_id 范围 → SQLite 只读查询 |
| 输入（S2） | 新 batch_id + 对比 batch_id → 版本化数据集 |
| 输出 | RA Report（事实/假设/证据/确认）+ analysis_run 记录 |
| 禁止 | 确认根因（仅 reviewer）、修改 ticket 表、CA/PA 批准 |

### 2.2 AnalysisRunManifest 结构

```yaml
analysis_run_id: string
batch_ref: string            # ingestion_batch.batch_id
comparison_batch_ref: string # S2 only
schema_version: string
mapping_rule_version: string
analysis_rule_version: string
window:
  start: date
  end: date
recompute_mode: enum         # 'full' | 'incremental'
report_refs:
  - report_id: string
    type: enum               # 'single_ticket' | 'batch_summary' | 'difference'
created_at: datetime
```

### 2.3 MetricDefinition 模板

```yaml
metric_id: string
name: string
dimension: enum              # 'root_cause' | 'product_quality' | 'escape' | 'test_missed' | 'improvement' | 'comparison'
numerator:
  field: string
  filter: string
denominator:
  field: string
  filter: string
  trusted: boolean           # 有可信分母才为 true
window_days: int
cutoff_date: date
version: string
na_condition: string         # 何时标记 N/A
degraded_output: string      # 无分母时的降级输出方式
```

---

## 3. 六维分析引擎

| 维度 | 输入数据源 | 分析方法 | 输出要求 |
|---|---|---|---|
| 根因 | ticket.root_cause, 时间/责任字段, evidence_refs | 5 Whys/鱼骨候选 + 事实/假设分离 | `raw_statement → AI candidate → evidence-backed → reviewer-confirmed` 四层状态 |
| 产品质量 | ticket.product, module, severity, status | 数量/占比/Pareto/趋势；有分母时启用缺陷密度 | Top 风险模块、质量趋势图 |
| 流出 | 测试/发布/监控控制证据（人工补充） | 控制层逃逸矩阵 | escape_point + 最近拦截点；无证据输出 candidate escape layer |
| 漏测 | ticket.test_missed_analysis, test_missed_phase | PPDCS 归类 | 漏测模式 + 建议测试设计方法 |
| 改进 | ticket.improvement_measures + 分析结论 | CAPA 候选生成 | 待批准 CA/PA（纠正/预防 + Owner + 验收 + 有效性检查） |
| 环比/同比 | ticket.openeddate, resolveddate, 版本化状态 | 完整月份/季度窗口同口径聚合 | 基数 + 绝对变化 + 变化率 + N/A/可信度 |

### 根因四层状态机

```
raw_statement ──(AI analysis)──► AI candidate
                                      │
                            ┌─(evidence ≥3 lines)──► evidence-backed
                            │                              │
                            │                    ┌─(reviewer confirm)──► reviewer-confirmed
                            │                    │
                            └─(evidence <3 lines)──► stays at AI candidate (gap report)
```

- 自动跃迁：raw_statement → AI candidate（AI 分析）
- 禁止自动跃迁：AI candidate → evidence-backed（需三条证据线）
- 禁止自动跃迁：evidence-backed → reviewer-confirmed（需人工确认）

---

## 4. 关键流程

### S1 新增分析
```
1. 从 SQLite 读取清洗后数据（batch_id 或时间窗口）
2. 资格检查：P1/P2 适用性、内部问题拒绝、越权拒绝
3. 六维分析执行 → 事实/假设/证据缺口
4. 生成 analysis_run manifest
5. 输出 RA Report 草案 + CA/PA 候选
6. 写入 analysis_run 记录到 SQLite
```

### S2 更新分析
```
1. 读取新 batch + 对比 batch 的版本化数据集
2. 检测受影响的维度（变更字段映射到分析维度）
3. 增量重算受影响维度的指标
4. 环比/同比计算（完整同口径窗口）
5. 输出差异报告 + N/A 标记
6. 规则变更触发全量重算
```

---

## 5. 安全与治理约束

| 契约 | 要求 | 验证 |
|---|---|---|
| AnalysisRunManifest | 每份报告绑定 batch/version/窗口/重算模式 | schema check |
| MetricDefinition | 指标分子/分母/窗口/N-A 条件版本化 | schema check |
| 根因四层 | 不可自动跃迁至 evidence-backed / confirmed | fixture |
| 流出控制证据 | candidate/confirmed escape layer 分离 | fixture |
| 敏感字段策略 | 未分类字段不进 LLM/正文 | sensitive-field fixture |
| deny-by-default | 无凭据、无外部写入、无自动确认 | forbidden-path fixture |

---

## 6. 与相邻 Feature 契约

- **读取 FEAT-RA-INGESTION**: ticket/ticket_version/ingestion_batch（只读 SQLite）
- **提供给 FEAT-RA-IMPROVEMENT**: RA Report（已确认部分）+ analysis_run 引用
- **不直接对接 FEAT-RA-TRACKING**: 通过 IMPROVEMENT 间接传递

---

## 7. Gotchas

- **不要把 AI 推测写成根因**：只有 evidence-backed 状态才能进入 reviewer 确认
- **不要把无分母的数据称为密度**：降级为数量/占比/Pareto/趋势
- **不要让 S2 增量重算悄悄变成全量**：窗口不足、规则变更才触发全量
- **不要把 raw_statement 直接展示为结论**：必须经过 AI candidate 标注
- **不要假设所有 ticket 都有完整的 test_missed_analysis**：缺失字段标记为 unknown
