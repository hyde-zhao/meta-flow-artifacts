---
story_id: "CR172-S03-nas-replica-verification"
cr_id: "CR-172"
stage: "CP6R1"
status: "ready-for-verification"
implementation_version: "1.1"
implemented_by: "meta-dev"
implemented_at: "2026-07-18T21:45:00+08:00"
work_packet_ref: "process/context/stories/STORY-CR172-S03.CP6.rework-1.work-packet.json"
---

# CR172-S03 CP6 / CP6R1 实现说明

## 实现前置检查

| 检查项 | 结果 | 证据 |
|---|---|---|
| process route health | PASS | `process/current/CURRENT.json.health=ok` |
| Story / CP5 gate | PASS | `design_evidence_confirmed/lld_confirmed/dependencies_satisfied/file_conflict_free=true` |
| S01/S02 contract dependency | PASS | packet 记录 S01/S02 CP7 verified；只读消费两个上游模块 |
| 文件所有权 | PASS | 业务写入仅 S03 两个 primary 文件；过程证据仅 packet 允许路径 |
| 阻塞 clarification / OPEN / Spike | PASS | `0/0/0` |
| 真实动作授权 | PASS | lake/NAS/network/runtime/trading/deploy/Git remote 执行=`0` |

读取审计：首选角色文件 `agents/meta-dev.md`、`.agents/agents/meta-dev.md` 均不存在；Host 批准完整读取 `.claude/agents/meta-dev.md` fallback。为获得 receipt canonicalization、CAS、staging nominal binding、freshness、selected-replica tuple/API 与稳定 reason code，Host 批准读取 S03 LLD，read-expansion ref=`RE-20260718T123658Z0000-ea17df6d`，`full_doc_read_reason=implementation_contract_detail_required`。machine artifact 生成阶段另经 Host 批准 schema-only 读取 S02 Return/Evidence/CP result，refs=`RE-20260718T125222Z0000-e53fc96e`、`RE-20260718T125222Z0000-55943a08`、`RE-20260718T125222Z0000-039c1ad0`；业务正文未作为 S03 输入。CP result 最终一致性审计对本线程刚生成的 IMPLEMENTATION 执行 self-check，ref=`RE-20260718T130005Z0000-15a123a2`。未展开 HLD、ADR、全量 CR、无关 Story 或 S04/S05 LLD。

## 实现对象清单

| 对象 | 实现位置 | 目标 | 验证 |
|---|---|---|---|
| typed replica contracts | `engine/research_artifact_replica.py` | request、mapping、staging、5/5 vector、receipt、selection、result/reason | 构造不变量与 21 项 S03 tests |
| preflight / staging verifier | 同上 | first-write 前授权；source/staging 两次调用同一 S02 verifier | authorization/mismatch/tamper call counters |
| receipt / CAS publish | 同上 | immutable receipt、fresh decision、最多一次 CAS、previous preservation | revoke/persist/CAS recovery tests |
| selected-replica read | 同上 | 只按 current `DistributionSelectionV1` 返回 exact bundle/selection/receipt | old/fake/ref-drift negative tests |
| repository fixture port | S03 test 文件中的 nominal in-memory implementation | 无真实 adapter 的 storage fixture | port counters、non-distributable state |
| static guard / regression | `tests/research/test_cr172_nas_replica_verification.py` | 禁止 reseal、secondary seal digest、facade、bypass、真实操作 import | AST/static guard + S01/S02 regression |
| 平台 / 安装器 / Prompt / Skill | N/A | 本 Story 为 repository-local Python contract | 无平台安装面 |

## 设计契约映射

| 合同 | 实现动作 | 验证入口 |
|---|---|---|
| F-S03-01 / F-S03-10 | `require_action_eligible(...NAS_REPLICA_SYNC..., repository_fixture)` 在任何 port write 前执行；origin/target/context exact match | T-S03-A01、B02 |
| F-S03-02 / F-S03-04 | source 与 staged bundle 各调用一次 `verify_sealed_trial_return_bundle`；5/5 vector 全 true 才继续 | T-S03-P01、N01、N02、F02、Z01 |
| F-S03-03 | staging token 固定 `non_distributable`；失败不生成 selection | T-S03-P01、F01 |
| F-S03-05 | freshness 仅由 expected release + canonical source-selection hash 相等判定 | T-S03-N02/N03 |
| F-S03-06 / F-S03-07 | commit 前重新消费 fresh decision；immutable persist 后进行一次 expected-previous CAS | T-S03-A02、F03 |
| F-S03-08 / NFR-S03-03 | receipt canonical body 排除 mapping handles/物理路径；hash 使用显式 replica domain | T-S03-P02、B01 |
| F-S03-09 | 未导入 seal canonicalizer/digest，未构造 `ArtifactSealV1`，无 research pointer write | T-S03-P01、Z01 |
| F-S03-11 / NFR-S03-09 | nominal port 的不可覆写 read contract 校验 current selection 与 tuple 全字段 refs | T-S03-P01、N03 |
| failure preservation | staging/persist/revoke/CAS 失败均不覆盖 current selection | T-S03-F01、A02、F03 |

## 单元测试与 Fixture 计划

- Positive：5/5 verification、receipt correlation、CAS revision=1、exact selected tuple。
- Determinism：三组不同 mapping handle 产生同一 receipt body/hash，物理 mapping occurrence=`0`。
- Authorization：revoked、wrong origin、real target、wrong action kind 在任何 port write 前拒绝。
- Integrity：release/logical/content/manifest/freshness/unversioned/staged content tamper 全部 fail-closed。
- Recovery：copy interruption、commit revoke、receipt persist failure、CAS failure 保留 previous selection=`100%`。
- Read boundary：old/fake selection、receipt ref drift 均拒绝；path/latest/staging/receipt-only public read entry=`0`。
- Static：真实操作 import/env/read_bytes/rsync=`0`；seal canonicalizer/digest/reseal/verifier facade/S04 import=`0`。

