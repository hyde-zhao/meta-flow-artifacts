---
story_id: "STORY-002"
title: "Model NGFW install init login guard atoms"
story_slug: "model-ngfw-install-init-login-guard-atoms"
status: "verified"
priority: "P0"
wave: "W1"
depends_on: ["STORY-001"]
dependency_contracts:
  - "schema v1.1 field families"
dependency_type: ["contract"]
file_ownership:
  primary:
    - "atoms/fw/fw_install_ngfw_image.yaml"
    - "atoms/fw/fw_init_ngfw_minimal.yaml"
    - "atoms/fw/fw_login_web_management.yaml"
    - "atoms/fw/fw_check_login_state.yaml"
    - "atoms/fw/fw_verify_ngfw_health.yaml"
    - "packages/ngfw_installation.yaml"
  shared: []
  merge_owner: "STORY-002"
  forbidden:
    - ".input/"
    - "delivery/"
    - "src/atomic_ops/cli.py"
lld_gate:
  required_inputs:
    - "process/HLD.md"
    - "process/ARCHITECTURE-DECISION.md"
    - "process/stories/STORY-002-model-ngfw-install-init-login-guard-atoms.md"
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
  verification_evidence: "process/checks/CP7-STORY-002-model-ngfw-install-init-login-guard-atoms-VERIFICATION-DONE.md"
  downstream_stable_facts:
    - "ngfw_installation package references 5 install/login/health op_ids with 0 missing refs"
    - "5 STORY-002 atoms use schema_version 1.1, high-risk gate coverage 5/5, and no real executor boundary"
task_count: 7
created_at: "2026-05-18T14:35:00+0800"
updated_at: "2026-05-18T18:32:39+0800"
---

# STORY-002 Model NGFW install init login guard atoms

## 目标

创建安装、初始化、Web 登录、登录态守卫和安装后健康检查的防火墙 atom 契约与安装 package，使 UC-05..UC-08 和安装侧 UC-10 能被原生 atom catalog 表达。

## 开发上下文（dev_context）

- **背景说明**：需求确认安装、初始化、登录、登录态失效重登和安装后 VPP/Agent/HTTPS 健康检查。当前 CLI 不能执行这些动作，本 Story 只建 atom 契约。
- **输入文件**：`process/HLD.md`，`process/ARCHITECTURE-DECISION.md`，`README.md`，`docs/naming-convention.md`，`schemas/atomic-op.schema.yaml`。
- **输出文件**：5 个 `atoms/fw/*.yaml` 和 `packages/ngfw_installation.yaml`。
- **接口约定**：所有 high-risk atom 使用 `risk.level=high` 和 `gate.required=true`；登录 atom 返回 `session_ref` / `state_ref` 占位契约；守卫 atom 输入 `state_ref`，输出 `state_status`。
- **设计约束**：初始化动作只包含通过串口登录、关闭验证码、配置管理路由、统一改密为 `Ngfw@123`；不加入 SSH/license；不保存原始默认密码、完整 token、cookie、真实设备地址或 FTP 凭据。
- **命名规范**：`fw_install_ngfw_image`、`fw_init_ngfw_minimal`、`fw_login_web_management`、`fw_check_login_state`、`fw_verify_ngfw_health`。
- **平台目标**：`atoms/` 和 `packages/`，不写 `delivery/`。

### 依赖与并行门控

| 上游 Story | 类型 | LLD 门控 | 开发门控 | 说明 |
|---|---|---|---|---|
| STORY-001 | contract | 可与 STORY-001 同批写 LLD，但必须引用字段冻结草案 | STORY-001 CP5 confirmed | 需要 schema v1.1 字段族。 |

### 文件所有权

| 类型 | 文件 | Owner / 合并规则 |
|---|---|---|
| primary | `atoms/fw/fw_install_ngfw_image.yaml` | 当前 Story 独占 |
| primary | `atoms/fw/fw_init_ngfw_minimal.yaml` | 当前 Story 独占 |
| primary | `atoms/fw/fw_login_web_management.yaml` | 当前 Story 独占 |
| primary | `atoms/fw/fw_check_login_state.yaml` | 当前 Story 独占 |
| primary | `atoms/fw/fw_verify_ngfw_health.yaml` | 当前 Story 独占 |
| primary | `packages/ngfw_installation.yaml` | 当前 Story 独占 |
| forbidden | `.input/`、`delivery/`、`src/atomic_ops/cli.py` | 禁止复制参考实现或新增执行命令 |

