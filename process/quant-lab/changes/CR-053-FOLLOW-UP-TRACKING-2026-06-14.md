---
status: "active"
version: "1.0"
change_id: "CR-053"
source_gate: "CP8"
created_at: "2026-06-14T13:05:00+08:00"
release_decision: "READY_WITH_RISK"
---

# CR-053 Follow-up Tracking

> 本台账只记录 CR053 CP8 后续候选，不自动创建正式 CR，不授权真实迁移、NAS、数据湖、交易、凭据读取或 git remote 操作。host-orchestrator 回收后可按用户确认同步 `STATE.md.cr_tracking`。

## 1. 汇总

| 项目 | 内容 |
|---|---|
| 当前 CR053 结论建议 | `READY_WITH_RISK`，静态 migration inventory / dry-run 交付就绪。 |
| 风险接受候选 | `R-CR053-01`、`R-CR053-02`、`R-CR053-03` |
| 后续候选数量 | 4 |
| 不授权项 | 9 项，见 CP8 checkpoint 和 release context。 |

## 2. 风险接受候选

| Risk ID | 分流类别 | 决策类型 | 推荐处理 | 备选方案 | owner | 验收标准 | 重访条件 |
|---|---|---|---|---|---|---|---|
| R-CR053-01 | risk_acceptance | risk_acceptance | 接受真实 NAS path / capacity / permission 未验证，不阻断 CR053 静态 close。 | 备选：NOT_READY，要求先做 NAS read-only inventory；不推荐，因为会扩大 CR053 授权范围。 | human / host-orchestrator | CP8 用户明确接受；后续真实迁移前有独立 NAS 授权。 | 用户要求真实 NAS 迁移或 CR060+ 启动。 |
| R-CR053-02 | risk_acceptance | risk_acceptance | 接受 backup / restore rehearsal 和 rollback_ref planned-only，不阻断 CR053 静态 close。 | 备选：NOT_READY，要求先生成 git bundle / restore evidence；不推荐，因为 CR053 未授权真实备份恢复。 | human / host-orchestrator | CP8 用户明确接受；CR058/CR060+ 前不得绕过 rollback_ref。 | CR058 repo-local move 或 CR060+ NAS 实迁启动前。 |
| R-CR053-03 | risk_acceptance | follow_up_tracking | 接受 CR058 manual review / rollback gates 为未来前置。 | 备选：立即合并到 CR053 返工；不推荐，因为会扩大当前 close gate 范围。 | host-orchestrator | CR058 启动前有独立 checkpoint 和门禁。 | 用户明确启动 CR058。 |

## 3. 后续候选

| Candidate ID | 分流类别 | 决策类型 | 推荐处理 | 备选方案 | owner | 验收标准 | 重访条件 |
|---|---|---|---|---|---|---|---|
| FU-CR053-01 | follow_up_candidate | follow_up_tracking | closed-current-delivery：已转正式 `CR-058`，路径 `process/changes/CR-058-REPO-LOCAL-MECHANICAL-MIGRATION-RELAYOUT-GATE-2026-06-14.md`；CP2/CP3/CP5 approved，CP6 static-only/no-op PASS，CP7 static-only PASS_WITH_RISK，CP8 于 2026-06-14T16:59:36+08:00 经用户“同意”关闭为 `READY_WITH_RISK`。 | 备选 A：如用户要求真实迁移，启动后续 CR059 execution gate。备选 B：单项豁免改写历史引用，需 path / owner / risk / rollback_ref / approval_decision_id。 | host-orchestrator / human | CR058 static-only/no-op relayout gate 已关闭；真实 move / rewrite 仍需 CR059 或等价独立 runtime_authorization gate。 | CR059 仅作为后续候选，不自动启动；不授权真实迁移。 |
| FU-CR053-02 | follow_up_candidate | runtime_authorization | 将 CR060+ NAS / archive real migration 保持为后续候选，不自动启动。 | 备选 A：先做 NAS read-only inventory spike；适用于需要路径事实。备选 B：取消 NAS 实迁，仅保留 Git-only。 | host-orchestrator / human | 独立 NAS 授权、路径白名单、容量/权限验证、restore rehearsal、failure manifest。 | 用户提供 NAS 路径并要求真实迁移。 |
| FU-CR053-02A | closed-current-delivery | runtime_authorization | CR059 已完成 NAS 准备基础设施：NFS 挂载、systemd unit、目录骨架、manifest seed、backup point plan。真实 NAS / archive data copy 仍未执行。 | 备选：如需要撤销准备，禁用 systemd unit 并保留空目录；不删除 NAS 数据。 | host-orchestrator / human | `process/changes/CR-059-MIGRATION-PREPARATION-FOUNDATION-2026-06-14.md`、`docs/release/NAS-MOUNT-PLAN-CR059.md`、`docs/release/NAS-DIRECTORY-MAP-CR059.md` 可读。 | 后续真实 NAS/archive migration 启动前重新验证 mount、freeze manifest、生成 backup evidence。 |
| FU-CR053-01B | closed-current-delivery | runtime_authorization | CR060 已执行并关闭 repo-local docs-only identity convergence：README / USER-MANUAL 切到 `quant-lab` canonical，并保留 `local_backtest` legacy alias；CP8 于 2026-06-15T07:14:03+08:00 经用户“批准”关闭为 `READY_WITH_RISK`。 | 备选 A：如需撤销，按 rollback 文档反向修订。备选 B：后续先做 CR062 Git remote cutover。 | host-orchestrator / human | `process/changes/CR-060-REPO-LOCAL-MECHANICAL-MIGRATION-EXECUTION-GATE-2026-06-15.md`、`docs/release/MIGRATION-EXECUTION-MANIFEST-CR060.yaml`、`process/checkpoints/CP8-CR060-DELIVERY-READINESS.md` 可读。 | Git remote、NAS、lake、runtime、package/import 和历史 rewrite 仍需后续 CR；CR060 关闭不授权这些操作。 |
| FU-CR053-01C | active | follow_up_tracking | CR061 已按用户 2026-06-15 请求转为正式 `Package / Import / Layout Convergence` CR，路径 `process/changes/CR-061-PACKAGE-IMPORT-LAYOUT-CONVERGENCE-2026-06-15.md`；用户于 2026-06-15T07:52:05+08:00 回复“同意”，CP2/CP3/CP5 门禁包已按 approve 回填。当前仍不执行真实代码迁移。 | 备选 A：保持 CR061 gate-only 后暂停；备选 B：先切 CR062 Git remote；备选 C：要求补更细 import scan 后再审。 | host-orchestrator / human | `docs/release/CR061-PACKAGE-IMPORT-CANDIDATE-LIST.md`、`process/checkpoints/CP5-CR061-PACKAGE-IMPORT-LAYOUT-READINESS.md` 可读。 | 下一步需实现授权门禁；Git/NAS/lake/runtime/凭据/CR046 recovery 不从 CR061 继承授权。 |
| FU-CR053-03 | follow_up_candidate | runtime_authorization | 将独立 data lake migration CR 保持为候选，不从 CR053 继承授权。 | 备选 A：保持 `MARKET_DATA_LAKE_ROOT` 永久不动。备选 B：先做只读 lake policy audit。 | host-orchestrator / data owner | 独立 backup / restore drill、lake root 切换方案、publish gate 和回滚证据。 | 用户明确要求真实数据湖迁移或 root replacement。 |
| FU-CR053-04 | follow_up_candidate | runtime_authorization | 将 trading / QMT / MiniQMT runtime authorization 保持为候选，不自动启动。 | 备选 A：只做 package manifest offline review。备选 B：取消交易运行授权，保留研究端流程。 | host-orchestrator / trading owner | 独立交易 runtime gate、凭据边界、脱敏证据、账户查询/交易授权分离。 | 用户要求连接 QMT/MiniQMT、账户查询或交易动作。 |

