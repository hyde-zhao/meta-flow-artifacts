---
story_id: "ST-GB-002"
title: "只发布调用前已有提交并生成matching evidence"
story_slug: "committed-ref-publish"
lld_version: "1.1"
tier: "L"
status: "approved"
confirmed: true
created_by: "host-orchestrator-inline/meta-dev"
created_at: "2026-07-16"
confirmed_by: "user"
confirmed_at: "2026-07-16T15:56:00Z"
shared_fragments: []
feature_design_refs: ["process/docs/features/cr050-publish/DESIGN.md", "process/docs/features/cr050-publish/TEST-PLAN.md", "process/docs/features/cr050-publish/TASKS.md"]
lld_policy: {required_level: "full-lld", trigger_reasons: ["remote-write", "evidence", "security"], rationale: "published evidence controls later default writes."}
open_items: 0
---

# LLD: ST-GB-002 — Committed Ref Publish

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-16 | host-orchestrator-inline/meta-dev | 冻结captured HEAD、no-auto-commit、paired publish、matching evidence与resume。 |
| 1.1 | 2026-07-16 | host-orchestrator-inline/meta-dev | CP5独立评审精确化§2/3/6/7：固定CLI为`meta-flow cr branch-publish`，显式声明open后须由operator先提交bootstrap产物并满足clean-tree门。 |

## 0. 工程依据与上游设计依据

工程依据为CP3 R3 HLD/ADR、Feature Matrix、FEAT-GB-02 pack、ST-GB-001声明的types/runner contract，以及现有`push_workspace`兼容行为。publish evidence是merge预期身份输入，但fresh Git refs仍拥有当前事实。

## 1. 目标

创建显式publish handler，将调用入口捕获的2/2 local HEAD普通推送到remote CR branch，生成matching `PublishEvidence`；工具不选择文件、不stage/commit/amend、不触碰default。

## 2. 需求（Functional / Non-Functional）

- 2/2 fresh observe并验证clean、attached、expected branch/upstream和non-FF。
- Plan固定entry HEAD full OID；执行时HEAD变化不得被自动纳入。
- 全仓preflight失败writes=0；后仓失败PARTIAL；resume fresh observe。
- dry-run plan/result确定，mutation=0；add/commit/amend/force次数=0。
- 操作前置：`branch-open`生成的bootstrap过程文件必须先由operator审查并提交；publish只接受调用入口已存在的committed full OID和clean tree，绝不替operator提交。

## 3. 模块拆分与职责

| 模块 | 职责 |
|---|---|
| lifecycle types | 增加PublishEvidence与publish-specific error codes |
| publish planner | 捕获entry OID、检查identity/upstream/remote ancestry |
| paired executor | ordinary exact CR ref push与post-check |
| result adapter | schema/hash/correlation，供merge读取expected identity |
| CLI/tests | `meta-flow cr branch-publish`独立入口、bare/spy/fault fixtures |

## 4. 代码结构与文件影响范围

修改`git_branch_lifecycle.py`、`git_sync.py`、`cli.py`和`tests/test_git_branch_lifecycle.py`；不新建第二结果数据库，不修改`cr_lifecycle.py` writer语义，不执行任何自动commit或default写。

## 5. 数据模型与持久化设计

`PublishEvidence`字段：`attempt_id/cr_id/repo/branch/entry_local_oid/remote_before_oid/remote_after_oid/terminal/observed_at/result_ref`。`matching`要求CR/repo/branch一致、terminal PASS|NO_CHANGE、remote_after=entry_local_oid。result append-only；后续HEAD变化只形成新attempt。

## 6. API / Interface设计

| 接口 | 输入 | 输出 | 调用方 | 失败 |
|---|---|---|---|---|
| `plan_publish(intent,snapshots)` | open contract+fresh refs | Plan | CLI | dirty/wrong_upstream/non_ff |
| `execute_publish(plan,runner)` | frozen OIDs | Attempt+Evidence[] | Host | PARTIAL/post_verify_failed |
| `validate_publish_evidence(ref,fresh)` | result+fresh CR refs | matching identity | merge planner | evidence_mismatch/ref_drift |
| `meta-flow cr branch-publish` | CLI参数+publication authz | attempt+PublishEvidence | operator/Host | dirty/uncommitted时BLOCKED且writes=0 |

## 7. 核心处理流程

1. 验证operation=publish、CR/branch/route和publication authz；若open留下未提交bootstrap产物，提示operator先审查并commit，工具本身不执行stage/commit。
2. 捕获2/2 entry HEAD，检查clean/branch/upstream；query remote CR并做ancestor判定。
3. preflight全部通过才执行普通exact branch push；不得运行add/commit/amend。
4. 每仓post-query必须等于entry OID；否则PARTIAL/FAILED。
5. result写完整per-repo terminal；resume时已成功仓fresh exact match才NO_CHANGE。

## 8. 技术细节

push refspec使用full captured OID到`refs/heads/<cr-branch>`，不依赖执行时HEAD别名。remote不存在视为create eligible；存在且非ancestor阻断。命令和结果stderr统一redact。旧result只是关联证据，不允许覆盖fresh remote drift。

## 9. 安全与性能设计

argv-only、ref validation、authz最小化；publication authz不能写default/delete。每仓probe和push数量有界。command spy扫描`add|commit|amend|merge|rebase|--force`并要求0。

## 10. 测试设计

| 场景 | 预期 | 方式 |
|---|---|---|
| clean 2/2 publish | remote CR=entry HEAD | bare |
| dirty/wrong branch/upstream/non-FF | BLOCKED,writes=0 | negative |
| HEAD在plan后变化 | 只推entry OID或post-check拒绝 | controlled runner |
| artifact/project后仓失败 | PARTIAL+resume | fault |
| dry-run/no-auto-commit | mutation=0, forbidden argv=0 | spy |
| matching evidence旧/错CR/错repo | merge validation拒绝 | unit |

## 11. 实施步骤

| TASK-ID | 动作 | 文件 | 描述 | 测试 |
|---|---|---|---|---|
| TASK-GB-002-01 | 修改 | lifecycle | publish plan/evidence/validation | unit |
| TASK-GB-002-02 | 修改 | lifecycle/git_sync/cli | exact branch push和paired outcomes | bare |
| TASK-GB-002-03 | 修改 | tests | negative/HEAD drift/partial/spy | pytest |

## 12. 风险、难点与预研建议

clarification=0。风险为HEAD竞态、remote并发和用户误以为publish会commit。以captured full OID、non-FF/post-check和明确CLI文案缓解；若需commit功能另立CR，不在实现中扩张。

## 13. 回滚与发布策略

新handler可独立移除而不影响existing `workspace push`。schema接线失败时保留tests并回滚CLI exposure；不删除远端成功事实。真实remote publish需操作级授权，CP7使用临时bare remote。

## 14. DoD

- [ ] 工程依据/目标/需求/模块拆分/代码结构/数据模型/API/流程/技术细节/安全/测试/实施/风险/DoD完整。
- [ ] SC-GB-02/07、TC-GB-004/005/009/010覆盖。
- [ ] remote after=entry OID覆盖2/2；forbidden command=0。
- [ ] matching evidence负例全部拒绝。
- [x] CP5批准后confirmed=true；仍须等待ST-GB-001通过依赖门。

## 人工确认区

- 结论：approved
- 审查人：user
- 风险接受：真实remote与独立QA未验证。
