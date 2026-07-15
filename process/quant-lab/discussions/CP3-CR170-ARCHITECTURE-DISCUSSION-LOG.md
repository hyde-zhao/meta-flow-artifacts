---
status: ready-for-human-decision
version: "0.2"
change: CR-170
phase: solution-design
execution_mode: inline-fallback
dispatch_ref: AD-CR170-CP3-META-SE-INLINE-20260715T125800+0800
---

# CP3 CR-170 Architecture Discussion Log

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 0.1 | 2026-07-15 | host-orchestrator inline meta-se-critical | 收敛四个 Architecture Gray Areas、advisor 输入、候选方案与 CP3 Decision Brief；保留用户“不拉起子 Agent”约束和 inline-fallback 证据。 |
| 0.2 | 2026-07-15 | host-orchestrator inline meta-se-critical | 处理 CP3 深度评审：逐 Gate 核实 21-unit 现有 N/A 路径，冻结更严/受控放宽/保持 15/5/1，补 caller/auth-ref 与 T3 existing behavior；更新 AGA-02/03，不新增 DQ。 |

## Architecture Gray Areas

| AGA | 状态 | 推荐 | 备选 | 进入 DQ |
|---|---|---|---|---|
| AGA-CR170-01 五态代码承载 | OPEN-FOR-CP3 | 21-unit 表驱动 internal typed decision；不改 global bool helper | per-Gate patch；global helper change | DQ-POLICY-FORM |
| AGA-CR170-02 complete N/A 结果 | OPEN-FOR-CP3 | reviewable applicable -> NR；prohibited G1-P06 -> BLOCKED；方向分组 15/5/1 | all BLOCKED；complete=present | DQ-COMPLETE-NA |
| AGA-CR170-03 Gate6 两层职责 | OPEN-FOR-CP3 | 保护 bottom-up merge；只硬化 T0/T1/T2；T3 existing early-return 不改 | rewrite merge；tier 下沉 | DQ-ADMISSION |
| AGA-CR170-04 compatibility/verifier | OPEN-FOR-CP3 | public/schema/adapter 不变；FU006 future consumer | delete adapter；implement verifier | DQ-COMPATIBILITY |

## Advisor Table

| 视角 | 已核事实 | 建议 | 风险 |
|---|---|---|---|
| 产品 / 用户 | UC-58 是 target-artifact internal hardening；四类 consumer 中 independent verifier 尚未实现 | future-consumer 边界写入 HLD/ADR/CP3；当前由 Gate maintainer 自验证 | 错误声称 independent verification |
| 架构 | `_has_na_reason` 只返回 bool，无法表达 applicability/owner/boundary/tier | internal policy + five-state decision；公共 callable 不变 | 全局 helper 改义爆炸半径过大 |
| Gate owner | Gate 1-5 有 21-unit 判定面；Gate1 escape 可被别的 unavailable claim 掩盖 | inventory 固定 6/6/1/5/3；Gate1 三层断言 | 只看最终 status 会漏判 |
| Direction review | 全量代码核实显示：15 个 existing escape/structured-N/A PASS、5 个 no-N/A BLOCKED/NR 路径、1 个固定 value/provenance validation | inventory 显式增加 baseline path、direction、disposition；按 15/5/1 分组回归 | 只写统一 complete-N/A 结果会掩盖方向相反的行为变化 |
| Admission owner | `evaluate_shared_contract` 已传播 NEEDS_REVIEW；resolver 当前可能升 PASS | merge 先回归、通过则不改；resolver 最小硬化 | 重写正确 merge 或 tier 语义反转 |
| 兼容维护 | CR168/169 adapter 仍有 typed payload/pre/post guard 职责 | 只回归不删除；FU009 四条件+ADR 才评估 | canonical 修复被误读为 adapter 可删 |
| 安全 / 交付 | current runner 不调用 canonical；Stage3/real data 未授权 | `stage3_entry_ready=false`；独立 Launch CR | 把 static hardening 误称 Stage3 ready |

## 用户输入收敛

1. 用户已批准 CP2 产品范围与五项推荐产品决策。
2. 用户要求按评审补强“独立验证者为 future consumer”并继续到 CP3。
3. 用户此前明确不拉起子 Agent；本阶段沿用已记录的 inline-fallback，不伪造 subagent dispatch。
4. 评审提出的 consumer 端到端视角已进入 UC-58 第六步、HLD §2.3/§8 和 CP3 Decision Brief。
5. 新一轮评审指出 Gate 1 unit 现有路径不同；Host 对 Gate 1-5 全量复核后确认应整改 inventory 精度，而不扩大 CR 范围。
6. LLD 输入已冻结：caller 显式提供 `n_a_boundaries/authorization_ref`，evaluator 不合成；T3 当前 BLOCKED+NOT_AUTHORIZED early-return 只回归、不改生产逻辑。

## 结论

产品范围和目标无需扩大；表驱动方案 A 不变。CP3 推荐方案精确化为 21-unit 方向分组 15/5/1、policy-specific complete-N/A disposition、caller authoring 边界与 T3 零修改。需要用户确认的仍是原四项 DQ，不重新打开 Stage3、aggregate 或 verifier 实现。
