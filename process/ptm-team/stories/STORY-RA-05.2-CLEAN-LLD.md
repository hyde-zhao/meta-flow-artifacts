---
story_id: "STORY-RA-05.2-CLEAN"
canonical_story_id: "ST-RA-05.2-CLEAN"
title: "字段映射、清洗与质量报告"
source_cr: "CR-030"
feature: "FEAT-RA-INGESTION"
lld_policy: "full-lld"
tier: "Standard"
design_evidence_type: "full-lld"
created_by: "meta-dev"
created_at: "2026-07-16"
open_items: []
shared_fragments:
  - "shared_skill: skills/itr-ticket-ingestion/SKILL.md (with ST-RA-05.1-INGEST, ST-NRA-03, ST-RA-06.1-DETECT)"
---

# ST-RA-05.2-CLEAN: 字段映射、清洗与质量报告 — LLD

## 0. 工程依据与模板索引

| 来源 | 消费内容 |
|---|---|
| HLD / Feature DESIGN / Feature Matrix | 字段映射、质量报告、敏感字段策略和 `full-lld` 约束 |
| Story `ST-RA-05.2-CLEAN` | 输出文件、验收标准和依赖 |

本文保留原有详细章节；目标、需求、模块拆分、代码结构、数据模型、API、流程、技术细节、安全、测试、实施、风险与 DoD 分别由后续编号章节定义。

> 对应 HLD REV-03 的清洗/质量标记阶段和 `IngestionQualityReport` 治理契约。
> 本 Story 在共享 SKILL.md 中追加 §清洗与质量报告部分。

## 1. 文件影响范围

| 文件 | 操作 | 所有权 | 说明 |
|---|---|---|---|
| `skills/itr-ticket-ingestion/SKILL.md` | 修改（共享写入） | F-020 | 追加 §字段映射、§清洗规则、§质量报告三节。ST-RA-05.1-INGEST 已创建骨架 |
| `skills/itr-ticket-ingestion/templates/quality-report.yaml` | 新建 | F-020 | IngestionQualityReport 模板 |
| `skills/itr-ticket-ingestion/templates/field-mapping.yaml` | 新建 | F-020 | ITR → ticket 字段映射配置 |

**不修改的文件**: `data/schema.sql`、`data/dao.py`（只消费其定义的 schema 字段名）。

## 2. 接口设计

### 2.1 清洗管线入口

```python
def clean_tickets(raw_tickets: list[dict], batch_id: str,
                  mapping_config: dict, quality_config: dict) -> CleanResult:
    """
    输入: ITR 原始 ticket 数组、batch_id、映射配置、质量阈值配置
    输出: CleanResult {cleaned, failed, quality_report}
    """
```

### 2.2 CleanResult 结构

```python
@dataclass
class CleanResult:
    cleaned: list[dict]        # 映射并清洗成功的 ticket 字典列表
    failed: list[FailedRecord] # 不可清洗的记录
    quality_report: dict       # IngestionQualityReport（见 §2.4）
    stats: CleaningStats       # 统计摘要

@dataclass
class CleaningStats:
    total_input: int
    total_cleaned: int
    total_failed: int
    null_count: int
    anomaly_count: int
    duplicate_count: int
    conflict_count: int
    unclassified_fields: int

@dataclass
class FailedRecord:
    index: int                 # 在原始数组中的位置
    source_ticket_id: str | None
    reason: str                # 失败原因
    fields: list[str]          # 缺失的必填字段
```

### 2.3 字段映射配置

```yaml
# skills/itr-ticket-ingestion/templates/field-mapping.yaml
schema_version: "1.0"
mappings:
  # source_field → target_field → type → required
  - source: "id"            # 假设的 ITR 字段名
    target: "source_ticket_id"
    type: "string"
    required: true

  - source: "product"
    target: "product"
    type: "string"
    required: true

  - source: "module"
    target: "module"
    type: "string"
    required: false

  - source: "severity"
    target: "severity"
    type: "enum"
    required: false
    valid_values: ["致命", "严重", "一般", "提示"]

  - source: "status"
    target: "status"
    type: "string"
    required: false

  - source: "title"
    target: "title"
    type: "string"
    required: false

  - source: "description"
    target: "description"
    type: "string"
    required: false

  - source: "root_cause"
    target: "root_cause"
    type: "string"
    required: false

  - source: "test_missed_analysis"
    target: "test_missed_analysis"
    type: "string"
    required: false

  - source: "test_missed_phase"
    target: "test_missed_phase"
    type: "string"
    required: false

  - source: "improvement_measures"
    target: "improvement_measures"
    type: "string"
    required: false

  - source: "openeddate"
    target: "openeddate"
    type: "string"
    required: false

  - source: "resolveddate"
    target: "resolveddate"
    type: "string"
    required: false

  - source: "source_updated_at"  # 或从响应元数据获取
    target: "source_updated_at"
    type: "datetime"
    required: true
```

