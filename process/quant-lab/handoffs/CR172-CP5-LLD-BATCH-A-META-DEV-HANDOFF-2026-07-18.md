---
handoff_type: "stage-dispatch"
change_id: "CR-172"
stage: "story-planning-cp5-design-evidence"
canonical_role: "meta-dev"
reasoning_profile: "default"
status: "ready"
created_at: "2026-07-18T10:48:00+08:00"
context_ref: "process/context/CP5-CR172-PATH-I-LLD-CONTEXT.yaml"
batch_id: "CR172-CP5-LLD-BATCH-A"
story_ids:
  - "CR172-S01-action-authorization-eligibility-governance"
  - "CR172-S02-trial-return-artifact-native-hook"
---

# CR172 CP5 LLD Batch A — meta-dev Handoff

按 `lld-designer` Skill 和 context capsule，为 S01、S02 分别生成一份完整 full LLD。

## 唯一写入面

- `process/stories/STORY-CR172-S01-action-authorization-eligibility-governance-LLD.md`
- `process/stories/STORY-CR172-S02-trial-return-artifact-native-hook-LLD.md`
- `process/handoffs/CR172-CP5-LLD-BATCH-A-META-DEV-RETURN-SUMMARY.md`

不得修改 Story 卡片、Feature 文档、Development Plan、state、ledger、checkpoint、源码、测试或 fixture。

## 强制合同

- S01：六类 approval record `6/6` 独立；执行资格 DAG `5/5`；approval union=`0`；runtime 自身已批但同 scope `data_lake_read` 缺失/拒绝/过期时 runner/workspace/pointer=`0/0/0`。
- S02：唯一 hook 只能是 `engine/mature_multifactor_research.py::run_stage3_mature_multifactor_research`，在 `build_strategy_candidate(...)` 成功后、`_ProducerLineageTrial.finish(...)` 前；`build_portfolio_path`、`write_stage3_outputs`、lineage store hook=`0/0/0`。
- `trial_portfolio_return_series@v1` canonical payload 恰好两列 `timestamp,simple_return`；`return_basis` 在 manifest 显式写明；不得添加/别名化 `net_return/gross_return/nav`。
- 必须冻结 timestamp UTC/交易日编码、canonical serialization/hash domain、seal/selection 原子顺序、异常回滚、legacy-off 兼容和测试入口。
- 只做设计证据，不实现代码；六类真实动作 authorized/executed=`0/6`,`0/6`。

如出现第二 hook、公共 lineage schema 修改、v1 schema 扩张、真实 backend/运行授权或跨 Story 文件 owner 冲突，写入 LLD 第 12 节并返回 `NEEDS_DESIGN_CLARIFICATION`；不要自行扩大范围。

