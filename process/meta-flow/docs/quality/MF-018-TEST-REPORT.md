# MF-018 Test Report

## 验证范围

MF-018 覆盖轻量质量模型、eval matrix、质量治理 CLI 和从现有 CP result / event ledger / read-expansion ledger 派生的最小 workflow metrics。

## 验证对象清单

| 对象 | 路径 / 命令 | 结果 |
|---|---|---|
| Quality model policy | `process/policies/QUALITY-MODEL.yaml` | PASS |
| Eval matrix policy | `process/policies/EVAL-MATRIX.yaml` | PASS |
| Quality checker | `meta_flow/checks/quality_governance.py` | PASS |
| CLI wiring | `meta_flow/cli.py` | PASS |
| Delivery template route | `delivery/skills/context-manifest-builder/templates/*QUALITY*`, `*EVAL*` | PASS |
| Workspace metadata compatibility | `meta_flow/workspace/routing.py` | PASS |
| Focused tests | `tests/test_quality_governance.py` | PASS |

## 追踪矩阵

| 需求 / 契约 | 验证方式 | 证据 | 结果 |
|---|---|---|---|
| 新增 `QUALITY-MODEL` 模板 | `meta-flow quality model-check --project-root .` | policy checker output | PASS |
| 新增 `EVAL-MATRIX` 模板 | `meta-flow quality eval-check --project-root .` | policy checker output | PASS |
| 禁止手工 metrics truth source | unit test + policy validation | `tests/test_quality_governance.py` | PASS |
| workflow metrics 只从现有结果和 ledgers 派生 | `meta-flow doctor workflow --project-root .` | doctor output | PASS |
| 不引入 dashboard / ranking / portfolio | policy forbidden-source validation | `tests/test_quality_governance.py` | PASS |
| 未来项目可初始化默认 policy | `meta-flow quality init --project-root /tmp/mf018-quality-init-check --force` | command output | PASS |
| 模板进入 delivery 安装产物 | install dry-run includes context-manifest-builder templates | command output | PASS |
| 安装 dry-run 不破坏现有交付 | install dry-run | command output | PASS |

## 分层验证计划与结果

| 层级 | 命令 | 结果 |
|---|---|---|
| Policy structure | `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 meta-flow quality model-check --project-root .` | PASS |
| Eval matrix structure | `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 meta-flow quality eval-check --project-root .` | PASS |
| Quality doctor | `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 meta-flow doctor quality --project-root .` | PASS |
| Workflow metrics doctor | `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 meta-flow doctor workflow --project-root .` | PASS |
| Quality policy init | `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 meta-flow quality init --project-root /tmp/mf018-quality-init-check --force` | PASS |
| Automated regression | `PYTHONDONTWRITEBYTECODE=1 PYTEST_ADDOPTS='-p no:cacheprovider' uv run --with pytest --python 3.11 pytest -q` | PASS, 114 passed |
| Delivery guardrail | `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 python scripts/check_delivery_guardrails.py` | PASS |
| Install dry-run | `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 meta-flow install --platform codex --scope project --component agent --project-dir . --dry-run` | PASS |
| Whitespace check | `git diff --check` | PASS |

## 问题与剩余风险

| ID | Severity | 状态 | 说明 | 下一步 |
|---|---|---|---|---|
| MF-018-RISK-01 | LOW | CLOSED | policy 模板已进入 `delivery/skills/context-manifest-builder/templates/`，并新增 `meta-flow quality init` 初始化路由。 | 后续只需在发布文档中说明初始化命令。 |

## 阶段决策

MF-018 自动验证通过，建议进入 CP7 / QA 后续确认；delivery 模板路由已补齐，无剩余阻断项。
