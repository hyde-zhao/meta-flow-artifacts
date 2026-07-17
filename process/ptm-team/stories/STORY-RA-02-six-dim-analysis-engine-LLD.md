---
story_id: "STORY-RA-02"
canonical_story_id: "ST-RA-02"
title: "六维分析引擎（reverse-analysis Skill 核心）"
status: "lld-draft"
design_evidence_type: "full-lld"
lld_policy_required_level: "full-lld"
feature: "FEAT-RA-ANALYSIS"
source_cr: "CR-030"
wave: 2
tier: "Tier-A"
created_by: "meta-dev"
created_at: "2026-07-16"
depends_on: ["ST-RA-01"]
shared_fragments:
  - shared_file: "skills/reverse-analysis/SKILL.md"
    serial_group: "reverse-analysis"
    write_section: "six-dimension engine, root-cause state machine"
  - shared_file: "skills/reverse-analysis/templates/ra-report.yaml"
    serial_group: "reverse-analysis-templates"
    write_section: "RA report schema"
  - shared_file: "skills/reverse-analysis/templates/metric-definition.yaml"
    serial_group: "reverse-analysis-templates"
    write_section: "metric definition schema"
open_items:
  - "OPEN-RA02-01: CA/PA 候选与 improvement-tracker 的精确交接格式"
  - "OPEN-RA02-02: 环比同比窗口的最小样本量阈值"
---

# ST-RA-02 LLD: 六维分析引擎

## 0. 工程依据与模板索引

| 来源 | 消费内容 |
|---|---|
| HLD / Feature DESIGN / Feature Matrix | 六维分析、可信治理契约和 `full-lld` 约束 |
| Story `ST-RA-02` | 输出文件、验收标准和依赖 |

本文保留原有详细章节；目标、需求、模块拆分、代码结构、数据模型、API、流程、技术细节、安全、测试、实施、风险与 DoD 分别由后续编号章节定义。

> 对应 HLD REV-03 六维分析方法表、根因四层状态机和 MetricDefinition 契约。

---

## 1. 文件影响范围

| 文件 | 操作 | 变更性质 | 所有者 |
|------|------|---------|--------|
| `skills/reverse-analysis/SKILL.md` | 修改 | 在 ST-RA-01 的资格检查段之后，追加六维引擎段（§4–§9） | FEAT-RA-ANALYSIS（shared，写六维引擎段） |
| `skills/reverse-analysis/templates/ra-report.yaml` | 创建 | RA Report 输出模板 schema | FEAT-RA-ANALYSIS |
| `skills/reverse-analysis/templates/metric-definition.yaml` | 创建 | MetricDefinition 定义与管理模板 schema | FEAT-RA-ANALYSIS |

**不修改**：`data/schema.sql`、`data/dao.py`（只读消费）、其他 Skill 或 Agent 文件。

---

## 2. 接口设计

### 2.1 六维分析引擎对外契约

| 维度 | 输入字段（从 ticket 表） | 分析方法 | 输出 |
|------|------------------------|---------|------|
| **根因** | `root_cause`, 时间字段, `evidence_refs` | 5 Whys / 鱼骨图候选生成 + 事实/假设分离 | `RootCauseAnalysis`：raw_statement, ai_candidates[], evidence_support, reviewer_status |
| **产品质量** | `product`, `module`, `severity`, `status`, `openeddate` | 按维度聚合 → 数量/占比/Pareto/趋势 | `ProductQualityAnalysis`：top_risk_modules[], quality_trend, defect_density（仅当有可信分母） |
| **流出** | 测试/发布/监控控制证据（ticket 字段 + 人工补充） | 控制层逃逸矩阵 | `EscapeAnalysis`：escape_points[], nearest_intercept_point, candidate_layers[] |
| **漏测** | `test_missed_analysis`, `test_missed_phase` | PPDCS 归类 | `TestMissedAnalysis`：ppdcs_category, missed_pattern, recommended_test_method |
| **改进** | `improvement_measures`, 分析结论 | CAPA 候选草案生成 | `ImprovementCandidates`：capa_items[]（纠正/预防 + Owner + 验收 + 有效性检查） |
| **环比同比** | `openeddate`, `resolveddate`, 版本化状态 | 完整月份/季度窗口同口径聚合 | `ComparisonAnalysis`：baseline, current, absolute_change, change_rate, credibility, na_reasons[] |

### 2.2 根因四层状态机接口

