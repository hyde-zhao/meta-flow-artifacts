---
story_id: STORY-038-02
story_slug: sw-pool-merge
story_name: SW 节点归并（pool_merge）
cr_id: CR-038
checkpoint: CP6
result: PASS
created_at: "2026-08-15"
author: meta-dev
design_evidence_type: technical-note
---

# CP6-STORY-038-02-sw-pool-merge-CODING-DONE

## Entry Criteria

| 条目 | 状态 |
|------|------|
| Story status 为 in-development / dev-ready | PASS（dev-ready，W1 首次写入方） |
| 技术说明已确认（technical-note）且 CP5 全量通过 | PASS（Story 卡片 `## 技术说明` 已补齐，dev_gate=cp5-approved） |
| 文件所有权无冲突 | PASS（pool_merge.py shared，S13 为 merge_owner；本 Story 为 W1 首次写入方，仅写 `_merge_sw_pc`/`_resolve_ledger` SW 补丁，不引入 trex_instances 等多实例字段） |
| 依赖满足（depends_on=[]） | PASS（无上游依赖） |
| AI 可执行任务清单存在 | PASS（TASKS.md TASK-E1-02-01~03） |

## Checklist

### 实现对象清单

| 文件 | 操作 | 说明 |
|------|:---:|------|
| `skills/topo-planning/src/pool_merge.py` | 修改 | `_merge_sw_pc` 精确 switch/client 块匹配；`_resolve_ledger` 限定 DUT 平台等价类解析；主循环 unmatched 判定简化 |
| `skills/topo-planning/tests/test_pool_merge_sw.py` | 新增 | 6 个 SW 归并台账映射单测（R-F-011） |

### 设计契约映射

| 契约 / TASK-ID | 实现位置 | 结果 |
|----------------|----------|------|
| R-F-011（SW 归并 + 台账映射一致无冲突） | `_merge_sw_pc`（switch 块匹配）+ `_resolve_ledger`（SW 台账映射） | PASS |
| TASK-E1-02-01（SW switch 块补充 device_type/hardware_platform） | `_merge_sw_pc` 内 `block_key={'SW':'switch','PC':'client'}` 精确匹配 | PASS |
| TASK-E1-02-02（SW 台账映射覆盖 _resolve_ledger） | `_resolve_ledger` 复用既有 ledger_model/ledger_status/port_status，限定 DUT 平台解析 | PASS |
| TASK-E1-02-03（单测 R-F-011 无冲突） | `tests/test_pool_merge_sw.py` 6 用例 | PASS |
| 不新增 UnifiedNode 字段 / 不改变 TG/DUT 归并路径 | 仅 `_merge_sw_pc`/`_resolve_ledger` 内部分支；TG/DUT 分支未动 | PASS |

### 单元测试与 Fixture 计划

| # | 检查项 | 结果 | 证据 |
|---|--------|------|------|
| 1 | SW switch 块回填 device_type/hardware_platform | PASS | `test_sw_merge_switch_block_backfill` |
| 2 | 无匹配 switch 块不阻断（字段保持 None） | PASS | `test_sw_merge_no_switch_block_not_blocking` |
| 3 | 精确匹配 switch 块（client 块同 host 不命中） | PASS | `test_sw_merge_precise_switch_block_only` |
| 4 | SW 台账映射一致（ledger_model/ledger_status/port_status） | PASS | `test_sw_ledger_mapping_r_f_011` |
| 5 | port_status 1:1 占用无重复 | PASS | `test_sw_ledger_port_allocation_no_duplicate` |
| 6 | SW 不触发 DUT 平台等价类解析（无噪音警告） | PASS | `test_sw_no_dut_hardware_platform_resolution` |

### 最小实现切片

| 切片 | 内容 | 状态 |
|------|------|:---:|
| S1 | `_merge_sw_pc` 精确 switch/client 块匹配并回填 | PASS |
| S2 | `_resolve_ledger` hardware_platform 解析限定 DUT | PASS |
| S3 | 主循环 unmatched 判定改为 `node.source != 'merged'` | PASS |
| S4 | 新增 6 个 SW 单测 | PASS |

### 平台差异处理

- N/A：纯内存归并 + 台账只读关联，不涉及平台目录 / 安装结构 / 平台路径，无 `PLATFORM-CONTRACTS.yaml` 依赖。

### 验证结果

