---
story_id: "STORY-RA-06.2-REFRESH"
canonical_story_id: "ST-RA-06.2-REFRESH"
title: "S2 增量重算、差异报告"
status: "lld-approved"
design_evidence_type: "full-lld"
lld_policy_required_level: "full-lld"
feature: "FEAT-RA-ANALYSIS"
source_cr: "CR-030"
wave: 3
tier: "Tier-A"
created_by: "meta-dev"
created_at: "2026-07-16"
depends_on: ["ST-RA-05.3-ANALYZE", "ST-RA-06.1-DETECT"]
shared_fragments:
  - shared_file: "skills/reverse-analysis/SKILL.md"
    serial_group: "reverse-analysis"
    write_section: "S2 incremental recompute, comparison"
open_items:
  - "OPEN-RA062-01: 规则版本变更检测的精确范围（mapping_rule_version vs analysis_rule_version vs schema_version）"
  - "OPEN-RA062-02: 环比同比 N/A 条件的最小同口径样本阈值"
---

# ST-RA-06.2-REFRESH LLD: S2 增量重算与差异报告

## 0. 工程依据与模板索引

| 来源 | 消费内容 |
|---|---|
| HLD / Feature DESIGN / Feature Matrix | S2 重算、差异报告与 `full-lld` 约束 |
| Story `ST-RA-06.2-REFRESH` | 输出文件、验收标准和依赖 |

本文保留原有详细章节；目标、需求、模块拆分、代码结构、数据模型、API、流程、技术细节、安全、测试、实施、风险与 DoD 分别由后续编号章节定义。

> 对应 HLD REV-03 S2 数据链路后半段：变更检测结果 → 受影响维度映射 → 增量/全量重算 → 差异报告 + 措施刷新提示。

---

## 1. 文件影响范围

| 文件 | 操作 | 变更性质 | 所有者 |
|------|------|---------|--------|
| `skills/reverse-analysis/SKILL.md` | 修改 | 在 ST-RA-05.3 的 S1 管线段之后，追加 S2 增量重算段（§13–§15：变更消费、维度映射、增量/全量引擎、差异报告） | FEAT-RA-ANALYSIS（shared，写 S2 段） |

**不修改**：`templates/analysis-run-manifest.yaml`、`templates/ra-report.yaml`、`templates/metric-definition.yaml`（复用 ST-RA-05.3 和 ST-RA-02 已建模板）。

---

## 2. 接口设计

### 2.1 S2 增量重算入口契约

| 接口项 | 定义 |
|--------|------|
| **触发** | ptm-tse 收到新 batch 完成 + 变更检测结果（来自 ST-RA-06.1-DETECT） |
| **输入 — 新 batch** | new_batch_id（已清洗的新数据） |
| **输入 — 基准** | baseline_batch_id（上一次 S1 分析的 batch_id） |
| **输入 — 变更集** | change_set（来自 ST-RA-06.1-DETECT：新增/修改/未变化 ticket 列表 + 字段级变更） |
| **输入 — 规则版本** | mapping_rule_version, analysis_rule_version 的新旧对比 |
| **输出 — 差异报告** | ra-report.yaml（report_type = difference）|
| **输出 — 环比同比** | ra-report.yaml 的 comparison section |
| **输出 — analysis_run** | 一条新的 analysis_run 记录（recompute_mode = 'incremental' 或 'full'） |
| **禁止** | 覆盖历史 analysis_run、修改 baseline ra-report、自动确认措施状态变更 |

### 2.2 变更字段 → 分析维度映射表

