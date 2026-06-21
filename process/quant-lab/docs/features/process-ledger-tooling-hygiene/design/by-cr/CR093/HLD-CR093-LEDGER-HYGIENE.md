---
status: confirmed
version: "1.1"
complexity: "standard"
selected_option: "A. current-state semantic checker plus minimal ledger normalization"
confirmed: true
confirmed_by: "user"
confirmed_at: "2026-06-18T17:55:00+08:00"
source_cr: "CR-093"
parent_cr: "CR-092"
cp2_checkpoint: "process/checkpoints/CP2-CR093-LEDGER-HYGIENE-SCOPE-REVIEW.md"
---

# CR093 Ledger Hygiene HLD

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.1 | 2026-06-18 | host-orchestrator | 回填 CP3 approved：用户接受 DQ-CP3-CR093-01..04，允许进入 CP5 LLD / TEST-PLAN / TASKS。 |
| 1.0 | 2026-06-18 | host-orchestrator | 初始 HLD，覆盖 CR019 / CR025 历史账本旧账治理、checker 语义边界、候选方案、推荐方案、风险和 CP3 决策项。 |

## 1. 问题定义

CR093 要解决的是 `meta-flow check cr-tracking --project-root .` 当前被历史旧账阻断的问题。已知失败集中在两类：

| 类别 | 当前表现 | 影响 |
|---|---|---|
| CR019 follow-up tracking 状态语义不一致 | 旧台账中 `closed`、`closed-current-delivery`、`closed-spike-complete`、`cancelled-user-deleted` 与正式 CR frontmatter 不完全一致；旧 checker 仍用硬编码 Markdown 片段判断 | checker exit 1，无法作为后续门禁的干净健康信号 |
| CR025 / CR092 historical active_change 误扫 | `STATE.md` 历史记录里出现 nested `active_change`，checker 全文扫描后误判为当前 active pointer | 已关闭历史 CR 被当作当前冲突，干扰 active formal CR 识别 |

### 目标

| 优先级 | 目标 | 可度量成功标准 |
|---|---|---|
| P0 | 区分“当前权威状态”和“历史审计文本” | checker 当前状态读取入口限定为 3 类：顶层 STATE、正式 CR frontmatter、当前 tracking / index 表；history / discussion 只进入 audit-only 扫描 |
| P0 | 建立 closed / cancelled 状态等价关系 | 至少覆盖 CR020、CR030、CR040、CR041、CR042、CR043、CR044、CR045、CR021..024 的当前正式状态 |
| P0 | 保留历史追溯 | 不删除 `STATE.md.history` 和 CR019 台账历史段落；所有文本修订必须是最小规范化 |
| P1 | 让 cr-tracking 输出可恢复 | summary 必须包含 active formal CR、blocked formal CR、follow-up candidates、spike candidates、stale status conflicts 5 类 |
| P1 | 限制权限边界 | 0 次 NAS、runtime、凭据、账户、交易、provider、lake、publish 操作 |

### 非目标

- 不重写整个 Meta Flow 状态机。
- 不恢复 CR020 用户删除的 gateway 路线。
- 不启动 CR089、CR091-FU-02、CR091-FU-03。
- 不删除历史审计文本。
- 不读取数据目录、报告目录、NAS、凭据、账户或交易日志。

## 2. Architecture Gray Areas

| 灰区 ID | 关键问题 | 为什么影响架构 | 影响面 | 推荐 |
|---|---|---|---|---|
| AGA-CR093-01 | checker 应继续全文扫描，还是分层读取当前状态与 audit-only 历史文本？ | 决定是否彻底修复 nested active_change 误报 | checker parser / STATE / history | 推荐分层读取 |
| AGA-CR093-02 | CR019 旧台账是改文本，还是让 checker 接受状态等价关系？ | 决定修复是数据规整还是规则语义化 | CR019 tracking / formal CR status | 推荐规则语义化 + 最小文本规范化 |
| AGA-CR093-03 | source=cp8-follow-up 缺少 tracking row 的旧警告是否一并强修？ | 可能扩大到 CR053/058/074/075/078/080/092 等旧 CR | CR-INDEX / follow-up tracking | 推荐本轮只保留 warning，不作为 exit 1 阻断 |
| AGA-CR093-04 | 是否直接把旧失败 suppress 掉？ | 决定后续是否还能发现真实状态冲突 | checker quality / audit risk | 不推荐 suppress；应有分类和证据 |

