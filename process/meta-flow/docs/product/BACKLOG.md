---
status: baseline
version: "1.1"
created_at: "2026-07-11"
owner: "meta-pm"
active_change_ref: "CR-046"
source_use_cases: "docs/product/USE-CASES.md"
source_mvp_scope: "docs/product/MVP-SCOPE.md"
---

# Meta Flow 产品 Backlog

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 | 文档处理方式 |
|---|---|---|---|---|
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

## 不授权项索引

| ID | 项目 | 状态 | 当前影响 | 未来入口 |
|---|---|---|---|---|
| NA-EI-001 | credentials / runtime / production write / publish / trading | not-authorized | 不进入 CR-046 CP2 或实现默认授权 | 需要独立用户授权、安全边界与回滚方案 |
| NA-EI-002 | repository commit / push | not-authorized | 本轮只允许本地工件与验证 | 用户明确要求后按仓库发布流程处理 |
| NA-EI-003 | quant-lab lineage business-code changes | not-authorized | CR-163 仅 process-evidence append-only pilot | 新业务目标、独立 CR、HLD/LLD 与业务验收 |

## Gotchas

- `unavailable` 是证据状态，不是待实现功能；不得把“伪造 receipt/telemetry”放入 backlog。
- Deferred 不自动扩大 MVP。只有满足重启条件并通过新的范围决策后，才可进入 Story Map。
- CR-163 pilot 的回修只能追加 correction/supersession evidence，不能以“backlog 清理”为由改写历史行。
