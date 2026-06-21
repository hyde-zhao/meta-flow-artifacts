---
plan_id: "MF-018-RUNTIME-COMPONENT-VALIDATION-PLAN"
created_at: "2026-06-21T17:41:32+08:00"
owner: "host-orchestrator"
status: "planned"
source_context: "process/context/MF-018.RUNTIME-AUTHORIZATION.context.yaml"
source_scope_gate: "process/checkpoints/MF-018-RUNTIME-AUTHORIZATION-SCOPE.md"
selected_final_path: "option_c_true_publish_or_production_write"
execution_status: "not-started"
---

# MF-018 Runtime 组件分阶段验证计划

## 目标

最终目标是完成真实 runtime 验证；但在真实验证前，必须逐步验证 trace、脱敏、collector、本地后端、SaaS 契约、凭据边界、sandbox / dry-run、失败保护和证据链组件。任何阶段失败都不得升级到下一阶段。

## 总体路线

| 阶段 | 名称 | 目标 | 是否允许外部 SaaS | 是否允许凭据 | 是否允许写入 | 阶段结论 |
|---|---|---|---|---|---|---|
| C0 | Runtime target definition | 定义真实 runtime 目标、操作类型、影响面和成功标准 | 否 | 否 | 否 | 必须先完成 |
| C1 | Trace schema local fixture | 定义 OpenTelemetry / OpenInference trace schema，用本地 fixture 生成样例 | 否 | 否 | 否 | 必须 PASS |
| C2 | Redaction and deny-list validation | 验证 prompt/output/tool/error/secret 脱敏策略 | 否 | 否 | 否 | 必须 PASS |
| C3 | Local OTLP collector pipeline | 验证本地 OpenTelemetry Collector 接收、过滤、导出到本地 sink | 否 | 否 | 否 | 必须 PASS |
| C4 | Local trace backend | 验证 Phoenix local 或 Langfuse self-host 接收脱敏 trace | 否 | 否 | 否 | 必须 PASS |
| C5 | SaaS contract dry-run | 只生成 SaaS export 配置和字段映射，不上传真实数据 | 否 | 否 | 否 | 必须人工确认 |
| C6 | Credentialed sandbox dry-run | 在 sandbox 中用最小凭据验证上传或外部调用 | 允许，仅 sandbox | 允许，仅临时最小权限 | 禁止 production write | 需要 runtime gate |
| C7 | Small-sample true runtime | 小样本真实 runtime，严格限制 blast radius 和回滚条件 | 允许，限批准服务 | 允许，限批准 secret | 允许，限批准写入集 | 需要 high-risk gate |
| C8 | Production runtime decision | 根据 C7 证据决定是否进入持续 production runtime | 允许，另批 | 允许，另批 | 允许，另批 | 另行 CP / release gate |

## 推荐技术路线

| 组件 | 推荐 | 备选 | 决策理由 |
|---|---|---|---|
| Trace 协议 | OpenTelemetry OTLP | provider-specific SDK | OTLP 厂商中立，便于先本地后 SaaS |
| LLM / Agent 语义 | OpenInference conventions | 自定义 span attributes | OpenInference 覆盖 LLM call、agent step、tool invocation、retrieval 等语义 |
| Collector | Local OpenTelemetry Collector | 应用直连 SaaS | Collector 可集中脱敏、批处理和路由，降低应用侧耦合 |
| 本地后端 | Phoenix local | Langfuse self-host | Phoenix local 启动和验证成本较低；Langfuse self-host 更接近 SaaS observability |
| SaaS 候选 | Langfuse Cloud | LangSmith / W&B Weave | Langfuse OTLP 路线清晰；LangSmith 适合 LangChain/LangGraph；Weave 适合 W&B eval 工作流 |

## 阶段明细

### C0 Runtime target definition

| 项目 | 要求 |
|---|---|
| 输入 | 真实 runtime 操作类型、目标服务、环境、写入范围、成功标准 |
| 输出 | `process/checkpoints/MF-018-RUNTIME-TARGET-DEFINITION.md` |
| PASS 条件 | 操作类型、目标环境、写入范围、blast radius、rollback 和 on_failure 均明确 |
| BLOCK 条件 | 任何目标、写入或回滚信息缺失 |

### C1 Trace schema local fixture

| 项目 | 要求 |
|---|---|
| 输入 | OpenTelemetry / OpenInference span 字段白名单 |
| 输出 | `process/evidence/MF-018.RUNTIME.C1.trace-fixture.index.json` |
| PASS 条件 | 本地 fixture 可生成 trace_id、span_id、operation_name、status、duration_ms、token_count 等非敏感字段 |
| BLOCK 条件 | trace 样例包含 prompt、completion、tool payload、secret 或原始文件内容 |