```yaml
root_cause_state:
  current_level: enum  # raw_statement | ai_candidate | evidence_backed | reviewer_confirmed
  raw_statement: string       # 原始 ITR 字段内容
  ai_candidates:              # AI 分析候选列表
    - hypothesis: string
      confidence: enum        # high | medium | low
      method: enum            # 5whys | fishbone | combined
      supporting_evidence: string[]
      contradicting_evidence: string[]
  evidence_backed:
    confirmed_cause: string
    evidence_lines:
      - line_id: string
        type: enum            # fact | hypothesis
        source: string
    evidence_count: int       # 必须 >= 3
  reviewer_confirmed:
    confirmed_by: string
    confirmed_at: datetime
    reviewer_notes: string
  transition_log:
    - from_level: string
      to_level: string
      trigger: string         # auto | manual | threshold_met
      timestamp: datetime
```

### 2.3 状态转换规则（不可自动跃迁）

| 转换 | 触发方式 | 前置条件 |
|------|---------|---------|
| `raw_statement → ai_candidate` | **自动**（AI 分析） | 无额外条件 |
| `ai_candidate → evidence_backed` | **阈值触发**（非自动跃迁） | valid evidence lines >= 3 |
| `evidence_backed → reviewer_confirmed` | **人工确认** | reviewer 显式操作 |
| `ai_candidate → raw_statement` | **人工回退** | reviewer 拒绝候选 |

### 2.4 MetricDefinition 模板

```yaml
# skills/reverse-analysis/templates/metric-definition.yaml
metric_id: string
name: string
dimension: enum              # root_cause | product_quality | escape | test_missed | improvement | comparison
numerator:
  field: string              # 来源字段路径（如 ticket.product + ticket.status）
  filter: string             # 过滤条件（如 severity in [P1, P2]）
denominator:
  field: string              # 来源字段路径
  filter: string
  trusted: boolean           # 有可信分母才为 true
window:
  start: date
  end: date
  type: enum                 # month | quarter | custom
cutoff_date: date
metric_version: string       # 口径版本号
na_condition: string         # N/A 条件描述
degraded_output: string      # 无可信分母时的降级输出方式
  # 示例：degraded_output: "输出绝对数量 + Pareto 分布 + 趋势方向；禁止标注为'缺陷密度'"
```

**关键降级规则**：
- 当 `denominator.trusted = false` 或分母不可获取时，`degraded_output` 生效
- 降级时只能输出：数量、占比、Pareto 分布、趋势方向
- **绝不允许**在任何输出中将降级数据标记为"密度"、"缺陷密度"或"DPMO"

---

## 3. 数据模型

### 3.1 RA Report 模板（ra-report.yaml）

```yaml
report_id: string
analysis_run_id: string
ticket_ref: string           # 用于逐单报告；批量报告为 null
report_type: enum            # single_ticket | batch_summary | difference
created_at: datetime
sections:
  eligibility:
    status: enum
    evidence_line_count: int
    gaps: []
  root_cause:
    current_level: enum
    raw_statement: string
    ai_candidates: []
    evidence_backed: {}
    reviewer_status: enum    # unconfirmed | confirmed
  product_quality:
    metrics: []              # MetricDefinition 引用列表
    top_risk_modules: []
    quality_trend: string
    defect_density: {}       # 仅在有可信分母时填充
    degraded_notice: string  # 降级原因（若适用）
  escape_analysis:
    escape_points: []
    nearest_intercept: string
    candidate_layers: []     # 无确认证据时填充
    confirmed_layers: []     # 有控制证据时填充
  test_missed:
    ppdcs_category: string
    missed_pattern: string
    recommended_method: string
    unknown_fields: []       # 缺失字段标记
  improvement_candidates:
    capa_items:
      - type: enum           # corrective | preventive
        basis: string        # 依据（关联分析结论）
        target: string
        owner_candidate: string
        priority: enum
        acceptance_criteria: string
        effectiveness_check: string
  comparison:
    mode: enum               # yoy | mom | none
    baseline_period: {}
    current_period: {}
    changes: []
    na_reasons: []           # 窗口不足时的 N/A 原因
```

### 3.2 分析输出的数据边界

| 数据类别 | 来源 | 允许写入的位置 | 禁止 |
|---------|------|-------------|------|
| 分析事实（fact） | ticket 字段 | ra-report.yaml → analysis_run | 不写回 ticket 表 |
| AI 假设（hypothesis） | AI 分析生成 | ra-report.yaml → ai_candidates[] | 不标记为 confirmed |
| 确认结论（confirmed） | 人工 reviewer | ra-report.yaml → reviewer_confirmed | 不自动生成 |
| 指标计算结果 | 公式 + ticket 数据 | ra-report.yaml → metrics[] | 不修改 MetricDefinition 模板 |

---

## 4. 核心流程

### 4.1 六维分析总流程

