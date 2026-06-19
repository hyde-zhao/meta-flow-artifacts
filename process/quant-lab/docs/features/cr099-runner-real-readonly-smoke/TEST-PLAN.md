---
feature_id: "cr099-runner-real-readonly-smoke"
cr_id: "CR-099"
status: "verified-with-risk"
owner: "host-orchestrator"
created_at: "2026-06-19T16:04:52+08:00"
source_design: "docs/features/cr099-runner-real-readonly-smoke/DESIGN.md"
---

# CR099 Runner Real Readonly Smoke Test Plan

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 0.1 | 2026-06-19 | host-orchestrator | 初版 TEST-PLAN，覆盖 run contract validation、preflight blocked paths、redacted evidence schema 和 future runtime smoke acceptance。 |
| 0.2 | 2026-06-19 | host-orchestrator | 用户回复“同意”批准 CP5；CP6 离线 checker / tests 已实现并通过。 |

## 1. 测试目标

证明 CR099 在不越权的前提下具备可执行设计：

- CP5 前不会读取 env、启动 gateway 或执行 runner。
- 缺少逐 run 授权时，系统停在 blocked，不尝试自愈。
- run contract 字段足以支撑一次真实 readonly smoke。
- evidence schema 能拒收 raw account / positions / order / fill / log 信息。
- 未来若获得授权，可验证 runner 调用 `health`、`capabilities`、`query_positions_readonly` 的红线和输出。

## 2. 测试层级

| 层级 | 范围 | 当前 CP5 是否执行 | 说明 |
|---|---|---|---|
| 静态 contract review | run contract 必填字段、allowlist、不授权边界 | 是，作为设计审查 |
| schema validation | redacted evidence 字段和 forbidden counters | 是，设计审查；CP6 可实现 checker |
| offline fixture tests | fake contract / fake evidence | CP6 后执行 | 不读 env，不打开 socket |
| blocked preflight tests | 缺授权、缺 env、非 allowlist endpoint | CP6 后执行 | 证明失败关闭 |
| real runner smoke | 连接 Windows gateway 的 3 endpoint smoke | 否 | 只在 CP5 approved 后另行逐 run 授权 |
| manual evidence validation | 用户手工运行后提供 redacted evidence | 可选 fallback | 只读取用户指定单一 evidence 文件 |

## 3. 验收场景

| 场景 ID | 场景 | 前置条件 | 期望结果 |
|---|---|---|---|
| T-CR099-01 | 缺少 CP5 approved | CP3 approved 但 CP5 pending | 不读取 env，不运行，状态 blocked |
| T-CR099-02 | 缺少逐 run 授权 | CP5 approved，但无授权文本 | 不读取 env，不运行，状态 `blocked-pending-runtime-authorization` |
| T-CR099-03 | 非 allowlist endpoint | 合同出现 submit / cancel / buy / sell | contract validation FAIL |
| T-CR099-04 | redacted evidence schema | 输入脱敏 evidence | schema PASS，forbidden counters 全部为 0 |
| T-CR099-05 | raw payload leak | evidence 出现账号、证券代码、数量、资金、委托、成交或日志原文 | FAIL / blocked-security |
| T-CR099-06 | gateway 不可达 | 有逐 run 授权但连接失败 | blocked preflight evidence，不扩大权限重试 |
| T-CR099-07 | authorized real smoke | CP5 approved + 用户逐 run 授权 + gateway ready | runner 调用 3 endpoint，输出 redacted evidence |
| T-CR099-08 | user manual fallback | Codex 不代理 runtime，用户提供单一 redacted evidence 文件 | 仅验证该文件 schema 和 forbidden counters |

## 4. 拒收规则

任一情况出现即拒收：

- 读取 HMAC secret 或 Windows `.env` 未获逐 run 授权。
- 启动 gateway 或执行 runner runtime 未获逐 run 授权。
- evidence 包含账号、证券代码、数量、资金、委托、成交或日志原文。
- 调用了 submit / cancel、buy / sell、simulation / live 相关 endpoint。
- 访问 NAS、provider/lake/catalog publish 或 release publish。
- 自动扫描 evidence 目录，而不是读取用户指定或 run contract 指定的单一范围。

## 5. CP6 / CP7 建议命令

CP5 通过后，CP6 可只实现 / 校验离线 contract 与 schema。真实 runtime 命令必须等待用户逐 run 授权后再确定，不在 CP5 内固化。

```bash
uv run --python 3.11 pytest -q tests/test_cr099_runner_real_readonly_smoke_contract.py
uv run --python 3.11 python scripts/check_cr099_redacted_evidence.py --evidence <single-redacted-evidence-file>
```

上述命令是后续设计入口；当前 CP5 不执行、不读取 `<single-redacted-evidence-file>`。
