---
change_id: "CR091"
title: "QMT Strategy Runner Offline CP7 Review"
owner: "meta-qa-critical"
created_at: "2026-06-18T14:59:24+08:00"
review_mode: "cp7-quality-review"
stage_decision: "PASS_WITH_RISK"
---

# CR091 QMT Strategy Runner Review

## Findings

未发现需要 meta-dev 回修的 `BLOCKER`、`HIGH`、`MEDIUM` 或 `LOW` 实现缺陷。

## Non-Blocking Observations

| ID | 等级 | 观察 | 证据 | 处理 |
|---|---|---|---|---|
| OBS-CR091-CP7-001 | INFO | `meta-flow check cr-tracking --project-root .` 返回 exit 1，但 summary 显示当前 active formal CRs 为 `CR-091`，错误集中在历史 CR025 nested active_change 和 CR019 follow-up 状态旧账 | 命令输出列出 `STATE line 35656 nested.active_change points to finished CR CR-025`、CR019 follow-up 多个历史状态错误 | 不阻断 CR091 离线 CP7；建议 host-orchestrator 后续单独治理 |
| OBS-CR091-CP7-002 | INFO | `write_evidence_summary()` 是本地写出函数，本轮 checker 未调用落盘；函数写出前有 `assert_redacted()` | `trading/strategy_runner/evidence.py` | 后续若启用 evidence 落盘，应由调用方限定输出目录 |

## Review Scope

| 范围 | 路径 / 对象 | 状态 |
|---|---|---|
| 代码 | `trading/strategy_runner/` | reviewed |
| 脚本 | `scripts/check_cr091_strategy_runner_package.py` | reviewed |
| 测试 | `tests/test_cr091_strategy_runner_contracts.py` | reviewed |
| fixture | `tests/fixtures/cr091_strategy_runner/` | reviewed |
| 设计证据 | HLD / LLD / TEST-PLAN / CP5 / CP6 / IMPLEMENTATION | reviewed |
| 非范围 | QMT runtime、NAS、`.env`、凭据、账户、交易写、simulation/live | not executed |

## Contract Review

| 审查项 | 结论 | 说明 |
|---|---|---|
| Adapter 输出统一 | PASS | 多因子、legacy、package adapter 均输出 target portfolio 与 order intent draft |
| `qmt_allowed=false` | PASS | pytest 对 order intent draft 做断言 |
| `not_authorization=true` | PASS | target portfolio、adapter result、checker JSON 和 fixtures 均保留该语义 |
| Package fail closed | PASS | schema、checksum、payload checksum、manifest flags 非 false 均被阻断 |
| Cache fail closed | PASS | active pointer 逃逸、缺失、缺 `.immutable` 均有检查或测试 |
| Readonly gateway boundary | PASS | wrapper 只允许 health / capabilities / query_positions；`submit_order` 被阻断 |
| Evidence redaction | PASS | forbidden counters 合并，敏感 key / value 被阻断 |

## Security Boundary Review

静态扫描覆盖：

- `trading/strategy_runner`
- `scripts/check_cr091_strategy_runner_package.py`
- `tests/test_cr091_strategy_runner_contracts.py`
- `tests/fixtures/cr091_strategy_runner`

扫描模式覆盖 `xtquant`、`.env`、`os.environ`、`dotenv`、`requests`、`socket`、`urllib`、`http.client`、`subprocess`、`StdlibQmtRestTransport`、`QmtRestTransport`、NAS / credential / account / submit / cancel / simulation / live / provider / lake / publish 等。

结论：

- 未发现 CR091 文件直接 import `xtquant`。
- 未发现 `.env`、`os.environ`、`dotenv` 读取入口。
- 未发现 `requests`、`urllib`、`http.client`、`subprocess` 或真实 socket 客户端导入。
- 命中 `socket` 的位置为注释“不打开 socket”和 `gateway_socket_open` forbidden counter，不是真实 socket 调用。
- 命中 NAS / credential / account / submit / cancel / simulation / live / provider / lake / publish 的位置为 forbidden counter、manifest false flags、fixture 0 值、evidence alias 和负向测试。
- 本地 `read_text` / `read_bytes` 仅用于 fixture、manifest、payload、cache pointer 和 checksum；未读取 `.env`、凭据或账户文件。

## Test Review

| 测试 | 结论 | 覆盖点 |
|---|---|---|
| `13 passed in 0.12s` | PASS | adapter、package/cache、fake gateway、evidence |
| checker JSON `passed=true` | PASS | fixture package intake、active pointer、adapter dispatch、fake query_positions、forbidden counters |
| `git diff --check` | PASS | 空白错误 |
| `py_compile` | PASS | CR091 新增 Python 文件语法 |

测试设计没有只覆盖 happy path：负向测试覆盖 empty targets、bad checksum、missing payload checksum、missing immutable marker、authorization flag true、submit_order scope deny、gateway socket counter nonzero 和 sensitive payload redaction。

## Residual Risks

| 风险 ID | 等级 | 说明 | 建议 |
|---|---|---|---|
| R-CR091-CP7-001 | HIGH | 本轮完全离线，不证明真实 QMT / MiniQMT / XtQuant / gateway / runner runtime 可用 | CP8 必须风险接受；真实验证必须新建 runtime authorization |
| R-CR091-CP7-002 | MEDIUM | 历史 CR tracking 旧账导致全局 consistency 命令非零 | 不阻断 CR091；建议单独流程债治理 |

## Review Decision

`PASS_WITH_RISK`。CR091 离线 implementation slice 无需回修；风险集中在未授权真实 runtime 和历史 CR tracking 旧账，必须进入 CP8 风险接受 / follow-up tracking。
