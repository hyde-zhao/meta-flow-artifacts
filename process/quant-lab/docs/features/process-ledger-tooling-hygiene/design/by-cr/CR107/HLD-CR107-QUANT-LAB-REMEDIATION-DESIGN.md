---
doc_id: "HLD-CR107-QUANT-LAB-REMEDIATION-DESIGN"
cr_id: "CR-107"
title: "quant-lab remediation design for package identity and process tracking hygiene"
status: "ready-for-cp3-review"
owner: "host-orchestrator"
created_at: "2026-06-21T21:28:55+08:00"
scope: "P2 remediation design only"
---

# CR107 quant-lab 整改推荐设计

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 0.1 | 2026-06-21 | host-orchestrator | 基于 CR105/CR106 完成 P2 推荐设计；冻结后续 CR 拆分、风险边界、验证矩阵和 CP3 决策项。 |
| 0.2 | 2026-06-21 | host-orchestrator | 按用户反馈补充 `docs/design`、`docs/features`、`docs/quality` 设计/质量文档层整改计划；CR108 从用户文档修订扩展为分层文档资产 current-truth remediation。 |
| 0.3 | 2026-06-21 | host-orchestrator | 补充遗漏复盘：`docs/*` 分层目录是软链接目录，未跟随软链接的盘点导致误判为空；将 `process` 历史设计材料、工具/测试/脚本身份引用和 release/context 过程材料纳入静态盘点。 |

## 1. 问题定义

CR105 已完成 adoption/process state repair，CR106 已完成 P1 baseline。当前进入 P2 设计阶段，需要解决的问题不是业务策略或交易逻辑，而是 adoption 后的治理面偏离：

| 问题 | 现象 | 影响 |
|---|---|---|
| package identity fallout | `pyproject.toml` 已收敛为 `quant-lab`，新增 `quant_lab` CLI；README 仍写“没有 console script 入口”。 | 用户文档和 package identity 不一致，后续安装/验证入口容易误导。 |
| Host Orchestrator 口径漂移 | README / USER-MANUAL 仍写 `meta-po` 负责编排；当前 AGENTS.md 约定为 Host Orchestrator 主进程编排。 | 过程文档和当前 meta-flow adoption 协议不一致。 |
| 设计/Feature/质量文档层漂移 | `docs/design`、`docs/features`、`docs/quality` 中仍有历史 `meta-po`、`local_backtest`、旧 package identity、旧授权路径和旧实现状态表述。 | 设计文档层可能继续向后续整改输出错误边界；需要分类为 current truth 修订、legacy audit 保留、历史迁移说明或后续 CR。 |
| 过程设计材料误读风险 | `process/HLD.md`、`process/REQUIREMENTS.md`、`process/STORY-BACKLOG.md`、`process/checkpoints`、`process/release`、`process/context` 等保留大量历史设计和交接材料。 | 这些材料不应被批量重写，但如果不分类，后续 agent 可能把历史过程证据误读为当前真相。 |
| 工具/测试/脚本身份引用漂移 | `tests`、`scripts`、配置、agent/skill 安装面可能仍引用 `local-backtest`、`local_backtest`、旧 CLI 或旧 role 名。 | 不一定需要立即修改业务代码，但需要在 P3/P4 中通过静态盘点判断是否只是测试 fixture / 历史证据 / 当前执行入口。 |
| CR tracking historical hygiene | 历史 `status` / `lifecycle_status`、summary_ref、follow-up / spike candidate 索引仍有 WARN/FAIL。 | 默认 CR lifecycle check 不干净，影响后续 active/blocked/candidate 判断。 |
| process/state durability | state v2、base ledgers、quality policies、identity 产物刚建立，需要在后续整改中保持稳定。 | 远端同步或重建 index 时可能再次引入状态漂移。 |

### 遗漏复盘