### C2 Redaction and deny-list validation

| 项目 | 要求 |
|---|---|
| 输入 | 敏感字段 deny-list、正则脱敏规则、字段白名单 |
| 输出 | `process/checks/MF-018-RUNTIME-REDACTION-CHECK.md` |
| PASS 条件 | secret/token/cookie/private key、prompt 正文、completion 正文、tool I/O 正文默认不可导出 |
| BLOCK 条件 | 任何 secret-like 字段可进入 evidence 或 export payload |

### C3 Local OTLP collector pipeline

| 项目 | 要求 |
|---|---|
| 输入 | 本地 collector 配置草案，仅导出到 local file/stdout/local backend |
| 输出 | `process/evidence/MF-018.RUNTIME.C3.collector.index.json` |
| PASS 条件 | collector 可接收 OTLP、本地过滤并输出脱敏 trace；不访问 SaaS |
| BLOCK 条件 | 配置包含外部 endpoint、凭据或 production write |

### C4 Local trace backend

| 项目 | 要求 |
|---|---|
| 输入 | Phoenix local 或 Langfuse self-host 本地 endpoint |
| 输出 | `process/evidence/MF-018.RUNTIME.C4.local-backend.index.json` |
| PASS 条件 | 本地后端可查看脱敏 trace，字段完整且无敏感正文 |
| BLOCK 条件 | 需要外部网络、云账号或凭据才能完成 |

### C5 SaaS contract dry-run

| 项目 | 要求 |
|---|---|
| 输入 | SaaS 候选、字段映射、数据保留、关闭方式、allowed host |
| 输出 | `process/checkpoints/MF-018-SAAS-CONTRACT-DRY-RUN.md` |
| PASS 条件 | 只生成配置和 payload preview，不上传真实数据 |
| BLOCK 条件 | 需要真实 API key、上传 trace 或访问外部 endpoint |

### C6 Credentialed sandbox dry-run

| 项目 | 要求 |
|---|---|
| 输入 | 临时最小权限 secret、sandbox endpoint、allowed hosts、timeout_seconds |
| 输出 | `process/evidence/MF-018.RUNTIME.C6.sandbox.index.json` |
| PASS 条件 | 只向 sandbox 上传脱敏小样本，证据脱敏落盘，失败自动停止 |
| BLOCK 条件 | secret 无边界、endpoint 非 sandbox、payload 未脱敏或无 rollback / abort 策略 |

### C7 Small-sample true runtime

| 项目 | 要求 |
|---|---|
| 输入 | 高风险人工授权原文、写入集、blast radius、rollback owner、观察窗口 |
| 输出 | `process/evidence/MF-018.RUNTIME.C7.true-runtime.index.json` |
| PASS 条件 | 小样本真实 runtime 成功，变更不超过授权写入集，观察窗口无异常 |
| BLOCK 条件 | 任何未授权写入、超出 blast radius、回滚不可用、敏感数据泄漏 |

### C8 Production runtime decision

| 项目 | 要求 |
|---|---|
| 输入 | C1-C7 全部证据 |
| 输出 | `process/checkpoints/MF-018-RUNTIME-PRODUCTION-DECISION.md` |
| PASS 条件 | 用户明确批准是否进入持续 production runtime |
| BLOCK 条件 | C7 未通过或用户未批准 |

## 门禁规则

| 规则 | 说明 |
|---|---|
| 逐级推进 | C0-C8 不允许跳级；C6/C7/C8 必须单独人工授权 |
| 默认拒绝 | 未授权时不得读取凭据、访问 SaaS、上传 trace、publish/live 或 production write |
| 数据最小化 | 默认只允许结构化 metadata，不导出 prompt/completion/tool payload/file content |
| 本地优先 | 先 Phoenix local 或 Langfuse self-host，再 SaaS |
| 证据脱敏 | 所有 evidence index 只保存摘要、路径、哈希、计数和脱敏输出 |
| 失败停止 | 任一阶段 `BLOCKED` / `FAIL` 后停止升级，创建 follow-up 或回退 |

## 当前状态

| 阶段 | 状态 | 说明 |
|---|---|---|
| C0 | BLOCKED | 用户已选择最终 C 路径，但真实 runtime 目标和写入边界尚未定义 |
| C1-C8 | NOT_STARTED | 等 C0 通过 |

## 下一步

先完成 C0：请提供真实 runtime 目标定义。没有 C0，不应生成任何外部执行 precheck，更不能执行真实 runtime。
