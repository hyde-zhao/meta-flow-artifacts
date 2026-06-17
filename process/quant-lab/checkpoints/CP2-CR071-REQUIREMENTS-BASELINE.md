---
checkpoint_id: "CP2-CR071-REQUIREMENTS-BASELINE"
checkpoint_name: "CR071 Requirements Baseline"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-06-16T07:16:44+08:00"
reviewed_by: "user"
reviewed_at: "2026-06-16T07:33:48+08:00"
auto_check_result: "process/checks/CP2-CR071-REQUIREMENTS-BASELINE.md"
context_capsule: "process/context/CP2-CR071-REQUIREMENT-CONTEXT.yaml"
auto_final_authorization: false
target:
  phase: "requirement-clarification"
  story_id: ""
  artifacts:
    - "process/changes/CR-071-COPY-FIRST-SHADOW-ROOT-MIGRATION-2026-06-16.md"
    - "docs/release/COPY-FIRST-SHADOW-ROOT-MIGRATION-PLAN-CR071.md"
    - "docs/release/COPY-FIRST-SHADOW-ROOT-MANIFEST-CR071.yaml"
---

# CP2 CR071 Requirements Baseline 人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP2-CR071-REQUIREMENTS-BASELINE.md` | PASS | 0 | 用户范围明确；本机 copy-first、排除项和不授权边界均已登记。 |

## Decision Brief

推荐决策：批准 CR071 CP2 范围基线，正式启动 MIG-A 为本机 copy-first shadow root 迁移门禁；CP2 只确认需求 / 范围，不执行复制。

备选方案：保持 CR071 为 candidate 等待；缩小为只写计划不执行；取消 CR071。推荐方案优先，因为用户已给出明确目标和排除项，且 copy-first 可在旧 root 保留前提下降低风险。

影响维度：本机目录结构、未来工作根、Git 本地工作树、数据/报告排除、凭据边界、CR tracking 和后续 CP5 执行授权。

