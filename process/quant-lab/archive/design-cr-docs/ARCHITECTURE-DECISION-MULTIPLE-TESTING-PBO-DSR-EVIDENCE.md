---
status: accepted-cp3
version: "0.2"
cr_id: "CR-164"
---

# Architecture Decisions: CR164 Computable Statistical Evidence

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 0.1 | 2026-07-12 | host-orchestrator inline meta-se-critical | 提议 method envelope、bootstrap、DSR count、aggregation、consumer 与授权决策。 |
| 0.2 | 2026-07-12 | host-orchestrator | DQ-CP3-CR164-001..004 经用户 `approve` 接受；ADR-001..008 生效。 |

## ADR-CR164-001 Method-neutral envelope and pure calculators

- 状态：ACCEPTED（CP3，2026-07-12）。
- 决定：建立一个 validated `StatisticalEvidenceInput` 与统一 `MethodEvidence` envelope；BH、WRC/SPA、PBO/CSCV、DSR 为无 I/O 的独立 calculator；聚合器不重算。
- 备选：单体计算器（耦合高）；动态插件 registry（当前方法固定，复杂度无收益）。
- 切换：方法数或第三方扩展显著增加时另行评审 registry。

## ADR-CR164-002 Sealed identity before computation

- 决定：所有方法绑定同一 family ref/hash/raw count/candidate membership；任何 mismatch/tamper 为 blocked，缺失且无矛盾为 typed_unavailable。
- 禁止：calculator 后验删除失败候选或重新定义 family。

## ADR-CR164-003 WRC/SPA stationary bootstrap policy

- 决定：MVP 冻结 `bootstrap_method=stationary_bootstrap`、`block_length_mode=fixed_window`、`block_length` 为显式正整数；seed、replication count、benchmark/null、window、input/config hash 全部进入 provenance 和 summary hash。
- 理由：固定 window 可在 fixture/static 范围内完整复跑，避免把未经独立验证的自动选择器隐藏进统计结论。
- 备选：`automatic_politis_romano_family` 自动选择。只有明确算法版本、参考实现、边界 fixture 与独立验证通过并经新设计决策后启用；不得把“stationary bootstrap 方法来源”误写成“自动 block selector 已实现”。
- 来源边界：Politis & Romano (1994), DOI `10.1080/01621459.1994.10476870`，是 stationary-bootstrap 方法来源；本 ADR 不把该来源误称为本项目 automatic selector 的现成规范。
- 失败：window<1、replications 不足、seed/config 缺失、return matrix 不对齐均 typed_unavailable 或 blocked（冲突/非法值为 blocked）。

## ADR-CR164-004 DSR raw-count mode and non-alias ceiling

- 决定：schema 强制 `dsr_input_method=raw_trial_count`、`raw_trial_count`、`raw_trial_count_ref`、`family_ref/hash`、observed Sharpe、sample length、skew、kurtosis；`effective_trial_count=null`、ref/method 为空、availability=`typed_unavailable`。
- 语义：可生成独立的 raw-count DSR evidence，但不能满足要求 effective count 的现有 deflated-performance/admission claim；consumer 必须分别表达 method evidence 与 claim availability。
- 禁止：复制 raw count 到 effective 字段，或以 DSR present 推导 C1/admission-ready。

## ADR-CR164-005 Conservative disagreement lattice

- 决定：claim schema 列出 mandatory methods；聚合顺序 `blocked > fail > typed_unavailable > pass`。仅当 mandatory set 全部 PASS 才可 claim PASS。
- reason codes：`method_input_blocked`、`method_policy_failed`、`mandatory_method_unavailable`、`method_evidence_ref_missing`、`method_disagreement_conservative_block`。
- 例：BH PASS + PBO FAIL → FAIL；BH PASS + DSR unavailable → TYPED_UNAVAILABLE；任一 BLOCKED → BLOCKED。非 mandatory informational 方法不得改善或恶化 claim，必须另列。

## ADR-CR164-006 Existing consumers only

- 决定：输出投影到 CR151、CR154 与 admission package 3/3；UC-59/60 仅兼容消费，不新增 real adapter；所有 consumer 采用 worst-state merge。
- 禁止：新建平行 gate、直接修改 lineage、由 consumer 重算统计。

## ADR-CR164-007 Deterministic local artifact contract

- 决定：JSON-safe finite values、稳定 key/sequence、schema-versioned canonical serialization 和 SHA-256 summary hash；相同 fixture 10 次 hash distinct count=1；orphan refs=0。
- 备注：精确文件路径、dataclass 字段类型与 atomic write 策略由 CP5 LLD 冻结，本 ADR 不授权 I/O 实现。

## ADR-CR164-008 Design-only authorization

- 决定：CP3 仅批准设计与五个 CP4 outcome 输入。CP5 前不得改源代码/测试；不运行真实统计、数据、provider、NAS、broker、外部框架或远端写入。

## Decision Register

| DQ | 对应 ADR | 推荐批准内容 | 备选 / 回退 |
|---|---|---|---|
| DQ-CP3-CR164-001 | 001/002/007 | envelope + pure calculators + sealed binding | 修改架构后重跑 CP3 |
| DQ-CP3-CR164-002 | 003/004 | fixed-window bootstrap + raw-count DSR non-alias | automatic selector 另行验证；或暂停相关 method |
| DQ-CP3-CR164-003 | 005/006 | severity lattice + existing consumers only | 不允许 OR/多数票；范围变化回 CP2 |
| DQ-CP3-CR164-004 | 008 | 五 Story 输入且 design-only | 修改或暂停 CR164 |
