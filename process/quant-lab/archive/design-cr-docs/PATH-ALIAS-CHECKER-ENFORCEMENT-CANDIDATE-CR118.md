---
title: "Path Alias Checker Enforcement Candidate"
cr_id: "CR-118"
status: "closed-ready"
created_at: "2026-06-22T18:21:08+08:00"
created_by: "host-orchestrator"
---

# Path Alias Checker Enforcement Candidate

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| v0.1 | 2026-06-22 | host-orchestrator | 从 `FU-CR117-001` 创建 CR118 checker enforcement 候选说明，供 CP2 审查。 |
| v0.2 | 2026-06-22 | host-orchestrator | CP2 approved；补充 owner discovery 和 CP5 implementation gate 推荐范围。 |
| v1.0 | 2026-06-22 | host-orchestrator | CP5 approved 后完成最小 checker enforcement 与 fixture 测试，进入 CP8 readiness。 |
| v1.1 | 2026-06-22 | host-orchestrator | CP8 approved；CR118 closed READY，不新增 follow-up。 |

## 背景

CR117 已确认路径展示约定：新增 checkpoint、CR、context capsule 和人工门禁提示优先引用 `process/...` 路径，`docs/design/*` 可作为历史 / 语义别名保留，不作为唯一证据路径。

CR118 的候选问题是：是否需要把该约定进一步转为机器可检查的 enforcement，例如在未来 checker 中发现新门禁或新 CR 只写 `docs/design/*` 而没有 `process/...` 证据入口时给出提示或失败。

## 推荐路线

| 阶段 | 推荐动作 | 当前授权 | 说明 |
|---|---|---|---|
| CP2 | 确认范围、非目标和授权边界 | 已启动 | 当前只写过程材料，不改源码 / tests / checker。 |
| CP5 / implementation gate | 若 CP2 通过，定义最小 checker owner、fixture 和测试范围 | 未授权 | 需要用户后续批准。 |
| 实现 / 验证 | 最小化修改 checker 与测试 fixture | 未授权 | 不连接 runtime、不访问 NAS、不读取凭据。 |
| CP8 | 汇总检查结果和剩余风险 | 未开始 | 仅在实现和验证门禁完成后进入。 |

## 非目标

- 不在 CP2 期间修改源码。
- 不在 CP2 期间修改 tests。
- 不在 CP2 期间修改 checker implementation。
- 不批量重写历史 `STATE.md`、checkpoint 或旧 CR 正文。
- 不修改目录、symlink 或文件位置。
- 不启动 runtime、NAS、凭据、交易写或 provider/lake/catalog publish。

## CP2 关注点

| 关注点 | 推荐判断 | 风险 |
|---|---|---|
| enforcement 必要性 | 可作为后续实现候选，但应先限制为新增门禁 / CR 的路径引用检查 | 过早扩大到历史全文扫描会制造大量噪音 |
| checker owner | CP2 只允许 owner discovery，不允许实现修改 | 直接修改 checker 会绕过设计和测试边界 |
| 验证方式 | 后续仅允许本地静态 / fixture 验证 | runtime / NAS / publish 均不相关且不应授权 |

## CP2 审批结果

| 字段 | 内容 |
|---|---|
| 审批状态 | approved |
| 审批时间 | 2026-06-22T18:54:16+08:00 |
| 授权范围 | 设计、owner discovery、检查目标定义和 CP5 implementation gate 草案 |
| 仍未授权 | 源码修改、tests 修改、checker implementation change、目录 / symlink 变更、runtime、NAS、凭据、交易写或 publish |

## Owner Discovery

| 对象 | 候选路径 | 推荐职责 | 说明 |
|---|---|---|---|
| primary checker | `scripts/check_human_gate_decision_brief.py` | 在 CP2/CP3/CP5/CP8 人工门禁结构检查中加入 path alias 引用规则 | 当前脚本已检查 Decision Brief、待决策表、不授权项和回复格式，最接近 path alias gate enforcement 的主 owner。 |
| support checker | `scripts/check_cr_tracking_consistency.py` | 只做 CR / STATE / follow-up 状态一致性旁路验证 | 不建议把 path alias 文案规则放进该脚本作为主逻辑。 |
| tests | `tests/` 下新增或扩展 checker fixture 测试 | 证明推荐路径通过、唯一 legacy alias 失败或提示 | CP5 若批准，再实施。 |

## CP5 推荐实现范围

| 范围项 | 推荐方案 | 非目标 |
|---|---|---|
| enforcement 对象 | 仅检查新增或当前人工门禁中的证据路径表达；优先针对 `process/checkpoints/CP*.md` | 不批量扫描并重写历史 `STATE.md` 或旧 CR 正文。 |
| 通过条件 | `process/docs/design/*` 或 `process/...` 证据入口存在时通过 | 不要求所有历史 `docs/design/*` 文本消失。 |
| 失败 / 提示条件 | 新门禁只把 `docs/design/*` 作为唯一证据路径且缺少 `process/...` 对应入口时失败或提示 | 不把公开 docs 路径、release docs 或 README 文案误判为错误。 |
| 验证 | 本地静态 fixture / unit test | 不启动 runtime、不访问 NAS、不读取凭据。 |

## 实现结果

| 字段 | 内容 |
|---|---|
| 实现文件 | `scripts/check_human_gate_decision_brief.py` |
| 测试文件 | `tests/test_cr118_human_gate_path_alias_checker.py` |
| 行为变化 | checker 会提取人工门禁中的 `docs/design/*` legacy alias；若缺少对应 `process/docs/design/*` 入口，则报错。 |
| 通过场景 | 同一证据同时提供 `process/docs/design/<file>` 与 `docs/design/<file>` legacy alias。 |
| 失败场景 | checkpoint 只提供 `docs/design/<file>` 作为设计证据入口。 |
| 验证结果 | `tests/test_cr118_human_gate_path_alias_checker.py` 通过；当前 CP5 checkpoint checker 自检通过。 |

## CP8 关闭结论

| 字段 | 内容 |
|---|---|
| 审批状态 | approved |
| 审批时间 | 2026-06-22T19:08:32+08:00 |
| 关闭状态 | CR118 closed-current-delivery / READY |
| 接受风险 | 历史 `docs/design/*` alias 不批量重写。 |
| 后续事项 | 当前不新增 follow-up；未来若发现误报 / 漏报，从实际反馈另起 CR。 |
