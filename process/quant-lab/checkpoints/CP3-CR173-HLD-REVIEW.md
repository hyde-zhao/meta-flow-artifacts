---
checkpoint: CP3
checkpoint_id: CP3-CR173-HLD-REVIEW
cr_id: CR-173
title: Effective-Trial Offline Methodology — HLD Review
status: approved
gate_profile: standard-code-critical
opened_at: "2026-07-16T13:40:00+08:00"
reviewed_by: "user"
reviewed_at: "2026-07-16T15:11:03+08:00"
auto_check_result: process/checks/CP3-CR173-HLD-CONSISTENCY.result.json
context_ref: process/context/CP3-CR173.context.json
route_plan_ref: process/checks/CP0-CR173.route-plan.json
target:
  phase: solution-design
  artifacts:
    - docs/design/BLUEPRINT-EFFECTIVE-TRIAL-OFFLINE-METHODOLOGY.md
    - docs/design/DOMAIN-MAP-EFFECTIVE-TRIAL-OFFLINE-METHODOLOGY.md
    - docs/design/DEPENDENCY-MAP-EFFECTIVE-TRIAL-OFFLINE-METHODOLOGY.md
    - docs/design/HLD-EFFECTIVE-TRIAL-OFFLINE-METHODOLOGY.md
    - docs/design/ARCHITECTURE-DECISION-EFFECTIVE-TRIAL-OFFLINE-METHODOLOGY.md
---

# CP3 Effective-Trial Offline Methodology — HLD Review

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | Waiver | 说明 |
|---|---:|---:|---:|---|
| `process/checks/CP3-CR173-HLD-CONSISTENCY.result.json` | PASS | 0 | 0 | 18 项检查：17 PASS、1 N/A-with-reason；8/8 requirements、3/3 simulations、两项 CP3 设计义务和 CR-172 前置归属均已收敛。 |
| `process/checks/CR173-SCOPE-AUTHZ-CONSISTENCY-OVERRIDE.json` | PASS | 0 | 0 | 否定语句推断误报已纠正；结构化 `required_capabilities=[]`、`required_evidence=[]` 为权威值，未扩大授权。 |

## Decision Brief

### 审批者摘要

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 确认 CR-173 是否采用可复现、non-alias、fixture-only 的 `spectral_participation_ratio` estimator，并决定 public C1 projection 是否因公共 contract 非兼容而拆出。 |
| 推荐动作 | `approve` 下方两项推荐：批准限定 claim 的 estimator；批准 estimator-only split，并把 projection 登记为后续 CR candidate。 |
| approve 后会发生什么 | 进入 CP3 后的 story-planning：只规划 standalone estimator、七字段 evidence 和 golden vectors；登记 public C1 versioned projection 后续候选但不创建正式 CR；随后 CP4 自动预检并进入 CP5 全量设计证据人工门禁。 |
| approve 不授权什么 | 不授权 Story 之外的提前实现，不授权真实数据/策略、matrix producer、公共 C1 写入、Gate-1 blocker 解除、runtime、trading、publish/deploy 或 Git remote write；CP5 前实现数必须为 0。 |
| 不确认会阻塞什么 | 阻塞 estimator 的 Story/Feature 实现设计与 CP4/CP5；现有 effective count 和全部公共 C1 consumers 继续 `typed_unavailable`。 |

### Context Capsule Summary

