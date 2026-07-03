---
status: baseline
version: "1.0"
complexity: "complex"
confirmed: true
cr_ref: "CR-037"
confirmed_by: "user"
confirmed_at: "2026-07-02T19:43:16+08:00"
source_hld: "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-HLD.md"
---

# Meta Flow Project Governance Architecture Decisions

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-02 | meta-se | 记录 current state enforcement、project state、registry、roadmap refresh 和迁移边界的核心 ADR |

## Agent/Skill 组合方案

| 角色 / Skill | 使用方式 | 输入 | 输出 | 说明 |
|---|---|---|---|---|
| meta-se | 蓝图、HLD、ADR、Feature 矩阵设计 | 已批准实施计划、STATE、CR index | 本设计文档集 | 不改代码，不改产品场景文档 |
| blueprint-design | 定义 Feature / Capability 边界和依赖方向 | 实施计划 P0/P1/P2 | BLUEPRINT / DOMAIN-MAP / DEPENDENCY-MAP | 蓝图适用，因存在多个 Feature 和数据归属问题 |
| hld-designer | 输出 HLD 与架构灰区 | 蓝图、实施计划 | HLD、ADR 候选 | 灰区由已批准实施计划收敛 |
| implementation-design | 仅用于 Feature 设计矩阵适用性判断 | HLD、蓝图、ADR | FEATURE-DESIGN-MATRIX | 不生成 Story，不进入实现 |

## 平台适配差异

| 平台 / 空间 | 设计结论 | 风险控制 |
|---|---|---|
| meta-flow 源码仓库 | 实现 CLI、schema、checker、contract guardrail | 通过 Story / LLD / tests 进入后续实现 |
| meta-flow-artifacts 过程归档库 | 保存 process、project state、roadmap、refresh result、ledger、migration evidence | roadmap refresh 自动写入范围仅限过程侧 |
| quant-lab 发布库 | 作为 P2 只读样本和 stale/follow-up 目标 | 不自动修改代码、tests、正式 docs |

## 设计确认点（需人工确认）

| Decision ID | 决策类型 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|
| ADR-PG-001 | architecture | `STATE.current.json` 使用 allowlist schema + field budgets，audit -> enforce | 继续黑名单补丁 | 推荐方案可阻止未知字段；备选成本低但无法防止下一个自造字段 | 影响所有默认读取入口和状态写入 | audit 期间发现合法字段缺失时扩展 schema，不回退黑名单 |
| ADR-PG-002 | architecture | 新增 `PROJECT.current.json`，current state 只保存 `project_state_ref` | 将 project fields 直接放入 current state | 推荐方案保持 current state 瘦身；备选实现简单但会膨胀 | 影响 P1 项目治理对象和 P0 allowlist | 如果 project state 超预算，收紧 project schema |
| ADR-PG-003 | security | roadmap refresh 只自动写过程归档库，发布库只输出 stale / follow-up | 自动修改 quant-lab 发布库 | 推荐方案符合授权边界；备选自动化高但跨仓风险高 | 影响 runtime authorization、rollback、审计 | 未来如需发布库写入，必须单独 CR + human gate |
| ADR-PG-004 | implementation | `feature_refs` / `capability_refs` 必须引用 YAML registry | 长期消费 markdown / Python 常量或自由字符串 | 推荐方案可机器校验；备选迁移快但不可持续 | 影响 impact migration 和 quant-lab capability 归一 | registry 不足时扩展 registry，不退回自由字符串 |
| ADR-PG-005 | architecture | ROADMAP-REFRESH 使用独立 result schema 和 checker | 复用 CP result checker | 推荐方案语义清晰；备选减少 checker 数量但污染 CP 语义 | 影响 checkpoint/result 边界 | CLI 可聚合 UX，但 schema 不合并 |
| ADR-PG-006 | follow_up_tracking | roadmap follow-up 使用 FU-RF / SP-RF / RA-RF，不写 `RELEASE-CONTEXT` | 写入 release context 或普通 follow-up | 推荐方案区分项目治理和发布上下文；备选检索简单但语义混乱 | 影响 CR tracking、release readiness | 若 FU-RF 过重，可在 CR tracking UI 聚合 |

## 变更记录

| 日期 | ADR | 变更 |
|---|---|---|
| 2026-07-02 | ADR-PG-001..006 | 初始记录，来自已批准实施计划和本轮 HLD |
