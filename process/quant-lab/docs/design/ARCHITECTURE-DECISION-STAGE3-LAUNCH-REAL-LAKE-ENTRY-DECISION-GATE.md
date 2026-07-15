---
title: "Architecture Decisions — Stage 3 Launch / Real-Lake Entry Decision Gate"
cr_id: "CR-171"
status: "approved-cp3"
---

# Architecture Decisions — Stage 3 Launch / Real-Lake Entry Decision Gate

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| v0.1 | 2026-07-15 | meta-se-critical | 新建 ADR-CR171-001..004，与 HLD §5–§11 对齐；仅记录 CP3 待确认设计，不生成授权。 |
| v0.2 | 2026-07-15 | host-orchestrator | 评审修订：ADR-002..004 增加逐项、非日历式重开/切换条件；与 HLD §19/§21 和 CP3 state closure 对齐。 |
| v0.3 | 2026-07-15 | user / host-orchestrator | 用户批准 CP3 的 4 项 ADR 推荐；仅进入 CP7 design verification，未产生 read/computation/runtime/write 授权。 |

## ADR-CR171-001：采用两 CR 的 C1–C4 activation 分离

- **状态：** approved at CP3 on 2026-07-15。
- **上下文：** CP2 已选 `c1_c4_real_producer`，但 CR-171 不得执行 producer、映射、binding 或 computation。
- **决定：** CR-171 只交付决策、静态可行性、候选 read contract、历史事实和 waiver contract；独立 Real-Evidence Activation CR 才拥有 real-data-to-producer mapping、release/schema/PIT/lineage binding、real computation authorization、producer-result/run identity binding。
- **备选：** current runner 单 CR；将所有实现并入 CR-171。
- **理由：** 前者会保留 `R-CR170-RUNNER-GAP`，后者混淆 high-risk authorization 与实现/aggregate 风险。
- **后果：** CP3 不能把 current runner 当隐含 fallback；C4 的 `incompatible` verdict 需要后续独立 rework。
- **重新打开条件：** C1–C4 全部被授权前的静态合同证明为无法通过后续 activation 补齐，或人类明确要求重开 CP2 route。

## ADR-CR171-002：五字段只读合同只能作为 pending-human candidate

- **状态：** approved at CP3 on 2026-07-15。
- **决定：** `data_release`、`datasets`、`date_range`、`read_identity`、`output_directory` 每项在未获人类冻结时均为 `pending_human_authorization`；无字段可从历史 run、文档路径或 CP2 approval 推断。
- **Deny-default：** credentials/environment、provider、lake/NAS write、catalog/current-pointer mutation、runtime/real computation、trading/broker。
- **备选：** 从 legacy run 声明一个具体 release/目录；保持不定义合同。
- **理由：** 前者产生伪授权，后者不能为 activation CR 提供可审计输入。候选语义同时确保 CP3 可以完成而不读取数据。
- **失败行为：** 任意 1/5 字段缺失、含 wildcard 或 deny 类不完整，activation scope 为 blocked。
- **重新打开条件：** 只有以下任一事件可重开本 ADR：① 经授权的数据 owner 提交 5/5 的具体有限值及对应的 release/schema/PIT/lineage 证据，需在单独的 authorization gate 评审；② 任一候选字段需要 wildcard、历史路径推断、credentials discovery 或突破 6/6 deny-default；③ output evidence 需要写入而没有独立 write authority。重开只允许回到 CP3 或未来 activation CR 的授权设计；不得把候选字段直接当作 read permission。

## ADR-CR171-003：历史证据只作三值 current-entry classification

- **状态：** approved at CP3 on 2026-07-15。
- **决定：** historical evidence 必须保持 `legacy / require-revalidation`；current-entry report 只可为 `revalidated_for_current_entry`、`insufficient_for_current_entry`、`incompatible_rework_required`。旧资料中的 `reaffirmed_as_legacy_only` 仅为 legacy annotation，不增设第四个 current-entry verdict。
- **禁止：** repair、backfill、rerun、manifest rewrite、schema/PIT/lineage/code/evidence remediation。
- **备选：** 将修复并入 revalidation；仅继续使用 historical PASS。
- **理由：** 两者均会破坏 claim integrity，并把 CR-171 从决策门扩大为实现或数据修复门。
- **后果：** `revalidated_for_current_entry` 也不改变所有 Stage 3/runtime/read/computation readiness false ceiling。
- **重新打开条件：** 只有以下任一事件可重开本 ADR：① 授权的 revalidation report 需要三值 schema 之外的 verdict；② 发现必须 repair/backfill/rerun/manifest rewrite 才能回答 current-entry 问题；③ current-entry contract 的 schema/PIT/lineage 定义经正式 CR 变更。前两者必须创建/路由独立 rework CR；不得在 CR-171 report lane 内修复，且 historic PASS 不能替代重开。

## ADR-CR171-004：event-bounded FU-006 waiver 没有延展分支

- **状态：** approved at CP3 on 2026-07-15。
- **决定：** waiver 只在 design state 可用；在首个 real-evidence admission PASS/PASS_WITH_RISK 前（E1）及 Stage 3 exit-gate start 前（E2）失效。任一事件且 FU-006 independent verifier evidence 缺失时，所请求的动作必须 block。
- **备选：** calendar-expiry、可继承 CR-170 waiver、warning-only waiver。
- **理由：** calendar 或 warning 难以机械关联成熟动作；CR-170 已明确不可继承。
- **后果：** CP3 approval 不产生 waiver extension，FU-006 的验收仍是成熟准入前的独立工作。
- **重新打开条件：** 只有以下任一事件可重开本 ADR：① FU-006 independent verifier evidence 被一个正式 gate 接受，从而只改变相应 E1/E2 的 `evidence absent` 条件；② admission 或 exit-gate 的规范性事件定义经正式 CR 变更。calendar extension、warning-only 降级或继承 CR-170 waiver 不是可接受的重开结果；提出任一项必须重开 CP2 并重新进行 runtime-high-risk 风险审查。

## 一致性矩阵

| ADR | HLD sections | Requirement | CP3 decision |
|---|---|---|---|
| 001 | §5–§8、§17 | REQ-CR171-001 | DQ-CP3-CR171-001 |
| 002 | §7、§11、§14 | REQ-CR171-003 | DQ-CP3-CR171-002 |
| 003 | §9、§11、§13 | REQ-CR171-004/005 | DQ-CP3-CR171-003 |
| 004 | §10、§13–§14 | REQ-CR171-002 | DQ-CP3-CR171-004 |

## Not Authorized

These ADRs do not authorize real data/lake/NAS/provider/credential access; output writes; computation/producer execution; aggregate/CR155 promotion; runtime/simulation/trading; code/tests/Stories/LLD; Git remote write; publish/deploy. `stage3_started=false`, `stage3_entry_ready=false`, `real_computation_authorized=false`, aggregate=false and CR155 promotion=false remain mandatory.
