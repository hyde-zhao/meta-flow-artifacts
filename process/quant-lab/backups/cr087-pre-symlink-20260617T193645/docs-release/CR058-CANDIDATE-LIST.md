---
status: "implemented-cp6-static"
version: "1.0"
change_id: "CR-058"
title: "CR058 Candidate List and Owner Classification Gate"
created_by: "meta-dev"
created_at: "2026-06-14T15:40:00+08:00"
source_story: "process/stories/CR058-S02-candidate-list-and-owner-classification-gate.md"
source_lld: "process/stories/CR058-S02-candidate-list-and-owner-classification-gate-LLD.md"
execution_mode: "static-only-no-op"
real_move_authorized: false
bulk_rewrite_authorized: false
credential_read_authorized: false
---

# CR058 Candidate List

## 1. 范围和结论

本文是 CR058 Batch-A 的静态 candidate list 合同与人工分类清单，不是执行计划，不生成脚本，不执行 `move`、`rename`、`delete`、批量路径替换、Git 远端操作或运行时访问。

| 项目 | 结论 |
|---|---|
| 输入来源 | `docs/release/MIGRATION-INVENTORY-CR053.md`、`docs/release/PATH-REFERENCES-CR053.md`、CR058 S02 LLD。 |
| 当前动作 | 只固化 candidate row、owner、classification、risk、verification_rule 和 fail-closed 规则。 |
| 可进入候选的范围 | 面向未来的用户文档、项目身份说明、release/user docs 中明确可人工复核的引用。 |
| 默认保护范围 | 历史 `process/**`、checkpoint、handoff、Story、LLD、DEV-LOG 和 legacy fallback evidence。 |
| 执行授权 | `execute_allowed=false`；CP6 不授权真实迁移或 rewrite。 |

## 2. Candidate 字段合同

每一行 candidate 必须具备下列字段；缺任一字段时不得进入执行清单，只能进入 `manual-review` 或 `blocked`。

| 字段 | 必填 | 允许值 / 规则 |
|---|---:|---|
| `candidate_id` | yes | `CR058-CAND-###`。 |
| `path_pattern` | yes | repo-relative exact path 或路径族；禁止真实 NAS / lake 路径。 |
| `reference_family` | yes | `project_identity` / `legacy_alias` / `runtime_config` / `historical_audit` / `trading_boundary` / `data_lake_boundary` / `other`。 |
| `context_kind` | yes | `user_doc` / `release_doc` / `design_doc` / `process_evidence` / `runtime_config` / `source_code` / `test_fixture` / `report_pointer`。 |
| `classification` | yes | `mechanical-candidate` / `manual-review` / `preserve-audit` / `blocked_sensitive` / `keep` / `out-of-scope`。 |
| `proposed_action` | yes | `keep` / `replace_candidate` / `move_candidate` / `manual_review` / `preserve_audit` / `blocked_sensitive` / `out_of_scope`。 |
| `owner` | yes | 负责确认的人或角色；未知时不得执行。 |
| `risk` | yes | `low` / `medium` / `high` / `blocked`。 |
| `verification_rule` | yes | 后续 CP7 或人工复核入口。 |
| `rollback_ref_required` | yes | `replace_candidate` / `move_candidate` 必须为 `true`。 |
| `source_evidence` | yes | CR053 / CR058 证据路径或人工新增理由。 |
| `execute_allowed` | yes | 当前固定为 `false`。 |

禁止值：`execute_move`、`execute_rewrite`、`rename_now`、`delete_now`、`bulk_replace_now`。

## 3. Candidate 分类清单

