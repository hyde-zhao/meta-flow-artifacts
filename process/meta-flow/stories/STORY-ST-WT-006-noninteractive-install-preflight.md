---
story_id: "ST-WT-006"
title: "提供三平台非交互安装与 preflight 入口"
story_slug: "noninteractive-install-preflight"
cr_id: "CR-047"
status: "verified-with-risk"
priority: "P1"
wave: "W3"
lld_wave: "LW3"
qa_wave: "QW5"
depends_on: ["ST-WT-002", "ST-WT-004", "ST-WT-005"]
dependency_contracts: [{story_id: "ST-WT-002", type: "contract", lld_gate: "declared-contract", dev_gate: "upstream-contract-frozen"}, {story_id: "ST-WT-004", type: "contract", lld_gate: "declared-contract", dev_gate: "upstream-contract-frozen"}, {story_id: "ST-WT-005", type: "runtime", lld_gate: "declared-contract", dev_gate: "upstream-verified"}]
feature_refs: ["FEAT-WT-05"]
feature_design_refs: ["process/docs/features/cr047-operator-status/DESIGN.md", "process/docs/features/cr047-operator-status/TEST-PLAN.md", "process/docs/features/cr047-operator-status/TASKS.md", "process/docs/features/cr047-delivery/DESIGN.md"]
lld_policy:
  required_level: "full-lld"
  trigger_reasons: ["multi-platform", "installer-cli", "release-preflight"]
  rationale: "Three platform examples must remain aligned with the canonical installer and release gate contract."
  waiver_reason: ""
  revisit_condition: "Official platform contract changes installation command or path."
  evidence_path: "process/stories/STORY-ST-WT-006-noninteractive-install-preflight-LLD.md"
file_ownership:
  primary: ["README.md", "delivery/README.md", "delivery/doc/USER-MANUAL.md", "tests/test_cr047_operator_status.py"]
  shared: ["meta_flow/cli.py", "delivery/scripts/install.py", "scripts/check_delivery_guardrails.py"]
  merge_owner: "ST-WT-006"
  forbidden: ["real user install", "credentials/runtime/SaaS", "repository commit/push", "backup/**", "process/quant-lab/**"]
lld_gate: {required_inputs: ["accepted HLD", "accepted ADR", "CR047-FEATURE-DESIGN-MATRIX", "Feature design", "ST-WT-002/004 contracts", "ST-WT-005 verified preflight"], design_evidence_type: "full-lld", design_evidence_path: "process/stories/STORY-ST-WT-006-noninteractive-install-preflight-LLD.md", status: "approved"}
implementation_gate: {evidence_required: true, evidence_path: "process/stories/STORY-ST-WT-006-noninteractive-install-preflight-IMPLEMENTATION.md", evidence_type: "implementation-md", status: "passed"}
verification_gate: {validation_mode: "dry-run-only", cp7_result: "PASS_WITH_RISK", remaining_risks: [], status: "verified-with-risk"}
dev_gate: {design_evidence_confirmed: true, lld_confirmed: true, dependencies_satisfied: true, file_conflict_free: true}
task_count: 2
created_at: "2026-07-14T15:40:00Z"
updated_at: "2026-07-15T13:22:00Z"
---

## 目标

让 Codex、Claude Code、Qoder 的项目级安装示例在非交互环境中可复制执行，并提供一个不扩大权限的标准 preflight 入口。

## 开发上下文（dev_context）

| 项目 | 内容 |
|---|---|
| 输入 | REQ-WT-013/015..017、平台契约、ST-WT-002/004/005 输出 |
| 输出 | 三平台 `--project-dir . --dry-run` 示例、标准 preflight、故障排除说明 |
| 设计约束 | 只做 dry-run/fixture；不写用户目录；示例统一使用 `uv run --python 3.11` |
| 平台 | codex、claude、qoder 共 3 个目标 |

### 依赖与文件所有权

文档必须消费最终 routing、guardrail 与 preflight 命令，不得另造第四个入口。共享 CLI/installer 若无需改动应保持只读。

### AI 可执行任务清单

| TASK-ID | 动作 | 目标 | 描述 |
|---|---|---|---|
| TASK-WT-006-01 | 修改 | README/USER-MANUAL | 补齐三平台非交互 dry-run 示例 |
| TASK-WT-006-02 | 修改/验证 | preflight/docs tests | 提供标准命令、失败语义和可复制证据 |

## 技术说明

设计证据类型为 full-lld；正式证据见 `process/stories/STORY-ST-WT-006-noninteractive-install-preflight-LLD.md`。

## 实现执行上下文

CP5 尚未批准；不会执行真实安装。实现阶段也只允许临时目录或 dry-run。

## 验证上下文

对三平台分别执行 project scope、full component、显式 project-dir 的非交互 dry-run；另验证缺少 project-dir 的可解释失败。

## 量化验收标准

- [ ] Codex/Claude/Qoder dry-run=3/3 通过。
- [ ] 三套示例均显式包含 `--project-dir .` 与 `--dry-run`。
- [ ] preflight 覆盖 5 个发布门且保留各门退出码。
- [ ] 真实用户目录写入次数=0，凭据/外部运行次数=0。

## 阻塞说明

平台契约或最终 preflight 命令未稳定时不得发布示例；需要真实安装或凭据时转授权请求。
