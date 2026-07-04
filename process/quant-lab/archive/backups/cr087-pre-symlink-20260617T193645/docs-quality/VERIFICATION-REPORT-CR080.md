---
status: "verified-with-risk"
version: "1.0"
story_id: "CR080"
story_slug: "data-reports-restore"
feature_id: "migration-target-root-data-reports"
validation_mode: "mixed"
verification_result: "PASS_WITH_RISK"
source_story: "process/changes/CR-080-DATA-REPORTS-RESTORE-COPY-GATE-2026-06-17.md"
source_implementation: "process/checks/CR080-DATA-REPORTS-RESTORE-EXECUTION-2026-06-17.md"
created_by: "meta-qa"
created_at: "2026-06-17T07:14:53+08:00"
updated_at: "2026-06-17T07:14:53+08:00"
---

# Verification: CR080 Data/reports Restore

## 1. 结论

| 项目 | 内容 |
|---|---|
| 阶段决策 | PASS_WITH_RISK |
| validation_mode | mixed |
| 是否可进入下一阶段 | yes |
| 需要路由 | CP8 risk acceptance |
| CP7 证据 | `process/checks/CP7-CR080-DATA-REPORTS-RESTORE-VERIFICATION-DONE.md` |

## 2. 验证范围

| 项 | 内容 |
|---|---|
| Feature / Story | CR080 Data/reports restore or copy gate |
| 验证范围 | legacy retained assets 中 `reports/`、`data/` 复制到 authoritative target root 后的 metadata-only 结果验证 |
| 非范围 | 内容读取、样本解析、checksum/hash、NAS compare、provider rebuild、lake/catalog publish、remote Git、QMT/MiniQMT runtime、CR046 recovery、旧根删除/移动 |
| 上游设计 | `process/checkpoints/CP2-CR080-DATA-REPORTS-RESTORE-BASELINE.md` / `process/checkpoints/CP3-CR080-DATA-REPORTS-RESTORE-DESIGN-REVIEW.md` / `process/checkpoints/CP5-CR080-DATA-REPORTS-RESTORE-READINESS.md` |
| 实现摘要 | `process/checks/CR080-DATA-REPORTS-RESTORE-EXECUTION-2026-06-17.md` |
| 已接受风险 | R-CR080-001 metadata-only 不证明内容级一致性 |
| 阻塞条件 | N/A |

## 3. 验证对象清单

| 对象 | 类型 | 来源 / 变更原因 | 验证方式 | 是否阻塞 | 证据 |
|---|---|---|---|---|---|
| `process/changes/CR-080-DATA-REPORTS-RESTORE-COPY-GATE-2026-06-17.md` | state-process | 正式 CR | review | yes | PASS |
| `process/STATE.md` | state-process | copy executed fact 回写 | review | yes | PASS |
| `process/changes/CR-INDEX.yaml` | state-process | active CR 状态同步 | review | yes | PASS |
| `/home/hyde/workspace/quant-lab/reports/` | local asset | target root restore | metadata-only | yes | PASS |
| `/home/hyde/workspace/quant-lab/data/` | local asset | target root restore | metadata-only | yes | PASS |
| `process/checks/CR080-DATA-REPORTS-RESTORE-PREFLIGHT-2026-06-17.md` | process check | copy 前置检查 | review | yes | PASS |
| `process/checks/CR080-DATA-REPORTS-RESTORE-EXECUTION-2026-06-17.md` | process check | copy 执行证据 | review | yes | PASS |

## 4. 验证追踪矩阵

| Scenario | Requirement | Story | Design Contract | Implementation | Test / Check | Status | Risk |
|---|---|---|---|---|---|---|---|
| CR080-SCN-001 | target root 缺 `reports/` / `data/` | CR080 | legacy -> target local copy | `rsync -a` execution | source/target count and bytes match | PASS | N/A |
| CR080-SCN-002 | 不读取内容 / 不泄露敏感信息 | CR080 | denylist + metadata-only evidence | rsync excludes + no content commands | target denylist count 0 | PASS | 内容级一致性不证明 |
| CR080-SCN-003 | 不删除 / 不覆盖非空 target | CR080 | fail-closed + no `--delete` | execution stats deleted=0 | dry-run deleted=0 | PASS | N/A |
| CR080-SCN-004 | 外部系统不触碰 | CR080 | no NAS/provider/lake/catalog/remote/runtime | no external commands | review | PASS | 外部一致性未验证 |
| CR080-SCN-005 | 状态账本收敛 | CR080 | STATE / CR-INDEX reflect copy executed | STATE / CR-INDEX patch | dev rerun PASS | PASS | N/A |

## 5. 设计契约验证清单

| 契约 | 来源 | 验证方式 | 是否阻塞 | 结果 | 证据 |
|---|---|---|---|---|---|
| 必须 CP2/CP3/CP5 approved 后执行 | checkpoint-manager / CR080 DQ | review | yes | PASS | 三份 checkpoint approved |
| 必须 preflight PASS 后执行 copy | CR080 DQ-CP5-02 | review | yes | PASS | preflight PASS |
| 必须本地 legacy -> target | CR080 DQ-CP2-02 | metadata review | yes | PASS | source/target root matching |
| 不得 `--delete` | CR080 DQ-CP3-02 | execution review | yes | PASS | deleted=0，未使用 `--delete` |
| 不得 checksum / content read | CR080 DQ-CP2-03 / DQ-CP5-03 | command review | yes | PASS | no checksum / no parsing |
| credential-like path 排除 | CR080 DQ-CP3-03 | count review | yes | PASS | denylist count 0 |
| partial failure 不自动 cleanup | CR080 DQ-CP3-05 / DQ-CP5-04 | review | yes | PASS | 未触发 partial / cleanup |