| 变更字段（ticket 表） | 影响的分析维度 | 重算范围 |
|----------------------|-------------|---------|
| `root_cause` | 根因（root_cause） | 单 ticket 根因重算 + 聚合根因统计 |
| `product`, `module` | 产品质量（product_quality） | 受影响的 product/module 聚合 + Pareto |
| `severity` | 产品质量、根因 | 严重度分布 + 相关统计 |
| `status` | 产品质量、流出 | 状态分布 + 逃逸分析 |
| `test_missed_analysis`, `test_missed_phase` | 漏测（test_missed） | PPDCS 归类重算 |
| `improvement_measures` | 改进（improvement） | CA/PA 候选差异 |
| `openeddate`, `resolveddate` | 环比同比（comparison） | 窗口内聚合重算 |
| 新增 ticket | **全部**（新事件的所有维度） | 该 ticket 全维度分析 |
| 规则版本变更 | **全部** | 全量重算（所有 ticket + 所有维度） |

### 2.3 环比同比接口

```yaml
comparison:
  mode: enum               # yoy | mom | none
  baseline_period:
    start: date
    end: date
    total_count: int
  current_period:
    start: date
    end: date
    total_count: int
  metrics:
    - metric_id: string
      name: string
      baseline_value: number
      current_value: number
      absolute_change: number
      change_rate: number       # 百分比
      credibility: enum         # high | medium | low
      na_reasons: []            # 如 ["insufficient_window", "denominator_change"]
```

**N/A 条件**（任一满足即标记）：
1. 无同口径历史窗口（如同月去年数据不存在）
2. 窗口内样本量 < 最小阈值（建议 10）
3. 分母口径版本变更（denominator 定义不一致）
4. 规则版本变更导致旧数据不可比

---

## 3. 数据模型

### 3.1 差异报告（ra-report difference 类型）

基于 `ra-report.yaml` 模板扩展，`report_type = 'difference'` 的专有字段：

```yaml
report_type: difference
sections:
  change_summary:
    new_tickets: int
    modified_tickets: int
    unchanged_tickets: int
    affected_dimensions: []    # 被影响的维度名称列表
    recompute_mode: enum       # incremental | full
    full_recompute_reason: string  # 当为 full 时填写原因
  dimension_diffs:             # 逐维度的前后差异
    - dimension: string
      affected_ticket_count: int
      before_summary: {}       # 指标快照
      after_summary: {}
      significant_changes: []
  comparison:                  # 环比同比
    mode: enum
    metrics: []
    na_reasons: []
  improvement_refresh:
    affected_measures:
      - measure_id: string
        suggestion: enum       # keep | complete | needs_review | invalidated
        reason: string
    new_candidates: []         # 新 CA/PA 候选
```

### 3.2 依赖的 SQLite 表

| 表 | 操作 | 关键字段 |
|----|------|---------|
| `ticket` | SELECT | 新 batch 数据 + baseline batch 数据 |
| `ticket_version` | SELECT | 版本差异对比 |
| `ingestion_batch` | SELECT | mapping_rule_version 对比 |
| `analysis_run` | SELECT + INSERT | 历史 run 引用 + 新 run 写入 |
| `change_history` | SELECT（ST-RA-06.1-DETECT 产出） | 变更集引用 |

---

## 4. 核心流程

### S2 增量重算总流程

