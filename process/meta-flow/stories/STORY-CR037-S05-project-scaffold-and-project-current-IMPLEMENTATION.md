---
story_id: "CR037-S05"
cr_id: "CR-037"
title: "project scaffold and PROJECT.current"
stage: "CP6"
status: "implemented"
owner: "meta-dev"
created_at: "2026-07-03T00:00:00+08:00"
context_ref: "process/context/stories/CR037-S05.CP6.work-packet.json"
return_packet_ref: "process/returns/CR037-S05.CP6.return.json"
evidence_ref: "process/evidence/CR037-S05.CP6.index.json"
---

# CR037-S05 Implementation

## 实现前置检查

| 条目 | 结论 | 证据 |
|---|---|---|
| Capsule first | PASS | 已先读 `process/context/stories/CR037-S05.CP6.work-packet.json`，再读 `process/handoffs/CR037-S05-CP6-IMPLEMENTATION-HANDOFF.md`。 |
| Story 状态与设计证据 | PASS | Story 为 `dev-ready`；S05 LLD 为 full-lld，CP6 packet 授权实现。 |
| 依赖门控 | PASS | S01/S02 CP7 为 `PASS_WITH_RISK`，携带风险 `R-CR037-SECOND-MECHANISM`、`R-CR037-REGISTRY-DRIFT`。 |
| 写入范围 | PASS | 只写 S05 handoff Allowed Writes 内路径；未写 `process/quant-lab/**`、`docs/**`、`delivery/**` 或 credential-like 路径。 |
| 授权边界 | PASS | 未执行 runtime、production write、publish、live、真实外部调用、凭据读取、quant-lab 发布库写入或真实 ledger apply。 |

## 实现对象清单

| 对象 | 路径 | 实现内容 |
|---|---|---|
| project module namespace | `meta_flow/project/__init__.py` | 暴露 project scaffold/checker API。 |
| PROJECT.current checker | `meta_flow/project/state.py` | 新增 allowlist schema、16KB budget、forbidden fields、relative refs、CLI check 入口。 |
| project scaffold service | `meta_flow/project/scaffold.py` | 新增 dry-run plan、apply、冲突不覆盖、baseline payload、受控 current-state ref 写入。 |
| workspace routing | `meta_flow/workspace/routing.py` | 将 `project` 加入 `PROCESS_SCAFFOLD_DIRS`，使 bootstrap/link 覆盖 `process/project/`。 |
| current-state contract | `meta_flow/state/current.py` | 增加 `project_state_ref` 相对路径约束和断链检查；写入仍通过 `update_current_state()`。 |
| CLI | `meta_flow/cli.py` | 新增 `meta-flow project scaffold` / `meta-flow project check` 顶层入口。 |
| runtime baseline | `process/project/PROJECT.current.json` | scaffold apply 创建最小 refs-only project current。 |
| tests | `tests/test_cr037_project_current.py` | 覆盖 schema、budget、forbidden fields、dry-run/apply/noop/conflict、current ref contract、CLI。 |

## 设计契约映射

| LLD / Feature 契约 | 实现位置 | 验证 |
|---|---|---|
| `process/project/` 被 workspace bootstrap/check 覆盖 | `PROCESS_SCAFFOLD_DIRS` 新增 `project`；`workspace check` 仍通过。 | `pytest` + `meta-flow workspace check --project-root .` |
| `PROJECT.current.json` allowlist schema | `PROJECT_CURRENT_ALLOWED_KEYS` / `PROJECT_CURRENT_REQUIRED_KEYS`。 | `tests/test_cr037_project_current.py` |
| 16KB budget | `PROJECT_CURRENT_MAX_BYTES = 16 * 1024`，按文件 UTF-8 bytes 检查。 | over-budget unit case |
| forbidden fields / credential-like keys | `PROJECT_CURRENT_FORBIDDEN_KEYS` 与 key part scan。 | forbidden/security unit cases |
| refs-only relative refs | `validate_project_ref()` 拒绝绝对路径、`..`、`process/quant-lab/**`。 | relative-ref negative tests |
| dry-run no-write | `build_project_scaffold_plan()` 无副作用，CLI 默认 dry-run。 | dry-run test |
| apply 不覆盖冲突文件 | existing invalid/different `PROJECT.current.json` 产生 `conflict`，`apply_project_scaffold()` 中止。 | conflict test |
| apply 创建最小 refs-only baseline | `build_project_current_payload()` 只写 id/name/updated_at + 空 refs。 | apply test + project check |
| current state 只保存 `project_state_ref` | `apply_project_scaffold()` 调用 `current.update_current_state({"project_state_ref": ...})`；未知 project fields 被 writer 拒绝。 | contract tests + `state check --mode enforce` |