| 项 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP3-CR173.context.json`；审计视图为 `process/context/CP3-CR173-CONTEXT.yaml`。 |
| read_profile | compact。 |
| 默认读取策略 | capsule-first；先读最小上下文，只有在字段不足、设计深审或公共 contract 审计时扩展。 |
| 全文档读取 | CP3 深度评审需要展开 CR173 产品章节、场景/矩阵、C1 公共 contract 源码与回归调用面；所有扩展均已写入 `READ-EXPANSION-LEDGER.ndjson`。 |
| 只读源码结论 | public C1 生产路径 `8/8`、回归/authorization 路径 `12/12` 已分类；源码修改数和测试源码修改数均为 0。 |
| 禁止操作 | real lake/NAS/provider/credential、real computation、runtime/trading、public C1 contract write、Git remote write 均为 0。 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| CP2 已批准基线 | 8 个 DQ、2 个 CP3 obligations | scanned | 10 | 0 | 作为冻结输入，不重开 CP2。 |
| Architecture Gray Areas | 4 个 AGA、4 个候选路径 | scanned | 4 | 2 | 方法选择与公共 contract split 需要人工确认。 |
| Blueprint / HLD / ADR | 5 份 companion 设计产物 | scanned | 5 ADR | 2 | ADR-001/002/003/005 合并为方法 bundle；ADR-004 形成 split 决策。 |
| 源码 contract inventory | 8 个生产路径 + 12 个回归/授权路径 | scanned | 20 | 0 | 形成 `cross-owner + cross-domain + non-compatible` 事实，不作为自由裁量项。 |
| CP3 自动结果 | 18 items | scanned | 2 | 2 | 17 PASS、1 N/A-with-reason；等待本人工门禁。 |
| 非阻断开放项 | O-CR173-001/002 | scanned | 2 | 0 | future activation / future projection CR；当前不阻断 estimator-only。 |
| Story/LLD/实现 | 下游阶段 | N/A | 0 | 0 | CP3 批准前不得创建或执行。 |

### 决策分层

| 层级 | 数量 | 项目 |
|---|---:|---|
| 必须用户决策 | 2 | `DQ-CR173-CP3-001`、`DQ-CR173-CP3-002`。 |
| 高风险策略确认 | 2 | 二阶有效维度不得提升为 FWER/DSR/admission calibration；public projection 不得双写或跨 owner 静默落地。 |
| agent 默认处理 | 5 | companion 命名、exact-rational canonical domain、结构化 status、append-only recovery、current index 不覆盖。 |
| 仅审计记录 | 2 | future empirical-matrix stability、future public schema migration。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| `DQ-CR173-CP3-001` | architecture | 是否批准 `spectral_participation_ratio` estimator 及其限定 claim？ | 批准 `n_eff=(tr R)^2/tr(R²)`；只解释为 sealed-trial 二阶相关结构的 effective dimensionality。 | 转 methodology Spike，并让 effective count 全量保持 `typed_unavailable`；未来另评 alpha-specific copula/Li–Ji 类方法。 | 推荐方案在显式有效 `R` 上单值可识别、严格满足 `1≤n_eff≤n`、无随机积分且可确定验证；代价是不能解释 tail/FWER/DSR。备选避免模型误用，但暂不交付 offline count。 | 主要风险是把二阶有效维度误读为 error-rate 或 admission calibration；真实矩阵采样误差仍未解决。 | 若业务要求 alpha/tail/FWER calibrated effective tests、methodology owner 不接受二阶 estimand、或不能提供 sealed matrix，则停止当前 claim，转 Spike 并保持 unavailable。 |
| `DQ-CR173-CP3-002` | scope | 是否批准 CR-173 收缩为 estimator-only，并把 public C1 projection 拆为后续 CR candidate？ | 批准 split：CR-173 只生成 standalone 七字段 evidence；current C1/Gate-1/admission 继续 unavailable。 | 暂停整个 CR-173，既不做 estimator 也不做 projection。 | 推荐方案保留离线方法价值，并把 `int|None`/4-field → decimal/7-field、跨 4 owner 域的迁移风险隔离；代价是 CR-173 CP8 仍不能让公共 C1 computable。备选回滚面最小，但会悬置方法风险。 | 若错误留在本 CR，会出现双真相、破坏 lineage invariant、Gate-1/admission worst-state 和 CR155 regression。 | 只有后续 CR 具备 C1 owner approval、versioned schema、trust binding、old/new migration、无 dual truth、8+12 回归与 rollback，才可重新启动 projection。 |

### 两项评审整改的 CP3 结论

| 义务 | 结论 | 证据 | 对范围的影响 |
|---|---|---|---|
| `DO-CR173-CP3-001` | `PASS` | HLD §3–§5；ADR-001..003 | 不转 Spike；但 claim 永久限定为二阶 effective dimensionality。 |
| `DO-CR173-CP3-002` | `PASS_BY_SPLIT` | Dependency Map public C1 inventory；HLD §10/§14；ADR-004 | 推荐把 CR-173 收缩为 estimator-only；projection Story/实现数为 0。 |

### CR-172 前置条件归属与能力预期

| 层面 | CR-173 是否负责 | CR-173 CP8 后状态 | 后续 owner / 门禁 |
|---|---:|---|---|
| offline estimator、non-alias standalone schema、deterministic golden vectors | 是 | `offline_method_ready=true` 可被终验；只表示二阶 effective dimensionality 方法就绪 | CR-173 methodology owner |
| five fields=`5/5`、data owner identity、fresh conflict precheck | 否 | 仍待满足；缺任一项不得重开 activation CP2 | CR-172 data/activation lane |
| CR-172 PATH-C/A、strategy identity、PIT/lineage/run identity、real producer binding | 否 | 未授权、未实施 | CR-172 runtime-high-risk gates |
| public C1 projection / Gate-1 blocker / admission consumer | 否 | 继续 `typed_unavailable` / blocked | future public-C1 projection CR，C1 contract owner |
| `c1_computable=true` | 否 | **仍 false** | 需 CR-173 methodology + CR-172 activation/binding + future projection/migration 三层同时满足 |

Public projection 不是重开 CR-172 CP2 的前置。CR-173 完成、five fields/data owner/fresh precheck 满足后，可恢复 CR-172；若 projection 尚未完成，CR-172 必须执行既有 `DQ-CR172-003` 降级路径：`effective_trial_count=typed_unavailable`、`c1_computable=false`。

### CP5 / CP7 / CP8 下游防误用检查

| 检查项 | 目标值 | 失败路由 |
|---|---:|---|
| `estimation_method/version/hash/二阶维度 limitation` 完整 | `4/4` | 回退 CP5 设计或 CP7 `NEEDS_REWORK` |
| 新结果进入 public C1/Gate1/DSR/admission 的生产调用 | `0` | `NEEDS_DESIGN_CLARIFICATION` / `NEEDS_REWORK` |
| 把 participation ratio 声明为 Li-Ji、BH/FWER/Šidák/DSR/admission calibration | `0` | blocker，不得进入 CP8 |
| CP8 public claim | `public_effective_trial_count_populatable=false`、`c1_computable=false` | CP8 `NOT_READY` |
| future projection schema | 必须评估 `effective_dimensionality` 或等价 versioned discriminator，禁止双真相 | 后续 CR 人工门禁 |

### 用户需决策事项

| 字段 | 内容 |
|---|---|
| 本轮待人工决策项 | 2。 |
| 推荐回复 | `approve`，表示同时接受两项推荐。 |
| approve 后下一人工门禁 | CP5；在此之前 Host 自动完成 estimator-only 的 Story/Feature 实现设计准备和 CP4 自动预检。 |
| 修改 | `修改: DQ-CR173-CP3-001=<具体修改>` 或 `修改: DQ-CR173-CP3-002=<具体修改>`。 |
| reject | CR-173 保持 CP3，不创建 Story、不实现；effective count 与 public C1 继续 unavailable。 |

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| CP2 已人工批准 | PASS | CP2 checkpoint/result/GATE ledger | 8/8 产品决策已冻结；本轮不重开。 |
| CP3 capsule 可用 | PASS | `process/context/CP3-CR173.context.json` | capsule-first，扩展读取有 ledger。 |
| meta-se-critical 真实调度完成 | PASS | CP3 handoff、dispatch/handoff ledgers、return summary | canonical role/meta-se-critical/critical profile 可追溯。 |
| Blueprint/HLD/ADR/discussion 完整 | PASS | Deliverables | 5+2 设计/讨论产物齐备。 |
| CP3 自动预检无 blocker | PASS | CP3 result | blocker=0、waiver=0。 |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---:|---|---|---|---|
| 1 | 目标与 success criteria 均可量化 | PASS | HLD §1.2 | 用户批准；100%、5/5、7/7、6×3、8+12、forbidden=0。 |
| 2 | 至少两个真实方法方案与切换条件 | PASS | HLD §2–§3 | 用户批准推荐方案；participation ratio、alpha-specific copula、Spike 治理回退。 |
| 3 | Estimand/输入/有效域可识别 | PASS | HLD §4；ADR-001/002 | 用户接受限定 claim；公式、PSD 域、`[1,n]` 证明与 bias ceiling 完整。 |
| 4 | 数值、PSD、舍入和 canonical hash 唯一 | PASS | HLD §4.2/§5.2；ADR-003 | exact base-10 rational、deterministic LDLᵀ、无 tolerance、无 float。 |
| 5 | 七字段 schema 与 fail-closed | PASS | Domain Map；ADR-003/005 | present 7/7；失败 count=null；raw fallback=0。 |
| 6 | Public C1 touch 100% 分类 | PASS | Dependency Map；HLD §10 | 8/8 生产 + 12/12 回归/授权。 |
| 7 | Projection split 与 owner/迁移契约可操作 | PASS | HLD §10/§14；ADR-004 | 用户批准 split；本 CR 公共修改=0；后续 CR 前置完整。 |
| 8 | Use Case → Architecture trace 完整 | PASS | HLD §7 | REQ-001..008=8/8。 |
| 9 | 关键场景和失败/回退路径 | PASS | HLD §8–§9 | 3/3 design simulations；Spike/split/append-only route。 |
| 10 | HLD/ADR/Risk/NFR/Blueprint 内部一致 | PASS | HLD §11–§17；ADR matrix | 无 positive public C1 或 FWER overclaim。 |
| 11 | 相邻对象边界清楚 | PASS | Blueprint/Dependency Map | lineage owner、method owner、future C1 owner、activation owner 分离。 |
| 12 | Gotchas 与理论依据可追溯 | PASS | HLD §3.4/§17；各 companion Gotchas | participation-ratio 与 Li–Ji 原始研究边界已说明。 |
| 13 | Story/LLD/实现/真实操作保持 0 | PASS | CP3 operation counts | CP3 不越权。 |
| 14 | CR-172 前置条件与 C1 computable 条件未混合 | PASS | CR-172 §PATH-B 恢复前置与 C1 computable 条件分层；CR-173 §CR-172 前置条件归属审计 | 用户知情批准；CR-173 只关闭 methodology prerequisite；public projection 不阻塞重开 CP2，但阻塞 C1 computable。 |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| CP3 auto precheck PASS | PASS | CP3 result | 0 blockers、0 waivers。 |
| 两项 CP3 decisions 已完整呈现 | PASS | Decision Brief | 推荐、备选、影响、风险和切换条件齐备。 |
| Estimator-only HLD 可馈入后续 Story planning | PASS | HLD/ADR/Blueprint；本人工审查结果 | 用户已批准进入 Story planning。 |
| Public projection 不会泄漏进 CR-173 Story | PASS | HLD §14；ADR-004；本人工审查结果 | 用户已批准 estimator-only split。 |
| Unauthorized scope 明示 | PASS | HLD §1.3；本 checkpoint | 全部操作计数为 0。 |

## Deliverables

| 交付物 | 路径 | 审查结果 | 说明 |
|---|---|---|---|
| CP3 context | `process/context/CP3-CR173.context.json` | PASS | compact capsule。 |
| Blueprint | `docs/design/BLUEPRINT-EFFECTIVE-TRIAL-OFFLINE-METHODOLOGY.md` | PASS | 用户批准 capability / owner boundary。 |
| Domain Map | `docs/design/DOMAIN-MAP-EFFECTIVE-TRIAL-OFFLINE-METHODOLOGY.md` | PASS | 用户批准 objects / states / numeric schema。 |
| Dependency Map | `docs/design/DEPENDENCY-MAP-EFFECTIVE-TRIAL-OFFLINE-METHODOLOGY.md` | PASS | 用户批准 8+12 public C1 inventory 与 split。 |
| HLD | `docs/design/HLD-EFFECTIVE-TRIAL-OFFLINE-METHODOLOGY.md` | PASS | 用户批准 estimator-only 推荐架构及限定 claim。 |
| ADR | `docs/design/ARCHITECTURE-DECISION-EFFECTIVE-TRIAL-OFFLINE-METHODOLOGY.md` | PASS | 用户批准 ADR-001..005。 |
| discussion log | `process/discussions/CP3-CR173-HLD-DISCUSSION-LOG.md` | PASS | 4 AGA、4 options、3 simulations。 |
| discussion checkpoint | `process/checks/CP3-CR173-DISCUSSION-CHECKPOINT.json` | PASS | completed-awaiting-host-gate。 |
| CP3 result | `process/checks/CP3-CR173-HLD-CONSISTENCY.result.json` | PASS | 18 items：17 PASS、1 N/A-with-reason。 |
| scope/authz consistency | `process/checks/CR173-SCOPE-AUTHZ-CONSISTENCY-OVERRIDE.json` | PASS | 纠正否定语句误推断；后续 status-sync 需重应用。 |
| meta-se return | `process/handoffs/CR173-CP3-META-SE-RETURN-SUMMARY.md` | PASS | delegated stage completion evidence。 |
| meta-pm scope alignment return | `process/handoffs/CR173-CP3-META-PM-SCOPE-ALIGNMENT-RETURN-SUMMARY.md` | PASS | 8 个产品文档 estimator-only / public-boundary / CR172 前置归属对齐。 |
| launch message | `process/checks/CP3-CR173-HUMAN-GATE-LAUNCH-MESSAGE.md` | PASS | Host 已生成并通过 human-gate launch 校验。 |

## 人工审查结果

- 状态：`approved`
- 审查人：`user`
- 审查时间：`2026-07-16T15:11:03+08:00`
- 决定：批准 `DQ-CR173-CP3-001`（采用 spectral participation ratio，claim 仅限二阶 effective dimensionality）与 `DQ-CR173-CP3-002`（estimator-only，public C1 projection 拆为后续 candidate）。
- 风险接受：接受二阶模型的限定 claim、public projection split，以及 CR-173 CP8 后 public C1 仍 `typed_unavailable` / `c1_computable=false` 的预期；不包含任何真实数据、runtime、公共 contract 写、实现执行或 Git remote write 授权。
- 用户指令：`批准CP3，并推进到下一个人工门禁`
