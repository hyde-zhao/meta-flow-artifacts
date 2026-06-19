# CP3 CR093 HLD Discussion Log

## 背景

CR093 来源于 CR091 / CR092 follow-up 候选 `CR091-FU-04 Ledger hygiene`。用户已在 `2026-06-18T17:45:00+08:00` 同意 CP2，确认本 CR 只治理 CR019 / CR025 历史账本与 `cr-tracking` 一致性，不授权立即修改 checker 或旧账本。

本讨论日志记录 HLD 形成前的 Architecture Gray Areas 和 advisor table-first 输入。它是审计和恢复材料，不替代 HLD、CP3 Decision Brief 或后续 CP5 readiness。

## Architecture Gray Areas

| 灰区 ID | 关键问题 | 为什么影响架构 | 影响面 | 推荐 |
|---|---|---|---|---|
| AGA-CR093-01 | checker 应全文扫描还是分层读取当前状态与 audit-only 历史文本？ | 决定 CR025 / CR092 nested active_change 是否继续误报。 | checker parser / STATE / history | 推荐分层读取。 |
| AGA-CR093-02 | CR019 旧台账是改文本还是让 checker 接受状态等价关系？ | 决定修复是数据规整还是规则语义化。 | CR019 tracking / formal CR status | 推荐规则语义化 + 最小文本规范化。 |
| AGA-CR093-03 | source=cp8-follow-up 缺少 tracking row 的旧 warning 是否一并强修？ | 可能扩大到 CR053/058/074/075/078/080/092 等旧 CR。 | CR-INDEX / follow-up tracking | 推荐本轮只保留 warning，不作为 exit 1 阻断。 |
| AGA-CR093-04 | 是否直接 suppress 旧失败？ | 决定后续是否还能发现真实状态冲突。 | checker quality / audit risk | 不推荐 suppress。 |

## Advisor Table

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| A. 当前状态语义化 checker + 最小台账规范化 | 修根因；保留历史；可测试；改动面受控。 | 需要新增 parser / fixture 测试。 | checker / STATE / CR019 tracking / CR-INDEX | 推荐 | 若 CP5 发现 parser 成本过高，可降级为最小台账规范化。 |
| B. 只改 CR019 / STATE 文本迎合旧 checker | 代码改动少，短期可能通过。 | 保留全文误扫缺陷，后续 history 仍可能误报。 | CR019 tracking / STATE text | 不推荐 | 只有用户要求不改脚本时切换。 |
| C. suppress 已知错误 | 最快。 | 容易隐藏真实冲突，审计价值低。 | checker output / risk | 不推荐 | 只适合作为临时 unblock，需明确到期。 |
| D. 重写全量 CR ledger checker | 长期统一。 | 本轮成本高，容易扩大范围。 | checker architecture / tests | 备选 Spike | 状态机规则继续扩张时另起 CR。 |

## 方案形成输入与处理结果

| 来源 | 影响的 HLD 章节 | 处理结果 | 说明 |
|---|---|---|---|
| lane-product | HLD §1 / §6 | adopted | 用户目标是消除当前账本检查阻断，而不是启动 runtime。 |
| lane-architecture | HLD §3 / §4 / §5 / §10 | adopted | 推荐三层状态读取模型。 |
| lane-quality | HLD §7 / §8 / §9 | adopted | 必须用 fixture 防止真实 current conflict 被错降级。 |
| lane-docs-check | HLD §11 / §12 | adopted | CP3 approve 不等于实施授权，不授权项必须重复列出。 |

## Deferred Architecture Ideas

| ID | 想法 / 风险 / 扩展方向 | 延后原因 | 触发切换或重启条件 |
|---|---|---|---|
| DAI-CR093-01 | 全量 CR ledger checker 重写 | 当前 CR 目标集中在 CR019 / CR025 旧账。 | 后续 warning / 状态规则继续膨胀。 |
| DAI-CR093-02 | 一并清理 cp8-follow-up missing tracking warnings | 范围扩大，容易触碰多个旧 CR 台账。 | 用户要求清理 warning-only 项。 |
| DAI-CR093-03 | schema 化 CR tracking 数据源 | 成本较高。 | checker 状态集合不可维护时。 |
