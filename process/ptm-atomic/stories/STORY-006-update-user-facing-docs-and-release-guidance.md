---
story_id: "STORY-006"
title: "Update user-facing docs and release guidance"
story_slug: "update-user-facing-docs-and-release-guidance"
status: "verified"
priority: "P1"
wave: "W3"
depends_on: ["STORY-001", "STORY-002", "STORY-003", "STORY-004", "STORY-005"]
dependency_contracts:
  - "all delivered contracts and validation commands"
dependency_type: ["runtime"]
file_ownership:
  primary:
    - "README.md"
    - "docs/USER-MANUAL.md"
    - "docs/engineer-handbook.md"
    - "docs/test-case-template.yaml"
    - "CHANGELOG.md"
  shared: []
  merge_owner: "STORY-006"
  forbidden:
    - ".input/"
    - "delivery/"
lld_gate:
  required_inputs:
    - "process/HLD.md"
    - "process/ARCHITECTURE-DECISION.md"
    - "process/PLATFORM-INSTALL-SPEC.md"
    - "process/stories/STORY-006-update-user-facing-docs-and-release-guidance.md"
  upstream_contracts: ["STORY-001", "STORY-002", "STORY-003", "STORY-004", "STORY-005"]
  status: "ready_by_CR-003"
dev_gate:
  lld_confirmed: true
  dependencies_satisfied: true
  dependency_condition: "STORY-001..005 CP6 and CP7 are PASS; upstream implementation and verification evidence is available; docs must reflect actual delivered files and verified behavior."
  file_conflict_free: true
  cp5_confirmed_required: true
  cp5_confirmed: true
  implementation_evidence_available: true
  implementation_complete: true
  cp6_status: "PASS"
  implementation_evidence: "process/checks/CP6-STORY-006-update-user-facing-docs-and-release-guidance-CODING-DONE.md"
  verification_evidence_available: true
  verification_complete: true
  cp7_status: "PASS"
  verification_evidence: "process/checks/CP7-STORY-006-update-user-facing-docs-and-release-guidance-VERIFICATION-DONE.md"
  upstream_verification_evidence:
    - "process/checks/CP7-STORY-001-freeze-schema-v11-contract-and-field-docs-VERIFICATION-DONE.md"
    - "process/checks/CP7-STORY-002-model-ngfw-install-init-login-guard-atoms-VERIFICATION-DONE.md"
    - "process/checks/CP7-STORY-003-model-capacity-ten-domain-config-and-verification-atoms-VERIFICATION-DONE.md"
    - "process/checks/CP7-STORY-004-model-multi-device-batch-configuration-contract-VERIFICATION-DONE.md"
    - "process/checks/CP7-STORY-005-add-read-only-security-gate-and-validation-checks-VERIFICATION-DONE.md"
task_count: 6
created_at: "2026-05-18T14:35:00+0800"
updated_at: "2026-05-19T09:08:28+0800"
---

# STORY-006 Update user-facing docs and release guidance

## 目标

在上游 schema、atom、package、批次契约和安全 gate 稳定后，更新用户可见文档和发布记录，确保 README 与实际交付能力一致。

## 开发上下文（dev_context）

- **背景说明**：本 Story 是文档收口，不提前承诺未实现能力。文档必须清楚说明 CLI 仍只读，不执行真实设备安装或配置。
- **输入文件**：STORY-001..005 的 LLD、实现和验证结果，`process/PLATFORM-INSTALL-SPEC.md`，README 当前安装/命令事实。
- **输出文件**：`README.md`、`docs/USER-MANUAL.md`、`docs/engineer-handbook.md`、`docs/test-case-template.yaml`、`CHANGELOG.md`。
- **接口约定**：文档中的命令示例使用 `uv run` 或 `uv tool`；新增 package 查询示例只使用 `atomic-ops list/show/packages/validate`。
- **设计约束**：不得声称 CLI 可执行设备动作；不得写真实设备 IP、token、cookie、FTP 凭据、原始默认密码；唯一允许显式出现的密码策略值为 `Ngfw@123`。
- **命名规范**：文档中的 op_id 与已交付 atom 一致；不得引用未实现 op_id。
- **平台目标**：README 原生交付面，无独立安装包。

