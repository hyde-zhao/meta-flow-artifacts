---
document_id: "FEEDBACK-CR166"
cr_id: "CR-166"
status: "OPEN_FOR_FEEDBACK"
created_at: "2026-07-13T14:38:00+08:00"
---

# CR-166 反馈与后续分流

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-13 | host-orchestrator inline | 定义缺陷、设计澄清、授权扩展和后续能力的分流规则。 |

## 反馈分流

| 反馈类型 | 处理路径 | 不允许的默认动作 |
|---|---|---|
| canonical hash、fold decision、aggregate 或 projection 缺陷 | ISSUE → regression subset → CR-166 回修或修复 CR | 不删除失败测试，不降低 fail-closed |
| C2 schema 或 consumer contract 变化 | 设计澄清 → CP3 | 不直接修改公共 contract |
| 真实数据、provider、lake、NAS、credential 或 runtime 需求 | 新独立 CR + 明确授权门禁 | 不在 CR-166 内隐式扩大权限 |
| C3 economic cost | 保留 `FU-CR161-004` 后续候选 | reserved slot 不得被当作已实现 |
| C4 capacity/liquidity | 保留 `FU-CR161-005` 后续候选 | reserved slot 不得被当作已实现 |
| event-specific walk-forward | 独立适用性/HLD 决策 | 不把 N/A envelope 解释为 producer coverage |
| Stage 3 启动 | 独立 CR、真实数据连接与人工门禁 | CR-166 CP8 不授权启动 |

## 提交反馈时应包含

1. 受影响的 component schema/version、fold ID 或 consumer 名称。
2. 最小 synthetic fixture 或可脱敏的逻辑输入；不得提交凭据或生产数据样本。
3. 期望状态与实际状态，以及 canonical identity/reason code。
4. 是否涉及真实数据、运行时或外部写入；若涉及，明确标记 `authorization-required`。
