---
story_id: "STORY-005"
title: "Add read-only security gate and validation checks"
story_slug: "add-read-only-security-gate-and-validation-checks"
status: "verified"
priority: "P0"
wave: "W2"
depends_on: ["STORY-001", "STORY-002", "STORY-003", "STORY-004"]
dependency_contracts:
  - "schema v1.1 fields"
  - "NGFW atom/package catalog"
  - "batch contract"
dependency_type: ["contract"]
file_ownership:
  primary:
    - "scripts/security_gate_check.py"
  shared:
    - "src/atomic_ops/commands/validate.py"
    - "src/atomic_ops/commands/show.py"
    - "src/atomic_ops/commands/list_ops.py"
    - "src/atomic_ops/cli.py"
  merge_owner: "STORY-005"
  forbidden:
    - ".input/"
    - "delivery/"
lld_gate:
  required_inputs:
    - "process/HLD.md"
    - "process/ARCHITECTURE-DECISION.md"
    - "process/stories/STORY-005-add-read-only-security-gate-and-validation-checks.md"
  upstream_contracts: ["STORY-001", "STORY-002", "STORY-003", "STORY-004"]
  status: "ready_by_CR-003"
dev_gate:
  lld_confirmed: true
  dependencies_satisfied: true
  dependency_condition: "STORY-001, STORY-002, STORY-003, and STORY-004 CP6 PASS; upstream contracts stable; no real device action command added."
  file_conflict_free: true
  cp5_confirmed_required: true
  cp5_confirmed: true
  upstream_contract_evidence:
    - "process/checks/CP6-STORY-001-freeze-schema-v11-contract-and-field-docs-CODING-DONE.md"
    - "process/checks/CP6-STORY-002-model-ngfw-install-init-login-guard-atoms-CODING-DONE.md"
    - "process/checks/CP6-STORY-003-model-capacity-ten-domain-config-and-verification-atoms-CODING-DONE.md"
    - "process/checks/CP6-STORY-004-model-multi-device-batch-configuration-contract-CODING-DONE.md"
  implementation_complete: true
  cp6_status: "PASS"
  verification_complete: true
  cp7_status: "PASS"
  verification_evidence: "process/checks/CP7-STORY-005-add-read-only-security-gate-and-validation-checks-VERIFICATION-DONE.md"
  security_gate_facts_stable: true
  downstream_stable_facts:
    - "security gate command: uv run --python 3.11 python scripts/security_gate_check.py"
    - "default scan paths: atoms/, packages/, docs/, schemas/, scripts/, src/atomic_ops/"
    - "exclusions include .input/, .git/, .venv/, __pycache__, pytest/mypy/ruff caches, and generated cache dirs"
    - "exit codes: 0 success, 31 sensitive finding, 32 high-risk gate finding, 33 input/path/YAML/schema error"
    - "current catalog result: text_files=66, yaml_files=44, sensitive_findings=0, gate_findings=0"
    - "high-risk catalog result: high_risk_atoms=25, gate_failures=0"
    - "CLI boundary unchanged: no run/execute/apply/configure real device action command"
    - "read-only boundary: no connection libraries, no atom execution, no runtime .input read"
  implementation_evidence:
    - "process/checks/CP6-STORY-005-add-read-only-security-gate-and-validation-checks-CODING-DONE.md"
task_count: 7
created_at: "2026-05-18T14:35:00+0800"
updated_at: "2026-05-18T18:32:39+0800"
---

# STORY-005 Add read-only security gate and validation checks

## 目标

新增或等价实现只读安全 gate 和校验增强，确保敏感信息、high-risk gate、命名、package 引用和 CLI 命令边界可被机器检查。

## 开发上下文（dev_context）

- **背景说明**：HLD v1.1 要求最小机器校验入口，敏感信息失败退出码 31，high-risk gate 失败退出码 32，输入错误退出码 33。
- **输入文件**：`process/HLD.md` §8，`process/ARCHITECTURE-DECISION.md` ADR-002/003/006，STORY-001..004 产物。
- **输出文件**：`scripts/security_gate_check.py`；可对 `src/atomic_ops/commands/validate.py`、`show.py`、`list_ops.py`、`cli.py` 做只读展示/校验增强。
- **接口约定**：安全 gate 默认检查 `atoms/`、`packages/`、`docs/`、`schemas/`、`scripts/`、`src/atomic_ops/`，排除 `.git/`、`.venv/`、缓存目录和 `.input/`。
- **设计约束**：不得新增 `run/execute/apply/configure` 等真实设备动作命令；不得连接设备、读取真实凭据或访问网络。
- **命名规范**：脚本名为 `security_gate_check.py`，命令入口 `uv run --python 3.11 python scripts/security_gate_check.py`。
- **平台目标**：`scripts/` 和只读 CLI。

