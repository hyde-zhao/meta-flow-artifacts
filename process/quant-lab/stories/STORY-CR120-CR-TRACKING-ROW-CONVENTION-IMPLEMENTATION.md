---
story_id: "STORY-CR120-CR-TRACKING-ROW-CONVENTION"
cr_id: "CR-120"
status: "implemented"
created_at: "2026-06-22T20:06:33+08:00"
updated_at: "2026-06-22T20:06:33+08:00"
owner: "host-orchestrator"
---

# STORY CR120 CR Tracking Row Convention Implementation

## 实现前置检查

| 条目 | 结果 | 证据 | 说明 |
|---|---|---|---|
| CP2 approved | PASS | `process/checkpoints/CP2-CR120-CR-TRACKING-ROW-CONVENTION-REVIEW.md` | 用户确认将 `FU-CR114-004` 升格为 CR120。 |
| CP5 approved | PASS | `process/checkpoints/CP5-CR120-CR-TRACKING-ROW-CONVENTION-IMPLEMENTATION-REVIEW.md` | 用户授权最小 source / tests / checker implementation。 |
| owner 范围 | PASS | CP5 Decision Brief | 仅修改外部 meta-flow checker 与 focused tests。 |
| legacy checker 边界 | PASS | DQ-CP5-CR120-04 | 不修改当前仓库 `scripts/check_cr_tracking_consistency.py`。 |
| runtime 边界 | PASS | DQ-CP5-CR120-03 | 未授权 runtime、NAS、凭据、交易写、provider fetch、lake write 或 catalog publish。 |
| dirty diff 风险 | PASS | DQ-CP5-CR120-05 | 接受叠加 CR118 / CR119 当前 diff；验证覆盖相关目标测试。 |

## 实现对象清单

| 对象 | 路径 | 动作 | 验证 |
|---|---|---|---|
| meta-flow CR tracking checker | `/home/hyde/workspace/meta-flow/meta_flow/checks/cr_tracking.py` | 增加 formal CR / follow-up row 关系解析与 active/finished convention 检查 | focused pytest + `meta-flow check cr-tracking` |
| focused tests | `/home/hyde/workspace/meta-flow/tests/test_cr120_cr_tracking_row_convention.py` | 新增 active source row、缺失 `related_active_cr`、错误 formal link、finished formal close 四类 fixture | focused pytest |
| CR120 process evidence | `process/stories/STORY-CR120-CR-TRACKING-ROW-CONVENTION-IMPLEMENTATION.md` 等 | 固化 CP6 / CP7 / CP8 证据和人工门禁输入 | CP8 precheck |

## 设计契约映射

| 契约 | 实现位置 | 验证 |
|---|---|---|
| formal CR 可声明 `source_follow_up_id`，checker 必须找回 source FU row | `FormalCR.source_follow_up_id` 与 `discover_formal_crs` | `test_source_follow_up_row_must_point_to_formal_cr` |
| active source FU row 必须指向 active formal CR | `formal_cr_for_follow_up_row` / `formal_row_points_to_cr` | `test_valid_active_source_follow_up_row_passes` |
| active source FU row 必须包含 `related_active_cr=<CR-ID>` | `is_formal_active` validation | `test_active_source_follow_up_row_requires_related_active_cr` |
| formal CR finished 后 source FU row 必须 closed / closed-current-delivery | `is_formal_finished` validation | `test_source_follow_up_row_must_close_when_formal_cr_finished` |
| scalar `blocked_by` 和 Markdown 表格 `关联` / `相关 active CR` 均可被解析 | `relation_text_from_cells` / `normalize_header` | focused fixture + 当前 CR tracking 自检 |

## 验证结果

| 命令 | 结果 | 说明 |
|---|---|---|
| `uv run --python 3.11 pytest -q tests/test_cr120_cr_tracking_row_convention.py` | PASS | 外部 meta-flow focused tests：4 passed。 |
| `uv run --python 3.11 python -m py_compile /home/hyde/workspace/meta-flow/meta_flow/checks/cr_tracking.py /home/hyde/workspace/meta-flow/tests/test_cr120_cr_tracking_row_convention.py` | PASS | 语法检查通过。 |
| `uv run --python 3.11 meta-flow check cr-tracking --project-root .` | PASS | 当前 quant-lab CR tracking 自检通过。 |
| `PYTHONDONTWRITEBYTECODE=1 PYTEST_ADDOPTS='-p no:cacheprovider' uv run --python 3.11 pytest -q tests/test_cr118_human_gate_path_alias_checker.py tests/test_cr119_human_gate_launch_message_checker.py` | PASS | CR118 / CR119 目标测试保持通过。 |

## 不授权边界

- 未修改当前仓库 legacy checker `scripts/check_cr_tracking_consistency.py`。
- 未批量重写历史 `process/STATE.md`。
- 未启动 runtime。
- 未访问 NAS。
- 未读取凭据、账户、资金、持仓、委托、成交或 raw log。
- 未执行 submit / cancel / buy / sell、simulation / live、provider fetch、lake write、catalog publish。

## 交接摘要

CR120 已完成最小 checker hardening。后续 CP8 只需确认是否关闭为 READY、是否不新增 follow-up，以及是否继续保持 runtime / NAS / 凭据 / 交易写 / publish 不授权。
