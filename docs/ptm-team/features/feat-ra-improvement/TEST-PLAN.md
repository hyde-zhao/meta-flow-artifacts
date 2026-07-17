---
status: draft
version: "1.0"
source_cr: "CR-030"
feature_id: "FEAT-RA-IMPROVEMENT"
created_by: "meta-se"
created_at: "2026-07-16"
---

# FEAT-RA-IMPROVEMENT — 测试计划

## 1. 测试场景

| ID | 场景 | Story | 类型 | 预期 |
|---|---|---|---|---|
| T-IMP-01 | 基于已确认 RA 生成 CA/PA 草案 | ST-RA-03 | positive | 每项含依据/类型/目标/Owner/验收 |
| T-IMP-02 | 未确认 RA 不能生成 CA/PA | ST-RA-03 | negative | 拒绝，提示需先确认分析 |
| T-IMP-03 | reviewer 批准 CA/PA | ST-RA-03 | positive | proposal.status='approved' |
| T-IMP-04 | reviewer 拒绝 CA/PA | ST-RA-03 | positive | proposal.status='rejected'，保留记录 |
| T-IMP-05 | 批准后生成 Approved Improvement Input | ST-RA-03 | positive | 输入含 source_ra/proposal_id/approval_ref |
| T-IMP-06 | 未批准不生成 Approved Input | ST-RA-03 | negative | 0 个未批准输入 |
| T-IMP-07 | Approved Input 不可变性 | ST-RA-03 | boundary | 生成后字段不可修改 |
| T-IMP-08 | 消费者不可用标为 blocked | ST-RA-03 | boundary | consumer_status='blocked' |
| T-IMP-09 | CA/PA Proposal 字段完整性 | ST-RA-03 | positive | 必填字段齐全 |

## 2. Fixture 设计

| Fixture | 用途 |
|---|---|
| `fixtures/confirmed_ra_report.json` | T-IMP-01, 03, 04, 05 |
| `fixtures/draft_ra_report.json` | T-IMP-02 |
| `fixtures/approved_capa.json` | T-IMP-05, 07 |
