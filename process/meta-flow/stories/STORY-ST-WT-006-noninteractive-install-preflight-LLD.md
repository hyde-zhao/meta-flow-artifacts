---
story_id: "ST-WT-006"
title: "提供三平台非交互安装与 preflight 入口"
story_slug: "noninteractive-install-preflight"
lld_version: "1.0"
tier: "L"
status: "approved"
confirmed: true
created_by: "host-orchestrator-inline/meta-dev"
created_at: "2026-07-14"
confirmed_by: "user"
confirmed_at: "2026-07-15T12:05:17Z"
feature_design_refs: ["process/docs/features/cr047-operator-status/DESIGN.md", "process/docs/features/cr047-operator-status/TEST-PLAN.md", "process/docs/features/cr047-operator-status/TASKS.md", "process/docs/features/cr047-delivery/DESIGN.md"]
lld_policy: {required_level: "full-lld", trigger_reasons: ["multi-platform", "installer-cli", "release-preflight"]}
open_items: 0
---

# LLD: ST-WT-006 — Noninteractive Install Preflight

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-14 | host-orchestrator-inline/meta-dev | 冻结三平台 dry-run 示例与单一 preflight 用户入口。 |

## 0. 工程依据与上游设计依据

工程依据为 REQUIREMENTS、平台契约、Feature operator/delivery designs与当前 installer行为：project scope在非交互环境必须显式 `--project-dir`。本 Story消费 ST-WT-002 routing、ST-WT-004 guardrail、ST-WT-005 verified preflight。

## 1. 目标

让 Codex、Claude Code、Qoder 三个平台的首个项目安装示例在非交互终端3/3通过，并让 operator 能用一个稳定入口运行五项发布前置检查、看到每门结果和剩余风险。

## 2. 需求（Functional / Non-Functional）

- 三平台示例均使用 `uv run --python 3.11 meta-flow install <platform> --scope project --component full --project-dir . --dry-run`。
- 缺 `--project-dir` 的非交互调用保持明确非零失败和修复提示。
- README、delivery README、USER-MANUAL 命令与 canonical CLI一致，不复制另一套 orchestrator。
- 验证只用临时目录/dry-run，真实用户目录写入=0、凭据/外部 runtime=0。
- preflight展示 pytest/Ruff/guardrail/Doctor/CR tracking五门原始状态，不把 warning美化为全绿。

## 3. 模块拆分与职责

| 模块 | 职责 |
|---|---|
| `README.md` | 开发者首入口和三平台非交互 quick start |
| `delivery/README.md` | 交付包等价示例与约束 |
| `delivery/doc/USER-MANUAL.md` | 故障排除、warning/blocker解释 |
| `delivery/scripts/install.py`, `meta_flow/cli.py` | canonical installer/CLI；仅在契约缺口时最小修改 |
| `tests/test_cr047_operator_status.py` | 三平台 dry-run与文档命令一致性 fixture |

## 4. 代码结构与文件影响范围

主要修改三份用户文档和 operator status test；shared CLI/installer/guardrail默认只读。不得真实安装到用户目录、改凭据、运行SaaS或push。

## 5. 数据模型与持久化设计

`InstallExample(platform, scope, component, project_dir, dry_run)`；`PreflightDisplay(gate, status, exit_code, evidence_ref, risk_refs)`。文档命令由测试解析并与预期 argv比对，无新增持久化 truth。

## 6. API / Interface 设计

使用既有 `meta-flow install {codex|claude|qoder}` 和 ST-WT-005 preflight入口。调用时机为安装前dry-run/发布前检查；输入是显式project-dir；输出是计划或五门摘要。失败降级为修复提示，不自动改路径或请求凭据。

## 7. 核心处理流程

1. operator复制对应平台命令。
2. CLI验证非交互project-dir、platform/component/scope。
3. dry-run只渲染目标变更，不落盘。
4. preflight依次消费五门结果并展示blocker/warning/risk refs。
5. 失败时文档指向具体命令和边界；不建议绕过。

## 8. 技术细节与设计细节

文档测试从 fenced shell块提取命令，要求三平台集合精确相等且含两个强制flag。命令使用续行时也必须可拼接执行。平台路径解释引用 `delivery/doc/PLATFORM-CONTRACTS.yaml`，不类比推断。

## 9. 安全与性能设计

dry-run fixture设置隔离 HOME/临时project，断言用户目录无新增。无网络、凭据或production write。三平台命令加preflight运行时间只作回归观测，不设新SLA。

## 10. 测试设计

| 场景 | 预期 |
|---|---|
| codex/claude/qoder完整命令 | 3/3 exit 0、无落盘 |
| 缺 project-dir的非交互调用 | 非零且提示显式参数 |
| 文档命令解析 | 3个平台、scope/component/project-dir/dry-run齐全 |
| preflight一门失败 | 显示具体gate与exit，整体失败 |
| inherited risk | READY_WITH_RISK说明保留 |
| 隔离HOME前后清单 | 写入数=0 |

## 11. 实施步骤

1. `TASK-WT-006-01`：更新三份文档的三平台非交互示例和故障排除。
2. `TASK-WT-006-02`：补文档命令解析、3/3 dry-run、无写入和preflight展示测试。
3. 运行定向 tests和临时目录dry-run；记录命令/exit/evidence。

## 12. 风险、难点与预研建议

风险是文档漂移、dry-run仍写用户目录、把warning描述为READY。以命令解析fixture、隔离HOME快照、DQ-06结论上限解决。官方平台契约变化时回设计更新，而非静默改示例。

## 13. 回滚与发布策略

文档与tests同一变更发布；若installer contract回归，回滚示例到最后可执行版本并保持显式失败说明。无真实安装迁移或外部发布动作。

## 14. DoD（Definition of Done）

- [ ] 0–14 章节完整、`open_items=0`。
- [ ] 三平台非交互dry-run=3/3通过，强制flag覆盖=100%。
- [ ] 用户目录写入、凭据、外部runtime次数均为0。
- [ ] 五门状态/exit/risk refs完整，结论上限未美化。
- [ ] CP5 未批准前 `confirmed=false`，不开始实现。

## 人工确认区

- 结论：`approved`
- 审查人：user
- 审查时间：2026-07-15T12:05:17Z
- 修改意见：
- 风险接受项：inline fallback；CP7/CP8 风险上限。
