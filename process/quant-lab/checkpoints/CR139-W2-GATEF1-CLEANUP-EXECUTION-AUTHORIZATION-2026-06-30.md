---
checkpoint_id: "CR139-W2-GATEF1-CLEANUP-EXECUTION-AUTHORIZATION-2026-06-30"
checkpoint_name: "CR139 W2 Gate F-1 Cleanup Execution Authorization"
type: "manual_runtime_authorization"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-06-30T07:19:31+08:00"
reviewed_by: "user"
reviewed_at: "2026-06-30T07:30:00+08:00"
auto_check_result: "process/checks/CR139-W2-GATEF1-CLEANUP-AUTHORIZATION-PREFLIGHT-2026-06-30.md"
target:
  phase: "post-cp8-cleanup"
  workflow_id: "CR139-W2"
  active_change: "CR-139"
---

# CR139 W2 Gate F-1 Cleanup Execution Authorization

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CR139-W2-GATEF1-CLEANUP-AUTHORIZATION-PREFLIGHT-2026-06-30.md` | PASS | 0 | 19 条 F-1 cleanup 路径可进入授权评审；210 条 legacy 已排除。 |

## Decision Brief

### 审批者摘要

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 执行 CR139 W2 Gate F-1 低风险 cleanup：只清理 CR139 自产 staging 残留，不触碰 legacy canonical。 |
| 推荐动作 | `approve`：授权按精确 manifest 删除 2 个 orphan CR139 target 和 17 个 candidate dataset tree。 |
| approve 后会发生什么 | 执行 F-1 cleanup；生成 pre-delete snapshot、delete execution evidence、post-delete verification、active metadata unchanged proof。 |
| approve 不授权什么 | 不授权删除 210 个 legacy canonical candidates；不授权 active catalog/manifest 写入、provider catalog、NAS、runtime、credential、Git remote、额外迁移。 |
| 不确认会阻塞什么 | 阻塞 Gate F-1 cleanup，进而阻塞 cleanup 后的 Gate H NAS dry-run/sync。 |

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP8-CR139-W2-DELIVERY-CONTEXT.yaml` |
| capsule 状态 | ready |
| read_profile | compact |
| 默认读取策略 | CP8 capsule first；F-1 额外读取 cleanup delete manifest preview 和 F-1 preflight evidence。 |
| 全文档读取扩展 | 1 次；F-1 是删除授权，必须读取精确 delete manifest 和 active catalog/manifest hash。 |
| 缺失 / waived 理由 | N/A |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| CP8 approved decision | `process/checkpoints/CP8-CR139-W2-DELIVERY-READINESS.md` | scanned | 1 | 1 | DQ-002 批准 F-1/F-2 分阶段，F-1 需单独执行授权。 |
| Gate F cleanup preview | `process/evidence/CR139-W2-GATEF-CLEANUP-DELETE-MANIFEST-PREVIEW-2026-06-29.jsonl` | scanned | 3 | 2 | F-1 includes orphan + candidate; legacy excluded to F-2. |
| Gate F-1 preflight | `process/evidence/CR139-W2-GATEF1-CLEANUP-AUTHORIZATION-PREFLIGHT-2026-06-30.json` | scanned | 2 | 2 | Active catalog conflict absent; delete scope count validated. |
| 用户推进顺序 | 当前对话 | scanned | 1 | 1 | 用户要求按顺序推进，无风险不发起门禁；删除属于风险动作，必须发起授权。 |

Coverage arithmetic: candidate issue groups = 7 gross across sources, deduplicated decision items = 4, duplicates merged = 3, N/A = 0.

### 决策分层

| 分类 | 数量 | 处理方式 |
|---|---:|---|
| 必须用户决策 | 1 | 是否授权执行 F-1 delete。 |
| 高风险策略确认 | 2 | 只删 19 条 F-1 scope；禁止 legacy 210 条。 |
| agent 默认处理 | 1 | delete execution 采用 exact-manifest paths，不使用 glob。 |
| 仅审计记录 | 1 | active catalog/manifest hash 必须保持不变。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| GATEF1-DQ-001 | runtime_authorization | 是否执行 Gate F-1 cleanup。preflight 证明 19 条 F-1 路径不被 active catalog 指向，且 210 条 legacy 已排除。 | 授权执行 F-1 exact-manifest delete：2 orphan CR139 targets + 17 candidate dataset trees。 | A: 暂缓 F-1；B: 只删 2 orphan，不删 candidate；C: 一次性连同 legacy 删除。 | 推荐方案清理 CR139 staging 冗余并降低 NAS 前同步体积。A 保留 2.76GB 冗余；B cleanup 不完整；C 越过 CP8 DQ-003，风险过高。 | 删除是破坏性动作；若路径匹配错误可能误删 current truth。因此必须 exact manifest、禁止 glob、post-verify active hash 不变。 | 若 pre-delete snapshot 发现 active catalog/manifest 指向任一 F-1 path，立即中止；若部分删除失败，停止后续删除并生成 partial evidence，不自动回滚。 |
| GATEF1-DQ-002 | security | 是否严格禁止 legacy canonical cleanup。 | 禁止删除 210 个 `review_delete_or_archive_legacy_canonical` records；留给 F-2 retain/superseded 决策。 | A: 同时删除 legacy；B: archive legacy。 | 推荐方案遵守 CP8 修正：legacy 默认 retain + mark superseded。A 不可逆风险高；B 属 F-2，不应混入 F-1。 | 避免删除历史研究/回测可能引用的数据。 | 若 execution manifest 中出现 legacy category，F-1 必须 FAIL。 |
| GATEF1-DQ-003 | implementation | 删除方式是否限定为 exact-manifest paths。 | 逐条读取 F-1 manifest 记录删除；禁止 glob、禁止 broad `rm -rf candidate/parquet`、禁止按 dataset 模糊匹配。 | A: 用目录 glob 批量删；B: 手工 rm。 | 推荐方案可审计、可复现、可按记录比对。A/B 都提高误删风险。 | 执行复杂度略高，但换来可机械复核。 | 若 manifest 记录数量不是 19 或 category 不在 allowlist，立即中止。 |
| GATEF1-DQ-004 | risk_acceptance | 发生部分失败时如何处理。 | `fail_stop_no_automatic_rollback`：停止剩余删除，记录 partial evidence，等待人工决策。 | A: 自动重试/继续；B: 自动恢复已删对象。 | 推荐方案避免在不确定状态继续扩大影响。A 可能扩大错误；B 需要额外备份/恢复授权。 | 可能留下部分残留，但证据清楚。 | 如果失败是权限/文件不存在且未影响 active truth，可单独授权 retry。 |

