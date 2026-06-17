# CP5-CR062 Repository Publication Readiness 人工检查点

## 自动预检摘要

自动预检结论：PASS_WITH_RISK。CR062 的 GitHub target candidate、publication scope、scan plan、remote conflict policy、branch policy、rollback / fail-closed strategy 已冻结。用户回复 `approve` 后，表示接受 CP2 / CP3 / CP5 推荐决策，并授权在门禁批准后按 CP5 DQ 执行受控 Git publication：read-only preflight、扫描、manifest 冻结、普通 commit、remote add（仅 absent 且 exact target）、push 当前分支。仍不授权 tag、branch rename、history rewrite、force push、remote deletion、NAS/data lake/provider/catalog、凭据、QMT/MiniQMT runtime、CR046 recovery、physical move 或 bulk import rewrite。

## Decision Brief

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP5-CR062-REPOSITORY-PUBLICATION-CONTEXT.yaml` |
| capsule 状态 | ready |
| read_profile | compact |
| 默认读取策略 | 先读 capsule；执行前再读 Git status / manifest / scan result |
| 全文档读取扩展 | 已读取 CR062 manifest、scope、scan plan、rollback ref |
| 缺失 / waived 理由 | N/A |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `process/STATE.md.human_gate_decisions.pending_human_decisions[]` | scanned | 0 | 0 | 当前无未批准 CR062 DQ。 |
| 自动预检结果 | `process/checks/CP5-CR062-REPOSITORY-PUBLICATION-READINESS.md` | scanned | 6 | 5 | readiness 风险转 DQ。 |
| publication manifest | `docs/release/REPOSITORY-PUBLICATION-MANIFEST-CR062.yaml` | scanned | 14 | 5 | post-approval 允许 / 禁止动作转 DQ。 |
| scan plan | `docs/release/PUBLICATION-SCAN-PLAN-CR062.md` | scanned | 8 | 2 | fail-closed 和 forbidden path 合并。 |
| rollback ref | `docs/release/ROLLBACK-REF-CR062.md` | scanned | 7 | 2 | no rewrite / forward fix 合并。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP5-CR062-01 | runtime_authorization | 是否批准 CR062 post-approval Git publication 执行范围？ | 批准：preflight、scan、manifest、normal commit、remote add if absent/exact target、push current branch。 | 备选 A：只批准 preflight/scan；备选 B：只提交不 push；备选 C：reject 并暂停。 | 推荐方案完成 GitHub baseline；只 preflight 更保守；只 commit 不完成远端目标。 | 会写 Git config / Git commit / GitHub remote，但仅在 approve 后执行。 | 任一 stop condition 命中则 BLOCKED，不继续。 |
| DQ-CP5-CR062-02 | implementation | commit manifest 是否采用 allowlist / denylist 冻结？ | 采用 post-approval `git status` + allowlist / denylist 冻结，只 stage approved paths。 | 备选 A：`git add .`；备选 B：code-only。 | 推荐控制风险；`git add .` 不可接受；code-only 牺牲审计。 | manifest 冻结会增加一步审查。 | forbidden path 出现时 BLOCKED。 |
| DQ-CP5-CR062-03 | security | secret/data/large/path scan 是否作为 commit/push 前硬门？ | 是；任一 BLOCKING/HIGH finding 阻断 commit/push。 | 备选 A：只记录风险继续；备选 B：只 secret scan。 | 推荐更安全；继续发布可能泄露敏感或大数据；只 secret scan 不够。 | 可能需要返工清理。 | 用户单项豁免需新 DQ。 |
| DQ-CP5-CR062-04 | risk_acceptance | 是否接受 CR061 full pytest 未运行，CR062 仅要求 publication scoped regression？ | 接受；post-approval 至少重跑 CR061 import smoke 和 `git diff --check`，full pytest 作为可选扩大项。 | 备选 A：push 前必须 full pytest；备选 B：只做 scan 不跑测试。 | 推荐覆盖 publication 关键风险且不扩大外部面；full pytest 更强但耗时；只 scan 不验证 import。 | 存在非 import 行为回归残余风险。 | 用户要求 full pytest 时先做命令风险扫描。 |
| DQ-CP5-CR062-05 | risk_acceptance | 失败或远端冲突时的回滚策略是否 fail-closed / forward-fix？ | 是；不 history rewrite、不 force push、不 remote deletion，失败后保留证据并停止。 | 备选 A：允许 force push 修正；备选 B：自动 set-url/删除 remote。 | 推荐保护历史；force / deletion 风险高。 | 远端部分写入时可能需要后续修复 CR。 | push 失败或远端冲突时 BLOCKED。 |

不授权项：

| Item ID | 不授权操作 |
|---|---|
| NA-CP5-CR062-001 | CP5 approve 前不执行任何 Git 写动作。 |
| NA-CP5-CR062-002 | 不执行 `git tag`。 |
| NA-CP5-CR062-003 | 不执行 branch rename / default branch rename。 |
| NA-CP5-CR062-004 | 不执行 history rewrite / rebase-publication / force push。 |
| NA-CP5-CR062-005 | 不删除 remote 或远端 ref。 |
| NA-CP5-CR062-006 | 不使用 `git add .` 或提交 forbidden paths。 |
| NA-CP5-CR062-007 | 不读取 `.env`、token、password、cookie、session、private key、账户或交易事实。 |
| NA-CP5-CR062-008 | 不执行 NAS 数据 copy / move / delete / archive promote。 |
| NA-CP5-CR062-009 | 不执行 `MARKET_DATA_LAKE_ROOT` replacement、provider fetch、lake write、catalog publish。 |
| NA-CP5-CR062-010 | 不启动 QMT / MiniQMT runtime、账户查询、下单、撤单、simulation/live。 |
| NA-CP5-CR062-011 | 不恢复或推进 CR046 CP7。 |
| NA-CP5-CR062-012 | 不执行 physical move / rename / delete 或 bulk import rewrite。 |

auto_final_authorization: false

## Entry Criteria

| 条件 | 结果 |
|---|---|
| CR062 正式 CR 已创建 | PASS |
| CP2 / CP3 gate docs 已创建 | PASS |
| publication manifest 已冻结 | PASS |
| scan plan 已创建 | PASS |
| rollback refs 已记录 | PASS_WITH_RISK |

## Checklist

| 检查项 | 结果 |
|---|---|
| 执行范围最小且明确 | PASS |
| post-approval 授权边界明确 | PASS |
| 决策项完整 | PASS |
| 不授权项完整 | PASS |
| 回滚策略存在 | PASS_WITH_RISK |
| 用户授权 | PASS |

## Exit Criteria

| 条件 | 结果 |
|---|---|
| 用户回复 `approve` 后才允许按 CP5 DQ 执行受控 Git publication | PASS |
| 用户回复 `修改: <具体修改点>` 后返工 | N/A |
| 用户回复 `reject` 后停止 CR062 | N/A |

## Deliverables

- `docs/release/REPOSITORY-PUBLICATION-MANIFEST-CR062.yaml`
- `docs/release/GIT-PUBLICATION-SCOPE-CR062.md`
- `docs/release/PUBLICATION-SCAN-PLAN-CR062.md`
- `docs/release/ROLLBACK-REF-CR062.md`
- `process/checks/CP5-CR062-REPOSITORY-PUBLICATION-READINESS.md`
- `process/checks/CP5-CR062-HUMAN-GATE-LAUNCH-MESSAGE.md`
- `process/context/CP5-CR062-REPOSITORY-PUBLICATION-CONTEXT.yaml`

## 人工审查结果

- 结论：approved
- 审查人：user
- 审查时间：2026-06-15T09:59:22+08:00
- 修改意见：用户回复“同意”，按 `approve` 解析，接受 DQ-CP5-CR062-01..05 的推荐方案。
- 风险接受项：授权在门禁批准后执行 CR062 受控 Git publication：read-only preflight、secret/data/large/path scan、manifest 冻结、普通 commit、remote add（仅当 remote absent 且 exact target）、push current branch。仍不授权 tag、branch rename、history rewrite、force push、remote deletion、NAS/data lake/provider/catalog/runtime、凭据或 `.env*` 读取、QMT/MiniQMT runtime、CR046 recovery、physical move 或 bulk import rewrite。