```
输入: new_batch_id, baseline_batch_id, change_set, 规则版本对
输出: ra-report（difference）+ analysis_run（incremental/full）

1. 规则版本变更检测
   1a. 比较 new_batch 的 mapping_rule_version vs baseline_batch
   1b. 比较 analysis_rule_version vs baseline
   1c. 任一不同 → recompute_mode = 'full'（全量重算）
   1d. 均相同 → recompute_mode = 'incremental'

2. 受影响维度映射（仅 incremental 模式）
   2a. 遍历 change_set 的字段变更列表
   2b. 应用 §2.2 变更字段 → 维度映射表
   2c. 合并去重 → affected_dimensions[]
   2d. 若无任何维度受影响 → 跳过重算，输出空差异报告

3. 增量重算（incremental）
   3a. 仅对 affected_dimensions 中的维度执行六维引擎
   3b. 对于单 ticket 变更 → 重算该 ticket 的相关维度
   3c. 对于聚合维度变更 → 重算该聚合窗口（受影响的 product/module 等分组）
   3d. 未受影响维度 → 从 baseline ra-report 复制引用
   3e. 环比同比 → 对新旧窗口执行 §4.2 的计算

4. 全量重算（full）
   4a. 所有维度全部重新执行（等同 S1 批量分析）
   4b. recompute_mode = 'full'
   4c. full_recompute_reason 写入 analysis_run

5. 差异生成
   5a. 逐维度比较 before（baseline ra-report）和 after（新计算结果）
   5b. 标记显著变化（变化率 > 阈值 或 新增 ticket）
   5c. 生成 dimension_diffs[]

6. 措施刷新提示
   6a. 从 improvement-tracker 获取当前已批准的 CA/PA 措施列表
   6b. 根据维度变更判断每个措施的受影响程度
   6c. 输出 suggestion: keep | complete | needs_review | invalidated
   6d. 不自动修改措施状态

7. 创建 analysis_run
   7a. recompute_mode = actual mode（incremental 或 full）
   7b. comparison_batch_ref = baseline_batch_id
   7c. report_refs 包含 difference ra-report
   7d. 调用 `insert_analysis_run()` 写入新的草案运行记录

8. 输出差异报告 + 环比同比 + 措施提示
```

### 环比同比计算

```
输入: product, 当前窗口, 比较窗口类型（yoy/mom）
输出: comparison section

1. 确定比较窗口
   1a. yoy: 同年比去年同期——current: (Jan 2026, Mar 2026), baseline: (Jan 2025, Mar 2025)
   1b. mom: 环比——current: (Mar 2026), baseline: (Feb 2026)

2. 窗口完整性检查
   2a. baseline 窗口数据是否存在于 SQLite → 不存在则 N/A
   2b. 窗口内 ticket 样本量是否 >= 最小阈值 → 不足则 N/A + 标注样本量

3. 同口径聚合
   3a. 使用相同的 MetricDefinition（metric_version 相同）
   3b. 使用相同的聚合维度（product/module/severity）
   3c. 若 metric_version 不同 → N/A + "口径变更"

4. 计算变化
   4a. absolute_change = current_value - baseline_value
   4b. change_rate = absolute_change / baseline_value（baseline_value > 0 时）
   4c. baseline_value = 0 时 → change_rate = N/A, 标注 "zero_baseline"

5. 可信度标注
   - high: 口径一致，窗口完整，样本量充足
   - medium: 口径一致，但样本量偏低或被拒绝 ticket 比例偏高
   - low: window 不完整或存在其他质量疑虑
```

---

## 5. 状态机

### S2 分析模式

```
incoming change_set
  │
  ├─► 规则版本未变 + 有受影响维度 → recompute_mode = 'incremental'
  │      └─► 仅重算 affected_dimensions
  │
  ├─► 规则版本未变 + 无受影响维度 → 跳过重算
  │      └─► 输出空差异报告，引用 baseline
  │
  └─► 规则版本变更 → recompute_mode = 'full'
         └─► 全量重算所有维度
```

### analysis_run 状态（复用 ST-RA-05.3）

S2 的 analysis_run 使用相同状态机（created → in_progress → completed | failed）。

`recompute_mode` 是 analysis_run 的属性，不是新状态：

| recompute_mode | 条件 | 后续影响 |
|---------------|------|---------|
| `incremental` | 规则版本未变 | 仅部分维度指标变化 |
| `full` | 规则版本变更 | 全量新指标，旧指标仍可在历史 run 中追溯 |

---

## 6. 错误处理与降级

