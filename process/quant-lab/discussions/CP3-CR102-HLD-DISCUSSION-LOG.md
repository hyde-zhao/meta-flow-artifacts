# CP3 CR102 HLD Discussion Log

## 基本信息

| 字段 | 内容 |
|---|---|
| CR | CR-102 |
| 主题 | NAS real package exchange authorization architecture |
| 时间 | 2026-06-20T21:49:03+08:00 |
| 记录人 | host-orchestrator |
| 讨论模式 | 主进程 inline advisor；未启动子 agent |
| 执行边界 | 只读取和写入本地 process/docs 账本，不访问 NAS、凭据、账户、runtime、交易或 publish |

## Architecture Gray Areas

| 灰区 ID | 问题 | 方案形成结论 | 影响章节 |
|---|---|---|---|
| AGA-CR102-01 | NAS gate 是一次性执行门，还是分层授权门？ | 选择 two-plane authorization design；CP3 只确认架构，不授权执行 | HLD §2/§3/§4 |
| AGA-CR102-02 | path/mount/permission 由谁提供？ | 只能由用户显式提供；agent 不探测、不推断、不读取 env | HLD §1/§5/§13 |
| AGA-CR102-03 | 后续 CP5 是否允许真实检查脚本？ | CP5 只产出 authorization matrix / execution plan shell / evidence schema，不执行 NAS | HLD §6/§8/§14 |
| AGA-CR102-04 | 如何处理 CR089 overlap？ | 保持 CR102 独立 gate；不恢复 CR089 / QMT runtime | HLD §10/§11/§14 |

## Advisor Table

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| A. Two-plane authorization design | 权限边界清晰；CP3 不触碰 NAS；后续可逐项授权 | 真实验证延后 | HLD、CP5、证据、安全 | 推荐 | 当前 path/mount/permission 未授权；用户要求最小权限 |
| B. Single-pass real NAS check | 进度快 | 未授权 path/mount/check 时风险高 | NAS、权限、证据 | 不推荐 | 用户后续明确提供 path/mount/permission 且授权 check |
| C. Merge back into CR089 | 一个 CR 处理完整运行链 | 扩大到 QMT/runtime/account 风险 | CR089、runtime、NAS | 不推荐 | 用户明确恢复 CR089 并接受 runtime gate |
| D. Cancel real NAS gate | 外部风险最低 | 真实交付链缺口保留 | follow-up tracking | 条件备选 | 用户不再需要真实 NAS exchange 验证 |

## 讨论结论

- 推荐方案：A. Two-plane authorization design。
- CP3 待人工决策项：DQ-CP3-CR102-01..05。
- 不授权项保持：NAS access/list/read/write/copy/publish/pull/delete/mount/check、凭据/env/账户、QMT/MiniQMT/XtQuant/gateway runtime、submit/cancel/simulation/live、provider/lake/catalog publish。
- 真实 NAS path/export/root/mount/identity 仍为 `UNSET_BY_USER`。

## Deferred Architecture Ideas

| ID | 想法 / 风险 | 延后原因 | 触发条件 |
|---|---|---|---|
| DAI-CR102-01 | check-only read permission preflight | 当前 check=false 且 path/mount 未提供 | 用户后续明确授权 check 并提供 path/mount/permission |
| DAI-CR102-02 | publish/pull/copy package exchange execution | publish/pull/copy 均为 high-risk 外部副作用 | 用户提供对象、身份、回退、证据格式并通过高风险 gate |
| DAI-CR102-03 | CR089 runtime combined validation | 会扩大到 QMT/MiniQMT/runtime/account | 用户明确恢复 CR089 并接受 runtime_authorization |
