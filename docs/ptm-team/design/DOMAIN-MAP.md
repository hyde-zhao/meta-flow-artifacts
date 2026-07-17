---
status: draft
version: "1.0"
source_cr: "CR-030"
---

# ptm-tse 逆向分析领域地图

## 术语表

| Term | 定义 | 来源 | 备注 |
|---|---|---|---|
| RA Report | 对一个已恢复现网问题的结构化逆向分析记录 | UC-RA-01..05 | 根因结论、CA/PA 与关闭状态可审计 |
| Evidence Line | 流量、策略、状态、资源、变更中的一条证据维度 | REQ-RA-002 | 至少三条有效才可确认根因 |
| Introduction Point | 问题最早被引入的阶段与证据锚点 | REQ-RA-005 | 需求/设计/实现/变更/运营 |
| Escape Point | 允许问题流入现网的失效控制点 | REQ-RA-006 | 必须识别最近可拦截点 |
| CA / PA | 纠正措施 / 预防措施 | REQ-RA-007 | 皆为草案直至人工批准 |
| Approved Improvement Input | 经过批准、可供下游只读消费的改进记录 | REQ-RA-008 | 不等同下游任务或外部工单 |

## 领域对象

| Object ID | 对象 | Owner Feature | 关键字段 / 属性 | 状态 | 规则来源 |
|---|---|---|---|---|---|
| OBJ-RA-01 | RA Report | FEAT-RA-ANALYSIS | ra_id、event_ref、severity、evidence_refs、facts、hypotheses、review_status | draft → analysis-confirmed → improvement-approved → tracking → closed | REQ-001–006,010 |
| OBJ-RA-02 | Evidence Assessment | FEAT-RA-ANALYSIS | line、validity、reference、gap、classification | pending / valid / insufficient | REQ-002,003 |
| OBJ-RA-03 | CA/PA Proposal | FEAT-RA-IMPROVEMENT | id、kind、owner、due_date、validation、side_effect、approval | draft / approved / rejected | REQ-007,010 |
| OBJ-RA-04 | Approved Improvement Input | FEAT-RA-IMPROVEMENT | source_ra、proposal_id、target_agent、acceptance、constraints、approval_ref | pending-consumer / consumed / blocked | REQ-008,011 |
| OBJ-RA-05 | Action Item | FEAT-RA-TRACKING | id、owner、due_date、status、blockers | not-started / in-progress / done / overdue | REQ-009 |
| OBJ-RA-06 | Effectiveness Check | FEAT-RA-TRACKING | method、window、result、recurrence_measure | planned / passed / failed / inconclusive | REQ-009,013 |

## 状态机

| State Machine ID | 对象 | 状态 | 合法转换 | 非法转换处理 |
|---|---|---|---|---|
| SM-RA-01 | RA Report | draft → evidence-sufficient → analysis-confirmed → improvement-approved → tracking → closed | 每次转换均需前置状态与人工确认引用；closed 另需所有关闭条件 | 拒绝转换并记录缺失条件 |
| SM-RA-02 | CA/PA Proposal | draft → approved/rejected → pending-consumer → consumed/blocked | approved 仅由人工 reviewer 写入 | 未批准禁止分发 |
| SM-RA-03 | Action Item | not-started → in-progress → done/overdue | done 需验证记录；RA closed 需全部适用项 done | 保持开放并列出阻塞 |

## 业务规则

| Rule ID | 规则 | Owner | 影响场景 | 验证入口 |
|---|---|---|---|---|
| RULE-RA-01 | P1 已恢复且证据保全完成才可自动建议创建；P2 要显式选择 | ANALYSIS | SCN-RA-01 | eligibility fixture |
| RULE-RA-02 | 少于三条有效证据线时，禁止根因状态为 confirmed | ANALYSIS | SCN-RA-02 | threshold fixture |
| RULE-RA-03 | AI 草案与人工确认必须可区分 | ANALYSIS | SCN-RA-03 | report schema review |
| RULE-RA-04 | 未批准 CA/PA 不得生成消费者输入 | IMPROVEMENT | SCN-RA-04 | contract fixture |
| RULE-RA-05 | 未完成关闭条件不得关闭 RA | TRACKING | SCN-RA-05 | closure fixture |
| RULE-RA-06 | 涉及凭据、外部读取或生产写入的请求必须拒绝并转独立 CR | ANALYSIS | SCN-RA-06 | forbidden-path test |