## 单元测试与 Fixture 计划

| 测试层级 | 覆盖项 | 状态 |
|---|---|---|
| Unit | valid/unknown/forbidden/over-budget/ref-path PROJECT.current checker。 | PASS |
| Integration | scaffold dry-run/apply/noop/conflict，CLI scaffold/check。 | PASS |
| Contract | current-state writer 拒绝 embedded project fields；断链 ref 在 enforce check 失败。 | PASS |
| Regression | `tests/test_state_v2.py`、`tests/test_cr_lifecycle.py`。 | PASS |

## 最小实现切片

| Slice | 改动 | 局部验证 |
|---|---|---|
| S05-1 Schema/checker | 新增 `meta_flow/project/state.py`。 | S05 checker 单测 PASS。 |
| S05-2 Scaffold/apply | 新增 `meta_flow/project/scaffold.py` 并创建 runtime baseline。 | dry-run/apply/noop/conflict 单测 PASS；实际 apply PASS。 |
| S05-3 Current/CLI/workspace integration | 更新 `routing.py`、`current.py`、`cli.py`。 | `project check`、`workspace check`、`state check --mode enforce` PASS。 |
| S05-4 Evidence | 写 return/evidence/CP6 result/summary。 | return/evidence/result checks PASS。 |

## 平台差异处理

N/A。S05 不涉及 Claude / Codex / OpenClaw 平台安装结构或 agent/skill frontmatter。路径约束使用项目相对路径，避免设备相关绝对路径进入 project/current state。

## 验证结果

| 命令 | 结果 | 摘要 |
|---|---|---|
| `PYTHONDONTWRITEBYTECODE=1 PYTEST_ADDOPTS='-p no:cacheprovider' uv run --python 3.11 pytest -q tests/test_cr037_project_current.py tests/test_state_v2.py tests/test_cr_lifecycle.py` | PASS | `42 passed, 15 subtests passed`。 |
| `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 meta-flow state check --project-root . --mode enforce` | PASS | `State v2 Check: OK`。 |
| `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 meta-flow context check-story-packet --packet process/context/stories/CR037-S05.CP6.work-packet.json --project-root .` | PASS | `Story Context Packet Check: OK`。 |
| `git diff -- process/quant-lab docs` | PASS | 无输出。 |
| `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 meta-flow project check --project-root .` | PASS | `Project Current Check: OK`。 |
| `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 meta-flow workspace check --project-root .` | PASS | `process_link_health: ok`；artifact repo dirty 为并行工作区状态。 |

## 未覆盖项

- 未创建 `PROJECT-SCALE.yaml`、`ROADMAP.yaml`、`MILESTONES.yaml`；这些属于后续 Story。S05 baseline 中只保留空 refs 列表，不写未来对象 refs，避免断链。
- 未执行真实 publish、runtime、production write、外部调用或 ledger apply；按 handoff 明确不授权。

## 设计缺口反馈

- 无需设计 delta。S05 按 LLD 推荐命令 `meta-flow project scaffold/check` 实现；CP5 OPEN `O-S05-01` 默认推荐方案已落地。
- 后续 S06 若写入 `scale_ref` / `roadmap_ref` / `milestones_ref`，应在目标对象存在后再更新 PROJECT.current，避免 `--require-ref-targets` 模式下断链。

## 后续交接

- 交给 meta-qa 的入口：`process/returns/CR037-S05.CP6.return.json`、`process/evidence/CR037-S05.CP6.index.json`、`process/checks/CP6-CR037-S05-CODING-DONE.result.json`。
- 重点复核：refs-only schema 是否足够窄；conflict no-overwrite 是否满足数据保护；`STATE.current.json` 是否仅新增 `project_state_ref` 而未嵌入 project governance 字段。
