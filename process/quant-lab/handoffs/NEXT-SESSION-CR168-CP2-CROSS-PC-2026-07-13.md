---
handoff_id: "CR168-CP2-CROSS-PC-2026-07-13"
workflow_id: "CR-168"
change_id: "CR-168"
from_role: "host-orchestrator"
to_role: "host-orchestrator"
stage: "requirement-clarification"
checkpoint: "CP2"
status: "awaiting-user"
created_at: "2026-07-13T19:31:26+08:00"
created_by: "host-orchestrator"
cross_pc_recovery: true
dispatch:
  required: false
  semantic: "stage-dispatch"
  mode: "handoff-only"
  platform: "codex"
  agent_role: "meta-se"
  canonical_role: "meta-se"
  codex_agent_name: ""
  reasoning_profile: "default"
  dispatch_trigger: "CP2-approved"
  agent_path: ""
  tool_name: ""
  agent_id: ""
  agent_name: ""
  thread_id: ""
  spawned_at: ""
  resumed_at: ""
  completed_at: ""
  evidence: "本文件只用于跨电脑 Host 会话恢复；不是 meta-se 已执行或已调度的证据。"
  fallback_reason: "用户明确要求本工作流不拉起子 Agent；CP2 通过后仍由 Host Orchestrator inline 完成后续功能职责，除非用户另行授权。"
  approved_by: "user"
  approved_at: "2026-07-13"
question_permission:
  can_ask_user: true
  mode: "relay-via-host-orchestrator"
  structured_choice_allowed: false
  allowed_question_scope: "CP2 Decision Brief 的五个待决策项，以及在后续已批准阶段出现的范围、架构、风险澄清。"
  forbidden_question_scope: "不得以普通澄清绕过 CP2/CP3/CP5/CP8；不得扩大真实数据、凭据、runtime、publish、交易或远端写入授权。"
  broker_agent: "host-orchestrator"
context_policy:
  capsule_first: true
  capsule_path: "process/context/CP2-CR168-REQUIREMENT-CONTEXT.yaml"
  context_ref: "process/context/CP2-CR168-REQUIREMENT-CONTEXT.yaml"
  story_packet_ref: ""
  read_profile: "compact"
  max_source_files: 12
  full_doc_read_policy: "only-on-missing-conflict-audit-or-deep-review"
  full_doc_read_reason: ""
  allowed_reads:
    - "process/state/STATE.current.json"
    - "process/current/CURRENT.json"
    - "process/changes/summaries/CR-168.summary.json"
    - "process/checks/CP1-CR168-USE-CASE-COMPLETENESS.result.json"
    - "process/checks/CR168-PRODUCT-BASELINE-EVIDENCE.json"
    - "process/checks/CP2-CR168-DISCUSSION-CHECKPOINT.json"
    - "process/checkpoints/CP2-CR168-REQUIREMENTS-BASELINE.md"
  must_read:
    - "process/state/STATE.current.json"
    - "process/current/CURRENT.json"
    - "process/context/CP2-CR168-REQUIREMENT-CONTEXT.yaml"
    - "process/checkpoints/CP2-CR168-REQUIREMENTS-BASELINE.md"
  read_if_needed:
    - "docs/product/REQUIREMENTS.md — 仅在审计 9 项 requirement、15 项 QAC 或五项 DQ 时读取"
    - "docs/product/SCENARIOS.yaml — 仅在审计 17 个场景、10 类 C3 input fail-closed 或 Gate 4 projection guard 时读取"
    - "process/changes/CR-168.md — 仅在 formal CR 字段冲突或深度人工审计时读取"
  do_not_read_by_default:
    - "process/STATE.md"
    - "process/archive/**"
    - "process/discussions/**"
    - "process/stories/**"
    - "docs/design/HLD.md"
    - "完整 CR-166 Story/LLD/测试报告"
    - "完整历史会话 transcript"
---

# CR-168 CP2 跨电脑恢复交接

## 1. 当前停止点（不可跳过）

CR-168 已完成 CP0、产品基线增量刷新、CP1 自动检查和 CP2 自动预检；当前为：

