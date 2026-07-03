---
status: draft
version: "1.0"
feature_id: "FEAT-PG-002"
---

# Ledger Compaction Tasks

| TASK-ID | 顺序 | 任务 | 输入 | 输出文件 | 文件所有权 | 验证入口 | 状态 |
|---|---:|---|---|---|---|---|---|
| T-LC-001 | 1 | 定义 ledger retention policy schema 和默认配置 | DESIGN.md | `process/policies/LEDGER-RETENTION.yaml` 或模板 | primary | policy check | pending |
| T-LC-002 | 2 | 实现 dry-run compact planner | T-LC-001 | `meta_flow/state/event_ledger.py` 或新模块 | primary | pytest | pending |
| T-LC-003 | 3 | 实现 archive summary / archive index 生成 | T-LC-002 | `process/archive/ledger/**` runtime output | primary | integration | pending |
| T-LC-004 | 4 | 实现 apply / backup / rollback 安全写入 | T-LC-003 | ledger compact module | primary | integration | pending |
| T-LC-005 | 5 | 接入 `meta-flow ledger compact` CLI 并澄清 `state compact` help | T-LC-002 | `meta_flow/cli.py` | shared | CLI test | pending |
| T-LC-006 | 6 | 更新 event checker 兼容 compact marker 或 archive refs | T-LC-003 | `meta_flow/state/event_ledger.py` | shared | event check | pending |
| T-LC-007 | 7 | 增加 TC-PG-017 回归测试 | T-LC-005 | `tests/` | primary | pytest / CLI | pending |

## 阻塞项

| Blocker ID | 影响 TASK | 问题 | 需要谁决策 | 推荐处理 |
|---|---|---|---|---|
| BLK-LC-001 | T-LC-001 | 具体 retention 默认值尚未冻结 | host-orchestrator / user | LLD 中给出默认值和切换条件，CP5 前确认 |
| BLK-LC-002 | T-LC-005 | 命令名仍有 `ledger compact` / `event compact` 备选 | user / host-orchestrator | 采用 CP3 基线：默认 `meta-flow ledger compact`，如 CP5 修改再同步 |
