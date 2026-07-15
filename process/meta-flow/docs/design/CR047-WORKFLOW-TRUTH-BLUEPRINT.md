---
status: accepted
version: "1.1"
cr_ref: "CR-047"
source_story_map: "process/docs/product/STORY-MAP.md"
source_mvp_scope: "process/docs/product/MVP-SCOPE.md"
confirmed_by: "user"
confirmed_at: "2026-07-14T15:29:31Z"
---

# CR-047 Workflow Truth and Delivery Governance Blueprint

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.1 | 2026-07-14 | host-orchestrator-inline / meta-se | CP3 R2：加入动态 Doctor 基线、CR-046 对象身份制 hash firewall、CP6/CP7 双重校验与叠加式验证结论上限。 |
| 1.0 | 2026-07-13 | host-orchestrator-inline / meta-se | 建立 CR-047 能力边界、数据归属、依赖方向与 CP3 决策项。 |

## 蓝图结论

CR-047 不新建“总状态服务”、第二套质量模型或新 ledger。它在已有 State v2、CR lifecycle/index、CURRENT 投影、workspace routing、Doctor、guardrail 和 installer 边界内修复 contract，用一个只读组合门证明它们一致。

## 能力地图

| Capability ID | 能力域 | 用户价值 | 覆盖 Story | Owner Feature |
|---|---|---|---|---|
| CAP-WT-01 | Canonical workflow truth | closed/missing CR 不再驱动活动流程 | ST-WT-001 | FEAT-WT-01 |
| CAP-WT-02 | Portable process/document routing | clean clone 经一次 link 即得到唯一过程真相 | ST-WT-002 | FEAT-WT-02 |
| CAP-WT-03 | Severity-aware quality governance | Doctor 可区分 blocker、warning 与 legacy unavailable | ST-WT-003 | FEAT-WT-03 |
| CAP-WT-04 | Clean-tree delivery integrity | clean clone 的规则、cache、lint 和回归可确定判定 | ST-WT-004, ST-WT-005 | FEAT-WT-04 |
| CAP-WT-05 | Operator-ready installation and status disclosure | CI/Agent 可非交互 dry-run，CR-046 当前风险如实呈现 | ST-WT-006, ST-WT-007 | FEAT-WT-05 |

## Feature / Epic 边界

| Feature ID | 名称 | 负责事项 | 不负责事项 | 拥有数据 | 只读数据 | 禁止依赖 |
|---|---|---|---|---|---|---|
| FEAT-WT-01 | Truth Consistency | State v2/CR catalog/CURRENT 关系校验，JSON-only CR tracking，CR-033 candidate | 不把 Markdown summary 升格为机器真相；不自动激活 candidate | consistency result 与查找结果；各源数据仍由原 owner 写入 | State、CR index/ledger、CURRENT | 不得直接改写历史 CP/result/ledger |
| FEAT-WT-02 | Artifact Routing | `process` 相对路由、route health、内部 docs canonical 路径校验 | 不创建根 `docs/product|design|quality` 可写副本；不处理 prelink backup | `.meta-flow-process.yaml` 与 route health result | State routing ref、artifact docs tree | 不得依赖设备绝对路径或 quant-lab 业务状态 |
| FEAT-WT-03 | Quality Governance | artifact budget 分类、Quality Model source、read-expansion correction、Run ledger future contract、Doctor exit semantics | 不截断或原位美化历史证据；不伪造 run 时间 | quality policy 和 derived diagnostic result | CP results、ledgers、retention policy | 不得依赖手工 dashboard/score 作为 truth |
| FEAT-WT-04 | Delivery Integrity | tracked rule source contract、generated wrapper dry-run、cache classification、Ruff/pytest/release preflight | 不提交个人 Memory Policy；不把 ignored cache 当交付包输入 | tracked delivery source 和 preflight result | Git tracked set、installer dry-run、Python source/tests | 不得把 ignored 根 wrapper 作为 clean-clone 必需输入 |
| FEAT-WT-05 | Operator & Status Convergence | README 非交互命令、CR-046 current-state 回链、READY_WITH_RISK 披露 | 不执行真实用户安装、runtime、SaaS、publish 或 Git push | 用户文档与当前状态投影 | installer contract、CR-046 formal evidence | 不得将 fixture rejection 写为 platform receipt |

