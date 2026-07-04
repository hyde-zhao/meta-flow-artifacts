---
status: "draft-for-cp5-review"
version: "1.0"
change_id: "CR-061"
title: "CR061 Package / Import Candidate List"
created_by: "host-orchestrator"
created_at: "2026-06-15T07:29:19+08:00"
execution_mode: "candidate-list-only"
real_move_authorized: false
bulk_rewrite_authorized: false
credential_read_authorized: false
---

# CR061 Package / Import Candidate List

## 1. 范围和结论

本文冻结 CR061 package/import/layout 候选对象。当前不是执行清单，不修改 `pyproject.toml`、`uv.lock`、源码目录、测试或文档命令。

| 项目 | 结论 |
|---|---|
| 当前动作 | 只固化 candidate row、owner、classification、risk、verification_rule 和 fail-closed 规则。 |
| 主要风险 | 现有 295 个 Python 文件依赖顶层 import roots；bulk rewrite 风险高。 |
| 推荐方向 | staged compatibility-first：先保留 legacy roots，再引入 `quant_lab` compatibility namespace。 |
| 默认保护范围 | `process/**`、历史 checkpoint / handoff / Story / LLD / DEV-LOG、`.env`、NAS data bodies、lake data bodies。 |
| 执行授权 | `execute_allowed=false`。 |

## 2. Candidate 字段合同

| 字段 | 必填 | 允许值 / 规则 |
|---|---:|---|
| `candidate_id` | yes | `CR061-CAND-###`。 |
| `path_pattern` | yes | repo-relative exact path 或路径族；禁止 NAS / lake body path。 |
| `reference_family` | yes | `project_metadata` / `import_root` / `cli_entrypoint` / `runtime_config` / `historical_audit` / `trading_boundary` / `test_regression`。 |
| `context_kind` | yes | `source_code` / `test` / `script` / `metadata` / `doc` / `process_evidence` / `runtime_config`。 |
| `classification` | yes | `mechanical-candidate` / `compatibility-candidate` / `manual-review` / `preserve-audit` / `keep` / `out-of-scope` / `blocked-sensitive`。 |
| `proposed_action` | yes | `keep` / `metadata_update_candidate` / `compatibility_bridge_candidate` / `manual_review` / `preserve_audit` / `out_of_scope` / `blocked_sensitive`。 |
| `owner` | yes | 负责确认的人或角色。 |
| `risk` | yes | `low` / `medium` / `high` / `blocked`。 |
| `verification_rule` | yes | 后续 CP7 或人工复核入口。 |
| `rollback_ref_required` | yes | metadata update、compatibility bridge、move / rewrite 候选必须为 true。 |
| `source_evidence` | yes | CR060 / CR058 / scan 证据路径。 |
| `execute_allowed` | yes | 当前固定为 `false`。 |

禁止值：`move_now`、`rename_now`、`delete_now`、`bulk_replace_now`、`git_push_now`、`runtime_connect_now`。

## 3. Candidate 分类清单

