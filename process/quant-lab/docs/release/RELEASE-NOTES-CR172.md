---
status: cp8-approved-git-delivery-completed
version: "1.1"
release_artifact_profile: full
release_decision: READY_WITH_RISK
candidate_version: "CR-172-PATH-I-rc.1"
created_at: "2026-07-19T01:47:00+08:00"
updated_at: "2026-07-19T09:29:15+08:00"
---

# CR-172 PATH-I Release Notes

## 1. 摘要

| 项目 | 内容 |
|---|---|
| 版本 | `CR-172-PATH-I-rc.1`（内部交付候选标签，不是已发布 tag） |
| 发布结论 | `READY_WITH_RISK` |
| 发布范围 | PATH-I repository-local authorization/artifact/replica/materialization contracts 与 integrated QAC |
| 最高正向声明 | `path_i_repository_contract_ready=true` |
| 主要风险 | 真实 producer/adapters/runtime/C1/signal/activation 未授权；checkpoint retry event-ID 工具债务待后续处理 |
| 人工门禁 | approved；DQ-001..003 已接受 |

## 2. 版本号决策

| 项目 | 内容 |
|---|---|
| 当前版本 | N/A；本 CR 没有已发布的独立 PATH-I 版本 |
| 目标版本 | `CR-172-PATH-I-rc.1` |
| 变更类型 | `rc` |
| 兼容性 | repository-local additive / compatible；runtime compatibility 未验证 |
| 推荐原因 | 五个 Story 已独立 CP7 PASS，三项高风险边界已由用户接受；runtime deployment 与真实数据 activation 仍未授权，不能标记 runtime released |

## 3. 新增能力 / 用户可见变化

| Change ID | 内容 | 影响用户 | 来源 |
|---|---|---|---|
| REL-CR172-001 | 六类动作的独立授权/eligibility deny-default repository contract | 研究治理维护者 | S01 |
| REL-CR172-002 | trial-return sealed artifact 与唯一 verifier contract | 研究 artifact 维护者 | S02 |
| REL-CR172-003 | research replica freshness、receipt 与 CAS contract | 研究存储维护者 | S03 |
| REL-CR172-004 | execution-local materialization current-exact capability contract | 执行缓存维护者 | S04 |
| REL-CR172-005 | 15/27/11 追踪、失败恢复、claim ceiling、zero-op integrated QAC | 审批者/验证者 | S05 |

## 4. 行为变化 / 修复问题

| Change ID | 类型 | 内容 | 用户影响 |
|---|---|---|---|
| FIX-CR172-001 | security/fail-closed | caller self-report、URI equivalence、stale decision、CAS conflict、forged capability、sensitive marker 全部 fail closed | 降低授权绕过和错误 selection 推进风险 |
| FIX-CR172-002 | trace | SC-021/REQ-013 与 SC-006/REQ-006 显式语义绑定 | 防止“总量全绿但需求错绑” |

## 5. 破坏性变更

| Breaking ID | 是否存在 | 内容 | 迁移引用 |
|---|---|---|---|
| BR-CR172-001 | no | 未替换现有 runner、lineage、配置、状态 schema、安装路径或外部接口 | `docs/release/MIGRATION-CR172.md` |

## 6. 安装、部署与升级

| 场景 | 方式 | 验证证据 |
|---|---|---|
| 安装/升级 | N/A；本 CR 不交付 installer 或平台安装包 | `docs/release/DEPLOY-CHECKLIST-CR172.md` |
| deployment | not-authorized | CP8 packet authz boundary |
| repository delivery | 已完成严格限定范围的双仓 Git 交付：quant-lab `fc8b7c4e0c934f40d17ca1bcc0079f07d58b32bb` -> `origin/master`；artifact 内容提交 `562872610d17458bf6f37d1c6cb32a2377d10644` -> `origin/main`；`process/meta-flow/**` 排除 | `process/checks/CR172-PAIRED-GIT-DELIVERY.result.json` |

