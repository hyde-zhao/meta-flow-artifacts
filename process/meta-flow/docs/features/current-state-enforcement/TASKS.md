---
status: draft
version: "1.0"
feature_id: "FEAT-PG-001"
---

# Current State Enforcement Tasks

| TASK-ID | 顺序 | 任务 | 输入 | 输出文件 | 文件所有权 | 验证入口 | 状态 |
|---|---:|---|---|---|---|---|---|
| T-CS-001 | 1 | 定义 current state allowlist、optional keys 和 field budget 表 | HLD ADR-PG-001、DESIGN.md | `meta_flow/state/current.py` | primary | unit tests | pending |
| T-CS-002 | 2 | 实现 `validate_current_state()` 并接入 `write_current_state()` | T-CS-001 | `meta_flow/state/current.py` | primary | pytest | pending |
| T-CS-003 | 3 | 新增 `update_current_state()` deep-merge API | T-CS-002 | `meta_flow/state/current.py` | primary | pytest | pending |
| T-CS-004 | 4 | 将 CR lifecycle active change 更新收敛到受控 API | T-CS-003 | `meta_flow/workflow/cr_lifecycle.py` | shared | integration test | pending |
| T-CS-005 | 5 | 扩展 `meta-flow state check` 为 allowlist + disallowed + budget 三合一 | T-CS-002 | `meta_flow/state/current.py` / `meta_flow/cli.py` | shared | CLI test | pending |
| T-CS-006 | 6 | 同步 agent 写契约与 state-router 文档 | T-CS-003 | `delivery/rules/AGENT-SKILL-CONTRACT.md`、state-router skill | shared | rg guardrail | pending |
| T-CS-007 | 7 | 增加 unknown field、budget、internal writer 回归测试 | T-CS-001..006 | `tests/test_state_v2.py` 或新测试文件 | primary | pytest | pending |

## 阻塞项

| Blocker ID | 影响 TASK | 问题 | 需要谁决策 | 推荐处理 |
|---|---|---|---|---|
| BLK-CS-001 | T-CS-001 | audit 阶段发现真实合法 unknown 字段 | host-orchestrator / user | 记录字段来源，决定加入 explicit optional key 或迁移到 ledger / project state |
| BLK-CS-002 | T-CS-003 | deep-merge 删除语义未定 | meta-se / host-orchestrator | LLD 中明确禁止 patch 删除，删除需专门 API |
