---
status: draft
version: "1.0"
feature_id: "FEAT-GB-03"
feature_name: "Merge Proof and Cleanup"
source_blueprint: "process/docs/design/CR050-GIT-BRANCH-BLUEPRINT.md"
source_hld: "process/docs/design/CR050-GIT-BRANCH-HLD.md"
source_adr: "process/docs/design/CR050-GIT-BRANCH-ARCHITECTURE-DECISION.md"
source_matrix: "process/docs/design/CR050-FEATURE-DESIGN-MATRIX.md"
related_stories: ["ST-GB-003"]
lld_policy_summary: "1 full-lld"
---

# Feature Design: Merge Proof and Cleanup

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-16 | host-orchestrator-inline/meta-se | 冻结current 2/2 merge入口、fresh reproof、recovery refs和安全删除顺序。 |

## 摘要、背景与边界

来源UC-GB-003、REQ-GB-007..010/014、TC-GB-006..011/015/017。Feature负责在current matching 2/2 merge PASS基础上重新fetch/observe并证明identity、exact tip、protected policy与ancestry，建立local-only recovery refs，然后先远端、后本地清理。它不执行merge、不把CR closed/branch absent/旧result当proof、不force-delete、不用patch similarity。

## 模块、数据与接口

| 模块 | 负责 | 输出 | 失败 |
|---|---|---|---|
| finish gate | 验证current projection+独立delete authz | eligible intent | PARTIAL/旧PASS/authz失配→BLOCKED |
| proof builder | fresh refs、known tip、ancestry、protected | `MergeProof[repo]` | unknown/drift/non-ancestor→BLOCKED |
| recovery manager | local namespaced exact OID ref | `RecoveryRef` | 同名不同OID→BLOCKED |
| cleanup executor | remote CR delete→verify absent→local `branch -d` | Attempt | 中途失败PARTIAL并保留恢复面 |

接口`plan_finish(intent,current_projection,snapshots,authz)->Plan`无写；`execute_finish(plan)->Attempt`按2/2 proof/recovery/remote/local顺序；result校验后才供CR close消费。delete authz与merge authz不可继承。

## 核心流程、失败与幂等

1. 要求current matching 2/2 projection PASS和独立delete authz。
2. fresh fetch/query两仓CR/default；解析known published tip并验证branch identity/protected rule。
3. positive proof仅允许known tip等于fresh default或是其祖先；squash/rebase/unknown fail closed。
4. 2/2 proof全过后创建/reuse`refs/meta-flow/recovery/<cr>/<fingerprint>`；不同OID collision阻断。
5. 按确定顺序删除2/2 remote CR refs并验证absent；两仓remote完成前local delete=0。
6. 最后`git branch -d`本地CR branch；已absent可NO_CHANGE但不能跳过known-tip proof。

## 权限、安全、测试与下游

remote delete只使用exact ref deletion且不force；本地仅`branch -d`。argv-only、redaction、protected/default拒绝。bare tests覆盖正向、PARTIAL入口、old PASS漂移、non-ancestor、recovery collision、remote partial、idempotent absent和dry-run。command spy确保`branch -D`/force/reset=0。CR close只能消费validated finish result，不反向证明Git事实。

## 实现顺序与风险

先finish eligibility/proof types，再recovery manager，再cleanup executor/CLI/CR close gate，最后failure/idempotence tests。Git ancestry无法证明squash/rebase是接受的fail-closed边界；需要trusted forge receipt时另立adapter CR，不能在本Feature猜测。
