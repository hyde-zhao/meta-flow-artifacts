---
story_id: "STORY-003"
title: "Model capacity ten domain config and verification atoms"
story_slug: "model-capacity-ten-domain-config-and-verification-atoms"
status: "verified"
priority: "P0"
wave: "W1"
depends_on: ["STORY-001"]
dependency_contracts:
  - "schema v1.1 field families"
dependency_type: ["contract"]
file_ownership:
  primary:
    - "atoms/fw/fw_config_interface.yaml"
    - "atoms/fw/fw_verify_interface.yaml"
    - "atoms/fw/fw_config_object.yaml"
    - "atoms/fw/fw_verify_object.yaml"
    - "atoms/fw/fw_config_acl_policy.yaml"
    - "atoms/fw/fw_verify_acl_policy.yaml"
    - "atoms/fw/fw_config_policy_route.yaml"
    - "atoms/fw/fw_verify_policy_route.yaml"
    - "atoms/fw/fw_config_static_route.yaml"
    - "atoms/fw/fw_verify_static_route.yaml"
    - "atoms/fw/fw_config_nat.yaml"
    - "atoms/fw/fw_verify_nat.yaml"
    - "atoms/fw/fw_config_bandwidth.yaml"
    - "atoms/fw/fw_verify_bandwidth.yaml"
    - "atoms/fw/fw_config_black_white_list.yaml"
    - "atoms/fw/fw_verify_black_white_list.yaml"
    - "atoms/fw/fw_config_ssl_vpn.yaml"
    - "atoms/fw/fw_verify_ssl_vpn.yaml"
    - "atoms/fw/fw_config_dynamic_routing.yaml"
    - "atoms/fw/fw_verify_dynamic_routing.yaml"
    - "packages/ngfw_capacity_config.yaml"
    - "packages/ngfw_verification.yaml"
  shared: []
  merge_owner: "STORY-003"
  forbidden:
    - ".input/"
    - "delivery/"
    - "src/atomic_ops/cli.py"
lld_gate:
  required_inputs:
    - "process/HLD.md"
    - "process/ARCHITECTURE-DECISION.md"
    - "process/stories/STORY-003-model-capacity-ten-domain-config-and-verification-atoms.md"
  upstream_contracts: ["STORY-001"]
  status: "ready_by_CR-003"
dev_gate:
  lld_confirmed: true
  dependencies_satisfied: true
  dependency_condition: "STORY-001 CP6 PASS and schema v1.1 contract frozen."
  file_conflict_free: true
  cp5_confirmed_required: true
  cp5_confirmed: true
  upstream_contract_evidence: "process/checks/CP6-STORY-001-freeze-schema-v11-contract-and-field-docs-CODING-DONE.md"
  implementation_complete: true
  cp6_status: "PASS"
  verification_complete: true
  cp7_status: "PASS"
  verification_evidence: "process/checks/CP7-STORY-003-model-capacity-ten-domain-config-and-verification-atoms-VERIFICATION-DONE.md"
  contract_frozen: true
  downstream_stable_facts:
    - "10 capacity domains, 20 op_ids, and package scopes are frozen"
    - "ngfw_capacity_config has 10 config op_ids; ngfw_verification has exactly 10 capacity verification op_ids per D-004"
    - "10/10 configuration atoms have high-risk gate coverage"
task_count: 13
created_at: "2026-05-18T14:35:00+0800"
updated_at: "2026-05-18T18:32:39+0800"
---

# STORY-003 Model capacity ten domain config and verification atoms

## 目标

把 capacity 参考资料抽象为 atomic-ops 原生配置域契约，首批覆盖 HLD 指定 10 个配置域，并为每个域提供配置 atom、验证 atom 和参数契约。

## 开发上下文（dev_context）

- **背景说明**：capacity 不能整体保留或搬运到 atomic；本 Story 只抽象接口族、参数和返回契约，不复制 `.input/capacity` 源码或 env。
- **输入文件**：`process/HLD.md`，`process/ARCHITECTURE-DECISION.md`，`schemas/atomic-op.schema.yaml`，`docs/naming-convention.md`。
- **输出文件**：20 个 `atoms/fw/` 配置/验证 atom，2 个 package。
- **接口约定**：每个配置 atom 必须输入 `state_ref` 或 `session_ref`、`config_domain`、`params`、`idempotency_key`；输出 `config_result`、`error_type`、`diag_snapshot_ref`。每个验证 atom 必须输入配置结果引用和验证规则，输出 `verification.summary_ref`。
- **设计约束**：配置失败只输出诊断和人工处理信号，不自动回滚；所有配置 atom 为 high-risk，必须带 `risk` 和 `gate`。
- **命名规范**：配置 atom 使用 `fw_config_<domain>`；验证 atom 使用 `fw_verify_<domain>`；不新增 `run/execute/apply/configure` CLI 命令。
- **平台目标**：`atoms/` 和 `packages/`。

### 依赖与并行门控

| 上游 Story | 类型 | LLD 门控 | 开发门控 | 说明 |
|---|---|---|---|---|
| STORY-001 | contract | 可与 STORY-001 同批写 LLD，但需引用字段冻结草案 | STORY-001 CP5 confirmed | 需要 schema v1.1 和 high-risk gate。 |

### 文件所有权

| 类型 | 文件 | Owner / 合并规则 |
|---|---|---|
| primary | `atoms/fw/fw_config_*.yaml`、`atoms/fw/fw_verify_*.yaml` | 当前 Story 独占 |
| primary | `packages/ngfw_capacity_config.yaml`、`packages/ngfw_verification.yaml` | 当前 Story 独占 |
| forbidden | `.input/`、`delivery/`、`src/atomic_ops/cli.py` | 禁止复制源码或新增执行命令 |

