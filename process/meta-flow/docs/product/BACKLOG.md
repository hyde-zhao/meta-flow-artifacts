---
status: baseline
version: "1.7"
created_at: "2026-07-11"
owner: "meta-pm"
active_change_ref: "CR-051"
source_use_cases: "process/docs/product/USE-CASES.md"
source_mvp_scope: "process/docs/product/MVP-SCOPE.md"
---

# Meta Flow 产品 Backlog

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 | 文档处理方式 |
|---|---|---|---|---|
| 1.2 | 2026-07-13 | host-orchestrator-inline-fallback | 增量追加 CR-047 deferred、风险和不授权边界。 | 原文档增量更新 |
| 1.3 | 2026-07-15 | host-orchestrator inline fallback | 增量追加 CR-050 forge receipt、Git Town adapter、自动 commit planner 候选及远端写入不授权边界。 | 原文档增量更新 |
| 1.4 | 2026-07-16 | host-orchestrator inline fallback | 用户将独立 fast-forward-only merge 纳入当前 CR-050；它不再属于 deferred。Forge/PR adapter、merge commit、策略绕过与自动 commit 仍在范围外；新增 default-branch write 独立授权索引。 | 原文档增量更新；等待 CP2 R2 |
| 1.5 | 2026-07-17 | meta-pm | 增量追加CR-051逐项目真实迁移/软链接、真实remote pilot、bare control转换和线性历史策略候选；保留既有Backlog/NA ID。 | 原文档增量更新；等待CP2 |
| 1.6 | 2026-07-17 | meta-pm | CR-051 CP2 R2：Backlog/NA ID不变；记录existing-control与显式merge-main已被选择，bare control与rebase+force继续作为有条件future candidate；移除“idle态待决策”陈旧语义。 | 原文档增量更新；等待CP2 R2总体approve |
| 1.7 | 2026-07-18 | meta-pm | CR-051 CP2 R3：Backlog/NA ID不变；将自动main↔integration同步、rebase/force与真实同步执行保持为后续候选/不授权项；R2 per-CR merge-main仅保留superseded历史。 | 原文档增量更新；等待CP2 R3总体approve |
| 1.0 | 2026-07-11 | meta-pm | 初始化独立 backlog，保留既有 DEF-PG/DEF 引用并加入 CR-046 deferred/non-authorized 项 | 缺失产物初始化；不重新编号上游 deferred ID |
| 1.1 | 2026-07-12 | meta-pm | CR-046 CP2 scope rework R2：确认五项 scope finding 均为当前 MVP required，不进入 deferred；保留既有 backlog ID | 原文档增量更新 |

## 既有 Deferred 引用

`DEF-01..04` 与 `DEF-PG-001..004` 继续以 `USE-CASES.md` / `MVP-SCOPE.md` 为正式定义；本文件只提供恢复索引，不复制或替换旧基线。

## CR-046 Deferred

> CR046-SCOPE-F01..F05 已由用户在 R1 review 中明确纳入 required scope；compaction 语义保持、通用 post-close correction、机器 audit report、null-provenance dogfooding 和 dispatch 证据限制不得转入本表或后续 CR。

| Backlog ID | 来源 | 候选项 | 状态 | 延后原因 | 重启条件 |
|---|---|---|---|---|---|
| BL-EI-001 | DEF-EI-001 / SGA-05 | 跨平台统一加密签名 receipt | deferred | 当前平台能力不一致，本轮先冻结 evidence-level 与 unavailable 语义 | 所有目标平台提供稳定、可验证、版本化签名 contract |
| BL-EI-002 | DEF-EI-002 / SGA-06 | 基于估算 token 的强制计费或配额门 | deferred | estimate 不能冒充 measured telemetry | 平台 telemetry 稳定覆盖，且估算误差模型通过独立验证 |
| BL-WT-001 | DEF-WT-001 | repository-verifiable platform receipt producer | deferred | 当前平台能力不可用，本 CR 只能保留证据上限 | 平台提供稳定 discovery/selector/receipt contract |
| BL-WT-002 | DEF-WT-002 | 独立 runtime/SaaS/pilot 验证 | deferred | 本 CR 未授权凭据、runtime、SaaS、production write | 独立 runtime-high-risk CR 与用户授权 |
| BL-GB-001 | DEF-GB-001 / SGA-GB-04 | Forge receipt adapter 支持 squash/rebase merge 后的可证明 cleanup | deferred | 需要平台 API、identity、token、PR receipt 和最小权限契约 | 用户选择目标 forge 并启动独立 CR |
| BL-GB-002 | DEF-GB-002 / SGA-GB-01 | Git Town/stacked branch adapter | deferred | 普通 CR branch 可由原生 Git 满足，外部工具增加配置和同步策略 | 真实 stacked/offline ship 场景出现 |
| BL-GB-003 | DEF-GB-003 / SGA-GB-05 | 自动 stage/commit planner | deferred | 文件选择、secret scan、双仓消息和回滚边界尚未批准 | 用户明确批准 path allowlist、scan 与 rollback 契约 |

## CR-051 Deferred / Follow-up

