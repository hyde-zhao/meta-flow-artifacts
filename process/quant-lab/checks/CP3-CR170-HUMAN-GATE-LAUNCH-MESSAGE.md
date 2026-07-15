请审查人工门禁 `CP3-CR170-CANONICAL-RELIABILITY-HLD-REVIEW`。

checklist 路径: `process/checkpoints/CP3-CR170-CANONICAL-RELIABILITY-HLD-REVIEW.md`
自动预检结论: 已生成 Decision Brief；发起前请以 `meta-flow check human-gate --checkpoint process/checkpoints/CP3-CR170-CANONICAL-RELIABILITY-HLD-REVIEW.md` 的结果为准。

审批者摘要:
- 本次确认服务的整体目标: 请见 checkpoint `### 审批者摘要`；本消息只承载发起确认所需摘要。
- 推荐动作: 默认推荐 approve，除非你要求调整范围、风险、授权边界或推荐方案。
- approve 后会发生什么: 接受本轮 Decision Brief 中列出的推荐方案，并允许进入 checkpoint 声明的下一阶段。
- approve 不授权什么: 不授权真实运行、凭据读取、安全边界变更、外部接口调用、数据写入、publish、live / 交易类操作。
- 不确认会阻塞什么: 当前 checkpoint 后续推进会保持等待，直到你回复 approve / 修改 / reject。

Context Capsule: 请见 checkpoint 的 `### Context Capsule Summary`，其中包含 capsule、read_profile、默认读取策略和全文档读取边界。
决策收集覆盖: 请见 checkpoint 的 `### Decision Collection Coverage`，本消息只承载发起确认所需摘要。
决策分层:
- 必须用户决策: 4
- 高风险策略确认: 以 checkpoint `### 决策分层` 为准。
- agent 默认处理: 低风险、可逆、局部实现细节默认不进入用户主确认表，完整清单以 checkpoint 为准。
- 仅审计记录: 已按项目规则处理且无需用户拍板的事项只保留审计摘要。
本轮待人工决策项: 4
blocking / high-risk 决策摘要: DQ-CP3-CR170-POLICY-FORM, DQ-CP3-CR170-COMPLETE-NA, DQ-CP3-CR170-ADMISSION, DQ-CP3-CR170-COMPATIBILITY

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP3-CR170-POLICY-FORM | architecture | 五态如何进入代码且避免全局误伤？ | 21-unit 表驱动 internal typed decision；Gate 局部消费；不改 global bool helper。 | per-Gate patch；global helper change。 | 推荐可审计且一致；per-Gate 易漂移；global helper 无法表达 boundary。 | 决定 LLD/module 形态；错误抽象可能扩大五个 Gate 的回归面。 |
| DQ-CP3-CR170-COMPLETE-NA | security | 完整结构化 N/A 能否视为 evidence PASS，且哪些 policy 允许 reviewable N/A？ | complete_na_disposition=reviewable` 时 applicable complete N/A -> auditable NR；G1-P06=`prohibited` 继续 BLOCKED；generic/incomplete 非 PASS；按更严/受控放宽/保持 15/5/1 回归。 | all N/A BLOCKED；complete N/A=present。 | 推荐保留合法 N/A 又不重开虚假 PASS，并保护 trial-count/provenance；all blocked 过严；as-present 不安全。 | 决定 Gate/tier 真值表与双向爆炸半径；错误选择会重开 PASS、误伤合法 N/A 或放松固定数值校验。 |
| DQ-CP3-CR170-ADMISSION | architecture | bottom-up merge 与 admission resolver 如何分工？ | 先回归并保护 merge；只最小硬化 T0/T1/T2；T3 现有 BLOCKED+NOT_AUTHORIZED early-return 只做 1/1 回归、production diff=0。 | rewrite merge；tier 下沉；新增 status enum。 | 推荐避免重写正确逻辑、T3 与 public break；备选扩大爆炸半径。 | 影响 Gate6 worst-state 与 admission 兼容；重写 merge/T3 可能破坏既有 caller。 |
| DQ-CP3-CR170-COMPATIBILITY | scope | adapters/verifier/runner 是否随 canonical 硬化一起改变？ | public API/schema/adapters 不变；FU006 为 future verifier；FU009 四条件+ADR 才评估 adapter 简化。 | 本 CR 删除 adapter或实现 verifier。 | 推荐保持 defense-in-depth 与 CR 边界；备选扩大范围且丢独立性证据。 | 影响后续 FU006/FU009；current runner integration 仍为0；误删 guard 会降低 fail-closed。 |

如果你回复 approve，表示你接受以上 4 项推荐方案，不表示授权以下不授权项。
不授权项: 不授权真实运行、凭据读取、安全边界变更、外部接口调用、数据写入、publish、live / 交易类操作；若 checkpoint 中列有额外不授权项，以 checkpoint 为准。

请只回复以下三个 exact 选项之一：
- approve
- 修改: <具体修改点>
- reject
