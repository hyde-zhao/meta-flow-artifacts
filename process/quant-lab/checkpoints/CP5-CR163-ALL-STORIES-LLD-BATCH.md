---
checkpoint_id: "CP5-CR163-ALL-STORIES-LLD-BATCH"
checkpoint_name: "CR163 All-Story Full LLD Review"
type: "batch_auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-07-11T12:00:00+08:00"
reviewed_by: "user"
reviewed_at: "2026-07-11T12:25:00+08:00"
decision_brief_profile: "compact"
context_ref: "process/context/CP5-CR163-TRIAL-LINEAGE-INSTRUMENTATION-LLD-CONTEXT.yaml"
auto_check_results:
  - "process/checks/CP5-CR163-S01-family-contract-validator-LLD-IMPLEMENTABILITY.result.json"
  - "process/checks/CP5-CR163-S02-recorder-seal-supersession-LLD-IMPLEMENTABILITY.result.json"
  - "process/checks/CP5-CR163-S03-two-producer-chain-instrumentation-LLD-IMPLEMENTABILITY.result.json"
  - "process/checks/CP5-CR163-S04-existing-admission-projection-LLD-IMPLEMENTABILITY.result.json"
  - "process/checks/CP5-CR163-S05-integrity-recovery-permission-regression-LLD-IMPLEMENTABILITY.result.json"
target:
  phase: "story-planning"
  story_id: "all-cr163-stories"
  artifacts:
    - "process/stories/STORY-CR163-S01-family-contract-validator-LLD.md"
    - "process/stories/STORY-CR163-S02-recorder-seal-supersession-LLD.md"
    - "process/stories/STORY-CR163-S03-two-producer-chain-instrumentation-LLD.md"
    - "process/stories/STORY-CR163-S04-existing-admission-projection-LLD.md"
    - "process/stories/STORY-CR163-S05-integrity-recovery-permission-regression-LLD.md"
---

# CP5 CR163 全量 Story LLD 人工审查

## 自动预检摘要

| 预检范围 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| CP4 Story/DAG/parallel safety | PASS | 0 | 5 Stories、4 Waves、9 typed edges、cycle/invalid/file overlap=0。 |
| S01 full LLD | PASS | 0 | 14 章节、contract/state/validator、接口/测试/TASK 映射通过。 |
| S02 full LLD | PASS | 0 | v1.1；append-only/canonical seal/supersession、no-resume 产品限制、10k characterization 通过。 |
| S03 full LLD | PASS | 0 | v1.1；2/2 chains、CPI 4/4、single writer、programmatic/CLI config contract 通过。 |
| S04 full LLD | PASS | 0 | 3/3 existing consumers、validator projection、fail-closed/no new gate。 |
| S05 full LLD | PASS | 0 | 8/8 REQ、12/12 scenarios、5/5 negative classes、CR155 regression。 |
| CP5 context / clarification | PASS | 0 | capsule-first OK；三 lane 均无 clarification；blocking_open_count=0。 |

## Decision Brief

