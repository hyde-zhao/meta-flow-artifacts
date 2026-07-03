---
status: draft
version: "1.0"
feature_id: "FEAT-PG-004"
feature_name: "Capability / Feature Registry"
source_design: "process/docs/features/capability-feature-registry/DESIGN.md"
confirmed_by: ""
confirmed_at: ""
---

# Feature Test Plan: Capability / Feature Registry

## 测试范围

| Scope ID | 覆盖内容 | 来源 Story / Scenario | 测试层级 | 自动化状态 |
|---|---|---|---|---|
| CFR-UT-01 | Feature registry schema：必填字段、状态枚举、owner、source refs。 | CR-D-S01 / UC-PG-004 | unit | planned |
| CFR-UT-02 | Capability registry schema：domain、feature refs、aliases、deprecated_by。 | CR-D-S01 / UC-PG-004 | unit | planned |
| CFR-UT-03 | Resolver resolved / unresolved / deprecated / conflict 分支。 | CR-D-S01 / SIM-PG-003 | unit | planned |
| CFR-UT-04 | ID 唯一性和 alias collision。 | CR-D-S01 / R-PG-003 | unit | planned |
| CFR-INT-01 | registry loader + checker 在完整 registry 上 PASS。 | CR-D-S01 | integration | planned |
| CFR-INT-02 | impact normalization consumer 通过 resolver 获取 canonical refs。 | CR-D-S01 / FEAT-PG-005 | integration / contract | planned |
| CFR-INT-03 | roadmap/stale consumer 遇到 unresolved refs 时 blocked。 | CR-D-S01 / FEAT-PG-006 / FEAT-PG-008 | integration / contract | planned |
| CFR-CON-01 | `feature_refs` / `capability_refs` 字段不得接受自由字符串落盘。 | ADR-PG-004 | contract / security | planned |
| CFR-MAN-01 | 人工审查初始 registry ID、owner、source refs 是否清晰。 | CR-D-S01 | manual | manual |

## 风险驱动测试

| Risk ID | 风险 | 测试方式 | 证据 | 未覆盖原因 |
|---|---|---|---|---|
| CFR-R-01 | registry 初始化不完整导致 blocked findings。 | unresolved fixture 期望 resolver blocked，candidate report 不写 canonical。 | resolver negative tests。 | 无。 |
| CFR-R-02 | ID 命名不可维护。 | schema 要求 owner/source refs；人工审查命名。 | unit + manual review。 | 命名质量需人工判断。 |
| CFR-R-03 | 下游绕过 resolver。 | contract test 校验 consumer 必须调用 resolver result。 | consumer contract tests。 | 依赖下游 Story 实现。 |
| CFR-R-04 | registry 泄露敏感信息。 | credential-like key/value negative fixtures。 | security unit tests。 | 无。 |

## 单元测试

| Case ID | 输入 | 期望行为 | 验证入口 |
|---|---|---|---|
| CFR-UNIT-01 | 最小合法 feature registry。 | schema PASS。 | registry schema tests。 |
| CFR-UNIT-02 | 最小合法 capability registry。 | schema PASS。 | registry schema tests。 |
| CFR-UNIT-03 | resolver 输入未注册 ref。 | 返回 `E_REF_UNRESOLVED` blocked finding。 | resolver tests。 |
| CFR-UNIT-04 | 两个 ID 共享同一 alias。 | checker ERROR alias collision。 | alias tests。 |
| CFR-UNIT-05 | 输入 deprecated ref。 | audit WARN；enforce ERROR 或要求使用 `deprecated_by`。 | mode tests。 |

## 集成测试

| Case ID | 场景 | 期望行为 | 验证入口 |
|---|---|---|---|
| CFR-INT-01 | registry loader 读取 feature/capability registry。 | 返回 immutable registry snapshot。 | loader integration test。 |
| CFR-INT-02 | impact migration 调 resolver。 | resolved refs 写入 normalized candidate，unresolved 写 blocked finding。 | synthetic impact consumer test。 |
| CFR-INT-03 | stale-check 调 resolver。 | stale finding 使用 canonical ID。 | synthetic stale consumer test。 |
| CFR-INT-04 | registry invalid 时下游 checker 执行。 | 下游 blocked，不继续猜测 refs。 | integration negative test。 |

## 契约测试

| Contract ID | 调用方 | 被调用方 | 契约 | 破坏时预期 |
|---|---|---|---|---|
| CFR-CON-01 | FEAT-PG-005 impact migration | registry resolver | consumer 必须传 refs/hints，消费 resolver result，不直接写自由字符串。 | contract FAIL。 |
| CFR-CON-02 | FEAT-PG-006 roadmap refresh | registry resolver | roadmap result 中 refs 必须 canonical。 | unresolved blocked。 |
| CFR-CON-03 | FEAT-PG-009 migration readiness | registry candidate flow | 外部样本只生成 candidate report，不写 canonical registry。 | security FAIL。 |

## 权限 / 安全 / 失败路径

| Case ID | 触发条件 | 期望行为 | 测试入口 |
|---|---|---|---|
| CFR-SEC-01 | resolver 遇到自由字符串或未知 capability。 | blocked finding，不自动创建 ID。 | resolver negative test。 |
| CFR-SEC-02 | registry 中出现 credential-like key/value。 | checker ERROR。 | security fixture。 |
| CFR-SEC-03 | migration hint 与 registry canonical ID 冲突。 | registry wins，输出 conflict finding。 | source priority test。 |
| CFR-SEC-04 | 外部样本映射试图直接写 registry。 | 拒绝写入 canonical registry，只输出 candidate report。 | migration contract test。 |

## 手工验收

| Check ID | 操作 | 期望结果 | 责任方 |
|---|---|---|---|
| CFR-MAN-01 | 审查初始 registry ID 命名。 | ID 稳定、无缩写歧义、owner/source refs 清晰。 | human / qa |
| CFR-MAN-02 | 审查 aliases 和 deprecated 策略。 | alias 不冲突，deprecated 有替代 ID。 | human / qa |
| CFR-MAN-03 | 审查 unresolved candidate report。 | 未确认能力未进入 canonical registry。 | human / qa |

## 未自动化项

| Item | 原因 | 替代证据 |
|---|---|---|
| capability 领域归属是否准确 | 需要领域判断。 | 人工 registry review。 |
| ID 命名是否符合长期阅读习惯 | 语义质量不能完全自动化。 | CP5 人工确认或 reviewer finding。 |