| 场景 | 处理 | 输出 |
|------|------|------|
| baseline batch 不存在 | blocked | 缺失 baseline batch 错误 |
| change_set 为空 | 跳过所有维度重算 | 空差异报告 + N/A |
| 窗口不足（环比同比） | 标记 N/A | na_reasons[] = "insufficient_window" |
| 样本量不足 | 标记 N/A + 低可信度 | comparison.credibility = low |
| 规则版本变更 | 触发全量重算 | recompute_mode = full |
| 某维度计算异常 | 跳过该维度 | 该维度 marked skipped |
| 措施数据不可用 | 输出 needs_review | suggestion = needs_review |

---

## 7. 测试设计

### 7.1 正向测试

| ID | 场景 | 预期 |
|----|------|------|
| T-ANL-16 | S2 增量重算（受影响维度） | 仅 affected_dimensions 维度重新计算 |
| T-ANL-17 | S2 环比同比计算 | comparison 含窗口/基数/变化率/可信度 |
| T-ANL-S04-01 | 变更字段精确映射到维度 | §2.2 映射表所有条目可验证 |
| T-ANL-S04-02 | 差异报告中 dimension_diffs 正确 | 前后值对比准确，significant_changes 标记 |
| T-ANL-S04-03 | 措施刷新提示生成 | affected_measures 含 keep/needs_review 等 |

### 7.2 负向/边界测试

| ID | 场景 | 预期 |
|----|------|------|
| T-ANL-18 | S2 窗口不足 N/A | comparison.na_reasons 非空，metrics 为空 |
| T-ANL-19 | S2 规则变更全量重算 | recompute_mode = full, full_recompute_reason 非空 |
| T-ANL-S04-04 | 无受影响的维度 | 空差异报告 |
| T-ANL-S04-05 | change_set 为空 | 跳过重算，输出 N/A |
| T-ANL-S04-06 | baseline batch 不存在 | blocked |
| T-ANL-S04-07 | baseline_value = 0（环比同比） | change_rate = N/A, 标注 zero_baseline |

### 7.3 Fixture 设计

| Fixture | 覆盖 |
|---------|------|
| `fixtures/s2_before_after_datasets.json` | T-ANL-16, 17, 18 |
| `fixtures/s2_rule_version_change.json` | T-ANL-19 |
| `fixtures/s2_empty_change_set.json` | T-ANL-S04-05 |
| `fixtures/s2_zero_baseline.json` | T-ANL-S04-07 |

---

## 8. 安全与权限

| 检查项 | 要求 |
|--------|------|
| 只读取，不覆盖 | 不覆盖 baseline ra-report 或 baseline analysis_run |
| SQLite 写入范围 | 仅通过公共 `insert_analysis_run()` 创建新的草案运行记录（和 S1 的 ST-RA-05.3 一致） |
| 措施建议不自动执行 | improvement_refresh.suggestion 仅为提示；不修改 measure 状态 |
| 环比同比不推断缺失数据 | 窗口不足/样本不足标记 N/A，不填充推算值 |

---

## 9. 实施步骤

### TASK-ID 映射

| TASK-ID | 内容 | 输出位置 |
|---------|------|---------|
| TASK-ANL-21 | 实现变更字段 → 分析维度映射 | SKILL.md §13（变更消费与映射段） |
| TASK-ANL-22 | 实现增量重算引擎 | SKILL.md §14（增量引擎段） |
| TASK-ANL-23 | 实现全量重算触发条件检测 | SKILL.md §14（全量触发子段） |
| TASK-ANL-24 | 实现差异报告生成 | SKILL.md §15（差异报告段） |
| TASK-ANL-25 | 实现环比同比计算（含 N/A 条件） | SKILL.md §15（环比同比子段） |

### 执行顺序

```
TASK-ANL-21（维度映射）──► TASK-ANL-22（增量引擎）──► TASK-ANL-24（差异报告）
                                  │
TASK-ANL-23（全量触发）─────────┘      │
                                       ├──► 集成测试
TASK-ANL-25（环比同比）────────────────┘
```

---

## 10. 回滚策略

