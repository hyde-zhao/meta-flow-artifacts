---
story_id: STORY-038-13
story_slug: multi-instance-trex
cr_id: CR-038
checkpoint: CP7
result: PASS_WITH_RISK
validation_mode: mixed
created_at: "2026-08-15"
author: meta-qa
design_evidence_type: full-lld
---

# CP7-STORY-038-13-multi-instance-trex-VERIFICATION-DONE

## 结论

| 项目 | 内容 |
|---|---|
| 阶段决策 | PASS_WITH_RISK |
| validation_mode | mixed（pytest + dry-run 等价验证） |
| 路由 | none（verified；多实例真机下发 not_authorized） |

## 验证对象清单

| 文件 | 操作 | 验证结果 |
|---|---|---|
| `topology_model.py` | 修改（`InterfaceInfo.interface_kind` 默认 physical + `NodeInfo.trex_instance` 默认 None） | PASS |
| `parser.py` | 修改（`_parse_interface` 解析 interface_kind + 节点构造 trex_instance） | PASS |
| `physical_pool.py` | 修改（`get_trex_instances`/`get_trex_instance_of_port` + `get_free_ports` 透传） | PASS |
| `pool_merge.py` | 修改（`UnifiedNode.trex_instances`/`UnifiedInterface.instance` + `_resolve_env`/`_merge_trex_instances`/`_merge_tg`/`to_pool_node`） | PASS |
| `exporter.py` | 修改（`_resolve_tg_instance`/`_build_nodes`/`_resolve_ip_plan`） | PASS |
| 3 测试文件 + 3 fixture | 新增（30 用例） | PASS |

## 验证追踪矩阵

| 场景/契约 | 测试 | 结果 |
|---|---|---|
| SCN-038-POS-010（多实例 env-file） | `test_build_nodes_multi_instance` | PASS |
| SCN-038-POS-011（pppoe 无 ip/next_hop=None） | `test_interface_kind_pppoe_no_ip` / `test_interface_kind_physical_ip_checked` | PASS |
| SCN-038-BND-003 / R-NF-004（node2 单值回退） | `test_build_nodes_node2_no_new_keys` / `test_resolve_tg_instance_node2_fallback` | PASS |
| R-F-013~017 | `test_get_trex_instances_*`/`test_merge_tg_*`/`test_resolve_tg_instance_*`/`test_parser_interface_kind` | PASS |
| 凭据拒绝 | `test_resolve_env_credential_refused` | PASS |
| S06 duck-typing 零改动 | `test_interface_kind_constructor_passthrough` | PASS |

## 设计契约验证

- R-F-013：`get_trex_instances`/`get_trex_instance_of_port` + `get_free_ports` 透传（physical_pool 唯一真相源，ADR-CR038-02）
- R-F-014：`_merge_tg` 按 name 归并（pool 侧为准）+ `_resolve_env` 非凭据占位解析
- R-F-015：`_resolve_tg_instance` 确定性决策表（声明→单实例→歧义→不存在→回退），结构化 `TREX_INSTANCE_NOT_FOUND`/`TREX_INSTANCE_AMBIGUOUS`
- R-F-016/017：`InterfaceInfo.interface_kind` 默认 physical + `_resolve_ip_plan` pppoe 域不参与 next_hop
- OPEN-06-01 已闭环（S13 加真实字段 + S06 duck-typing 零改动，ADR-CR038-05）
- 设计偏差：STORY 卡片「接口级 instance 决定实例」落地为「节点级 `NodeInfo.trex_instance` 决定实例」，接口级 instance 仅用于 free_ports 归属反查，LLD §5.1/§5.3 无冲突，已留痕

## 分层验证计划

- 层A：`test_trex_multi_instance.py`+`test_pool_merge_trex.py`+`test_exporter_trex.py` 独立复跑 30 passed；全量 340 passed / 0 failed
- 层B：N/A（多实例真机 TRex 下发 not_authorized）

## 8 维度验收

| # | 维度 | 结果 |
|---|---|---|
| 1 | 完整性（BLOCKING） | PASS（11 产物 ≥ 验收标准） |
| 2 | 平台适配（BLOCKING） | PASS（N/A：纯 Python 数据模型/归并/导出） |
| 3 | 验收标准覆盖（BLOCKING） | PASS（POS-010/011、BND-003、R-F-013~017、R-NF-004 全覆盖） |
| 4 | 安全合规（BLOCKING） | PASS（`_resolve_env` 拒绝 PASSWORD/TOKEN/SECRET，GE1_1~4 禁改动） |
| 5 | 命名规范（REQUIRED） | PASS |
| 6 | Frontmatter（REQUIRED） | PASS |
| 7 | 可安装性（REQUIRED） | PASS（N/A：纯代码） |
| 8 | 文档覆盖（OPTIONAL） | 文档阶段检查 |

## 测试证据

- `uv run --python 3.11 pytest skills/topo-config/tests/test_trex_multi_instance.py skills/topo-planning/tests/test_pool_merge_trex.py skills/topo-planning/tests/test_exporter_trex.py -q` → 30 passed
- 全量 → 340 passed, 0 failed

## 问题与缺陷

| # | 严重度 | 问题 | 状态 |
|---|---|---|---|
| Q1 | Info（范围） | OPEN-06-02：环回完整 ip_plan 端到端未闭环 | 归属后续 Story/CR |
| Q2 | Minor | LCQ-STORY-038-13-01：devices.yaml `instances` 补充块 schema 按方案 A 实现，正式固化待集成期复核 | 集成期复核 |

## 剩余风险

- OPEN-06-02 未闭环（环回完整 ip_plan 端到端）
- LCQ-STORY-038-13-01/02/03 schema 固化待复核
- 多实例真机 TRex 下发 not_authorized（独立 runtime_authorization）
- 三 shared 文件（physical_pool.py/pool_merge.py/exporter.py）后续改动需 merge_owner 协调

## 阶段决策

PASS_WITH_RISK → verified；风险（OPEN-06-02 + schema 固化 + 真机下发不授权）汇入 CP8。