### 2.3.1 severity 与 pri 字段标准化映射（CP5 Round 2）

> CP5 Round 2 评审：ITR 响应同时包含 `pri`（优先级）和 `severity`（严重度），QCOMB-02 的高严重度门控需要明确枚举来源和标准化规则。

**字段定义**：

| ITR 字段 | 语义 | 本系统 target 字段 | 标准化规则 |
|---|---|---|---|
| `pri` | ITR 问题优先级（如 P1, P2, P3, P4）| `ticket.priority` | 直接映射；不在枚举内 → `quality_flag=anomaly` |
| `severity` | ITR 问题严重度（如 致命, 严重, 一般, 提示）| `ticket.severity` | 直接映射；不在枚举内 → `quality_flag=anomaly` |

**pri ↔ severity 映射关系（用于 QCOMB 高严重度判定）**：

| pri 值 | severity 枚举 | 高严重度判定 | 说明 |
|---|---|---|---|
| `P1` | `致命` 或 `严重` | **是** | P1 问题总是高严重度，无论 severity 字段是否有值 |
| `P2` | `严重` 或 `一般` | **是**（如果 severity=严重）| P2 问题若 severity=致命 或严重，视为高严重度 |
| `P3` | `一般` | 否 | P3/P4 不做高严重度门控 |
| `P4` | `提示` | 否 | |

**QCOMB-02 的高严重度判定逻辑（修正后）**：

```python
def is_high_severity(ticket: dict) -> bool:
    """判定是否触发 QCOMB-02 高严重度质量门控"""
    pri = ticket.get('priority', '')
    severity = ticket.get('severity', '')

    # P1 总是高严重度
    if pri == 'P1':
        return True
    # P2 且 severity 为致命或严重
    if pri == 'P2' and severity in ('致命', '严重'):
        return True
    return False
```

**映射失败处理**：

| 场景 | 处理 | quality_flag |
|---|---|---|
| `pri` 不在 {P1, P2, P3, P4} | 记录原始值，标注 anomaly | `anomaly` |
| `severity` 不在 {致命, 严重, 一般, 提示} | 记录原始值，标注 anomaly | `anomaly` |
| `pri` 缺失 | 不影响清洗，但 QCOMB-02 不触发 | 不影响 quality_flag |
| `severity` 缺失 | 不影响清洗，但 QCOMB-02 仅基于 pri 判定 | 不影响 quality_flag |
| `pri` 和 `severity` 同时缺失 | 该记录不参与高严重度门控 | 不影响 quality_flag（除非其他字段异常） |

**验收标准（追加到 ST-RA-05.2-CLEAN AC）**：

- AC-6: P1 记录 `root_cause` 缺失 → QCOMB-01 判定为 blocked
- AC-7: `pri=P1` 的记录被 blocked → QCOMB-02 判定 overall_status=blocked
- AC-8: `pri` 值不在枚举 → quality_flag=anomaly
- AC-9: `severity` 值不在枚举 → quality_flag=anomaly

```yaml
# skills/itr-ticket-ingestion/templates/quality-report.yaml
batch_id: "string"
schema_version: "string"  # 映射配置版本号
generated_at: "ISO8601"

input_summary:
  total_fetched: 0
  http_status: 200
  response_hash: "sha256:..."

quality_summary:
  total_clean: 0
  total_flagged: 0
  total_failed: 0
  null_rate: 0.0          # 必填字段空值比例
  anomaly_rate: 0.0        # 异常值比例
  duplicate_rate: 0.0      # source_ticket_id 重复率
  conflict_count: 0         # 冲突数
  unclassified_fields: 0   # 未分类字段数

thresholds:
  null_rate_threshold: 0.3
  anomaly_rate_threshold: 0.1
  duplicate_rate_threshold: 0.5
  min_analyzable_ratio: 0.5
  max_conflict_count: 50

overall_status: "clean"   # clean | flagged | blocked
analyzable_ratio: 0.0     # total_clean / total_fetched

issues:
  - field: "string"
    type: "null|anomaly|duplicate|conflict|unclassified"
    count: 0
    description: "string"
    sample_ids: ["string"]  # 最多 5 条示例

is_analyzable: true       # analyzable_ratio >= min_analyzable_ratio
block_reason: ""          # 阻断时填写原因
```