### 依赖与并行门控

| 上游 Story | 类型 | LLD 门控 | 开发门控 | 说明 |
|---|---|---|---|---|
| STORY-001 | contract | 需要 schema 字段和敏感互斥规则 | CP5 confirmed | gate 校验字段来源 |
| STORY-002 | contract | 需要安装/登录 atom 样例 | CP5 confirmed | high-risk/login 检查对象 |
| STORY-003 | contract | 需要 capacity atom 样例 | CP5 confirmed | 10 域检查对象 |
| STORY-004 | contract | 需要 batch atom 样例 | CP5 confirmed | 批次并发和失败隔离检查对象 |

### 文件所有权

| 类型 | 文件 | Owner / 合并规则 |
|---|---|---|
| primary | `scripts/security_gate_check.py` | 当前 Story 独占 |
| shared | `src/atomic_ops/commands/validate.py` | 若增强 package/schema 引用校验，由 STORY-005 合并 |
| shared | `src/atomic_ops/commands/show.py`、`list_ops.py` | 若脱敏展示新增字段，由 STORY-005 合并 |
| shared | `src/atomic_ops/cli.py` | 仅用于证明不新增真实动作命令或增加只读校验参数 |
| forbidden | `.input/`、`delivery/` | 禁止读取运行时资产或写安装包目录 |

### AI 可执行任务清单

| TASK-ID | 动作 | 目标文件 | 描述 |
|---|---|---|---|
| S005-T1 | 创建 | `scripts/security_gate_check.py` | 实现敏感信息扫描、high-risk gate 校验、输入错误处理。 |
| S005-T2 | 校验 | `scripts/security_gate_check.py` | 失败退出码：敏感信息 31、high-risk gate 32、输入错误 33。 |
| S005-T3 | 修改 | `src/atomic_ops/commands/validate.py` | 如需要，增强只读 validate 对 package/schema 引用的报告，不新增设备动作。 |
| S005-T4 | 修改 | `src/atomic_ops/commands/show.py` / `list_ops.py` | 如需要，对 `session_ref` / `state_ref` 示例做脱敏展示。 |
| S005-T5 | 校验 | `src/atomic_ops/cli.py` | 确认命令集不包含 `run/execute/apply/configure`。 |
| S005-T6 | 扫描 | repo | 确认 `.input/` 未作为扫描输入复制源，真实敏感值未落盘。 |
| S005-T7 | 记录 | LLD/CP6 | 记录安全 gate 的规则清单、排除路径和误报处理方式。 |

## 验证上下文（validation_context）

- **验证入口**：`uv run --python 3.11 python scripts/security_gate_check.py`；`uv run atomic-ops --help`；`uv run atomic-ops validate --package <package>`。
- **验证方式**：脚本退出码测试、CLI help smoke test、敏感模式 fixture、high-risk gate fixture、package 引用校验。
- **依赖环境**：STORY-001..004 产物可供扫描。
- **关键验证场景**：明文 token 触发退出 31；high-risk atom 缺少 gate 触发退出 32；不存在路径触发退出 33；CLI help 不出现真实动作命令。

## 量化验收标准（acceptance_criteria）

- [ ] **命令入口**：`uv run --python 3.11 python scripts/security_gate_check.py` 可执行，正常仓库返回 0。
- [ ] **敏感失败**：含 `token/cookie/authorization/password/ftp_pass/secret` 明文 fixture 返回 31，且不回显完整敏感值。
- [ ] **gate 失败**：high-risk atom 缺少 `gate.required=true` 或 `gate.reason` 为空返回 32。
- [ ] **输入错误**：路径不存在、YAML 解析失败或 schema 缺失返回 33。
- [ ] **命令边界**：`atomic-ops --help` 中新增真实设备动作命令数量为 0。
- [ ] **只读边界**：新增脚本和 CLI 增强均不连接设备、不访问网络、不写 `.input/`。
- [ ] **覆盖对象**：默认扫描对象包含 `atoms/packages/docs/schemas/scripts/src/atomic_ops` 6 类路径。

## 回滚策略

若安全 gate 误报导致无法通过现有仓库，回退脚本或缩小规则到 HLD 最小契约；不得通过删除 high-risk gate 要求或允许明文凭据来放行。

## 边界说明

本 Story 不执行真实设备动作，不新增设备 executor，不新增 `run/execute/apply/configure` 命令。

## 阻塞说明（如有）

（无）