## 6. 分层验证计划

| 验证层 | 方法 | 目标 | 触发条件 | 必跑 | 结果 | 未覆盖风险 |
|---|---|---|---|---|---|---|
| 静态检查 | `git diff --check` scoped | 状态和 CR 文档格式 | process 文件改动 | yes | PASS | N/A |
| 单元测试 | N/A | 本轮无代码单元 | 无代码变更 | no | N/A | N/A |
| 契约测试 | CR tracking / human gate checks | 台账和门禁一致 | CR 状态变更 | yes | PASS | N/A |
| 集成测试 | `rsync -an --stats` | 源/目标 quick-check 一致 | copy 完成 | yes | PASS | 不证明内容 hash |
| 平台 dry-run | N/A | 不涉及安装平台 | 无安装变更 | no | N/A | N/A |
| 回归测试 | 禁止项复核 | 不恢复 CR046、不触碰外部 | 高风险边界 | yes | PASS | N/A |
| 人工审查 | meta-dev/meta-qa 子 agent | 授权边界和风险 | CP6/CP7 | yes | PASS_WITH_RISK | 内容级一致性风险 |

## 7. 自动化验证结果

| Command ID | 命令 / 检查 | 结果 | 证据 | 说明 |
|---|---|---|---|---|
| CMD-01 | `uv run --python 3.11 python scripts/check_cr_tracking_consistency.py --project-root .` | PASS | `CR tracking consistency: PASS` | CR 台账一致。 |
| CMD-02 | `scripts/check_human_gate_decision_brief.py` for CP2/CP3/CP5 | PASS | each `decision_count=5` | 三门发起消息合规。 |
| CMD-03 | `git diff --check -- <CR080 files>` | PASS | no output | whitespace OK。 |
| CMD-04 | metadata-only `find` / `du` | PASS | CP7 summary | 源/目标 count and bytes match。 |
| CMD-05 | `rsync -an --stats` for reports/data | PASS | CP7 summary | 0 created / 0 deleted / 0 transferred。 |

## 8. Prompt / Skill Fixture 验证

| Fixture ID | 输入 / 场景 | 期望 | 实际 | 结果 | 证据 |
|---|---|---|---|---|---|
| N/A | 本轮不涉及 Prompt / Skill | N/A | N/A | N/A | N/A |

## 9. 平台适配验证

| 平台 | 检查项 | 预期 | 结果 | 证据 |
|---|---|---|---|---|
| Codex | 使用真实子 agent 复核 CP6/CP7 | yes | PASS | `dev-zhu` / `qa-cao` |
| install | project / user scope dry-run | n/a | N/A | 无安装变更 |

## 10. 人工 / 语义质量审查

| 检查项 | 结果 | 是否阻塞 | 说明 |
|---|---|---|---|
| 需求一致性 | PASS | no | 满足 CR080 restore/copy gate。 |
| 场景覆盖 | PASS_WITH_RISK | no | 覆盖 metadata-only restore；内容质量不覆盖。 |
| 文档可用性 | PASS | no | CR / STATE / CR-INDEX 已同步。 |
| 错误信息可行动 | PASS | no | fail-closed 和后续 CR 边界明确。 |
| 是否只覆盖 happy path | PASS | no | 覆盖 target 非空 fail-closed、禁止项和 partial cleanup 边界。 |

## 11. 问题清单

| ID | 等级 | 问题 | 影响 | 建议处理 | Owner | 状态 |
|---|---|---|---|---|---|---|
| Q-CR080-001 | INFO | 第一轮 CP6 发现 STATE 未回写 | 会阻断 CP6 | 已回写 STATE / CR-INDEX 后复核 | host-orchestrator | RESOLVED |

## 12. 剩余风险

| Risk ID | 风险 | 等级 | 是否接受 | 接受人 / 条件 | 后续处理 |
|---|---|---|---|---|---|
| R-CR080-001 | metadata-only 验证不证明内容 hash / 内容质量 | MEDIUM | yes, pending CP8 | 用户 CP8 approve | 如需内容级证明，另起内容验证 CR。 |
| R-CR080-002 | NAS/provider/lake/catalog 未验证 | LOW | yes, pending CP8 | 用户 CP8 approve | 仅在 legacy 资产不足或用户要求时启动独立候选。 |
| R-CR080-003 | 旧根未退役 | LOW | yes, pending CP8 | 用户 CP8 approve | CR077 / old root retirement 仍需独立门禁。 |

## 13. 质量评审与修复输入

| 产物 | 路径 | 结论 |
|---|---|---|
| TEST-REPORT | `docs/quality/TEST-REPORT-CR080.md` | PASS_WITH_RISK |
| REVIEW | `docs/quality/REVIEW-CR080.md` | approve-with-risk |
| FIXES | `docs/quality/FIXES-CR080.md` | none |

## 14. 阶段决策

| 结论 | 路由 | 条件 / 说明 |
|---|---|---|
| PASS_WITH_RISK | CP8 | 风险接受项进入 CP8 Decision Brief。 |

## 15. CP8 输入

| 输入项 | 内容 |
|---|---|
| 风险接受候选 | R-CR080-001 / R-CR080-002 / R-CR080-003 |
| 后续 CR 候选 | content validation CR（仅用户需要内容级证明时）；CR077 old root retirement；CR078 remote governance |
| 不授权项 | content read / checksum / sample parsing / credentials / NAS/provider/lake/catalog / remote Git / runtime / old root deletion |
| 发布准备关注点 | READY_WITH_RISK，不等于真实发布或旧根退役授权 |
