---
status: draft-awaiting-cp3
version: "1.0"
change_id: CR-173
source_story_map: docs/product/STORY-MAP.md#cr173-product-planning-outcomes非正式-story
source_mvp_scope: docs/product/MVP-SCOPE.md#cr173-mvp-scope
selected_boundary: estimator-only
confirmed_by: ""
confirmed_at: ""
---

# Blueprint：Effective-Trial Offline Methodology

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-16 | meta-se-critical | 建立 CR-173 estimator-only 能力边界；将跨 owner、跨域且非兼容的 public C1 projection 隔离为后续 CR 候选。 |

## 蓝图适用性

CR-173 同时涉及方法学、证据语义与既有公共 C1 consumer，存在数据归属和禁止依赖问题，蓝图适用。本文只定义能力边界，不创建正式 Feature、Epic、Story、DAG、Wave、LLD 或实现任务。

## 能力地图

| Capability ID | 能力域 | 用户价值 | CR173 outcome | Owner 边界 |
|---|---|---|---|---|
| CAP-ET-01 | sealed-trial dependency input contract | 保证 effective count 的输入不是从 raw count 猜测而来 | CR173-O01 | methodology owner；lineage owner 只提供只读 sealed identity |
| CAP-ET-02 | spectral participation-ratio estimator | 把显式二阶依赖结构转为可审计的独立维度等价量 | CR173-O01/O03 | methodology owner |
| CAP-ET-03 | seven-field standalone typed evidence | 区分 present、typed_unavailable 与 blocked，并携带方法与 provenance | CR173-O02 | methodology owner |
| CAP-ET-04 | deterministic golden-vector contract | 用 6 类 × 3 次固定向量证明规范化与结果稳定 | CR173-O03 | methodology owner / independent verifier future consumer |
| CAP-ET-05 | public C1 touch classification | 防止 estimator 通过隐式 alias 污染既有 C1 consumer | CR173-O04 的 CP3 disposition | C1 evidence contract owner；CR-173 只读审计，不写公共 contract |

## 能力边界

| Boundary ID | 名称 | 负责事项 | 不负责事项 | 拥有数据 | 只读数据 | 禁止依赖 |
|---|---|---|---|---|---|---|
| BDY-ET-INPUT | Offline dependency contract | 接收显式、已 sealed trial IDs 对齐的 canonical correlation matrix；验证完整性、矩阵域与 provenance | 不读取 lake/NAS，不估计 correlation matrix，不推断 strategy identity | CR173 input envelope schema、canonical input hash | sealed family ref/hash、raw count、ordered trial IDs | 真实 producer、provider、credential、历史目录推断 |
| BDY-ET-METHOD | Estimator core | 计算 `spectral_participation_ratio`，冻结输出范围、舍入与 serialization | 不做 FWER/DSR/tail-copula 校准，不生成 admission 决策 | method ID/version/hash、computation identity | 通过 BDY-ET-INPUT 的 normalized input | public C1 gate、aggregate、runtime |
| BDY-ET-EVIDENCE | Standalone typed evidence | 生成七个顶层字段与结构化 status/reason，append-only 表达恢复 | 不向现有 FamilyEvidenceProjection/StatisticalEvidenceSummary 写入正向 truth | standalone effective-trial evidence | estimator result、input lineage | raw alias、默认补值、竞争 gate |
| BDY-C1-PROJECTION | Public C1 projection boundary | 本轮仅分类 100% call-path 与兼容性 | 不在 CR-173 修改、实现或发布公共 C1 contract | 由 C1 evidence contract owner 持有 | CR-173 只读 inventory | estimator owner 直接写公共 consumer |

## 数据归属

| 数据对象 | 唯一 Owner | CR-173 权限 | 写入规则 |
|---|---|---|---|
| sealed family lineage / trial IDs | experiment-family lineage owner | read-only fixture/static contract | CR-173 不重放、不修复、不写 store |
| sealed-trial correlation matrix | 上游 dependency-evidence producer；本 CR 仅 fixture author | consume-only | 必须显式提供并与 trial IDs 1:1 对齐；CR-173 不估计矩阵 |
| method specification and hash | methodology owner | own | ID=`spectral_participation_ratio`；版本与 canonical specification 一起 hash |
| standalone seven-field evidence | methodology owner | own | present 必须 7/7 且 non-alias；失败时 count=null |
| FamilyEvidenceProjection / C1InputStatus | lineage/C1 contract owner | read-only audit | 当前 CR 禁止修改；后续 projection CR 才能演进 |
| StatisticalEvidenceSummary / Gate-1 / admission package | statistical/reliability/admission owners | read-only audit | 不得由 estimator-only CR 跨 owner 直接改写 |

