---
status: archived-changes-requested
version: "1.0.1"
created_at: "2026-07-16"
owner: "host-orchestrator-inline / meta-se"
cr_ref: "CR-050"
source_requirements: "process/docs/product/REQUIREMENTS.md"
source_story_map: "process/docs/product/STORY-MAP.md"
review_gate: "CP3"
superseded_by_scope_change: "CP2 R2 explicit merge baseline; content revision deferred until CP2 R2 approval"
---

# CR-050 Governed Git Branch Lifecycle Blueprint

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-16 | host-orchestrator-inline / meta-se | 建立原生 Git 双仓 CR branch start/publish/finish 能力边界、数据归属、集成方向和安全边界，供 CP3 决策。 |
| 1.0.1 | 2026-07-16 | host-orchestrator | 仅更新审查状态：用户新增显式两仓 ff-only merge，当前 Blueprint 对 v1.7 产品基线不完整；保留 v1.0 正文，待 CP2 R2批准后生成实质修订。 |

## 1. 蓝图适用性

CR-050 同时改变 Host Orchestrator 的 CR 启动顺序、project/artifact 两仓 Git ref、删除授权和审计证据，且包含远端写入与不可逆清理风险，因此需要 Blueprint、Domain Map、Dependency Map 和 HLD。它不是单文件 CLI 增量，也不适合只写 technical note。

## 2. 目标与边界

目标是把以下旅程收敛为一个可审计能力：

1. `start/open`：在两仓均通过预检后，从各自刷新后的远端默认分支建立同名 CR branch 与 upstream；
2. `publish`：只发布调用前已存在的 commit，不 stage、commit、amend 或 force；
3. `finish`：外部 merge 完成后，以 exact tip 与 Git ancestry 证明为前提清理目标 branch。

边界内不包含 forge API、自动 merge、squash/rebase 推断、跨仓原子事务、自动提交、Git Town/`gb` 依赖、凭据管理或 force 类补偿。

## 3. 能力地图

| Capability | 用户结果 | Feature | 主要输入 | 主要输出 |
|---|---|---|---|---|
| CAP-GB-01 Governed CR Start | 两仓从各自 remote default exact tip 开始同名 CR | FEAT-GB-01 Paired Branch Start | CR start intent、workspace route、remote/default override | branch/upstream/base OID、逐仓 start result |
| CAP-GB-02 Committed Ref Publication | 远端 CR ref 精确等于显式提交后的本地 HEAD | FEAT-GB-02 Safe Ref Publish | CR/branch identity、local/remote OID、clean status | publish result、remote verification、恢复入口 |
| CAP-GB-03 Proof-gated Cleanup | 未证明合并时删除数为 0，证明充分时精确清理 | FEAT-GB-03 Merge Proof and Cleanup | recorded/local/remote tip、protected policy、remote default | proof、recovery ref、remote/local cleanup result |

## 4. Feature 边界

### FEAT-GB-01 Paired Branch Start

- 负责：两仓发现、Git/version/route/clean/detached/remote/default/ref collision 预检；确定性 branch policy；远端刷新；本地 branch 准备；现有 CR bootstrap 衔接；`push -u` 与 OID 核验。
- 不负责：选择提交文件、生成业务 commit、自动回滚已成功的远端写入、后续 merge。
- 消费：现有 workspace route、现有 CR bootstrap validation。
- 产出：一个 operation result，包含 overall/per-repo terminal status、before/after OID、执行与跳过步骤、recovery route。

### FEAT-GB-02 Safe Ref Publish

- 负责：校验当前 branch/upstream/clean 状态、remote ref 非快进风险；推送 exact branch refspec；核验 remote OID。
- 不负责：stage/commit/amend、secret 选择、PR 创建或 force-push。
- 消费：FEAT-GB-01 建立的 branch identity 与 start evidence。
- 产出：publish attempt result；`remote_tip == local_head` 才能逐仓 PASS。

### FEAT-GB-03 Merge Proof and Cleanup

- 负责：重新观察 remote default/target ref；protected-ref、tip-drift、ancestry 校验；建立本地 recovery ref；精确远端删除；两仓远端步骤完成后再执行 `branch -d`。
- 不负责：merge、patch-id 猜测、forge receipt 生产、force-delete、自动删除 recovery ref。
- 消费：最后一次可信 publish/start OID、当前 local/remote refs、protected policy。
- 产出：proof 与 cleanup attempt result；证明不足必须 BLOCKED 且不执行远端删除。

## 5. 共享支撑组件

