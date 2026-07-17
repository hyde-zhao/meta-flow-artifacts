---
checkpoint_id: "CP2"
checkpoint_name: "需求 / 场景 / 范围基线门"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-07-15T08:55:00+00:00"
reviewed_by: "user"
reviewed_at: "2026-07-16T09:10:00+08:00"
approval_source: "用户消息：通过cp2"
auto_check_result: "process/checks/CP2-CR030-REV02.result.json"
target:
  phase: "requirement-clarification"
  story_id: ""
  artifacts:
    - "docs/product/USE-CASES.md"
    - "docs/product/REQUIREMENTS.md"
    - "docs/product/SCENARIOS.yaml"
    - "docs/product/TEST-MATRIX.md"
    - "docs/product/STORY-MAP.md"
    - "docs/product/MVP-SCOPE.md"
---

# CP2 人工审查 — CR-030 需求 / 场景 / 范围基线

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP1-CR030-REV02.result.json` | PASS | 0 | 场景已扩展为用户中心的 ITR 摄取、保存、分析总结和改进候选旅程。 |
| `process/checks/CP2-CR030-REV02.result.json` | PASS | 0 | 需求、场景、MVP、追溯和安全边界已按 v1.2 一致化；等待人工确认。 |
| `process/checks/CP2-DISCUSSION-CHECKPOINT-CR030.json` | completed | 0 | 用户已批准七个用户旅程，以及 ITR 读取、保存、总结/改进候选边界。 |

## Decision Brief

### 审批者摘要

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 确认 ptm-tse 面向五类真实用户：在不扩大凭据、写入或其他系统访问的前提下，受控取得并保存指定 ITR 问题单，产出可审计的总结和改进候选。 |
| 推荐动作 | approve：采用 `USE-CASES.md` v1.2 七个用户旅程；仅允许固定 ITR URL 的 GET、无凭据/认证推断、无写入；原始快照引用和规范化记录只进非 Git 受限数据区，且总结/CA/PA 只作为人工待确认草案。 |
| approve 后会发生什么 | 先做一次受控 ITR schema 探测，并用真实字段刷新数据模型、蓝图、HLD、跨 Agent 契约与 CP3 材料；不会自动创建任务、执行改进或进入 Story/实现。 |
| approve 不授权什么 | 不授权凭据、认证头推断、其他外部系统访问、任何 ITR/生产写入、自动修复、自动下游任务或内部问题流程。 |
| 不确认会阻塞什么 | 阻塞 ITR 调用、数据持久化设计、HLD 重建、CP3、Story 拆解和 Agent/Skill 实施。 |

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP2-REQUIREMENT-CONTEXT.yaml` |
| capsule 状态 | ready（已确认） |
| read_profile | compact |
| 默认读取策略 | capsule-first；全文只在审计/冲突/深度评审时读取 |
| 全文档读取扩展 | 已按 human-audit 登记 CR、场景讨论和 REV-02 影响分析；受控 schema 探测不输出问题单内容 |

### 决策分层

| 分类 | 数量 | 处理方式 |
|---|---:|---|
| 必须用户决策 | 4 | 确认用户旅程、ITR GET、保存边界和总结/措施的人工确认边界。 |
| 高风险策略确认 | 2 | ITR HTTP 读取和潜在敏感数据保存；用户批准推荐。 |
| agent 默认处理 | 4 | 文档编号、场景展开、追溯和静态验证入口由 inline fallback 完成并留痕。 |
| 仅审计记录 | 3 | v1.0 批准失效、REV-01 历史基线和用户禁止拉子 agent 的授权边界。 |

### 待人工决策清单与结果

