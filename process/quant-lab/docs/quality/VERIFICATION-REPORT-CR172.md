---
status: complete
version: "1.0"
scope: "CR-172 PATH-I repository-local contracts"
created_at: "2026-07-19T01:43:00+08:00"
validation_mode: "repository-fixture-static-and-contract"
verification_result: "PASS_WITH_RISK"
release_decision: "READY_WITH_RISK"
---

# CR-172 PATH-I 验证报告

## 1. 验证范围

本报告聚合 S01～S05 的最终独立 CP7 机器证据，判断 PATH-I repository-local 合同、fixture、guard 和失败关闭边界是否可进入 CP8 人工终验。默认读取入口为 `process/context/CP8-CR172-PATH-I-PREPARATION-CONTEXT.json`；未扩读完整 CR、HLD、LLD、IMPLEMENTATION、源码或测试正文。

范围内：六动作授权合同、trial-return sealed artifact、NAS replica verification、execution cache materialization、跨 Story 追踪/claim ceiling/zero-operation QAC。

范围外：真实数据湖/NAS/执行机访问、multi-trial runtime、真实 trial-return 生成、empirical R/public C1、SignalBatch 传输、交易、部署、发布、Git remote write、PATH-C/A activation-resume。

## 2. 验证对象清单

| 对象 | 类型 | 最终证据 | 结果 | 发布含义 |
|---|---|---|---|---|
| S01 authorization/eligibility | contract + tests | `process/checks/CP7-CR172-S01-REVERIFY-1.result.json` | PASS | 六动作 deny-default 合同可交付 |
| S02 trial-return artifact | contract + tests | `process/checks/CP7-CR172-S02-REVERIFY-2.result.json` | PASS | sealed artifact repository contract 可交付 |
| S03 NAS replica | contract + tests | `process/checks/CP7-CR172-S03-REVERIFY-1.result.json` | PASS | freshness/CAS/replica contract 可交付；真实 NAS 未授权 |
| S04 execution cache | contract + tests | `process/checks/CP7-CR172-S04-REVERIFY-1.result.json` | PASS | materialization contract 可交付；执行机未授权 |
| S05 integrated QAC | fixture + regression | `process/checks/CP7-CR172-S05-REVERIFY-1.result.json` | PASS | 15/27/11 精确追踪和 claim ceiling 可交付 |
| 五个 Evidence Index | structured evidence | `process/evidence/CR172-S0*.CP7*.index.json` | 5/5 OK | 支持 CP8 capsule-first 消费 |
| CP5 设计基线 | approved gate | `process/checkpoints/CP5-CR172-ALL-STORIES-LLD-BATCH.md` | approved | repository-local 范围与不授权项未放宽 |

## 3. 验证追踪矩阵

| Story | 关键承诺 | 最终检查 | 状态 | 剩余风险 |
|---|---|---|---|---|
| S01 | 6 actions / 12 fields / 5 DAG edges / one invariant validator / canonical URI | 46/46；historical findings 2/2 CLOSED | PASS | trusted real approval adapter absent |
| S02 | exact sealed payload、13-field binding、ordered authorization refs、atomic commit | 35/35；S01+S02 81/81；finding 1/1 CLOSED | PASS | native trial-return producer absent |
| S03 | fresh decision、selected exact read、immutable receipt、CAS conflict safety | 27/27；S01-S03 108/108；findings 2/2 CLOSED | PASS | real NAS adapter/runtime not verified |
| S04 | current exact capability、4/4 bytes verification、sensitive refs fail closed | 19/19；S01-S04 127/127；findings 3/3 CLOSED | PASS | real execution adapter durability/performance unverified |
| S05 | 15 requirements / 27 scenarios / 11 outcomes；semantic binding exact | S05 27/27；S01-S05 154/154；finding 1/1 CLOSED | PASS | fixture evidence cannot establish runtime readiness |

## 4. 设计契约验证

| 契约 | 结果 | 量化证据 | 证据路径 |
|---|---|---|---|
| 六动作独立授权与 deny-default | PASS | actions/records/enforcement=`6/6/6`；permission union=`0` | S01/S05 CP7 Result |
| artifact 单 seal/verifier 真相 | PASS | payload=`2/2`；S02 verifier=`1`；secondary digest=`0/0` | S02/S05 CP7 Result |
| replica freshness 与 CAS | PASS | stale/forged/reused mutation=`0`；CAS conflict preserves peer writer | S03 CP7 Result |
| materialization current-exact capability | PASS | invalid capabilities accepted=`0/6`；valid current exact=`1/1` | S04 CP7 Result |
| failure recovery | PASS | invalid selection advance=`0`；previous preserved=`2/2`；persist/CAS delta=`0/0` | S03/S04/S05 CP7 Result |
| trace semantic exactness | PASS | requirements/scenarios/outcomes=`15/27/11`；semantic mismatch=`0` | S05 CP7 Result |
| claim ceiling | PASS | high-order claims true=`0/5`；deferred counters nonzero=`0/12` | S05 CP7 Result |

