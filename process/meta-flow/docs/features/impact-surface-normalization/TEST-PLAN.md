---
status: draft
version: "1.0"
feature_id: "FEAT-PG-005"
feature_name: "Impact Surface Normalization"
source_design: "process/docs/features/impact-surface-normalization/DESIGN.md"
confirmed_by: ""
confirmed_at: ""
---

# Feature Test Plan: Impact Surface Normalization

## 测试范围

| Scope ID | 覆盖内容 | 来源 Story / Scenario | 测试层级 | 自动化状态 |
|---|---|---|---|---|
| ISN-UT-01 | normalized impact schema：四字段、必填/可选、类型校验。 | CR-E-S01 / UC-PG-005 | unit | planned |
| ISN-UT-02 | impact surface enum：合法枚举和 unknown enum。 | CR-E-S01 | unit | planned |
| ISN-UT-03 | `affected_paths` 路径格式和非授权语义。 | CR-E-S01 | unit / security | planned |
| ISN-CON-01 | `feature_refs` / `capability_refs` 必须调用 registry resolver。 | CR-E-S01 / ADR-PG-004 | contract | planned |
| ISN-INT-01 | 新 CR audit 模式 unknown surface 输出 WARN。 | CR-E-S01 | integration | planned |
| ISN-INT-02 | 新 CR enforce 模式 unknown surface 输出 ERROR。 | CR-E-S01 | integration | planned |
| ISN-INT-03 | legacy `impact_surface` 生成 migration report，不修改源 CR。 | CR-E-S01 / R-PG-005 | integration / security | planned |
| ISN-MAN-01 | 人工审查 unclassified legacy impact 和新增 enum 请求。 | CR-E-S01 | manual | manual |

## 风险驱动测试

| Risk ID | 风险 | 测试方式 | 证据 | 未覆盖原因 |
|---|---|---|---|---|
| ISN-R-01 | 历史漂移太多导致普通 check 刷屏。 | legacy fixtures 只生成 migration report，不让普通 check FAIL 全历史。 | migration report tests。 | 无。 |
| ISN-R-02 | enum 过窄导致合法影响面无法表达。 | unknown enum fixture + manual expansion checklist。 | unit + manual review。 | 是否新增 enum 需要架构判断。 |
| ISN-R-03 | registry resolver 不稳定。 | resolver unavailable/invalid fixture，期望 refs validation blocked。 | contract negative tests。 | 依赖 FEAT-PG-004。 |
| ISN-R-04 | `affected_paths` 被误作授权。 | 包含 approval/write authorization 语义的 fixture，期望 security finding。 | security tests。 | 无。 |

## 单元测试

| Case ID | 输入 | 期望行为 | 验证入口 |
|---|---|---|---|
| ISN-UNIT-01 | 含四字段的合法 normalized impact block。 | schema PASS。 | impact schema tests。 |
| ISN-UNIT-02 | `impact_surfaces` 含 unknown enum。 | audit WARN / enforce ERROR。 | mode-aware unit tests。 |
| ISN-UNIT-03 | `affected_paths` 含空字符串、绝对危险模式或授权语义。 | checker ERROR 或 security finding。 | path/security fixture tests。 |
| ISN-UNIT-04 | `feature_refs` 缺失 registry canonical ID。 | resolver unresolved blocked。 | resolver contract test。 |
| ISN-UNIT-05 | legacy value 可分类为 surface + path。 | migration classifier 给出 suggested fields。 | classifier tests。 |

## 集成测试

| Case ID | 场景 | 期望行为 | 验证入口 |
|---|---|---|---|
| ISN-INT-01 | 新 CR audit check。 | unknown surface WARN，不阻断 audit。 | CR checker integration test。 |
| ISN-INT-02 | 新 CR enforce check。 | unknown surface ERROR。 | CR checker integration test。 |
| ISN-INT-03 | 历史 CR migration scan。 | 生成 report，源 CR hash 不变。 | migration report integration test。 |
| ISN-INT-04 | registry invalid 时执行 impact check。 | refs validation blocked，surface/path 仍可报告。 | integration negative test。 |

## 契约测试

| Contract ID | 调用方 | 被调用方 | 契约 | 破坏时预期 |
|---|---|---|---|---|
| ISN-CON-01 | impact checker | FEAT-PG-004 resolver | refs 必须经 resolver，输出 canonical refs 或 blocked finding。 | contract FAIL / blocked。 |
| ISN-CON-02 | change-impact-analysis | impact schema | 新 CR 使用 normalized fields，不再写混合自由文本。 | checker WARN/ERROR。 |
| ISN-CON-03 | checkpoint-manager | impact checker | CP 只消费摘要和 finding，不复制完整历史 CR。 | CP precheck FAIL。 |

## 权限 / 安全 / 失败路径

| Case ID | 触发条件 | 期望行为 | 测试入口 |
|---|---|---|---|
| ISN-SEC-01 | enforce 模式 unknown surface。 | ERROR，提示改用已知 enum 或发 CR 扩展。 | mode negative test。 |
| ISN-SEC-02 | unresolved `feature_refs` / `capability_refs`。 | blocked finding，不写 normalized refs。 | resolver contract negative test。 |
| ISN-SEC-03 | `affected_paths` 包含授权、approval 或 runtime write 语义。 | security finding，要求转 runtime_authorization 决策。 | path security fixture。 |
| ISN-SEC-04 | migration scanner 尝试改写历史 CR。 | test 断言源文件 hash 不变，只写 report。 | migration no-write test。 |

## 手工验收

| Check ID | 操作 | 期望结果 | 责任方 |
|---|---|---|---|
| ISN-MAN-01 | 审查 migration report 中 unclassified legacy values。 | 每项有建议处理或明确 blocked reason。 | human / qa |
| ISN-MAN-02 | 审查 impact surface enum。 | 14 类治理面足以覆盖 CR-037 预期影响面，无明显重叠。 | human / qa |
| ISN-MAN-03 | 抽查 normalized CR 示例。 | surfaces、paths、feature refs、capability refs 语义分离。 | human / qa |

## 未自动化项

| Item | 原因 | 替代证据 |
|---|---|---|
| 是否新增 impact surface enum | 需要架构和治理语义判断。 | 人工确认 / CR 决策项。 |
| legacy value 的最终归类 | 部分历史文本语义可能不完整。 | migration report + 人工回修记录。 |