### 审批者摘要

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 确认 CR163 五个 Story 的实现设计已经具体到对象、文件、接口、失败路径、测试、TASK、依赖门和回滚，可进入受控的源码/测试实现。 |
| 推荐动作 | `approve`：批准五份 full LLD、跨 Story contract、DAG/file owner/merge order，并按 W1→W2→W3→W4 开始 CP6/CP7 循环。 |
| approve 后会发生什么 | 主编排器把五份 LLD 标记 confirmed；先调度 S01 实现/验证，再 S02，再并行 S03/S04，最后 S05；允许在 Story 文件边界内修改源码和测试并生成实现证据。 |
| approve 不授权什么 | 不授权真实研究运行、真实 lake/NAS/provider/credential、simulation/paper/live/broker/trading、外部 registry/framework、Git remote write、publish/release；不授权 effective-trial/FDR/PBO/DSR/OOS/TCA/capacity 计算或 CR155 lineage backfill。 |
| 不确认会阻塞什么 | 五个 Story 均保持 `confirmed=false`，任何源码/测试实现、CP6、CP7 和未来 ExperimentFamilyManifest producer 均不得启动。 |

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP5-CR163-TRIAL-LINEAGE-INSTRUMENTATION-LLD-CONTEXT.yaml` |
| capsule 状态 | ready-for-cp5-review |
| read_profile | minimal |
| 默认读取策略 | capsule-first；每个 lane 只读分配的 Story/Feature 输入；完整 LLD 仅由 Host 为 CP5 deep review 展开。 |
| 全文档读取扩展 | 5 次，分别绑定 S01..S05 LLD 的 `RE-20260711T035710Z0000-*` 审计事件。 |
| 缺失 / waived 理由 | 无；五个 Story 均为独立 full-lld，checkpoint waiver=0。 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `process/state/STATE.current.json` | scanned | 0 | 0 | CP4 PASS 后无遗留 gate DQ。 |
| CP4 return / result | `process/handoffs/CR163-CP4-META-SE-RETURN-SUMMARY.md` / CP4 result | scanned | 2 | 1 | DAG/file owner/Feature waiver 合并为 DQ-CP5-005。 |
| 三个 meta-dev return summaries | `process/handoffs/CR163-CP5-META-DEV-*-RETURN-SUMMARY.md` | scanned | 5 | 4 | 按 core/producer/consumer/verification 聚合为 DQ-001..004。 |
| 五份 CP5 自动预检 | `process/checks/CP5-CR163-S0*-LLD-IMPLEMENTABILITY.result.json` | scanned | 5 | 5 | 均 PASS；结论进入相应 DQ。 |
| LLD clarification queue | `process/state/QUESTION-LEDGER.ndjson` | scanned | 0 | 0 | 三 lane 无 clarification，blocking_open_count=0。 |
| 下游正式产物 | 五份 full LLD、Feature matrix、三组 Feature design package | scanned | 5 | 5 | 五份 LLD 的核心取舍均进入 DQ。 |
| 用户显式选择题 | CP2/CP3 approvals | scanned | 0 | 0 | 上游 scope/architecture 已 resolved-by-user，不重复询问。 |

### 决策分层

| 分类 | 数量 | 处理方式 |
|---|---:|---|
| 必须用户决策 | 5 | DQ-CP5-CR163-001..005。 |
| 高风险策略确认 | 1 | DQ-005 同时确认源码/测试实现授权边界与 deny-default runtime boundary。 |
| agent 默认处理 | 8 | 具体类名、私有 helper、fixture 文件内组织、错误消息文字等按 LLD 默认实现并留 evidence。 |
| 仅审计记录 | 7 | 5/5 LLD、5/5 precheck、3/3 dispatch lanes、blocking clarification=0、waiver=0。 |

### 设计证据与跨 Story 契约摘要

| Story | 关键冻结内容 | 文件 owner | 开发门 |
|---|---|---|---|
| S01 | 六持久化对象、session façade、9 typed commands、state/count/errors/validation projection | `engine/experiment_family_lineage.py` + contract tests | CP5 approved；无上游 |
| S02 | create-only spec、append-only events、canonical bytes、SHA-256、immutable manifest、full supersession；`sealed_event_count`/`sealed_last_sequence` 保证旧版本可复算 | `engine/experiment_family_lineage_store.py` + store tests | S01 contract confirmed |
| S03 | 两条 orchestration-owned session、CPI-001..004 4/4、wrapper 只传配置、hook 纯计算、post-hook 单 selection writer | two engine paths + two wrappers + producer adapter test | S01/S02 contract confirmed |
| S04 | CR151/CR154/package 三 consumer 共用 validation-bound projection；manual count reconciliation-only；更差状态不可改善 | three existing consumer modules + projection test | S01/S02 contract confirmed |
| S05 | 12 场景、10→1 hash、5 negative classes、tamper/recovery/supersession、13 forbidden counters=0、CR155 blocked/no backfill | integrity/regression test fixtures | S01-S04 ready-for-verification/verified |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP5-CR163-001 | implementation | 是否批准 S01/S02 core contract、canonical primitive owner、sealed event boundary 与 immutable supersession 设计？ | 批准两份 LLD；core 定义 canonical primitive，store 复用；manifest 固化 event count/last sequence。 | 合并 core/store 为单 Story；或回退 CP3 重设对象边界。 | 推荐维持职责分离且旧版本可复算；合并减少文件但扩大单 Story；回退成本最高。 | 决定所有下游 producer/consumer 的稳定 API 与审计真实性。 | 实现发现 canonical bytes 无法由旧 event boundary 复算时立即 NEEDS_DESIGN_CLARIFICATION，重开 CP5。 |
| DQ-CP5-CR163-002 | implementation | 是否批准 S03 的 2/2 chains、4/4 mappings、orchestration 单 session owner 与 post-hook 单 selection writer？ | 批准；wrapper 不开 session、hook 不写 lineage。 | hook 写 selection；或两层各开 session。 | 推荐消除重复 trial/selection；备选更局部但产生 double-count/ownership 歧义。 | 直接决定 `trial_count` 是否是可信系统事实。 | 若真实 hook 变成异步独立进程，停止并新建 transport/ownership ADR/CR。 |
| DQ-CP5-CR163-003 | implementation | 是否批准 S04 只复用 CR151/CR154/package，三者消费同一 validation-bound projection并 fail closed？ | 批准；无 native seal=typed_unavailable，mismatch/tamper=blocked，不建新 gate。 | 各 consumer 重算；或新建 CR163 gate。 | 推荐单一事实、最小 policy 面；重算会漂移；新 gate 重复治理。 | 决定 legacy manual count 是否还能绕过 lineage。 | 未来 backfill/inferred lineage 只能经独立 provenance policy/CR。 |
| DQ-CP5-CR163-004 | implementation | 是否批准 S05 的完整性/恢复/权限/CR155 fixture 设计与 13 个零操作 counter？ | 批准 12/12 场景、5/5 negative classes、10-seal determinism、CR155 blocked/no backfill。 | 缩减为 happy-path 单测；或执行真实研究验证。 | 推荐以 fixture/static 证据覆盖全部风险且不触碰真实 runtime；缩减证据不足；真实运行超授权。 | 决定 tamper、incomplete、supersession 和历史负回归能否独立验证。 | fixture 暴露架构冲突时回 CP5；任何真实数据/runtime 需求走独立授权门。 |
| DQ-CP5-CR163-005 | security | 是否批准五 Story 全量设计并授权按 DAG 开始源码/测试实现，同时维持 deny-default runtime 边界？ | 批准；W1 S01→W2 S02→W3 S03∥S04→W4 S05，逐 Story CP6/独立 CP7。 | 请求具体 LLD 修改；或暂停 CR163。 | 推荐让实现严格受文件 owner/依赖门控制；修改可降低返工但重跑相关 precheck；暂停最安全但不产生价值。 | `approve` 将授权仓库内源码/测试变更，但不会授权真实研究、数据、凭据、交易、发布或统计校正。 | 越过 owner、出现 runtime/data/credential需求、或设计 delta 时立即停止并路由 rework/reopen CP5/新 CR。 |

| 字段 | 内容 |
|---|---|
| 推荐决策 | `approve`：接受 DQ-CP5-CR163-001..005，并启动受控 Story 实现/验证循环。 |
| 备选方案 | `修改: <DQ ID + 修改点>` 路由相关 meta-dev 返工；或 `reject` 回退 CP3/暂停 CR163。 |
| 影响维度 | 审计真实性、接口稳定性、计数正确性、fail-closed、可验证性、文件冲突、权限、安全和交付顺序。 |
| 优劣分析 | 推荐方案是经 CP3/CP4 收敛的最小完整闭环；不引入 DB/registry/runtime，也不把 instrumentation 误写成统计校正。 |
| 风险与回退 | CP6/CP7 按 Story 滚动；任何设计 delta、权限扩大或 fail-closed 破坏均停止并回设计门。 |
| 用户需决策事项 | `DQ-CP5-CR163-001`、`DQ-CP5-CR163-002`、`DQ-CP5-CR163-003`、`DQ-CP5-CR163-004`、`DQ-CP5-CR163-005`。 |

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| CP3 approved / CP4 PASS | 通过 | CP3 checkpoint、CP4 result | 上游门禁有效。 |
| Feature design packages complete | 通过 | FEAT-20/21/22 DESIGN/TEST-PLAN/TASKS | required package 3/3。 |
| Five full LLDs ready-for-review | 通过 | S01..S05 LLD | S02/S03 已按用户审查升级 v1.1。 |
| Five CP5 prechecks PASS | 通过 | CP5 result JSONs | 5/5 PASS。 |
| Blocking clarification=0 | 通过 | QUESTION ledger / context | blocking_open_count=0。 |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---:|---|---|---|---|
| 1 | S01/S02 core contract 与 seal/supersession 可实现 | 通过 | DQ-001 / S01/S02 LLD | 接受推荐方案及 S02 v1.1 hardening。 |
| 2 | S03 两 chain / 四 mapping / no-double-count 可实现 | 通过 | DQ-002 / S03 LLD | 接受推荐方案及 S03 v1.1 CLI/programmatic contract。 |
| 3 | S04 三 consumer / fail-closed / no-new-gate 可实现 | 通过 | DQ-003 / S04 LLD | 接受推荐方案。 |
| 4 | S05 integrity/recovery/permission/CR155 regression 可验证 | 通过 | DQ-004 / S05 LLD | 接受推荐方案。 |
| 5 | DAG、file owner、merge order 与实现授权边界可接受 | 通过 | DQ-005 / scoped plan | 授权仓库内源码/测试实现；deny-default runtime 边界不变。 |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| 五份 LLD 可标记 confirmed | 通过 | 本 checkpoint | 5/5 confirmed。 |
| dev_gate 可按 DAG 计算 | 通过 | scoped plan / Story cards | S01 首先进入 dev-ready。 |
| 用户明确批准/修改/拒绝 | 通过 | 人工审查结果 | 用户指示整改后批准 CP5。 |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| CP4 planning package | `process/checks/CP4-CR163-TRIAL-LINEAGE-INSTRUMENTATION-STORY-DAG-PARALLEL-SAFETY.result.json` | 通过 | PASS。 |
| 五份 full LLD | `process/stories/STORY-CR163-S0*-LLD.md` | 通过 | 5/5 confirmed。 |
| 五份 CP5 precheck | `process/checks/CP5-CR163-S0*-LLD-IMPLEMENTABILITY.result.json` | 通过 | 5/5 PASS。 |
| CP5 context / queue | `process/context/CP5-CR163-TRIAL-LINEAGE-INSTRUMENTATION-LLD-CONTEXT.yaml` / QUESTION ledger | 通过 | context OK，blocking=0。 |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-07-11T12:25:00+08:00
- 修改意见：S02/S03 已按审查意见整改并复验；接受 DQ-CP5-CR163-001..005，按 DAG 自动推进至下一人工门禁。
- 风险接受项：无默认风险接受；approve 仅授权仓库内 Story 范围源码/测试实现，不授权任何真实 runtime/data/credential/statistical/backfill/external/release 操作。
