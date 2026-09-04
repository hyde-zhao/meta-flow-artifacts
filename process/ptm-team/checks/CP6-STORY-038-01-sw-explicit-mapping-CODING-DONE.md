---
story_id: STORY-038-01
story_slug: sw-explicit-mapping
story_name: SW 节点显式映射（含显式 SW 优先级 + role 保留）
cr_id: CR-038
checkpoint: CP6
result: PASS
created_at: "2026-08-15"
author: meta-dev
design_evidence_type: full-lld
---

# CP6-STORY-038-01-sw-explicit-mapping-CODING-DONE

## Entry Criteria

| 条目 | 状态 |
|------|------|
| Story status 为 in-development / dev-ready | PASS（dev-ready，W1 起点） |
| 设计证据已确认（full-lld）且 CP5 全量通过 | PASS（`STORY-038-01-sw-explicit-mapping-LLD.md`，dev_gate=cp5-approved） |
| 文件所有权无冲突 | PASS（primary=`topo_mapper.py` 仅本 Story 写入；tests/ 为新建） |
| 依赖满足（depends_on=[]） | PASS（无上游依赖） |
| AI 可执行任务清单存在 | PASS（TASKS.md TASK-E1-01-01~05） |

## Checklist

### 实现对象清单

| 文件 | 操作 | 说明 |
|------|:---:|------|
| `skills/topo-config/src/topo_mapper.py` | 修改 | `__init__` 增 `explicit_sw_reserved`/`_candidates`；新增 `_compute_explicit_sw_reserved`、`_build_unmapped_reason`；`match()` 回填状态；`_try_l2_pass_through` 增 reserved 跳过；`_build_result` 接入结构化 reason |
| `skills/topo-config/tests/test_topo_mapper_sw.py` | 新增 | 8 个单测（POS-001/002、NEG-001/002、BND-002、R-F-018 direct/end-to-end/顺序无关、R-NF-004） |
| `skills/topo-config/tests/fixtures/sw_explicit.sample.yaml` | 新增 | 显式 SW 物理池造数 fixture |

### 设计契约映射

| 契约 / TASK-ID | 实现位置 | 结果 |
|----------------|----------|:---:|
| R-F-001（SW 显式匹配） | `_build_candidates`/`_backtrack`（现状）+ S3 确认 | PASS |
| R-F-002（role 保留） | `_build_result` matched 分支 `role=node.role`（现状，S3 确认） | PASS |
| R-F-018（显式 SW 优先于自动透传） | `_compute_explicit_sw_reserved`（S1）+ `_try_l2_pass_through` 跳过（S2） | PASS |
| R-NF-002（结构化失败 reason） | `_build_unmapped_reason`（S4）+ `_build_result` 接入 | PASS |
| R-NF-004（回归无破坏） | reserved 默认空集，无显式 SW 拓扑零影响 | PASS |
| TASK-E1-01-01~05 | 见 IMPLEMENTATION.md 设计契约映射表 | PASS（5/5） |

### 单元测试与 Fixture 计划

| # | 用例 | 覆盖场景 | 结果 |
|---|------|---------|:---:|
| 1 | `test_explicit_sw_matched_and_role` | SCN-038-POS-001/002 | PASS |
| 2 | `test_sw_no_candidate_reason` | SCN-038-NEG-001 | PASS |
| 3 | `test_sw_interface_mismatch_reason` | SCN-038-NEG-002 | PASS |
| 4 | `test_mock_still_virtual` | SCN-038-BND-002 | PASS |
| 5 | `test_explicit_sw_reserved_excludes_passthrough` | R-F-018（direct） | PASS |
| 6 | `test_explicit_sw_reserved_end_to_end` | R-F-018（端到端） | PASS |
| 7 | `test_explicit_sw_reserved_order_invariance` | R-F-018（顺序无关） | PASS |
| 8 | `test_cr037_node2_regression` | R-NF-004 | PASS |

Fixture：`sw_explicit.sample.yaml`；台账由 `_fresh_ledger` 生成全 free；凭据仅 `test-placeholder` 占位。

### 最小实现切片

| 切片 | 内容 | 状态 |
|------|------|:---:|
| S1 | `__init__` + `_compute_explicit_sw_reserved` + `match()` 回填 | PASS |
| S2 | `_try_l2_pass_through` reserved 跳过 | PASS |
| S3 | 确认 matched 分支 `role=node.role` 已生效 | PASS |
| S4 | `_build_unmapped_reason` + `_build_result` 接入 | PASS |
| S5 | 新建 fixture + 单测 8 用例 | PASS |

### 平台差异处理

- N/A：纯内存映射引擎，不涉及平台目录/安装结构/`PLATFORM-CONTRACTS.yaml`。

### 验证结果

- `uv run --python 3.11 pytest skills/topo-config/tests/test_topo_mapper_sw.py -v` → **8 passed**。
- 本 Story 范围 + 基线（排除并发 Story 两个 pppoe 测试文件）→ **90 passed**。
- 全量 `uv run --python 3.11 pytest skills/topo-config/tests/ -q` → **134 passed, 1 failed**；唯一失败项 `test_physical_pool_pppoe.py::test_password_cipher_huawei_cipher_ok` 属并发 Story（`physical_pool.py`/`commands.py` PPPoE 密码 cipher，非本 Story 所有权），与本 Story 改动无因果关系（本 Story 未修改 `physical_pool.py`/`commands.py`）。

