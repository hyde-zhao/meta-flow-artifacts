---
doc_type: IMPLEMENTATION
story_id: STORY-038-01
story_slug: sw-explicit-mapping
cr_id: CR-038
feature_id: F-CR038-E1
title: SW 节点显式映射（含显式 SW 优先级 + role 保留）— 实现执行证据
status: done
version: "0.1"
created_at: "2026-08-15"
owner: meta-dev
lld_ref: process/stories/STORY-038-01-sw-explicit-mapping-LLD.md
dev_gate: cp5-approved
---

# STORY-038-01 实现执行证据 — SW 节点显式映射

> 消费设计证据：`process/stories/STORY-038-01-sw-explicit-mapping-LLD.md`（full-lld，14 段）。
> 强输入：Story 卡片 `process/stories/STORY-038-01.md` + Feature 设计 `docs/features/cr038-sw-mapping/{DESIGN,TEST-PLAN,TASKS}.md`。

## 实现前置检查

| 检查项 | 结果 | 说明 |
|--------|------|------|
| Story status | PASS | `dev-ready`（W1 起点，`depends_on: []`） |
| 设计证据已确认 | PASS | full-lld 已产出，`dev_gate=cp5-approved`，由 host-orchestrator 分发实现 |
| dev_context / validation_context / acceptance_criteria | PASS | 三字段完整（见 Story 卡片） |
| 输出文件路径明确 | PASS | primary=`skills/topo-config/src/topo_mapper.py`；新增 tests/ 文件 |
| 文件所有权无冲突 | PASS | 本文件 primary 仅本 Story 写入；tests/ 为新建，不修改既有测试 |
| AI 可执行任务清单 | PASS | `TASKS.md` TASK-E1-01-01~05 |
| 依赖类型可判定 | PASS | `depends_on=[]`，无上游门控 |
| HLD / ADR / FEATURE-DESIGN-MATRIX 已确认 | PASS | HLD-CR-038 §10.1 + ADR-CR038-01 + Feature DESIGN §2/§3/§4 |
| 平台目标明确 | PASS | 纯内存映射引擎，不涉及平台目录/安装结构（平台差异 N/A） |

## 实现对象清单

| 文件 | 操作 | 所有权 | 说明 |
|------|:---:|--------|------|
| `skills/topo-config/src/topo_mapper.py` | 修改 | primary | 新增 `explicit_sw_reserved`/`_candidates` 实例状态、`_compute_explicit_sw_reserved`、`_build_unmapped_reason`；`match()` 回填状态；`_try_l2_pass_through` 增加 reserved 跳过；`_build_result` 接入结构化 reason |
| `skills/topo-config/tests/test_topo_mapper_sw.py` | 新增 | 本 Story | 8 个单测（POS-001/002、NEG-001/002、BND-002、R-F-018 direct/end-to-end/顺序无关、R-NF-004） |
| `skills/topo-config/tests/fixtures/sw_explicit.sample.yaml` | 新增 | 本 Story | 显式 SW 物理池造数 fixture（TG1/dut1/SW1/SW2/SW3 + 5 环 + 第二条透传路径） |

**只读不修改**：`test_topo_mapper.py`、`test_topo_mapper_s4.py`、`physical_pool.py`、`commands.py`、`physical_pool.yaml`（模板）。

## 设计契约映射

