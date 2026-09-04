---
story_id: STORY-038-02
story_slug: sw-pool-merge
cr_id: CR-038
checkpoint: CP7
result: PASS_WITH_RISK
validation_mode: mixed
created_at: "2026-08-15"
author: meta-qa
design_evidence_type: technical-note
---

# CP7-STORY-038-02-sw-pool-merge-VERIFICATION-DONE

## 结论

| 项目 | 内容 |
|---|---|
| 阶段决策 | PASS_WITH_RISK |
| validation_mode | mixed（pytest + dry-run 归并 diff） |
| 路由 | none（verified；风险汇入 CP8） |

## 验证对象清单

| 文件 | 操作 | 验证结果 |
|---|---|---|
| `skills/topo-planning/src/pool_merge.py` | 修改（`_merge_sw_pc` 精确 switch 块 + `_resolve_ledger` SW 台账映射 + 主循环 unmatched 简化） | PASS |
| `skills/topo-planning/tests/test_pool_merge_sw.py` | 新增（6 用例） | PASS |

## 验证追踪矩阵

| 场景/契约 | 测试 | 结果 |
|---|---|---|
| R-F-011（SW 归并台账映射一致无重复） | `test_sw_ledger_mapping_r_f_011` / `test_sw_ledger_port_allocation_no_duplicate` | PASS |
| switch 块精确匹配（client 块不命中） | `test_sw_merge_precise_switch_block_only` | PASS |
| 无 switch 块不阻断 | `test_sw_merge_no_switch_block_not_blocking` | PASS |
| 不触发 DUT 平台等价类解析 | `test_sw_no_dut_hardware_platform_resolution` | PASS |

## 设计契约验证

- R-F-011：SW 节点 `device_type`/`hardware_platform`/`ledger_model`/`ledger_status`/`port_status` 与 physical_ledger 一致、无重复占用
- 不新增 `UnifiedNode` 字段（多实例字段由 S13 新增）；不改变 TG/DUT 归并路径
- 平台差异 N/A（纯内存归并 + 台账只读关联）

## 分层验证计划

- 层A：`test_pool_merge_sw.py` 独立复跑 6 passed；全量 340 passed / 0 failed
- 层B：N/A（无真机下发）

## 8 维度验收

| # | 维度 | 结果 |
|---|---|---|
| 1 | 完整性（BLOCKING） | PASS（2 产物 ≥ 验收标准） |
| 2 | 平台适配（BLOCKING） | PASS（N/A：纯内存归并） |
| 3 | 验收标准覆盖（BLOCKING） | PASS（R-F-011 全覆盖） |
| 4 | 安全合规（BLOCKING） | PASS（台账只读关联，无 production 写入） |
| 5 | 命名规范（REQUIRED） | PASS |
| 6 | Frontmatter（REQUIRED） | PASS（Story 卡片技术说明完整） |
| 7 | 可安装性（REQUIRED） | PASS（N/A：纯代码） |
| 8 | 文档覆盖（OPTIONAL） | 文档阶段检查 |

## 测试证据

- `uv run --python 3.11 pytest skills/topo-planning/tests/test_pool_merge_sw.py -q` → 6 passed
- 全量 → 340 passed, 0 failed

## 问题与缺陷

| # | 严重度 | 问题 | 状态 |
|---|---|---|---|
| Q1 | Minor（输入缺陷） | 缺 `process/returns/STORY-038-02.return.json` + `process/evidence/STORY-038-02.index.json` | 建议 CP8 前回填 |
| Q2 | Info（文档措辞） | R-F-009 `device_groups.sw` vs 实现 `switch` 块命名差异（LCQ-CR038-DOC-SW-BLOCK-NAMING） | 命名口径回写走 CR |

## 剩余风险

- 证据链缺口（缺 return/index）
- `switch` 块命名口径待回写（非本 Story 阻断）

## 阶段决策

PASS_WITH_RISK → verified；风险（证据链缺口 + 命名口径）汇入 CP8。