## 5. 九项必须保持 false 的能力声明

| Claim | 当前值 | 状态 | 重新评估条件 |
|---|---:|---|---|
| `stage3_started` | false | not-authorized | future activation-resume CP2 approved |
| `stage3_entry_ready` | false | deferred | real evidence prerequisites verified |
| `c1_computable` | false | deferred | empirical-R/effective-count positive contract approved |
| `real_data_authorized` | false | not-authorized | independent real-lake authorization |
| `multi_trial_runtime_authorized` | false | not-authorized | independent runtime gate |
| `signal_transport_authorized` | false | not-authorized | dedicated signal-transfer CR/gate |
| `path_c_or_a_resumed` | false | deferred | activation-resume CP2 decision |
| `public_c1_projection_ready` | false | deferred | public projection migration/compatibility CR |
| `empirical_r_effective_count_ready` | false | deferred | FU-CR173-001 or another approved methodology upgrade |

最高允许正向声明只有 `path_i_repository_contract_ready=true`。该声明不等于 activation、runtime-ready、Stage 3 ready 或 released。

## 6. 六类真实动作与零操作边界

| Action | Authorized | Executed | 状态 |
|---|---:|---:|---|
| data-lake read | 0 | 0 | not-authorized |
| multi-trial runtime | 0 | 0 | not-authorized |
| trial-return generation | 0 | 0 | not-authorized |
| empirical-R computation | 0 | 0 | not-authorized |
| NAS replica sync | 0 | 0 | not-authorized |
| execution materialization | 0 | 0 | not-authorized |

真实 lake、NAS、runtime、network、credential、signal、trading、deploy、Git remote 操作均为 `0`。

## 7. 自动化与机器证据

| 检查 | 结果 | 说明 |
|---|---|---|
| 五个 CP7 Result basic check | 5/5 OK | 仅 process symlink/input-path/final-attempt 既有 warning |
| 五个 Evidence Index check | 5/5 OK | 结构可消费 |
| S05 scoped | 27/27 | failed/skipped=`0/0` |
| S01-S05 combined | 154/154 | failed/skipped=`0/0` |
| semantic trace | PASS | mismatch/unknown/duplicate/uncovered=`0/0/0/0` |
| process route | PASS | symlink target 与 project name 一致 |
| workspace git status | RISK | source/artifact 两仓均 dirty；未执行 commit/push |
| checkpoint ledger full check | PASS_WITH_RISK | 受影响重复 event ID=`10`、额外冲突行=`12`，完整 `event check=FAIL`；最终事件仍可由 `checked_at/result_ref/context_ref` 唯一定位；`FU-CR172-LEDGER-001` 待处理 |

S01～S04 带 current-state correlation 的复核在 CP8 准备期间报告 `next_action=await_agent_return`；这是当前等待本 agent 交回的预期瞬态，不改变各最终 CP7 Result 的 PASS。Host 建立 CP8 pending gate 后必须将 next action 切换为 `await_user` 并执行终态关联审计。

## 8. Findings、Waivers 与剩余风险

- Open blocker：`0`。
- Waiver：`0`。
- 历史 NEEDS_REWORK findings：`9`，全部 `CLOSED`，详见 `docs/quality/FIXES-CR172.md`；未删除历史证据。
- HIGH residual：真实 producer、trusted approval adapter、real NAS/execution adapter、runtime path enforcement、empirical-R/public C1、signal transport 与 activation 均未授权或 deferred。
- 双仓库 dirty：允许准备人工终验，不允许据此宣称已发布或已推送。
- Checkpoint ledger：`cp ledger-append` 的默认 retry identity 造成 10 个受影响的重复 `event_id`、12 条额外冲突记录；不影响最终 Result 内容，但全量 ledger 校验不可宣称 PASS，且当前 CP8 不授权改写 append-only 历史。

## 9. 阶段结论

验证结论：`PASS_WITH_RISK`。CP8 推荐 `release_decision=READY_WITH_RISK`、自动预检 `decision=PASS`、人工门禁 `pending`。只有用户完成三项 DQ 的风险接受后，Host 才可接受 PATH-I repository-local slice；真实 activation 和 PATH-C/A 继续 deferred。
