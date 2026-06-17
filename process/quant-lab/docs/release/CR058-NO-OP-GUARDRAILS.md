---
status: "implemented-cp6-static"
version: "1.0"
change_id: "CR-058"
title: "CR058 No-op Guardrails"
created_by: "meta-dev"
created_at: "2026-06-14T15:40:00+08:00"
source_story: "process/stories/CR058-S05-mechanical-migration-dry-run-and-authorization-boundary.md"
execution_mode: "static-only-no-op"
---

# CR058 No-op Guardrails

## 1. 结论

CR058 Batch-A CP6 只交付静态工程资产和检查证据。所有真实操作保持未授权；任何真实动作必须另起 `runtime_authorization` 或 implementation 决策项，并重新经过门禁。

| 维度 | 当前结论 |
|---|---|
| 文件系统 | 不移动、不重命名、不删除、不批量替换。 |
| 数据面 | 不扫描 NAS、不读取 untracked data、不写 lake、不 publish catalog。 |
| 凭据 | 不读取 `.env`、token、password、cookie、session、private key、账户凭据。 |
| 交易运行 | 不启动 QMT / MiniQMT，不连接、不查询账户、不下单、不 simulation/live。 |
| Git 远端 | 不 push、不 tag、不 remote rename、不 history rewrite。 |
| CR046 | 不恢复、不调度 CP7、不触发 trading runtime verification。 |

## 2. 不授权项清单

| guardrail_id | 操作 | 状态 | 需要什么才能改变 |
|---|---|---|---|
| NA-CR058-001 | 真实 repo-local file move / rename / delete | not-authorized | 独立 runtime_authorization、candidate manifest、rollback_ref、执行窗口和用户确认。 |
| NA-CR058-002 | 批量路径替换 / reference rewrite | not-authorized | Candidate list + preserve-audit allowlist + rollback_ref + 用户授权。 |
| NA-CR058-003 | 改写历史 `process/**` / checks / checkpoints / handoffs / Story / DEV-LOG | not-authorized | 单项豁免 DQ、owner、rollback_ref 和风险接受。 |
| NA-CR058-004 | NAS mount / scan / mkdir / copy / delete / migration | not-authorized | 独立 NAS / CR060+ 授权。 |
| NA-CR058-005 | `MARKET_DATA_LAKE_ROOT` replacement 或 lake write | not-authorized | 独立 data lake migration CR、backup / restore / publish gate。 |
| NA-CR058-006 | provider fetch / catalog publish | not-authorized | 独立 runtime_authorization。 |
| NA-CR058-007 | `.env`、token、password、cookie、session、private key、账户凭据读取 | not-authorized | 独立 security / credential handling 决策；默认禁止。 |
| NA-CR058-008 | QMT / MiniQMT runtime、连接、账户查询、下单、撤单、simulation/live | not-authorized | 独立 trading runtime CR；不得从 CR058 继承。 |
| NA-CR058-009 | git push / tag / remote rename / history rewrite | not-authorized | 独立 git remote gate。 |
| NA-CR058-010 | `git reset --hard` 或 `git checkout --` 回退用户未明确要求的工作树 | not-authorized | 用户逐项明确要求且有回滚/备份策略。 |
| NA-CR058-011 | 从 CR053 继承真实迁移授权 | not-authorized | 必须重开相应 gate，不得静默继承。 |
| NA-CR058-012 | 恢复 CR046 CP7 或推进 CR046 runtime verification | not-authorized | 用户单独明确恢复 CR046。 |

## 3. 静态 guardrail 检查规则

| check_id | 检查项 | 通过条件 |
|---|---|---|
| NOG-CR058-001 | 所有 CR058 文档声明 `execution_mode=static-only-no-op` 或等价 no-op 结论。 | PASS。 |
| NOG-CR058-002 | Candidate list 中 `execute_allowed=false`。 | PASS。 |
| NOG-CR058-003 | Rollback gate 中缺 commit / bundle / manifest 时 BLOCKED。 | PASS。 |
| NOG-CR058-004 | Preserve-audit allowlist 覆盖至少 8 类历史证据。 | PASS。 |
| NOG-CR058-005 | 不授权项覆盖 12 类禁止操作。 | PASS。 |
| NOG-CR058-006 | 验证命令仅限静态文本 / markdown / diff check。 | PASS。 |

## 4. 允许的验证命令

下列命令只做静态检查，不触发真实迁移或运行时：

```bash
git diff --check -- docs/release/CR058-CANDIDATE-LIST.md docs/release/CR058-PRESERVE-AUDIT-ALLOWLIST.md docs/release/CR058-ROLLBACK-GATE.md docs/release/CR058-NO-OP-GUARDRAILS.md process/stories/CR058-BATCH-A-IMPLEMENTATION.md process/checks/CP6-CR058-BATCH-A-STATIC-NO-OP-CODING-DONE.md
```

```bash
rg -n "execute_allowed\\|real_move_authorized\\|bulk_rewrite_authorized\\|not-authorized\\|BLOCKED" docs/release/CR058-*.md process/stories/CR058-BATCH-A-IMPLEMENTATION.md process/checks/CP6-CR058-BATCH-A-STATIC-NO-OP-CODING-DONE.md
```

## 5. 不适用验证

| 验证 | N/A 原因 |
|---|---|
| pytest / runtime tests | 本轮不改 Python 代码和运行时行为。 |
| install dry-run | 本轮不改安装器、Agent、Skill 或平台路径。 |
| NAS / lake dry-run | 用户未授权真实路径、扫描、复制、写入或 publish。 |
| QMT / MiniQMT validation | CR058 不涉及交易 runtime；CR046 仍暂停。 |
| git bundle verify | 本轮不创建 bundle；rollback gate 记录为 BLOCKED。 |
