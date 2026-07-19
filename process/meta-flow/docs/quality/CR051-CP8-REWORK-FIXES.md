---
change_id: CR-051
report_type: fixes
revision: R2
status: completed
date: 2026-07-19
---

# CR-051 CP8 终验回修摘要

## 已完成

1. 合并 ST-AW-002/003/004 三份 design delta，并同步 3 组 Feature DESIGN/TEST-PLAN/TASKS 与 3 份 LLD。
2. ST-AW-002 增加 typed CapacityProof、owner.json、calibration/revocation persistence、显式 phase、fresh before/after 与 terminal 幂等；resume 正式只读/manual-only。
3. ST-AW-003 增加共享 canonical policy、integration containment 与 exact expected-OID scoped CAS cleanup；补齐 4 个高优测试。
4. ST-AW-004 增加 consumer target policy、canonical CLI mode、PARTIAL staged projection、retry route、dependency/DAG 测试。
5. 全仓 697 tests + 70 subtests、Ruff、scoped format、compile、guardrail、diff、delta/LLD checks 全部通过。

## 不在本次修复范围

- 不执行真实 remote/worktree/ref/link/目录迁移、commit/push/publish 或 main↔integration sync。
- 不借本轮机械格式化 92 个历史文件。
- 不把 Host inline-fallback 描述为独立 QA；该限制进入 CP8 风险接受。
