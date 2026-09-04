---
story_id: STORY-038-05
story_slug: pppoe-client-op
cr_id: CR-038
feature_id: F-CR038-E2
title: PPPoE Client 拨号 op（op_mapper 映射）
doc_type: IMPLEMENTATION
priority: P0
wave: 2
status: done
source_lld: process/stories/STORY-038-05-pppoe-client-op-LLD.md
source_feature_design: docs/features/cr038-pppoe-config/DESIGN.md
created_at: "2026-08-15"
owner: meta-dev
---

# STORY-038-05 IMPLEMENTATION — PPPoE Client 拨号 op（op_mapper 映射）

## 实现前置检查

| 条目 | 结果 | 证据 |
|---|---|---|
| Story `status=dev-ready`，`dev_gate=cp5-approved` | PASS | `process/stories/STORY-038-05.md` |
| 设计证据 full-lld 已确认（CP5 approved） | PASS | `process/checkpoints/CP5-CR038.md`（§人工审查结果：approve） |
| CP5 自动预检 + 全量人工确认通过 | PASS | CP4 DAG 安全 PASS + CP5-CR038 approved |
| `depends_on: []`（无依赖） | PASS | Story frontmatter `dependency_type: none` |
| 文件所有权 primary `op_mapper.py` 无冲突 | PASS | 其它 Story（S03 commands.py+physical_pool.py、S04 switch_configurator.py）不共享 op_mapper.py |
| AI 可执行任务清单存在 | PASS | LLD §8 TASK-E2-05-01~03 |
| 平台目标明确（纯 Python 映射，不涉及安装结构） | PASS | 不涉及 `PLATFORM-CONTRACTS.yaml` |
| LLD clarification 阻断项 | 无 blocking | LCQ-STORY-038-05-01 `blocks_lld=false`（外部依赖登记落点方案 A，本 Story 仅固化 `external_dep_ref` 字符串） |

## 实现对象清单

| 对象 | 归属文件 | 类型 | 变更 |
|---|---|---|---|
| `INTENT_TO_OP` | `skills/policy-route-execution/scripts/op_mapper.py` | 模块级常量 | 新增 |
| `UNAVAILABLE_OPS` | 同上 | 模块级常量 | 新增 |
| `PPPOE_CLIENT_MANUAL_STEPS` | 同上 | 模块级常量 | 新增 |
| `_CIPHER_PREFIXES` / `_SECRET_KEYS` | 同上 | 模块级常量（私有） | 新增 |
| `_probe_op_availability()` | 同上 | 纯函数 | 新增 |
| `_assert_no_plaintext_secret()` | 同上 | 纯函数 | 新增 |
| `_build_manual_steps()` | 同上 | 纯函数 | 新增 |
| `map_intent()` | 同上 | 纯函数（核心入口） | 新增 |
| 现有 26 op 四表 / 现有函数 | 同上 | 既有真相源 | 不变 |
| `tests/test_op_mapper_pppoe_client.py` | 仓库根 `tests/` | 单元测试 | 新增 |

## 设计契约映射

| 契约 | 需求 | 实现点 | 验收场景 |
|---|---|---|---|
| R-F-005 | PPPoE Client op 归属 ptm-atomic 执行层，op_mapper 增补映射 | `INTENT_TO_OP["pppoe_client"]="fw_config_pppoe_client"` + `map_intent` 原子化分支 | SCN-038-POS-005（TC-POS-005-01/02） |
| RA-038-001 | ptm-atomic 未暴露 PPPoE 拨号 op，跨仓库新增 | `UNAVAILABLE_OPS` 快照 + `external_dep_ref` + `_probe_op_availability` | SCN-038-PRE-001（TC-PRE-001-01）、TC-RA-001-01/02 |
| DQ-038-03 | 默认 fallback=atomic-skip 写 reason，可切 abort | `map_intent` `fallback` 参数 + `_build_manual_steps` + abort 抛 `OpNotFoundError` | TC-ABORT-01、TC-POS-005-01 |
| ADR-02 | 凭据 `${ENV_VAR}` 占位 / cipher 密文 / 禁止明文 | `_assert_no_plaintext_secret` + 手工步骤模板占位 | TC-SEC-01 |
| R-C-006 | 拨号地址 163.0.0.x 不进静态 IPAM | 本 Story 不涉及 IPAM（手工步骤仅描述验证） | 不适用（跨 Story 边界） |

三选一实际判定结论（LLD §5.3）：**默认走 (c) atomic-skip（运行时降级手工步骤）+ 登记 (b) 外部依赖 follow-up；(a) 为 ptm-atomic 补 op 后自动激活路径**。当前 `fw_config_pppoe_client` 不在 `OP_ID_TO_SUBCOMMAND`，命中 `UNAVAILABLE_OPS` 快照（probe_conclusion=missing）。