```text
status=awaiting-user
current_phase=requirement-clarification
pending_gate=CP2
stop_reason=required_human_gate
```

CP2 尚未获批。新电脑上的会话只能发起/等待 CP2 人工决定；不得创建 HLD、进入 CP3、拆 Story、写 LLD、实现、运行测试或宣称 C3 已交付。

唯一有效的 CP2 审查入口：

- checklist / Decision Brief：`process/checkpoints/CP2-CR168-REQUIREMENTS-BASELINE.md`
- 自动结果：`process/checks/CP2-CR168-REQUIREMENTS-BASELINE.result.json`
- 默认上下文胶囊：`process/context/CP2-CR168-REQUIREMENT-CONTEXT.yaml`
- 当前机器状态：`process/state/STATE.current.json`

## 2. 跨电脑检出与路由恢复

两仓库必须作为相邻目录检出，保持 `quant-lab/process -> ../meta-flow-artifacts/process/quant-lab` 的相对软链接有效：

```text
<workspace>/quant-lab
<workspace>/meta-flow-artifacts
```

在新电脑执行以下只读/快进恢复步骤：

```bash
cd <workspace>/meta-flow-artifacts
git checkout main
git pull --ff-only origin main

cd <workspace>/quant-lab
git fetch origin --prune
git checkout --track origin/work/cr168-economic-cost-impact-evidence
git pull --ff-only
uv run --python 3.11 meta-flow workspace check --project-root .
```

若 `workspace check` 报告软链接、`project_name`、route metadata 或 `STATE.md` 不健康，立即停止；不得手工新建 `process/`、不得重建 `STATE.md`。

## 3. 恢复后的最小读取序列

按如下顺序读取，默认不扩展全文：

1. `process/state/STATE.current.json`
2. `process/current/CURRENT.json`
3. `process/context/CP2-CR168-REQUIREMENT-CONTEXT.yaml`
4. `process/changes/summaries/CR-168.summary.json`
5. `process/checks/CP1-CR168-USE-CASE-COMPLETENESS.result.json`
6. `process/checks/CR168-PRODUCT-BASELINE-EVIDENCE.json`
7. `process/checkpoints/CP2-CR168-REQUIREMENTS-BASELINE.md`

只有缺字段、发现冲突、人工审计或深度评审时，才读取 `CR-168.md` 或产品全文；扩展读取必须写入 read-expansion ledger。

## 4. 已确认事实与范围

- 正式 CR：`CR-168 — Economic Cost / Slippage / Impact Computable Evidence Producer Foundation`。
- 基线：quant-lab 从 `origin/work/cr166-walk-forward-oos-evidence` 的 `e8507cbe89fef57683f959a4c20d18d8fecb4426` 派生；旧默认分支 `master` 未作为启动基线。
- C3 只做 fixture/static-only 的 economic cost、slippage 和透明 impact approximation；只接受显式 synthetic/static 参数。
- Gate 4 是 C3+C4 联合门禁：CR-168 只投影 `impact_model_family`、`impact_model_ref`、`cost_underestimation_status`、`no_real_tca_claim`；C4 `reserved/not-built/typed_unavailable` 必须映射为三个 refs absent-no-na-reason，字段级 `*_na_reason` / `*_n_a_reason` 或通用 `na_reason` / `n_a_reason` 逃逸必须由 projection `BLOCKED/REJECTED`；canonical Gate 4 和 aggregate orchestration 不修改，capacity/aggregate PASS=`0`。
- C4 calculators 保留给 `FU-CR161-005`；C1–C4 aggregate integration、最终 admission package 与 CR-155 综合 regression/promotion 保留给 `FU-CR161-007`。
- CR-155 lifecycle 已关闭，但 admission package 仍是 `BLOCKED`、`paper_candidate=false`；不得 promotion、unblock 或重解释。

## 5. CP2 需要人工确认的五项决定