## 最小实现切片

| Slice | 内容 | 局部验证 | 回滚点 |
|---|---|---|---|
| T02-A | typed values、canonical receipt、nominal port、preflight | `py_compile` + positive/authorization tests | 删除 S03 新模块 |
| T02-B | staging reverify、5/5 vector、fresh commit、CAS | tamper/revoke/persist/CAS tests | 回退到 T02-A |
| T02-C | current-selection exact read contract | selected read positive/negative tests | 回退 read method |
| T03/T04 | 21 项 tests、S01/S02 回归与 static guard | S03 21 passed；组合 102 passed | 删除 S03 test 文件 |

## 平台差异处理

N/A。本 Story 不涉及 Claude/Codex/OpenClaw schema、安装路径、MCP、真实 NAS adapter 或 deploy。所有验证通过 `uv run --python 3.11` 在 repository fixture 环境执行。

## 验证结果

| 验证 | 结果 |
|---|---|
| S03 scoped pytest | PASS，`21 passed` |
| S01+S02+S03 targeted regression | PASS，`102 passed` |
| `py_compile` | PASS，exit=`0` |
| whitespace | PASS，两 primary 文件 diagnostics=`0/2`；`--no-index` expected diff exit=`1` |
| static zero-operation guards | PASS，包含在 S03 21 项测试 |
| return / evidence / CP result checks | PASS；return-check=OK，evidence-index 已生成，CP result-check=OK；仅保留 symlink/input-hash/final-attempt 审计 warnings |

## 未覆盖项

- 真实 NAS/mount/rsync/network adapter、credential/env、lake/runtime/signal/trading/deploy：超出授权且刻意为 0。
- S04 materialization 与 S05 integrated verification：仍锁定，未启动、未读取其 LLD、未修改其文件。
- 真实文件 chunk I/O 性能：当前仅提供 typed port contract 与 in-memory fixture；不声明真实 NAS SLA。
- `DEV-LOG.md`：packet 未授权写入，故以 Story 实现摘要、IMPLEMENTATION、return/evidence/result 作为等价可审计交接。

## 设计缺口反馈

无。LLD v1.3 与 packet 无规范冲突；Story 卡片中“可提供窄 verifier facade”的旧表述被 packet 与 LLD v1.3 的显式禁止项覆盖，实际实现 facade=`0`。长期 Feature/HLD/ADR 修改=`0`，Design Delta=N/A。

## 后续交接

交给 meta-qa 时只需读取 work packet、return packet、evidence index 与 CP6 result。重点复核：双 verifier call count=`2`、receipt `original_seal_sha256` 来源、selected read exact tuple、previous selection preservation、static zero-operation guard。S04 只有在 S03 独立 CP7 PASS 后才可解锁。

## CP6R1 验证失败回修

### 输入与读取审计

- 回修输入：`process/checks/CP7-CR172-S03-VERIFICATION.result.json`，阶段结论=`NEEDS_REWORK`。
- Finding：`F-CP7-S03-001`（BLOCKER fresh-decision fail-closed）与 `F-CP7-S03-002`（REQUIRED deterministic concurrent-current CAS regression）。
- 本轮 packet：`process/context/stories/STORY-CR172-S03.CP6.rework-1.work-packet.json`。
- Story / IMPLEMENTATION final-self-check expansions：`RE-20260718T133648Z0000-9cfbb30b`、`RE-20260718T133648Z0000-8df8ffc3`。本轮未读取 LLD、HLD、ADR、Feature 长文或 S04/S05。

### 最小修复与契约映射

| Finding | 修复 | 回归证据 |
|---|---|---|
| F-CP7-S03-001 | `ReplicaPreflightV1.preflight_decision` 保存首次 typed decision；新增 `COMMIT_DECISION_NOT_FRESH`；`_validate_commit_recheck` 精确比较 action/context/origin/target 和严格递增 `evaluated_at`；publish 在 staging 前先行调用，commit 内再次调用 | reused/equal-time/older=`COMMIT_DECISION_NOT_FRESH`；expired/revoked=`COMMIT_AUTHORIZATION_INVALID`；五类 stage/persist/CAS/selection=`0/0/0/0` |
| F-CP7-S03-002 | in-memory nominal port 在一次 CAS 调用内注入 concurrent current；CAS 按 expected previous 返回冲突 | `POINTER_CONFLICT`、attempt selection=`None`、CAS=`1`、current=`concurrent` |

### CP6R1 验证结果

| 验证 | 结果 |
|---|---|
| S03 scoped pytest | PASS，`27 passed` |
| S01+S02+S03 targeted regression | PASS，`108 passed` |
| `py_compile` | PASS，exit=`0` |
| whitespace | PASS，两 primary diagnostics=`0/2`；`--no-index` exit=`1` 为预期 diff |
| Return / Evidence / CP result | PASS；return-check=OK，evidence-index 已刷新，result-check consistency=OK；仅有外置 process path/final-attempt audit warnings |
| real NAS/lake/runtime/network/env/credential/trading/Git remote | executed=`0` |

### 剩余风险与交接

- `R-CR172-SYNC-REPLICA-STALE` 已在 repository-local 实现层关闭，仍需 meta-qa-critical CP7R1 独立复验。
- 真实 NAS adapter/runtime 继续为 `not_authorized`，本回修不改变授权面。
- 长期设计差异、waiver、forbidden touch=`0/0/0`；S04 在 CP7R1 PASS 前继续锁定。
