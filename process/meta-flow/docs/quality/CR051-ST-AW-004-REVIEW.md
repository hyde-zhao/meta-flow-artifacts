---
project_id: "meta-flow"
change_id: "CR-051"
story_id: "ST-AW-004"
stage: "CP7"
report_type: "independent-review"
decision: "PASS_WITH_RISK"
reviewed_at: "2026-07-18T14:33:23Z"
reviewed_by: "meta-qa-critical (qa-yan)"
---

# CR-051 / ST-AW-004 独立质量评审

## Findings（按严重度）

### BLOCKER / HIGH / MEDIUM

无。

### LOW

#### QR-AW004-L01 — selector writer crash 可遗留 lock directory

| 字段 | 内容 |
|---|---|
| 位置 | `meta_flow/workflow/artifact_aggregate.py`：`FileAggregateStore.compare_and_set_current` |
| 状态 | OPEN / NON-BLOCKING |
| 影响 | 进程在获取 lock directory 后崩溃，后续 writer 会 fail closed，需人工诊断；不会发生 last-write-wins，但可用性下降 |
| 证据 | 50 次有限重试后返回 conflict；CP6 风险 `R-AW004-LOCK-CRASH` |
| 建议 | CP8 披露；后续以 native crash fixture 和受控恢复手册验证，不应在本 Story 中自动清锁 |
| Owner | CP8 decision owner |

#### QR-AW004-L02 — native Windows 跨进程锁行为未验证

| 字段 | 内容 |
|---|---|
| 位置 | File store selector lock / atomic replace 平台行为 |
| 状态 | OPEN / NON-BLOCKING |
| 影响 | 当前只能证明 Linux fixture；不能宣称 native Windows crash/rename 语义已覆盖 |
| 证据 | CP6 skipped + 本轮验证环境 Linux |
| 建议 | 在 native Windows 执行同 payload、冲突 payload、crash-after-lock、retry/recovery fixture |
| Owner | CP8 decision owner |

### INFO

#### QR-AW004-I01 — 真实 remote/worktree 与 workflow eval 均不在本轮执行范围

Story packet 明确 `NO_RUNTIME` / `NO_REPOSITORY_PUBLICATION` 且 `workflow_eval_required=false`。这不是实现缺陷；但 CP8 不得把纯 aggregate gate 的通过解释为真实远端、worktree 或发布能力已获授权/验证。

## 关键代码与契约审查

### Published handle fail-closed

- required-set 在 reader 调用前检查 source/artifact 恰好各一次。
- handle schema、correlation、mode、receipt/ref/digest/key 校验不通过时不产生 `ValidatedLegSet`。
- payload 必须从 `result_ref` 重读；embedded payload 不被消费。
- 显式 `published=false` 产生 `unpublished-result`，无法进入 compute/persist/project。
- invalid coordinate path 的 store result_count=0、projector calls=0。

### Canonical digest

- `canonical_json_digest` 的 omit 只作用于传入对象的顶层 dict，不递归删除同名键。
- `compute_aggregate` 在 `result.to_dict()` 顶层 omit 自身 `payload_digest` 后生成 digest。
- nested `published_handle_refs.*.payload_digest` 保持 canonical binding；定向篡改回归通过。
- `AggregateResult.to_dict()` 不携带 aggregate ref、receipt、writer/time 等持久化后字段，避免自引用。

### Immutable store 与 selector CAS

- immutable result 使用 content-derived aggregate ID 与 exclusive create；existing same payload 幂等，different payload conflict。
- persist 顺序是 canonical validation → append → exact readback + digest → selector CAS。
- selector CAS 只在 expected current 匹配时更新；同/冲突并发测试证明 current winner 数为 1，last-write-wins=0。
- stale lock 选择 fail closed；这是可用性风险，不是证据篡改风险。

### Projection hard gate

- projector 前置同时要求 overall PASS、terminal、ELIGIBLE、receipt identity/digest、readback_valid、current_selected、非 conflict/failed。
- 调用 projector 前再次重读 aggregate 并比较完整 payload，再检查 store current ref。
- 15 个 non-PASS、stale 结果和四类无效 receipt 均在 projector 前 HOLD。
- controlled writer 只把 aggregate ref 加入 state source refs、更新 next action，并追加 `aggregate_projection` ledger event；不清空 active change/story，不进入 delivered。
- `close_cr` / `sync_cr_status` 在同文件中是其他显式 CLI 能力；AggregateCompletionProjector 未调用它们，integration test 也用 forbidden monkeypatch 验证调用数为 0。

### CLI dry-run 与禁止边界

- dry-run 在 validated compute 后直接返回，未要求 store，不执行 persist 或 project。
- aggregate 模块静态扫描未发现 Git、worktree、subprocess、manual sync、rollback、close/status-sync 依赖或调用。
- 本轮 QA 未执行 Git/worktree/manual sync/close/status-sync/rollback，也未写源码/测试。

## 人工 / 语义质量审查

| 维度 | 结论 | 说明 |
|---|---|---|
| 需求一致性 | PASS | 五项 acceptance 均有实现入口与运行证据 |
| 场景覆盖 | PASS | happy path + 16 决策表 + invalid + non-PASS + stale/conflict/failure |
| 输入信任边界 | PASS | 只消费重读且 correlation/receipt/digest 绑定的 handle |
| 错误信息 | PASS | validation code、disposition、next route 可行动 |
| 数据不可变性 | PASS | append-only result；conflict 不覆盖 |
| 并发安全 | PASS_WITH_RISK | CAS/fail-closed 通过；native Windows crash 未测 |
| 权限 / 安全 | PASS | aggregate 零 Git/worktree；真实运行未授权 |
| 文档可用性 | PASS | CP6 证据、测试计划与 CP7 报告可追踪 |

## 评审结论

实现没有需要 `NEEDS_REWORK` 的缺陷，阻断 finding 数为 0。以 `PASS_WITH_RISK` 交还 host-orchestrator；低风险项只作为 CP8 风险披露/后续验证输入，不由本 reviewer 修改实现或批准 gate。