| ID | 推荐决定 |
|---|---|
| `DQ-CP2-CR168-METHOD` | 纳入 fee/tax/spread/slippage/impact 分解；impact 仅使用静态参数，并输出 `cost_underestimation_status`、limitations、`no_real_tca_claim=true`。 |
| `DQ-CP2-CR168-C3-C4` | 冻结最小共享 header；C4-exclusive capacity/ADV/liquidity/alpha-decay 字段仅 reserved，C4 calculator=`0`。 |
| `DQ-CP2-CR168-GATE4` | 实现 1 条 C3-to-Gate-4 projection；C4 unavailable 映射为三个 refs absent-no-na-reason；任何字段级/通用 reason 逃逸由 projection 阻断；不修改 canonical Gate 4/aggregate；保留 B01 并新增 B02。 |
| `DQ-CP2-CR168-FIXTURE` | 使用两个 fixture 族：daily multifactor synthetic；daily multifactor + ML multi-strategy-type compatibility。 |
| `DQ-CP2-CR168-CLAIM` | 保持 Stage2=true、Stage3=false；真实 TCA/calibration/data/runtime=false；C4=0、event=false、CR155 promotion=false。 |

## 6. 成功标准与不授权边界

CP2 待确认的量化目标：typed component/schema=`1/1`、字段族=`9/9`、fixture=`2/2`、C3 input fail-closed=`10/10`、场景=`17`（P0=`16`、P1=`1`）、重复运行=`10 -> 1 hash`、Gate 4 projection=`1`、B01/B02=`2/2`、reason 逃逸与 C4 缺失导致的 capacity/aggregate PASS=`0`、canonical Gate 4/aggregate 修改=`0`、parallel gate/envelope/registry=`0`、C4/event calculators=`0`、真实数据/TCA/runtime/trading=`0`、CR155 promotion=`0`、错误质量路径 `process/docs/quality/`=`0`。

CP3 前向义务已记录但尚未启动：availability→Gate 4 flat-payload 映射、reason-key denylist、capability-registry-missing 处理、required evidence kind 映射、impact-specific `n/a-with-reason`、component/envelope hash domain。`R-CR168-VERIFIER-INDEPENDENCE` 为非阻断风险；若 CP7 仍 inline，CP8 必须显式暴露。

即使 CP2 获批，也不授权：真实数据、凭据、provider/NAS/lake、真实 TCA/calibration/capacity sizing、C4/event/FU-007 aggregate integration、runtime/broker/QMT/simulation/paper/live/trading、catalog/store/registry 写入、publish/deploy/tag/release/Git remote write、Stage 3 启动或 CR-155 promotion。

## 7. 继续规则

1. 用户回复 `approve` 后，先回填 CP2 checkpoint 和 gate ledger，并运行 CP2 approval/state-transition/result consistency 检查。
2. 只在 CP2 approved 后，创建 capsule-first 的 CP3 design context；CP3 前不得把产品基线当成已冻结 HLD。
3. CP3、CP5 仍是人工门禁。实现授权最早在 CP5 approved 后获得；CP2 或 CP3 批准绝不等于实现授权。
4. 用户仍偏好不拉起子 Agent；任何 inline 执行必须记录为用户批准的 inline fallback，不能伪造 subagent dispatch。

## 8. 已验证检查与已知非阻塞债务

已通过：process route health、CP0/CP1/CP2 result consistency、CP0/CP1 state transition、CR168 conflict precheck、CR tracking、CP2 human-gate 格式、checkpoint/gate ledger、JSON/YAML parse、`git diff --check`。

已知但不扩大 CR-168 范围的历史债务：全局 read-expansion ledger 含早期不符合现行字段要求的记录；全局 strict CR lifecycle check 含早期 ledger/index 状态词和 summary 缺失。CR-168 专属 route/lifecycle diagnostics 已清零。

## 9. 新会话的精确起始动作

新会话应使用同目录的 `CR168-CP2-RESUME-PROMPT.md`。如用户已经审阅修订后的 Decision Brief 并接受五项推荐，直接使用：

```text
approve
```

该句只批准 CP2 范围与进入 CP3；不批准实现、真实数据、runtime、交易、发布或 Git 远端写入。
