---
status: confirmed
version: "1.2"
created_at: "2026-07-16"
owner: "host-orchestrator-inline / meta-se"
cr_ref: "CR-050"
review_gate: "CP3-R3"
supersedes: "process/archive/CR-050/design/CR050-GIT-BRANCH-DEPENDENCY-MAP.v1.0.1.md"
---

# CR-050 Governed Git Branch Lifecycle Dependency Map

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-16 | host-orchestrator-inline / meta-se | 建立open/publish/finish模块依赖和禁止依赖。 |
| 1.0.1 | 2026-07-16 | host-orchestrator | 因显式merge范围变化标记changes-requested并归档。 |
| 1.1 | 2026-07-16 | host-orchestrator-inline / meta-se | 把独立merge command、default-write authorization、artifact→project choreography、merge-attempt→finish gate加入单向依赖与Operation DAG。 |
| 1.2 | 2026-07-16 | host-orchestrator-inline / meta-se | CP3 R3在attempt与workflow writer之间加入2/2 Paired Projection Gate；补充artifact-first PARTIAL不可推进与project-first切换条件。 |

## 1. 依赖原则

```text
Host Orchestrator / meta_flow.cli
        |
        v
CR lifecycle coordinator --------------------> existing CR/state/current writers
        |
        v
Branch lifecycle planner/executor -----------> append-only result + existing ledger adapters
        |                                              |
        +---------------------> 2/2 projection gate ---+--> existing CR/state/current writers
        |               |
        v               v
workspace routing   authorization validator
        |               |
        +-------> native Git adapter ---------> project/artifact repos/remotes
```

下层不得反向驱动CR阶段；authorization只判断typed authority，不执行Git；result只记录attempt，不触发mutation。

## 2. 允许依赖

| From | To | 调用时机 | 契约 |
|---|---|---|---|
| Host/CLI | CR lifecycle coordinator | 显式open/publish/merge/finish | typed args、operation分离、传播非零/PARTIAL |
| coordinator | existing CR bootstrap/status writer | open准备后、workflow状态迁移时 | 复用writer；旧bootstrap默认无remote write |
| coordinator | branch lifecycle service | 每个operation | intent/authz/source result→plan/outcome |
| lifecycle service | workspace routing | observation前 | 发现project/artifact；configured artifact不得静默跳过 |
| lifecycle service | authorization validator | 每次actual remote mutation前 | operation/repo/ref/OID必须全匹配 |
| lifecycle service | native Git adapter | probe/execute/verify | argv list、cwd、timeout/redaction、allowlist |
| lifecycle service | result writer | attempt终态 | append-only；完整per-repo terminal/resume |
| result validator | paired projection gate | 两仓terminal与post-check齐备后 | 仅2/2 PASS/NO_CHANGE开放workflow PASS/finish；PARTIAL保持active |
| Host/result | RUN/CR ledger writer | result校验后 | 只追加摘要/ref，不复制plan正文 |

## 3. 禁止依赖

| ID | 禁止方向/行为 | 原因 | 检查 |
|---|---|---|---|
| FD-GB-01 | Git adapter→CR/state/current writer | 下层工具不能决定业务阶段 | import boundary |
| FD-GB-02 | result→Git mutation或“读取即恢复” | 证据不是命令 | API/fixture |
| FD-GB-03 | publish/finish→merge handler | merge必须独立显式触发 | command-spy |
| FD-GB-04 | merge handler→finish/delete | default write不授权删除 | command-spy |
| FD-GB-05 | branch service→forge API/credential store | 本CR无平台receipt/credential契约 | import/command scan |
| FD-GB-06 | any→shell/force/reset/rebase/merge-commit/stash/auto-commit | 防注入与历史改写 | argv allowlist+negative fixtures |
| FD-GB-07 | artifact success→project success推断 | 两仓非原子 | result schema |
| FD-GB-08 | merge PASS→branch delete | finish必须fresh reproof和独立delete authz | finish fixture |
| FD-GB-09 | CR closed/delivered→merge/ancestry proof | workflow truth不能替代Git truth | proof fixture |
| FD-GB-10 | path/branch名→authorization inheritance | authority需绑定operation/repo/OID | authz negative fixture |
| FD-GB-11 | artifact repo outcome/ref→paired workflow PASS/CR close | 单仓成功不能证明源码仓成功 | projection-gate partial fixture |

