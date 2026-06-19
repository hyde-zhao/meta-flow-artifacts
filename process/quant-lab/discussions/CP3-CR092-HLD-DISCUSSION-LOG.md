# CP3 CR092 HLD Discussion Log

## 背景

CR092 来源于 CR091 CP8 后续候选 `CR091-FU-01`。用户已在 `2026-06-18T16:17:35+08:00` 同意 CR092 CP2 的 5 项决策，确认本 CR 当前只进入真实只读 runtime smoke 的设计门禁，不授权真实 runtime。

本讨论日志用于记录 HLD 形成前的 Architecture Gray Areas 和 advisor table-first 输入。它是人类审计和恢复材料，不替代 HLD、CP3 Decision Brief 或后续 CP5 readiness。

## Architecture Gray Areas

| 灰区 ID | 关键问题 | 为什么影响架构 | 影响面 | 推荐 |
|---|---|---|---|---|
| AGA-CR092-01 | 真实只读 smoke 是否应由 runner 直接调用 gateway，还是由人工执行脚本调用？ | 决定模块边界、执行授权、evidence 归属和失败回退方式。 | runner / manual guide / evidence / runtime authorization | 推荐“人工逐 run 执行 + runner 只提供设计合同 / 脱敏 evidence 模板”，CP5 前不实现真实调用。 |
| AGA-CR092-02 | 是否复用 CR089 只读 smoke 经验？ | CR089 与 CR092 都涉及 query_positions，但 CR089 仍 blocked-readiness-approved；复用不当会误启动旧门禁。 | CR089 / CR092 / QMT runtime / NAS boundary | 推荐只复用脱敏 evidence 和 scope 思路，不复用 CR089 状态或自动启动。 |
| AGA-CR092-03 | evidence 是否允许落盘？ | evidence 落盘可能泄露账户、持仓、委托、成交或日志原文；必须定义输出目录、脱敏字段和敏感扫描。 | evidence / security / validation | 推荐 CP3 只定义 evidence contract；CP5 才决定是否允许本地脱敏输出目录。 |
| AGA-CR092-04 | smoke 成功后是否自动解锁 NAS package exchange 或 order-write？ | 成功的只读 smoke 可能被误解为交易能力可用；必须避免 scope creep。 | NAS / order-write / simulation/live / follow-up CR | 推荐只读 smoke 不解锁 NAS 或 order-write；二者继续保持 CR091-FU-02 / FU-03。 |

## Advisor Table

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| A. 人工逐 run 只读 smoke 设计门禁 | 权限最小；用户可在交易主机本地控制执行；不会让 Codex 读取凭据或账户原文。 | 自动化程度低；需要清晰 manual guide 和 evidence 模板。 | runtime authorization / evidence / docs / CP5 readiness | 推荐 | 若未来需要自动化，必须另起 runtime execution CR。 |
| B. runner 内置真实 readonly transport | 自动化强；离线 runner 到真实网关路径更短。 | 容易误读为 runtime 授权；实现会触碰 gateway、凭据和账户边界。 | code / gateway / credentials / account / tests | 不推荐当前 CP3 | 只有 CP5 明确批准并逐 run 授权后才可评估。 |
| C. 直接恢复 CR089 runtime smoke | 复用已有路径和历史经验。 | CR089 仍 blocked；可能带入 NAS package exchange 和旧授权语义。 | CR089 / CR092 / NAS / runtime | 不推荐 | 若用户明确要求合并 CR092 到 CR089，需重新做 CR 影响分析。 |
| D. 暂停真实 smoke，保持 CR091 离线状态 | 风险最低；不触碰任何真实环境。 | 无法推进真实只读 readiness 设计。 | backlog / risk acceptance | 备选 | 如果用户 reject CP3 或不再需要 runtime 路线，则回退。 |

## 方案形成输入与处理结果

| 来源 | 影响的 HLD 章节 | 处理结果 | 说明 |
|---|---|---|---|
| lane-product | §1 / §5 / §6 | adopted | 用户目标是从 CR091 离线 runner 过渡到真实只读 smoke，但先做设计门禁。 |
| lane-architecture | §3 / §4 / §8 / §9 | adopted | 推荐人工逐 run smoke，runner 不直接连接真实 gateway。 |
| lane-quality | §7 / §12 / §13 | adopted | evidence 必须脱敏，forbidden counters 必须覆盖全部禁止动作。 |
| lane-docs-check | §15 / §17 | adopted | 后续必须输出 manual guide / TEST-PLAN / rollback 边界。 |

## Deferred Architecture Ideas

| ID | 想法 / 风险 / 扩展方向 | 延后原因 | 触发切换或重启条件 |
|---|---|---|---|
| DAI-CR092-01 | runner 内置真实 readonly transport | 当前 CP2 只批准设计，不批准 runtime 实现。 | CP5 approved 且用户逐 run 授权。 |
| DAI-CR092-02 | NAS package exchange 与 runtime smoke 合并 | NAS 是独立高风险边界。 | 用户启动 CR091-FU-02。 |
| DAI-CR092-03 | submit/cancel 或 simulation/live 设计 | order-write 风险远高于 readonly smoke。 | 用户启动 CR091-FU-03。 |
| DAI-CR092-04 | 修复 CR019 / CR025 账本旧账 | 与 runtime smoke 架构无直接关系。 | 用户启动 CR091-FU-04 ledger hygiene。 |