### 依赖与并行门控

| 上游 Story | 类型 | LLD 门控 | 开发门控 | 说明 |
|---|---|---|---|---|
| STORY-001..005 | runtime | 可提前设计文档结构 | 上游实现和验证完成 | 用户文档必须反映实际文件和命令输出。 |

### 文件所有权

| 类型 | 文件 | Owner / 合并规则 |
|---|---|---|
| primary | `README.md` | 当前 Story 独占 |
| primary | `docs/USER-MANUAL.md` | 当前 Story 独占 |
| primary | `docs/engineer-handbook.md` | 当前 Story 独占 |
| primary | `docs/test-case-template.yaml` | 当前 Story 独占 |
| primary | `CHANGELOG.md` | 当前 Story 独占 |
| forbidden | `.input/`、`delivery/` | 禁止复制参考资产或写安装包目录 |

### AI 可执行任务清单

| TASK-ID | 动作 | 目标文件 | 描述 |
|---|---|---|---|
| S006-T1 | 修改 | `README.md` | 更新已交付能力、目录结构、核心命令和安全边界，只描述已实现内容。 |
| S006-T2 | 修改 | `docs/USER-MANUAL.md` | 增加安装/配置/验证 package 查询与引用说明，强调不执行设备动作。 |
| S006-T3 | 修改 | `docs/engineer-handbook.md` | 增加贡献者如何新增 v1.1 atom、配置域和 batch 契约。 |
| S006-T4 | 修改 | `docs/test-case-template.yaml` | 更新测试用例引用示例，不包含真实凭据或设备地址。 |
| S006-T5 | 修改 | `CHANGELOG.md` | 记录 schema、atom、package、CLI/scripts 和 docs 变化。 |
| S006-T6 | 校验 | docs/README | 执行命令示例、敏感信息扫描和 op_id 引用一致性检查。 |

## 验证上下文（validation_context）

- **验证入口**：`uv run atomic-ops --help`；`uv run atomic-ops packages`；`uv run atomic-ops validate --package <package>`；`uv run --python 3.11 python scripts/security_gate_check.py`。
- **验证方式**：文档命令 smoke test、链接/路径人工审查、敏感信息扫描、README 与实际 CLI 命令一致性检查。
- **依赖环境**：STORY-001..005 已实现且验证通过。
- **关键验证场景**：文档不出现未实现 package/op_id；所有 Python 命令示例使用 uv；不出现 `delivery/` 作为本项目交付目录。

## 量化验收标准（acceptance_criteria）

- [ ] **文档文件数量**：5 个目标文档完成更新，缺失数为 0。
- [ ] **命令一致性**：README 核心命令仍只包含 `sync/list/show/packages/validate`，真实设备动作命令数量为 0。
- [ ] **uv 一致性**：Python/脚本示例使用 `uv run` 或 `uv tool`，裸 `pip install` 默认入口数量为 0。
- [ ] **交付面一致性**：README 正向交付目录只包含原生交付面，`delivery/` 正向引用数量为 0。
- [ ] **敏感信息**：真实 IP、token、cookie、FTP 凭据、原始默认密码数量为 0。
- [ ] **op_id 可解析**：文档示例引用的新增 op_id 在 atom catalog 中存在，缺失引用数为 0。
- [ ] **安全边界**：文档明确 CLI 不连接设备、不执行 atom、不保存凭据。

## 回滚策略

若文档提前承诺未实现能力或示例命令失败，回退相关文档段落到上一个稳定版本；保留已实现代码和 atom，不回退上游 Story。

## 边界说明

本 Story 不新增功能代码，不修改 schema/atom/package 契约；只在上游验证完成后更新文档。

## 阻塞说明（如有）

（无）