```
输入: 资格检查通过的 ticket（或 ticket 集合）+ 证据分类结果
输出: RA Report 草案（ra-report.yaml 实例）

1. 根因分析
   1a. 读取 ticket.root_cause 作为 raw_statement
   1b. AI 生成 ai_candidates[]（5 Whys 或鱼骨图方法）
   1c. 从 ST-RA-01 传入证据分类结果中提取 evidence_lines
   1d. 若 evidence_count >= 3，状态推进至 evidence_backed
   1e. 若 evidence_count < 3，保持在 ai_candidate 并输出缺口

2. 产品质量分析
   2a. 按 product/module/severity/status 分组聚合
   2b. 生成数量、占比、Pareto 排序
   2c. 若有可信分母 → 计算缺陷密度
   2d. 若无分母 → 应用 degraded_output，禁止密度标注
   2e. 按 openeddate 生成趋势（同比/环比由 §4.3 的 comparison 维度负责）

3. 流出分析
   3a. 读取控制层证据（人工补充字段）
   3b. 有证据 → 标记为 confirmed_layers[]
   3c. 无证据 → 仅标记 candidate_layers[]，附说明

4. 漏测分析
   4a. 读取 ticket.test_missed_analysis, test_missed_phase
   4b. 归类到 PPDCS（Prevention/Protection/Detection/Containment/Sustainment）
   4c. 缺失字段 → 标记 unknown_fields[]

5. 改进分析
   5a. 基于已确认的根因和流出分析，生成 CA/PA 候选草案
   5b. 每项含 type/basis/target/owner_candidate/priority/acceptance/effectiveness
   5c. 标记为 draft → 待 ST-RA-03 管理批准状态

6. 环比同比分析
   6a. 从 SQLite 读取同期 ticket 数据（完整月份/季度窗口）
   6b. 按同口径聚合计算基数、绝对变化、变化率
   6c. 窗口不足 → 标记 N/A + 原因
   6d. 标注可信度
```

### 4.2 根因四层状态机的实现

```
raw_statement:
  └─► AI 自动分析 → ai_candidate（confidence=low|medium|high + evidence 引用）
        ├─► evidence_lines >= 3 → 自动推进到 evidence_backed（有证据支撑）
        │     └─► reviewer 人工确认 → reviewer_confirmed
        ├─► evidence_lines < 3 → 停留在 ai_candidate
        │     输出: gap_report（缺失证据线 + 建议补充方）
        └─► reviewer 拒绝候选 → 回退到 raw_statement
```

---

## 5. 状态机（根因四层）

| 状态 | 含义 | 进入条件 | 退出条件 |
|------|------|---------|---------|
| `raw_statement` | 原始 ITR 字段内容，未经 AI 分析 | 初始状态 | AI 自动分析后进入 `ai_candidate` |
| `ai_candidate` | AI 生成候选根因 + 置信度 | AI 自动分析完成 | evidence >= 3 → `evidence_backed`；evidence < 3 → 保持 + gap report；reviewer 拒绝 → `raw_statement` |
| `evidence_backed` | 有三条以上证据支撑的根因 | evidence_lines >= 3 | reviewer 人工确认 → `reviewer_confirmed` |
| `reviewer_confirmed` | 经人工 reviewer 确认的结论 | reviewer 显式确认操作 | 不可自动退出；仅 reviewer 可回退 |

**硬规则**：
- 从 `ai_candidate` 到 `evidence_backed`：必须有 >= 3 条有效证据线，**不可由 AI 自动跃迁**（需显式满足阈值条件）
- 从 `evidence_backed` 到 `reviewer_confirmed`：**必须人工操作**，AI 不可代劳
- `reviewer_confirmed` 状态的结论不可被后续自动分析覆盖

---

## 6. 错误处理与降级

| 维度 | 异常场景 | 处理方式 |
|------|---------|---------|
| 根因 | root_cause 字段为空 | raw_statement 标记为 "no_statement_available" |
| 根因 | 所有证据线为 gap | ai_candidate 置信度为 low，不推进至 evidence_backed |
| 产品质量 | 无报告日期 | 不生成趋势图，标记 "no_temporal_data" |
| 产品质量 | 分母不可信 | 应用 degraded_output，禁止"缺陷密度"标签 |
| 流出 | 无控制层证据字段 | 全部 escape_point 标记 candidate |
| 漏测 | test_missed_analysis 为空 | 标记 unknown_fields[]，不强行归类 PPDCS |
| 改进 | 无根因/流出分析 | 不生成 CA/PA 候选 |
| 环比同比 | 无同口径历史窗口 | 标记 N/A + 窗口不足原因 |
| 环比同比 | 样本量 < 最小阈值 | 标记 "sample_insufficient" + 可信度 = low |
| 全部维度 | 数据源不可读 | 标记 analysis_run.status = failed |