## 4. Operation DAG

### 4.1 Open

```text
validate intent/route/authz -> read-only precheck all
  -> fetch/default ff-only all -> create local branches
  -> existing bootstrap writer -> push -u exact refs -> verify -> result/ledger
```

### 4.2 Publish

```text
observe all -> clean/identity/upstream/non-FF precheck all
  -> ordinary exact push per repo -> immediate remote OID verify -> result/ledger
```

### 4.3 Explicit merge

```text
load matching publish evidence
  -> fresh observe project+artifact
  -> validate exact branch/published/default OID + authz + ff eligibility for ALL
  -> dry-run: emit plan only
  -> artifact exact default update -> verify
  -> project exact default update -> verify
  -> 2/2 terminal result
       -> paired projection gate
          PASS/NO_CHANGE 2/2 -> workflow merge PASS; finish eligible (not authorized)
          PARTIAL/BLOCKED -> projection=false; CR active; retain 2/2 CR branches; finish blocked
```

默认实现应使用不含force的exact ref update，使remote自行执行non-FF/protection检查；不得先在本地创建merge commit。CP5应冻结为argv语义`git push --porcelain <remote> <published_oid>:refs/heads/<default>`，不含`+`、`--force`或`--force-with-lease`，并要求fresh post-check。

### 4.4 Finish

```text
require current 2/2 merge PASS + separate delete authz
  -> fetch/observe all -> re-identify tips -> proof all
  -> recovery refs all -> remote CR deletes all -> verify absent all
  -> local branch -d all -> result/ledger -> CR close may consume result
```

## 5. 文件所有权候选

| 路径 | owner | 变化 |
|---|---|---|
| `meta_flow/workspace/git_sync.py` | shared native Git adapter | safe probe/executor/repo observation，不放业务状态机 |
| `meta_flow/workflow/git_branch_lifecycle.py` | FEAT-GB-01/02/03/04 | typed intent/plan/attempt、operation handlers、paired choreography |
| `meta_flow/workflow/cr_lifecycle.py` | FEAT-GB-01 integration | 显式open coordinator，复用bootstrap writer |
| `meta_flow/cli.py` | CLI integration | 独立open/publish/merge/finish与dry-run/output/authz参数 |
| `tests/test_git_branch_lifecycle.py` | lifecycle tests | bare remote正/负/partial/resume/command-spy fixtures |
| `tests/test_workspace_git_sync.py` | adapter tests | 兼容既有status/push并补底层ref观察 |
| `README.md`, `delivery/doc/USER-MANUAL.md` | operator docs | 四阶段旅程、三类授权、PARTIAL恢复、禁止项 |

若CP4发现四Story同时改同一核心类型/runner，先由ST-GB-001落shared contract；后续Story串行扩展operation handler，禁止并行改同一文件。

## 6. 断环规则

| 潜在循环 | 断环 |
|---|---|
| clean precheck需要branch先于bootstrap，而CR-first又需formal CR | coordinator先验证intent并准备local branch，再复用bootstrap；旧bootstrap不import branch service |
| merge依赖publish证据，而resume需fresh refs | result仅提供expected identity；fresh refs决定当前eligibility，不反写旧result |
| finish依赖merge result，merge可能PARTIAL | finish只消费current 2/2 PASS；PARTIAL没有自动“修正”路径 |
| CR close与branch finish互相证明 | finish产出可被close消费的result；finish不读取closed来证明Git事实 |
| artifact先写导致过程真相可能领先源码 | attempt writer只写逐仓事实；paired projection gate必须等2/2 post-check后才允许existing writer推进 |

## 7. Gotchas

1. artifact→project是审计顺序，不是事务；即使preflight全过，远端仍可在两次push间变化。
2. merge handler不能复用finish的delete authorization，finish也不能复用merge的default-write authorization。
3.把merge实现为本地checkout+merge再push会污染worktree并引入merge commit风险；优先保持ref-to-ref exact普通push语义。
4. native Git adapter返回的stderr可能含remote URL；result必须保存安全摘要而非泄露凭据。
5. 若CP4/CP5无法用模块边界和fixture保证`PARTIAL => projection=false`，必须采用project-first并重开CP2/CP3，不能让读取方自行约定“不要误读”。
