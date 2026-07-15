---
status: ready-for-review
version: "1.0"
feature_id: "FEAT-WT-02"
---

# Artifact Routing Tasks

| TASK-ID | 顺序 | 任务 | 输入 | 输出文件 | Owner | 验证 |
|---|---:|---|---|---|---|---|
| TASK-WT-002-01 | 1 | 强化 portable routing/metadata contract | ADR-WT-002 | `meta_flow/workspace/routing.py`, `meta_flow/cli.py` | ST-WT-002 primary/shared | TP-R-01/02/04 |
| TASK-WT-002-02 | 2 | 增加 canonical docs copy 与 clean-clone fixtures | REQ-WT-005 | `tests/test_cr047_artifact_routing.py`, existing routing tests | ST-WT-002 primary | TP-R-01..03 |
| TASK-WT-002-03 | 3 | 同步目录契约说明 | verified behavior | `delivery/rules/DIRECTORY-CONTRACT.md`, `.yaml` | ST-WT-002 primary | contract diff review |

## 阻塞项

不得读取、移动或删除 prelink backup；遇到 regular path 只报告并停止。