---

## 7. 测试设计

### 7.1 正向测试

| ID | 场景 | 预期 |
|----|------|------|
| T-ANL-08 | 六维分析正常输出 | 六个维度均产生 ra-report section |
| T-ANL-S02-01 | 根因四层正常推进 | raw → ai_candidate → evidence_backed（>= 3 条证据） |
| T-ANL-S02-02 | Pareto 排序正确 | product_quality.top_risk_modules 按数量降序 |
| T-ANL-S02-03 | 有分母时缺陷密度计算 | defect_density 字段有值 |
| T-ANL-S02-04 | 有控制证据时 confirmed escape | escape_analysis.confirmed_layers 非空 |
| T-ANL-S02-05 | PPDCS 归类正确 | test_missed.ppdcs_category 匹配输入字段 |
| T-ANL-S02-06 | CA/PA 候选生成 | improvement_candidates.capa_items 非空且含六项必填字段 |
| T-ANL-S02-07 | 环比同比正常计算 | comparison.changes[] 含绝对变化和变化率 |

### 7.2 负向/边界测试

| ID | 场景 | 预期 |
|----|------|------|
| T-ANL-07 | 根因四层不可自动跃迁 | ai_candidate → evidence_backed 只在 evidence >= 3 时触发 |
| T-ANL-09 | 无分母时不称密度 | degraded_notice 非空，defect_density 为空 |
| T-ANL-10 | 无控制证据只输出 candidate | escape_analysis.candidate_layers 非空，confirmed_layers 为空 |
| T-ANL-20 | 敏感字段不进 LLM/报告 | 未分类字段被过滤或脱敏 |
| T-ANL-S02-08 | root_cause 为空 | raw_statement = "no_statement_available" |
| T-ANL-S02-09 | 所有证据线为 gap | 停留在 ai_candidate，confidence = low |
| T-ANL-S02-10 | 窗口不足 | comparison.na_reasons 非空，changes 为空 |

### 7.3 Fixture 设计

| Fixture | 覆盖 |
|---------|------|
| `fixtures/six_dim_full_data.json` | T-ANL-08, 09, 10 |
| `fixtures/root_cause_state_machine.json` | T-ANL-07, T-ANL-S02-01 |
| `fixtures/sensitive_field_ticket.json` | T-ANL-20 |
| `fixtures/no_denominator_ticket.json` | T-ANL-09 |
| `fixtures/insufficient_comparison_window.json` | T-ANL-S02-10 |

---

## 8. 安全与权限

| 检查项 | 要求 | 实现 |
|--------|------|------|
| 敏感字段过滤 | 未在 `sensitive_field_classification` 中分类的字段不进入 LLM 提示词或报告正文 | ra-report 生成前进行字段校验 |
| AI 结论限制 | ai_candidate 标注 confidence + evidence 引用，不写入 confirmed 段 | 状态机硬规则 |
| 无外部调用 | 六维分析禁止发起 HTTP/文件系统外部读取（SQLite 除外） | SKILL.md `禁止事项` 显式列出 |
| 指标计算隔离 | MetricDefinition 的计算不访问 ticket 表外的数据 | 执行步骤中明确数据边界 |

---

## 9. 实施步骤

### TASK-ID 映射

| TASK-ID | 内容 | 输出位置 |
|---------|------|---------|
| TASK-ANL-07 | 实现根因四层状态机 | SKILL.md §4（根因引擎段） |
| TASK-ANL-08 | 实现产品质量分析（含无分母降级） | SKILL.md §5（质量引擎段） |
| TASK-ANL-09 | 实现流出矩阵（candidate/confirmed 分离） | SKILL.md §6（流出引擎段） |
| TASK-ANL-10 | 实现漏测 PPDCS 归类 | SKILL.md §7（漏测引擎段） |
| TASK-ANL-11 | 实现改进 CA/PA 候选草案 | SKILL.md §8（改进引擎段） |
| TASK-ANL-12 | 实现环比同比（窗口/N-A/可信度） | SKILL.md §9（比较引擎段） |
| TASK-ANL-13 | 实现 MetricDefinition 模板与管理 | `templates/metric-definition.yaml` |
| TASK-ANL-14 | 创建 ra-report 输出模板 | `templates/ra-report.yaml` |

### 执行顺序

