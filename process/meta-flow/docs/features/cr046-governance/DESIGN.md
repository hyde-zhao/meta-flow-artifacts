---
status: draft
version: "1.0"
feature_id: "FEAT-EI-GOVERNANCE"
related_stories: ["ST-EI-004"]
lld_policy_summary: "1 full-lld"
---

# Feature Design: Workspace and Lifecycle Governance

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-12 | meta-se | 冻结 RouteTruth、phase/gate、finalization、health/read-expansion 与 semantic compaction 设计。 |

## 目标与边界

本 Feature 让 workspace/state/doctor/CP 共用一个 RouteTruth resolver，并验证 delivered finalization、health/read-expansion refs 和 compact/restore 语义。它消费 CORE typed graph，不拥有 identity schema、audit 展示或 correction写入。

## 现有位置

| 路径 | 变更 |
|---|---|
| `meta_flow/workspace/routing.py` | real local-directory metadata、portable anchors、shared RouteTruth |
| `meta_flow/state/current.py`, `ledger_compaction.py` | finalization/semantic manifest/apply gate |
| `meta_flow/checks/state_transition.py`, `context_doctor.py` | phase/gate、health/read-expansion共享检查 |
| `meta_flow/context_pack/*`, `tests/test_workspace_routing.py`, `tests/test_state_transition.py` | authz与负例 |

## 数据与流程

`RouteMetadata` 至少含 schema/project/mode/path_format/anchored relative paths；非空 routing_ref必须可解析。RouteTruth状态仅为 valid-symlink、valid-local-directory、legacy-explicit、dangling、conflict。compaction先捕获 source hash、typed nodes/edges、terminal selections、correction/health refs，再 restore重建；digest全等才 apply。

| 失败 | 行为 | 回退 |
|---|---|---|
| dangling/conflict route | 所有消费者一致 BLOCKED | candidate不应用或 explicit legacy-null |
| phase work提前开 gate | reject future fact | 恢复 pending_gate=null |
| delivered残留 active refs | finalization FAIL | 清理后重新验证，不改历史 |
| unauthorized read expansion | BLOCKED | 补 authorization/ref或撤销读取 |
| semantic digest mismatch | 源 ledger不变，隔离 candidate | 修实现后重跑 |

## Story LLD 合同与 Gotchas

ST-EI-004 LLD必须给出字段级 RouteTruth、portable anchor规则、原子切换、compaction manifest/digest、shared文件 merge owner与完整 fixtures。local-directory compatibility不是 dangling ref豁免；display fallback identity不得进入 semantic digest。