### 2.5 质量阈值（本 LLD 版本默认值）

| 参数 | 默认值 | 可配置 | 说明 |
|---|---|---|---|
| `null_rate_threshold` | 0.30 (30%) | 是 | 必填字段空值占比上限 |
| `anomaly_rate_threshold` | 0.10 (10%) | 是 | 异常值占比上限 |
| `duplicate_rate_threshold` | 0.50 (50%) | 是 | source_ticket_id 重复率上限 |
| `min_analyzable_ratio` | 0.50 (50%) | 是 | 可分析记录占比下限 |
| `max_conflict_count` | 50 | 是 | 单批次冲突数上限 |

### 2.6 质量阈值组合风险规则（CP5 评审 H4）

> CP5 评审意见 H4：初版阈值 `null≤30%`、`duplicate≤50%`、`analyzable≥50%` 可同时通过，但仍可能以一半可分析数据形成报告。为避免低质量数据被误判为"可分析"，增加以下组合规则。

| 规则 ID | 规则 | 判定 | 优先级 | 说明 |
|---:|---|---|:---:|---|
| QCOMB-01 | **最小关键字段完整率**：高严重度记录（severity ∈ {致命, 严重}）的 `root_cause` 字段 null_rate > 40% | `overall_status = blocked` | 高于单字段阈值 | P1/P2 问题单若无根因字段，分析没有价值 |
| QCOMB-02 | **高严重度记录质量不得 flagged**：severity=致命 的记录中 quality_flag ∈ {blocked} 的数量 > 0 | `overall_status = blocked` | 高于 analyzable_ratio | 致命级问题单必须全部可分析，不允许任何一条被阻断 |
| QCOMB-03 | **双因子质量门控**：`analyzable_ratio < 0.5` AND `null_rate > 0.25` | `overall_status = blocked` | — | 当可分析比例低且空值率高时，无论单项阈值是否通过都阻断 |
| QCOMB-04 | **分析方法降级**：`analyzable_ratio < 0.7` | 报告标注 `analysis_confidence = degraded` | informational | 不足 70% 可分析时，趋势/同比的可信度降低，仅输出数量/占比 |
| QCOMB-05 | **批量冲突上限**：冲突数 > 20% 的总记录数 | `overall_status = blocked` | — | 冲突过多意味着 schema 或映射有系统性问题，不应硬合并 |

**实现整合**：

- 单字段阈值（QCOMB 之前）先判定各记录 `quality_flag`
- QCOMB-01~05 在批次级别汇总后判定 `overall_status`
- 最终 `overall_status` = `blocked` 如果任一组合规则触发
- QCOMB-04 不阻断，仅在报告中附加标注

**测试入口**：

| 测试 ID | 场景 | 预期 |
|---|---|---|
| T-CLEAN-QCOMB-01 | 致命级记录 root_cause 缺失 50% | overall_status=blocked |
| T-CLEAN-QCOMB-02 | 致命级记录被 blocked | overall_status=blocked |
| T-CLEAN-QCOMB-03 | analyzable=40% + null=30% | overall_status=blocked（双因子触发） |
| T-CLEAN-QCOMB-04 | analyzable=60% | 不阻断，但 report.analysis_confidence=degraded |
| T-CLEAN-QCOMB-05 | 冲突数占 25% | overall_status=blocked |

## 3. 数据模型

### 3.1 quality_flag 枚举与判定规则

| quality_flag | 判定条件 | 对分析的影响 |
|---|---|---|
| `clean` | 必填字段全部有效，无可疑值，无冲突 | 可直接分析 |
| `incomplete` | 必填字段有空值（非 source_ticket_id 和 product 的字段） | 该记录可分析但标注数据不完整 |
| `anomaly` | 字段值格式/范围异常（如 severity 不在枚举内） | 该记录可分析但标注异常字段 |
| `blocked` | source_ticket_id 或 product 缺失；或质量报告 overall_status=blocked | 不可分析，批次阻断 |

**判定优先级**: `blocked` > `anomaly` > `incomplete` > `clean`

