---
checkpoint_id: "CP5-CR071-COPY-FIRST-SHADOW-ROOT-READINESS"
checkpoint_name: "CR071 Copy-first Shadow Root Execution Readiness"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-06-16T07:16:44+08:00"
reviewed_by: "user"
reviewed_at: "2026-06-16T07:33:48+08:00"
auto_check_result: "process/checks/CP5-CR071-COPY-FIRST-SHADOW-ROOT-READINESS.md"
context_capsule: "process/context/CP5-CR071-EXECUTION-CONTEXT.yaml"
auto_final_authorization: false
target:
  phase: "story-planning"
  story_id: ""
  artifacts:
    - "process/changes/CR-071-COPY-FIRST-SHADOW-ROOT-MIGRATION-2026-06-16.md"
    - "docs/release/COPY-FIRST-SHADOW-ROOT-MANIFEST-CR071.yaml"
---

# CP5 CR071 Copy-first Shadow Root Execution Readiness 人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP5-CR071-COPY-FIRST-SHADOW-ROOT-READINESS.md` | PASS | 0 | 命令、排除项、preflight、验证与失败路径已冻结；复制尚未执行。 |

## Decision Brief

推荐决策：批准 CR071 CP5 execution readiness。若 CP2、CP3、CP5 均获得 `approved`，Host Orchestrator 可执行本机 copy-first：创建 `/home/hyde/workspace/quant-lab` 并用 `rsync -a` 复制当前 root，排除 `reports/`、`data/`、`.venv/`、`node_modules/` 和 `.env`。

备选方案：保持 CP5 pending；只执行 dry-run；改用 tracked-only 复制；改为不复制 `.git`。推荐方案优先，因为它最贴近用户的 copy-first shadow root 指令，并保留旧 root。

影响维度：本机文件系统新增目录、复制耗时、目标根缺少数据/报告/虚拟环境、Git local config 被复制但不远端写、后续 CP6/CP7/CP8 需要记录结果。

