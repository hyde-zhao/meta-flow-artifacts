---
status: pending
version: "1.0"
scope: "CR-051 / ST-AW-003"
created_at: "2026-07-18T14:34:53Z"
created_by: "meta-qa-critical (qa-cao)"
---

# Fixes: CR-051 / ST-AW-003

| Fix ID | 来源 Finding | 修复 / 澄清方向 | 路由 | 影响文件 / 设计对象 | 复验 / 回归范围 | 状态 |
|---|---|---|---|---|---|---|
| FIX-AW003-001 | REV-AW003-001 / TST-AW003-001 | 修正 `execute_leg` 的 freshness 时钟采样顺序或显式 clock 契约，使正常 observer 的快照不会因在函数入口后生成而被误判 stale；仍必须拒绝真正超出允许偏差的未来/陈旧快照。同步更新已过时的模块 docstring。 | meta-dev | `meta_flow/workflow/artifact_leg_lifecycle.py`、`tests/test_artifact_leg_lifecycle.py`；不要求设计 delta，除非需改变 frozen freshness 语义 | 1) 新增 `now=None` source complete PASS；2) artifact fresh health + complete PASS；3) ordinary resume new attempt PASS；4) post-observe PASS；5) 真正 future skew/stale 仍 BLOCKED+runner=0；6) 现有 38 directed；7) 56 adjacent；8) Ruff/format/py_compile/diff/dangerous argv/import | pending |

## 回修验收标准

- 默认 API 路径不得要求调用方人工传入一个晚于 observation 的固定 `now` 才能运行。
- source 与 artifact 的合法 fresh observer 路径 runner 调用数符合计划，且 fresh post-OID proof 后才发布 PASS handle。
- authz/OID/health/drift 不满足时 runner 仍为 0；不得以放宽全部 future timestamp 校验规避缺陷。
- evidence-only retry、coordination-only abort、no-cross-leg-rollback 与 artifact main/control/sibling mutation=0 不得回归。
- 不执行真实 remote/worktree/ref/link mutation，不读取凭据，不授权 repository publication。

## 复验路由

回修完成后交还 `meta-qa-critical` 进行最小回归 + packet 全量 CP7 复验。当前 Story 保持 `ready-for-verification` / rework routing，不得标记 `verified`。
