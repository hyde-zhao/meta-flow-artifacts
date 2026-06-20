---
checkpoint_id: "CP2"
checkpoint_name: "CR101 Cross-Platform Strategy Delivery Scope Review"
type: "manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-06-20T00:30:00+08:00"
reviewed_by: "user"
reviewed_at: "2026-06-20T08:17:33+08:00"
auto_check_result: "process/checks/CP2-CR101-CROSS-PLATFORM-STRATEGY-DELIVERY-SCOPE-PRECHECK.md"
target:
  phase: "requirement-clarification"
  story_id: "CR101"
  artifacts:
    - "process/changes/CR-101-CROSS-PLATFORM-STRATEGY-DELIVERY-ADAPTER-REALIGNMENT-2026-06-20.md"
---

# CP2 CR101 Cross-Platform Strategy Delivery Scope Review

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP2-CR101-CROSS-PLATFORM-STRATEGY-DELIVERY-SCOPE-PRECHECK.md` | PASS | 0 | 启动前无 active formal CR；FU-CR091-005 已转正式 CR101；CR089 保持 blocked。 |

自动终验授权: false。本 CP2 只发起 scope / risk / runtime-authorization 人工确认，不授权自动终验、真实运行、凭据读取、NAS、交易或 publish。

## Decision Brief

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CR091-FU-05-START-HANDOFF-2026-06-19.md` |
| capsule 状态 | ready |
| read_profile | minimal |
| 默认读取策略 | 先读 handoff、STATE、当前需求基线、CR-INDEX、CR091 follow-up tracking；CR098 / CR100 / HLD / strategy_runner 代码按需读。 |
| 全文档读取扩展 | 0 |
| 缺失 / waived 理由 | N/A |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `process/STATE.md` | scanned | 1 | 1 | 当前无 active formal CR，CR101 启动后进入 CP2 pending |
| current baseline | `process/baseline/CURRENT-REQUIREMENT-BASELINE.yaml` | scanned | 4 | 4 | runner / target / adapter / not_authorized 均纳入 |
| follow-up tracking | `process/changes/CR-091-FOLLOW-UP-TRACKING-2026-06-18.md` | scanned | 5 | 5 | FU-CR091-005 为候选；FU-CR091-003 被其阻塞 |
| CR index | `process/changes/CR-INDEX.yaml` | scanned | 4 | 4 | CR089 blocked；FU-CR091-005 可启动；runtime candidates 暂缓 |
| startup handoff | `process/context/CR091-FU-05-START-HANDOFF-2026-06-19.md` | scanned | 4 | 4 | DQ 草案已汇总 |
| 用户显式约束 | 当前对话 | scanned | 6 | 1 | 不授权 NAS、凭据、runtime、simulation/live、交易、publish |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP2-CR101-01 | scope | 是否启动 CR101，并将本轮限定为跨平台策略交付与 adapter layer 重对齐？ | 启动 standard CR101，仅做设计 / schema / fixture / checker / package layout 重对齐，不直接实现真实运行链路。 | 暂缓启动，保留 FU-CR091-005 为 candidate；或只做文档勘误不进入 CR。 | 推荐方案能先修正历史 `miniqmt_runner` 混淆；暂缓会继续阻塞 CR089 / order-write；只做勘误缺少后续实现门禁。 | 影响 manifest、checker、package layout、adapter protocol 和后续 HLD / CP5；不证明真实 runtime ready。 | 若 CP3 发现范围过大，回退到 CP2 拆分 target taxonomy 与 adapter protocol 两个 CR。 |
| DQ-CP2-CR101-02 | architecture | runner 与 MiniQMT 的关系如何冻结？ | runner 固定为 quant-lab-side；MiniQMT 只是当前 broker / execution adapter target，不承载策略运行职责。 | MiniQMT runner-hosted；或保留双 runner 模型。 | 推荐方案职责清晰并可扩展到 Goldminer；MiniQMT-hosted 会把 runner 和接口宿主耦合；双 runner 增加验证矩阵。 | 影响 `strategy_runner` 边界、adapter protocol、HLD、checker 和 package manifest。 | 若未来确需平台宿主 runner，另起 architecture CR 并定义独立 target。 |
| DQ-CP2-CR101-03 | implementation | 当前策略交付 target 支持哪些平台？ | 当前 implemented 仅 QMT direct-run；manifest / validation / package layout 保留 future targets，但不实现 Goldminer / generic Python。 | 同时实现 MiniQMT target；或只做抽象 schema 不交付 QMT target。 | 推荐方案匹配“当前仅支持 QMT”且控制范围；同时实现 MiniQMT 会扩大风险；只做抽象无法解除 CR089 阻塞。 | 影响 package generation、entrypoint、fixture、checker、docs/qmt HLD。 | 若 QMT direct-run target 无法离线建模，CP3 回退为 Spike 或拆分。 |
| DQ-CP2-CR101-04 | runtime_authorization | 本 CR 是否授权真实 NAS / QMT / MiniQMT / XtQuant / gateway / simulation/live / 交易 / publish？ | 不授权。本 CR 只允许离线文件、schema、fixture、checker 和本地包结构验证；真实验证后续逐 run gate。 | 授权用户手工真实验证；或授权 agent 代跑 runtime / NAS / publish。 | 推荐方案权限最小且符合当前对话；用户手工验证需要单独 evidence schema；agent 代跑越过当前边界。 | 不证明真实 NAS、QMT direct-run、MiniQMT adapter 或交易链路 ready。 | 需要真实验证时另起 `QMT-DIRECT-RUN-VALIDATION-FU`、`MINIQMT-GATEWAY-ADAPTER-VALIDATION-FU` 或 NAS gate。 |
| DQ-CP2-CR101-05 | risk_acceptance | 是否接受先完成离线重对齐、真实运行后置的 READY_WITH_RISK 路线？ | 接受离线优先路线；CP8 若真实 runtime 未执行，可按证据结论关闭为 READY_WITH_RISK 并保留后续 gate。 | 等真实环境授权后再做；或把 CR101 关闭为 blocked。 | 推荐方案先修正架构和交付契约；等待真实环境会阻塞设计；blocked 不能消除当前混淆。 | 残余风险是离线验证不证明真实 runtime、NAS 权限或交易安全。 | 若用户不接受 READY_WITH_RISK，CP2 退回并改为 runtime 授权 CR。 |