风险与回退：若执行前 target 非空、rsync 不可用或用户修改排除项，必须阻断并重提 CP5；复制中断时不自动删除 target，先记录 partial-copy 并等待用户决策。

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP5-CR071-EXECUTION-CONTEXT.yaml` |
| capsule 状态 | ready |
| read_profile | compact |
| 默认读取策略 | 先读 capsule；仅在缺失、冲突、字段不足、人工审计或深度评审时读取完整文档 |
| 全文档读取扩展 | 0 次 |
| 缺失 / waived 理由 | Story LLD N/A：single-operation local migration gate。 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `process/STATE.md.human_gate_decisions.pending_human_decisions[]` | scanned | 0 | 0 | 当前队列尚未登记 CR071；本文件创建后回写。 |
| 委托 Agent 交还摘要 | `process/handoffs/*RETURN-SUMMARY.md` | n/a | 0 | 0 | 本轮未调度子 agent；执行 readiness 由 host 直接冻结。 |
| 自动预检结果 | `process/checks/CP5-CR071-COPY-FIRST-SHADOW-ROOT-READINESS.md` | scanned | 6 | 6 | 命令、排除、preflight、验证、失败路径和不授权纳入 DQ。 |
| discussion log / checkpoint | `process/discussions/*` / `process/checks/*DISCUSSION-CHECKPOINT.json` | n/a | 0 | 0 | 无阻断 LLD clarification。 |
| 下游正式产物 | CR071 formal CR / manifest / plan | scanned | 6 | 6 | execution readiness 完整纳入 DQ。 |
| 用户显式选择题 | 当前对话 | scanned | 5 | 5 | 用户明确允许 CP2/CP3/CP5 后执行本机 copy-first，禁止外部动作。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP5-CR071-01 | implementation | CP5 approved 后是否允许执行本机 copy-first？ | 允许，但必须 CP2/CP3/CP5 全部 approved 且 preflight PASS。 | 只生成计划；只 dry-run。 | 推荐方案达成用户目标；只计划不能迁移；dry-run 可作为额外保守路径。 | 新增本机 target 目录。 | target 非空或 rsync 缺失时阻断。 |
| DQ-CP5-CR071-02 | implementation | 是否冻结精确命令？ | 使用 `mkdir -p` + `rsync -a`，排除 5 个路径。 | 改用 `cp -a`；改用 tracked-only archive。 | 推荐方案可控且保留元数据；cp 排除弱；tracked-only 可能遗漏未跟踪工作文件。 | 影响复制完整性。 | 命令变更需重提 CP5。 |
| DQ-CP5-CR071-03 | security | 执行前 preflight 如何处理？ | 重新检查 target 不存在或为空、rsync 可用、source 存在。 | 忽略 target 状态；覆盖。 | 推荐方案防覆盖；覆盖有破坏风险。 | 避免误写已有目录。 | preflight FAIL 时停止。 |
| DQ-CP5-CR071-04 | implementation | 执行后验证范围是什么？ | 验证 target exists、old root exists、排除项 absent、target Git status readable。 | 只看 rsync exit code；跑完整 pytest。 | 推荐方案覆盖迁移关键条件；只看 exit code 不足；完整 pytest 超出复制验证范围。 | 验证 target 结构与排除项。 | 验证失败进入 CP7 finding。 |
| DQ-CP5-CR071-05 | risk_acceptance | 复制失败或 partial target 是否自动清理？ | 不自动清理；记录 partial-copy 并等待用户授权。 | 自动删除 target；自动覆盖重试。 | 推荐方案避免二次破坏；自动清理需 destructive 授权。 | 可能留下 partial 目录。 | 用户授权后再清理。 |
| DQ-CP5-CR071-06 | runtime_authorization | CP5 approve 是否授权外部动作或最终关闭？ | 不授权外部动作，也不自动 CP8 关闭；复制后继续 CP6/CP7/CP8。 | 同步 remote Git / NAS / data lake / runtime；复制后直接关闭。 | 推荐方案保持门禁纪律；备选越权或缺验证。 | 复制后仍需记录执行和验证。 | 后续用户单独授权时另起 CR。 |

### CP5 追加 Decision Brief 字段

| 字段 | 内容 |
|---|---|
| 设计证据类型分布 | Story LLD N/A；execution readiness checkpoint 1 个。 |
| LLD clarification queue 收敛状态 | N/A，无实现灰区队列。 |
| 未回答阻断项为 0 的证据 | CP5 自动预检 PASS；target 当前 absent；rsync 可用。 |
| 文件 owner | host-orchestrator 仅创建 target root 和本机复制；不改旧 root。 |
| merge order | N/A，本轮不做 Git merge / push / commit。 |

### 不授权项

如果你回复 approve，表示接受上述 6 项推荐方案；不表示授权以下操作：

- 不授权复制 `reports/`、`data/`、`.venv/`、`node_modules/` 或 `.env`。
- 不授权删除、重命名或移动旧 root；不授权自动删除 partial target。
- 不授权读取、打印或迁移任何凭据或 `.env` 内容。
- 不授权 remote Git write、branch/default branch governance、history rewrite、force push、tag、remote deletion 或 set-url。
- 不授权 NAS/data lake/provider/catalog/runtime/QMT/MiniQMT/CR046 recovery。
- 不授权复制后自动切换 shell、IDE、systemd、cron、数据湖 root 或日常工作根。

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| CP5 context ready | 待审查 | `process/context/CP5-CR071-EXECUTION-CONTEXT.yaml` |  |
| 自动预检 PASS | 待审查 | `process/checks/CP5-CR071-COPY-FIRST-SHADOW-ROOT-READINESS.md` |  |
| manifest 可读 | 待审查 | `docs/release/COPY-FIRST-SHADOW-ROOT-MANIFEST-CR071.yaml` |  |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | 是否允许 CP2/CP3/CP5 后执行 copy-first | 待审查 | DQ-CP5-CR071-01 |  |
| 2 | 命令是否冻结 | 待审查 | DQ-CP5-CR071-02 |  |
| 3 | preflight 是否严格 | 待审查 | DQ-CP5-CR071-03 |  |
| 4 | 执行后验证是否充分 | 待审查 | DQ-CP5-CR071-04 |  |
| 5 | partial target 是否禁止自动清理 | 待审查 | DQ-CP5-CR071-05 |  |
| 6 | 是否继续禁止外部动作和自动关闭 | 待审查 | DQ-CP5-CR071-06 |  |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| 用户明确 approve / 修改 / reject | 待审查 | 本文件人工审查结果 |  |
| CP5 approve 前不执行复制 | 待审查 | target absent / no execution evidence |  |
| CP5 approve 后仍需 preflight | 待审查 | manifest |  |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| CP5 auto precheck | `process/checks/CP5-CR071-COPY-FIRST-SHADOW-ROOT-READINESS.md` | 待审查 |  |
| CP5 context capsule | `process/context/CP5-CR071-EXECUTION-CONTEXT.yaml` | 待审查 |  |
| CP5 launch message | `process/checks/CP5-CR071-HUMAN-GATE-LAUNCH-MESSAGE.md` | 待审查 |  |

## 人工审查结果

- 结论：approved
- 审查人：user
- 审查时间：2026-06-16T07:33:48+08:00
- 修改意见：N/A
- 风险接受项：用户回复“同意”，按 `approve` 处理，接受 DQ-CP5-CR071-01..06 推荐方案；该批准授权在 CP2/CP3/CP5 均 approved 且 preflight PASS 后执行本机 copy-first `mkdir -p` + `rsync -a`，排除 `reports/`、`data/`、`.venv/`、`node_modules/`、`.env`。该批准不授权删除/重命名/移动旧 root，不授权自动清理 partial target，不授权读取凭据、NAS、远端 Git 写动作、data lake、QMT/MiniQMT runtime、环境切换或 CR046 recovery。