| 遗漏 | 直接原因 | 纠正措施 |
|---|---|---|
| 初版 CR107 未覆盖 `docs/design` / `docs/features` / `docs/quality` | 盘点时使用未跟随软链接的目录扫描，`docs/design`、`docs/features`、`docs/quality` 被误判为无文件。 | CR108 的文档盘点必须使用 `find -L` 或 `rg`，并把软链接目录作为正式 artifact surface 处理。 |
| 初版范围偏向 README / USER-MANUAL | 过度依赖 package identity scan 的直接输出，未先建立 formal document surface inventory。 | CR108 入口先产出分层文档偏离矩阵，再决定修订 / 保留 / 迁移说明 / 后续候选。 |
| 初版未显式区分 `process` 历史材料 | 将 `process` 视为运行态和 CR tracking 面，未把其中历史 HLD/requirements/checkpoint/release 视为可能被后续读取的设计证据面。 | `process` 历史设计材料纳入 inventory-only；默认 legacy audit keep，不批量改写。 |

## 2. 目标与非目标

### 目标

- 修正当前真相文档与 package identity / Host Orchestrator 协议的偏离。
- 纳入 `docs/design`、`docs/features`、`docs/quality` 分层文档资产，建立“修订 / 保留 / 迁移说明 / 后续候选”分类矩阵。
- 规范 CR tracking 的机器字段、历史状态保留策略、summary/evidence 缺口处理策略。
- 设计 P3/P4 的实施 CR 和验证矩阵。
- 保持所有整改在非 runtime、非凭据、非生产写、非交易边界内。

### 非目标

- 不修改策略、交易、回测、数据源、QMT/MiniQMT、runtime 业务逻辑。
- 不访问 `.env`、token、API key、cookie、私钥、账号或任何凭据。
- 不启动 runtime、SaaS、trace upload、production write、真实交易或真实发布。
- 不启动 CR-033。
- 不关闭、处理或绕过 CR102-CR104；它们按用户说明在主线独立关闭。

## 3. Architecture Gray Areas

| Gray Area | 选项 | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|---|
| GA-1 package identity 是否直接修用户文档 | A. P3 直接修 README / USER-MANUAL current truth | 快速消除用户可见偏离 | 可能混入历史审计段落 | README, USER-MANUAL, package identity | 推荐 | 仅修改 current truth 段落；历史段落保留。 |
| GA-1 package identity 是否直接修用户文档 | B. 只记录偏离，不改文档 | 风险最低 | 偏离继续存在 | docs consistency | 不推荐 | 仅当用户要求不改文档时切换。 |
| GA-2 CR tracking 是否一次性全量清历史 | A. 分层修复机器字段和最小 summary/evidence | 能让机器入口逐步收敛，风险可控 | 需要保留旧 status 审计语义 | process/changes, state ledgers | 推荐 | P3 先处理机器阻断，再处理 WARN。 |
| GA-2 CR tracking 是否一次性全量清历史 | B. 一次性重写历史 CR index | 快速追求 clean check | 大量历史状态语义可能被误删 | audit trail | 不推荐 | 仅在完整备份和用户批准后考虑。 |
| GA-3 Host Orchestrator 口径是否替换所有历史 meta-po | A. 只替换 current-facing 文档 | 保留历史审计，同时修当前指导 | 仍会在旧 process 归档中看到 meta-po | README, USER-MANUAL | 推荐 | 历史过程文件不改。 |
| GA-3 Host Orchestrator 口径是否替换所有历史 meta-po | B. 全仓替换 meta-po | 表面一致 | 破坏历史追溯 | process archive | 不推荐 | 不建议切换。 |
| GA-4 后续 CR 拆分 | A. docs/package、CR tracking、verification 三类拆分 | 审查清晰，回滚容易 | CR 数略多 | P3/P4 plan | 推荐 | P2 CP3/CP5 可冻结为 4 个后续 CR。 |
| GA-4 后续 CR 拆分 | B. 单个大整改 CR | 门禁少 | 风险集中，难验证 | all remediation | 不推荐 | 仅当用户要求极短流程时使用。 |
| GA-5 docs 目录是否只修用户文档 | A. 纳入 README、USER-MANUAL、`docs/design`、`docs/features`、`docs/quality` 分层整改 | 覆盖完整设计链，防止 P3/P4 继续消费旧设计口径 | 范围更大，需要分类矩阵控制 | docs, design, feature, quality | 推荐 | 仅修 current-facing/current-truth；历史审计材料不批量重写。 |
| GA-5 docs 目录是否只修用户文档 | B. 只修 README / USER-MANUAL | 快速消除用户入口偏离 | 设计/Feature/质量文档仍可能误导后续整改 | user docs only | 不推荐 | 仅当用户明确要求最小文档面时切换。 |

