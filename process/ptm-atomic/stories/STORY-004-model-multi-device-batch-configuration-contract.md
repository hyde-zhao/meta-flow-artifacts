---
story_id: "STORY-004"
title: "Model multi-device batch configuration contract"
story_slug: "model-multi-device-batch-configuration-contract"
status: "verified"
priority: "P0"
wave: "W2"
depends_on: ["STORY-001", "STORY-003"]
dependency_contracts:
  - "schema v1.1 batch field family"
  - "capacity domain naming and result envelope"
dependency_type: ["contract"]
file_ownership:
  primary:
    - "atoms/fw/fw_config_batch_interface.yaml"
    - "atoms/fw/fw_config_batch_object.yaml"
    - "atoms/fw/fw_config_batch_acl_policy.yaml"
    - "atoms/fw/fw_config_batch_policy_route.yaml"
    - "atoms/fw/fw_config_batch_static_route.yaml"
    - "atoms/fw/fw_config_batch_nat.yaml"
    - "atoms/fw/fw_config_batch_bandwidth.yaml"
    - "atoms/fw/fw_config_batch_black_white_list.yaml"
    - "atoms/fw/fw_config_batch_ssl_vpn.yaml"
    - "atoms/fw/fw_config_batch_dynamic_routing.yaml"
    - "packages/ngfw_batch_config.yaml"
    - "docs/batch-configuration-contract.md"
  shared: []
  merge_owner: "STORY-004"
  forbidden:
    - ".input/"
    - "delivery/"
lld_gate:
  required_inputs:
    - "process/HLD.md"
    - "process/ARCHITECTURE-DECISION.md"
    - "process/stories/STORY-004-model-multi-device-batch-configuration-contract.md"
  upstream_contracts: ["STORY-001", "STORY-003"]
  status: "ready_by_CR-003"
dev_gate:
  lld_confirmed: true
  dependencies_satisfied: true
  dependency_condition: "STORY-001 CP6 PASS and STORY-003 CP6 PASS with 10-domain contract frozen."
  file_conflict_free: true
  cp5_confirmed_required: true
  cp5_confirmed: true
  implementation_complete: true
  cp6_status: "PASS"
  verification_complete: true
  cp7_status: "PASS"
  verification_evidence: "process/checks/CP7-STORY-004-model-multi-device-batch-configuration-contract-VERIFICATION-DONE.md"
  contract_frozen: true
  downstream_stable_facts:
    - "Exactly 10 batch op_ids are frozen under fw_config_batch_<domain>."
    - "ngfw_batch_config package scope is frozen with exactly 10 batch operations."
    - "batch.max_concurrency default is 1 and schema maximum is 5."
    - "10/10 batch atoms have high-risk gate coverage."
    - "Batch idempotency key rule is documented."
    - "Failure isolation uses partial_failed, per_device_results[], failed_devices[], and verification_summary_ref."
  upstream_contract_evidence:
    - "process/checks/CP6-STORY-001-freeze-schema-v11-contract-and-field-docs-CODING-DONE.md"
    - "process/checks/CP6-STORY-003-model-capacity-ten-domain-config-and-verification-atoms-CODING-DONE.md"
  implementation_evidence:
    - "process/checks/CP6-STORY-004-model-multi-device-batch-configuration-contract-CODING-DONE.md"
task_count: 8
created_at: "2026-05-18T14:35:00+0800"
updated_at: "2026-05-18T18:32:39+0800"
---

# STORY-004 Model multi-device batch configuration contract

## 目标

把多防火墙设备配置建模为独立批次契约，包含设备清单引用、选择器、并发限制、失败隔离、幂等键、逐设备结果和批次验证汇总。

## 开发上下文（dev_context）

- **背景说明**：多设备批次配置会放大误配置影响面，必须独立于单设备 capacity Story 建模。
- **输入文件**：`process/HLD.md` 多设备批次配置契约，`process/ARCHITECTURE-DECISION.md` ADR-005，STORY-001/003 LLD 输出。
- **输出文件**：10 个 `fw_config_batch_<domain>.yaml`、`packages/ngfw_batch_config.yaml`、`docs/batch-configuration-contract.md`。
- **接口约定**：每个批次 atom 输入 `device_inventory_ref`、`device_selector`、`batch_ref`、`batch.max_concurrency`、`config_domain`、`params`、`state_ref`、`idempotency_key`；输出 `batch_status`、`per_device_results[]`、`failed_devices[]`、`verification_summary_ref`。
- **设计约束**：`batch.max_concurrency` 默认 1，最大 5；high-risk 默认 1；失败设备隔离，不自动回滚已成功设备；不内联真实设备地址、用户名、密码、token、cookie 或 FTP 凭据。
- **命名规范**：`fw_config_batch_interface` 等 10 个 domain 级 op_id。
- **平台目标**：`atoms/`、`packages/`、`docs/`。

