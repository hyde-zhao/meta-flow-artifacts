---
title: "CR120 CR Tracking Formal/FU Row Convention Hardening"
cr_id: "CR-120"
status: "draft-for-cp2"
created_at: "2026-06-22T19:44:17+08:00"
updated_at: "2026-06-22T19:54:11+08:00"
owner: "host-orchestrator"
---

# CR120 CR Tracking Formal/FU Row Convention Hardening

## 目标

把最近 CR113-CR119 中反复人工维护的 CR tracking 行约定显式化，为后续 CP5 是否实施 checker / fixture hardening 提供范围边界。

## 候选约定范围

| 场景 | 期望约定 |
|---|---|
| follow-up candidate 启动为正式 CR | follow-up row 应写入 `formal_cr_path`，状态切到 active，blocked_by 指向 related_active_cr。 |
| 正式 CR active | `CR-INDEX.yaml.active_crs[]`、recent item、items 中应一致表达 lifecycle / readiness / gate。 |
| 正式 CR CP8 closed | active list 清空或移除对应 CR，formal CR 与 FU row 应改为 closed-current-delivery / closed。 |
| no-follow-up closure | FU row 应写入 `no_follow_up_created` 或等价 closure reason。 |
| mirror row | 上游 tracking 和 CR-INDEX 对同一 FU 的状态、formal path、gate、next_action 不应冲突。 |

## 非目标

- 不在 CP2 修改 `scripts/check_human_gate_decision_brief.py` 或其他 checker。
- 不新增 / 修改 tests。
- 不批量重写历史 `process/STATE.md`。
- 不启动 runtime、NAS、凭据、交易写、provider fetch、lake write 或 catalog publish。

## CP5 前置问题

| 问题 | 处理 |
|---|---|
| checker owner 是现有 human gate checker 还是独立 CR tracking checker？ | owner discovery 指向 `../meta-flow/meta_flow/checks/cr_tracking.py`；当前仓库 `scripts/check_cr_tracking_consistency.py` 是 legacy checker，不作为推荐实现目标。 |
| 是否只做 fixture 约束，还是修改正式 checker？ | CP5 推荐修改正式 `meta-flow check cr-tracking` checker 并新增 focused tests。 |
| 是否扫描历史 CR113-CR119 文件？ | 默认不批量扫描；只用目标 fixture 和显式文件检查。 |
| 如何处理已存在未提交 CR118 / CR119 checker diff？ | CP5 需验证 CR118 / CR119 目标测试仍通过，且不回滚既有改动。 |

## Owner Discovery

| owner 类型 | 路径 | 结论 |
|---|---|---|
| 推荐 checker owner | `../meta-flow/meta_flow/checks/cr_tracking.py` | `meta-flow check cr-tracking` 的实际实现入口，包含 `FollowUpRow`、`IndexItem`、formal CR / follow-up row 校验。 |
| 推荐测试 owner | `../meta-flow/tests/` | 适合新增 focused fixture tests 覆盖 formal CR / FU row convention。 |
| legacy checker | `scripts/check_cr_tracking_consistency.py` | 当前仓库 legacy consistency checker，不是 `meta-flow check cr-tracking` 的主实现入口；本 CR 默认不改。 |
| legacy tests | `tests/test_cr093_cr_tracking_consistency.py` | 旧规则回归测试；本 CR 默认不改，除非 CP5 改变 owner。 |

## 当前推荐

CP2 已批准 scope 与 staged authorization。下一步审查 CP5 最小实现候选：优先复用 `meta_flow.checks.cr_tracking` 的现有 parser / validator 风格，限定在明确 fixture，不做全量历史扫描，不触碰 runtime / NAS / credentials / trading / publish。
