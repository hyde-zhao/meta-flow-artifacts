---
status: draft-for-cp4
version: "1.1"
feature_id: "FEAT-AW-01"
related_story: "ST-AW-001"
---

# FEAT-AW-01 Feature Test Plan

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-18 | meta-se-critical | 建立 project-first/legacy、ambiguity、ownership、relocation 与零副作用测试契约。 |
| 1.1 | 2026-07-18 | meta-dev | CP5 R2 增加 anchor enum/父子矩阵、无环校验、absolute/dot/traversal/escape 与 anchor-only portable digest 用例，关闭 `CP5-QA-R1-F01`。 |

## 测试范围

| Scope ID | 覆盖内容 | 来源 | 测试层级 | 自动化状态 |
|---|---|---|---|---|
| TP-AW-R01 | project-first 唯一 docs/process target、无 sibling target | TC-AW-001；REQ-AW-001/003/NF002 | unit + fixture | planned |
| TP-AW-R02 | explicit legacy dual-read 与唯一 write target | TC-AW-002；REQ-AW-002/003 | fixture | planned |
| TP-AW-R03 | 无 authoritative version 的双候选 fail closed | TC-AW-003；REQ-AW-002/NF001 | negative fixture | planned |
| TP-AW-R04 | project/path/branch/owned namespace mismatch | TC-AW-010 | negative fixture | planned |
| TP-AW-R05 | route dry-run/重复/relocation 确定且零副作用 | TC-AW-012；NF001..002 | contract | planned |
| TP-AW-R06 | `project_worktree` anchor 可构造；unknown/wrong-parent/cycle/escape fail closed | CP5-QA-R1-F01；Design Review 1/3/5 | unit + negative fixture | planned |

## 参数与边界矩阵

| Case ID | 输入 | 期望 |
|---|---|---|
| R-TC-01 | project-first v1 + valid relative refs | PASS；read/write 均落当前 worktree docs/process |
| R-TC-02 | legacy v1 + 新路径同时存在 | PASS；按显式 version 选择写目标，不移动文件 |
| R-TC-03 | version 缺失 + 两套候选存在 | BLOCKED；write target=None；候选 2/2 披露 |
| R-TC-04 | unknown schema/layout | BLOCKED；稳定错误码 |
| R-TC-05 | absolute、`..`、NUL/newline、option-like project ID | 100% 拒绝；读取/Git/写入 spy=0 |
| R-TC-06 | requested project 与 config/namespace 不一致 | BLOCKED；mismatch 字段完整 |
| R-TC-07 | workspace 根整体移动 | 逻辑 route 相同；canonical 设备绝对前缀违规=0 |
| R-TC-08 | 相同输入重复 10 次 | decision、target order、code、config digest 一致率=100% |
| R-TC-09 | `docs_relative/process_relative.anchor=project_worktree`，其余 parent edges 均匹配固定 DAG | schema 可构造并 PASS；拓扑顺序固定为 root→control/sibling→worktree→leaf |
| R-TC-10 | unknown anchor 或任一字段使用错误父 anchor | `anchor_unknown` / `anchor_parent_invalid`；target 数=0；读取/Git/写入 spy=0 |
| R-TC-11 | self-cycle 或间接 anchor cycle fixture | `anchor_cycle`；target 数=0；不得递归溢出或降级猜测 |
| R-TC-12 | DAG 任一 edge 的 absolute、`.`、`..`、越界解析 | `absolute_canonical_path` / `path_escape`；mutation=0 |
| R-TC-13 | 同一 anchor+relative payload 在两个设备根目录解析 | canonical/config digest 相同；runtime absolute path 不进入 payload/digest；逻辑 target 相同 |

## 风险驱动测试

| Risk ID | 风险 | 测试方式 | 证据 | 未覆盖原因 |
|---|---|---|---|---|
| R-AW-R01 | 双可写真相 | 组合 layout/version/path-exists 矩阵 | pytest JSON snapshot | N/A |
| R-AW-R02 | sibling escape | 双项目 sentinel、path traversal、common-prefix trap | touched path audit=0 | N/A |
| R-AW-R03 | 兼容 adapter 漂移 | 新旧 API 对合法旧 fixture 结果比较 | regression result | N/A |
| R-AW-R04 | canonical 路径设备绑定 | 两个 temp root 重定位 | serialized payload scan | N/A |
| R-AW-R05 | anchor schema 不可构造或形成环 | 全字段 allowed-parent × wrong-parent 矩阵、自环/间接环 | typed error snapshot + resolver spy=0 | N/A |

## 权限 / 安全 / 失败路径

| Case ID | 触发条件 | 期望行为 | 测试入口 |
|---|---|---|---|
| SEC-R-01 | sibling dirty/untracked sentinel | 不读取内容、不阻断当前 route、不修改 sentinel | `test_sibling_is_not_consumed` |
| SEC-R-02 | control 内嵌 project worktree ref | BLOCKED；无 mkdir/link/Git | `test_nested_target_blocked` |
| SEC-R-03 | sparse path 被当授权 | 无 owned-path proof 则 BLOCKED | `test_sparse_is_not_authority` |
| SEC-R-04 | route config malformed | 输出字段/code/repair route；不泄漏 remote URL | error contract test |
| SEC-R-05 | unknown anchor、错误父 anchor、自环/间接环 | loader 在 target resolve 前 typed BLOCKED；target/Git/file/link mutation=0 | `test_anchor_graph_rejects_unknown_parent_and_cycle` |

## 验证命令（实现后）

```bash
PYTHONDONTWRITEBYTECODE=1 PYTEST_ADDOPTS='-p no:cacheprovider' uv run --python 3.11 pytest -q tests/test_cr051_project_artifact_routing.py tests/test_workspace_routing.py
```

测试只能使用临时目录；不得指向真实 `meta-flow-artifacts` 或修改现有 link。

## 手工验收

| Check ID | 操作 | 期望结果 | 责任方 |
|---|---|---|---|
| MAN-AW-R01 | 在两台设备/等价不同根目录读取同一 canonical config | project/layout/逻辑 target 一致，持久字段无设备绝对路径 | qa/human |
