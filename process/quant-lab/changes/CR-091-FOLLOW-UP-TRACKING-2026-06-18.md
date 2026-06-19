---
source_cr: "CR-091"
status: "open"
created_at: "2026-06-18T15:58:59+08:00"
created_by: "host-orchestrator"
updated_at: "2026-06-19T18:45:00+08:00"
checkpoint_source: "CP8"
cr_index_path: "process/changes/CR-INDEX.yaml"
---

# CR091 后续事项跟踪台账

## 目的

本台账记录 CR091 CP8 后续候选事项。候选项未启动前不得预创建正式 CR；只有用户决定推进某一项时，才在 `process/changes/` 下创建正式 CR，并把本台账中的状态改为 `active`。

CR091 已于 `2026-06-18T15:40:09+08:00` 关闭为 `closed-current-delivery / READY_WITH_RISK`。本台账不重开 CR091，也不授权真实 runtime、NAS、凭据、账户或交易动作。

## 分流总览

| 类别 | 数量 | 阻断当前交付 | 说明 |
|---|---:|---|---|
| 关闭范围 | 1 | 否 | CR091 离线 runner 研究 / 设计 / 实现 / fixture 验证已关闭 |
| 不授权范围 | 1 | 否 | 真实运行、NAS、凭据、账户、交易写和 publish 均不授权 |
| 风险接受项 | 3 | 否 | `R-CR091-CP7-001..003` 已由 CP8 接受 |
| 后续 CR 候选项 | 4 | 否 | `CR091-FU-01..04` |
| 取消 / deferred 项 | 1 | 否 | 真实交易、真实账户和真实运行全部后置 |

## 后续 CR / Spike 候选索引

| 候选编号 | 标题 | 状态 | 类型 | 优先级 | 影响面 / 冲突键 | 正式 CR 路径 | 相关 active CR / blocked_by / superseded_by | 当前门控 | 阻塞原因 | 下一步 | 来源 |
|---|---|---|---|---:|---|---|---|---|---|---|---|
| CR091-FU-01 | Real QMT readonly runtime smoke design gate | closed | CR | 1 | runtime_authorization; per_run_authorization; readonly_query_positions; credential_boundary; redacted_evidence | `process/changes/CR-092-REAL-QMT-READONLY-RUNTIME-SMOKE-DESIGN-GATE-2026-06-18.md` | `CR-092` | closed | CR092 已关闭为 READY_WITH_RISK；真实运行仍未授权 | 后续候选转入 `process/changes/CR-092-FOLLOW-UP-TRACKING-2026-06-18.md` | DQ-CP8-CR091-04 |
| CR091-FU-02 | NAS package exchange gate | closed | CR | 2 | nas_package_exchange; package_pull; package_publish; credential_boundary | `process/changes/CR-100-NAS-PACKAGE-EXCHANGE-OFFLINE-READINESS-GATE-2026-06-19.md` | `CR-100` | closed | CR100 已关闭为 READY_WITH_RISK；真实 NAS 仍未授权 | 后续真实 NAS publish / pull / copy / 校验需独立 gate | DQ-CP8-CR091-04 |
| CR091-FU-03 | Order-write / submit-cancel design gate | candidate | CR | 3 | submit_cancel; order_write; simulation_live; trading_runtime_boundary |  |  | 未启动 | 最高风险，必须独立 CR | 不建议立即启动 | DQ-CP8-CR091-04 |
| CR091-FU-04 | Ledger hygiene | closed | CR | 4 | CR019 tracking; CR025 nested active_change; cr_tracking consistency | `process/changes/CR-093-LEDGER-HYGIENE-CR019-CR025-TRACKING-CLEANUP-2026-06-18.md` | `CR-093` | closed | CR093 已关闭为 READY_WITH_RISK；R-CR093-01/02 已由用户接受 | 当前交付已关闭；后续 warning cleanup / checker convergence 需用户另选候选后新建 CR | DQ-CP8-CR091-04 |

## CR 冲突预检

