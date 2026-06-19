---
discussion_id: "CP3-CR099-HLD-DISCUSSION"
cr_id: "CR-099"
status: "ready-for-cp3-review"
owner: "host-orchestrator"
created_at: "2026-06-19T15:54:07+08:00"
source_context: "process/context/CP2-CR099-RUNNER-REAL-READONLY-SMOKE-CONTEXT.yaml"
---

# CP3 CR099 HLD Discussion Log

## 讨论目标

本讨论记录用于审计 CR099 HLD 的 Architecture Gray Areas。CR099 的目标是让 runner 在逐 run 授权下真实消费 Windows gateway readonly path，并输出脱敏 evidence；CP3 只确认架构，不授权运行。

## Advisor Table

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| A. CR098 runner adapter + CR097 gateway pattern + CR099 run contract | 证明 runner integration；复用既有 adapter / gateway；授权和 evidence 可审计 | 需要 CP5 细化 run manifest 与 evidence schema | runner、qmt client、runtime authorization、redacted evidence | 推荐 | CP5 能定义完整 run contract，用户后续可逐 run 授权 |
| B. 用户手工 Windows-side 执行并回传脱敏 evidence | Codex 不触碰 secret / env；适合无法安全代理运行的环境 | 需要用户按步骤执行；自动化较弱 | manual evidence、redaction、verification | fallback | Codex 不能安全读取 env 或 gateway 不适合由 WSL 侧发起时切换 |
| C. 只做离线 reverify | 风险最低；不触碰 runtime | 不能关闭真实 runner runtime 风险 | tests、offline fixture | 仅作为 blocked fallback | 用户拒绝 runtime 授权或环境不可用 |
| D. 合并 NAS / order-write / 非空复测 | 覆盖更多后续事项 | 范围和权限风险显著扩大 | NAS、trading write、account data | 不推荐 / 禁止 | 只能另起独立 CR |

## Architecture Gray Areas

| 灰区 ID | 问题 | 推荐结论 | 状态 |
|---|---|---|---|
| AGA-CR099-01 | 真实运行由谁执行 | CP5 后按 run contract 和逐 run 授权执行；无法安全代理时切换为用户手工执行 | decision-item |
| AGA-CR099-02 | env / HMAC material 如何进入 | 只通过授权引用和 run contract 进入；无授权不得读取 | decision-item |
| AGA-CR099-03 | evidence schema 如何收敛 | 使用 CR099 专用 redacted evidence schema，禁止 raw payload | decision-item |
| AGA-CR099-04 | 是否合并后续目标 | 不合并；非空 / 交易日 / NAS / order-write 独立 | decision-item |

## 决策映射

| 灰区 | CP3 决策项 | HLD 章节 |
|---|---|---|
| AGA-CR099-01 | DQ-CP3-CR099-01 / DQ-CP3-CR099-02 | §5 / §6 / §9 |
| AGA-CR099-02 | DQ-CP3-CR099-02 / DQ-CP3-CR099-03 | §8 / §9 |
| AGA-CR099-03 | DQ-CP3-CR099-03 / DQ-CP3-CR099-04 | §7 / §10 |
| AGA-CR099-04 | DQ-CP3-CR099-05 | §3 / §16 |

## 不授权边界

- CP3 讨论不授权读取 HMAC secret 或 Windows `.env`。
- CP3 讨论不授权启动 gateway、执行 runner runtime 或查询账户。
- CP3 讨论不授权输出账户、证券代码、数量、资金、委托、成交或日志原文。
- CP3 讨论不授权 NAS、交易写、simulation/live、provider/lake/catalog publish 或 release publish。