```
TASK-ANL-13（MetricDefinition 模板）┐
                                  ├─► TASK-ANL-07 → TASK-ANL-08 → TASK-ANL-09 → TASK-ANL-10 → TASK-ANL-11 → TASK-ANL-12
TASK-ANL-14（ra-report 模板）─────┘                                                        │
                                                                                          ▼
                                                                             TASK-ANL-14（回填到 ra-report）
```

---

## 10. 回滚策略

| 回滚场景 | 操作 |
|---------|------|
| 六维引擎实现错误 | 回退 SKILL.md 到 ST-RA-01 后状态（保留资格检查段） |
| 模板 schema 变更 | 修改 templates/*.yaml；analysis_run 中引用 schema_version 以区分旧数据 |
| 需要完全移除某维度 | 删除 SKILL.md 对应段落 + ra-report.yaml 对应 section；不影响其他维度 |
| MetricDefinition 口径变更 | 修改 metric-definition.yaml + 递增 metric_version；触发下游全量重算 |

**关键约束**：MetricDefinition 的 `metric_version` 变更必须触发 S2 全量重算（在 ST-RA-06.2 中实现检测）。

---

## 11. 平台差异检查

| 平台 | 差异项 | 影响 | 处理 |
|------|--------|------|------|
| 全部 | ra-report.yaml 与 metric-definition.yaml 为 YAML 文件 | 无平台差异 | YAML 跨平台一致 |
| 全部 | 六维分析逻辑为 Skill 文本中的执行步骤描述 | 无平台差异 | AI 在各平台按相同步骤执行 |
| 全部 | 根因状态机为 Skill 内状态管理 | 无平台差异 | 不依赖运行时状态存储 |

**结论**：无平台差异，所有平台共享同一套 Skill 定义和模板文件。

---

## 12. 与相邻模块的集成契约

### 12.1 上游消费

| 上游 | 消费内容 | 方式 |
|------|---------|------|
| ST-RA-01（资格检查） | eligibility_result + evidence_lines[] | 同 Skill 内，ST-RA-01 段输出 → ST-RA-02 段输入 |
| ST-RA-INGEST-DB（SQLite） | ticket 表所有规范化列（只读） | `data/dao.py` 的只读查询接口 |

### 12.2 下游提供

| 下游 | 提供内容 | 传递方式 |
|------|---------|---------|
| ST-RA-05.3（S1 管线） | 完整的六维分析引擎 | 同 Skill 内直接调用 |
| ST-RA-06.2（S2 刷新） | 六维分析引擎 + MetricDefinition | 同 Skill 内复用引擎，新参数传入 |
| ST-RA-03（改进输入治理） | ra-report 中的 improvement_candidates | 文件传递（ra-report.yaml），只读消费 |

### 12.3 模板间契约

- `ra-report.yaml` 的 `sections.product_quality.metrics[]` 必须引用 `metric-definition.yaml` 中的 `metric_id`
- `metric_version` 变更时，`analysis_run` 的 `analysis_rule_version` 必须同步更新
- ra-report 中的 `improvement_candidates.capa_items[]` 不包含批准状态；批准状态由 ST-RA-03 追加

---

## 13. 开放项与假设

| ID | 类型 | 描述 | 状态 | 重访条件 |
|----|------|------|------|---------|
| OPEN-RA02-01 | decision | CA/PA 候选与 improvement-tracker 的精确交接格式 | OPEN（待 ST-RA-03 LLD 对齐） | CP5 统一确认前必须与 ST-RA-03 接口对齐 |
| OPEN-RA02-02 | decision | 环比同比窗口的最小样本量阈值 | OPEN | 实现前需确定阈值（建议 >= 10 个样本） |
| OPEN-RA02-03 | 假设 | 5 Whys 和鱼骨图方法通过 Skill 文本提示词引导 AI 执行，不需要独立代码实现 | OPEN | 若 CP7 发现 AI 分析质量不稳定，可能需要方法模板 |
| OPEN-RA02-04 | 假设 | PPDCS 归类映射表在 Skill 文本中以对照表形式维护 | OPEN | 若映射规则增多，可抽为独立 PPDCS 映射模板 |
| OPEN-RA02-05 | 假设 | 环比同比窗口按自然月/自然季度对齐 | OPEN | 若需要自定义窗口（如按发布日期），需扩展 |

---

## 14. LLD 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|------|------|--------|---------|
| 1.0 | 2026-07-16 | meta-dev | 初始 LLD，覆盖六维分析引擎、根因四层状态机、MetricDefinition 和 RA Report 模板设计 |
| 1.1 | 2026-07-16 | host-orchestrator | CP5 B12：迁移为当前 full-lld 证据兼容格式，增加 canonical Story ID 与 §0 工程依据/章节索引；不改变原设计契约。 |