### 依赖与并行门控

| 上游 Story | 类型 | LLD 门控 | 开发门控 | 说明 |
|---|---|---|---|---|
| STORY-001 | contract | 需 batch 字段族草案 | STORY-001 CP5 confirmed | 依赖 schema batch/risk/gate 字段。 |
| STORY-003 | contract | 需 10 域命名和 result envelope 草案 | STORY-003 CP5 confirmed | 批次按同一配置域建模。 |

### 文件所有权

| 类型 | 文件 | Owner / 合并规则 |
|---|---|---|
| primary | `atoms/fw/fw_config_batch_*.yaml` | 当前 Story 独占 |
| primary | `packages/ngfw_batch_config.yaml` | 当前 Story 独占 |
| primary | `docs/batch-configuration-contract.md` | 当前 Story 独占 |
| forbidden | `.input/`、`delivery/` | 禁止复制运行时资产或写安装包目录 |

### AI 可执行任务清单

| TASK-ID | 动作 | 目标文件 | 描述 |
|---|---|---|---|
| S004-T1 | 创建 | 10 个 `fw_config_batch_<domain>.yaml` | 按 STORY-003 的 10 域创建批次配置 atom。 |
| S004-T2 | 创建 | `packages/ngfw_batch_config.yaml` | 引用 10 个批次 atom。 |
| S004-T3 | 创建 | `docs/batch-configuration-contract.md` | 说明 inventory、selector、并发、幂等、失败隔离、验证汇总。 |
| S004-T4 | 校验 | 批次 atom | 确认 `batch.max_concurrency` 默认 1、最大 5，高风险默认 1。 |
| S004-T5 | 校验 | 批次 atom | 确认 `device_inventory_ref`、`device_selector`、`batch_ref`、`idempotency_key` 均存在。 |
| S004-T6 | 校验 | 批次返回 | 确认 `per_device_results[]`、`failed_devices[]`、`verification_summary_ref` 均存在。 |
| S004-T7 | 扫描 | atoms/docs/packages | 确认真实 IP、密码、token、cookie、FTP 凭据数量为 0。 |
| S004-T8 | 记录 | LLD/CP6 | 记录失败隔离和不自动回滚策略。 |

## 验证上下文（validation_context）

- **验证入口**：`uv run --python 3.11 python scripts/validate_schema.py atoms/fw/fw_config_batch_*.yaml`；`uv run atomic-ops validate --package ngfw_batch_config`。
- **验证方式**：schema 校验、package 引用校验、批次契约人工审查、security gate 检查。
- **依赖环境**：STORY-001/003 的 LLD 和契约确认。
- **关键验证场景**：`max_concurrency=6` 失败；缺少 `idempotency_key` 失败；单设备失败进入 `failed_devices[]` 而不是全局自动回滚。

## 量化验收标准（acceptance_criteria）

- [ ] **独立 Story**：多设备批次配置不与单设备 capacity atom 混写在同一 Story 文件所有权内。
- [ ] **批次 atom 数量**：新增 10 个 `fw_config_batch_<domain>` atom，10 域覆盖率 10/10。
- [ ] **并发约束**：每个批次 atom 定义 `batch.max_concurrency` 默认 1，最大 5，高风险默认 1。
- [ ] **设备选择**：每个批次 atom 包含 `device_inventory_ref` 和 `device_selector`。
- [ ] **幂等性**：每个批次 atom 包含 `idempotency_key`，并说明 key 组成。
- [ ] **失败隔离**：每个批次 atom 返回 `per_device_results[]` 和 `failed_devices[]`，且状态枚举含 `partial_failed`。
- [ ] **验证汇总**：每个批次 atom 返回 `verification_summary_ref` 或等价 summary 引用。
- [ ] **敏感边界**：真实 IP、密码、token、cookie、FTP 凭据数量为 0。

## 回滚策略

若批次契约无法通过 schema 或安全 gate，回退批次 atom、batch package 和 batch 文档；不影响 STORY-003 的单设备配置/验证 atom。

## 边界说明

本 Story 不实现批次 executor，不并发连接设备，不读取真实 inventory 文件，不写真实设备地址。

## 阻塞说明（如有）

（无）
