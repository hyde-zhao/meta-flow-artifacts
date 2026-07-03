---
status: draft
version: "1.0"
feature_id: "FEAT-PG-008"
feature_name: "Project Stale Check"
source_design: "process/docs/features/project-stale-check/DESIGN.md"
confirmed_by: ""
confirmed_at: ""
---

# Feature Test Plan: Project Stale Check

## 测试范围

| Scope ID | 覆盖内容 | 来源 Story / Scenario | 测试层级 | 自动化状态 |
|---|---|---|---|---|
| UNIT-ST-01 | stale finding schema required fields。 | CR-G-S02 / OBJ-PG-015 | unit / contract | planned |
| UNIT-ST-02 | project phase vs roadmap semantic mismatch。 | UC-PG-006 / SIM-PG-002 | unit | planned |
| UNIT-ST-03 | roadmap refresh doc impact -> stale finding。 | FEAT-PG-006 | unit / integration | planned |
| UNIT-ST-04 | cr-tracking 结构问题不由 stale-check 处理。 | RULE-PG-011 | unit | planned |
| INTEG-ST-01 | project refs + roadmap refs -> stale-check result。 | CR-G-S02 | integration | planned |
| INTEG-ST-02 | no-auto-fix guard 阻断正式文档写入。 | FD-PG-007 | security / integration | planned |
| INTEG-ST-03 | actionable finding -> FU-RF/SP-RF/RA-RF candidate。 | FEAT-PG-007 | integration / contract | planned |
| MAN-ST-01 | 命令名、输出摘要和 severity 可人工理解。 | Q-PG-003 | manual | planned |

## 风险驱动测试

| Risk ID | 风险 | 测试方式 | 证据 | 未覆盖原因 |
|---|---|---|---|---|
| R-ST-01 | stale-check 自动修改正式文档。 | 构造 write action，期望 FAIL。 | security fixture | 无。 |
| R-ST-02 | 与 cr-tracking 结构检查重叠。 | 构造 CR 编号/状态结构错误，期望 stale-check 不处理。 | boundary fixture | 无。 |
| R-ST-03 | 语义规则误报。 | fixture + 人工抽样审查 severity。 | manual checklist | 语义判断需人工参与。 |
| R-ST-04 | 缺关键 refs 时仍猜测 PASS。 | 缺 project/roadmap refs，期望 blocked finding。 | failure fixture | 无。 |

## 权限 / 安全 / 失败路径

| Case ID | 触发条件 | 期望行为 | 测试入口 |
|---|---|---|---|
| SEC-ST-01 | stale-check 尝试写 HLD / TEST-STRATEGY / release docs。 | FAIL，输出 no-auto-fix finding。 | security fixture |
| SEC-ST-02 | stale-check 需要 release repo 正式修复。 | 只生成 stale finding / FU-RF，不写 release repo。 | integration fixture |
| SEC-ST-03 | project_ref 或 roadmap_ref 缺失。 | result 含 blocked finding，不给 PASS。 | failure fixture |
| SEC-ST-04 | 规则要求读取 `process/quant-lab/**`。 | 本设计范围不读取；后续迁移 Story 单独授权。 | manual review |

## 单元测试建议

| Test ID | 对象 | 输入 | 期望 |
|---|---|---|---|
| UT-ST-001 | finding schema | 完整 finding | PASS。 |
| UT-ST-002 | finding schema | 缺 `expected_semantic` | FAIL。 |
| UT-ST-003 | phase rule | project phase=paper-readiness，test strategy=backtest-only | stale finding。 |
| UT-ST-004 | boundary rule | CR status missing | not handled by stale-check。 |
| UT-ST-005 | severity resolver | release docs stale | severity 按规则输出 warning/blocking。 |

## 集成测试建议

| Test ID | 流程 | 输入 | 期望 |
|---|---|---|---|
| IT-ST-001 | clean project | project/roadmap/docs refs 一致 | PASS / no findings。 |
| IT-ST-002 | doc stale | roadmap 与 docs 语义不一致 | result 含 stale finding。 |
| IT-ST-003 | refresh impacts | ROADMAP-REFRESH result 含 doc impacts | stale finding + FU-RF candidate。 |
| IT-ST-004 | missing refs | 缺 HLD/test refs | warning 或 blocked finding，按 rule severity。 |
| IT-ST-005 | no auto fix | finding 指向 formal docs | 不写文件，只输出 recommended_route。 |

## 契约测试建议

| Contract ID | 消费方 | 契约 | 期望 |
|---|---|---|---|
| CT-ST-001 | FEAT-PG-007 | actionable finding 可转 follow-up candidate。 | candidate 含 source finding ref。 |
| CT-ST-002 | FEAT-PG-006 | refresh result 可作为 stale-check 输入。 | doc impacts 被识别。 |
| CT-ST-003 | meta-qa | stale-check result 可被验证报告引用。 | result/summary refs 稳定。 |
| CT-ST-004 | cr-tracking | stale-check 不接管 CR tracking 结构错误。 | boundary PASS。 |

## 手工验收

| Check ID | 操作 | 期望结果 | 责任方 |
|---|---|---|---|
| MAN-ST-01 | 审查 CLI 命名和帮助文案。 | project stale-check 语义清楚，不像 cr-tracking。 | human / se |
| MAN-ST-02 | 审查 3 条 stale finding 示例。 | expected/observed/source/recommended route 可理解。 | human / qa |
| MAN-ST-03 | 审查 no-auto-fix guard。 | 没有自动修改正式文档或发布库。 | human / security |
| MAN-ST-04 | 审查误报处理。 | 可 waive 或降级规则，保留追踪。 | human / qa |