### 3.2 敏感字段分类（raw/cleaned/report）

| 分类 | 定义 | 字段示例 | 存储策略 | 使用策略 |
|---|---|---|---|---|
| `raw` | 原始数据，不可直接暴露 | `raw_json`（完整原始响应） | 仅存储在 SQLite `ticket.raw_json` | 不进 LLM/报告正文 |
| `cleaned` | 映射清洗后，可安全分析 | `title`, `description`, `root_cause` | 存储在对应字段 | 可进入分析流程 |
| `report` | 聚合/脱敏后，可对外展示 | severity 分布、产品质量统计 | 存储在分析报告 | 可出现在报告中 |

### 3.3 未知字段保留策略

```python
def preserve_unknown_field(field_name: str, field_value: any,
                           raw_json: str) -> dict:
    """
    未在 field-mapping.yaml 中定义的字段：
    1. 不映射到 ticket 表字段
    2. 保留在 raw_json 中（raw_json 始终保存完整原始响应）
    3. 在 quality_report.issues 中追加 type='unclassified'
    4. 不静默丢弃
    """
```

## 4. 核心流程

### 4.1 清洗主流程

```
── 输入 ──
raw_tickets: list[dict]  (来自 ST-RA-05.1 的 HTTP 响应解析)
batch_id: str
          │
          ▼
┌─ 1. 加载映射配置 ──────────────────────────────────────┐
│  field_mapping = load_mapping("field-mapping.yaml")     │
│  版本号记录在 quality_report.schema_version             │
└────────────────────────────────────────────────────────┘
          │
          ▼
┌─ 2. 逐条清洗 ──────────────────────────────────────────┐
│  FOR each raw_ticket:                                   │
│    a. 映射字段: source → target（按 mapping）           │
│    b. 校验必填字段: source_ticket_id, product 不能为空   │
│       - 缺失 → FailedRecord('missing_required')        │
│    c. 校验枚举字段: severity 等检查 valid_values          │
│       - 不在枚举 → quality_flag='anomaly'               │
│    d. 检测空值: 非必填字段的空值记录                      │
│       - 存在 → quality_flag='incomplete'               │
│    e. 检测未知字段: 不在 mapping 中的字段                 │
│       - 保留在 raw_json，记录到 issues                   │
│    f. 检测重复: source_ticket_id 是否在本批次中出现多次   │
│       - 是 → duplicate_count++                         │
│    g. 设置时间戳: first_seen_at, last_seen_at,          │
│       source_updated_at                                 │
│    h. 追加 raw_json（完整原始记录）                      │
└────────────────────────────────────────────────────────┘
          │
          ▼
┌─ 3. 计算统计 ──────────────────────────────────────────┐
│  null_rate = 含空值条数 / total_fetched                  │
│  anomaly_rate = 异常值条数 / total_fetched               │
│  duplicate_rate = 重复条数 / total_fetched               │
│  analyzable_ratio = (total_clean + incomplete            │
│                       + anomaly) / total_fetched         │
└────────────────────────────────────────────────────────┘
          │
          ▼
┌─ 4. 阈值判断 ──────────────────────────────────────────┐
│  IF null_rate > null_threshold:                         │
│     OR anomaly_rate > anomaly_threshold:                │
│     OR duplicate_rate > duplicate_threshold:           │
│     OR analyzable_ratio < min_analyzable_ratio:         │
│     OR conflict_count > max_conflict_count:             │
│    → overall_status = 'blocked'                        │
│  ELIF any issue exists:                                │
│    → overall_status = 'flagged'                         │
│  ELSE:                                                 │
│    → overall_status = 'clean'                           │
└────────────────────────────────────────────────────────┘
          │
          ▼
┌─ 5. 生成质量报告 ──────────────────────────────────────┐
│  填充 quality-report.yaml 模板                           │
│  返回: CleanResult {cleaned, failed, quality_report}    │
└────────────────────────────────────────────────────────┘
```

### 4.2 敏感字段检查流程

```
function check_sensitive_fields(cleaned_record):
    FOR each field in cleaned_record:
        classification = CLASSIFICATION_DICT.get(field, 'unclassified')
        IF classification == 'unclassified':
            quality_report.issues.append({
                field, type='unclassified',
                description='字段未在分类词典中注册'
            })
            # 标记为 blocked 当且仅当 overall_status 条件满足
    # 未分类字段保留在 raw_json 中，不写入 ticket 的 cleaned 字段
```

