---
status: cp8-approved-git-delivery-completed
version: "1.1"
release_artifact_profile: full
release_decision: READY_WITH_RISK
deployment_status: not-authorized
created_at: "2026-07-19T01:47:00+08:00"
updated_at: "2026-07-19T09:29:15+08:00"
---

# CR-172 PATH-I Deploy Checklist

## 1. 发布前输入检查

| 输入 | 状态 | 证据路径 | 说明 |
|---|---|---|---|
| CP8 Preparation Context | PASS | `process/context/CP8-CR172-PATH-I-PREPARATION-CONTEXT.json` | 14 writes、claim ceiling、3 DQ 明确 |
| CP8 Release Context | PASS | `process/release/RELEASE-CONTEXT-CR172.yaml` | full profile；fact_diff 完整 |
| TEST-REPORT | PASS_WITH_RISK | `docs/quality/TEST-REPORT-CR172.md` | repository coverage PASS，runtime gap 显式 |
| REVIEW | READY_WITH_RISK | `docs/quality/REVIEW-CR172.md` | blocker=0，required risk=4；其中 checkpoint ledger retry identity 风险并入 DQ-001 |
| BLOCKER findings | 0 | five final CP7 results | 历史 findings 9/9 CLOSED |
| Waivers | 0 | CP7/CP8 results | 不用 waiver 放宽 runtime 边界 |

## 2. 发布候选快照

| 检查项 | 状态 | 证据 / 摘要 |
|---|---|---|
| 变更范围清楚 | PASS | PATH-I 4 contracts + S05 fixture/QAC + 14 CP8 docs |
| source repository 状态 | PASS | branch `master`；提交 `fc8b7c4e0c934f40d17ca1bcc0079f07d58b32bb` 已到达 `origin/master`；授权范围提交后 clean |
| artifact repository 状态 | PASS_WITH_SCOPE | branch `main`；quant-lab 内容提交 `562872610d17458bf6f37d1c6cb32a2377d10644` 已到达 `origin/main`；`process/quant-lab/**` clean；无关 `process/meta-flow/**` 本地修改未提交 |
| 缓存与临时文件 | PASS_WITH_SCOPE | CP7 禁用 cacheprovider，CP8 只写 14 个文档/JSON/YAML；未扫描 denied source/test trees |
| 敏感信息 | PASS_WITH_SCOPE | credential reads=0；CP8 文档只写 policy ID 与计数 |
| checkpoint ledger 全量校验 | RISK | retry event ID 受影响重复值=`10`、额外冲突行=`12`；最终 result refs 无歧义，但 `event check=FAIL`，当前不授权历史改写 |

CP8 审查时的双仓 dirty 风险已通过严格限定范围的成对提交/推送处置。这里只能声明 repository delivery 完成；仍不得据此声明 runtime `RELEASED`、真实 deployment 或可换机恢复已验证。

## 3. 安装 / 升级 / 幂等验证矩阵

| 平台 | 组件 | Scope | 场景 | 是否适用 | 验证命令 / 方法 | 结果 | N/A 原因 |
|---|---|---|---|---|---|---|---|
| Codex | installer/agents/skills/rules | project/user | fresh install dry-run | no | N/A | N/A | CR-172 不修改平台安装面 |
| Claude Code | installer/agents/skills/rules | project/user | upgrade/repeated/idempotency | no | N/A | N/A | CR-172 不修改平台安装面 |
| All | install/uninstall | project/user | rollback/uninstall | no | repository rollback plan | N/A | 无安装产物 |
| Repository | PATH-I contracts/tests | source + artifact | delivery snapshot | yes | workspace check/git-status + remote ref verification | PASS_WITH_RISK | scoped commit/push 已完成；见 `process/checks/CR172-PAIRED-GIT-DELIVERY.result.json`；runtime 风险不变 |

## 4. 平台、权限和能力边界