## 单元测试与 Fixture 计划

- 框架：`unittest` + `unittest.mock`，沿用仓库 `tests/` 现有 `load_module` 约定（`importlib.util.spec_from_file_location`）。
- 无外部 Fixture（纯函数，无网络、无 subprocess、无落盘）。
- (a) 原子化分支用 `mock.patch.dict(OP_ID_TO_SUBCOMMAND, ...)` 注入，不触网（LLD §8.3）。
- 测试用例矩阵（LLD §7.2 全量落地）：TC-POS-005-01/02、TC-PRE-001-01/02、TC-RA-001-01/02、TC-ABORT-01、TC-SEC-01（4 例）、TC-NFR-03-01（3 例）、TC-INTENT-01、非法 fallback 1 例、unknown path 1 例，共 17 例。

## 最小实现切片

| Slice | TASK-ID | 内容 | 验证 |
|---|---|---|---|
| S1 | TASK-E2-05-01 | 新增 `INTENT_TO_OP` + `UNAVAILABLE_OPS` + `_probe_op_availability` + `map_intent` 的 available/unknown 分支 | TC-PRE-001-01/02、TC-POS-005-02 |
| S2 | TASK-E2-05-02 | 新增 `_build_manual_steps` + `PPPOE_CLIENT_MANUAL_STEPS` + `_assert_no_plaintext_secret` + `map_intent` 的 atomic-skip/abort 分支 | TC-POS-005-01、TC-RA-001-01/02、TC-ABORT-01、TC-SEC-01、TC-INTENT-01 |
| S3 | TASK-E2-05-03 | 新增测试文件全部用例 + 回归 | 全部 PASS + `validate_mapping_consistency` PASS |

> 实施顺序 S1→S2→S3 串行（同一文件单写），本 Story 由单一 meta-dev 线程完成，无并行冲突。

## 平台差异处理

N/A —— 本 Story 为纯 Python 映射/探测/降级函数，无平台路径、无安装结构、无 OS 特定行为（NFR-2 无副作用）。不涉及 `delivery/doc/PLATFORM-CONTRACTS.yaml`。

## 验证结果

- `uv run --python 3.11 pytest tests/test_op_mapper_pppoe_client.py tests/test_op_mapper_rollback.py tests/test_step_refs.py tests/test_tg_op_mapping.py` → **93 passed, 0 failed**（新增 17 例 + 回归 76 例）。
- `validate_mapping_consistency().passed == True`，`EXPECTED_OP_COUNT == 26`（TC-NFR-03-01）。
- 手动 dry-run：`map_intent("pppoe_client", {username, password="${ENV_PPPOE_USER_PASSWORD}"})` 返回 `available=False, fallback=atomic-skip`，`reason` 含 ptm-atomic / pppoe-client / 降级手工步骤（外部依赖 follow-up 已登记，跨仓库依赖），`manual_steps` 6 条含 `${ENV_PPPOE_USER_PASSWORD}` 占位，无明文。

## 未覆盖项

- 端到端「NGFW 接口拨号获 163.0.0.x」属真机 `runtime_authorization`，本 Story 不覆盖（独立授权，CP7 meta-qa 真机阶段）。
- 编排层对 `available=False` 时 `manual_steps` 字段的消费分支属调用方（ptm-te 编排 / case_runner），非本 Story 文件所有权。
- ptm-atomic 本体扩展（`--interface-kind=pppoe-client` 或新增 `fw_config_pppoe_client` op）属跨仓库 follow-up，本 Story 仅固化 `external_dep_ref`。

## 设计缺口反馈

无阻断性缺口。LLD §10.1「cipher 密文（非 ${ENV_*} 但非空）→ 通过」与「明显明文 → 抛」存在轻微表述歧义；实现按 LCQ-STORY-038-03-01 方案 A 收敛为**保守密文前缀 allowlist**（`${ENV_*}` + H3C `$c$3$` / Huawei `%^%#` / Ruijie `$1$/$5$/$6$`），其余非空值判明文拒绝，确定性且与 S03 一致。已在 `_CIPHER_PREFIXES` 常量注明。

## 后续交接（给 meta-qa 验证入口）

- LLD 路径：`process/stories/STORY-038-05-pppoe-client-op-LLD.md`
- 实现文件：`skills/policy-route-execution/scripts/op_mapper.py`
- 测试入口：`uv run --python 3.11 pytest tests/test_op_mapper_pppoe_client.py`
- 三选一结论：默认 (c) atomic-skip + 登记 (b) 外部依赖 follow-up；(a) 为 ptm-atomic 补 op 后自动激活路径
- 风险提示：端到端真机拨号不属本 Story；编排层需新增 `manual_steps` 消费分支（调用方范围）；LCQ-STORY-038-05-01 外部依赖台账由 host-orchestrator CP5 后统一汇总