## 4. 候选方案比较

| 方案 | 内容 | 优势 | 代价 | 结论 |
|---|---|---|---|---|
| S1 单体整改 | 一个 CR 同时改 docs、CR tracking、验证和收尾。 | CR 数少。 | 容易把文档、过程索引和验证混在一起；失败难定位。 | 不推荐。 |
| S2 双轨整改 | 一个 CR 修分层文档资产/package，一个 CR 修 CR tracking，最后用验证收尾。 | 清晰且 CR 数适中；覆盖 README、USER-MANUAL、`docs/design`、`docs/features`、`docs/quality`。 | CR108 范围更大，需要先做文档分类矩阵。 | 推荐。 |
| S3 只记录不实施 | 只保留偏离矩阵，暂不整改。 | 最保守。 | 用户文档和机器入口继续偏离。 | 不推荐。 |

推荐方案：S2 双轨整改。

## 5. 推荐架构

```text
CR107 P2 Design
  |
  +-- CR108 layered docs/package identity current-truth remediation
  |     - README current truth
  |     - USER-MANUAL current truth
  |     - package identity / CLI docs
  |     - docs/design current indexes and CR-scoped HLD/ADR drift classification
  |     - docs/features DESIGN / TEST-PLAN / TASKS drift classification
  |     - docs/quality REVIEW / TEST-REPORT / VERIFICATION-REPORT current-truth boundaries
  |
  +-- CR109 CR tracking historical hygiene
  |     - lifecycle_status machine enum normalization
  |     - summary_ref / evidence policy
  |     - follow-up / spike candidate classification
  |
  +-- CR110 non-runtime verification and release readiness
        - identity/state/adoption doctors
        - human-gate / cr-tracking checks
        - docs current-truth scan
        - P4 close readiness
```

## 6. 模块职责

| 模块 / 文件面 | 责任 | 禁止事项 |
|---|---|---|
| `README.md` | current-facing package identity、CLI、Host Orchestrator 口径。 | 不改历史 CR 事实，不宣称 runtime-ready。 |
| `docs/USER-MANUAL.md` | 用户入口、授权路径、CLI 说明。 | 不写入凭据、真实账户、runtime 授权。 |
| `docs/design/*.md` | 当前设计索引、HLD、ADR、蓝图、领域图、依赖图、Feature 矩阵的 current-truth 对齐和 legacy 分类。 | 不批量重写历史设计审计；不把旧 runtime gate 改写为已授权。 |
| `docs/features/**` | Feature DESIGN / TEST-PLAN / TASKS 中的 package identity、Host Orchestrator、授权路径和实现状态口径复核。 | 不借文档整改修改 Story 业务范围或代码任务。 |
| `docs/quality/**` | REVIEW / TEST-REPORT / VERIFICATION-REPORT 中当前可消费结论和历史证据边界分类。 | 不把历史 runtime / NAS / QMT 结果改写为当前授权；不访问真实报告数据。 |
| `process/HLD.md`, `process/REQUIREMENTS.md`, `process/STORY-BACKLOG*.md` | 历史过程设计 / 需求 / Story 证据分类。 | 不作为当前 truth source 批量改写；只做 inventory-only 和 legacy/current 标注建议。 |
| `process/checkpoints`, `process/context`, `process/release` | 历史门禁、交接和发布上下文材料的可读边界。 | 不重写历史 gate 结论；不把旧 runtime 授权解释成当前授权。 |
| `tests`, `scripts`, `.codex/agents`, `.agents/skills` | 静态识别旧 package identity、旧 CLI、旧 agent role 口径是否影响当前非 runtime 验证入口。 | 不改业务逻辑；不启动 runtime；不访问凭据。 |
| `docs/design/PACKAGE-IDENTITY.yaml` | package identity truth source。 | 不为兼容而回退主身份。 |
| `process/changes/CR-INDEX.*` | CR tracking 机器入口。 | 不静默删除历史 status 语义。 |
| `process/state/*LEDGER.ndjson` | CP/CR/event 审计。 | 不用 Markdown 摘要替代 ledger。 |