| 组件 | 所有者 | 职责 | 禁止承担 |
|---|---|---|---|
| Native Git Probe/Executor | `meta_flow.workspace.git_sync` | 以 argv list、固定 cwd、无 shell 方式执行允许的 Git probe/mutation；返回有界 stdout/stderr 与 exit code | CR 状态写入、自由 shell、凭据解析、业务补偿 |
| Branch Lifecycle Planner | branch lifecycle workflow module | 将 intent + observations 变成确定性 plan，区分 dry-run、preflight、mutation、verification | 直接成为 Git ref 真相源或跨仓事务协调器 |
| CR Start Coordinator | existing CR lifecycle boundary | 在同一次显式 start 中编排 `preflight → local branch prepare → CR bootstrap → remote tracking publish` | 隐式 hook、跳过 CR-first、自动 commit |
| Evidence/Run Adapter | existing result + RUN/CR ledgers | 保存 versioned attempt result 与 ref，不复制 Git 对象或建立第二套 branch state DB | 覆盖旧 attempt、把 PARTIAL 写成 PASS |

## 6. 数据归属

| 对象 | 真相所有者 | 持久化位置 | 读写规则 |
|---|---|---|---|
| Git local/remote refs | 对应 Git repository/remote | `.git` 与 remote | lifecycle service 只按允许命令读写；result 只是观察证据 |
| CR lifecycle/status | existing CR/state writers | `process/changes/**`、state/index/ledgers | branch service 只读 CR identity；由现有 writer 更新状态 |
| Branch operation attempt | branch lifecycle service | `process/checks/CR-BRANCH-<CR>-<operation>-<attempt>.result.json` 或显式 `--output` | append-only attempt；重试用新 attempt 与 supersession/ref，不覆盖历史 |
| Run/CR correlation | existing ledger writers | `RUN-LEDGER.ndjson` / `CR-LEDGER.ndjson` | 只追加 result ref、decision、repo terminal status 摘要 |
| Recovery ref | repository-local Git ref | `refs/meta-flow/recovery/<cr>/<branch>` | local-only、同名同 OID 幂等；不同 OID 冲突即阻断；不得自动 push |
| CURRENT/state projection | existing state/current writers | existing current/state files | 只在对应工作流状态迁移时更新；不保存完整 Git plan/result |

## 7. 集成契约

| 调用方 | 时机 | 触发 | 输入 | 输出 | 后续 | 降级/失败 |
|---|---|---|---|---|---|---|
| Host Orchestrator → CR Start Coordinator | 用户正式开启 CR 且授予 branch publication | 显式 CLI/action；不得后台 hook | CR ID/title/slug、remote、可选 default override、dry-run | CR bootstrap refs + paired start result | requirement clarification | 任一预检失败零 branch mutation；partial 时停止并给 resume |
| Host/Developer → Publish | 两仓显式 commit 后 | 显式 command | CR ID、expected branch/OID | paired publish result | CP/协作/远端 review | dirty/wrong/non-FF/block；不 stage/commit |
| Host/Approver → Finish | 外部 merge 后且删除授权明确 | 显式 command | CR ID、branch、evidence ref、remote/default | paired cleanup result | CR close/branch retirement evidence | ancestry/tip/protected 不满足即 BLOCKED；不猜测 |

## 8. 权限与风险分区

- Read-only：route、status、symbolic remote HEAD、ref/OID、ancestry、branch format。
- Local mutation：fetch remote-tracking refs、ff-only default refresh、switch/create branch、local recovery ref、`branch -d`。
- Remote mutation：`push -u`、普通 push、exact remote branch delete；每次必须由显式调用授权。
- 永久禁止：force-push/force-delete、reset --hard、自动 rebase、自动 merge、自动 stash、shell interpolation、凭据读取/打印。

`--dry-run` 只允许 read-only probe；不得把 `fetch --dry-run` 的输出冒充已刷新本地 refs，也不得声称未在本地可证明的 ancestry 已通过。

## 9. 依赖与发布切片

```text
FEAT-GB-01 Paired Branch Start
  └─ FEAT-GB-02 Safe Ref Publish
       └─ FEAT-GB-03 Merge Proof and Cleanup
```

三项 Feature 对应 ST-GB-001..003 和 SL-GB-01..03，按生命周期串行交付。共享 Git executor、result contract、安全策略和回滚边界，因此保留一个 CR/HLD；Story/LLD 在 CP4/CP5 分开。

## 10. Gotchas

1. `git push --dry-run` 不是完整零副作用证明；设计必须在命令前后比较 local/remote refs，且 dry-run 不执行 fetch/pull/switch/update-ref。
2. 远端 branch 不存在不代表已经安全合并；必须仍有 recorded/local tip 并通过 ancestry，或未来可信 forge receipt。
3. 两仓“预检都通过”不等于后续原子成功；任何成功事实必须保留，不能用 force 伪装一致。
4. 当前 artifact tree 可能已含 CR 过程文件；新能力只能在未来 `cr start` 的 clean precondition 下启用，不能拿本 CR 的脏工作树做真实 dogfood。
5. recovery ref 是本地安全锚，不是发布 tag；不得自动 push，也不能作为“已合并”证明。