| 检查项 | 结果 | 证据 | 处理结论 |
|---|---|---|---|
| 是否已有 `STATE.md.active_change` | PASS | CR092 创建前顶层 `active_change` 为空 | 允许启动 CR092 |
| 是否存在未关闭正式 CR | PASS_WITH_RISK | `meta-flow check cr-tracking` 报历史旧账，但无当前 active formal CR 锁 | 历史旧账转 CR091-FU-04 |
| 正式文档影响面是否重叠 | PASS | CR091 / CR089 相关但均非 active execution | CR092 只做设计门禁 |
| Story / LLD 批次是否重叠 | PASS | CR092 新建 scoped LLD 批次 | 不复用 CR091 LLD 为执行授权 |
| 文件 owner 是否冲突 | PASS | 当前不改代码 | CP5 前不得实施 |
| 外部接口 / 安全 / 运行授权是否重叠 | PASS_WITH_RISK | 真实 runtime 高风险 | 进入 standard 模式和人工门禁 |
| `STATE.md.active_change` 是否指向已关闭 CR | PASS_WITH_RISK | 顶层已切为 CR092；历史嵌套 CR025 旧账仍存在 | 不阻塞 CR092，保留给 CR091-FU-04 |
| 台账候选与正式 CR 文件是否同步 | PASS | 本台账 `CR091-FU-01` 链接 CR092 | 已同步 |

## 状态索引同步

| 对象 | 路径 | 同步要求 | 当前状态 |
|---|---|---|---|
| 运行时状态 | `process/STATE.md.cr_tracking` | 记录 active、blocked、candidate、spike_candidate、stale_status_conflicts | CR093 closed-current-delivery；无 active formal CR |
| CR 索引 | `process/changes/CR-INDEX.yaml` | 记录每个候选项的状态、正式 CR 路径、影响面、blocked_by 和下一步 | CR093 closed-current-delivery |
| 一致性检查 | `meta-flow check cr-tracking --project-root .` | 新增台账、启动候选、关闭 CR 或状态冲突修复后执行 | CR019 / CR025 旧账阻断已治理；剩余 broader warnings 为 warning-only |

## 不授权范围

| 项目 ID | 范围 | 原因 | 需要未来授权时的动作 | 来源 |
|---|---|---|---|---|
| NA-CR091-01 | QMT / MiniQMT / XtQuant / gateway / runner runtime、NAS、凭据、账户、submit/cancel、simulation/live、provider/lake/publish | CR091 CP8 只关闭离线 runner 交付；CR092 创建也只进入设计门禁 | 必须在独立人工门禁和逐 run 授权中明确 | DQ-CP8-CR091-03 |

## 风险接受项

| 项目 ID | 风险 | 接受条件 | 回退 / 切换条件 | 来源 |
|---|---|---|---|---|
| RA-CR091-01 | 离线 CP7 不证明真实 runtime ready | CP8 接受 READY_WITH_RISK | 启动 runtime authorization gate | R-CR091-CP7-001 |
| RA-CR091-02 | 历史 cr-tracking 旧账仍存在 | 不阻塞 CR091 / CR092 当前设计门禁 | 启动 ledger hygiene | R-CR091-CP7-002 |
| RA-CR091-03 | evidence output path 未启用 | 后续真实 evidence 需独立 gate | 纳入 CR092 或 future evidence output gate | R-CR091-CP7-003 |

## 关闭范围

| 项目 ID | 已关闭内容 | 关闭证据 | 来源 |
|---|---|---|---|
| CLOSE-CR091-01 | CR091 离线 runner 研究 / 设计 / 实现 / fixture 验证 | `process/checkpoints/CP8-CR091-DELIVERY-READINESS.md` | CP8 |

## 取消 / Deferred 项

| 项目 ID | 内容 | 状态 | 原因 | 可重启条件 | 来源 |
|---|---|---|---|---|---|
| DEF-CR091-01 | 真实交易、真实账户、真实运行 | deferred | 不在 CR091 关闭范围；CR092 也先做设计门禁 | 用户明确授权并通过对应门禁 | CP8 |
