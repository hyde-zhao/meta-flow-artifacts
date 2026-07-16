---
status: baseline
version: "1.4"
created_at: "2026-07-11"
owner: "meta-pm"
active_change_ref: "CR-050"
source_use_cases: "docs/product/USE-CASES.md"
source_mvp_scope: "docs/product/MVP-SCOPE.md"
---

# Meta Flow 产品 Backlog

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 | 文档处理方式 |
|---|---|---|---|---|
| 1.2 | 2026-07-13 | host-orchestrator-inline-fallback | 增量追加 CR-047 deferred、风险和不授权边界。 | 原文档增量更新 |
| 1.3 | 2026-07-15 | host-orchestrator inline fallback | 增量追加 CR-050 forge receipt、Git Town adapter、自动 commit planner 候选及远端写入不授权边界。 | 原文档增量更新 |
| 1.4 | 2026-07-16 | host-orchestrator inline fallback | 用户将独立 fast-forward-only merge 纳入当前 CR-050；它不再属于 deferred。Forge/PR adapter、merge commit、策略绕过与自动 commit 仍在范围外；新增 default-branch write 独立授权索引。 | 原文档增量更新；等待 CP2 R2 |
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

## Gotchas

- `unavailable` 是证据状态，不是待实现功能；不得把“伪造 receipt/telemetry”放入 backlog。
- Deferred 不自动扩大 MVP。只有满足重启条件并通过新的范围决策后，才可进入 Story Map。
- CR-163 pilot 的回修只能追加 correction/supersession evidence，不能以“backlog 清理”为由改写历史行。