风险与回退：若 CP2 reject，CR071 不进入 CP3/CP5，也不创建目标目录；若用户修改排除清单或 `.env` 策略，必须重写 CP2/CP3/CP5 并重新发起确认。

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP2-CR071-REQUIREMENT-CONTEXT.yaml` |
| capsule 状态 | ready |
| read_profile | compact |
| 默认读取策略 | 先读 capsule；仅在缺失、冲突、字段不足、人工审计或深度评审时读取完整文档 |
| 全文档读取扩展 | 0 次 |
| 缺失 / waived 理由 | Scenario discussion log N/A：用户已直接给出范围、排除项和禁止操作边界。 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `process/STATE.md.human_gate_decisions.pending_human_decisions[]` | scanned | 0 | 0 | 当前队列尚未登记 CR071；本文件创建后回写。 |
| 委托 Agent 交还摘要 | `process/handoffs/*RETURN-SUMMARY.md` | n/a | 0 | 0 | 本轮未调度子 agent；范围由用户直接给出。 |
| 自动预检结果 | `process/checks/CP2-CR071-REQUIREMENTS-BASELINE.md` | scanned | 5 | 5 | 范围、排除项、外部禁止和风险接受纳入 DQ。 |
| discussion log / checkpoint | `process/discussions/*` / `process/checks/*DISCUSSION-CHECKPOINT.json` | n/a | 0 | 0 | 用户请求已明确，无需额外 SGQ。 |
| 下游正式产物 | CR071 formal CR / migration plan / manifest | scanned | 5 | 5 | source/target、排除项、not-authorized 和执行条件纳入 DQ。 |
| 用户显式选择题 | 当前对话 | scanned | 5 | 5 | 用户明确：旧目录保留、不复制指定目录、.env 默认不复制、禁止 NAS/remote Git/runtime/lake。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP2-CR071-01 | scope | 是否正式启动 CR071 / MIG-A？ | 启动为正式 active gate，CR046 仍保持 paused；本轮只进入 CP2/CP3/CP5 审查。 | 保持 candidate；取消 CR071。 | 推荐方案可审计推进；candidate 会延迟迁移；取消则无新 root。 | 影响 STATE / CR-INDEX；不影响 CR046 runtime。 | reject 时恢复 candidate / 不执行。 |
| DQ-CP2-CR071-02 | scope | 是否接受 copy-first shadow root 范围？ | 只从 `/home/hyde/workspace/local_backtest` 复制到 sibling `/home/hyde/workspace/quant-lab`。 | 改为 rename/move；只写计划不复制。 | 推荐方案保留旧 root 且可验证；rename/move 风险高；只写计划不达成本轮目标。 | 影响本机目录布局。 | target 非空或用户修改 root 时重提。 |
| DQ-CP2-CR071-03 | security | 是否确认排除清单？ | 不复制 `reports/`、`data/`、`.venv/`、`node_modules/` 和 `.env`。 | 额外排除 `runs/` / cache；允许复制部分 data/report。 | 推荐方案严格匹配用户请求；扩展排除需重新评估；复制 data/report 会扩大风险和耗时。 | 目标 root 初始不含数据/报告/虚拟环境。 | 用户修改清单时重写 manifest。 |
| DQ-CP2-CR071-04 | runtime_authorization | CP2 approve 是否授权 NAS、远端 Git、QMT runtime 或数据湖切换？ | 不授权；这些仍需独立 CR / runtime_authorization。 | 合并 NAS；合并 remote Git；合并 data lake；恢复 CR046。 | 推荐方案分层清晰；备选会引入外部副作用。 | 防止越权访问、写远端或触发 runtime。 | 后续用户明确要求时另起 CR。 |
| DQ-CP2-CR071-05 | risk_acceptance | 是否接受 target 初始不是完整 artifact/data mirror？ | 接受；target 不含 reports/data/.venv/node_modules/.env。 | 要求完整镜像；只复制 Git tracked 文件。 | 推荐方案符合用户排除项；完整镜像体量和敏感风险更高；tracked-only 会遗漏未跟踪过程文件。 | 后续运行可能需要重新生成 venv、data 和 reports。 | 后续需 data/report 时另起迁移或生成任务。 |

### CP2 追加 Decision Brief 字段

| 字段 | 内容 |
|---|---|
| 用户真实意图 | 将当前本机工作区复制到 canonical `quant-lab` shadow root，同时保留旧目录和安全边界。 |
| 场景覆盖 | target absent happy path、excluded paths absent、target non-empty blocked。 |
| Deferred Ideas | 日常工作根切换、data/report/NAS/data-lake 迁移、remote Git 变更、CR046 runtime 恢复均后置。 |
| 回退方式 | CP2 不通过则不创建目标目录；复制后回退默认继续使用旧 root，不自动删除 target。 |

### 不授权项

如果你回复 approve，表示接受上述 5 项推荐方案；不表示授权以下操作：

- 不授权删除、重命名或移动 `/home/hyde/workspace/local_backtest`。
- 不授权复制 `reports/`、`data/`、`.venv/`、`node_modules/` 或 `.env`。
- 不授权读取、打印或迁移 `.env`、token、密码、cookie、session、private key、账户或交易事实。
- 不授权 NAS、远端 Git 写动作、branch/default branch governance、history rewrite、force push、tag、remote deletion 或 set-url。
- 不授权 `MARKET_DATA_LAKE_ROOT` 切换、provider fetch、lake write、catalog publish、QMT / MiniQMT runtime 或 CR046 recovery。

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| CR071 formal CR 已创建 | 待审查 | `process/changes/CR-071-COPY-FIRST-SHADOW-ROOT-MIGRATION-2026-06-16.md` |  |
| CP2 context ready | 待审查 | `process/context/CP2-CR071-REQUIREMENT-CONTEXT.yaml` |  |
| 自动预检 PASS | 待审查 | `process/checks/CP2-CR071-REQUIREMENTS-BASELINE.md` |  |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | CR071 是否正式启动 | 待审查 | DQ-CP2-CR071-01 |  |
| 2 | copy-first shadow root 范围是否正确 | 待审查 | DQ-CP2-CR071-02 |  |
| 3 | 排除清单是否完整 | 待审查 | DQ-CP2-CR071-03 |  |
| 4 | 外部和 runtime 操作是否继续禁止 | 待审查 | DQ-CP2-CR071-04 |  |
| 5 | target 非完整 artifact/data mirror 风险是否接受 | 待审查 | DQ-CP2-CR071-05 |  |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| 用户明确 approve / 修改 / reject | 待审查 | 本文件人工审查结果 |  |
| CP2 approve 不执行复制 | 待审查 | 不授权项 / CR071 |  |
| 修改项已回到 DQ | 待审查 | 待用户回复 |  |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| CP2 auto precheck | `process/checks/CP2-CR071-REQUIREMENTS-BASELINE.md` | 待审查 |  |
| CP2 context capsule | `process/context/CP2-CR071-REQUIREMENT-CONTEXT.yaml` | 待审查 |  |
| CP2 launch message | `process/checks/CP2-CR071-HUMAN-GATE-LAUNCH-MESSAGE.md` | 待审查 |  |

## 人工审查结果

- 结论：approved
- 审查人：user
- 审查时间：2026-06-16T07:33:48+08:00
- 修改意见：N/A
- 风险接受项：用户回复“同意”，按 `approve` 处理，接受 DQ-CP2-CR071-01..05 推荐方案；该批准只确认 CR071 / MIG-A 范围基线，不授权删除/重命名/移动旧 root，不授权复制 `reports/`、`data/`、`.venv/`、`node_modules/` 或 `.env`，不授权读取凭据、NAS、远端 Git 写动作、data lake、QMT/MiniQMT runtime 或 CR046 recovery。
