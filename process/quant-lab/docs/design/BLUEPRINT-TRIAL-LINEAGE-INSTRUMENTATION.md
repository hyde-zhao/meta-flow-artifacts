---
status: draft-for-cp3
version: "0.1"
cr_id: "CR-163"
source_story_map: "docs/product/STORY-MAP.md"
source_mvp_scope: "docs/product/MVP-SCOPE.md"
confirmed_by: ""
confirmed_at: ""
---

# Blueprint: Trial Lineage Instrumentation

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 0.1 | 2026-07-11 | meta-se-critical | 初始 CP3 蓝图；冻结能力边界、2 条 producer chains / 4 mappings、数据归属、consumer 方向和五 Story 输入。 |

## 1. 蓝图适用性

CR163 同时跨越 family lifecycle、append-only 记录、确定性封存、两个 producer chain、单次 run identity 与既有 admission consumer，存在共享数据 owner 与禁止反向依赖，因此必须生成独立 Blueprint / Domain Map / Dependency Map。核心产物仍只有一个：`ExperimentFamilyManifest` 及其 lineage lifecycle；producer instrumentation 和 admission projection 都是该核心产物的边界适配，不拆 companion HLD。

## 2. 能力地图

| Capability ID | 能力域 | 用户价值 | 覆盖 Story | Owner Feature |
|---|---|---|---|---|
| CAP-CR163-01 | Family declaration and identity | 在搜索前固定 family 与 search-space 承诺，阻止事后缩小 | CR163-S01 | FEAT-CR163-LINEAGE |
| CAP-CR163-02 | Append-only trial recording | 保留 trial / attempt / terminal / selection 的完整历史 | CR163-S02 | FEAT-CR163-LINEAGE |
| CAP-CR163-03 | Producer coverage | 两条真实候选生产链及其 hook 均不能绕过 lineage | CR163-S03 | FEAT-CR163-PRODUCER-ADAPTERS |
| CAP-CR163-04 | Deterministic seal and validation | 生成可复算、不可原地修改、可 supersede 的 manifest | CR163-S01, CR163-S02 | FEAT-CR163-LINEAGE |
| CAP-CR163-05 | Admission evidence projection | 向 CR151/CR154/package 暴露 fail-closed raw lineage 输入 | CR163-S04 | FEAT-CR163-ADMISSION-ADAPTER |
| CAP-CR163-06 | Integrity and negative regression | 证明 count/tamper/recovery/permission/CR155 边界 | CR163-S05 | FEAT-CR163-VALIDATION |

## 3. Feature / Epic 边界

| Feature ID | 名称 | 负责事项 | 不负责事项 | 拥有数据 | 只读数据 | 禁止依赖 |
|---|---|---|---|---|---|---|
| FEAT-CR163-LINEAGE | Experiment-family lineage core | Spec、Trial、Attempt、Selection、Manifest、Validation；状态、count、seal、supersession | candidate 计算、统计校正、单次 run manifest、admission policy | family spec；JSONL events；sealed manifest versions；validation result | run_id / experiment_id / artifact refs | lake/provider/runtime；CR151/CR154 policy internals；producer-specific candidate classes |
| FEAT-CR163-PRODUCER-ADAPTERS | Producer instrumentation adapters | 在两条 chain 的 orchestration 与 hook 边界调用统一 lifecycle contract；4/4 mapping | lineage 数据定义、hash 算法、admission 决策 | 无长期数据；仅产生 command | family contract；producer inputs；candidate refs | 直接写 sealed manifest；按 wrapper/hook 调用次数计 trial；反向调用 consumer |
| FEAT-CR163-ADMISSION-ADAPTER | Existing admission projection | 将 validation 结果投影为 availability/ref/raw count；接入 CR151/CR154/package | 新 gate、effective count、FDR/PBO/DSR、修复 lineage | 无 lineage 真相；只拥有 consumer projection payload | sealed manifest；validation result | 直接读取未封存 JSONL；接受手填 count 作为 `present`；更改 runtime auth flags |
| FEAT-CR163-VALIDATION | Fixture/static verification | 12/12 P0 场景、4/4 mapping、tamper/recovery/CR155/permission 验证 | 真实数据/runtime 或历史回填 | fixture 与验证证据（后续 CP5+） | 所有公开 contract | 生产数据、凭据、外部写入 |

数据归属唯一性：只有 `FEAT-CR163-LINEAGE` 可写 lineage 真相。Producer adapter 只能提交命令，consumer adapter 只能消费 sealed projection，validator 只产生 validation result，不改历史。

## 4. 跨 Feature 流程

