---
status: "confirmed-cp3-archived-baseline"
version: "0.2"
cr_id: "CR-166"
source_story_map: "docs/product/STORY-MAP.md"
source_mvp_scope: "docs/product/MVP-SCOPE.md"
confirmed_by: "user"
confirmed_at: "2026-07-13T12:11:57+08:00"
---

# Blueprint：CR166 Walk-forward / OOS Computable Evidence Producer

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 0.1 | 2026-07-13 | host-orchestrator inline meta-se-critical | 冻结 method-neutral envelope、C2 contract/validator/producer、daily/ML adapter、三个既有 consumer projection、event N/A 与五个 CP4 outcome 输入。 |
| 0.2 | 2026-07-13 | host-orchestrator | 回填 CP3 人工批准；五个 outcome 进入 CP4 正式 Story/Feature 设计与文件所有权冻结。 |

## 能力地图

| Capability ID | 能力域 | 用户价值 | 产品候选 | Owner Feature |
|---|---|---|---|---|
| CAP-166-01 | Evidence envelope foundation | C2 与未来 C3/C4 可在稳定 header 下独立演进，且不依赖 C1 方法合同 | CR166-S01 | FEAT-166-01 Neutral envelope & contracts |
| CAP-166-02 | Temporal/leakage validation | 在产生 OOS claim 前阻断时间逆序、purge/embargo、metric 与 lineage 缺口 | CR166-S02 | FEAT-166-02 Fold validator & adapters |
| CAP-166-03 | Deterministic C2 production | 生成可重算 fold outcome、pass rate、reason、provenance 与 canonical hash | CR166-S03 | FEAT-166-03 C2 producer |
| CAP-166-04 | Conservative integration | 让 CR151、CR154、admission package 消费同一 evidence，不创建平行 gate | CR166-S04 | FEAT-166-04 Existing-consumer projections |
| CAP-166-05 | Compatibility and verification | 证明 daily/ML、8 类 fail-closed、event N/A、CR155 regression 与零外部操作 | CR166-S05 | FEAT-166-05 Fixture/static verification |

## Feature / Epic 边界

| Feature ID | 名称 | 负责事项 | 不负责事项 | 拥有数据 | 只读数据 | 禁止依赖 |
|---|---|---|---|---|---|---|
| FEAT-166-01 | Neutral envelope & contracts | 通用 availability、canonical domain hash、stable header、静态 component catalog、C2 input/evidence schema | 统计方法计算、admission policy、runtime registry/store | `StrategyEvidenceEnvelope`、`ComponentDescriptor`、neutral canonical primitives | CR164 C1 public API/hash fixtures | C2 直接依赖 C1 method schema；动态发现外部 component |
| FEAT-166-02 | Fold validator & adapters | 半开时间区间、fold/policy/metric/lineage/authorization 校验；daily 与 ML 输入映射 | 读取真实数据、推断缺失 validation/OOS boundary、event-specific fold 设计 | normalized input、validation issues、adapter result | legacy daily manifest、ML purged/embargo policy、显式 fixture payload | 网络、lake/NAS/provider resolver；consumer 反向写 input |
| FEAT-166-03 | C2 producer | 纯函数生成 fold evidence、aggregate pass rate/outcome、reason、hash、provenance | 阈值治理、真实 research run、C3/C4 calculator | `WalkForwardOOSComponent`、fold evidence、component hash | FEAT-01 envelope；FEAT-02 validated input | consumer internals；外部 I/O；未知 component 参与 PASS |
| FEAT-166-04 | Existing-consumer projections | 将同一 C2 ref/hash/availability/reasons 投影到 CR151、CR154、StrategyAdmissionPackage；worse-state merge | 创建新 gate、提升 blocked/typed_unavailable、重算 fold | 三种 projection DTO/映射 | FEAT-03 component；现有 consumer contracts | 修改 consumer policy owner；consumer→producer 反向依赖 |
| FEAT-166-05 | Fixture/static verification | daily/ML fixtures、8 类负向、10-run、event N/A、CR155 与 permission regression | 真实数据、event feed、独立 runtime verifier、publish/deploy | test fixtures、evidence index、verification result | FEAT-01..04 public contracts | lake/NAS/provider/credential/runtime/broker access |

## 跨 Feature 流程

