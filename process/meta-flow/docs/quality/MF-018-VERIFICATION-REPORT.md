# MF-018 Verification Report

## 验证范围

MF-018 验证范围覆盖：

- 轻量质量模型 policy 与 eval matrix policy。
- `meta-flow quality init/model-check/eval-check`。
- `meta-flow doctor quality/workflow`。
- delivery 模板路由和用户文档 / release notes。
- CP6 return、evidence index、CP6 result、QA test report 和 review。

非范围：

- 不生成 dashboard。
- 不做 agent performance ranking。
- 不做 portfolio reporting。
- 不创建手工维护的 `WORKFLOW-METRICS` 真相源。

## 验证模式

`validation_mode=mixed`。

理由：本次同时涉及 CLI 代码、policy 模板、delivery skill 模板、安装 dry-run、文档和过程证据。

## 验证对象清单

| 对象 | 路径 / 命令 | 验证方式 | 结果 |
|---|---|---|---|
| Quality governance checker | `meta_flow/checks/quality_governance.py` | unit tests + CLI checks | PASS |
| CLI wiring | `meta_flow/cli.py` | CLI commands + full pytest | PASS |
| Workspace routing compatibility | `meta_flow/workspace/routing.py` | focused tests + full pytest | PASS |
| Quality model process policy | `process/policies/QUALITY-MODEL.yaml` | `quality model-check` | PASS |
| Eval matrix process policy | `process/policies/EVAL-MATRIX.yaml` | `quality eval-check` | PASS |
| Delivery quality model template | `delivery/skills/context-manifest-builder/templates/QUALITY-MODEL-TEMPLATE.yaml` | install dry-run + policy parity review | PASS |
| Delivery eval matrix template | `delivery/skills/context-manifest-builder/templates/EVAL-MATRIX-TEMPLATE.yaml` | install dry-run + policy parity review | PASS |
| User documentation | `delivery/README.md`, `delivery/doc/USER-MANUAL.md` | semantic review | PASS |
| Release notes | `docs/release/RELEASE-NOTES.md` | semantic review | PASS |
| CP6 evidence | `process/checks/CP6-MF-018-QUALITY-GOVERNANCE-CODING-DONE.result.json` | `meta-flow cp result-check` | PASS |

## 验证追踪矩阵

| Contract | Implementation | Verification | Status | Risk |
|---|---|---|---|---|
| Quality model template exists | `process/policies/QUALITY-MODEL.yaml` and delivery template | `meta-flow quality model-check --project-root .` | PASS | none |
| Eval matrix template exists | `process/policies/EVAL-MATRIX.yaml` and delivery template | `meta-flow quality eval-check --project-root .` | PASS | none |
| Future projects can initialize policies | `meta-flow quality init --project-root .` | focused unit test + temp project command | PASS | none |
| Workflow metrics are derived-only | `run_workflow_doctor()` | tests + `doctor workflow` | PASS | none |
| No dashboard/ranking/portfolio/manual metrics source | forbidden-source validation | tests + review | PASS | none |
| Delivery install route carries templates | context-manifest-builder templates | install dry-run output | PASS | none |
| User docs mention initialization | README / USER-MANUAL / release notes | semantic review | PASS | none |

## 分层验证计划

| 层级 | 命令 / 方法 | 结果 |
|---|---|---|
| Unit / fixture | `PYTHONDONTWRITEBYTECODE=1 PYTEST_ADDOPTS='-p no:cacheprovider' uv run --with pytest --python 3.11 pytest -q tests/test_quality_governance.py` | PASS |
| Full regression | `PYTHONDONTWRITEBYTECODE=1 PYTEST_ADDOPTS='-p no:cacheprovider' uv run --with pytest --python 3.11 pytest -q` | PASS, 115 passed |
| Policy model check | `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 meta-flow quality model-check --project-root .` | PASS |
| Eval matrix check | `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 meta-flow quality eval-check --project-root .` | PASS |
| Init route check | `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 meta-flow quality init --project-root /tmp/mf018-quality-init-check --force` | PASS |
| CP6 result check | `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 meta-flow cp result-check --result process/checks/CP6-MF-018-QUALITY-GOVERNANCE-CODING-DONE.result.json --project-root .` | PASS |
| CP6 evidence check | `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 meta-flow story evidence-check --index process/evidence/MF-018.CP6.index.json --project-root .` | PASS |
| Install dry-run | `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 meta-flow install --platform codex --scope project --component agent --project-dir . --dry-run` | PASS |
| Delivery guardrail | `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 python scripts/check_delivery_guardrails.py` | PASS |
| Static whitespace | `git diff --check` | PASS |

## Agent Dispatch Evidence

| 检查项 | 状态 | 证据 | 说明 |
|---|---|---|---|
| 子 agent 调度模式 | WAIVED | 当前用户指令：“按照你的建议推进” | 当前工具约束不允许在用户未明确要求子 agent / 并行代理时拉起 `meta-qa`；由 host-orchestrator inline fallback 执行验证。 |
| canonical role | WAIVED | `inline-fallback` | 目标角色为 `meta-qa`，实际执行者为 host-orchestrator。 |
| Codex custom agent | WAIVED | `inline-fallback` | 未冒充 `meta-qa` 子 agent。 |
| reasoning profile | WAIVED | `inline-fallback` | 使用当前主进程推理配置。 |
| dispatch trigger | PASS | MF-018 CP7 verification | 用户要求按建议推进 CP7。 |
| agent 标识 | WAIVED | `inline-fallback` | 无子 agent id / thread id。 |
| 平台工具证据 | WAIVED | current tool policy | 未调用 `spawn_agent`。 |
| 完成时间 | PASS | `2026-06-21T08:30:43+00:00` | CP7 验证完成时间。 |
| inline fallback 授权 | WAIVED | 当前对话 | 用户要求“按照你的建议推进”；本报告明确披露 fallback 边界。 |

## 问题与剩余风险

No blocking findings.

剩余风险：无。

## 阶段决策

`PASS`。

MF-018 满足 CP7 验证要求，可进入发布准备 / CP8。`READY` 不等于真实发布；任何真实发布、publish、远端写入或生产操作仍需独立授权。