### 4.3 quality_flag 设置流程

```
function set_quality_flag(record, issues):
    IF any(issue.type == 'blocked_critical' for issue in issues):
        return 'blocked'
    IF any(issue.type == 'anomaly' for issue in issues):
        return 'anomaly'
    IF any(issue.type == 'null_value' for issue in issues_on_required):
        IF affected field is source_ticket_id or product:
            return 'blocked'
        ELSE:
            return 'incomplete'
    IF any(issue.type == 'null_value' for issue in issues):
        return 'incomplete'
    RETURN 'clean'
```

## 5. 状态机

`IngestionQualityReport.overall_status` 三态转换：

```
       ┌─────────────────────────────────────┐
       │                                     │
       ▼                                     │
   ┌───────┐  all thresholds passed  ┌───────┴──┐
   │ clean  │◄────────────────────────│ pending   │
   └───┬───┘                         └─────┬─────┘
       │                                   │
       │ some issues present               │ threshold exceeded
       ▼                                   ▼
   ┌─────────┐                       ┌─────────┐
   │ flagged  │                       │ blocked  │
   └─────────┘                       └─────────┘
```

| 状态 | 含义 | 对 analysis_run 的影响 |
|---|---|---|
| `clean` | 所有指标通过阈值 | analysis_run 正常创建 |
| `flagged` | 有警告但未超过阻断阈值 | analysis_run 可创建，标注风险 |
| `blocked` | 任一项超过阈值 | **禁止创建 analysis_run** |

`flagged` 状态下，下游 `analysis_run` 创建时必须包含 `risk_warnings: [IngestionQualityReport flagged]`。

## 6. 错误处理与降级

| 错误场景 | 处理策略 | 降级行为 |
|---|---|---|
| 必填字段缺失（source_ticket_id） | 该条记录标记为 `FailedRecord`，不进入 cleaned 列表 | 该条记录不写入 ticket |
| 必填字段缺失（product） | 同 source_ticket_id | 同 |
| 非必填字段空值 | 字段值设为 None/空，quality_flag='incomplete' | 记录可继续分析 |
| 枚举字段值不在 valid_values 内 | 保留原值，quality_flag='anomaly' | 记录可继续分析 |
| 未知字段 | 保留在 raw_json 中，记录在 issues | 记录可继续分析 |
| source_ticket_id 批次内重复 | 保留第一条，后续记录 duplicate_count++ | 只保留第一条的 cleaned 版本 |
| 质量阈值全部超标 → blocked | 整个 batch 标记为 blocked | analysis_run 不可创建 |
| 质量阈值部分超标 → flagged | batch 标记为 flagged | analysis_run 可创建，标注风险 |

**降级原则**：
- 不静默丢弃记录（除非 source_ticket_id 或 product 缺失——无法建立唯一标识）
- 不静默删除字段（未知字段保留在 raw_json）
- blocked 状态永久化存储到 `IngestionQualityReport`，供审计追溯

## 7. 测试设计

### 7.1 Fixture 计划

| Fixture | 内容 | 覆盖场景 |
|---|---|---|
| `fixtures/itr_valid_response.json` | 5 条完全有效的 ITR ticket | T-CLN-01, T-CLN-12 |
| `fixtures/itr_partial_response.json` | 含空值/缺失非必填字段 | T-CLN-02, T-CLN-03 |
| `fixtures/itr_missing_required.json` | 缺少 source_ticket_id 或 product | T-CLN-04 |
| `fixtures/itr_unknown_fields.json` | 含映射配置中未定义的字段 | T-CLN-05 |
| `fixtures/itr_duplicates.json` | 含重复 source_ticket_id | T-CLN-06 |
| `fixtures/itr_anomaly_values.json` | severity 等枚举字段含非法值 | T-CLN-08 |
| `fixtures/itr_low_quality.json` | 大量空值/异常导致阈值超标 | T-CLN-09, T-CLN-10, T-CLN-11 |
| `fixtures/field-mapping-config.yaml` | 标准映射配置 | 全部测试 |

### 7.2 测试用例与覆盖矩阵

