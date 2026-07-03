---
status: draft
version: "1.0"
feature_id: "FEAT-PG-003"
feature_name: "Project State Governance"
source_design: "process/docs/features/project-state-governance/DESIGN.md"
confirmed_by: ""
confirmed_at: ""
---

# Feature Test Plan: Project State Governance

## 测试范围

| Scope ID | 覆盖内容 | 来源 Story / Scenario | 测试层级 | 自动化状态 |
|---|---|---|---|---|
| PSG-UT-01 | `PROJECT.current.json` schema：必填字段、unknown fields、ref fields、更新时间格式。 | CR-C-S01 / UC-PG-002 | unit | planned |
| PSG-UT-02 | `PROJECT.current.json` 16KB 预算和 forbidden fields。 | CR-C-S01 / R-PG-002 | unit / security | planned |
| PSG-UT-03 | `PROJECT-SCALE.yaml` 枚举、gate bias 和 not-authorized 声明。 | CR-C-S02 / UC-PG-003 | unit / contract | planned |
| PSG-UT-04 | `ROADMAP.yaml` / `MILESTONES.yaml` ID 唯一性、状态枚举和 refs 完整性。 | CR-C-S02 | unit | planned |
| PSG-INT-01 | scaffold dry-run 只输出 plan，不创建文件。 | CR-C-S01 | integration | planned |
| PSG-INT-02 | scaffold apply 创建 project objects 后，checker PASS，current-state writer 写入 `project_state_ref`。 | CR-C-S01 | integration | planned |
| PSG-INT-03 | 已存在文件冲突时 apply 中止，不覆盖文件。 | CR-C-S01 | integration / security | planned |
| PSG-CON-01 | 下游 reader 能从 `PROJECT.current.json` 解析 roadmap/milestone refs。 | CR-C-S02 / FEAT-PG-006 | contract | planned |
| PSG-CON-02 | `STATE.current.json` 只能保存 `project_state_ref`，不得内嵌 project fields。 | CR-C-S01 / FEAT-PG-001 | contract / security | planned |
| PSG-MAN-01 | 人工审查 project object baseline 可读、路径清晰、未混入发布库写授权。 | CR-C-S01 / CR-C-S02 | manual | manual |

## 风险驱动测试

| Risk ID | 风险 | 测试方式 | 证据 | 未覆盖原因 |
|---|---|---|---|---|
| PSG-R-01 | `PROJECT.current.json` 成为新巨型状态。 | 构造超预算和内嵌长文本 fixture，期望 checker ERROR。 | unit test result / checker finding。 | 无。 |
| PSG-R-02 | scale bias 被误用为 gate 授权。 | 构造包含 `auto_approve` / `skip_gate` 等语义的 fixture，期望 FAIL。 | negative fixture。 | 无。 |
| PSG-R-03 | scaffold 覆盖已有文件。 | tmp workspace 中预置冲突文件，执行 apply，期望 fail 且内容 hash 不变。 | integration test。 | 无。 |
| PSG-R-04 | current state 内嵌 project fields。 | 构造 current state patch，期望 FEAT-PG-001 checker 拒绝未知重型字段。 | contract test。 | 依赖 FEAT-PG-001 writer/checker 完成。 |

## 单元测试

| Case ID | 输入 | 期望行为 | 验证入口 |
|---|---|---|---|
| PSG-UNIT-01 | 最小合法 `PROJECT.current.json`。 | schema PASS，预算 PASS。 | `pytest` project-state schema tests。 |
| PSG-UNIT-02 | project current 含未知顶层字段。 | checker ERROR，指出字段路径。 | `pytest` negative fixtures。 |
| PSG-UNIT-03 | project current 超过 16KB。 | checker ERROR `E_PROJECT_STATE_OVER_BUDGET`。 | `pytest` budget tests。 |
| PSG-UNIT-04 | `PROJECT-SCALE.yaml` 含合法 gate bias。 | checker PASS。 | `pytest` scale schema tests。 |
| PSG-UNIT-05 | milestone status 非枚举值。 | checker ERROR。 | `pytest` roadmap fixture tests。 |

## 集成测试

| Case ID | 场景 | 期望行为 | 验证入口 |
|---|---|---|---|
| PSG-INT-01 | 空 workspace 执行 scaffold dry-run。 | 输出待创建文件列表，不写文件。 | tmp workspace CLI test。 |
| PSG-INT-02 | 空 workspace 执行 scaffold apply。 | 创建 project objects，project-state check PASS。 | tmp workspace CLI test。 |
| PSG-INT-03 | scaffold apply 后写入 current `project_state_ref`。 | current state 只包含 ref，不包含 project fields。 | integration + contract test。 |
| PSG-INT-04 | 下游 reader 读取 project snapshot。 | 返回 typed snapshot；断 ref 时返回 blocked finding。 | reader integration test。 |

## 契约测试

| Contract ID | 调用方 | 被调用方 | 契约 | 破坏时预期 |
|---|---|---|---|---|
| PSG-CON-01 | FEAT-PG-006 / FEAT-PG-008 | project object reader | reader 输入 `project_state_ref`，输出 typed project snapshot。 | `E_PROJECT_OBJECT_INVALID`，下游 blocked。 |
| PSG-CON-02 | project scaffold | FEAT-PG-001 writer | 只提交 `project_state_ref` patch、actor、reason。 | writer 拒绝未知或重型字段。 |
| PSG-CON-03 | checkpoint/gate 消费方 | `PROJECT-SCALE.yaml` | gate bias 是建议，不是授权。 | checker FAIL 或人工审查 FAIL。 |

## 权限 / 安全 / 失败路径

| Case ID | 触发条件 | 期望行为 | 测试入口 |
|---|---|---|---|
| PSG-SEC-01 | scaffold apply 命中已有不同内容文件。 | fail，不覆盖，输出冲突 finding。 | integration negative test。 |
| PSG-SEC-02 | `PROJECT.current.json` 内嵌完整 HLD、transcript、credential-like key 或超预算文本。 | checker ERROR。 | unit security fixtures。 |
| PSG-SEC-03 | `PROJECT-SCALE.yaml` 声明自动批准、跳过 CP 或直接写 gate policy。 | checker ERROR。 | unit security fixtures。 |
| PSG-SEC-04 | 未经 writer 直接修改 `STATE.current.json` 加入 project fields。 | guardrail/checker FAIL。 | contract test with FEAT-PG-001。 |

## 手工验收

| Check ID | 操作 | 期望结果 | 责任方 |
|---|---|---|---|
| PSG-MAN-01 | 审查 scaffold 后的 `process/project/` 文件。 | 文件职责清晰，project current 只含摘要和 refs。 | human / qa |
| PSG-MAN-02 | 审查 `PROJECT-SCALE.yaml`。 | 能表达项目规模与 gate bias，但明确不授权跳过人工门禁。 | human / qa |
| PSG-MAN-03 | 从 `STATE.current.json` 的 `project_state_ref` 追踪到 roadmap/milestone。 | 路径可恢复，断链时有明确错误。 | human / qa |

## 未自动化项

| Item | 原因 | 替代证据 |
|---|---|---|
| project scale 语义是否符合用户治理偏好 | 需要人工判断默认 bias 是否过强。 | CP5 Decision Brief / 人工验收记录。 |
| roadmap/milestone 初始内容是否覆盖真实项目管理习惯 | 内容质量非纯 schema 问题。 | 人工审查 + 后续 roadmap refresh evidence。 |