| 需求 / TASK-ID | 实现位置 | 验证入口 | 结果 |
|----------------|----------|----------|------|
| R-F-001（SW 显式匹配） | `_build_candidates`/`_backtrack`（现状）+ S3 确认 | `test_explicit_sw_matched_and_role` | PASS |
| R-F-002（role 保留） | `_build_result` matched 分支 `role=node.role`（现状，S3 确认） | `test_explicit_sw_matched_and_role` | PASS |
| R-F-018（显式 SW 优先于自动透传） | `_compute_explicit_sw_reserved`（S1）+ `_try_l2_pass_through` 跳过（S2） | `test_explicit_sw_reserved_excludes_passthrough` + `test_explicit_sw_reserved_end_to_end` + `test_explicit_sw_reserved_order_invariance` | PASS |
| R-NF-002（结构化失败 reason） | `_build_unmapped_reason`（S4）+ `_build_result` 接入 | `test_sw_no_candidate_reason` + `test_sw_interface_mismatch_reason` | PASS |
| R-NF-004（回归无破坏） | reserved 默认空集，无显式 SW 拓扑零影响 | `test_cr037_node2_regression` + 既有 topo-config 全量 | PASS |
| TASK-E1-01-01 | `__init__` 增状态 + `_compute_explicit_sw_reserved` + `match()` 回填 | 单测断言 reserved 集 | PASS |
| TASK-E1-01-02 | `_try_l2_pass_through` 循环内 `if sw_name in self.explicit_sw_reserved: continue` | `test_explicit_sw_reserved_excludes_passthrough` | PASS |
| TASK-E1-01-03 | 确认 matched 分支 `role=node.role` 已生效（无需改字段） | `test_explicit_sw_matched_and_role` | PASS |
| TASK-E1-01-04 | 新增 `_build_unmapped_reason` + `_build_result` 改调 | `test_sw_no_candidate_reason` + `test_sw_interface_mismatch_reason` | PASS |
| TASK-E1-01-05 | 新建 fixture + 单测文件 8 用例 | 全量 pytest | PASS |

## 单元测试与 Fixture 计划

| # | 用例 | 覆盖场景 | 断言（量化） | 结果 |
|---|------|---------|-------------|:---:|
| 1 | `test_explicit_sw_matched_and_role` | SCN-038-POS-001/002 | 5 环全 matched；sw3 `role='pppoe-server'`；5 link matched | PASS |
| 2 | `test_sw_no_candidate_reason` | SCN-038-NEG-001 | SW `status='unmapped'`；reason 含 `无 SW 类型物理设备` | PASS |
| 3 | `test_sw_interface_mismatch_reason` | SCN-038-NEG-002 | SW `status='unmapped'`；reason 含 `连通性约束` | PASS |
| 4 | `test_mock_still_virtual` | SCN-038-BND-002 | Mock `status='virtual'`、`physical=None`；link `unmapped`、reason 含 `Mock` | PASS |
| 5 | `test_explicit_sw_reserved_excludes_passthrough` | R-F-018 核心（direct） | `_try_l2_pass_through` 返回 `info['switch']=='SW1' != 'SW2'` | PASS |
| 6 | `test_explicit_sw_reserved_end_to_end` | R-F-018 端到端 | `explicit_sw_reserved=={'SW2'}`；`sw_x.physical=='SW2'`；auto_sws=={'SW1'}；`auto_sws ∩ reserved == ∅` | PASS |
| 7 | `test_explicit_sw_reserved_order_invariance` | R-F-018 顺序无关 | 交换节点插入顺序 → reserved 恒 `{'SW2'}`，auto_sws ∩ reserved == ∅ | PASS |
| 8 | `test_cr037_node2_regression` | R-NF-004 | 无显式 SW → reserved 空，透传 auto_sws=={'SW1'} | PASS |

Fixture：`sw_explicit.sample.yaml`（TG1/dut1/SW1/SW2/SW3，5 环直连 + `c6: TG1↔SW2` 第二条透传路径）；台账由 `_fresh_ledger` 从 fixture 扫描 `device_id`+`interfaces` 生成全 free 台账（复用 CR-037 S4 模式）。凭据字段仅用 `test-placeholder` 占位，不引入真实凭据。

## 最小实现切片

| 切片 | TASK-ID | 内容 | 验证 | 状态 |
|------|---------|------|------|:---:|
| S1 | TASK-E1-01-01 | `__init__` 增 `explicit_sw_reserved=set()`/`_candidates=None`；新增 `_compute_explicit_sw_reserved`；`match()` 在 `_build_candidates` 后回填两状态 | 单测断言 reserved | PASS |
| S2 | TASK-E1-01-02 | `_try_l2_pass_through` 循环内 `if sw_name in self.explicit_sw_reserved: continue` | `test_explicit_sw_reserved_excludes_passthrough` | PASS |
| S3 | TASK-E1-01-03 | 确认 matched 分支 `role=node.role` 已生效（审查结论：无需改字段） | `test_explicit_sw_matched_and_role` | PASS |
| S4 | TASK-E1-01-04 | 新增 `_build_unmapped_reason`；`_build_result` unmapped 分支改调 | NEG-001/NEG-002 | PASS |
| S5 | TASK-E1-01-05 | 新建 fixture + 单测文件 8 用例 | 全量 pytest | PASS |

