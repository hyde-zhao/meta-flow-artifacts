---
checkpoint_id: "CP4-CR030-STORY-DAG"
status: "auto-check"
created_by: "meta-se"
created_at: "2026-07-16"
updated_at: "2026-07-16"
source_cr: "CR-030"
source_hld: "docs/design/HLD.md (v1.2, REV-03)"
source_plan: "process/DEVELOPMENT-PLAN.yaml"
---

# CP4 自动预检：Story DAG 并行安全性

## Entry Criteria

- [x] CP3 HLD confirmed (confirmed_at: 2026-07-16)
- [x] DEVELOPMENT-PLAN.yaml 已生成
- [x] FEATURE-DESIGN-MATRIX.md 已生成
- [x] Feature 设计文件已生成（4 个 Feature × 3 文件 = 12 个）
- [x] Story 卡片已生成（14 张）

## Checklist

### 1. Story 完整性检查

| 检查项 | 结果 | 说明 |
|---|---|---|
| Story 总数 | PASS (14) | 10 full-lld + 4 technical-note |
| Wave 分布 | PASS | W1: 4, W2: 6, W3: 3, W4: 1 |
| 原始 Story 追溯 | PASS | 原始 9 个 Story 全部映射到实现 Story |
| lld_policy 分配 | PASS | 10 full-lld, 4 technical-note, 0 waived |
| feature_design_refs 完整 | PASS | 所有 Story 有 feature_design_refs |

### 2. 依赖完整性检查

| 检查项 | 结果 | 说明 |
|---|---|---|
| depends_on 指向存在 Story | PASS | 所有依赖的目标 Story 在计划中 |
| Wave 间依赖正确 | PASS | W2 depends W1, W3 depends W2 |
| 无循环依赖 | PASS | 单向依赖，无环 |
| 软依赖合理 | PASS | 1 个软依赖（02→01）；05.1→INGEST-DB 已升级为硬依赖以保障受限存储单写 |

### 3. 文件所有权检查

| 检查项 | 结果 | 说明 |
|---|---|---|
| 所有 Story 有 output_files | PASS | 14/14 |
| 共享文件无并行冲突 | PASS | 三个共享文件组内 Story 串行： |
|   - itr-ticket-ingestion/SKILL.md | PASS | 05.1 → 05.2 → NRA-03 → 06.1 (串行) |
|   - reverse-analysis/SKILL.md | PASS | 01 → 02 → 05.3 → NRA-01 → NRA-02 → 06.2 (串行) |
|   - improvement-tracker/SKILL.md | PASS | 03 → 04 → 06.3 (串行) |
| 独占文件无冲突 | PASS | INGEST-DB 独占 data/* 文件 |
| 跨 Wave 共享文件串行 | PASS | 06.1 在 W2，但 itr-ticket-ingestion 串行链已覆盖 |

### 4. 并行策略检查

| 检查项 | 结果 | 说明 |
|---|---|---|
| Wave 内并行可行性 | PASS | W1: INGEST-DB → 05.1 → 05.2 → NRA-03（受限存储与共享 Skill 顺序串行） |
|   | PASS | W2: 01→02→05.3 串行；06.1 可与 01-05.3 并行（不同文件） |
|   | PASS | W3: 03→04→06.3 串行；06.2 可与 03 并行（不同文件） |
| max_parallel_lld=3 | PASS | W2 最大并行度为 2，W3 最大并行度为 2 |
| max_parallel_dev=2 | PASS | 所有 Wave 内最大同时写文件 Story 数为 2 |

### 5. Feature 设计完整性检查

| Feature | DESIGN.md | TEST-PLAN.md | TASKS.md | 状态 |
|---|---|---|---|---|
| FEAT-RA-INGESTION | ✅ | ✅ | ✅ | PASS |
| FEAT-RA-ANALYSIS | ✅ | ✅ | ✅ | PASS |
| FEAT-RA-IMPROVEMENT | ✅ | ✅ | ✅ | PASS |
| FEAT-RA-TRACKING | ✅ | ✅ | ✅ | PASS |

### 6. 架构一致性检查

| 检查项 | 结果 | 说明 |
|---|---|---|
| HLD 四边界映射正确 | PASS | INGESTION/ANALYSIS/IMPROVEMENT/TRACKING |
| 单写规则遵守 | PASS | 每个 Feature 明确数据 owner |
| deny-by-default 覆盖 | PASS | ST-NRA-02、各 Skill 安全约束覆盖 |
| SQLite 唯一物理 owner | PASS | F-020 唯一写入 DDL/DAO；F-021/F-023 仅调用已定义公共接口，不直接 SQL |
| 信任治理契约覆盖 | PASS | 7 项契约在 Feature 设计中逐项落地 |

### 7. 风险检查

| 风险 | 状态 | 说明 |
|---|---|---|
| improvement-tracker 共享文件 | ACCEPTED | W3 内 IMPROVEMENT→TRACKING 串行，无冲突 |
| 负向 Story technical-note | ACCEPTED | 4 个负向 Story 使用 technical-note，正向 LLD 覆盖 |
| S2 全量重算触发 | MANAGED | ST-RA-06.2 包含触发条件检测 |

## Exit Criteria

- [x] Story 数量与 Wave 划分与 HLD §16（分阶段落地建议）一致
- [x] 依赖图中无循环依赖
- [x] 并行 Story 间无文件写入冲突
- [x] 所有 Story 有明确的 lld_policy 和 feature_design_refs
- [x] Feature 设计文件覆盖所有 4 个 required Feature

## 结论

**PASS（Round 4 复核）** — 14 个 Story 的依赖关系无环，文件所有权无并行冲突。`data/.gitignore` 由 ST-RA-INGEST-DB 单写，05.1 在其后消费受限存储接口；Wave 划分为 4 个 Wave。

### 待 CP5 确认的开放项

1. **CP4-DQ-CR030-01** (architecture): ST-RA-05/06 分解为 7 个子 Story 是否合理？→ CP5 决策
2. **CP4-DQ-CR030-02** (architecture): improvement-tracker 共享 Skill 是否合理？→ CP5 决策
3. **CP4-DQ-CR030-03** (implementation): SQLite 文件路径 → 默认 `data/ptm-tse.db`
4. **CP4-DQ-CR030-04** (risk_acceptance): 负向 Story technical-note 风险接受

### 建议给 CP5 Decision Brief 的输入

- 全量设计证据需覆盖 10 个 full-lld Story
- 4 个 technical-note Story 需在卡片内补充完整技术说明
- 三个 Skill 文件按串行组写入：itr-ticket-ingestion (4 stories)、reverse-analysis (6 stories)、improvement-tracker (3 stories)
