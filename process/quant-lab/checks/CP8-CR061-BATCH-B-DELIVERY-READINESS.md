# CP8-CR061 Batch B Delivery Readiness 自动预检

## Entry Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| 目标 Story 已验证 | PASS_WITH_RISK | `process/checks/CP7-CR061-BATCH-B-PACKAGE-IMPORT-LAYOUT-VERIFICATION-DONE.md` | CP7 PASS_WITH_RISK。 |
| 发布上下文胶囊存在 | PASS | `process/release/RELEASE-CONTEXT-CR061.yaml` | release_decision=READY_WITH_RISK。 |
| 发布 profile 已判定 | PASS | release context | full profile。 |
| 发布就绪产物存在 | PASS | `docs/release/*-CR061.md` | 五份文档已生成。 |

## Checklist

| # | 检查项 | 状态 | 证据 | 处理意见 |
|---|---|---|---|---|
| 1 | 需求闭环 | PASS | CP5/CP6/CP7 evidence | Batch B scope closed。 |
| 2 | Story 闭环 | PASS_WITH_RISK | CP7 | full pytest 风险待 CP8 接受。 |
| 3 | 文档齐套 | PASS | release docs | CR061-specific release docs complete。 |
| 4 | 安装验证通过 | N/A | no install change | 不涉及安装器。 |
| 5 | 平台规则一致 | PASS | no platform rule change | 未改 AGENTS / CLAUDE。 |
| 6 | 交付目录合规 | PASS | docs/process only | 无 delivery package 改动。 |
| 7 | 缓存和临时文件清理 | PASS | validation commands | 使用 no cache / tmp pycache。 |
| 8 | guardrail 通过 | PASS | final checks | human gate / CR tracking / diff check PASS。 |
| 9 | 风险和遗留问题明确 | PASS | release context | R-CR061B-V01..03。 |
| 10 | Release Context Capsule 完整 | PASS | `process/release/RELEASE-CONTEXT-CR061.yaml` | 摘要和路径引用。 |
| 11 | 发布 profile 合法 | PASS | full | package migration slice。 |
| 12 | 发布结论合法 | PASS | READY_WITH_RISK | 不写 RELEASED。 |
| 13 | 发布就绪产物完整 | PASS | release docs | 五份 CR-specific docs。 |
| 14 | 版本号决策完成 | PASS | release context / notes | no-version-bump。 |
| 15 | 安装升级矩阵完成 | N/A | no install change | N/A。 |
| 16 | 迁移兼容性判断完成 | PASS | `docs/release/MIGRATION-CR061.md` | compatibility-first。 |
| 17 | 发布后观察计划完成 | PASS | `docs/release/FEEDBACK-CR061.md` | signals and routing。 |
| 18 | 用户终验确认 | PENDING | CP8 checkpoint | 等待用户。 |

## Exit Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| 自动预检通过 | PASS_WITH_RISK | 本文件 | 风险进入 CP8 DQ。 |
| 发布就绪可判定 | PASS | READY_WITH_RISK | 可发起人工终验。 |
| 人工终验通过 | PENDING | CP8 checkpoint | 等待用户 approve / 修改 / reject。 |

## Deliverables

| 交付物 | 路径 | 状态 | 说明 |
|---|---|---|---|
| Release context | `process/release/RELEASE-CONTEXT-CR061.yaml` | PASS | full profile。 |
| Release notes | `docs/release/RELEASE-NOTES-CR061.md` | PASS | ready。 |
| Deploy checklist | `docs/release/DEPLOY-CHECKLIST-CR061.md` | PASS | ready。 |
| Rollback | `docs/release/ROLLBACK-CR061.md` | PASS | ready。 |
| Migration | `docs/release/MIGRATION-CR061.md` | PASS | ready。 |
| Feedback | `docs/release/FEEDBACK-CR061.md` | PASS | ready。 |

## 结论

- 结论：`PASS_WITH_RISK`
- 阻断项：0
- 豁免项：full pytest 未运行；安装器验证 N/A。
- 下一步：发起 CP8 人工终验。