## 4. 不授权项分流

| Item ID | 分流类别 | 决策类型 | 推荐处理 | 备选方案 | owner | 验收标准 | 重访条件 |
|---|---|---|---|---|---|---|---|
| NA-CR053-01 | not_authorized | runtime_authorization | 本轮不授权 NAS mount / scan / mkdir / copy / delete / migration。 | 单项授权 NAS read-only inventory；不推荐，需新 CR 或 Spike。 | human | CP8 approve 明确不授权。 | 用户提供路径并发起 NAS 授权。 |
| NA-CR053-02 | not_authorized | runtime_authorization | 本轮不授权真实目录 move / rename / delete 或 repo-local mechanical move。 | 单项授权 repo-local move；不推荐，需 CR058。 | human | CP8 approve 明确不授权。 | CR058 启动。 |
| NA-CR053-03 | not_authorized | runtime_authorization | 本轮不授权 `MARKET_DATA_LAKE_ROOT` replacement 或真实数据湖迁移。 | 只读 lake policy audit；需新门禁。 | human / data owner | CP8 approve 明确不授权。 | 数据湖迁移诉求出现。 |
| NA-CR053-04 | not_authorized | security | 本轮不授权 Windows full archive / cold / full lake mount。 | 只读 package exchange review；需交易主机安全设计。 | trading owner | CP8 approve 明确不授权。 | 交易主机映射诉求出现。 |
| NA-CR053-05 | not_authorized | security | 本轮不授权凭据、`.env`、token、password、cookie、session、private key 读取。 | N/A；默认禁止。 | human / security owner | CP8 approve 明确不授权。 | 独立凭据处理方案获批。 |
| NA-CR053-06 | not_authorized | runtime_authorization | 本轮不授权 provider fetch / lake write / catalog publish。 | 只读 catalog audit；需新门禁。 | data owner | CP8 approve 明确不授权。 | 数据发布诉求出现。 |
| NA-CR053-07 | not_authorized | runtime_authorization | 本轮不授权 QMT / MiniQMT runtime、账户查询或交易动作。 | 离线 package review；需新门禁。 | trading owner | CP8 approve 明确不授权。 | 交易运行诉求出现。 |
| NA-CR053-08 | not_authorized | runtime_authorization | 本轮不授权 git push / tag / remote rename / history rewrite。 | 本地 commit / bundle checkpoint；需用户单独授权。 | human / host-orchestrator | CP8 approve 明确不授权。 | 用户要求远端发布或 rename。 |
| NA-CR053-09 | not_authorized | follow_up_tracking | 本轮不自动启动 CR058 / CR060+ 或真实迁移。 | 用户明确启动单个 follow-up。 | human / host-orchestrator | CP8 approve 明确不授权。 | 用户选择推进后续 CR。 |