| 字段 | 内容 |
|---|---|
| 推荐决策 | `approve`：接受 DQ-CP2-CR101-01..05 推荐方案，进入 CP3 HLD；不授权真实运行。 |
| 备选方案 | 暂缓候选；拆分为 target taxonomy CR 和 adapter protocol CR；转 runtime authorization CR；关闭为 blocked。 |
| 影响维度 | 用户价值、实现复杂度、可验证性、维护成本、平台兼容、安全 / 权限、交付影响。 |
| 优劣分析 | 推荐方案先修正架构真相并保持离线可验证；备选要么扩大权限，要么延迟解除 CR089 / FU-CR091-003 阻塞。 |
| 风险与回退 | 风险等级 medium-high；CP3 可拆分或转 Spike；CP8 可 READY_WITH_RISK；真实验证必须另起 gate。 |
| 用户需决策事项 | DQ-CP2-CR101-01、DQ-CP2-CR101-02、DQ-CP2-CR101-03、DQ-CP2-CR101-04、DQ-CP2-CR101-05。 |

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| workspace check 已通过 | PASS | `process_link_health: ok` | 可写 process |
| CR tracking 检查通过 | PASS | `meta-flow check cr-tracking --strict-warnings` OK | 启动前无 active formal CR |
| 正式 CR 已创建 | PASS | `process/changes/CR-101-CROSS-PLATFORM-STRATEGY-DELIVERY-ADAPTER-REALIGNMENT-2026-06-20.md` | 当前 gate 为 CP2 pending |
| 不授权边界明确 | PASS | 当前对话 | 不授权真实 NAS / 凭据 / runtime / simulation/live / 交易 / publish |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | 范围限定为重对齐，不直接实现真实链路 | PASS | DQ-CP2-CR101-01 | 用户回复“同意”，接受推荐方案。 |
| 2 | runner / MiniQMT 边界清晰 | PASS | DQ-CP2-CR101-02 | 用户回复“同意”，接受推荐方案。 |
| 3 | 当前 target 与 future target 边界清晰 | PASS | DQ-CP2-CR101-03 | 用户回复“同意”，接受推荐方案。 |
| 4 | runtime authorization 明确不授权 | PASS | DQ-CP2-CR101-04 | 用户回复“同意”，接受不授权边界。 |
| 5 | READY_WITH_RISK 路线可接受 | PASS | DQ-CP2-CR101-05 | 用户回复“同意”，接受离线优先路线。 |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| CP2 scope approved | PASS | 用户回复“同意” | 按 `approve` 处理。 |
| CP3 输入稳定 | PASS | DQ-CP2-CR101-01..05 | 可进入 CP3 HLD。 |
| 不授权项未被误读为执行授权 | PASS | DQ-CP2-CR101-04 | CP2 approval 不授权真实运行、凭据、NAS、交易或 publish。 |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| Formal CR | `process/changes/CR-101-CROSS-PLATFORM-STRATEGY-DELIVERY-ADAPTER-REALIGNMENT-2026-06-20.md` | PASS | CP2 approved，进入 CP3。 |
| CP2 precheck | `process/checks/CP2-CR101-CROSS-PLATFORM-STRATEGY-DELIVERY-SCOPE-PRECHECK.md` | PASS | 自动预检 PASS。 |
| CP2 review | `process/checkpoints/CP2-CR101-CROSS-PLATFORM-STRATEGY-DELIVERY-SCOPE-REVIEW.md` | PASS | 本文件已回填人工审查结果。 |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-06-20T08:17:33+08:00
- 修改意见：用户回复“同意”，接受 DQ-CP2-CR101-01..05 的推荐方案。
- 风险接受项：接受离线优先重对齐路线；CP2 approval 不授权 NAS、凭据、QMT/MiniQMT/XtQuant/gateway runtime、simulation/live、交易或 publish。