| candidate_id | path_pattern | reference_family | context_kind | classification | proposed_action | owner | risk | verification_rule | rollback_ref_required | source_evidence | execute_allowed | 说明 |
|---|---|---|---|---|---|---|---|---|---:|---|---:|---|
| CR058-CAND-001 | `README.md` | project_identity | user_doc | mechanical-candidate | replace_candidate | meta-doc / host-orchestrator | medium | 人工确认保留 legacy alias 兼容段；后续 CP7 静态审查 | true | `MIGRATION-INVENTORY-CR053.md` §3；`PATH-REFERENCES-CR053.md` §3/§4 | false | 面向未来的项目根说明可作为候选，但本轮不改写。 |
| CR058-CAND-002 | `docs/USER-MANUAL.md` | project_identity | user_doc | mechanical-candidate | replace_candidate | meta-doc / host-orchestrator | medium | 与 README 同步人工审查，不破坏旧命令说明 | true | `MIGRATION-INVENTORY-CR053.md` §3；`PATH-REFERENCES-CR053.md` §3/§4 | false | 用户手册中的未来项目根说明可作为候选。 |
| CR058-CAND-003 | `docs/release/*-CR053.md` | project_identity | release_doc | manual-review | manual_review | meta-dev / meta-qa | medium | meta-qa 静态复核是否属于 CR053 历史证据或未来 release 说明 | true | `PATH-REFERENCES-CR053.md` §4 | false | CR053 release docs 多数是历史交付证据，默认不机械改写。 |
| CR058-CAND-004 | `docs/design/BLUEPRINT.md` / `docs/design/DOMAIN-MAP.md` legacy alias 段 | legacy_alias | design_doc | keep | keep | meta-se / host-orchestrator | low | 确认已声明 `local_backtest` 为 legacy alias | false | `PATH-REFERENCES-CR053.md` §3 | false | 明确 legacy alias 的段落保留。 |
| CR058-CAND-005 | `docs/qmt/**` | trading_boundary | release_doc | manual-review | manual_review | meta-dev / meta-doc | high | QMT no-runtime boundary review；不得自动替换 C/S 边界名 | true | `MIGRATION-INVENTORY-CR053.md` §3；`PATH-REFERENCES-CR053.md` §3 | false | 交易边界语义不明，禁止机械替换。 |
| CR058-CAND-006 | `LOCAL_BACKTEST_*` historical design references | runtime_config | runtime_config | manual-review | manual_review | meta-dev / host-orchestrator | high | 兼容变量策略需独立设计，不读取 `.env` | true | `PATH-REFERENCES-CR053.md` §3 | false | 运行配置兼容名不得在本轮改动。 |
| CR058-CAND-007 | `MARKET_DATA_LAKE_ROOT` references | data_lake_boundary | runtime_config | keep | keep | meta-dev / host-orchestrator | high | 确认 ADR 边界：不替换 lake root 入口 | false | `PATH-REFERENCES-CR053.md` §3 | false | 数据湖入口保持不变，不替换为 `QUANT_LAB_*`。 |
| CR058-CAND-008 | `pyproject.toml` / import path / package name | project_identity | source_code | out-of-scope | out_of_scope | meta-dev | high | 独立 packaging / import compatibility design | true | CR058 S02 LLD §2.1 / §4 | false | 包名、导入路径不属于 Batch-A 静态文档实现。 |
| CR058-CAND-009 | git remote repository name | project_identity | other | out-of-scope | out_of_scope | host-orchestrator | blocked | 独立 git remote authorization gate | true | CR058 CP5 不授权项 | false | git remote rename / push / tag / history rewrite 禁止。 |

## 4. Fail-closed 规则

| 规则 ID | 条件 | 结果 | 说明 |
|---|---|---|---|
| CLG-CR058-01 | 必填字段缺失 | `manual-review` | 缺 owner、risk、verification_rule 或 source_evidence 时不得执行。 |
| CLG-CR058-02 | proposed_action 含真实执行语义 | `blocked` | 出现 `execute_move` / `execute_rewrite` / `delete_now` 直接失败。 |
| CLG-CR058-03 | 命中 historical audit path | `preserve-audit` | 交给 `CR058-PRESERVE-AUDIT-ALLOWLIST.md`。 |
| CLG-CR058-04 | 命中敏感路径 / 引用元数据 | `blocked_sensitive` 或 `excluded` | 不读取正文，不进入候选。 |
| CLG-CR058-05 | rollback_ref_required=true 但 rollback gate 未满足 | `execute_allowed=false` | 交给 `CR058-ROLLBACK-GATE.md`。 |

## 5. 后续验证入口

| 验证对象 | 入口 | 预期 |
|---|---|---|
| 字段完整性 | 本文 §2 / §3 | 每行 12 个字段完整。 |
| 非法 action | `rg -n "execute_move|execute_rewrite|rename_now|delete_now|bulk_replace_now" docs/release/CR058-*.md process/stories/CR058-BATCH-A-IMPLEMENTATION.md process/checks/CP6-CR058-BATCH-A-STATIC-NO-OP-CODING-DONE.md` | 只允许出现在“禁止值 / 禁止操作”语境，不得作为执行字段。 |
| no-op 边界 | `docs/release/CR058-NO-OP-GUARDRAILS.md` | 不授权项完整。 |

## 6. 未授权项

本文不授权真实 file move / rename / delete / rewrite，不授权 NAS / data lake / provider / QMT / MiniQMT / git remote / credential 操作，不恢复 CR046。
