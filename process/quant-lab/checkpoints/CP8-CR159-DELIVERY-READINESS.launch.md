# CP8 人工门禁发起：CR-159 Delivery Readiness（scoped READY_WITH_RISK）

## 门禁路径

- **checklist**：`process/checkpoints/CP8-CR159-DELIVERY-READINESS.md`
- **自动预检结果**：`process/checks/CP8-CR159-DELIVERY-READINESS.result.json`（decision: PASS）
- **context capsule**：`process/context/CP8-CR159-DELIVERY-CONTEXT.yaml`（read_profile=compact）
- **follow-up 台账**：`process/changes/CR-159-FOLLOW-UP-TRACKING-2026-07-06.md`（FU-CR159-001 / FU-CR159-002）

## 自动预检结论

`cp result-check` OK · `human-gate` OK · scoped verifier 3 PASS（B1 zero / B2 registered / A budget）· `event check` OK。

## 审批者摘要

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 确认 CR-159 CP8 delivery readiness（scoped）：三子项完成（A: STATE slim + 删 phase + 4 residual WARN；B1: CR-158 status→closed + required_evidence 留痕；C: N/A 工具已有校验），B2 + authz L2 infer deferred FU-CR159-001，4 residual WARN deferred FU-CR159-002，路标 v1.3 回写 |
| 推荐动作 | `approve`：批准 CR-159 以 `READY_WITH_RISK` 关闭（scoped，非全局 0 ERROR） |
| approve 后会发生什么 | CR-159 关闭为 `READY_WITH_RISK`；STATE→delivered, active_change=null；CR-INDEX item CR-159→closed |
| approve 不授权什么 | Git remote write/push、true release/publish、runtime、真实数据 lake/NAS/provider、凭据、broker、trading、catalog/store/registry 写入、外部框架，重开 closed CR-151/152/153/154/158，全局 `cr check` 0 ERROR 承诺 |
| 不确认会阻塞什么 | CR-159 保持 active，不进入 delivered |

## Context Capsule 摘要

- **capsule**：`process/context/CP8-CR159-DELIVERY-CONTEXT.yaml`（read_profile=compact）
- **默认读取策略**：capsule-first；消费 CP8 result、follow-up台账、路标 v1.3、scoped verifier 证据。
- **全文档读取**：无需读取完整产品需求/HLD/LLD；实施证据由 scoped verifier 可执行命令机械核验。

## 实施关键发现（implementation correction）

- **CR-158 status**：CP2 DQ 推荐的 `closed-current-delivery-ready-with-risk` 实施验证发现**非合法 status enum**（cr check 报 invalid）；实际采用 `closed`，READY_WITH_RISK 语义保留在 `release_decision`/`summary`/`risk acceptance`，不编码进 `status` 字段。
- **authz L2**：cr check 用 `record_from_cr_file`（formal CR），不读 CR-INDEX item `required_evidence`；authz L2 来自 formal CR 正文 "real lake" 关键字触发 `infer_required_evidence_from_text` 误判（CR-151/152/153/154 的**不授权声明**被当作 required_evidence）。修正需改 4 个 closed CR 正文或 meta-flow infer 逻辑，deferred FU-CR159-001。
- **STATE 4 residual WARN**：`current_agent`/`last_action`/`cr_tracking_ref`/`cr_ledger_ref` 因 state-router/SKILL.md:104,114,290 契约保留；FU-CR159-002（schema allowlist 扩展）重访。

## 决策分层

| 分类 | 数量 |
|---|---:|
| 必须用户决策 | 1 |
| 高风险策略确认 | 1（READY_WITH_RISK scoped） |
| agent 默认处理 | 0 |
| 仅审计记录 | 3（CR-156/158 闭环、路标 v1.2 滞后修正、子项 C N/A） |

## 决策收集覆盖摘要

| 来源 | 扫描状态 | 候选问题数 | 纳入待决策数 |
|---|---|---:|---:|
| cp8_result | scanned | 1 | 1 |
| follow_up_tracking | scanned | 2 | 0（candidate） |
| product_baseline_docs | N/A | 0 | 0 |

## 本轮待人工决策项：1 项

| 决策 ID | 类型 | 推荐方案 |
|---|---|---|
| DQ-CP8-CR159-RELEASE | risk_acceptance | 批准 READY_WITH_RISK（scoped）；接受 4 residual WARN + B2 deferred + authz L2 infer + CR-158 status implementation correction |

完整决策表见 checklist `### 待人工决策清单`。

**风险接受**：`R-CR159-STATE-RESIDUAL-WARN`、`R-CR159-CP8-SCOPED-NOT-GLOBAL`、`R-CR159-AUTHZ-L2-DATA-FIX-VS-CAPABILITY-CHANGE`、`R-CR159-CR158-STATUS-IMPLEMENTATION-CORRECTION`。

## 回复与不授权项

如果你回复 approve，表示接受 CR-159 以 READY_WITH_RISK 关闭（scoped）；回复 修改: <具体修改点> 调整；回复 reject 拒绝，CR-159 保持 active。

approve 不表示授权以下不授权项：Git remote write/push、true release execution/publish、runtime、真实数据 lake/NAS/provider、凭据、broker、trading、catalog/store/registry 写入、外部框架，重开 closed CR-151/152/153/154/158，全局 cr check 0 ERROR 承诺。