| candidate_id | path_pattern | reference_family | context_kind | classification | proposed_action | owner | risk | verification_rule | rollback_ref_required | source_evidence | execute_allowed | 说明 |
|---|---|---|---|---|---|---|---|---|---:|---|---:|---|
| CR061-CAND-001 | `pyproject.toml` `[project].name` | project_metadata | metadata | mechanical-candidate | metadata_update_candidate | meta-dev / host-orchestrator | high | `uv lock` diff + `uv run --python 3.11 pytest -q` 或批准的 offline subset | true | CR060 deferred package/import boundary | false | 可从 `local-backtest` 收敛为 `quant-lab`，但必须同步 lock。 |
| CR061-CAND-002 | `uv.lock` package metadata | project_metadata | metadata | mechanical-candidate | metadata_update_candidate | meta-dev | medium | `uv lock` 生成，不手工编辑；YAML/TOML parse 和 pytest | true | `pyproject.toml` candidate | false | 只能由 `uv lock` 或等价 uv 命令生成。 |
| CR061-CAND-003 | `engine/**` | import_root | source_code | compatibility-candidate | compatibility_bridge_candidate | meta-dev | high | `python -c "import engine"` 和 `python -c "import quant_lab.engine"` smoke | true | repo scan 2026-06-15 | false | 当前保留 legacy root；后续可加 `quant_lab.engine` bridge。 |
| CR061-CAND-004 | `market_data/**` | cli_entrypoint | source_code | compatibility-candidate | compatibility_bridge_candidate | meta-dev / data owner | high | `python -m market_data.cli --help` 继续有效；新 namespace smoke 不写 lake | true | repo scan 2026-06-15 | false | CLI 和 lake/publish 边界高风险，不做 root replacement。 |
| CR061-CAND-005 | `strategies/**` | import_root | source_code | compatibility-candidate | compatibility_bridge_candidate | meta-dev | medium | `python -c "import strategies"` 和 strategy import smoke | true | repo scan 2026-06-15 | false | 研究策略 import root 保留兼容期。 |
| CR061-CAND-006 | `trading/**` | trading_boundary | source_code | manual-review | manual_review | meta-dev / trading owner | high | static import smoke only；不得连接 QMT/MiniQMT | true | CR046 paused boundary + repo scan | false | 与 CR046 潜在重叠，任何 runtime 行为禁止。 |
| CR061-CAND-007 | `experiments/**` | import_root | source_code | manual-review | manual_review | meta-dev | medium | selected offline experiment import smoke；不读取 provider | true | repo scan 2026-06-15 | false | 实验脚本大量依赖 legacy roots，先不批量改。 |
| CR061-CAND-008 | `scripts/**` | cli_entrypoint | script | manual-review | manual_review | meta-dev / meta-qa | high | only safe `--help` / static compile；禁止 provider fetch / lake write | true | repo scan 2026-06-15 | false | 许多脚本可写 lake 或 fetch provider，默认不执行。 |
| CR061-CAND-009 | `tests/**` | test_regression | test | mechanical-candidate | metadata_update_candidate | meta-qa | medium | add/import smoke tests after implementation; full offline pytest preferred | true | repo scan 2026-06-15 | false | 测试是回归基线，先新增 smoke，不大改旧断言。 |
| CR061-CAND-010 | `engine/diagnostics.py` `LOGGER_NAME` | project_metadata | source_code | manual-review | manual_review | meta-dev | low | logging name smoke; no behavior change expected | true | `rg local_backtest` scan | false | 可从 `local_backtest.cli_diag` 迁移到 `quant_lab.cli_diag`，但需确认日志兼容。 |
| CR061-CAND-011 | README / USER-MANUAL `UV_CACHE_DIR=/tmp/uv-cache-local-backtest` | runtime_config | doc | manual-review | manual_review | meta-doc / host-orchestrator | medium | command docs review; no `.env` read | true | `rg local-backtest` scan | false | cache path不是项目包名，改名会影响用户本地缓存；默认延后。 |
| CR061-CAND-012 | `notebooks/local_research_intro.ipynb` | historical_audit | doc | manual-review | manual_review | meta-doc | low | notebook metadata review without executing notebook | true | `rg local_backtest` scan | false | 不执行 notebook；可后续更新标题。 |
| CR061-CAND-013 | `process/**` / `checkpoints/**` / `DEV-LOG.md` | historical_audit | process_evidence | preserve-audit | preserve_audit | host-orchestrator | high | historical evidence remains unchanged | false | CR058 preserve-audit allowlist | false | 不批量替换历史语境。 |
| CR061-CAND-014 | `.env` / credentials / token paths | runtime_config | runtime_config | blocked-sensitive | blocked_sensitive | security owner | blocked | do not read values | false | AGENTS + CR060 boundaries | false | 禁止读取凭据正文。 |
| CR061-CAND-015 | Git remote / branch / tag / push | project_metadata | metadata | out-of-scope | out_of_scope | host-orchestrator | blocked | CR062 only | true | CR060 release context | false | 归 CR062。 |
| CR061-CAND-016 | NAS data / data lake root / catalog / provider fetch | runtime_config | runtime_config | out-of-scope | out_of_scope | data owner | blocked | CR063 / CR064 only | true | CR060 release context | false | 归 CR063/CR064。 |

## 4. Fail-closed 规则

| 规则 ID | 条件 | 结果 | 说明 |
|---|---|---|---|
| CLG-CR061-01 | 必填字段缺失 | `manual-review` | 缺 owner、risk、verification_rule 或 source_evidence 时不得执行。 |
| CLG-CR061-02 | proposed_action 含真实执行语义 | `blocked` | 出现 `move_now` / `bulk_replace_now` / `delete_now` 直接失败。 |
| CLG-CR061-03 | 命中 historical audit path | `preserve-audit` | 交给 `PRESERVE-AUDIT-DECISIONS-CR061.md`。 |
| CLG-CR061-04 | 命中 sensitive path | `blocked-sensitive` | 不读取正文，不打印值。 |
| CLG-CR061-05 | 涉及 `trading/**` runtime | `blocked` | 只允许 static import/layout review；runtime 归 CR065 或 CR046 恢复。 |
| CLG-CR061-06 | rollback_ref 缺失 | `execute_allowed=false` | 真实实现前必须补齐。 |

## 5. 后续验证入口

| 验证对象 | 入口 | 预期 |
|---|---|---|
| 字段完整性 | 本文 §2 / §3 | 每行 candidate 字段完整。 |
| 非法 action | `rg -n "move_now|rename_now|delete_now|bulk_replace_now|git_push_now|runtime_connect_now" docs/release/CR061-*.md` | 只允许出现在禁止值语境。 |
| legacy import smoke | 后续 CP6 / CP7 | `engine`、`market_data`、`strategies`、`trading` 旧入口不破坏。 |
| new namespace smoke | 后续 CP6 / CP7 | `quant_lab.*` 仅在实现后验证。 |

## 6. 未授权项

本文不授权真实 file move / rename / delete / rewrite，不授权 `pyproject.toml` 或 `uv.lock` 修改，不授权 NAS / data lake / provider / QMT / MiniQMT / git remote / credential 操作，不恢复 CR046。
