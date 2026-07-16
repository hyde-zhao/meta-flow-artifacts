---
status: draft
version: "1.0"
feature_id: "FEAT-GB-04"
---

# FEAT-GB-04 Tasks

| TASK-ID | 顺序 | 任务 | 输出 | 所有权 | 验证 | 状态 |
|---|---:|---|---|---|---|---|
| TASK-GB-004-01 | 1 | 创建authz/eligibility/outcome/projection types | lifecycle module | primary | unit | pending |
| TASK-GB-004-02 | 2 | 创建ordinary exact OID ref push与post-check | git_sync/lifecycle | shared | bare+spy | pending |
| TASK-GB-004-03 | 3 | 创建artifact-first paired executor/resume | lifecycle/cli | primary/shared | fault fixture | pending |
| TASK-GB-004-04 | 4 | 创建2/2 projection writer gate和state/current不推进fixture | lifecycle/cr_lifecycle/current/tests | shared | projection fixture | pending |

阻塞项=0；若TASK-04不能机器保证PARTIAL projection=false，阻断并回CP2/CP3切project-first。
