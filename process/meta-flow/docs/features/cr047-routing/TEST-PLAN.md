---
status: ready-for-review
version: "1.0"
feature_id: "FEAT-WT-02"
---

# Artifact Routing Test Plan

## 测试范围

| Scope ID | 覆盖内容 | 来源 | 层级 | 状态 |
|---|---|---|---|---|
| TP-R-01 | clean source/artifact clone 一次 link/check | ST-WT-002 / TC-WT-002 | integration | planned |
| TP-R-02 | relative anchors 与 no-device-absolute metadata | REQ-WT-004 | contract | planned |
| TP-R-03 | internal docs writable copy count=1 | REQ-WT-005 | filesystem | planned |
| TP-R-04 | regular path、断链、project mismatch | routing failure | negative | planned |

## 风险驱动与失败路径

| Case | 触发 | 期望 | 入口 |
|---|---|---|---|
| R-NEG-01 | artifact repo 根执行 link 且 `process/` 已存在 | 明确 regular-path 错误，不移动数据 | `tests/test_cr047_artifact_routing.py` |
| R-NEG-02 | metadata 含设备绝对路径 | BLOCKED | 同上 |
| R-NEG-03 | 根内部 docs 出现第二 writable copy | 非零 | 同上 |
| R-POS-01 | source root link 到 `process/meta-flow` | `process_link_health=ok` | `tests/test_workspace_routing.py` |

## 手工验收

| Check | 操作 | 期望 | 责任方 |
|---|---|---|---|
| MAN-R-01 | 审查 persisted metadata | 仅 anchor + relative path | CP7 reviewer |