| Flow ID | 触发 | 参与 Feature | 数据写入 Owner | 失败路径 | 验证入口 |
|---|---|---|---|---|---|
| FLOW-166-01 | 显式 daily/ML fixture 输入 | FEAT-01 → FEAT-02 → FEAT-03 | 每层只写自己的 immutable value object | 缺失→typed_unavailable；冲突/非法/未授权→blocked | SC-CR166-P01/P02/N01..N06/A01/H01 |
| FLOW-166-02 | C2 component 生成完成 | FEAT-03 → FEAT-04 → existing consumers | FEAT-03 拥有 evidence；FEAT-04 只写 projection | 任何更差状态保留；ref/hash mismatch blocked | SC-CR166-P01/H01 + QAC-08 |
| FLOW-166-03 | event compatibility 请求 | FEAT-02 → FEAT-05 | FEAT-05 记录 applicability result | event 语义未冻结→`not_applicable_with_reason`，不创建 producer | SC-CR166-E01 |
| FLOW-166-04 | CP7 fixture/static 验证 | FEAT-05 → FEAT-01..04 | FEAT-05 写验证证据，不写生产 evidence store | false PASS、外部操作、CR155 提升任一出现即 FAIL | QAC-01..12 |

## 共享能力与集成契约

| Shared ID | 名称 | 使用方 | Owner | 调用方向 / 时机 | 输入 | 输出 | 降级 / 调用方同步修改 |
|---|---|---|---|---|---|---|---|
| SH-166-01 | Neutral canonical primitives | C1 compatibility wrapper、C2、未来 C3/C4 | FEAT-166-01 | envelope 或 component 计算 hash 时调用 | JSON-safe finite value + explicit domain | canonical bytes / `sha256:*` | 兼容 fixture 不一致即阻断迁移；C1 保留旧 public name/default domain |
| SH-166-02 | Static component catalog | envelope validator/producer | FEAT-166-01 | 反序列化与 mandatory component 判断前 | type、schema version、required/optional | active/reserved/unknown classification | unknown mandatory=blocked；unknown optional=preserved但无语义；C3/C4 仅 reserved |
| SH-166-03 | Common fold validator | daily/ML adapters、producer | FEAT-166-02 | producer 前唯一调用 | 7/7 字段族 | normalized input + typed issues | 缺失不推断；非法/矛盾 blocked；adapter 必须补齐显式 common fields |
| SH-166-04 | Consumer projection | CR151、CR154、StrategyAdmissionPackage | FEAT-166-04 | C2 evidence validation 与 hash 校验后 | evidence ref/hash/availability/outcome/reasons | 三种 existing-contract mapping | worse-state merge；3 consumers 同步增加 evidence ref/reason 读取面，不改变 gate policy |

## 五个 CP4 outcome 输入

| Candidate | Outcome | Dependency | 建议 lld_policy |
|---|---|---|---|
| CR166-S01 | neutral primitives、envelope 与 C2 input/evidence contract | 无 | full-lld |
| CR166-S02 | temporal/leakage/sufficiency validator + daily/ML adapter | S01 | full-lld |
| CR166-S03 | deterministic C2 producer、fold aggregation 与 canonical evidence | S01-S02 | full-lld |
| CR166-S04 | 3/3 existing-consumer projections + CR155 regression | S01-S03 | full-lld |
| CR166-S05 | QAC、negative、event N/A、permission 与 regression verification | S01-S04 | full-lld |

以上五项已获 CP3 批准，作为 CP4 正式 Story、DAG、Wave、文件 ownership 与 CP5 design evidence 的冻结输入；CP5 批准前仍不得实现。

## 待确认边界

| Decision ID | 决策类型 | 问题 | 推荐方案 | 备选方案 | 推荐 / 备选优劣 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP3-CR166-001 | architecture | envelope 与 canonical primitives 如何组织？ | neutral primitives + stable header + static component catalog；C1 compatibility wrapper | C2 依赖 C1；复制实现；dynamic registry | 推荐无错向依赖、无双实现、无 runtime discovery | 决定公共 schema 与 C1 regression 面 | C1 hash/API 无法 100% 保持则暂停抽取、保留 wrapper |
| DQ-CP3-CR166-002 | architecture | fold/leakage/status/denominator 如何冻结？ | 半开区间、显式 policy minima、缺失/冲突分层、declared-fold denominator | 字符串非空校验；隐式日期推断；忽略坏 fold | 推荐可审计且不会抬高 pass rate | 决定 8 类 P0 是否真正 fail-closed | 若真实交易日历参与，另起 Stage 3 resolver 设计 |
| DQ-CP3-CR166-003 | architecture | consumer 与 event compatibility 如何处理？ | 3 个薄 projection；event 明确 N/A | 新建 gate；event 空壳/日历折算 | 推荐保持 owner 单一且不制造假覆盖 | 决定现有门禁兼容与 event claim ceiling | event fold semantics 独立冻结后通过新 CR 接入 |
| DQ-CP3-CR166-004 | security | 是否允许五个 CP4 outcome 输入并保持 design-only/Stage ceiling？ | 批准后只进 CP4/CP5 准备；Stage2=true、Stage3=false | 修改设计；暂停 CR | 推荐继续交付但不扩大权限 | 不授权实现、真实数据或 runtime | 任何新数据/runtime/external/write 需求立即停止并请求授权 |
