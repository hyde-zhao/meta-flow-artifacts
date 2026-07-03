---
story_id: "STORY-001"
title: "Freeze schema v1.1 contract and field docs"
story_slug: "freeze-schema-v11-contract-and-field-docs"
status: "verified"
priority: "P0"
wave: "W1"
depends_on: []
dependency_contracts: []
dependency_type: []
file_ownership:
  primary:
    - "schemas/atomic-op.schema.yaml"
    - "docs/schema-field-reference.md"
    - "docs/error-codes.md"
    - "docs/naming-convention.md"
  shared:
    - "atoms/fw/fw_verify_acl_rule.yaml"
  merge_owner: "STORY-001"
  forbidden:
    - ".input/"
    - "delivery/"
lld_gate:
  required_inputs:
    - "process/HLD.md"
    - "process/ARCHITECTURE-DECISION.md"
    - "process/PLATFORM-INSTALL-SPEC.md"
    - "process/stories/STORY-001-freeze-schema-v11-contract-and-field-docs.md"
  status: "ready_by_CR-003"
dev_gate:
  lld_confirmed: true
  dependencies_satisfied: true
  file_conflict_free: true
  cp5_confirmed_required: true
  cp5_confirmed: true
  implementation_complete: true
  cp6_status: "PASS"
  verification_complete: true
  cp7_status: "PASS"
  verification_evidence: "process/checks/CP7-STORY-001-freeze-schema-v11-contract-and-field-docs-VERIFICATION-DONE.md"
  contract_frozen: true
task_count: 8
created_at: "2026-05-18T14:35:00+0800"
updated_at: "2026-05-18T18:32:39+0800"
---

# STORY-001 Freeze schema v1.1 contract and field docs

## 目标

冻结 schema v1.1 字段族和同步文档契约，使后续 atom、package、CLI 校验和安全 gate 都能以同一结构事实为输入。

## 开发上下文（dev_context）

- **背景说明**：CP3 已接受方案 B。当前 `schemas/atomic-op.schema.yaml` 使用 `additionalProperties: false`，现有字段无法结构化表达风险、凭据引用、登录状态、审查 gate、验证和多设备批次契约。
- **输入文件**：`process/HLD.md` §5/§8/§10/§12/§14，`process/ARCHITECTURE-DECISION.md`，`README.md`，`schemas/atomic-op.schema.yaml`，`docs/schema-field-reference.md`，`docs/error-codes.md`，`docs/naming-convention.md`，`atoms/fw/fw_verify_acl_rule.yaml`。
- **输出文件**：`schemas/atomic-op.schema.yaml`，`docs/schema-field-reference.md`，`docs/error-codes.md`，`docs/naming-convention.md`；可选择新增或迁移一个 v1.1 示例，但不得静默改变 `fw_verify_acl_rule` 业务语义。
- **接口约定**：新增字段族为 `risk`、`credential_ref`、`session_ref`、`state_ref`、`gate`、`verification`、`batch`；字段只描述契约和校验元数据，不表达真实 executor。
- **设计约束**：现有 schema v1.0 atom 必须继续可校验；新增 high-risk atom 必须带 `risk` 和 `gate`；`credential_ref` 与明文 `password/token/cookie/ftp_pass/secret` 互斥。
- **命名规范**：防火墙 atom 使用 `fw_` 前缀；保留 README 当前 `op_id` 正则；CP4 计划推荐 schema version 为 `"1.1"`，若实现坚持当前 semver 正则则必须在 LLD 中说明 `"1.1.0"` 的兼容切换条件。
- **平台目标**：README 原生交付面；不生成 `delivery/`，不引入安装器。
- **AI 可执行任务清单**：见下表。

### 依赖与并行门控

| 上游 Story | 类型 | LLD 门控 | 开发门控 | 说明 |
|---|---|---|---|---|
| 无 | none | CR-003 解除 CP4 人工前置门控，当前可进入 LLD | CP5 confirmed | 本 Story 是下游 contract 依赖源。 |

### 文件所有权

| 类型 | 文件 | Owner / 合并规则 |
|---|---|---|
| primary | `schemas/atomic-op.schema.yaml` | 当前 Story 独占写入 |
| primary | `docs/schema-field-reference.md` | 当前 Story 独占写入 |
| primary | `docs/error-codes.md` | 当前 Story 独占写入 |
| primary | `docs/naming-convention.md` | 当前 Story 独占写入 |
| shared | `atoms/fw/fw_verify_acl_rule.yaml` | 仅用于迁移策略或示例确认；如修改由 STORY-001 合并 |
| forbidden | `.input/`、`delivery/` | 禁止读取运行时资产作为复制来源，禁止写入交付包目录 |

### 文件系统布局

```text
schemas/
└── atomic-op.schema.yaml
docs/
├── schema-field-reference.md
├── error-codes.md
└── naming-convention.md
atoms/fw/
└── fw_verify_acl_rule.yaml
```

