---
checkpoint_id: "CP2-CR079-DATA-REPORTS-ACCESS-BASELINE"
checkpoint_name: "CR079 Data/reports Access Baseline"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-06-17T05:56:16+08:00"
reviewed_by: "user"
reviewed_at: "2026-06-17T06:12:28+08:00"
auto_check_result: "process/checks/CP2-CR079-DATA-REPORTS-ACCESS-BASELINE.md"
context_capsule: "process/context/CP2-CR079-DATA-REPORTS-ACCESS-CONTEXT.yaml"
auto_final_authorization: false
target:
  phase: "requirement-clarification"
  story_id: ""
  artifacts:
    - "process/changes/CR-079-REAL-DATA-REPORTS-ACCESS-GATE-2026-06-17.md"
---

# CP2 CR079 Data/reports Access Baseline 人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP2-CR079-DATA-REPORTS-ACCESS-BASELINE.md` | PASS | 0 | CR079 formal CR 已创建；本轮只发起真实访问门禁，不在批准前访问 `data/` 或 `reports/`。 |

## Decision Brief

推荐决策：批准 CR079 CP2 范围基线，正式启动 `CR076-real-access-candidate` 的最小真实访问门禁；CP2 只确认范围和安全边界，不单独授权访问，实际只读清点仍需 CP2 / CP3 / CP5 全部 approved。

备选方案：保持 `CR076-real-access-candidate` 不启动；扩大为内容读取 / 恢复 / 删除 CR；取消 data/reports 后续访问。推荐方案优先，因为它把“真实 access”收敛为可审计的 metadata-only 只读清点，避免直接触碰数据内容或破坏性动作。

影响维度：CR tracking、STATE、target/legacy root 的 `reports/` 和 `data/`、CR076 后续候选、CR077 old root retirement、CR078 remote Git governance、CR046 paused boundary、安全 / 权限边界。