### 10 域配置/验证清单

| 配置域 | 配置 atom | 验证 atom | 参数契约下限 |
|---|---|---|---|
| interface | `fw_config_interface` | `fw_verify_interface` | `interface_ref`, `admin_state`, `address_ref`, `params_digest` |
| object | `fw_config_object` | `fw_verify_object` | `object_type`, `object_name`, `object_ref`, `params_digest` |
| ACL/policy | `fw_config_acl_policy` | `fw_verify_acl_policy` | `policy_name`, `rule_ref`, `action`, `params_digest` |
| policy route | `fw_config_policy_route` | `fw_verify_policy_route` | `route_ref`, `source_ref`, `destination_ref`, `next_hop_ref` |
| static route | `fw_config_static_route` | `fw_verify_static_route` | `route_ref`, `destination_ref`, `next_hop_ref`, `metric` |
| NAT | `fw_config_nat` | `fw_verify_nat` | `nat_rule_ref`, `source_ref`, `translated_ref`, `direction` |
| bandwidth | `fw_config_bandwidth` | `fw_verify_bandwidth` | `profile_ref`, `limit_value`, `unit`, `target_ref` |
| black/white list | `fw_config_black_white_list` | `fw_verify_black_white_list` | `list_type`, `entry_ref`, `action`, `scope_ref` |
| SSL VPN | `fw_config_ssl_vpn` | `fw_verify_ssl_vpn` | `vpn_profile_ref`, `user_group_ref`, `portal_ref`, `policy_ref` |
| dynamic routing | `fw_config_dynamic_routing` | `fw_verify_dynamic_routing` | `protocol`, `neighbor_ref`, `area_ref`, `route_policy_ref` |

### AI 可执行任务清单

| TASK-ID | 动作 | 目标文件 | 描述 |
|---|---|---|---|
| S003-T1 | 创建 | interface atom pair | 定义 interface 配置与验证 atom。 |
| S003-T2 | 创建 | object atom pair | 定义 object 配置与验证 atom。 |
| S003-T3 | 创建 | ACL/policy atom pair | 定义 ACL/policy 配置与验证 atom。 |
| S003-T4 | 创建 | policy route atom pair | 定义 policy route 配置与验证 atom。 |
| S003-T5 | 创建 | static route atom pair | 定义 static route 配置与验证 atom。 |
| S003-T6 | 创建 | NAT atom pair | 定义 NAT 配置与验证 atom。 |
| S003-T7 | 创建 | bandwidth atom pair | 定义 bandwidth 配置与验证 atom。 |
| S003-T8 | 创建 | black/white list atom pair | 定义 black/white list 配置与验证 atom。 |
| S003-T9 | 创建 | SSL VPN atom pair | 定义 SSL VPN 配置与验证 atom。 |
| S003-T10 | 创建 | dynamic routing atom pair | 定义 dynamic routing 配置与验证 atom。 |
| S003-T11 | 创建 | `packages/ngfw_capacity_config.yaml` | 引用 10 个配置 atom。 |
| S003-T12 | 创建 | `packages/ngfw_verification.yaml` | 引用 10 个验证 atom，并包含 STORY-002 健康检查 op_id 时需声明跨 Story 引用。 |
| S003-T13 | 校验 | atoms/packages | 执行 schema、layout、package 引用和敏感信息检查。 |

## 验证上下文（validation_context）

- **验证入口**：`uv run --python 3.11 python scripts/validate_schema.py atoms/fw`；`uv run atomic-ops validate --package ngfw_capacity_config`；`uv run atomic-ops validate --package ngfw_verification`。
- **验证方式**：schema 校验、package 引用校验、10 域覆盖审查、安全 gate 检查。
- **依赖环境**：STORY-001 schema v1.1。
- **关键验证场景**：任一域缺少配置或验证 atom 失败；任一 high-risk 配置 atom 缺少 gate 失败；任一明文敏感值失败。

## 量化验收标准（acceptance_criteria）

- [ ] **配置域覆盖**：10 个 HLD 指定配置域全部覆盖，覆盖率 10/10。
- [ ] **atom 数量**：新增 20 个 atom，配置 atom 10 个、验证 atom 10 个。
- [ ] **TASK-ID 覆盖**：每个配置域至少 1 个 TASK-ID，域级 TASK-ID 覆盖率 10/10。
- [ ] **参数契约**：每个配置 atom 至少包含 `state_ref/session_ref`、`config_domain`、`params`、`idempotency_key` 四类参数或等价字段。
- [ ] **返回契约**：每个配置 atom 返回 `config_result`、`error_type`、`diag_snapshot_ref`，每个验证 atom 返回 `verification_summary_ref` 或等价 summary 引用。
- [ ] **high-risk gate**：10 个配置 atom 的 `risk/gate` 覆盖率 100%。
- [ ] **package 引用**：两个 package 引用缺失 op_id 数为 0。
- [ ] **禁止搬运**：`.input/capacity` 源码、env、日志、token 或请求体复制数量为 0。

## 回滚策略

若某个配置域不满足 schema 或安全 gate，可回退该域的配置/验证 atom pair 与 package 引用；不得保留只有配置无验证或只有验证无配置的半成品域。

## 边界说明

本 Story 不实现 capacity API 客户端，不连接设备，不执行配置，不新增 CLI 真实动作命令。

## 阻塞说明（如有）

（无）