### 用户需决策事项

| 决策项 | 推荐结论 | 用户回复 `approve` 的含义 |
|---|---|---|
| GATEF1-DQ-001 | 授权执行 F-1 exact-manifest delete | 删除 2 个 orphan CR139 targets + 17 个 candidate dataset trees，共 19 条 F-1 记录。 |
| GATEF1-DQ-002 | 严格禁止 legacy cleanup | 210 条 legacy canonical candidates 不在本轮执行范围内。 |
| GATEF1-DQ-003 | 只允许 exact-manifest paths | 执行器不得使用 glob、broad `rm -rf` 或手工模糊删除。 |
| GATEF1-DQ-004 | 接受 fail-stop 部分失败策略 | 删除失败时停止剩余删除，生成 partial evidence，等待人工决策。 |

## F-1 Authorized Scope If Approved

| Class | Count | Planned bytes | Planned rows | Authorization |
|---|---:|---:|---:|---|
| orphan CR139 targets | 2 | 1,357,070 | 327,358 | allowed |
| candidate dataset trees | 17 | 2,758,856,910 | 123,568,076 | allowed |
| F-1 total | 19 | 2,760,213,980 | 123,895,434 | allowed |
| legacy canonical candidates | 210 | N/A | N/A | forbidden in F-1 |

## Required Execution Evidence

| 证据 | 要求 |
|---|---|
| pre-delete snapshot | active catalog hash、active manifest hash、19 条路径存在性、210 legacy excluded |
| delete execution evidence | 每条 deleted path、category、dataset、pre/post exists、file/dir count、size |
| post-delete verification | 19 条 F-1 path 不存在；210 legacy 仍存在或未被执行器触碰；active catalog/manifest hash 不变 |
| operation counts | `orphan_delete=2`、`candidate_delete=17`、`legacy_delete=0`、active/provider/NAS/runtime/Git counters all 0 |
| partial failure behavior | fail-stop，写 partial evidence，不自动回滚 |

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| CP8 approved | 待审查 | `process/checkpoints/CP8-CR139-W2-DELIVERY-READINESS.md` |  |
| F-1 preflight PASS | 待审查 | `process/checks/CR139-W2-GATEF1-CLEANUP-AUTHORIZATION-PREFLIGHT-2026-06-30.md` |  |
| Delete scope exact | 待审查 | `process/evidence/CR139-W2-GATEF1-CLEANUP-AUTHORIZATION-PREFLIGHT-2026-06-30.json` |  |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | F-1 delete scope is exactly 19 records | 待审查 | GATEF1-DQ-001 |  |
| 2 | Legacy 210 records are excluded | 待审查 | GATEF1-DQ-002 |  |
| 3 | Exact-manifest delete only | 待审查 | GATEF1-DQ-003 |  |
| 4 | Partial failure policy accepted | 待审查 | GATEF1-DQ-004 |  |
| 5 | Non-authorized operations are explicit | 待审查 | Decision Brief |  |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| Human authorization recorded | 待审查 | 本文件 `人工审查结果` |  |
| F-1 execution boundaries accepted | 待审查 | GATEF1-DQ-001~004 |  |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| F-1 preflight check | `process/checks/CR139-W2-GATEF1-CLEANUP-AUTHORIZATION-PREFLIGHT-2026-06-30.md` | 待审查 |  |
| F-1 preflight evidence | `process/evidence/CR139-W2-GATEF1-CLEANUP-AUTHORIZATION-PREFLIGHT-2026-06-30.json` | 待审查 |  |
| F-1 authorization brief | `process/checkpoints/CR139-W2-GATEF1-CLEANUP-EXECUTION-AUTHORIZATION-2026-06-30.md` | 待审查 |  |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-06-30T07:30:00+08:00
- 修改意见：批准 Gate F-1 cleanup execution only；只删 preflight 列出的 2 个 orphan CR139 targets 和 17 个 CR139 candidate dataset trees；排除 210 个 legacy canonical candidates；exact-manifest paths only；禁止 active catalog/manifest/provider/NAS/runtime/credential/Git remote。
- 风险接受项：
  - 接受 F-1 delete 是破坏性动作，但范围限定为 CR139 自产 staging / orphan target，且 active catalog conflict count = 0。
  - 接受 fail-stop no automatic rollback 策略。

## Exact Replies

请直接回复以下任一整行：

```text
approve
修改: <具体修改点>
reject
```

`approve` 表示授权 Gate F-1 执行 19 条 exact-manifest delete；不授权删除 210 条 legacy canonical candidates，不授权 NAS/provider/runtime/Git/active metadata 写入。
