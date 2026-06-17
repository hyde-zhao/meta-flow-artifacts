---
cr_id: "CR-090"
status: "closed-local-remediation"
impact_level: "medium"
workflow_mode_before: "standard"
workflow_mode_after_change: "standard"
fast_lane_upgrade_reason: "涉及源码 docs 文件所有权与归档边界收敛，按 standard 记录。"
rollback_to: "CR088 artifact root and CR087 docs archive route baseline"
approval_result: "completed"
created_at: "2026-06-17T22:10:00+08:00"
created_by: "host-orchestrator"
approved_by: "user"
approved_at: "2026-06-17T22:10:00+08:00"
source: "user"
parent_cr: "CR-087"
source_decision_id: "USER-20260617-DOCS-PUBLIC-SURFACE-GOVERNANCE"
follow_up_type: "docs-public-surface-governance"
risk_class: "medium"
owner: "host-orchestrator"
revisit_condition: "source repo docs again contains process/design/quality/CR contract/release checklist/research governance documents"
acceptance_criteria: "quant-lab Git tracks only component user manuals, install/runbooks, quickstarts/reference manuals and release notes; archived docs exist under meta-flow-artifacts process namespace"
close_condition: "source docs retained list is verified; archive files exist; no data/reports/credential/NAS/runtime operation executed"
cr_index_path: "process/changes/CR-INDEX.yaml"
---

# CR090 Docs Public Surface Governance

## 变更描述

用户指出源码仓库 `docs/` 仍需治理：该归档到 `meta-flow-artifacts` 的文档继续归档，当前项目只保留各组件的发布说明书和用户手册。

本 CR 是 CR087 / CR088 后续的源码文档面收敛，不改变业务代码、运行授权、数据湖、NAS、QMT / MiniQMT 或凭据边界。

## 保留边界

源码仓库 `docs/` 只保留以下类型：

| 类型 | 例子 | 说明 |
|---|---|---|
| 项目用户手册 | `docs/USER-MANUAL.md` | 当前 production 项目的主用户入口 |
| 组件运行 / 安装手册 | `docs/QMT-*.md`、`docs/goldminer/CR045-BRIDGE-RUNBOOK.md` | 用户可读 runbook、install、incident / activation 手册 |
| 组件 quickstart / reference manual | `docs/CR030-FACTOR-RESEARCH-QUICKSTART.md`、`docs/CR025-BACKTRADER-MODULE-REFERENCE.md` | 操作入口和组件参考说明 |
| 发布说明 | `docs/release/RELEASE-NOTES.md` | 只保留 release notes；部署、迁移、回滚和反馈过程文档归档 |

## 归档边界

以下类型从源码 `docs/` 移到：

```text
/home/hyde/workspace/meta-flow-artifacts/process/quant-lab/docs/source-archive/docs
```

| 类型 | 处理 |
|---|---|
| roadmap / backfill roadmap | 归档 |
| CR register / contract / semantic alignment | 归档 |
| QMT / MiniQMT design-only framework docs | 归档 |
| release deploy checklist / migration / rollback / feedback | 归档 |
| research lifecycle / archive governance / registry specs | 归档 |

## 文档处理决策

| 受影响对象 | 处理方式 | 旧基线保留方式 | 批准状态 |
|---|---|---|---|
| `docs/` public surface | 原文档更新 / 删除已归档文件 | 已归档到 `process/docs/source-archive/docs/` | approved |
| `README.md` | 原文档更新 | 保留摘要，旧详细过程链接改归档路径 | approved |
| `docs/USER-MANUAL.md` | 原文档更新 | 保留用户边界，旧专题链接改归档路径 | approved |
| `docs/QMT-C-S-BRIDGE-RUNBOOK.md` | 原文档更新 | 保留用户摘要，详细 register 改归档路径 | approved |
| `docs/CR030-FACTOR-RESEARCH-QUICKSTART.md` | 原文档更新 | 保留 quickstart，边界 / reference matrix 改归档路径 | approved |
| `docs/release/RELEASE-NOTES.md` | 原文档更新 | 保留 release notes，部署 / 回滚 / 迁移引用改归档路径 | approved |
| `tests/test_cr0*_*.py` 文档 guardrail | 原文档更新 | guardrail 读取归档路径而不是源码 `docs/` | approved |
| `LEDGER.md` / `ledger.yaml` | 原文档更新 | 记录 CR090 docs public surface policy | approved |

## 五维度影响分析

| 维度 | 结论 | 处理动作 |
|---|---|---|
| 需求层 | 不改变业务需求 | 不修改 requirements |
| 场景层 | 不改变验证场景 | 仅调整历史 guardrail 文档读取路径 |
| 计划层 | 改变文档归档路由 | 源码 `docs/` 收窄，artifact repo 增加 `docs/source-archive` |
| 安全层 | 不新增运行权限 | 不读取 `.env`、凭据、data/reports 内容，不执行 QMT / MiniQMT / NAS runtime |
| 交付层 | 改变源码文档发布面 | `docs/` 只保留用户手册 / 组件手册 / release notes |

## 禁止动作

- 不读取或迁移 `data/`、`reports/` 内容。
- 不读取 `.env`、token、password、cookie、session、private key、账户、持仓或 broker 私有路径。
- 不执行 NAS 内容操作、provider fetch、lake write、publish、QMT / MiniQMT / XtQuant、simulation 或 live。
- 不删除 `meta-flow-artifacts` 归档文件。

## 验收结果

| 检查项 | 结果 | 证据 |
|---|---|---|
| 源码 `docs/` 保留清单 | PASS | 仅保留 10 个用户 / release 文档 |
| 归档文件存在 | PASS | `process/docs/source-archive/docs/` 下保留原相对路径 |
| 旧源码路径引用 | PASS_WITH_RISK | README / 用户文档改为归档路径；历史测试改读归档路径 |
| 禁止动作 | PASS | 未执行 data/reports/credential/NAS/runtime 操作 |

## 处理结论

CR090 已关闭为 `closed-local-remediation`。