## 3. 候选架构方案

| 方案 | 描述 | 优点 | 代价 / 风险 | 结论 |
|---|---|---|---|---|
| A | 当前状态语义化 checker + 最小台账规范化 | 修根因；保留历史；可测试；改动面受控 | 需要新增 parser / fixture 测试 | 推荐 |
| B | 只改 CR019 / STATE 文本迎合旧 checker | 代码改动少 | 保留全文误扫缺陷，后续 history 仍可能误报 | 不推荐 |
| C | 对已知错误做 suppress | 短期最快 | 容易隐藏真实冲突，审计价值低 | 不推荐 |
| D | 重写全量 CR ledger checker | 可长期统一 | 本轮成本高，容易扩大范围 | 备选，未来 Spike |

## 4. 推荐方案

推荐方案 A：把 checker 改为“当前状态语义化读取 + audit-only 历史扫描 + 最小台账规范化”。

### 模块职责

| 模块 / 文件 | 责任 | 允许变更 | 禁止变更 |
|---|---|---|---|
| `scripts/check_cr_tracking_consistency.py` | 读取权威当前状态、状态等价关系、输出分类 summary | CP5 approved 后可改 parser、状态集合、验证输出 | 不读取外部数据、不访问网络、不读取凭据 |
| `process/changes/CR-019-FOLLOW-UP-TRACKING-2026-05-31.md` | 保留 CR019 follow-up 历史与当前行 | CP5 approved 后可做最小状态规范化和说明补充 | 不删除历史决策 |
| `process/STATE.md` | 当前顶层状态与历史审计记录 | 仅允许更新当前 CR093 状态和必要历史记录 | 不重写旧 history |
| `process/changes/CR-INDEX.yaml` | active / blocked / candidate 索引 | 同步 CR093 状态和后续门禁 | 不自动启动其他 CR |

### 状态读取规则

| 数据层 | 来源 | 用途 | 是否作为当前缺陷 |
|---|---|---|---|
| current | STATE 顶层字段、orchestrator_session 当前字段、正式 CR frontmatter、CR-INDEX active / blocked 列表 | 判断当前 active / blocked / candidate | 是 |
| tracking-current | follow-up tracking 当前表格行 | 判断 candidate / active / closed 映射 | 是 |
| audit-history | STATE history、discussion log、旧叙述段落、backup | 人类审计和恢复 | 默认否；只报告为 audit warning |

## 5. 关键流程

1. 读取当前顶层 `STATE.md.active_change` 和 `cr_tracking.active_crs[]`。
2. 读取正式 CR frontmatter，按状态等价表分类为 active / blocked / closed / cancelled。
3. 读取 follow-up tracking 当前表格，做候选项与正式 CR path 映射。
4. 对 history / discussion / backup 命中的 nested active_change 只生成 audit warning，不作为 current failure。
5. 输出 5 类固定 summary，并返回符合门禁语义的 exit code。

## 6. Use Case Traceability

| Use Case | 推荐架构支持 | 验证方式 |
|---|---|---|
| UC-CR093-01 无 active CR 时不被历史旧账阻断 | audit-history 不作为 current failure | fixture：顶层 active 空，history 含旧 active_change，checker PASS 或 warning-only |
| UC-CR093-02 CR093 active 时准确显示 active formal CR | current 层读取正式 CR frontmatter 和 CR-INDEX | `meta-flow check cr-tracking` summary 包含 `CR-093` |
| UC-CR093-03 CR019 closed 等价状态不误报 | 状态等价表覆盖 closed-current-delivery / closed-spike-complete / cancelled-user-deleted | fixture 覆盖 CR020/030/040..045/021..024 |
| UC-CR093-04 后续 candidate / spike 可解释 | tracking-current 层保留 candidate / spike candidate | summary 保留 CR026、CR027、CR028 |

## 7. 关键场景模拟

