# CP3 人工门禁发起消息 — CR-139 Strategy Data Foundation

**Checklist 路径**：`process/checks/CP3-CR139-HLD-CONSISTENCY.md`
**人工审查稿**：`process/checkpoints/CP3-CR139-HLD-REVIEW.md`
**Context Capsule**：`process/context/CP3-CR139-STRATEGY-DATA-FOUNDATION-CONTEXT.yaml`

## 自动预检结论
**PASS**（24 项 PASS，0 PARTIAL/FAIL）。BLUEPRINT v1.13 / DOMAIN-MAP v1.12 / DEPENDENCY-MAP v1.12 / FEATURE-DESIGN-MATRIX v1.12 刷新完成；companion HLD「Strategy Data Foundation」v0.1 写侧/读侧分离 14+ 章节 + 10 条实质性 Gotchas；ADR v0.1 落 D1-D8 + Wave1 基线门 + V4 冻结；四组门禁落 CP3 门禁；既有合同闭环边界标注；Wave1 基线门落入关键流程；UC→架构 traceability 完整；6 AGA table-first advisor，AGA-1/3/5 经 live 确认采纳推荐 A1/C1/E1。

## Context Capsule 摘要
来源 handoff v0.7 + CR-139 + USE-CASES/REQUIREMENTS v1.17 + BLUEPRINT/MATRIX v1.12 旧基线；read_profile=compact；默认读取策略为 must_read，未读完整 HLD-DATA-LAKE/LLD；HLD-DATA-LAKE.md 正文未修改（git diff 为空）。核心：companion HLD 写侧/读侧分离收口 HLD-DATA-LAKE；四组门禁为 CP3 必过；既有合同闭环非新建。

## 审批者摘要
- **本次确认服务的整体目标**：通过 CP3 HLD 架构评审门，冻结蓝图/领域/依赖/矩阵刷新 + companion HLD + ADR，确认 AGA-1/3/5 三项架构走向决策，为 CP4/CP5 Story+LLD 铺路。
- **推荐动作**：`approve`——接受 CP3 全部设计产物与 AGA-1/3/5 推荐方案（A1/C1/E1，已 live 确认）。
- **approve 后会发生什么**：CP3 通过，STATE 推进到 story-planning（CP4/CP5），按 Wave1 P0（T8/T7/C2a/V1/C1/R1/C2b）拆 Story+LLD；CP5 通过后才实现。HLD 锁解除，CP5 实现锁仍在。
- **approve 不授权什么**：不授权 Story 实现/LLD 落地执行/runtime/NAS/QMT/trading/provider-lake-catalog 写入/物理分区迁移/Git remote write；CP3 通过不等于授权实现或运行时。
- **不确认会阻塞什么**：CP3 未通过则不得拆 Story、不得写 LLD、不得实现（规则 2）。

## 决策分层
- 必须用户决策：AGA-1（A1）、AGA-3（C1）、AGA-5（E1）——已 live 确认，CP3 门禁为最终 ratification
- 高风险策略确认：Wave1 基线门（CP2 已定）、V4 schema 冻结前置模拟盘前（HIGH3）、D5 lake features/ 附条件切换
- agent 默认处理：AGA-2/4/6 既定、蓝图/领域/依赖/矩阵刷新细节、HLD 14 章节内容、ADR 回写
- 仅审计记录：D1-D8（CP2 已 approve，CP3 架构复核）、DEF-139-01 feature store 切换条件、CP5 读写冲突升级 A3 回退条件

## 决策收集覆盖
已扫描来源：CP3 context capsule AGA-1..6、handoff v0.7 §6 D1-D8、meta-se 交还摘要 CP3-DQ-01/02/03、Wave1 基线门/V4 冻结。候选问题数 11，纳入待决策数 3（AGA-1/3/5，已 live 确认），其余为既定/CP2 已 approve/审计层。分类 / N/A 原因：AGA-2/4/6 既定、D1-D8 CP2 已 approve。

## 本轮待人工决策项：3

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| CP3-DQ-01 | architecture | 写侧/读侧分离是否分层同属 FEAT-02 | A1 分层同属 FEAT-02 + HLD 分读写章 | A3 拆 FEAT-02R 读侧 | A1 不增 Feature 可回退；A3 边界最清但增复杂度 | Feature 边界/Story owner/文件所有权 | CP5 读写冲突频繁→升级 A3 |
| CP3-DQ-02 | architecture | ML feature 层是否 lake features/ 子层（D5） | C1 lake features/ 子层 + DEF-139-01 | C2 独立 feature store（deferred） | C1 不引入依赖与 release 闭环一致；C2 在线 serving 友好但增依赖 | V3 物理归属/依赖/切换路径 | feature 规模/在线 serving→切换 C2 另起 CR |
| CP3-DQ-03 | architecture | 配置类事实源版本化是否复用 V1 pointer 语义 | E1 复用 V1 pointer + config_facts 子目录 | E2 各类独立 registry | E1 机制统一复用 publish.py；E2 量身定制但四套机制 | F1-F4 release 闭环/publish.py 复用 | policy_cycle 差异过大→该类独立 |

> 三项均经 host-orchestrator live 用户确认采纳推荐方案（2026-06-28T17:10:00+08:00）。完整待决策表见人工审查稿。

## 用户需决策事项
本轮需用户决策事项：CP3-DQ-01 (AGA-1) 写侧/读侧分离=A1、CP3-DQ-02 (AGA-3) ML feature 层=C1、CP3-DQ-03 (AGA-5) 配置类事实源=E1。回复 `approve` 表示接受 CP3-DQ-01 (AGA-1) / CP3-DQ-02 (AGA-3) / CP3-DQ-03 (AGA-5) 三项推荐方案及 CP3 全部设计产物。

## 不授权项
Story 实现/LLD 落地执行 / runtime / NAS / provider / catalog 运行时与凭据 / QMT·MiniQMT·gateway 运行时 / 下单撤单·模拟盘·实盘 / 真实数据湖写入 / published pointer 前移 / 物理分区迁移（Wave1 N1 后置到基线冻结后）/ 实盘读审计写 / broker facts 实盘写 / Git remote write / publish·live·production 写入。CP3 通过解除 HLD 锁（可拆 Story），CP5 实现锁仍在。后续按 action scope 单独 `runtime_authorization`。

---

**如果你回复 approve**：接受待人工决策清单内全部推荐方案（AGA-1=A1 / AGA-3=C1 / AGA-5=E1）及 CP3 全部设计产物，CP3 通过，推进到 story-planning（CP4/CP5，按 Wave1 P0 拆 Story+LLD）。
**回复 `修改: <具体修改点>`**：调整单项架构决策（如 `修改: CP3-DQ-01=A3`），host-orchestrator 按规则 36 评估是否需回退刷新设计产物并重新发起确认。
**回复 `reject`**：CP3 不通过，回退到 solution-design 重新设计。

`approve` 表示接受推荐方案，**不表示授权**上述"不授权项"中的任何 Story 实现/LLD 落地/runtime/凭据/外部接口/数据写入/publish/live/交易类操作；这些必须独立形成 `runtime_authorization` 决策项，且 Story 实现需 CP5 通过。