| Flow ID | 触发 | 参与 Feature | 数据写入 Owner | 失败路径 | 验证入口 |
|---|---|---|---|---|---|
| FLOW-CR163-01 | producer chain 在首个 trial 前启动 family session | PRODUCER-ADAPTERS → LINEAGE | LINEAGE | 缺 declaration：post-hoc=`blocked`；完全未 instrumentation=`typed_unavailable` | SC-CR163-P01, N01 |
| FLOW-CR163-02 | trial / retry / selection lifecycle | PRODUCER-ADAPTERS → LINEAGE | LINEAGE | identity conflict、orphan、非法转换均 append rejection evidence 并 block seal | P02, N02, B01, F01 |
| FLOW-CR163-03 | producer 完成搜索并请求 seal | PRODUCER-ADAPTERS → LINEAGE | LINEAGE | completeness/count/hash/chain 任一失败，不产生合法 present manifest | P03, R01, T01 |
| FLOW-CR163-04 | admission consumer 请求 family evidence | LINEAGE → ADMISSION-ADAPTER → existing consumers | LINEAGE（projection 无真相写入） | uninstrumented=`typed_unavailable`；invalid/tampered=`blocked`；legacy count 仅 reconciliation | P03, B02, G01 |

## 5. Producer Inventory Contract

| Mapping | 去重 chain | 接入点 | 强制契约 | 防双计数规则 |
|---|---|---|---|---|
| CPI-CR163-001 | Chain A public Stage 3 | public wrapper → `run_stage3_mature_multifactor_research` | orchestration 持有一个 `FamilyLineageSession` | wrapper 只传 session/context，不新建 trial |
| CPI-CR163-003 | Chain A public Stage 3 | `build_strategy_candidate` hook | 记录 selection/candidate artifact ref；验证 session 存在 | hook 不能按函数调用新建 trial；trial_id 来自 declared trial |
| CPI-CR163-002 | Chain B legacy CR039 | wrapper → `run_strategy_research` | orchestration 持有一个 `FamilyLineageSession` | wrapper 与 orchestration 共用 family_id |
| CPI-CR163-004 | Chain B legacy CR039 | `build_strategy_candidates` hook | 每个 stable parameter+seed identity 对应一个 declared trial；记录 selection | 列表构造数量不等于 attempts；重复 delivery 幂等 |

覆盖分母固定为 2 条去重 chain、4 个 mapping；S03 必须一次覆盖 4/4，不能拆成第六个 Story。

## 6. 共享能力与相邻契约

| Shared ID | 名称 | 使用方 | Owner | 调用方向 / 时机 / 触发 | 输入 | 输出 / 后续衔接 | 降级策略 / 调用方修改 |
|---|---|---|---|---|---|---|---|
| SH-CR163-01 | `FamilyLineageSession` façade | 两条 producer orchestration | LINEAGE | producer → session；搜索开始前 open，搜索完成后 seal | FamilySpec、run/experiment ids、artifact root；typed event commands | event receipts、sealed manifest ref、validation；交给 consumer projection | session 缺失时不得伪造 present；producer 调用点需显式传递 session |
| SH-CR163-02 | Event command contract | producer hooks / façade | LINEAGE | façade/hook → recorder；每次状态变化 | event_id、family_id、entity id、sequence、payload | idempotent receipt 或 conflict block | 同 id 同 payload 幂等；同 id 异 payload block |
| SH-CR163-03 | Seal / validate contract | producer end、auditor、consumer adapter | LINEAGE | session → sealer；consumer → validator | spec + full event stream + prior supersession refs | immutable manifest + ValidationResult | 任一 check 非 PASS 均不可 present |
| SH-CR163-04 | Admission evidence projection | CR151/CR154/package adapters | ADMISSION-ADAPTER | existing pipeline → projection，在 gate evaluation 前 | manifest ref + validation result | availability、raw count、ref、blocked reasons；existing gate继续取最差状态 | 无 lineage=`typed_unavailable`；无效=`blocked`；不触碰 effective fields |
| SH-CR163-05 | Single-run identity link | lineage core | existing run contracts | lineage → read-only identity | `run_id`, `experiment_id`, artifact refs | Manifest 中的 run links | 不修改 `ExperimentManifest` 的单次 run 语义 |

## 7. 明确禁止依赖

