---
status: archived-changes-requested
version: "1.0.1"
created_at: "2026-07-16"
owner: "host-orchestrator-inline / meta-se"
cr_ref: "CR-050"
review_gate: "CP3"
superseded_by_scope_change: "CP2 R2 explicit merge baseline; content revision deferred until CP2 R2 approval"
---

# CR-050 Governed Git Branch Lifecycle Dependency Map

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-16 | host-orchestrator-inline / meta-se | 建立 Host/CLI、CR coordinator、branch workflow、workspace Git、route、result/ledger 与 Git remote 的单向依赖和禁止依赖。 |
| 1.0.1 | 2026-07-16 | host-orchestrator | 仅更新审查状态：显式 merge 将改变 coordinator/authorization/default-write 依赖；保留 v1.0 正文，待 CP2 R2批准后修订。 |

## 1. 依赖目标

保持现有分层：CLI/Host 负责编排入口，workflow 负责领域策略，workspace 负责 Git 与路由适配，现有 CR/state/result/ledger writer 各自单写。任何下层组件不得反向驱动 CR 状态或假装跨仓事务。

## 2. 模块依赖图

```text
Host Orchestrator / meta_flow.cli
                |
                v
existing CR lifecycle coordinator -----> existing CR/state/current writers
                |
                v
Git branch lifecycle planner/executor --> operation result + existing ledger adapters
                |
        +-------+--------+
        v                v
workspace routing   native Git probe/executor
                         |
                         v
             project repo / artifact repo / remotes
```

依赖方向只能向下；observation/result 可返回上层，但不构成下层对上层模块的 import。

## 3. 允许依赖

| From | To | 调用时机 | 契约 |
|---|---|---|---|
| Host/CLI | CR lifecycle coordinator | start/open、publish、finish 显式调用 | 解析 typed args、输出机器/人类结果、传播非零状态 |
| CR lifecycle coordinator | existing CR bootstrap/status writer | start 的 branch local prepare 后；branch push前后按策略记录 | 不复制 CR writer；失败保留 formal CR/partial facts |
| CR lifecycle coordinator | branch lifecycle service | 每次 operation | 输入 intent/authz/result refs，消费 typed plan/outcome |
| branch lifecycle service | workspace routing | operation 观察前 | 获得 project/artifact roots；route blocking 原样阻断 |
| branch lifecycle service | native Git adapter | probe/execute/verify | 仅 argv list、固定 cwd、timeout/有界输出、无 shell |
| branch lifecycle service | operation result writer | attempt 终态 | append-only result；result 不成为 ref truth |
| result/Host | RUN/CR ledger writer | result 已落盘并验证后 | 只追加摘要/ref/decision，不复制完整 plan |
| native Git adapter | Git executable/repos/remotes | allowlisted command | 不读取凭据存储、不拼 shell、不 force |

## 4. 禁止依赖

| ID | 禁止方向/行为 | 原因 | 检查方式 |
|---|---|---|---|
| FD-GB-01 | workspace Git adapter → CR/state/current writer | 下层 Git 工具不能决定业务阶段 | import/architecture test |
| FD-GB-02 | routing → branch lifecycle policy | route 只提供位置/健康，不理解 CR branch | import boundary |
| FD-GB-03 | operation result → Git mutation | 证据不能被“读取即执行” | API review/fixture |
| FD-GB-04 | finish → forge API/auto merge | CP2 已排除，权限和 proof contract 不存在 | forbidden command/import scan |
| FD-GB-05 | any module → shell interpolation/force/reset/rebase/stash | 防止命令注入和历史改写 | argv allowlist + negative fixtures |
| FD-GB-06 | artifact repo success → project repo success 推断 | 两仓非原子且 OID 独立 | per-repo result schema |
| FD-GB-07 | CR closed/state delivered → merge proof PASS | workflow 状态不能替代 Git ancestry | finish proof fixture |
| FD-GB-08 | branch service → automatic commit/stage | 文件选择未授权 | command spy/negative fixture |

## 5. Operation DAG

### 5.1 Start

