---
review_id: "cr098-runner-readonly-integration"
cr_id: "CR-098"
status: "PASS_WITH_RISK"
owner: "host-orchestrator"
created_at: "2026-06-19T12:22:28+08:00"
---

# CR098 Runner Readonly Integration Review

## Findings

未发现 BLOCKER / HIGH / MEDIUM 实现缺陷。

## 评审摘要

| 维度 | 结论 | 说明 |
|---|---|---|
| 需求一致性 | PASS | 实现限定为 runner readonly integration，没有扩大到 NAS / order-write / runtime。 |
| 架构一致性 | PASS | runner 通过 injected `QmtClient` / transport，不直接依赖 Windows runtime。 |
| 安全边界 | PASS_WITH_RISK | 默认 fake/offline 和未授权 fail-closed 通过；真实 runtime 仍需单独授权验证。 |
| 测试覆盖 | PASS_WITH_RISK | 离线 / fixture / 回归覆盖充分；真实 runner smoke 未授权未执行。 |
| 脱敏 | PASS | raw/sensitive readonly payload 被阻断，evidence 仅保存摘要字段。 |
| 回归 | PASS | CR091 / CR020 相关回归通过。 |

## 人工 / 语义质量审查

- Happy path 覆盖 fake default 与 injected transport fixture。
- Negative path 覆盖缺 runtime authorization、order-like endpoint 和 sensitive/raw payload。
- 文档与过程证据明确说明 CP5/CP7 不授权真实 runtime。
- 剩余风险已分级，不应在 CP8 中被静默降级为 READY without risk。

## 建议

- CP8 使用 `READY_WITH_RISK`，除非用户明确放弃真实 runner smoke 风险。
- 真实 runner smoke 应另行逐 run 授权，不复用 CR097 的一次性 HMAC env 授权。
