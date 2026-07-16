---
status: draft
version: "1.0"
feature_id: "FEAT-GB-02"
feature_name: "Safe Ref Publish"
source_blueprint: "process/docs/design/CR050-GIT-BRANCH-BLUEPRINT.md"
source_hld: "process/docs/design/CR050-GIT-BRANCH-HLD.md"
source_adr: "process/docs/design/CR050-GIT-BRANCH-ARCHITECTURE-DECISION.md"
source_matrix: "process/docs/design/CR050-FEATURE-DESIGN-MATRIX.md"
related_stories: ["ST-GB-002"]
lld_policy_summary: "1 full-lld"
---

# Feature Design: Safe Ref Publish

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-16 | host-orchestrator-inline/meta-se | 冻结existing-commit-only publish、matching evidence和双仓PARTIAL。 |

## 摘要、背景与边界

publish只把调用开始时已存在的local HEAD发布到同名remote CR branch。它负责clean/identity/upstream/non-FF/fresh OID检查和matching evidence；绝不执行`git add`、`commit`、`amend`、merge或default更新。来源为UC-GB-002、REQ-GB-005/006/010、TC-GB-004/005/009/010。

| 模块 | 变更 | 输入 | 输出 | 失败 |
|---|---|---|---|---|
| publish planner | 新增 | 2/2 snapshots、expected local heads | deterministic plan | dirty/wrong/diverged→BLOCKED |
| executor | 扩展 | ordinary branch ref push steps | per-repo outcomes | 后仓失败→PARTIAL |
| evidence writer | 扩展 | validated outcomes | matching publish attempt | 字段缺失不能供merge消费 |
| CLI | 接线 | CR/branch/dry-run/output | exit+JSON/ref | 不隐式调用merge |

## 数据、接口与流程

`PublishEvidence`必须含CR/repo/branch、local expected OID、remote observed OID、terminal、attempt/result ref与freshness。`plan_publish`先观察2/2仓，要求clean/attached/expected branch/upstream以及remote tip是local expected tip祖先或相等；全仓preflight通过后按确定顺序ordinary push exact branch ref，每仓post-query必须等于expected local HEAD。只把调用开始时捕获的OID写入plan，执行期间HEAD变化导致post条件失败而不是自动发布新提交。

## 权限、安全与失败恢复

- publication authz只授权CR branch，不授权default或delete；remote策略拒绝保留原语义。
- 禁止shell/force/auto-commit；stderr和URL redaction。
- preflight失败2/2 writes=0；第一仓成功第二仓失败为PARTIAL；resume fresh观察，已成功仓exact match才NO_CHANGE。

## 测试、顺序与下游

unit覆盖evidence schema/旧attempt/HEAD drift；bare integration覆盖2/2 publish、dirty、wrong upstream、non-FF、partial/resume、dry-run；command spy断言stage/commit/amend/force调用=0。先复用ST-GB-001 types，再实现publish handler/evidence/CLI，最后测试。ST-GB-004只能消费matching evidence和fresh remote refs，不得只信历史result。

## 风险与回退

用户若未来需要工具代为选择文件并commit，必须单独CR定义path allowlist、secret scan、message与确认，不能扩张本Feature。