## 7. 迁移说明

| 是否需要迁移 | 影响对象 | 说明 |
|---|---|---|
| no | N/A | 无 state/config/path/frontmatter/data migration；详见 `docs/release/MIGRATION-CR172.md` |

## 8. 已知问题与风险

| Risk ID | 严重度 | 状态 | 处理 |
|---|---|---|---|
| R-CR172-RUNTIME-AUTHORIZATION-GAP | HIGH | accepted boundary / not-authorized | DQ-CP8-CR172-002；六动作继续 0/6 |
| R-CR172-REAL-R-DOMAIN-MISMATCH | HIGH | deferred | DQ-CP8-CR172-003；typed_unavailable 保留 |
| R-CR172-TRIAL-RETURN-SOURCE-ABSENT | HIGH | deferred | future native producer prerequisite |
| R-CR172-SYNC-REPLICA-STALE | HIGH | repository contract controlled / runtime-unverified | real adapter activation 前复验 |
| R-CR172-DUAL-REPO-DIRTY | HIGH | scoped delivery completed | DQ-001；两个授权 ref 均已用远端哈希验证；artifact 无关 `process/meta-flow/**` 修改继续排除 |
| R-CR172-CHECKPOINT-RETRY-EVENT-ID | HIGH | accepted process risk / follow-up | DQ-001；受影响重复 ID=`10`、额外冲突行=`12`，不声明 checkpoint ledger 全局校验 PASS；跟踪 `FU-CR172-LEDGER-001` |

## 9. 九项 false claim

`stage3_started=false`、`stage3_entry_ready=false`、`c1_computable=false`、`real_data_authorized=false`、`multi_trial_runtime_authorized=false`、`signal_transport_authorized=false`、`path_c_or_a_resumed=false`、`public_c1_projection_ready=false`、`empirical_r_effective_count_ready=false`。

任何一项需要变为 true 时，必须重新进入对应的 activation-resume CP2、runtime-high-risk gate 或独立 CR；本 CP8 approve 不授权该变化。

## 10. CP8 Fact Diff 摘要

| Promise Ref | 状态 | 发布影响 | 证据 |
|---|---|---|---|
| CR172-S01..S05 | EXECUTED_POSITIVE_RESULT 5/5 | READY | five final CP7 Result/Evidence |
| real activation | DEFERRED_FOLLOW_UP | READY_WITH_RISK | `process/release/RELEASE-CONTEXT-CR172.yaml` |
| empirical-R/public C1 | DEFERRED_FOLLOW_UP | READY_WITH_RISK | DQ-CP8-CR172-003 |
| PATH-C/A and signal | DEFERRED_FOLLOW_UP | READY_WITH_RISK | DQ-CP8-CR172-002/003 |

## 11. 回滚方式

| 回滚触发 | 回滚入口 | 说明 |
|---|---|---|
| repository contract regression、guard false-negative、trace mismatch | `docs/release/ROLLBACK-CR172.md` | 只回滚 CR-owned repository snapshot；无数据/状态恢复 |

## 12. 参考链接

| 类型 | 路径 |
|---|---|
| Release Context | `process/release/RELEASE-CONTEXT-CR172.yaml` |
| Delivery Context | `process/context/CP8-CR172-PATH-I-DELIVERY-CONTEXT.yaml` |
| Test Report | `docs/quality/TEST-REPORT-CR172.md` |
| Review | `docs/quality/REVIEW-CR172.md` |
| CP8 gate | `process/checkpoints/CP8-CR172-PATH-I-DELIVERY-READINESS.md` |
| Paired Git delivery | `process/checks/CR172-PAIRED-GIT-DELIVERY.result.json` |

`READY_WITH_RISK` 不等于 runtime `RELEASED`。CP8 本身不授权部署、publish、live、真实数据读取或交易；用户另行授予的严格限定范围双仓 Git commit/push 已完成，并以独立 push evidence 固化。