| Backlog ID | 来源 | 候选项 | 状态 | 延后原因 | 重启条件 |
|---|---|---|---|---|---|
| BL-AW-001 | DEF-AW-001 / SGQ-AW-002 | 按项目执行真实artifact文件迁移 | deferred-by-user | 用户选择先完成能力；每个项目的mapping、dirty状态、备份和回滚不同 | CR-051 READY后，用户选择目标项目并批准该项目迁移CR/操作 |
| BL-AW-002 | DEF-AW-002 / SGQ-AW-002 | 按项目重挂process/docs/checkpoints软链接并切换layout version | deferred-by-user | 链接切换必须与对应文件迁移和源码项目bootstrap一起验收 | 目标项目文件迁移完成，worktree health与preflight manifest通过 |
| BL-AW-003 | CR051-RISK-REMOTE-UNVERIFIED | 在真实shared artifact remote上做两项目并发pilot | deferred-authorization | 当前只允许本地bare remote fixture，不授权真实remote/ref mutation | 用户指定remote/repo/project/ref/OID并给出最小操作授权与回滚 |
| BL-AW-004 | DEF-AW-002 / SGA-AW-04 | 将control checkout转换为bare control repo | deferred-architecture / alternative-not-selected | 用户已选择保留existing control checkout并把worktree放在configurable sibling root；bare转换扩大操作面 | 真实试点证明当前control checkout持续造成branch占用或维护问题，并通过新CR重新决策 |
| BL-AW-005 | DEF-AW-003 / SGA-AW-03..04 | 自动main↔integration同步或rebase+force-with-lease线性历史策略 | deferred-risk / alternative-not-selected | 用户已选择CR外人工同步；自动同步/历史改写会扩大跨项目、force与回滚授权 | 人工同步成本被量化为不可接受，且用户通过新CR批准同步策略、无活跃artifact CR门和branch-scoped高风险授权 |

## 不授权项索引

| ID | 项目 | 状态 | 当前影响 | 未来入口 |
|---|---|---|---|---|
| NA-EI-001 | credentials / runtime / production write / publish / trading | not-authorized | 不进入 CR-046 CP2 或实现默认授权 | 需要独立用户授权、安全边界与回滚方案 |
| NA-EI-002 | repository commit / push | not-authorized | 本轮只允许本地工件与验证 | 用户明确要求后按仓库发布流程处理 |
| NA-EI-003 | quant-lab lineage business-code changes | not-authorized | CR-163 仅 process-evidence append-only pilot | 新业务目标、独立 CR、HLD/LLD 与业务验收 |
| NA-WT-001 | 处理 prelink backup | not-authorized | 用户明确“不需要处理” | 仅用户未来明确要求时 |
| NA-WT-002 | 子 Agent 调度 | disabled-by-user | CR-047 后续阶段使用审计化 inline fallback | 用户未来撤销该选择时恢复 |
| NA-GB-001 | CR-050 CP2 R2 阶段的源码实现与 repository commit/push/default-update/delete | not-authorized | 当前只形成产品基线和 Decision Brief | CP2/CP3/CP5 通过后按实现边界推进；每类真实远端 mutation 仍需显式授权 |
| NA-GB-002 | 隐式 merge、merge commit、自动冲突解决、forge API/credential、force-push/force-delete 或策略绕过 | not-authorized | 不进入原生 Git fast-forward MVP | 受保护仓 merge 需要未来 forge adapter、高风险 CR、平台契约与最小权限授权 |
| NA-GB-003 | 真实 default-branch write | not-authorized-at-CP2 | 产品范围包含显式 merge，但 CP2 R2 approval 本身不授权执行；branch protection 拒绝是合法 BLOCKED/PARTIAL | 后续 CP5/CP6 后由用户对具体仓、分支和 expected OID 提供独立操作授权 |
| NA-AW-001 | 真实artifact文件迁移或删除 | not-authorized / deferred-by-user | CR-051只开发能力和生成manifest | BL-AW-001按项目启动，需mapping/备份/回滚/验证 |
| NA-AW-002 | 现有软链接创建、替换、修复或删除 | not-authorized / deferred-by-user | 不改变当前项目或其他项目bootstrap路径 | BL-AW-002与目标项目迁移一起授权 |
| NA-AW-003 | 真实artifact worktree/branch/ref/commit/tag/stash mutation | not-authorized | 自动验证仅可在临时repo/bare remote执行 | CP5/CP6后对具体本地目标和操作单独授权；真实remote另见NA-AW-004 |
| NA-AW-004 | 真实remote fetch/push/default update/delete/force、main↔integration同步或forge API | not-authorized | 本轮不读取凭据、不绕过branch protection、不发布；单个artifact CR不得执行shared-main同步 | BL-AW-003或后续CR外人工维护流程中按repo/ref/OID最小授权，默认要求项目无活跃artifact CR |
| NA-AW-005 | credentials/runtime/SaaS/production write/trading | not-authorized | 不属于project-first/worktree能力MVP | 独立高风险CR与显式授权 |

## Gotchas

- `unavailable` 是证据状态，不是待实现功能；不得把“伪造 receipt/telemetry”放入 backlog。
- Deferred 不自动扩大 MVP。只有满足重启条件并通过新的范围决策后，才可进入 Story Map。
- CR-163 pilot 的回修只能追加 correction/supersession evidence，不能以“backlog 清理”为由改写历史行。
- `deferred-by-user` 是用户主动切分交付顺序，不是能力缺陷；CR-051不得为了“顺手完成”而启动BL-AW-001/002。
- persistent project worktree的idle态、异构双leg、aggregate gate、create-only integration bootstrap和control/worktree拓扑已由CP2 R3用户决策冻结；DQ-02的per-CR merge-main仅为superseded历史。CP3只细化状态机/schema/gate、correlation/OID与人工同步precheck，不得把未选备选重新当作默认值。
