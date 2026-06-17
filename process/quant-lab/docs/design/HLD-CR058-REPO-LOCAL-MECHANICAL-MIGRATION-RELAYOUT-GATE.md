---
status: "draft-for-review"
version: "0.1"
change_id: "CR-058"
created_by: "host-orchestrator"
created_at: "2026-06-14T13:59:00+08:00"
source_cr: "process/changes/CR-058-REPO-LOCAL-MECHANICAL-MIGRATION-RELAYOUT-GATE-2026-06-14.md"
---

# HLD CR058 Repo-local Mechanical Migration / Relayout Gate

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 0.1 | 2026-06-14 | host-orchestrator | 新增 CR058 repo-local relayout gate HLD 草案，定义 no-op scope、candidate list、preserve-audit allowlist、rollback_ref 和授权边界。 |

## 1. 问题定义

CR053 已关闭静态 migration inventory / dry-run，明确 `READY_WITH_RISK` 不授权真实迁移。CR058 需要把 `FU-CR053-01` 转成正式、可审查的 repo-local mechanical migration / relayout gate：先冻结候选范围和禁止范围，再为未来可能的本地仓库内 rename / path rewrite / future-facing docs rewrite 建立前置门禁。

成功标准：

| 编号 | 标准 | 度量 |
|---|---|---|
| SC-CR058-01 | 正式 CR 与来源台账可追溯 | `FU-CR053-01`、`CR-058`、`CR-INDEX.yaml` 三处互链数量 = 3。 |
| SC-CR058-02 | 不授权边界完整 | 禁止项数量 = 10，覆盖 file move/rewrite、NAS、lake、git remote、credential、runtime、provider/publish。 |
| SC-CR058-03 | 候选清单门禁可验证 | 每个 candidate 具备 `path_pattern`、`classification`、`owner`、`action_class`、`risk`、`verification_rule`、`rollback_ref_required` 7 个字段。 |
| SC-CR058-04 | preserve-audit 保护可验证 | `process/**`、`process/checks/**`、`process/checkpoints/**`、`process/handoffs/**`、历史 Story、DEV-LOG、legacy checkpoints 至少 7 类进入 allowlist。 |
| SC-CR058-05 | rollback_ref 可操作 | `pre_cr058_commit`、`pre_cr058_git_bundle`、`pre_reference_rewrite_manifest` 3 个 ref 未满足时 fail-closed。 |

## 2. 范围

| 类别 | In Scope | Out of Scope |
|---|---|---|
| repo-local gate | 候选列表 schema、人工复核规则、preserve-audit allowlist、rollback_ref、dry-run report 设计 | 真实 `mv` / `rename` / `delete` / 批量替换 |
| 文档 rewrite | README / USER-MANUAL / future-facing docs 的候选分类 | 当前轮直接改 README / USER-MANUAL 正文 |
| 历史审计 | 明确 preserve-audit 分类和禁止改写规则 | 重写 `process/**` 历史记录、handoff、checkpoint、旧 CR |
| 安全边界 | 禁止凭据读取、敏感路径输入、交易/runtime/data lake 操作 | `.env` / token / account / password / cookie / private key 读取 |
| Git | 本地 rollback_ref 和 bundle 计划 | git push / tag / remote rename / history rewrite |

## 3. 推荐架构

```
CR053 static inventory
  -> CR058 intake conflict precheck
  -> candidate list gate
  -> preserve-audit allowlist gate
  -> rollback_ref gate
  -> dry-run / no-op migration design
  -> CP5 human approval
  -> future CP6 may only produce approved no-op artifacts unless separately authorized
```

模块职责：

| 模块 | 职责 | 输入 | 输出 | 降级 / 失败行为 |
|---|---|---|---|---|
| Intake & Conflict Gate | 确认 CR053 closed、CR046 paused、不继承真实授权 | STATE、CR-INDEX、CR053 follow-up | CR058 formal CR、冲突预检 | 影响面重叠则 blocked，等待用户选择合并 / 等待 / supersede。 |
| Candidate List Gate | 只列可审查候选，不执行动作 | CR053 inventory、path references | candidate list 合同 | 缺 owner / action / risk 时 manual-review，不得执行。 |
| Preserve-audit Gate | 保护历史证据不被机械改写 | PATH-REFERENCES-CR053 | allowlist / denylist | 历史证据命中 rewrite 时 fail-closed。 |
| Rollback Ref Gate | 真实执行前要求可回退证据 | MIGRATION-PLAN-CR053 | rollback_ref checklist | 缺 commit / bundle / manifest 时 fail-closed。 |
| No-op Authorization Gate | 区分设计确认与真实执行授权 | CP2/CP3/CP5 DQ | 不授权项和后续授权条件 | 用户只 approve 设计时，不得执行真实动作。 |

## 4. 关键流程

1. 读取 CR053 follow-up 与 release context，确认 `FU-CR053-01` 可转正式 CR。
2. 比较 CR046 paused active 与 CR058 影响面，记录 non-overlap；不调度 CR046 CP7。
3. 创建 CR058 formal CR，更新台账和索引。
4. CP2 冻结范围：repo-local gate 设计，不执行迁移。
5. CP3 确认架构：candidate list -> preserve-audit -> rollback_ref -> no-op authorization。
6. CP5 确认 Story batch：S01..S05 全部 technical-note，统一确认后才允许进入任何 CP6 文档实现。

## 5. 非功能设计

| 维度 | 设计 |
|---|---|
| Safety | 默认 fail-closed；任何真实动作均需独立授权。 |
| Auditability | 历史 process / checkpoint / handoff / Story 保留原语境。 |
| Reversibility | 真实执行前必须存在 commit、bundle、manifest 三类 rollback_ref。 |
| Minimality | 不新增 mover 脚本，不新增数据湖 / NAS 访问，不改运行时。 |
| Determinism | 候选分类使用 exact path family，不使用模糊匹配自动替换。 |

## 6. 风险

| 风险 ID | 风险 | 影响 | 缓解 |
|---|---|---|---|
| R-CR058-01 | 把 CP5 设计通过误读为真实 move 授权 | 可能破坏仓库路径或历史审计 | 所有 checkpoint 写明 no-op；真实动作另起授权。 |
| R-CR058-02 | 历史 `process/**` 被机械替换 | 审计链失真 | preserve-audit allowlist 必须先通过。 |
| R-CR058-03 | rollback_ref 缺失 | 真实迁移不可回退 | 缺 `pre_cr058_commit` / bundle / manifest 时 fail-closed。 |
| R-CR058-04 | CR046 被误恢复 | 触发未授权 QMT/MiniQMT 验证 | CR058 tracking 明确 CR046 paused non-overlap，不调度 CP7。 |

## 7. ADR 候选点

| ADR | 决策 | 推荐 |
|---|---|---|
| ADR-CR058-001 | CR058 是否继承 CR053 授权 | 不继承；CR053 只提供输入证据。 |
| ADR-CR058-002 | 历史证据处理 | preserve-audit 优先，不做 bulk rewrite。 |
| ADR-CR058-003 | rollback_ref 前置 | commit + bundle + manifest 三件套，缺一 fail-closed。 |
| ADR-CR058-004 | 当前实现形态 | 只做设计与门禁，不新增 mover 脚本。 |

