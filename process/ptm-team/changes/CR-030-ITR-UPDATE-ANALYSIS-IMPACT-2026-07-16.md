---
change_id: "CR-030-REV-03"
parent_cr: "CR-030"
type: "requirements-and-hld-review-remediation"
status: "active"
rollback_to: "requirement-clarification"
required_gate: "CP2"
created_at: "2026-07-16T09:00:00+08:00"
trigger: "用户评审指出 S2 更新分析、数据清洗、多维分析和趋势对比未被完整建模。"
---

# CR-030-REV-03：ITR 新增/更新分析基线整改

## 变更结论

接受评审 G1–G3：S2 是独立用户旅程；清洗是摄取与分析之间的显式数据质量层；根因、产品质量、流出、漏测、改进和环比/同比必须成为可追溯的分析维度。该修订不扩大已批准的 ITR GET、非 Git 受限存储和人工确认权限，只补足数据模型、分析与报告语义。

## 五维度影响

| 维度 | 影响 | 处理 |
|---|---|---|
| 需求/场景 | 是 | 新增 UC-RA-08、REQ-RA-019..022、SCN-RA-11..13，明确 S1/S2。 |
| 计划 | 是 | Story Map 增加更新分析 Story；MVP/矩阵纳入增量合并与差异报告。 |
| 设计 | 是 | HLD 采用 raw → cleaned → versioned SQLite → analysis/report 的四/五阶段模型。 |
| 安全 | 否（边界不变） | 不新增来源、认证或写入；清洗保留质量/冲突标记，不静默覆盖。 |
| 交付/验证 | 是 | 新增变更历史、去重/合并、时间窗口和措施刷新 fixture。 |

## 文档处理决策

| 对象 | 处理 | 旧基线保留 |
|---|---|---|
| `USE-CASES.md`、`REQUIREMENTS.md`、`SCENARIOS.yaml`、`TEST-MATRIX.md`、`STORY-MAP.md`、`MVP-SCOPE.md` | 原文档更新 | 保留 v1.2 与既有 ID；新增连续 ID 并在修订记录说明。 |
| `HLD.md` | 原文档更新 | v1.0/REV-02 仅作历史；v1.2 草案明确替换其“无外部读取/三段链路”假设。 |

## 门禁

- `product_baseline_refresh_required=true`；回退 `requirement-clarification`。
- CP2 已批准的外部读取授权保持有效；但需求/场景基线须重新通过 CP1/CP2，CP3 不得使用旧 HLD。