风险与回退：若范围表述不清，回退为 `CR076-real-access-candidate`；若用户要求内容读取、恢复、复制或删除，必须修订 CR079 或另起更高风险执行 CR。回退只修改 process ledger，不访问或清理数据。

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP2-CR079-DATA-REPORTS-ACCESS-CONTEXT.yaml` |
| capsule 状态 | ready-for-human-gate |
| read_profile | compact |
| 默认读取策略 | 先读 capsule；仅在缺失、冲突、字段不足、人工审计或深度评审时读取完整文档 |
| 全文档读取扩展 | 0 次 |
| SGQ / discussion | N/A：用户明确要求启动真实 data/reports access CR，来源为 CR076 已登记 follow-up candidate。 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `process/STATE.md.human_gate_decisions.pending_human_decisions[]` | scanned | 0 | 0 | 创建 CR079 前无当前 gate DQ；本文件创建后回写。 |
| CR079 formal CR | `process/changes/CR-079-REAL-DATA-REPORTS-ACCESS-GATE-2026-06-17.md` | scanned | 5 | 5 | 范围、冲突、授权和不授权边界纳入 DQ。 |
| CR076 CP8 | `process/checkpoints/CP8-CR076-DELIVERY-READINESS.md` | scanned | 1 | 1 | `CR076-real-access-candidate` 是本 CR 来源。 |
| CR-INDEX follow-up | `process/changes/CR-INDEX.yaml` | scanned | 4 | 3 | real access、CR077、CR078 分流纳入；CR046 recovery 不通过本 CR 启动。 |
| discussion log / checkpoint | `process/discussions/*` / `process/checks/*DISCUSSION-CHECKPOINT.json` | n/a | 0 | 0 | 本轮不是新增产品场景发现；范围来自 CR follow-up。 |
| 委托 Agent 交还摘要 | `process/handoffs/*RETURN-SUMMARY.md` | n/a | 0 | 0 | 本轮不调度功能 agent。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP2-CR079-01 | scope | 是否正式启动 CR079？ | 启动 CR079，进入 CP3/CP5 真实访问门禁。 | 保持 candidate；取消 CR079。 | 推荐方案响应用户请求并保留审计；保持 candidate 不解决 readiness。 | 新增正式 CR 和待审门禁。 | reject 时恢复为 candidate 或取消。 |
| DQ-CP2-CR079-02 | scope | 范围是否限定为 metadata-only 只读清点？ | 限定为存在性、顶层项、数量、规模、mtime 和风险标记。 | 纳入内容读取；纳入恢复 / 删除。 | 推荐方案是最小真实访问；备选触碰更高敏感面。 | 不证明文件内容质量。 | 需要内容验证时另起或修订 CR。 |
| DQ-CP2-CR079-03 | security | 是否继续禁止 `.env`、凭据、账户和交易事实？ | 继续禁止。 | 同时授权凭据读取；授权账户 / 交易事实查询。 | 推荐方案维持最小权限；备选需要更高安全审批。 | 防止凭据和交易事实泄露。 | 用户另起 credential / runtime CR 时切换。 |
| DQ-CP2-CR079-04 | runtime_authorization | CP2 approve 是否单独授权执行访问？ | 不授权；必须 CP2/CP3/CP5 全部 approved 且 preflight PASS。 | CP2 后立即执行；一次性授权全部后续访问。 | 推荐方案符合门禁；备选边界过宽。 | 避免 CP2 越权变成执行许可。 | 三门 approved 后才可执行最小清点。 |
| DQ-CP2-CR079-05 | risk_acceptance | 是否接受 metadata-only 清点可能仍无法证明 readiness？ | 接受，并把内容质量 / 恢复路径作为后续风险。 | 现在扩大到内容读取或恢复；取消访问。 | 推荐方案先获得低风险事实；扩大范围风险高。 | data/reports 内容可用性仍可能未知。 | 清点后再决定 restore / NAS / provider CR。 |

### 不授权项

如果你回复 approve，表示接受本文件 5 项推荐方案；不表示授权以下操作：

- 不授权 CP2 单独触发 `reports/`、`data/` 访问；必须 CP2 / CP3 / CP5 全部 approve。
- 不授权读取文件内容、打印样本、解析 parquet/csv/json、计算内容 hash、复制、恢复、比对内容或删除。
- 不授权读取、打印、记录、复制或迁移 `.env`、token、密码、cookie、session、private key、账户或交易事实。
- 不授权 provider fetch、lake write、catalog publish、MARKET_DATA_LAKE_ROOT replacement、NAS promote 或 provider rebuild。
- 不授权远端 Git 写入、QMT/MiniQMT runtime、CR046 recovery、旧根删除/移动、optional groups、full tests 或 destructive cleanup。

### 自动终验授权

- 是否启用：false
- 授权范围：不适用
- 自动通过条件：N/A
- 授权原文：N/A

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| CP2 自动预检 PASS | approved | `process/checks/CP2-CR079-DATA-REPORTS-ACCESS-BASELINE.md` | 用户回复“好的，先完成CR079的迁移”，按 approve 处理。 |
| CR079 formal CR 存在 | approved | `process/changes/CR-079-REAL-DATA-REPORTS-ACCESS-GATE-2026-06-17.md` | 用户回复“好的，先完成CR079的迁移”，按 approve 处理。 |
| context ready | approved | `process/context/CP2-CR079-DATA-REPORTS-ACCESS-CONTEXT.yaml` | 用户回复“好的，先完成CR079的迁移”，按 approve 处理。 |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | 是否正式启动 CR079 | approved | DQ-CP2-CR079-01 | 用户接受推荐方案。 |
| 2 | 范围是否限定为 metadata-only | approved | DQ-CP2-CR079-02 | 用户接受推荐方案；不扩大为内容读取 / 恢复 / 删除。 |
| 3 | 凭据和交易事实是否继续禁止 | approved | DQ-CP2-CR079-03 | 用户接受推荐方案。 |
| 4 | CP2 是否不单独授权执行 | approved | DQ-CP2-CR079-04 | 用户接受推荐方案；执行仍需 CP3/CP5 同步 approved。 |
| 5 | metadata-only 剩余风险是否接受 | approved | DQ-CP2-CR079-05 | 用户接受推荐方案。 |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| 用户明确 approve / 修改 / reject | approved | 当前对话：用户回复“好的，先完成CR079的迁移” | 解释为 CR079 CP2/CP3/CP5 approve；“迁移”不扩大为复制 / 恢复 / 删除授权。 |
| CP2 approve 后仍不执行真实访问 | approved | 不授权项 | CP2 单独不触发执行；需三门同步 approved 后进入 metadata-only 清点。 |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| CP2 auto precheck | `process/checks/CP2-CR079-DATA-REPORTS-ACCESS-BASELINE.md` | approved | PASS。 |
| CP2 context | `process/context/CP2-CR079-DATA-REPORTS-ACCESS-CONTEXT.yaml` | approved | ready-for-human-gate。 |
| CP2 manual checkpoint | `process/checkpoints/CP2-CR079-DATA-REPORTS-ACCESS-BASELINE.md` | approved | 用户接受 DQ-CP2-CR079-01..05。 |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-06-17T06:12:28+08:00
- 修改意见：无；用户回复“好的，先完成CR079的迁移”，按 approve 处理。
- 风险接受项：接受 metadata-only 只读清点可能仍不能证明内容 readiness；确认 CR079 不授权内容读取、复制、恢复、删除、`.env` / 凭据、provider/lake/catalog、远端 Git、runtime、CR046、旧根退役或 cleanup。