### 关键字段契约

| 字段族 | 字段 | 类型 / 约束 |
|---|---|---|
| `risk` | `risk.level`, `risk.categories` | `level` in `low/medium/high`; high-risk atom 必填 |
| `credential_ref` | `kind`, `ref` | 只允许引用名或占位符，禁止明文认证载荷 |
| `session_ref` | `returns.data.session_ref` | 8..128 字符，禁止 `token=`、`Cookie:`、`password` |
| `state_ref` | `inputs.state_ref`, `returns.data.state_ref`, `state_status`, `expires_at` | `state_status` in `valid/expired/missing/invalid` |
| `gate` | `required`, `reason`, `approver_role`, `evidence_required` | high-risk atom 必须 `required=true` 且 `reason` 非空 |
| `verification` | `kind`, `rules`, `summary_ref` | 失败只诊断和人工处理，不自动回滚 |
| `batch` | `max_concurrency`, `device_inventory_ref`, `idempotency_key`, `failure_policy` | `max_concurrency` 1..5，默认 1；失败隔离枚举 |

### AI 可执行任务清单

| TASK-ID | 动作 | 目标文件 | 描述 |
|---|---|---|---|
| S001-T1 | 修改 | `schemas/atomic-op.schema.yaml` | 声明 schema v1.1 字段族、枚举、互斥和 high-risk 必填条件。 |
| S001-T2 | 修改 | `docs/schema-field-reference.md` | 同步新增字段、必填性、示例、禁止字段和兼容策略。 |
| S001-T3 | 修改 | `docs/error-codes.md` | 记录安全 gate 和 schema 扩展相关 finding/退出码边界，不复用为真实设备错误。 |
| S001-T4 | 修改 | `docs/naming-convention.md` | 冻结 `fw_` 前缀、防火墙动词、禁止新增真实动作命令的命名边界。 |
| S001-T5 | 判定 | `atoms/fw/fw_verify_acl_rule.yaml` | 明确保持 v1.0 不变或新增 v1.1 示例；禁止静默改变业务语义。 |
| S001-T6 | 校验 | `schemas/atomic-op.schema.yaml` | 确认现有 v1.0 atom 仍通过 schema 校验。 |
| S001-T7 | 扫描 | `docs/`、`schemas/`、`atoms/` | 确认真实 IP、token、cookie、FTP 凭据、原始默认密码数量为 0。 |
| S001-T8 | 记录 | LLD/CP6 | 记录 `"1.1"` 与当前 semver 正则差异的最终实现选择和回滚策略。 |

## 验证上下文（validation_context）

- **验证入口**：`uv run --python 3.11 python scripts/validate_schema.py atoms/fw/fw_verify_acl_rule.yaml`；实现后可扩展为全量 atom 校验。
- **验证方式**：schema 校验、字段参考一致性审查、命名规范审查、敏感模式扫描。
- **依赖环境**：Python 3.11+、uv、当前仓库依赖 `pyyaml` 和 `jsonschema`。
- **关键验证场景**：现有 v1.0 atom 兼容；新增 v1.1 字段族可校验；high-risk 缺少 gate 失败；明文敏感字段与 `credential_ref` 互斥。

## 量化验收标准（acceptance_criteria）

- [ ] **字段族覆盖**：7 个字段族全部在 schema 与字段参考中出现，缺失数为 0。
- [ ] **兼容性**：当前 `atoms/fw/fw_verify_acl_rule.yaml` 校验通过，现有 v1.0 atom 失败数为 0。
- [ ] **high-risk gate**：至少 1 个 schema 或静态检查规则能判定 high-risk atom 缺少 `gate.required=true` 为失败。
- [ ] **敏感互斥**：`credential_ref` 与明文 `password/token/cookie/ftp_pass/secret` 的互斥规则有机器检查或明确 LLD 实现路径。
- [ ] **文档同步**：schema 新增字段在 `docs/schema-field-reference.md` 中覆盖率为 100%。
- [ ] **命名规范**：防火墙新 atom 的 `fw_` 前缀和禁止真实动作命令规则已写入命名规范。
- [ ] **版本决策**：`schema_version` 最终值和兼容策略写入 LLD，且不与 schema 正则冲突。
- [ ] **禁止复制**：新增或修改内容中 `.input/` 源码、env、日志、凭据复制数量为 0。

## 回滚策略

若 schema v1.1 扩展导致现有 atom 或 CLI 读取失败，回退本 Story 对 `schemas/` 和字段参考的修改，保留 HLD/ADR 作为后续 CR 输入；下游 Story 不得在契约未恢复前继续开发。

## 边界说明

本 Story 不创建真实设备 executor，不新增 CLI 真实动作命令，不修改 `.input/`。按 CR-003 当前可生成 LLD；CP5 全量人工确认前不得启动实现。

## 阻塞说明（如有）

（无）