## 7. Use Case Traceability

| Use Case | 推荐设计覆盖 |
|---|---|
| 用户查看项目入口并运行命令 | CR108 修正 CLI 和 package identity current truth。 |
| 用户判断谁负责 meta-flow 编排 | CR108 将 `meta-po` current-facing 说明改为 Host Orchestrator 主进程编排。 |
| 后续设计整改读取 `docs/design` / `docs/features` | CR108 先产出分层文档偏离矩阵，标注 must-fix current truth、legacy audit keep、move-to-history、follow-up candidate、no-change with reason。 |
| 后续质量验证读取 `docs/quality` | CR108/CR110 区分当前验证真相、历史验证报告和不授权 runtime/NAS/QMT 证据。 |
| 后续 agent 读取 `process` 历史材料 | CR108/CR110 将 `process` 历史设计、checkpoint、release/context 材料标注为 inventory-only 或 legacy audit keep，避免误当当前范围。 |
| 后续静态测试读取脚本 / fixture | CR110 扫描 tests/scripts/config/tooling 中的旧身份引用，分类为 fixture、legacy alias、current-fix 或 follow-up。 |
| 后续整改前判断 active/blocked/candidate | CR109 收敛 CR tracking 机器字段和候选分类。 |
| P3/P4 验证不触碰 runtime | CR110 使用 identity/state/adoption/cr-tracking/human-gate 静态验证。 |

## 8. 关键场景模拟

| 场景 | 流程 | 预期 |
|---|---|---|
| package identity 检查 | 用户读取 README -> 看到 `quant-lab` / `quant_lab` -> 运行非 runtime CLI smoke。 | 文档与 `pyproject.toml` / PACKAGE-IDENTITY 一致。 |
| Host Orchestrator 授权路径 | 用户看到真实运行请求 -> USER-MANUAL 指向 CR/CP/human gate，不再写 meta-po 授权路径。 | 不误导到已废弃编排角色。 |
| 设计文档层偏离处理 | 扫描 `docs/design` / `docs/features` / `docs/quality` -> 生成分类矩阵 -> 只修改 current-facing/current-truth 项。 | 历史审计语义保留，后续整改消费的设计入口对齐当前 meta-flow。 |
| CR tracking 验证 | 运行 `meta-flow check cr-tracking --allow-multiple-active` -> WARN 分类清晰；默认失败项进入 P3/P4。 | 后续关闭 CR102-CR104 后可继续收敛默认检查。 |

## 9. 分阶段落地建议

| 阶段 | CR | 目标 | 退出条件 |
|---|---|---|---|
| P2 Design | CR107 | 完成 HLD、阶段计划、验证矩阵、CP3/CP5 门禁。 | CP3 approved；必要时 CP5 approved。 |
| P3 Layered docs/package remediation | CR108 | 修 current-facing README / USER-MANUAL / package identity docs，并纳入 `docs/design`、`docs/features`、`docs/quality` 分层文档资产偏离矩阵和 current-truth 修订。 | docs current-truth scan PASS；identity check PASS；分层矩阵中无未分类偏离。 |
| P3 CR tracking hygiene | CR109 | 收敛 CR index 机器字段、summary/evidence 缺口策略、candidate 分类。 | cr-tracking PASS；cr lifecycle 历史阻断降到可解释 WARN 或 closed。 |
| P4 Verification/readiness | CR110 | 非 runtime 回归、doctor、human-gate、release readiness。 | CP8 approved；P2-P4 delivered with risk。 |

CR108 建议子波次：

