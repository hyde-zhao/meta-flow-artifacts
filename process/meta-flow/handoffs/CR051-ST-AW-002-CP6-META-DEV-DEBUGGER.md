---
handoff_id: "H-CR051-ST-AW-002-CP6-META-DEV-DEBUGGER"
from_role: "host-orchestrator"
to_role: "meta-dev"
canonical_role: "meta-dev"
workflow_id: "meta-flow-self-dev"
cr_id: "CR-051"
story_id: "ST-AW-002"
wave_id: "W2"
stage: "story-execution"
status: "completed"
completed_at: "2026-07-18T12:50:00Z"
created_at: "2026-07-18T12:38:00Z"
context_ref: "process/context/stories/STORY-ST-AW-002.CP6.work-packet.json"
story_packet_ref: "process/context/stories/STORY-ST-AW-002.CP6.work-packet.json"
dispatch:
  required: true
  mode: "subagent"
  platform: "codex"
  canonical_role: "meta-dev"
  codex_agent_name: "meta-dev-debugger"
  reasoning_profile: "debugger"
  dispatch_trigger: "runtime-high-risk-non-atomic-state-machine"
  tool_name: "spawn_agent"
  agent_id: "/root/cr051_staw002_dev"
  thread_id: "/root/cr051_staw002_dev"
  spawned_at: "2026-07-18T12:39:00Z"
  completed_at: "2026-07-18T12:50:00Z"
question_permission:
  can_ask_user: false
  mode: "relay-via-host-orchestrator"
  broker_agent: "host-orchestrator"
context_policy:
  capsule_first: true
  story_packet_ref: "process/context/stories/STORY-ST-AW-002.CP6.work-packet.json"
  read_profile: "minimal"
  must_read:
    - "delivery/agents/meta-dev.md"
    - "process/context/stories/STORY-ST-AW-002.CP6.work-packet.json"
    - "process/returns/ST-AW-001.CP7-R2.return.json"
    - "process/docs/features/cr051-worktree/DESIGN.md"
    - "process/docs/features/cr051-worktree/TEST-PLAN.md"
  read_if_needed:
    - "process/stories/STORY-ST-AW-002-recoverable-project-worktree-LLD.md（实现前必须 deep_review 并写 read-log）"
    - "meta_flow/workspace/git_sync.py（进入 shared 单写窗口时）"
    - "tests/test_workspace_git_sync.py"
    - "tests/test_git_branch_lifecycle.py"
  do_not_read_by_default:
    - "process/STATE.md"
    - "process/DEVELOPMENT-PLAN.yaml"
    - "其他 Story / LLD / IMPLEMENTATION"
---

# ST-AW-002 CP6 实现交接

## 目标

实现 existing-control + sibling project worktree 的 registration/health/create/check/list/safe-remove、exact-OID create-only integration bootstrap、capacity precheck、durable intent 与可恢复 switch。

## 强制证明义务

- `CAP-01..11`：false-safe=0、underestimate=0；权限/枚举/误差未知 100% 在 mutation 前 BLOCKED。
- `DUR-01..14`：覆盖 ENOSPC、EACCES、file-fsync、replace、dir-fsync、torn/corrupt、process-kill、cross-device 等；提前 Git mutation=0；resume 幂等。
- `WT-01..14` 与适用 TC-AW 全覆盖；危险 argv（reset-hard/clean/stash/force/branch-delete）=0。

## 边界

- 所有 Git/worktree/remote 操作仅允许临时目录、本地 fixture 和 local bare remote；不得触碰真实 artifact repo、真实 refs/worktrees/remotes、`.git` 或 sibling 项目。
- `meta_flow/cli.py` 由 ST-AW-004 merge owner，当前不得修改。
- shared `git_sync.py` 进入单写窗口后允许最小扩展；若原语无法满足 ST-AW-003 contract，记录 design delta 候选，但不得修改 ST-AW-003 文件。
- CAP/DUR 任一失败时，自动 switch 必须 fail-closed 或 manual-only，不能以风险接受通过。

## 交付

实现、测试、IMPLEMENTATION、`ST-AW-002.CP6.return.json`、`ST-AW-002.CP6.index.json`，以及所有可复跑命令结果；不自行批准 CP6。