### 未覆盖项

- `_build_unmapped_reason` 分类 1（无 node_type）与分类 4（候选全占用）未单独新增用例（不在验收清单；分类 1 由既有测试间接覆盖，分类 4 属资源竞争边界）。
- 台账写回未新增用例（沿用 CR-037 S4 既有闭环，`test_ledger_allocate_on_success` 覆盖）。

### 设计缺口反馈

- 无阻塞缺口。LLD §13.2 灰区均已收敛；无 `blocks_lld=true` clarification item，无 OPEN / Spike。

### 文件影响范围

| 文件 | 变更类型 |
|------|----------|
| `skills/topo-config/src/topo_mapper.py` | 修改（`__init__`/`match`/`_compute_explicit_sw_reserved`/`_try_l2_pass_through`/`_build_result`/`_build_unmapped_reason`） |
| `skills/topo-config/tests/test_topo_mapper_sw.py` | 新增 |
| `skills/topo-config/tests/fixtures/sw_explicit.sample.yaml` | 新增 |

### 不改动的范围（已确认）

- 未修改 REQUIREMENTS-CR-038.md / HLD-CR-038.md / ARCHITECTURE-DECISION-CR-038.md / 蓝图三件套。
- 未修改 `physical_pool.py`、`commands.py`、`topology_model.py`、`parser.py`（本 Story 只读）。
- 未修改既有测试文件 `test_topo_mapper.py`、`test_topo_mapper_s4.py` 等。
- 未新增公开接口（`match()` 签名与返回 schema 不变；新增 `_compute_explicit_sw_reserved`/`_build_unmapped_reason` 为私有方法）。

### 安全约束核对

| 约束 | 结果 |
|------|:---:|
| 凭据 `${ENV_VAR}` 占位禁止明文 | PASS（本 Story 不涉及凭据；fixture 仅 `test-placeholder` 占位） |
| 默认 dry-run；真机下发独立 runtime_authorization | PASS（纯映射引擎，无 telnet/web/trex 调用） |
| 不修改已 approve 的需求/HLD/ADR/蓝图三件套 | PASS |

## Exit Criteria

| 条目 | 状态 |
|------|------|
| TASK-E1-01-01~05 全部完成 | PASS（5/5） |
| 5 条验收场景通过（POS-001/002、NEG-001/002、BND-002） | PASS |
| R-F-018 互斥单测通过（顺序无关） | PASS |
| CR-037 node2 场景回归无破坏 | PASS（90 passed，本 Story 范围 + 基线） |
| CP6 编码完成检查结果写入 | PASS（本文件） |
| 无凭据 / 无真机下发 / 无 production 写入 | PASS |

## Deliverables

| 产物 | 路径 |
|------|------|
| topo_mapper.py 增量 | `skills/topo-config/src/topo_mapper.py` |
| 显式 SW 单测 | `skills/topo-config/tests/test_topo_mapper_sw.py` |
| SW fixture | `skills/topo-config/tests/fixtures/sw_explicit.sample.yaml` |
| 实现执行证据 | `process/stories/STORY-038-01-sw-explicit-mapping-IMPLEMENTATION.md` |
| CP6 自检结果 | `process/checks/CP6-STORY-038-01-sw-explicit-mapping-CODING-DONE.md` |
| 实现摘要 | `DEV-LOG.md`（STORY-038-01 段落） |

## Agent Dispatch Evidence

- 本 Story 由 meta-dev 角色直接执行实现；调度证据由 host-orchestrator 写入 `process/state/AGENT-DISPATCH-LEDGER.ndjson`（本 CP6 不代写调度事件，避免伪造 `tool_name` / `agent_id`）。

## 已知限制

- `explicit_sw_reserved` 保守保留：显式 SW 匹配失败时其候选不释放回自动透传池（ADR-CR038-01 决策），真机环回极端场景可能「过度保守」导致整体失败，切换备选方案见 LLD §9。
- `_build_unmapped_reason` 分类 4/5 边界依赖 `self._candidates` 与 `assignment` 的确定性，若后续有人把 reserved 计算移入回溯循环，顺序无关性会被破坏（由 `test_explicit_sw_reserved_order_invariance` 守护）。

## 提供给 meta-qa 的验证入口

1. `uv run --python 3.11 pytest skills/topo-config/tests/test_topo_mapper_sw.py -v`（显式 SW 8 用例）。
2. `uv run --python 3.11 pytest skills/topo-config/tests/ -q`（全量回归；注意存在 1 个并发 Story 的失败项 `test_physical_pool_pppoe.py`，非本 Story 所有权）。
3. 核对 `DeviceMapping.role`（`pppoe-server` / `l2-pass-through`）传递与 `MappingResult` schema 不变（下游 switch_configurator 消费契约）。
