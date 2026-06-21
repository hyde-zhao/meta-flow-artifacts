---
feature_id: "cr099-runner-real-readonly-smoke"
cr_id: "CR-099"
status: "verified-with-risk"
owner: "host-orchestrator"
created_at: "2026-06-19T16:04:52+08:00"
source_design: "docs/features/cr099-runner-real-readonly-smoke/DESIGN.md"
---

# CR099 Runner Real Readonly Smoke Tasks

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 0.1 | 2026-06-19 | host-orchestrator | 初版任务清单，按 CP5 后的设计、离线实现、逐 run 授权和验证收敛拆分。 |
| 0.2 | 2026-06-19 | host-orchestrator | 用户回复“同意”批准 CP5；CR099-T04 离线 contract / schema checker implementation 已完成。 |

## 任务清单

| TASK ID | 阶段 | 输入 | 输出 | 文件范围 | 验证入口 |
|---|---|---|---|---|---|
| CR099-T01 | CP5 | CP3 approved HLD | run contract schema | `process/stories/CR099-RUNNER-REAL-READONLY-SMOKE-LLD.md` | CP5 auto precheck |
| CR099-T02 | CP5 | Feature Design | redacted evidence schema | `docs/features/cr099-runner-real-readonly-smoke/DESIGN.md` | CP5 Decision Brief |
| CR099-T03 | CP5 | TEST-PLAN | blocked-path acceptance matrix | `docs/features/cr099-runner-real-readonly-smoke/TEST-PLAN.md` | CP5 human gate |
| CR099-T04 | CP6 | CP5 approved | contract / schema checker implementation | `tests/test_cr099_runner_real_readonly_smoke_contract.py` and optional checker script | offline pytest |
| CR099-T05 | Runtime authorization | user per-run authorization | single-run manifest | future run manifest / evidence path | manual authorization review |
| CR099-T06 | CP6 / CP7 | authorized run or user evidence | redacted evidence or blocked evidence | `.quant-lab/evidence/qmt/cr099/redacted/<run_id>/` or user-provided file | evidence checker |
| CR099-T07 | CP8 | CP6 / CP7 evidence | closure / follow-up routing | CP8 release context and checkpoint | CP8 readiness |

## Merge / Execution Order

1. CP5 approval must precede any CP6 implementation or checker execution.
2. Offline contract / schema checker work precedes any real runner smoke.
3. Real runner smoke requires explicit per-run authorization after CP5 approval.
4. If user manual Windows-side evidence is chosen, read exactly the user-provided file; do not scan directories.

## 不授权任务

- 不创建或读取 HMAC secret。
- 不读取 Windows `.env`。
- 不启动 gateway。
- 不执行 runner runtime。
- 不访问 NAS。
- 不执行 submit / cancel、buy / sell、simulation / live。
- 不执行 provider / lake / catalog publish。
