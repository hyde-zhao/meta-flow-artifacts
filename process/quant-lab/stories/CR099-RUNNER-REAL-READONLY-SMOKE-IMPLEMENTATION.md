---
story_id: "CR099-RUNNER-REAL-READONLY-SMOKE"
story_slug: "RUNNER-REAL-READONLY-SMOKE"
status: "ready-for-verification"
owner: "host-orchestrator"
created_at: "2026-06-19T16:13:23+08:00"
scope: "offline contract and redacted evidence checker only"
---

# CR099 Runner Real Readonly Smoke Implementation

## 实现前置检查

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| CP5 已批准 | PASS | `process/checkpoints/CP5-CR099-RUNNER-REAL-READONLY-SMOKE-READINESS.md` | 用户回复“同意”，接受 DQ-CP5-CR099-01..05。 |
| 设计证据可读 | PASS | `process/stories/CR099-RUNNER-REAL-READONLY-SMOKE-LLD.md` | 状态更新为 `approved-for-offline-implementation`。 |
| 文件所有权 | PASS | CP5 Decision Brief / LLD | 仅新增 CR099 离线 checker、pytest 和过程证据。 |
| 禁止范围 | PASS | 本轮命令清单 | 未读取 HMAC env、Windows `.env`、账户原文；未启动 gateway；未执行 runner runtime；未访问 NAS。 |

## 实现对象清单

| 对象 | 路径 | 类型 | 验证方式 |
|---|---|---|---|
| CR099 run contract / redacted evidence checker | `scripts/check_cr099_redacted_evidence.py` | guardrail / schema checker | CR099 pytest、CLI help、py_compile |
| CR099 checker tests | `tests/test_cr099_runner_real_readonly_smoke_contract.py` | tests | pytest |
| CP6 evidence | `process/stories/CR099-RUNNER-REAL-READONLY-SMOKE-IMPLEMENTATION.md` | process evidence | CP6 check |
| CP6 coding done | `process/checks/CP6-CR099-RUNNER-REAL-READONLY-SMOKE-CODING-DONE.md` | checkpoint evidence | cr-tracking / review |

## 设计契约映射

| 设计契约 | 实现位置 | 验证 |
|---|---|---|
| run contract 必须含 `run_id` / `authorization_ref` / host / port / allowlist / env_ref / evidence dir / abort conditions | `check_run_contract()` | `test_cr099_run_contract_accepts_minimal_authorized_reference` |
| allowlist 只能是 3 个 readonly endpoint | `ALLOWED_ENDPOINTS` + `check_run_contract()` | `test_cr099_run_contract_rejects_order_endpoint` |
| contract 不保存 secret / `.env` | forbidden marker validation | `test_cr099_run_contract_rejects_secret_like_env_ref` |
| evidence 必须覆盖 3 个 endpoint | `check_redacted_evidence()` | `test_cr099_redacted_evidence_accepts_zero_position_summary` |
| forbidden counters 必须全 0 | `REQUIRED_FORBIDDEN_COUNTERS` | `test_cr099_redacted_evidence_rejects_nonzero_forbidden_counter` |
| raw payload 不得输出 | `raw_payload_emitted` false validation | `test_cr099_redacted_evidence_rejects_raw_payload_marker` |
| checker 不导入 runtime / network 模块 | static AST import check | `test_cr099_checker_does_not_import_runtime_or_network_modules` |

## 单元测试与 Fixture 计划

| Fixture / 测试 | 覆盖点 | 状态 |
|---|---|---|
| valid contract fixture | contract schema happy path | PASS |
| invalid order endpoint fixture | endpoint allowlist fail closed | PASS |
| secret-like env ref fixture | secret / env marker rejection | PASS |
| valid redacted evidence fixture | zero-position allowed redacted summary | PASS |
| nonzero forbidden counter fixture | security fail closed | PASS |
| raw payload fixture | raw leak rejection | PASS |
| AST import check | checker 不触碰 runtime / network | PASS |

## 最小实现切片

| Slice | 输出 | 局部验证 | 回滚点 |
|---|---|---|---|
| S1 contract checker | `check_run_contract()` | CR099 pytest | 删除 checker 中 contract mode |
| S2 evidence checker | `check_redacted_evidence()` | CR099 pytest | 删除 checker 中 evidence mode |
| S3 CLI wrapper | `main()` / `--contract` / `--evidence` / `--json` | CLI help / py_compile | 删除 CLI 入口 |
| S4 process evidence | 本文件 / CP6 check | cr-tracking | 回退过程证据 |

## 平台差异处理

N/A。CR099 CP6 不安装 Claude / Codex / OpenClaw 平台产物，不生成 agent / skill，不触发平台目录安装结构。

## 验证结果

| 命令 | 结果 | 说明 |
|---|---|---|
| `PYTHONDONTWRITEBYTECODE=1 PYTEST_ADDOPTS='-p no:cacheprovider' uv run --python 3.11 pytest -q tests/test_cr099_runner_real_readonly_smoke_contract.py` | PASS，`7 passed in 0.11s` | CR099 contract / evidence checker tests。 |
| `PYTHONDONTWRITEBYTECODE=1 PYTEST_ADDOPTS='-p no:cacheprovider' uv run --python 3.11 pytest -q tests/test_cr099_runner_real_readonly_smoke_contract.py tests/test_cr098_runner_readonly_integration.py` | PASS，`14 passed in 0.20s` | CR099 + CR098 regression。 |
| `PYTHONPYCACHEPREFIX=/tmp/cr099-cp6-pycompile uv run --python 3.11 python -m py_compile scripts/check_cr099_redacted_evidence.py tests/test_cr099_runner_real_readonly_smoke_contract.py` | PASS | 目标文件编译通过。 |
| `uv run --python 3.11 python scripts/check_cr099_redacted_evidence.py --help` | PASS | CLI 可用；未读取任何 evidence / env。 |

## 未覆盖项

- 未执行真实 Windows gateway runner smoke：CP5 明确不授权，后续如需真实 runtime 必须逐 run 授权。
- 未读取 `/home/hyde/.quant-lab/runtime/qmt/cr097/client.env.template`、Windows `.env`、账户原文或持仓原文。
- 未访问 NAS、provider/lake/publish、submit/cancel、simulation/live。
- 尚未验证用户提供的真实 redacted evidence 文件；未来必须由用户提供单一文件路径或 run contract 指定路径，禁止目录扫描。

## 设计缺口反馈

- 当前 CP6 只证明 run contract / evidence checker 可离线阻断越权输入，不证明 Windows gateway 可达。
- `blocked-preflight` evidence 可作为失败路径证据，但不能证明真实 runner smoke 成功。
- 空持仓 evidence 可被接受，但非空持仓路径仍需独立 follow-up。

## 后续交接

- CP7 可执行离线验证和质量评审，不需要真实 gateway。
- 如用户要求真实 runner smoke，必须重新明确授权 env path、host/port、runtime authorization ref、run_id、allowlist 和 evidence 输出路径。