## 数据归属规则

| 对象 | 唯一写 Owner | 派生消费者 | 写入规则 |
|---|---|---|---|
| `STATE.current.json` | `meta_flow.state.current.update_current_state` | state check、CURRENT builder、CR tracking | 受控 writer + allowlist/budget；不允许 summary 反向覆盖 |
| `CR-INDEX.json` + formal CR + `CR-LEDGER` | CR lifecycle/status-sync | CR tracking、CURRENT builder | status-sync 保持 lifecycle/gate/readiness 一致 |
| `process/current/CURRENT.json` | `state current-refresh` | Agent/CLI 发现层 | 仅从 State/CR/refs 重建，不接受手工业务状态 |
| `.meta-flow-process.yaml` | workspace link | workspace check、State routing ref | 只使用 anchor + relative path；重连不得只因时间戳制造漂移 |
| Internal docs | artifact `process/meta-flow/docs/**` | CP/Doctor/Reviewer | 唯一 canonical 可写副本；根 `docs/` 仅公开入口 |
| Quality diagnosis | Doctor/checker 派生输出 | CP7/CP8/CI | 只从 policies/results/ledgers 派生，warning 不反写为历史 PASS |
| CR-046 protected-object manifest | ST-WT-007 scope firewall | CP6/CP7 checker | 逐对象记录 source CR、object type、canonical/provenance ref、原始 SHA256 与 immutable 操作；不得把目录前缀当对象身份 |

## 跨 Feature 流程

| Flow ID | 触发 | 参与 Feature | 数据写入 Owner | 失败路径 | 验证入口 |
|---|---|---|---|---|---|
| FLOW-WT-01 | clone/link/recover | FEAT-WT-02 → FEAT-WT-01 | routing owner 写 metadata；state/CR owners 各自写 source | 路由错误立即 BLOCKED；truth conflict 输出字段级 finding | TC-WT-001, TC-WT-002 |
| FLOW-WT-02 | release preflight | FEAT-WT-01 → FEAT-WT-03 → FEAT-WT-04 | 各 checker 仅写 result/run event | 任一 blocker 非零；warning 汇总但不伪装 blocker | TC-WT-003..005 |
| FLOW-WT-03 | operator installation | FEAT-WT-04 → FEAT-WT-05 | installer 只在 dry-run 生成 plan | 缺 `--project-dir` 时文档契约测试失败；不进行真实安装 | TC-WT-006 |
| FLOW-WT-04 | historical status convergence | FEAT-WT-01 → FEAT-WT-05 | CR-046 保持 closed；CP6 pre-implementation 生成对象身份制 manifest；只追加 projection/correction | CP6 完成首验、CP7 再验；protected original 变更立即阻断并拆子 CR | TC-WT-007 |

## 共享能力

| Shared ID | 名称 | 使用方 | Owner | 调用方向 | 降级策略 |
|---|---|---|---|---|---|
| SH-WT-01 | Project-relative reference resolver | FEAT-WT-01..05 | existing workspace/state helpers | Feature → resolver | ref 无法解析时失败关闭，不猜测绝对路径 |
| SH-WT-02 | Typed event append/check | FEAT-WT-01,03,04 | existing event ledger | Feature → event API | 对历史缺口写 legacy/unavailable correction，不倒填 |
| SH-WT-03 | Git tracked/package input classifier | FEAT-WT-04,05 | delivery preflight | guardrail/installer → classifier | 不能分类时对 package input 失败关闭 |
| SH-WT-04 | Protected-object identity resolver | FEAT-WT-01,03,05 | existing ref/evidence helpers | firewall → formal CR/evidence index/ledger refs | 身份或原 hash 不可确定时阻断；禁止仅按路径前缀放行 |

