---
story_id: "ST-WT-003"
cr_id: "CR-047"
stage: "CP6"
status: "implemented_with_risk"
execution_mode: "inline-fallback"
implemented_at: "2026-07-15"
---

# ST-WT-003 Implementation

## 实现对象

- `meta_flow/checks/token_budget.py`：超预算对象按 lifecycle/read class 分类；active/default-required 或未分类为 BLOCKER，closed/reference-only 为带 remediation ref 的 WARN。
- `meta_flow/checks/quality_governance.py`：消费 append-only legacy correction ledger；历史授权证据不可恢复时保持 `legacy/unavailable`，不宣称 PASS。
- `meta_flow/workflow/cr_lifecycle.py`：summary JSON 使用 compact serialization，避免新 summary 自身无意义越界。
- `process/policies/QUALITY-MODEL.yaml`：修正 read-expansion ledger 路径并登记 correction source。
- `process/corrections/CR047-LEGACY-READ-EXPANSION.ndjson`：对 6 个历史 CP 追加带原 hash 的 provenance correction。
- `tests/test_cr047_quality_governance.py`：覆盖 blocker/warning、路径与 correction 降级语义。

## 契约映射

Doctor 只分类/报告，不改写历史结果。`B0_pre=21` 是历史回归锚；CP7 动态记录 `B0_cp7` 并解释 delta。warning 不等于无风险，active/default-required 超预算始终阻断。

## 验证与边界

当前 Doctor：Token OK、Artifact `OK_WITH_WARNINGS`、Quality OK、Workflow OK；历史超限均已分类且 blocker/unclassified=0。原 CP result/hash 未改写，无法恢复的授权仍显式 unavailable。

## 设计差异与交接

无设计差异；CP7 必须重新采样 B0_cp7，不能固化当前 observed 数。