```text
validate intent/authorization
  -> resolve route and repos
  -> read-only precheck all repos
  -> actual fetch/default fast-forward all repos
  -> recheck exact bases and collisions
  -> create/switch local CR branches (deterministic order)
  -> existing CR bootstrap writes formal process artifacts
  -> push -u exact branch refs (deterministic order)
  -> verify remote/upstream/OIDs
  -> persist result + append ledger refs
```

precheck 任一失败时 local/remote branch mutation=0。local prepare 后失败属于 PARTIAL，不自动删除已创建 branch；resume 重新观察事实。

### 5.2 Publish

```text
observe both -> precheck clean/identity/upstream/non-FF all
  -> push project exact refspec -> verify
  -> push artifact exact refspec -> verify
  -> result/ledger
```

若后仓失败，前仓事实保留，overall=PARTIAL；不得 force 或回退前仓 remote ref。

### 5.3 Finish

```text
observe/fetch both -> identify known tips -> protected/tip/ancestry proof all
  -> create/reuse local recovery refs all
  -> prepare/switch/ff-only local defaults all
  -> delete exact remote CR branch project -> verify absent
  -> delete exact remote CR branch artifact -> verify absent
  -> branch -d local CR refs all
  -> result/ledger
```

两仓 proof 全通过前不执行任何 remote delete；两仓 remote delete 全完成前不执行 local branch delete。这样不能消除 partial，但保留最大恢复面。

## 6. 文件所有权候选

CP3 只冻结职责，不冻结函数签名；CP4/CP5 可按以下候选分配：

| 文件/目录 | 主要 owner | 变化类型 |
|---|---|---|
| `meta_flow/workspace/git_sync.py` | FEAT-GB shared adapter | 扩展 safe Git probe/executor 与 repo observations |
| `meta_flow/workflow/git_branch_lifecycle.py` | FEAT-GB-01..03 | 新增领域 plan/execution/result contract |
| `meta_flow/workflow/cr_lifecycle.py` | FEAT-GB-01 integration | 复用 bootstrap 并接入 governed start，不复制 writer |
| `meta_flow/cli.py` | CLI integration | 暴露显式 start/publish/finish/dry-run 参数 |
| `tests/test_workspace_git_sync.py` | shared adapter tests | 保留既有行为并补底层 probe |
| `tests/test_git_branch_lifecycle.py` | FEAT-GB-01..03 | bare remote 正/负/partial/idempotency fixture |
| `README.md`, `delivery/doc/USER-MANUAL.md` | operator docs | 命令、授权、失败恢复与“不自动 merge/commit” |

若 CP4 发现三个 Story 同时修改同一核心函数，应由一个 shared-contract Story 先落公共类型/runner，后续 Story 只扩展 operation handler；不得用并行写同一文件制造隐式循环。

## 7. 循环消除

| 潜在循环 | 断环规则 |
|---|---|
| CR bootstrap 需要 branch，branch 又需要 formal CR | `CR Start Intent` 先通过无写校验；coordinator 先准备 local branch，再调用现有 bootstrap；二者不互相 import |
| result 需要 artifact branch，start 又需要 result | start 先返回内存 outcome，formal CR/bootstrap 后再持久化；早期失败用 CLI terminal JSON，不伪造已持久化 evidence |
| finish 需要 recorded tip，而 result 可能缺失 | remote/local exact tip可作为有类型来源；三者冲突/全缺时 BLOCKED，不回写猜测 |
| state close 与 branch finish 相互触发 | finish 是显式前置证据；CR close消费 result ref，但 finish不读取 closed 来证明 merge |

## 8. Gotchas

1. 把 start 直接塞入现有 `cr bootstrap` 内部会让旧调用方突然产生远端写；必须由显式新入口/flag 和授权隔离。
2. preflight-all 只能降低 partial 概率，不能消除网络/远端竞态；result 必须保存每个已执行 step。
3. recovery ref 创建成功也是 mutation；如果随后 proof/删除失败，结果应记录它，重试只允许同 OID 幂等复用。
4. artifact result writer 不能在 native Git adapter 内部调用，否则形成 Git→process writer 的反向依赖。