## 待确认边界

| Decision ID | 决策类型 | 问题 | 推荐方案 | 备选方案 | 推荐 / 备选优劣 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| CP3-DQ-01 | architecture | 真相一致性放在单一“总真相”还是 source-owned graph | 保留 State/CR/CURRENT 独立 owner，加入只读关系校验 | 新建单一总状态文件；或仅修 Markdown parser | 推荐不增加平行真相；代价是需要显式投影顺序 | 若写 owner 边界不清，漂移会再发 | 只有现有源无法表达新对象时才另立 CR |
| CP3-DQ-02 | architecture | 内部 docs 是否保留根级兼容 symlink | 只使用 `process/docs/**` canonical，引用同步归一，根 `docs/` 仅公开文档 | 建立 ignored `docs/product|design|quality` symlink；或把内部 docs 放回源码仓 | 推荐跨设备最简且可审计；代价是需要修正 legacy refs | 兼容 symlink 会恢复“本机有、clone 无”的歧义 | 只有外部工具硬性要求根路径时才引入单向 generated view |
| CP3-DQ-03 | risk_acceptance | Doctor 如何在不改写历史下归零 blocker | `B0_pre` 保留历史快照；CP7 采集动态 `B0_cp7` 并解释 delta；active/default-required 严格阻断，cold history 使用 summary/index/hash/correction | 固定 observed count；全局提高预算；原位截断（禁止） | 推荐可判定且不把 CR 自身产物合理化；代价是需记录分类与 delta | 错分类可能隐藏当前必读证据 | `blocking_active` 或 `unclassified` 非零即阻断 |
| CP3-DQ-04 | architecture | 规则/cache/lint 是否用新外层 pipeline | 扩展现有 guardrail/installer/checker；tracked/package cache block，ignored cache warn，Ruff+pytest 进 preflight | 新建独立 release orchestrator；或保持手工命令 | 推荐不增加调度层且可重用现有 CI；代价是 guardrail 需分类 finding | 单一大脚本可维护性风险 | guardrail 职责实际分裂为多团队/多语言时再拆组合层 |
| CP3-DQ-05 | architecture | ST-WT-007 如何证明未改写 CR-046 原件 | 对象身份制 manifest + before/after SHA256；CP6 pre-implementation 生成、CP6 完成首验、CP7 再验 | 仅人工审查；路径前缀 allowlist（禁止） | 推荐可机器证明且同目录对象不误判；代价是要解析 formal refs/evidence index/ledger | 漏列或错误身份会破坏历史完整性 | 任一 protected original 变更即阻断并拆子 CR/HLD |
| CP3-DQ-06 | risk_acceptance | 缺少独立 QA/平台证据时结论上限 | 有效 inline fallback 下 CP7 最高 `PASS_WITH_RISK`；任一继承风险 OPEN 时 CP8 最高 `READY_WITH_RISK` | CP7 前取得真实独立 QA/receipt；否则阻断 READY 诉求 | 推荐诚实反映证明层级；代价是本 CR 不会得到无风险 READY | 将 fixture/inline 误写为独立 attestation | 上限与 CR-046 各项继承风险叠加且独立计算 |

## Gotchas

- “一致”不等于“都写一份文件”；State、CR catalog 和 CURRENT 必须保持不同语义。
- `process/docs/**` 是当前 meta-flow 的 artifact canonical；不应因为 Skill 模板使用 `docs/**` 逻辑名称就创建本机副本。
- archive 不是删除。任何移动都必须保留 ref/hash/index，并且不可破坏 ledger-linked 路径。
- ignored cache 只能在确认未进入 package/staging 时降为 warning。