| 回滚场景 | 操作 |
|---------|------|
| S2 段回退 | 回退 SKILL.md 到 ST-RA-05.3 后状态（保留 S1 管线） |
| 增量重算逻辑错误 | 误差只影响新的 analysis_run（历史 run 不变）；修正后重新创建 analysis_run |
| 环比同比公式变更 | 递增 analysis_rule_version，触发下一次 S2 全量重算 |

**关键**：S2 的结果写入新的 `analysis_run` 记录，不覆盖旧记录。回滚只需删除错误的新 run。

---

## 11. 平台差异检查

| 平台 | 差异项 | 影响 | 处理 |
|------|--------|------|------|
| 全部 | 日期窗口计算 | 无差异 | 使用标准日期运算 |
| 全部 | 环比同比公式 | 无差异 | 纯算法，无平台依赖 |

**结论**：无平台差异。

---

## 12. 与相邻模块的集成契约

### 12.1 上游依赖

| 上游 | 输出 | 消费方式 |
|------|------|---------|
| ST-RA-05.3-ANALYZE（S1 管线） | baseline ra-report + baseline analysis_run | 文件引用 + SQLite 历史查询 |
| ST-RA-06.1-DETECT（变更检测） | change_set（新增/修改/未变化 ticket + 字段级变更） | 传递方式待 ST-RA-06.1 确定（建议：文件化 change_set 或 SQLite change_history 表） |
| ST-RA-02（六维引擎） | 六维分析方法 | 同 Skill 内复用 |

### 12.2 下游消费

| 下游 | 消费内容 | 传递方式 |
|------|---------|---------|
| ST-RA-03（改进输入治理） | 差异报告中的新 CA/PA 候选 | 文件引用 |
| ST-RA-06.3-TRACK（措施基线管理） | 措施刷新提示（keep/needs_review/invalidated） | 文件引用（通过 ra-report difference） |

### 12.3 change_set 格式契约

```yaml
# 由 ST-RA-06.1-DETECT 提供，本 Story 只定义消费规矩
change_set:
  baseline_batch_id: string
  new_batch_id: string
  changes:
    - ticket_id: string
      change_type: enum       # new | modified | unchanged
      changed_fields: []      # 对于 modified，列出变更字段名
      field_diffs: {}         # 对于 modified，字段新旧值映射
```

---

## 13. 开放项与假设

| ID | 类型 | 描述 | 状态 | 重访条件 |
|----|------|------|------|---------|
| OPEN-RA062-01 | decision | 规则版本变更的精确判定范围（仅 analysis_rule_version？还是 mapping_rule_version 也触发？schema_version 呢？） | OPEN | 与 ST-RA-06.1-DETECT 和 ST-RA-INGEST-DB 对齐 |
| OPEN-RA062-02 | decision | 环比同比 N/A 的最小同口径样本阈值 | OPEN | 建议 >= 10 |
| OPEN-RA062-03 | decision | change_set 的传递格式（SQLite 表 vs 文件化 YAML） | OPEN | 依赖 ST-RA-06.1-DETECT 的 LLD |
| OPEN-RA062-04 | 假设 | 措施刷新提示不自动修改状态，由 ST-RA-06.3-TRACK 消费 | OPEN | improvement_refresh.suggestion 为只读提示 |
| OPEN-RA062-05 | 假设 | 差异报告中 "significant change" 的变化率阈值为 20% | OPEN | 可作为可配置参数 |

---

## 14. LLD 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|------|------|--------|---------|
| 1.0 | 2026-07-16 | meta-dev | 初始 LLD，覆盖 S2 增量/全量重算、变更字段→维度映射、环比同比、差异报告和措施刷新提示 |
| 1.1 | 2026-07-16 | host-orchestrator | CP5 B12：迁移为当前 full-lld 证据兼容格式，增加 canonical Story ID 与 §0 工程依据/章节索引；不改变原设计契约。 |