- admission consumer 不得写、补齐或修复 lineage；否则 policy 层成为事实 owner。
- producer hook 不得直接写 sealed JSON；否则绕过 validation 与 supersession。
- family manifest 不得嵌入或替代单次-run `ExperimentManifest`；二者只通过 identity/ref 相连。
- legacy 手工 `trial_count` 不得作为 `present` 来源；最多与 sealed raw count 对账，不一致即 blocked。
- `effective_trial_count` 不得由 raw count复制；其 availability 维持 `typed_unavailable`，ref/method 为空。

## 8. 五 Story CP4 输入

| Candidate Story | Outcome | 依赖 | Likely file owner（建议） | lld_policy | 完成边界 |
|---|---|---|---|---|---|
| CR163-S01 | 公共对象、状态与 validator contract 可稳定消费 | 无 | 新 lineage contract/validator module | full-lld | 六对象、状态、错误码、availability 投影被冻结 |
| CR163-S02 | append-only recorder、canonical seal、supersession 可确定复算 | S01 | lineage storage/seal module | full-lld | JSON/JSONL、hash、immutable seal、recovery 全闭环 |
| CR163-S03 | 两条 chain / CPI-001..004 4/4 接入且无双计数 | S01, S02 | 两个 producer engine + 两个 wrapper/hook owner | full-lld | S03 单 Story；4/4 mapping，2/2 pre-search declaration |
| CR163-S04 | 既有 CR151/CR154/admission package 消费 projection | S01, S02 | three existing consumer modules + package adapter | full-lld | 无新 gate；manual legacy count fail-closed/reconciliation-only |
| CR163-S05 | 12 场景 integrity/recovery/permission/CR155 证据 | S01-S04 | tests/fixtures/evidence owner（CP5 后） | technical-note（若跨模块 fixture 复杂则升级 full-lld） | 12/12 P0、4/4 mapping、CR155 1/1 blocked |

建议 Wave：W1=S01；W2=S02；W3=S03 与 S04 可在文件 ownership 不冲突时并行；W4=S05。正式 DAG 与文件清单由 CP4 写入，不在本轮修改 `DEVELOPMENT-PLAN`。

## 9. 待确认边界

| Decision ID | 决策类型 | 问题 | 推荐方案 | 备选方案 | 推荐 / 备选优劣 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP3-CR163-001 | architecture | recorder API 形态 | lifecycle session façade + 内部 event commands | 纯 event-command API；纯 session snapshot API | 推荐兼顾 producer 易用与 append-only 审计；纯 command 调用负担高；纯 snapshot 难保事件历史 | producer 接入、测试与维护 | 非 Python/分布式 producer 增多时公开纯 command transport |
| DQ-CP3-CR163-002 | architecture | storage 与 seal 协议 | 每 family/version 的 immutable spec JSON + events JSONL + manifest JSON + validation JSON | 单一 snapshot JSON；SQLite/registry | 推荐最小、可 diff、无服务依赖；snapshot 丢事件；DB 超范围 | 确定性、并发、恢复 | 多进程并发或规模超过本地原子追加能力时切 SQLite/registry 新 CR |
| DQ-CP3-CR163-003 | architecture | legacy 手填 count | 仅 reconciliation；无合法 sealed lineage 时 typed_unavailable，有冲突 blocked | 接受为 present；完全忽略 | 推荐防后验造数且保兼容诊断；接受会破坏目标；忽略会丢冲突信号 | CR151/CR154 compatibility | 仅独立 backfill CR 定义 inferred provenance 后再扩展 |
| DQ-CP3-CR163-004 | security | 授权边界 | CP3 只批准设计，后续仍走 CP4/CP5/CP6/CP7/CP8 | 暂停；另起 runtime CR | 推荐允许离线 contract/fixture 继续，不扩大权限 | 防止 CP3 被误解为实现/runtime 授权 | 任何真实 data/runtime/credential/external write 需求立即 BLOCKED |

## 10. Gotchas

- wrapper 与 construction hook 是 4 个 instrumentation mappings，但只有 2 条 producer chains；把 mapping 数当 trial 数会系统性膨胀 raw count。
- retry 是 Attempt，不是 Trial；不同 seed 即使参数相同仍是不同 Trial。
- `excluded` 是保留并计数的 terminal trial，不是从 family 删除。
- uninstrumented 与 invalid 不同：前者在没有伪造记录时是 `typed_unavailable`，后者（post-hoc/incomplete/tampered/conflict）必须 `blocked`。
- seal hash 不能包含文件路径、mtime、写入顺序、当前时间或 manifest 自身 hash 字段。
- supersession 不能覆写旧文件或只改“latest”指针而丢 prior ref/hash；consumer 的 latest resolution 也必须验证整条链。
- present raw lineage 不等于 C1 computed；effective count/ref/method 必须继续 unavailable/empty。

