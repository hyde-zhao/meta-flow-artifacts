---
status: draft-for-cp5
version: "1.0"
feature_id: "FEAT-20"
---

# Feature Tasks: Experiment-family Lineage Core

| TASK-ID | 顺序 | Story | 任务 | 输入 | 输出文件 | Ownership | 验证入口 | 状态 |
|---|---:|---|---|---|---|---|---|---|
| TASK-CR163-20-01 | 1 | S01 | 冻结六对象、enums、commands、error codes | HLD/ADR | S01 LLD | design | CP5 LLD check | pending |
| TASK-CR163-20-02 | 2 | S01 | 规划 public core/validator/projection module | S01 LLD | `engine/experiment_family_lineage.py` | primary S01 | TP20-01/02/05 | pending |
| TASK-CR163-20-03 | 3 | S02 | 冻结 JSON/JSONL layout、canonical bytes/hash vectors | S01 contract | S02 LLD | design | CP5 LLD check | pending |
| TASK-CR163-20-04 | 4 | S02 | 规划 recorder/sealer/resolver module | S02 LLD | `engine/experiment_family_lineage_store.py` | primary S02 | TP20-03/04 | pending |
| TASK-CR163-20-05 | 5 | S05 | 汇总 core integrity/security fixtures | S01/S02 implementation evidence | core fixture tests | primary S05 | TP20 all | pending |

## 阻塞项

无。实现仍被 CP5 human gate 阻断；本任务清单不授权代码或测试实现。