- `uv run --python 3.11 pytest skills/topo-planning/tests/` → **147 passed**（含新增 6 个 SW 单测 + 既有 141 回归）。
- dry-run 归并 diff 核对：SW1 `device_type=ruijie-B`、`source=merged`；SW2/PC1/PC2 unmatched（无 switch/client 块或 host 缺失）；FW1/FW2/TG1 归并结果与 CR-037 基线一致，无 `cannot resolve hardware_platform` 噪音警告。

### 未覆盖项

- Mock 节点虚拟语义未单独新增 fixture（现有 pool_merge fixtures 无 Mock 节点；`_merge_sw_pc` 对 Mock 不查 devices、source 保持 `pool`）。
- PC → `client` 块精确匹配的正例未单独断言（仅通过 SW 反例证明 SW 不匹配 client 块；PC 节点在现有 fixture 中无 management.host，走 unmatched 路径，行为不变）。

### 设计缺口反馈

- 无阻塞。跨 Story 观察项：`REQUIREMENTS-CR-038.md` R-F-009 措辞为 `device_groups.sw` 块，而本 Story 技术说明、`DESIGN.md §6` 与现有 `devices_sample.yaml` 均使用 `switch` 块；本 Story 按技术说明以 `switch` 块为准。此命名差异属 STORY-038-08（模板回填）与 R-F-009 措辞口径，建议 host-orchestrator 在 CP5/集成期对齐，不影响本 Story 实现。

### 文件影响范围

| 文件 | 变更类型 |
|------|----------|
| `skills/topo-planning/src/pool_merge.py` | 修改（`_merge_sw_pc` / `_resolve_ledger` / 主循环 SW 分支） |
| `skills/topo-planning/tests/test_pool_merge_sw.py` | 新增 |

### 不改动的范围（已确认）

- 未新增 `UnifiedNode` / `UnifiedInterface` 字段（多实例 `trex_instances` / `instance` 由 STORY-038-13 后续合并）。
- 未改变 `_merge_dut` / `_merge_tg` / `_merge_unknown` 及 DUT/TG 归并路径。
- 未修改 REQUIREMENTS / HLD / ARCHITECTURE-DECISION / 蓝图三件套及 `common_forbidden` 所列对象。
- 未新增公开接口（`_merge_sw_pc` / `_resolve_ledger` 为私有函数，签名保持不变）。

## Exit Criteria

| 条目 | 状态 |
|------|------|
| TASK-E1-02-01~03 全部完成 | PASS（3/3） |
| R-F-011 验收通过 | PASS |
| 回归（既有 TG/DUT 归并 + 全量 topo-planning 测试）通过 | PASS（147 passed） |
| dry-run 归并 diff 核对补齐字段 | PASS |
| 无凭据 / 无真机下发 / 无 production 写入 | PASS（台账映射只读关联 physical_ledger） |

## Deliverables

| 产物 | 路径 |
|------|------|
| pool_merge.py 增量 | `skills/topo-planning/src/pool_merge.py` |
| SW 归并单测 | `skills/topo-planning/tests/test_pool_merge_sw.py` |
| CP6 自检结果 | `process/checks/CP6-STORY-038-02-sw-pool-merge-CODING-DONE.md` |
| 实现摘要 | `DEV-LOG.md`（STORY-038-02 段落） |

## Agent Dispatch Evidence

- 本 Story 由 meta-dev 角色直接执行实现；调度证据由 host-orchestrator 写入 `process/state/AGENT-DISPATCH-LEDGER.ndjson`（本 CP6 不代写调度事件，避免伪造 `tool_name` / `agent_id`）。

## 已知限制

- 台账映射只读关联 physical_ledger，不新增 production 写入；`_resolve_ledger` 不交叉校验 `ledger.model` 与 `device_type`（SW 硬件平台别名映射属 R-F-010 / STORY-038-09 范围，非本 Story）。
- `switch` 块若不含 `hardware_platform`，SW 节点 `hardware_platform` 保持 None（异常处理按技术说明，不阻断）。

## 提供给 meta-qa 的验证入口

1. `uv run --python 3.11 pytest skills/topo-planning/tests/test_pool_merge_sw.py -v`（SW 归并 6 用例）。
2. `uv run --python 3.11 pytest skills/topo-planning/tests/ -q`（全量 147 用例回归）。
3. 核对 SW 节点归并后 `device_type`/`hardware_platform`/`ledger_model`/`ledger_status`/`port_status` 与 physical_ledger 一致、无重复占用（R-F-011）。
