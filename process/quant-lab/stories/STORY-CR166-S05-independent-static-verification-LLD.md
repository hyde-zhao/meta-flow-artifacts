---
story_id: "CR166-S05-independent-static-verification"
title: "Independent fixture/static verification and claim audit"
story_slug: "independent-static-verification"
lld_version: "0.1"
tier: "L"
status: "confirmed"
confirmed: true
created_by: "host-orchestrator inline meta-dev"
created_at: "2026-07-13T12:34:00+08:00"
confirmed_by: "user"
confirmed_at: "2026-07-13T13:43:35+08:00"
feature_design_refs: ["docs/features/walk-forward-oos-verification/DESIGN.md", "docs/features/walk-forward-oos-verification/TEST-PLAN.md", "docs/features/walk-forward-oos-verification/TASKS.md", "docs/features/strategy-evidence-envelope/TEST-PLAN.md", "docs/features/walk-forward-oos-validation/TEST-PLAN.md", "docs/features/walk-forward-oos-producer/TEST-PLAN.md", "docs/features/walk-forward-oos-projections/TEST-PLAN.md"]
lld_policy: {required_level: "full-lld", trigger_reasons: ["cross-story-verification", "authorization-audit", "historical-failure-attribution", "Stage-claim-ceiling"], rationale: "Independent CP7 evidence spans every CR166 QAC and claim boundary."}
open_items: 0
---

# LLD: CR166-S05 — Independent Fixture/static Verification

## 0. 上游设计依据（工程依据）

CR166 HLD §1.3/8/11/14、ADR-008..010、all five Feature TEST-PLANs、SC-CR166-P01/P02/N01..N06/A01/H01/E01、QAC-01..12。

## 1. 目标（Goal）

建立独立本地 fixture/static 验证 lane，证明 contract、validator、producer、projection、permission 与 claim ceiling；不产生真实策略或真实 OOS 结论。

## 2. 需求（Requirements）

11/11 scenarios、12/12 QAC、8/8 negative classes、daily/ML=2/2、event N/A=1/1、consumer=3/3、10→1、operation=0、Stage flags 4/4、CR166 failure=0、触及 CR165 历史失败归因率=100%。

## 3. 模块拆分与职责

| 测试面 | 文件 |
|---|---|
| QAC/traceability | `tests/research/test_walk_forward_oos_qac.py` |
| authorization/static/claim | `tests/research/test_walk_forward_oos_authorization.py` |
| CR155/CR165 regression attribution | `tests/research/test_walk_forward_oos_cr155_regression.py` |
| fixtures | `tests/fixtures/walk_forward_oos/` |

Feature-level unit/integration tests 仍由 S01-S04 files owner；S05 只聚合公共行为，不复制生产算法。

## 4. 代码结构与文件影响

只新建上述 test/fixture paths；生产文件全部 read-only。CP7 evidence index/report 在 `process/evidence` / `docs/quality` 由验证阶段生成，不在实现测试中硬编码路径或写入仓库状态。

## 5. 数据模型与 Fixture 边界

daily 与 ML synthetic fixtures 只含最小日期、数值、logical refs；8 类 negative 通过局部 override 构造。严禁复制真实市场数据、真实 run ids、credentials、NAS/lake paths。event 只有 applicability fact，不建 fold fixture。

## 6. API / 测试接口与执行层

分层命令均通过 `uv run pytest`：contract→validation→producer→projection/regression→QAC/auth→full suite。每层记录 command/exit/count/hash/operation counters。full suite 不是用历史失败作 blanket waiver；只有实际触达 CR165 已重基线测试时逐项记录 test id、call path、baseline ref、归因。

## 7. 核心验证流程与判定

先验证 fixture inventory/expected refs，再执行正向；随后按 8 类 negative/boundary；再做 tamper/permission/static scan；最后 consumer/CR155/Stage claim。任一 false PASS、external call、CR155 promotion、Stage3/real evidence true、未归因触达失败→FAIL/NEEDS_REWORK。

## 8. 技术细节

QAC 测试不得仅断言测试数；需断言实际 result fields/counts/reasons。10-run 在同进程固定 input 重复；不使用 freeze-time 伪造确定性。operation counters 和 monkeypatch call guards 双重验证。event producer modules/fixtures 数显式为 0。

## 9. 安全与性能

禁止 network/provider/lake/NAS/env/credential/runtime/broker/trading/deploy/publish/remote write。测试只写 pytest 临时目录（若必要），不得写 catalog/store/current pointers。性能只验证无 I/O 与线性路径，不声明真实规模吞吐。

## 10. 测试设计

Feature plans 共 32 test groups，S05 汇总 12 QAC。覆盖 7 fields、3 leakage、2 adapters、3 consumers、8 fail classes、4 Stage flags、C3/C4=0、event producer=0、CR155 promotion=0、CR165 attribution。

## 11. 实施步骤

T01 fixture catalog；T02 QAC assertions；T03 auth/claim guards；T04 CR155/CR165 attribution；T05 layered execution/evidence index/report。

## 12. 风险与 Gotchas

无 OPEN/LCQ。Gotchas：fixture PASS≠real OOS；static scan≠authorization；测试数≠coverage；历史 failure 只能逐项归因；event N/A 不得被计为 producer coverage。

## 13. 回滚与发布

生产 Story 回退时相应 tests 标为 unavailable/删除调用而不删除安全要求；CP7 报告保留审计。无真实发布。

## 14. DoD / Definition of Done

- [x] 11/11 scenarios、12/12 QAC、8/8 negative 有独立证据
- [x] dereference/forbidden/CR155 promotion/overclaim=0
- [x] CR166 new-path failures=0；touched CR165 attribution=100%
- [x] CP5 已批准，`confirmed=true`；只允许 synthetic fixture/static 验证