| Check ID | 检查项 | 状态 | 证据 / 说明 | 阻断等级 |
|---|---|---|---|---|
| DEP-001 | process route 符合 contract | PASS | symlink health=ok；project_name=quant-lab | BLOCKING |
| DEP-002 | 平台 installer path | N/A | 无 installer/platform change | N/A |
| DEP-003 | repository contract suite | PASS | S01-S05 combined 154/154 | BLOCKING |
| DEP-004 | 不覆盖用户本地配置 | PASS | config/install changes=0 | BLOCKING |
| DEP-005 | rollback plan 已准备 | PASS | `docs/release/ROLLBACK-CR172.md` | REQUIRED |
| DEP-006 | deployment authorization | NOT_AUTHORIZED | deploy/publish/live=0 | BLOCKING-FOR-EXECUTION |
| DEP-007 | remote delivery execution | PASS_SCOPED | quant-lab `fc8b7c4e...` 与 artifact 内容 `5628726...` 均到达授权远端 ref；排除 `process/meta-flow/**`；无 force push/tag | COMPLETE |
| DEP-008 | checkpoint retry event identity | PASS_WITH_RISK | `FU-CR172-LEDGER-001`；生成器修复与历史 migration 分离，当前不改写 ledger | REQUIRED-RISK-ACCEPTANCE |

## 5. 六类真实动作

| Action | Authorized | Executed | 部署判断 |
|---|---:|---:|---|
| data-lake read | 0 | 0 | not-authorized |
| multi-trial runtime | 0 | 0 | not-authorized |
| trial-return generation | 0 | 0 | not-authorized |
| empirical-R computation | 0 | 0 | not-authorized |
| NAS replica sync | 0 | 0 | not-authorized |
| execution materialization | 0 | 0 | not-authorized |

## 6. 发布结论

| 项目 | 内容 |
|---|---|
| release_artifact_profile | full |
| release_decision | READY_WITH_RISK |
| fact_diff | delivered 5；deferred 3；missing_required 0 |
| 阻断人工终验项 | 0 |
| 阻断真实 deployment 项 | runtime deployment 仍未获授权；Git repository delivery 是独立授权，不等于 deployment |
| 风险接受项 | DQ-CP8-CR172-001..003 已接受；DQ-001 同时覆盖双仓 dirty 与 checkpoint ledger retry ID 债务 |

## 7. 不授权项

| Item ID | 不授权操作 | 原因 | 需要的独立授权 |
|---|---|---|---|
| NA-CR172-001 | credential/secret/env 原文读取 | CP8 no-credential boundary | security/runtime gate |
| NA-CR172-002 | real lake/NAS/execution host read/write | 仅 repository contract verified | activation-resume CP2 + runtime authorization |
| NA-CR172-003 | multi-trial/trial-return/empirical-R | native producer/runtime absent | approved implementation/runtime prerequisite |
| NA-CR172-004 | signal/trading/deploy/publish/live | 不在 PATH-I slice | independent CR/gate |
| NA-CR172-005 | 超出 quant-lab source 与 artifact `process/quant-lab/**` 的 Git 写入 | 当前授权是严格 scoped | 新的显式范围授权 |

## 8. Host 人工门后检查

1. CP8 checkpoint 已回填 approved；不得把它解释为真实 deployment 授权。
2. CP8 result/ledger/state/status-sync 已收敛到 `closed/cp8_closed/READY_WITH_RISK` 与 `delivered`。
3. scoped 双仓 Git 交付已完成：只提交源码仓库 quant-lab 相关修改及 artifact `process/quant-lab/**`，并已用远端 ref 验证提交；`process/meta-flow/**` 未纳入。
4. 不关闭 activation，不恢复 PATH-C/A；future activation-resume CP2 继续是硬门。
5. 不对当前 append-only `CHECKPOINT-LEDGER.ndjson` 做原地去重；先完成 `FU-CR172-LEDGER-001` 生成器修复，再通过独立批准的 compaction/migration 处理历史。
