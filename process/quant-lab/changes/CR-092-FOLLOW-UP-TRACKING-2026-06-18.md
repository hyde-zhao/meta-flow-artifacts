---
source_cr: "CR-092"
status: "open"
created_at: "2026-06-18T17:25:00+08:00"
created_by: "host-orchestrator"
updated_at: "2026-06-19T10:45:00+08:00"
checkpoint_source: "CP8"
cr_index_path: "process/changes/CR-INDEX.yaml"
---

# CR092 后续事项跟踪台账

## 目的

本台账记录 CR092 CP8 后续候选事项。CR092 当前交付已关闭为 `closed-current-delivery / READY_WITH_RISK`，交付范围仅包括 readiness / evidence guardrail：manual guide、模拟账户 evidence template、显式单文件静态 checker、测试和验证 / 发布就绪证据。

本台账不授权真实 QMT / MiniQMT / XtQuant / gateway / runner runtime、NAS、凭据、真实账户、submit/cancel、simulation/live、provider/lake/catalog/publish。候选项未启动前不得预创建正式 CR；只有用户明确选择推进某一项时，才在 `process/changes/` 下创建正式 CR，并把本台账中的状态改为 `active`。

## 分流总览

| 类别 | 数量 | 阻断当前交付 | 说明 |
|---|---:|---|---|
| 关闭范围 | 1 | 否 | CR092 readiness / evidence guardrail 已关闭 |
| 不授权范围 | 1 | 否 | runtime、NAS、凭据、真实账户、交易写和 publish 均不授权 |
| 风险接受项 | 4 | 否 | `R-CR092-CP7-001..004` 已由 CP8 接受 |
| 后续 CR 候选项 | 4 | 否 | `CR092-FU-02` 已转为 CR097 active；保留的 `CR091-FU-02..04` 仍是候选 / closed |
| 取消 / deferred 项 | 1 | 否 | 手工验证 / 真实运行均后置，需独立授权 |

## 后续 CR / Spike 候选索引

| 候选编号 | 标题 | 状态 | 类型 | 优先级 | 影响面 / 冲突键 | 正式 CR 路径 | 相关 active CR / blocked_by / superseded_by | 当前门控 | 阻塞原因 | 下一步 | 来源 |
|---|---|---|---|---:|---|---|---|---|---|---|---|
| CR-096 | User-provided simulated-account evidence checker run | closed | CR | 1 | simulated_evidence; single_file_read; credential_boundary; redacted_evidence | `process/changes/CR-096-USER-PROVIDED-SIMULATED-EVIDENCE-CHECKER-RUN-2026-06-19.md` | closed-current-delivery | closed | 用户已确认 `.quant-lab` 方案并授权按合同填入；`/home/hyde/.quant-lab/evidence/qmt/cr096/redacted/cr096-user-provided-evidence.yaml` 已通过单文件 checker；CP8 已同意关闭；不得读取目录、凭据、真实账户或 NAS | 当前交付已关闭；真实 runtime smoke 需独立 CR092-FU-02 授权 | DQ-CP8-CR092-04 |
| CR-097 | Real readonly runtime smoke per-run authorization | active | CR | 2 | runtime_authorization; per_run_authorization; qmt_runtime; readonly_query_positions; credential_boundary | `process/changes/CR-097-REAL-QMT-READONLY-RUNTIME-SMOKE-PER-RUN-AUTHORIZATION-2026-06-19.md` | active-formal-cr | runtime-preflight | 用户已明确授权真实 QMT runtime；仍需确认 QMT 客户端运行位置、账户模式、只读入口、evidence 输出路径和人工中止协助 | 收集 H-CR097-01..05 后执行 runtime preflight | DQ-CP8-CR092-04 |
| CR091-FU-02 | NAS package exchange gate | candidate | CR | 3 | nas_package_exchange; package_pull; package_publish; credential_boundary |  |  | 未启动 | 涉及 NAS，必须独立授权 | 等待用户选择 | DQ-CP8-CR092-04 |
| CR091-FU-03 | Order-write / submit-cancel design gate | candidate | CR | 4 | submit_cancel; order_write; simulation_live; trading_runtime_boundary |  |  | 未启动 | 最高风险，必须独立 CR | 不建议立即启动 | DQ-CP8-CR092-04 |
| CR091-FU-04 | Ledger hygiene | closed | CR | 5 | CR019 tracking; CR025 nested active_change; cr_tracking consistency | `process/changes/CR-093-LEDGER-HYGIENE-CR019-CR025-TRACKING-CLEANUP-2026-06-18.md` | `CR-093` | closed | CR093 已关闭为 READY_WITH_RISK；R-CR093-01/02 已由用户接受 | 当前交付已关闭；后续 warning cleanup / checker convergence 需用户另选候选后新建 CR | DQ-CP8-CR092-04 |

## 不授权范围

| 项目 ID | 范围 | 原因 | 需要未来授权时的动作 | 来源 |
|---|---|---|---|---|
| NA-CR092-01 | QMT / MiniQMT / XtQuant / gateway / runner runtime、NAS、凭据、真实账户、submit/cancel、simulation/live、provider/lake/publish | CR092 CP8 只关闭 readiness / evidence guardrail 交付 | 必须在独立人工门禁和逐 run 授权中明确 | DQ-CP8-CR092-03 |

## 风险接受项

| 项目 ID | 风险 | 接受条件 | 回退 / 切换条件 | 来源 |
|---|---|---|---|---|
| RA-CR092-01 | 静态 CP7 不证明真实 runtime readiness | CP8 接受 READY_WITH_RISK | 用户要求真实证明时启动 per-run runtime gate | R-CR092-CP7-001 |
| RA-CR092-02 | 未验证用户实际模拟账户 evidence | 用户明确不手工验证当前 evidence | 用户提供 evidence 文件路径后启动 CR092-FU-01 | R-CR092-CP7-002 |
| RA-CR092-03 | 未启动独立 meta-qa 子代理 | 当前只做静态 checker / docs 交付 | 高风险 runtime 验证时应升级独立 QA | R-CR092-CP7-003 |
| RA-CR092-04 | CR019 / CR025 历史账本旧账仍存在 | 不阻塞 CR092 closure | 启动 CR091-FU-04 ledger hygiene | R-CR092-CP7-004 |

## 关闭范围

| 项目 ID | 已关闭内容 | 关闭证据 | 来源 |
|---|---|---|---|
| CLOSE-CR092-01 | CR092 readiness / evidence guardrail：manual guide、template、checker、tests、CP6/CP7/CP8 证据和 release docs | `process/checkpoints/CP8-CR092-DELIVERY-READINESS.md` | CP8 |

## 取消 / Deferred 项

| 项目 ID | 内容 | 状态 | 原因 | 可重启条件 | 来源 |
|---|---|---|---|---|---|
| DEF-CR092-01 | 当前 CP8 内不做手工 evidence 验证，也不做真实 runtime smoke | deferred | 用户明确“不手动验证了”，接受其余 CP8 项 | 用户后续明确提供 evidence 文件或授权 runtime gate | CP8 |
