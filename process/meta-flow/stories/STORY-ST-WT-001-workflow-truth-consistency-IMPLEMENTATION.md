---
story_id: "ST-WT-001"
cr_id: "CR-047"
stage: "CP6"
status: "implemented"
execution_mode: "inline-fallback"
implemented_at: "2026-07-15"
---

# ST-WT-001 Implementation

## 实现对象

- `meta_flow/state/current.py`：新增只读 `validate_current_projection`，比较 State v2 与 CURRENT 的 change/story/gate/ref/status/health 投影。
- `meta_flow/checks/cr_tracking.py`：优先读取 `STATE.current.json`；active CR 必须存在于 JSON index 且非终态；串联 CURRENT 校验。
- `meta_flow/workflow/cr_lifecycle.py`：JSON index rebuild 保留尚无正式 CR 文件的 candidate，正式 CR 仍从 source-owned Markdown 重建。
- `process/changes/CR-INDEX.json`：登记 CR-033 candidate；删除 legacy `CR-INDEX.yaml`。
- `tests/test_cr047_truth_consistency.py`、`tests/test_cr_lifecycle.py`：覆盖 closed CR、缺 index、CURRENT drift、candidate 保留。

## 契约映射

State v2 是机器状态 owner，CR index 是生命周期 catalog，CURRENT 只做文件发现投影；新增检查均为 read-only relation validation，不创建第四套 truth，也不从 CURRENT 反写 State。

## 验证与边界

定向 truth/lifecycle 回归通过；真实仓运行 `meta-flow check cr-tracking` 返回 active=`CR-047`、candidate=`CR-033`、legacy YAML=0。未触碰 CR-046 原件、quant-lab、备份、凭据或外部 runtime。

## 设计差异与交接

无长期设计差异；实现严格采用已批准 source-owned truth graph。inline fallback 由用户授权，不构成独立 meta-dev 平台证明。
