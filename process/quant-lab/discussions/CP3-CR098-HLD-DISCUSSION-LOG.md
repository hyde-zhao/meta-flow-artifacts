---
discussion_id: "CP3-CR098-HLD-DISCUSSION"
cr_id: "CR-098"
status: "recorded"
created_at: "2026-06-19T12:00:19+08:00"
owner: "host-orchestrator"
---

# CP3 CR098 HLD Discussion Log

## Architecture Gray Areas

| 灰区 ID | 问题 | 为什么影响架构 | 来源 | 处理结论 |
|---|---|---|---|---|
| AGA-CR098-01 | runner 真实 readonly adapter 是作为 `ReadonlyGatewayClient` 扩展，还是新建并行 runtime adapter？ | 决定文件 owner、测试复用、fake/real 切换方式和回归范围。 | CR091 runner baseline、CP2 `DQ-CP2-CR098-02` | 推荐扩展为 runner-owned adapter facade，内部可注入 fake/real transport。 |
| AGA-CR098-02 | HMAC client env 由 runner 隐式读取还是由运行门禁显式传入？ | 决定 secret 暴露面和是否会把 CP3 误解为 runtime 授权。 | CP2 `DQ-CP2-CR098-03/04`、CR097 evidence 路由 | 推荐显式 env path + per-run authorization；默认不读。 |
| AGA-CR098-03 | evidence 是复用 CR091 evidence summary，还是创建 CR098 专用 runtime evidence schema？ | 决定下游 checker、CP7 证据和敏感字段泄露防线。 | CR091 evidence、CR097 redacted JSON | 推荐复用 CR091 summary 思路并新增 CR098 runtime fields。 |
| AGA-CR098-04 | 是否把 NAS package intake 并入 runner integration？ | 决定是否触碰 NAS 权限和 CR089/CR091-FU-02 边界。 | CP2 `DQ-CP2-CR098-05` | 不并入；保持独立候选。 |

## Advisor Table

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| A. runner-owned readonly gateway adapter facade | 复用 runner 现有 fake wrapper；真实 transport 可注入；默认离线安全；便于测试 | 需要扩展 adapter/evidence 合同 | `trading/strategy_runner/readonly_gateway.py`、tests、evidence | 推荐 | CP5 确认文件 owner 和 runtime env contract 后实施 |
| B. 独立 runtime runner CLI，绕过现有 wrapper | 与现有离线代码隔离 | runner 集成证明弱；重复 client/evidence 逻辑 | CLI、docs、runtime guide | 不推荐 | 若现有 wrapper 无法扩展才切换 |
| C. runner 直接 import XtQuant | 调用链短 | 触碰 Windows-only 依赖、账号、session 和交易接口风险 | runner runtime、依赖、账户 | 禁止作为本轮方案 | 只有未来独立高风险 CR 才能讨论 |
| D. 合并 NAS package exchange | 能同时验证包交换 | 范围膨胀，权限面扩大 | NAS、package、runner | 不推荐 | 用户启动 CR091-FU-02 时单独处理 |

## 讨论结论

- 推荐方案：A. runner-owned readonly gateway adapter facade。
- 关键边界：默认 fake/offline；真实 transport 只在逐 run runtime authorization 后启用。
- HLD 拆分判断：单一核心产物为 runner readonly integration，Story 预计不超过 4 个，不拆分 HLD。
- 延后项：NAS package exchange、order-write、非空持仓 / 交易日复测、实盘只读验证。