| 场景 | 输入 | 期望结果 | 结论 |
|---|---|---|---|
| S1 当前 active CR 为 CR093 | 顶层 `active_change=CR-093`，正式 CR status active | summary 显示 `active formal CRs: CR-093` | PASS |
| S2 历史 CR025 active_change 留在 history | `STATE.md.history` 中存在 nested `active_change: CR-025` | 仅 audit warning，不 exit 1 | PASS_BY_DESIGN |
| S3 CR019 台账 closed-current-delivery 与 formal status 不同字面 | formal CR status 使用 closed-current-delivery | 状态等价后不报 mismatch | PASS_BY_DESIGN |
| S4 真实 active 指向 closed CR | 顶层 active_change 指向 formal closed CR | 仍报 current failure | PASS |

## 8. 非功能设计

| 质量属性 | 设计要求 | 验证 |
|---|---|---|
| 可审计性 | 不删除历史文本；所有规范化保留来源说明 | diff review |
| 可维护性 | 状态等价关系集中定义，避免散落硬编码片段 | 单元测试 |
| 安全性 | checker 只读 Markdown / YAML，不访问外部系统 | 静态扫描 / 命令记录 |
| 可恢复性 | 保留原始 CR / history，最小改动可回退 | git diff / rollback notes |

## 9. 风险与缓解

| 风险 ID | 风险 | 等级 | 缓解 | 回退 |
|---|---|---|---|---|
| R-CR093-01 | 把真实 current conflict 错降为 audit warning | high | current / audit 数据层显式区分，并用 fixture 覆盖 S4 | 回退 checker parser |
| R-CR093-02 | 旧台账规范化损失追溯 | medium | 只做表格状态规范化，保留历史说明 | revert 文档改动 |
| R-CR093-03 | 顺手修 source=cp8-follow-up warnings 扩大范围 | medium | 本轮只记录 warning，不纳入 exit 1 修复 | 拆 CR |
| R-CR093-04 | 误把 CP3 approve 当作实施授权 | high | CP3 / CP5 分离；CP3 只冻结架构 | 回退到 pending CP3 |

## 10. ADR 候选

| ADR | 决策 | 推荐 | 备选 | 切换条件 |
|---|---|---|---|---|
| ADR-CR093-01 | 当前状态读取模型 | current / tracking-current / audit-history 三层 | 全文扫描 | 审计要求必须全文强校验时改为分层强校验 |
| ADR-CR093-02 | 状态等价策略 | 集中状态等价表 | 修改所有旧台账为统一字面 | 若状态集合过大且不可维护，转 schema 化 |
| ADR-CR093-03 | warning 范围 | cp8-follow-up missing row 保持 warning | 本轮全部修复 | 用户要求清 warnings 时另起 CR |

## 11. 分阶段落地

| 阶段 | 内容 | 出口 |
|---|---|---|
| CP3 | 冻结 HLD 和 ADR | 用户 approve DQ-CP3-CR093-01..04 |
| CP5 | 输出 LLD / TEST-PLAN / TASKS | 明确文件级改动、fixture 和回滚 |
| CP6 | 实施 checker / 最小台账规范化 | 单元测试和静态检查通过 |
| CP7 | 验证 cr-tracking 目标缺陷收敛 | `cr-tracking` 不再被 CR019 / CR025 阻断 |
| CP8 | 关闭 CR093 | 风险接受和 follow-up 分流完成 |

## 12. 不授权范围

CR093 的 CP3 approve 不授权：

- 修改 checker 或旧账本正文。
- QMT / MiniQMT / XtQuant / gateway / runner runtime。
- NAS 访问。
- `.env`、凭据、账号、账户、资金、持仓、委托、成交或日志原文读取。
- submit / cancel、buy / sell、simulation / live。
- provider fetch、lake write、catalog publish。
- 自动启动 CR089、CR091-FU-02、CR091-FU-03 或恢复 CR020。

## 13. HLD 拆分判定

本 HLD 只有一个核心产物：CR tracking ledger hygiene。Story 数预计为 1 个，ADR 聚焦同一 checker / ledger 语义边界；无需拆分 companion HLD。

## 14. 自审

| 检查项 | 结论 |
|---|---|
| 至少 2 个候选方案 | PASS，4 个 |
| Architecture Gray Areas 前置 | PASS，4 个 |
| 成功标准可度量 | PASS |
| 不授权项明确 | PASS |
| 场景模拟覆盖关键风险 | PASS |
