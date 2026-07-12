---
status: draft
version: "1.0"
feature_id: "FEAT-EI-CORE"
---

# Feature Tasks: Evidence Integrity Core

| TASK-ID | 顺序 | Story | 任务 | 主要输出范围 | 验证入口 | 状态 |
|---|---:|---|---|---|---|---|
| TASK-EI-001-01 | 1 | ST-EI-001 | 冻结 chronology/conditional gate规则与 fixtures | checks/state-transition/tests | CT-CORE-01 | pending |
| TASK-EI-002-01 | 2 | ST-EI-002 | 实现 typed dispatch/attempt/receipt/thread schema adapter | state/checks/tests | CT-CORE-02..05 | pending |
| TASK-EI-002-02 | 3 | ST-EI-002 | 实现 D0 freshness/re-probe 与 config hash invalidation | platform adapter/tests | CT-CORE-03/08 | pending |
| TASK-EI-002-03 | 4 | ST-EI-002 | 实现 reuse admission、no-receipt拒绝与 new-spawn lineage | platform adapter/tests | CT-CORE-05/09 | pending |
| TASK-EI-003-01 | 5 | ST-EI-003 | 实现 CP final attempt/hash/cross-truth correlator | cp_result/tests | CT-CORE-06 | pending |
| TASK-EI-CORE-INT | 6 | ST-EI-003 | 集成共享 enums/findings 并验证无第二真相源 | all core modules | core regression | pending |

阻塞项：真实 Codex runtime conformance 是外部能力；不阻塞 A-baseline repository implementation，但阻止 platform-attested 声明。
