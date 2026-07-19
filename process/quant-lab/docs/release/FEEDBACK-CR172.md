---
status: ready-for-human-gate
version: "1.0"
release_artifact_profile: full
release_decision: READY_WITH_RISK
created_at: "2026-07-19T01:47:00+08:00"
---

# CR-172 PATH-I Feedback

## 1. 反馈回流入口

| Feedback ID | 类型 | 来源 | 内容摘要 | 分流目标 | follow-up tracking 候选 | 状态 |
|---|---|---|---|---|---|---|
| FB-CR172-001 | defect | repository regression | 六动作、seal、CAS、materialization 或 semantic trace 失败 | same-story regression / new defect CR | yes | candidate |
| FB-CR172-002 | runtime-authorization | user/operation request | 请求真实 lake/NAS/runtime/producer/materialization | activation-resume CP2 / runtime-high-risk CR | yes | candidate |
| FB-CR172-003 | methodology | research review | empirical-R/effective-count/public C1 positive contract | FU-CR173-001 or approved alternative | yes | candidate |
| FB-CR172-004 | signal-transfer | product/runtime | 请求 SignalBatch transport/mailbox/ack/replay | dedicated signal-transfer CR | yes | candidate |
| FB-CR172-005 | delivery | Host/workspace | source/artifact repo snapshot 不一致或 dirty 无法交付 | paired delivery remediation | yes | candidate |

## 2. 发布后观察计划

这里的“发布后”只表示人工接受 repository-local slice 后；不表示真实 deployment 已执行。

| Signal ID | 观察信号 | 观察方式 | 触发阈值 | 分流 |
|---|---|---|---|---|
| OBS-CR172-001 | combined regression | local CI / approved validation | 任一失败或 skipped>0 | defect / block delivery |
| OBS-CR172-002 | semantic trace | S05 exact probe | mismatch/unknown/duplicate/uncovered 任一 >0 | immediate blocker |
| OBS-CR172-003 | authorization ceiling | zero-op oracle | real authorized/executed 任一 >0 且无独立 gate | security incident / block |
| OBS-CR172-004 | claim ceiling | release review | 九项 false 任一被无证据提升 | governance defect / rollback |
| OBS-CR172-005 | repository pairing | workspace git-status | source/artifact refs 不成对或丢失 | delivery remediation |
| OBS-CR172-006 | real adapter request | issue/change request | 首次出现 | activation-resume CP2 / follow-up CR |

## 3. 风险接受与 DQ 回链

| DQ | 观察重点 | 回退 / 切换条件 |
|---|---|---|
| DQ-CP8-CR172-001 | repository contract tests、双仓库交付状态 | 证据漂移则暂停接受并回退 CP7/CP8 |
| DQ-CP8-CR172-002 | 六动作持续 0/6、PATH-C/A false | 请求真实 activation 时重开 CP2，不在本 CR 静默升级 |
| DQ-CP8-CR172-003 | typed_unavailable、public C1 false | 方法/source/adapter 获批后切换到独立 CR |

## 4. 台账边界

本文件只是反馈入口，不是正式 follow-up tracking 台账，也不表示任何 candidate 已启动。Host 只有在用户决定后，才可按项目协议把候选写入 `process/changes/CR-*-FOLLOW-UP-TRACKING-YYYY-MM-DD.md`，同步 CR index/ledger；这些路径不在本任务 14 项写权限内。

默认不生成独立 `POST-RELEASE-OBSERVATION.md`；观察计划已完整并入本文件。