| 子波次 | 文档面 | 目标 | 处理策略 |
|---|---|---|---|
| CR108-W1 | README / USER-MANUAL | 修 package identity、CLI、Host Orchestrator 和授权路径 current truth。 | 直接修 current-facing 段落。 |
| CR108-W2 | `docs/design` | 复核 HLD、ADR、BLUEPRINT、DOMAIN-MAP、DEPENDENCY-MAP、FEATURE-DESIGN-MATRIX。 | 建立分类矩阵；只修 current index/current truth；历史 CR-scoped HLD 保留审计语义。 |
| CR108-W3 | `docs/features` | 复核 Feature DESIGN / TEST-PLAN / TASKS 的旧角色、旧包名、旧授权路径、旧实现状态。 | current Feature 索引修订；历史 Feature 证据保留并标注 legacy。 |
| CR108-W4 | `docs/quality` | 复核 REVIEW / TEST-REPORT / VERIFICATION-REPORT 的当前可消费结论和历史质量证据边界。 | 不改历史测试结果；补 current/legacy 说明和不授权边界。 |
| CR108-W5 | `process` historical design/readiness surfaces | 复核 process 根 HLD/REQUIREMENTS/STORY-BACKLOG、checkpoints、context、release 材料是否会被后续误读。 | inventory-only；默认 legacy audit keep；只在当前 context capsule 或 index 指向错误时修路由说明。 |

后续计划 CR 数：P2 1 个（CR107），P3 2 个（CR108-CR109），P4 1 个（CR110）。总计 P2-P4 推荐 4 个 CR；CR108 内部按 5 个子波次执行。

## 10. 风险与回退

| 风险 | 等级 | 缓解 |
|---|---|---|
| 历史文档中 `meta-po` 是审计事实，不应全量替换 | 中 | 只修 current-facing README/USER-MANUAL；process/archive 保留。 |
| package name 改为 `quant-lab` 后有旧脚本依赖 | 中 | 保留 legacy alias；P3 先做文档和非 runtime smoke。 |
| 设计文档层范围扩大导致误改历史审计 | 高 | CR108 必须先产出分类矩阵；历史 CR-scoped 设计/质量证据默认 legacy audit keep，只有 current index/current truth 才修改。 |
| CR lifecycle check 历史失败项数量大 | 高 | P3 拆成专门 CR109，先定义机器字段映射和 summary policy。 |
| CR102-CR104 仍 active | 中 | 本设计不处理；P3 前若主线未关闭，则继续以风险接受或 pending gate 处理。 |

## 11. CP3 待人工决策

| 决策 ID | 类型 | 问题 | 推荐方案 | 备选方案 |
|---|---|---|---|---|
| DQ-CP3-CR107-01 | architecture | 是否采用双轨整改设计？ | 采用 CR108 docs/package + CR109 CR tracking + CR110 verification。 | 合并为单个大 CR；或只记录不修。 |
| DQ-CP3-CR107-02 | scope | 是否把 CR108 扩展为 README、USER-MANUAL、`docs/design`、`docs/features`、`docs/quality` 分层文档资产整改，并把 `process` 历史设计/readiness 材料作为 inventory-only 面？ | 是；先分类矩阵，再修 current truth；`process` 历史材料默认 legacy audit keep。 | 只修 README / USER-MANUAL；或把 process inventory 后移到 CR110。 |
| DQ-CP3-CR107-03 | implementation | P3/P4 是否允许修改 current-truth 文档并静态扫描 tests/scripts/config/tooling，但不改业务代码？ | 允许。 | 文档也延后到 P4；或只允许 inventory 不允许修订。 |
| DQ-CP3-CR107-04 | risk_acceptance | CR102-CR104 未关闭时是否允许 P2 设计通过？ | 允许，但 P3 前重新检查。 | 阻塞 CR107 直到主线关闭。 |

## 12. 自审

| 检查项 | 结论 |
|---|---|
| 至少 2 个候选方案 | PASS |
| Architecture Gray Areas 前置 | PASS |
| 不触碰业务代码 / runtime / credentials | PASS |
| HLD 拆分判断 | PASS：当前是单一治理设计 HLD，后续实施拆成 CR108/CR109/CR110；不需要拆 HLD。 |
| 场景模拟 | PASS |