| 决策 ID | 类型 | 推荐方案 | 备选方案 | 结果 |
|---|---|---|---|---|
| CP2-DQ-CR030-REV01-01 | scope | 确认 UC-RA-01..07 的五类真实用户、目标、协作和成功结果为需求基线 | 指出需调整的画像/旅程；或暂缓确认 | **approved** |
| CP2-DQ-CR030-REV02-01 | runtime_authorization | 仅固定 ITR URL 的 GET、无凭据/认证推断、无写入；批准后一次受控 schema 探测 | 保持人工导入；或独立 runtime CR | **approved** |
| CP2-DQ-CR030-REV02-02 | security | 原始快照引用和规范化记录存非 Git 受限数据区；保留/脱敏由数据 Owner 确认 | 仅脱敏字段；或不保留原始响应 | **approved** |
| CP2-DQ-CR030-REV02-03 | scope | 自动生成总结和 CA/PA 候选，人工 reviewer 才能确认/分发/关闭 | 只输出总结；或自动创建下游任务 | **approved** |
| CP2-DQ-CR030-01 | scope | 仅恢复后现网问题；P1 必做/P2 可选 | 全严重度；或只 P1 | retained from v1.0 approval |
| CP2-DQ-CR030-02 | implementation | 已批准改进输入文件化交接，不直接写下游 | 自动建任务；或仅报告 | retained from v1.0 approval |
| CP2-DQ-CR030-03 | security | 无外部连接，只消费脱敏摘要/索引 | 只读连接；或受控写入 | retained from v1.0 approval |

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| REV-02 CP1 已通过 | PASS | `CP1-CR030-REV02.result.json` | 用户中心场景与 ITR 范围完整 |
| 产品基线已形成 | PASS | product 八份产物 v1.2 | 需求、场景、范围可追溯 |
| SGQ 已处理 | PASS | CP2 discussion log/checkpoint | 用户已确认七个用户旅程和三项 ITR 决策 |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | 场景以用户目标/旅程而非系统步骤命名 | approved | USE-CASES v1.2 | 用户已确认五类用户与七个旅程 |
| 2 | ITR 读取被限制为固定 GET、无凭据和无写入 | approved | REQ-011/014、SCN-RA-08 | 用户已批准 runtime 授权边界 |
| 3 | 问题单保存可追溯且不进入 Git/process | approved | REQ-015/016、MVP | 用户已批准数据保存边界 |
| 4 | 总结和改进措施保留人工确认 | approved | REQ-017/018、SCN-RA-10 | 用户已批准自动化边界 |
| 5 | 恢复后现网问题范围明确 | retained | USE-CASES / MVP | 不含内部问题 |
| 6 | 证据和人工确认边界明确 | retained | REQ-002/003/010 | 不以推测替代事实 |
| 7 | CA/PA 和下游输入可追溯 | retained | REQ-007/008、Story Map | 不直接写下游 |
| 8 | 安全与权限边界明确 | approved | REQ-011、MVP Out | ITR 单一受控读取例外已确认 |
| 9 | 延后项有重启条件 | retained | Backlog | 可追溯 |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| 需求/场景/范围已确认 | approved | 本文件与 CP2 REV02 result | 可进入 HLD 重建 |
| BLOCKING 未决项为 0 | approved | CP2 discussion checkpoint | 用户确认后为 0 |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| 使用场景 | `docs/product/USE-CASES.md` | approved | 7 个用户中心 UC |
| 需求 | `docs/product/REQUIREMENTS.md` | approved | 18 REQ，来源回链 v1.2 UC |
| 验证场景与矩阵 | `SCENARIOS.yaml` / `TEST-MATRIX.md` | approved | 10 SCN，含 ITR 摄取/失败保护/总结候选 |
| 规划产物 | Story Map / MVP / slices / backlog | approved | 5 Story + deferred，已按用户收益与数据边界校准 |

## 人工审查结果

- 原结论：**approved**（仅适用于 v1.0 流程中心场景，已失效）。
- 当前结论：**approved**。
- 本轮变更：已依据用户新增 ITR 问题单获取、保存、分析总结与改进措施要求形成 v1.2 七个用户中心旅程，并同步需求、验证场景、范围、切片和 Backlog；详见 `CR-030-EXTERNAL-ITR-INGESTION-AND-ANALYSIS-IMPACT-2026-07-15.md`。
- 用户决定：用户回复“批准CP2”，接受全部推荐方案；已完成一次受控 ITR GET schema 探测（HTTP 200、81 条记录），未输出问题单内容。
