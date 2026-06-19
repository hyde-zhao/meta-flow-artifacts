---
story_id: "CR099-RUNNER-REAL-READONLY-SMOKE"
cr_id: "CR-099"
status: "verified-with-risk"
lld_policy: "full-lld"
owner: "host-orchestrator"
created_at: "2026-06-19T16:04:52+08:00"
source_hld: "docs/qmt/CR099-QMT-RUNNER-REAL-READONLY-SMOKE-HLD.md"
feature_design_refs:
  - "docs/features/cr099-runner-real-readonly-smoke/DESIGN.md"
  - "docs/features/cr099-runner-real-readonly-smoke/TEST-PLAN.md"
  - "docs/features/cr099-runner-real-readonly-smoke/TASKS.md"
---

# CR099 Runner Real Readonly Smoke LLD

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 0.1 | 2026-06-19 | host-orchestrator | 初版 LLD，冻结 run contract、authorization gate、evidence schema、preflight、失败路径、验证入口和不授权边界。 |
| 0.2 | 2026-06-19 | host-orchestrator | 用户回复“同意”批准 CP5；进入 CP6 offline contract / schema checker implementation，仍不授权 runtime。 |

## 1. Story 目标

在 CR098 runner readonly adapter 和 CR097 Windows gateway readonly baseline 之上，定义一次真实 runner readonly smoke 的可审查执行合同。该 Story 的 CP5 目标是冻结设计，不执行 runtime。

## 2. 验收标准映射

| AC | 设计证据 | 验证方式 |
|---|---|---|
| runner 调用 `health`、`capabilities`、`query_positions_readonly` | run contract allowlist | CP6 contract tests；runtime 授权后 smoke |
| 每次运行有 `authorization_ref` / `run_id` | Run Contract | CP5 review；CP6 schema validation |
| evidence 脱敏 | Evidence Schema | forbidden counters / redaction checker |
| 无 raw account / positions / order / fill / log | Reject Rules | CP6 / CP7 checker |
| 缺授权不运行 | Authorization Gate | blocked preflight tests |

## 3. 接口与数据结构

### 3.1 Run Contract 必填字段

| 字段 | 类型 | 规则 |
|---|---|---|
| `schema_version` | int | 固定 `1` |
| `cr_id` | string | 固定 `CR-099` |
| `run_id` | string | 用户逐 run 授权提供 |
| `authorization_ref` | string | 用户可见授权引用，不含 secret |
| `gateway.host` / `gateway.port` | string / int | 用户逐 run 授权提供 |
| `gateway.allowlist` | list | 只能包含 3 个 readonly endpoint |
| `client_env.env_ref` | string | 仅授权引用；不得保存 secret 值 |
| `evidence.output_dir` | string | redacted evidence 目录 |
| `abort_conditions` | list | 必须包含缺授权、redaction、forbidden counter 等中止条件 |

### 3.2 Evidence 必填字段

| 字段 | 规则 |
|---|---|
| `run_id` / `authorization_ref` | 与 run contract 一致 |
| `execution_mode` | `codex-runner-authorized`、`user-manual-windows-side` 或 `blocked-preflight` |
| `endpoints.*.attempted` | 只对 allowlist endpoint 记录 |
| `raw_payload_emitted` | 必须为 `false` |
| `forbidden_counters.*` | 必须全部为 `0` 才可 PASS |

## 4. 控制流

1. 检查 CP5 checkpoint 是否 approved。
2. 检查用户是否提供逐 run 授权。
3. 校验 run contract 字段和 readonly allowlist。
4. 校验 redaction gate 和 evidence output dir。
5. 在授权范围内执行 runner readonly smoke，或进入用户手工执行 fallback。
6. 生成 redacted evidence 或 blocked preflight evidence。
7. CP6 / CP7 校验 evidence schema 和 forbidden counters。

## 5. 异常处理

| 异常 | 处理 |
|---|---|
| CP5 pending | 停止，返回 `blocked-pending-cp5` |
| 无逐 run 授权 | 停止，返回 `blocked-pending-runtime-authorization` |
| env ref 不明确 | 停止，返回 `blocked-missing-env-authorization` |
| allowlist 出现交易写 | 停止，返回 `blocked-invalid-endpoint` |
| gateway 连接失败 | 记录 redacted preflight blocked，不扩大权限 |
| redaction 失败 | 不落 raw payload，返回 `blocked-redaction` |
| forbidden counter 非 0 | 返回 `blocked-security` |

## 6. 测试与验证

| 验证项 | 方式 |
|---|---|
| contract 字段完整 | offline contract test |
| allowlist 限制 | offline contract test |
| redacted evidence schema | schema checker |
| raw leak 拒收 | fixture with forbidden fields |
| runtime smoke | CP5 后逐 run 授权 |
| user manual fallback | 单一用户指定 evidence 文件 validation |

## 7. 发布与回滚

本 Story 在 CP5 不发布、不运行。CP6 若只新增 checker / tests，回滚方式是删除 CR099 checker / tests 和 CP6 实现文件，保留 CR099 过程文档。真实 run 失败不回滚代码，而是把 CR099 标记为 blocked 或 READY_WITH_RISK 并进入 CP8 / follow-up。

## 8. Gotchas

- CP5 approved 后也不能自动读取 env 或运行 runner。
- evidence checker 必须拒绝 raw account / positions / orders / fills / logs，而不是只隐藏显示。
- `position_count_bucket=zero` 是合法结果，但应在 CP8 风险中说明非空路径未证明。
- 用户手工 evidence fallback 只能读取用户指定文件，不能扫描 `.quant-lab` 目录。