| ID | 场景 | 类型 | 输入 | 预期结果 | 对应 AC |
|---|---|---|---|---|---|
| T-CLN-01 | 有效数据全部清洗通过 | positive | 5 条有效 ticket | cleaned=5, quality_flag='clean', overall_status='clean' | AC-1, AC-2 |
| T-CLN-02 | 含空值非必填字段 | positive | title 为空 | quality_flag='incomplete', 记录不丢弃 | AC-2 |
| T-CLN-03 | 多字段空值 | boundary | 所有非必填字段为空 | quality_flag='incomplete', 记录仍在 cleaned 中 | AC-2 |
| T-CLN-04 | source_ticket_id 缺失 | negative | id 字段缺失 | FailedRecord, 不进入 cleaned | AC-1 |
| T-CLN-05 | 未知字段保留 | boundary | 含额外字段 `custom_field` | 未知字段记录在 issues，保留在 raw_json | AC-5 |
| T-CLN-06 | 批次内重复处理 | boundary | 2 条相同 source_ticket_id | duplicate_count=1，只保留 1 条 | AC-1 |
| T-CLN-07 | quality_report 字段完整 | positive | 正常清洗 | report 包含所有必需字段 | AC-3 |
| T-CLN-08 | 枚举字段异常值 | boundary | severity='unknown' | quality_flag='anomaly', 记录保留 | AC-2 |
| T-CLN-09 | analyzable_ratio 低于阈值 | boundary | 大量失败 → ratio=0.3 | overall_status='blocked' | AC-4 |
| T-CLN-10 | null_rate 超过阈值 | boundary | null_rate=0.45 | overall_status='blocked'（与 AC-4 等价） | AC-4 |
| T-CLN-11 | 所有阈值通过但有个别问题 | boundary | 少量空值但比率合格 | overall_status='flagged' | AC-4 |
| T-CLN-12 | 敏感字段不进输出 | security | 含未分类字段 | 不在 cleaned 字典中暴露（仅在 raw_json） | AC-5（隐式） |

### 7.3 测试执行方式

- 纯函数测试：`clean_tickets()` 不依赖外部 I/O
- 使用临时目录存放 fixture JSON/YAML
- 快照写入路径可 mock（`tmp_path`）

## 8. 安全与权限

| 安全规则 | HLD 来源 | 实现方式 | 验证方式 |
|---|---|---|---|
| 未分类字段不进 LLM/正文 | HLD §敏感字段策略 | `check_sensitive_fields()` 阻断未分类字段写入 cleaned dict | T-CLN-12 |
| 质量不达标不产出分析 | HLD `IngestionQualityReport` 契约 | `overall_status='blocked'` 时禁止 `analysis_run` 创建 | T-CLN-09, T-CLN-10 |
| 未知字段不静默丢弃 | FEAT-RA-INGESTION DESIGN.md §8 Gotchas | 追加到 `issues`，保留在 `raw_json` | T-CLN-05 |

## 9. 实施步骤

### 切片 1：字段映射配置与映射逻辑（TASK-CLEAN-01）
1. 创建 `field-mapping.yaml` 模板
2. 实现 `map_fields()`：source → target，处理枚举校验
3. 实现未知字段检测和保留
4. 测试：T-CLN-01, T-CLN-05, T-CLN-08

### 切片 2：空值/异常/重复检测（TASK-CLEAN-02）
1. 实现必填字段校验（source_ticket_id, product, source_updated_at）
2. 实现非必填字段空值标记
3. 实现批次内重复检测
4. 测试：T-CLN-02, T-CLN-03, T-CLN-04, T-CLN-06

### 切片 3：quality_flag 设定 + 统计计算（TASK-CLEAN-02 延续）
1. 实现 `set_quality_flag()` 判定逻辑
2. 实现 `compute_stats()` 统计
3. 测试：覆盖所有 quality_flag 路径

### 切片 4：IngestionQualityReport 生成 + 阈值判断（TASK-CLEAN-03, TASK-CLEAN-04）
1. 创建 `quality-report.yaml` 模板
2. 实现 `generate_quality_report()` 填充模板
3. 实现阈值判断逻辑（`null_rate`, `anomaly_rate`, `duplicate_rate`, `analyzable_ratio`, `conflict_count`）
4. 测试：T-CLN-07, T-CLN-09, T-CLN-10, T-CLN-11

### 切片 5：敏感字段分类（TASK-CLEAN-05）
1. 实现 `check_sensitive_fields()` 分类检查
2. 实现分类词典（raw/cleaned/report 三级）
3. 测试：T-CLN-12

## 10. 回滚策略

