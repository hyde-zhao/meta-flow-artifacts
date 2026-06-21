---
plan_id: "MF-018-RUNTIME-AUTHORIZATION-NEXT-PLAN"
created_at: "2026-06-21T16:57:51+08:00"
owner: "host-orchestrator"
status: "staged-component-validation-planned"
source_handoff: "process/handoffs/MF-018-RUNTIME-AUTHORIZATION-HANDOFF.md"
component_validation_plan: "process/checks/MF-018-RUNTIME-COMPONENT-VALIDATION-PLAN.md"
---

# MF-018 Runtime 下一步执行计划

## 当前目标

最终目标明确：完成真实 runtime。执行策略调整为分阶段推进，在真实 runtime 前完成 trace、脱敏、collector、本地后端、SaaS contract、credentialed sandbox 等组件验证。

## 推荐结论

推荐先走 `option_a_minimal_readonly`：本地 / 只读 / dry-run 验证。暂不读取凭据，不访问外部 SaaS，不做 trace upload，不执行 publish/live，不写 production。

## 执行顺序

| 顺序 | 阶段 | 目标 | 产物 | 是否需要用户再次批准 |
|---:|---|---|---|---|
| 1 | Scope gate | 选择 runtime 验证等级和目标对象 | `process/checkpoints/MF-018-RUNTIME-AUTHORIZATION-SCOPE.md` | 是 |
| 2 | Precheck | 检查凭据边界、网络边界、写权限、日志脱敏、停止条件 | `process/checks/MF-018-RUNTIME-AUTHORIZATION-PRECHECK.md` | 是，若超出只读 dry-run |
| 3 | Dry-run validation | 运行本地或 sandbox dry-run，生成脱敏证据 | `process/evidence/MF-018.RUNTIME.index.json` | 取决于 scope |
| 4 | Result gate | 汇总验证结果，决定是否升级真实外部运行 | `process/checkpoints/MF-018-RUNTIME-AUTHORIZATION-RESULT.md` | 是 |

## 推荐第一个人工选择题

| 选项 | 说明 | 风险 |
|---|---|---|
| A：只读 dry-run | 不使用凭据、不访问外部 SaaS、不写外部系统，只验证 runtime 包装和证据结构 | 最低 |
| B：credentialed dry-run | 允许使用明确边界内的凭据和 sandbox，但不写 production | 中，需要凭据和脱敏策略 |
| C：true publish / production write | 允许真实发布或写入生产目标 | 高，需要完整 runtime authorization CR、回滚和事故停止条件 |

## 当前建议

原推荐是选择 A。用户已回复 `c`，当前按 C：true publish / production write 路径记录。

## C 路径处理结果

| 字段 | 内容 |
|---|---|
| 用户选择 | `C：true publish / production write` |
| 当前状态 | `staged-component-validation-planned`，C0 仍 `BLOCKED` |
| 原因 | 最终 C 可以作为目标，但必须先通过 C0-C6 组件验证 |
| 下一步 | 先完成 C0 Runtime target definition |

C 不会直接执行。即使 scope 选择了 C，也必须先通过 runtime-high-risk-human-gate。

## 分阶段计划

| 阶段 | 名称 | 目标 | 产物 | 状态 |
|---|---|---|---|---|
| C0 | Runtime target definition | 定义真实 runtime 目标、写入范围、回滚和成功标准 | `process/checkpoints/MF-018-RUNTIME-TARGET-DEFINITION.md` | BLOCKED |
| C1 | Trace schema local fixture | 本地生成 OpenTelemetry / OpenInference trace 样例 | `process/evidence/MF-018.RUNTIME.C1.trace-fixture.index.json` | NOT_STARTED |
| C2 | Redaction and deny-list validation | 验证敏感字段不会进入 trace/export/evidence | `process/checks/MF-018-RUNTIME-REDACTION-CHECK.md` | NOT_STARTED |
| C3 | Local OTLP collector pipeline | 本地 Collector 接收、过滤、导出到本地 sink | `process/evidence/MF-018.RUNTIME.C3.collector.index.json` | NOT_STARTED |
| C4 | Local trace backend | Phoenix local 或 Langfuse self-host 接收脱敏 trace | `process/evidence/MF-018.RUNTIME.C4.local-backend.index.json` | NOT_STARTED |
| C5 | SaaS contract dry-run | 只生成 SaaS payload preview 和配置，不上传 | `process/checkpoints/MF-018-SAAS-CONTRACT-DRY-RUN.md` | NOT_STARTED |
| C6 | Credentialed sandbox dry-run | 最小凭据 + sandbox 小样本上传 | `process/evidence/MF-018.RUNTIME.C6.sandbox.index.json` | NOT_STARTED |
| C7 | Small-sample true runtime | 小样本真实 runtime，受限写入和可回滚 | `process/evidence/MF-018.RUNTIME.C7.true-runtime.index.json` | NOT_STARTED |
| C8 | Production runtime decision | 决定是否进入持续 production runtime | `process/checkpoints/MF-018-RUNTIME-PRODUCTION-DECISION.md` | NOT_STARTED |

详细计划见 `process/checks/MF-018-RUNTIME-COMPONENT-VALIDATION-PLAN.md`。
