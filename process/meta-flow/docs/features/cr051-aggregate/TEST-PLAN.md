---
feature_id: "FEAT-AW-04"
change_id: "CR-051"
story_ids: ["ST-AW-004"]
validation_mode: "mixed"
status: "ready-for-story-design"
version: "1.2"
---

# FEAT-AW-04 测试计划

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-18 | meta-se-critical | 建立四状态全组合、correlation、single-writer、projection 与 PARTIAL negative tests。 |
| 1.1 | 2026-07-18 | meta-dev-debugger | CP5 R2：增加 published handle重读、payload/receipt/key/ref完整性和无自引用evidence DAG测试。 |
| 1.2 | 2026-07-19 | Host Orchestrator（inline-fallback） | CP8 终验回修：补 target policy 双向验证、依赖边界/exact DAG、PARTIAL staged projection 与幂等 retry。 |

## 1. 范围与质量门

以决策表/状态转换/属性测试验证纯聚合，以故障注入验证 result persistence 与 projection boundary。P0 hard invariant 为：`BLOCKED > FAIL > IN_PROGRESS > PASS`、仅 2/2 required terminal PASS 可 PASS、PARTIAL 不成功、聚合零 Git 调用、单写冲突 fail closed。

Entry：DESIGN 与 ST-AW-004 full LLD 经 CP5 确认；FEAT-AW-03 `LegResult` schema/version 冻结；existing writer adapter 可用隔离 fixture 替代。

Exit：四状态16组合100%通过；raw/unpublished或invalid published-handle fixtures 100%拒绝；非all-PASS projection调用=0；相同输入deterministic；payload自引用/二次覆盖=0；P0安全缺陷=0。

## 2. 状态组合预期

| source \ artifact | BLOCKED | FAIL | IN_PROGRESS | PASS |
|---|---|---|---|---|
| BLOCKED | BLOCKED | BLOCKED | BLOCKED | BLOCKED |
| FAIL | BLOCKED | FAIL | FAIL | FAIL |
| IN_PROGRESS | BLOCKED | FAIL | IN_PROGRESS | IN_PROGRESS |
| PASS | BLOCKED | FAIL | IN_PROGRESS | PASS |

该表仅适用于 schema、digest、identity、required-set、correlation 全部验证通过的输入；否则预期一律 BLOCKED。

## 3. 测试矩阵

| Test ID | 层级 | 场景 | 预期 | 覆盖 |
|---|---|---|---|---|
| TP-AW04-001 | property/unit | 四状态 16 组合 | 与固定优先级表 100% 一致 | REQ-AW-012；RULE-AW-06 |
| TP-AW04-002 | unit | source/artifact 2/2 reread-validated published payload均terminal PASS | overall PASS、eligible、deterministic digest | TC-AW-008/009；F03 |
| TP-AW04-003 | negative | source PASS、artifact FAIL | overall FAIL；progress/effect=PARTIAL；projection=hold | TC-AW-014 |
| TP-AW04-004 | negative | 任一 result 缺失或非 terminal | IN_PROGRESS 或 validation BLOCKED；不得 PASS | REQ-AW-012 |
| TP-AW04-005 | validation | raw payload/ref、unpublished outcome或stale logical attempt | BLOCKED(invalid-published-result-set) | OBJ-AW-07；F03 |
| TP-AW04-006 | validation | wrong CR/project/operation/leg kind | 100% 拒绝；projection调用=0 | NF-AW-001 |
| TP-AW04-007 | validation | duplicate source、缺 artifact、unknown leg | BLOCKED(required-set) | REQ-AW-012 |
| TP-AW04-008 | validation | checksum/readback/schema version 错误 | BLOCKED(untrusted-result) | NF-AW-005 |
| TP-AW04-009 | persistence | aggregate append 失败 | 不投影；可重试相同 payload | R-AW-02 |
| TP-AW04-010 | concurrency | 两 writer 写相同 payload | idempotent same-result；单个 current ref | single writer |
| TP-AW04-011 | concurrency | 两 writer 写冲突 payload | BLOCKED(conflicting-payload)，不按最后写入胜出 | single writer |
| TP-AW04-012 | projection | all PASS 后 controlled writer 成功 | completion 仅一次，result ref 可追踪 | REQ-AW-013 |
| TP-AW04-013 | projection | writer 部分失败/timeout | aggregate不改；projection failure 可重试；不重跑 Git | failure recovery |
| TP-AW04-014 | isolation | aggregate/status 调用 | Git adapter、leg executor、manual sync 调用均为0 | dependency boundary |
| TP-AW04-015 | regression | CR-050 paired workflow result | 不被 CR-051 required-set 误消费 | compatibility |
| TP-AW04-016 | CLI | status/aggregate dry-run | 输出 input/result refs；不隐式 resume/close/sync | NF-AW-004 |
| TP-AW04-017 | published integrity | reader返回payload与handle/receipt digest不一致，或result_ref/key被替换 | 100% BLOCKED；pure aggregate/projection调用=0 | F03 |
| TP-AW04-018 | receipt integrity | 篡改writer_id/written_at/receipt_digest，或receipt绑定错误single-write key | 100%拒绝 | F03 |
| TP-AW04-019 | no self-reference | 扫描LegResultPayload/AggregateResult canonical schema | 自身result_ref、write/append receipt、writer/time/receipt digest字段=0；写前digest可复算 | F03 |
| TP-AW04-020 | evidence DAG | payload→receipt→validated handle→aggregate→aggregate receipt→projection | 每边可追踪且无回边/覆盖；Git/worktree调用=0 | F03 |

## 4. Fixture 与故障注入

- typed PublishedLegResultHandle factory生成四状态、raw/unpublished、missing/duplicate/stale/wrong identity/payload/receipt/key/ref digest/schema cases，并通过reader重读fixture验证。
- spy adapters 分别记录 Git、executor、aggregate store、CR/state/current writer、manual sync 调用。
- barrier fixture 制造相同/冲突 payload 并发写；fault fixture 注入 append、fsync/readback、projection timeout。
- 属性测试随机排列 required refs，确认输入顺序不改变决策与 canonical digest。

## 5. 证据要求

测试报告必须保存：16 组合结果、invalid-input 分类计数、projection 调用计数、writer concurrency 结果、dependency/command spy、deterministic digest 和 legacy regression。不得用“最终状态看似正确”替代中间副作用为 0 的证据。

## 6. 剩余风险与 Gotchas

真实 workflow writer 的跨进程竞争需要在明确 runtime authorization 下验证；本地 fixture 未覆盖的托管平台行为作为剩余风险，不得放宽 pure aggregate hard gate。测试数据中的 PARTIAL 必须写入 progress/effect 字段，若测试为了方便新增 overall PARTIAL，会把错误 schema 固化为实现要求。

## 7. CP8 终验回修执行证据

| 证明义务 | 自动化断言 | 状态 |
|---|---|---|
| consumer target policy | 错 mode/base/target/active ref 在 reread 后以 `TARGET_POLICY_MISMATCH` 拒绝 | PASS |
| dependency boundary | aggregate 不调用 Git、worktree、leg executor 或 manual sync | PASS |
| exact evidence DAG | payload→receipt→handle→aggregate→receipt→projection 顺序与引用精确一致 | PASS |
| PARTIAL staged projection | state 成功、ledger 失败保留 state receipt 并返回 PARTIAL | PASS |
| idempotent retry | retry 不重写 aggregate/不重跑 Git，补齐 ledger 后 COMPLETE | PASS |

ST-AW-003/legacy/ST-AW-004 定向集合 127 项通过；跨模块集合 307 项与 58 个 subtests 通过。