### 文件系统布局

```text
atoms/fw/
├── fw_install_ngfw_image.yaml
├── fw_init_ngfw_minimal.yaml
├── fw_login_web_management.yaml
├── fw_check_login_state.yaml
└── fw_verify_ngfw_health.yaml
packages/
└── ngfw_installation.yaml
```

### AI 可执行任务清单

| TASK-ID | 动作 | 目标文件 | 描述 |
|---|---|---|---|
| S002-T1 | 创建 | `atoms/fw/fw_install_ngfw_image.yaml` | 定义安装契约，输入为非敏感安装包引用、设备型号、完整性校验引用；输出安装完成待验证/失败状态。 |
| S002-T2 | 创建 | `atoms/fw/fw_init_ngfw_minimal.yaml` | 定义初始化契约，固定关闭验证码、配置管理路由、统一密码策略 `Ngfw@123` 三项动作。 |
| S002-T3 | 创建 | `atoms/fw/fw_login_web_management.yaml` | 定义 Web 登录契约，使用 `credential_ref`，返回脱敏 `session_ref`/`state_ref`。 |
| S002-T4 | 创建 | `atoms/fw/fw_check_login_state.yaml` | 定义配置前登录态守卫，状态缺失/过期时输出重登需求，最大重登 1 次。 |
| S002-T5 | 创建 | `atoms/fw/fw_verify_ngfw_health.yaml` | 定义 VPP、Agent、HTTPS 健康检查契约，失败只输出诊断引用和人工处理信号。 |
| S002-T6 | 创建 | `packages/ngfw_installation.yaml` | 以 op_id 引用安装、初始化、登录、守卫、健康检查 atom，不复制 atom 内容。 |
| S002-T7 | 校验 | `atoms/fw/*.yaml`、`packages/ngfw_installation.yaml` | 执行 schema、layout、package 引用和敏感信息检查。 |

## 验证上下文（validation_context）

- **验证入口**：`uv run --python 3.11 python scripts/validate_schema.py atoms/fw/fw_install_ngfw_image.yaml ...`；`uv run --python 3.11 python scripts/layout_check.py`；`uv run atomic-ops validate --package ngfw_installation`。
- **验证方式**：schema 校验、package op_id 可解析、security gate 检查、人工审查 atom 不含真实设备动作实现。
- **依赖环境**：Story-001 的 schema v1.1 契约已确认。
- **关键验证场景**：缺少 `gate` 的 high-risk 安装/init/login atom 失败；登录状态返回不含 token/cookie/password；安装健康检查失败不含自动回滚动作。

## 量化验收标准（acceptance_criteria）

- [ ] **atom 数量**：新增 5 个 `atoms/fw/fw_*.yaml`，文件名与 `op_id` 一致率 100%。
- [ ] **package 引用**：`packages/ngfw_installation.yaml` 引用 5 个 op_id，缺失引用数为 0。
- [ ] **high-risk gate**：安装、初始化、登录、守卫 atom 中 high-risk gate 覆盖率为 100%。
- [ ] **登录态边界**：登录/守卫 atom 中原始密码、完整 token、cookie、真实设备地址数量为 0。
- [ ] **初始化边界**：SSH/license 等未确认动作出现次数为 0。
- [ ] **健康检查覆盖**：VPP、Agent、HTTPS 三类检查均有参数或 verification 规则，覆盖率 3/3。
- [ ] **命名规范**：所有新增 op_id 匹配 `^(fw|tg|mock|sw)_[a-z]+_[a-z_]+$` 且前缀为 `fw_`。

## 回滚策略

如安装链路 atom 导致 package 或 schema 校验失败，回退本 Story 新增 atom 和 `packages/ngfw_installation.yaml`；不回退 STORY-001 契约，除非失败根因是 schema 字段设计错误。

## 边界说明

本 Story 不连接串口、不执行安装、不复制 `.input/ngfw-install` 脚本，不新增 CLI 执行命令。

## 阻塞说明（如有）

（无）