| 变更类型 | 回滚方式 |
|---|---|
| 字段映射错误 | 修改 `field-mapping.yaml` → 重新清洗已保存的 raw_json |
| 质量阈值不当 | 修改 `quality-report.yaml` 模板或阈值配置 → 重新评估已有 batch |
| quality_flag 判定逻辑错误 | 修复代码 → 通过 `raw_json` 重跑清洗（数据可恢复） |
| 敏感字段分类遗漏 | 追加分类 → 重新生成质量报告 |

**关键回滚保障**: `raw_json` 始终保存完整原始响应，清洗结果可随时通过修正后的映射/清洗规则重新生成。

## 11. 平台差异检查

| 检查项 | Codex | Claude Code | Qoder | 说明 |
|---|---|---|---|---|
| YAML 解析 | PyYAML / ruamel.yaml | 同左 | 同左 | 无差异 |
| JSON 序列化 | `json` 标准库 | 同左 | 同左 | 无差异 |
| Python 字符串处理 | 标准库 | 同左 | 同左 | 无差异 |
| 正则表达式 | `re` 标准库 | 同左 | 同左 | 无差异 |

**结论**: 无平台差异处理项。

## 12. 与相邻模块的集成契约

### 12.1 对 ST-RA-05.1-INGEST（上游）的输入契约

| 输入 | 格式 | 要求 |
|---|---|---|
| `raw_tickets` | `list[dict]` | ITR 响应体解析后的 ticket 数组 |
| `batch_id` | `str` | 格式 `batch-{YYYYMMDD}-{seq}` |

### 12.2 对 ST-RA-INGEST-DB（下游写入）的交付契约

| 数据 | 格式 | DAO 接口 |
|---|---|---|
| `cleaned tickets` | `list[dict]`（ticket 表字段已映射） | DAO 的 `insert_ticket()` / `upsert_ticket()` |
| `quality_report` | `dict`（IngestionQualityReport 结构） | 保存为 YAML，ref 写入 `ingestion_batch.quality_report_ref` |
| `batch_summary` | `dict`（total_fetched, total_cleaned, total_failed） | 写入 `ingestion_batch` 表对应字段 |

### 12.3 对 FEAT-RA-ANALYSIS（下游消费）的承诺

| 承诺 | 实现 |
|---|---|
| quality_flag 可靠 | 每条 ticket 有 quality_flag，非 clean 记录明确标注原因 |
| overall_status 阻断 | blocked → analysis_run 不可创建 |
| 可追溯 | quality_report_ref 链接到具体 batch |
| 敏感字段已分类 | cleaned 字段不包含未分类数据 |

## 13. 开放项与假设

### 开放项

| ID | 描述 | 状态 | 影响 |
|---|---|---|---|
| O-CLN-01 | ITR 字段的实际名称可能与假设不同 | OPEN（需首次受控探测） | `field-mapping.yaml` 的 source 字段名需根据真实响应调整 |
| O-CLN-02 | 敏感字段分类词典的完整列表 | OPEN | `check_sensitive_fields()` 的分类逻辑 |

### 假设

| ID | 假设 | 依据 | 风险 |
|---|---|---|---|
| A-CLN-01 | ITR 响应的 source_ticket_id 字段名为 `id` | 典型工单 API 模式 | 中（需首次探测确认） |
| A-CLN-02 | severity 枚举值为 ["致命", "严重", "一般", "提示"] | 中文 ITR 系统常见分类 | 中 |
| A-CLN-03 | 清洗是可重复的确定性操作（相同输入 → 相同输出） | 纯函数设计 | 低 |
| A-CLN-04 | 质量阈值默认值适用于首次使用场景 | FEAT-RA-INGESTION DESIGN.md §6.3 | 低（可配置） |

## 14. LLD 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-16 | meta-dev | 初始 LLD：字段映射 + 清洗 + quality_report + 14 章节 |
| 1.1 | 2026-07-16 | host-orchestrator | CP5 评审 H4 整改：新增 §2.6 质量阈值组合风险规则（QCOMB-01~05）。CP5 Round 2 整改：新增 §2.3.1 severity/pri 标准化映射规则 + QCOMB-02 高严重度判定逻辑 + 映射失败处理 + AC-6~9。 |
| 1.2 | 2026-07-16 | host-orchestrator | CP5 B12：迁移为当前 full-lld 证据兼容格式，增加 canonical Story ID 与 §0 工程依据/章节索引；不改变原设计契约。 |