切片间 S1→S2 有顺序（S2 读 S1 状态）；S3/S4 独立；S5 收尾。已按顺序完成。

## 平台差异处理

- **N/A**：本 Story 为纯内存映射引擎改动，不涉及平台目录、安装结构、`PLATFORM-CONTRACTS.yaml`、`PLATFORM-INSTALL-SPEC.md`。无跨平台路径推断，无平台特有 API。

## 验证结果

- 本 Story 单测：`uv run --python 3.11 pytest skills/topo-config/tests/test_topo_mapper_sw.py -v` → **8 passed**。
- 本 Story 范围 + 基线（排除并发 Story 的 `test_physical_pool_pppoe.py`/`test_commands_pppoe.py`）：`uv run --python 3.11 pytest skills/topo-config/tests/ --ignore=...pppoe...` → **90 passed**。
- 全量 `uv run --python 3.11 pytest skills/topo-config/tests/ -q` → **134 passed, 1 failed**；唯一失败项为 `test_physical_pool_pppoe.py::test_password_cipher_huawei_cipher_ok`，属于并发 Story（PPPoE 密码 cipher，`physical_pool.py`/`commands.py`，非本 Story 所有权），失败原因为该 Story 未完成的 `_validate_password_cipher` 实现（返回 `(False,'plaintext')` 而预期 `(True,'device_cipher')`）。本 Story 未修改 `physical_pool.py`/`commands.py`，该失败与本 Story 改动无因果关系。

## 未覆盖项

- `_build_unmapped_reason` 分类 1（无 node_type 声明）与分类 4（候选全被占用）未单独新增用例；分类 1 由既有 `test_port_insufficient`/解析缺陷间接覆盖，分类 4 属资源竞争边界，由 `_build_result` 确定性逻辑保障，未命中本 Story 验收场景清单。
- 台账写回（`_commit_allocation`）沿用 CR-037 S4 既有闭环，本 Story 未新增写回用例（不在验收清单，且由 `test_ledger_allocate_on_success` 等既有用例覆盖）。

## 设计缺口反馈

- 无阻塞缺口。所有灰区已在 LLD §13.2 收敛（透传方法名 `_try_l2_pass_through`、`explicit_sw_reserved` 实例属性形态、测试文件所有权新建规避）。
- 无 `blocks_lld=true` clarification item；无 OPEN / Spike。

## 后续交接

**给 meta-qa**：
- 实现文件：`skills/topo-config/src/topo_mapper.py`（修改）+ `skills/topo-config/tests/test_topo_mapper_sw.py`（新增）+ `skills/topo-config/tests/fixtures/sw_explicit.sample.yaml`（新增）。
- 验证入口：`uv run --python 3.11 pytest skills/topo-config/tests/test_topo_mapper_sw.py -v`；全量 `uv run --python 3.11 pytest skills/topo-config/tests/ -q`。
- 重点风险提示：
  1. `explicit_sw_reserved` 保守保留语义（ADR-CR038-01）：显式 SW 匹配失败时其候选不释放回自动透传池，可能引起「SW 不足→整体失败」；若真机环回出现过度保守，切回「仅跳过已分配 SW」备选（LLD §9）。
  2. `_build_unmapped_reason` 四级分类边界（分类 4 资源竞争 vs 分类 5 连通性失败），依赖 `self._candidates`（回溯前算定）与 `assignment`（回溯结果）确定性成立。
  3. **跨 Story 观察**：全量测试有 1 个失败项 `test_physical_pool_pppoe.py::test_password_cipher_huawei_cipher_ok`，位于并发 Story（`physical_pool.py`/`commands.py` 的 PPPoE 密码 cipher），非本 Story 所有权与改动，建议 meta-qa 在集成验证时与该 Story 的 owner 对齐，勿归因本 Story。
- 下游消费契约：`DeviceMapping.role`（`pppoe-server` / `l2-pass-through`）供 STORY-038-03/04（switch_configurator）判定 PPPoE Server；`MappingResult` schema 不变。