## 跨边界流程

| Flow ID | 触发 | 参与边界 | 数据写入 Owner | 失败路径 | 验证入口 |
|---|---|---|---|---|---|
| FLOW-ET-01 | CP5 后的 repository fixture | BDY-ET-INPUT → BDY-ET-METHOD → BDY-ET-EVIDENCE | methodology owner | 缺失/不足=`typed_unavailable`；矛盾/篡改=`blocked`；raw fallback=0 | SC-CR173-P01/F01/N01/B01/D01 |
| FLOW-ET-02 | public C1 projection 需求 | BDY-ET-EVIDENCE → BDY-C1-PROJECTION | C1 evidence contract owner | 当前 CR 停止于 standalone evidence；创建后续 CR 候选 | SC-CR173-C01、DO-CR173-CP3-002 |
| FLOW-ET-03 | 未来真实 activation | future producer → BDY-ET-INPUT | future activation owner | correlation sampling error 未证明时 typed_unavailable | SC-CR173-A01；不属于 CR-173 |

## 共享能力与集成契约

| Shared ID | 名称 | 使用方 | Owner | 调用方向 / 时机 | 输入 | 输出 | 降级 / 同步修改面 |
|---|---|---|---|---|---|---|---|
| SH-ET-01 | canonical sealed-trial identity | estimator | lineage owner | fixture estimator 调用前，只读 | family ref/hash、raw count、ordered trial IDs | validation-bound identity | 缺失 typed_unavailable；CR-173 不修改 lineage store |
| SH-ET-02 | standalone effective evidence | future projection adapter | methodology owner | estimator 成功或失败后 | normalized matrix envelope + method spec | 七字段 evidence | 不可验证时 unavailable/blocked；本 CR 无公共 caller 同步修改 |
| SH-ET-03 | public C1 adapter（deferred） | statistical gate、Gate 1、admission package | C1 contract owner | 后续 CR，经独立 CP2/CP3/CP5 后 | standalone evidence + trusted binding | versioned public projection | 旧 contract 保持 typed_unavailable；同步面见 HLD §10 |

## 待确认边界

| Decision ID | 决策类型 | 问题 | 推荐方案 | 备选方案 | 推荐 / 备选优劣 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CR173-CP3-001 | architecture | 是否采用二阶相关结构的谱 participation-ratio estimand？ | 采用 estimator-only `spectral_participation_ratio` | 转 methodology Spike，全部保持 typed_unavailable | 推荐可识别、确定、边界严格；备选避免错误 claim 但延迟价值 | 推荐不等同 FWER/DSR 校准，必须保留 claim ceiling | 若业务要求 alpha-specific/tail-dependence 等价量，或输入矩阵不能 sealed 对齐，转 Spike |
| DQ-CR173-CP3-002 | architecture | public C1 projection 是否留在 CR-173？ | 拆为后续 CR 候选，CR-173 收缩为 estimator-only | 暂停整个 CR-173 | 推荐保护公共 contract 与 owner；备选更保守但连 estimator 也无法交付 | projection 目标延后，当前 consumer 继续 typed_unavailable | 只有后续 CR 完成 owner approval、versioned migration 与全量回归后才能接入 |

## Gotchas

- correlation matrix 是上游显式事实，不是 CR-173 从 returns、p-values、目录或 raw count 估出来的值。
- `n_eff` 偶然等于 `n` 不表示 alias；必须仍有独立 method/version/hash、input lineage 与 computation ref。
- 七字段 standalone evidence 不等于公共 C1 已升级；没有后续 adapter 和 owner 门禁时，现有 consumers 必须继续 fail-closed。
- 谱 participation ratio 只度量二阶依赖有效维度；把它写成 FWER、DSR 或真实 admission 校准属于 overclaim。
