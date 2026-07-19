---
status: draft-for-cp4
version: "1.0"
feature_id: "FEAT-AW-01"
related_story: "ST-AW-001"
---

# FEAT-AW-01 Feature Tasks

| TASK-ID | 顺序 | 任务 | 输入 | 输出文件 | 文件所有权 | 验证入口 | 状态 |
|---|---:|---|---|---|---|---|---|
| TASK-AW-R01 | 1 | 冻结 `PathRef`、`ProjectArtifactConfig`、`RouteDecision` 与稳定错误码 | DESIGN、ADR-AW-002 | `meta_flow/workspace/project_artifact_routing.py` | primary | schema/path unit cases | pending |
| TASK-AW-R02 | 2 | 实现 anchor-relative 解析、layout dispatch、read order 与唯一 write target | TASK-AW-R01 | 同上 | primary | TC-AW-001..003 | pending |
| TASK-AW-R03 | 3 | 实现 project/namespace/owned-target proof 与零 sibling 内容读取 | TASK-AW-R02 | 同上 | primary | TC-AW-010 negative | pending |
| TASK-AW-R04 | 4 | 为现有 process route 增加单向 compatibility adapter，不引入 lifecycle/Git 反向依赖 | TASK-AW-R02 | `meta_flow/workspace/routing.py` | shared；需单写窗口 | existing routing regression | pending |
| TASK-AW-R05 | 5 | 建立 project-first/legacy/ambiguity/traversal/relocation/determinism fixtures | TASK-AW-R01..04 | `tests/test_cr051_project_artifact_routing.py`、必要时 `tests/test_workspace_routing.py` | primary；旧测试 shared | TEST-PLAN 全集 | pending |
| TASK-AW-R06 | 6 | 执行 Ruff、定向测试、touched-path 与禁止依赖审计 | TASK-AW-R05 | evidence only | none | `uv run` commands + import scan | pending |

## 依赖与文件冲突说明

- `TASK-AW-R01 → R02 → R03` 串行；`R04` 在 R02 后可与 R03 并行，但 `routing.py` 必须由本 Story 单写。
- FEAT-AW-02 只 import `project_artifact_routing.py` 的稳定 value objects/API，不得修改本 Feature 主文件；若接口需变更，先回写 design delta。
- `meta_flow/cli.py` 不在本 Feature 文件范围内；统一 CLI facade 由 core Development Plan 指定单一 owner，避免与 worktree lane 冲突。

## 阻塞项

| Blocker ID | 影响 TASK | 问题 | 需要谁决策 | 推荐处理 |
|---|---|---|---|---|
| N/A | N/A | 无新阻塞决策；CP3 DQ 已批准 | N/A | 按 CP5 设计证据确认后实现 |
