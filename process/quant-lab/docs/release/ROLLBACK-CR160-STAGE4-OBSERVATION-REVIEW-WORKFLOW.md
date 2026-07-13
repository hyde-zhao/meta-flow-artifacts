# CR160 Rollback - Stage 4 Observation Review Workflow

## Rollback Scope

Rollback is documentation/process rollback only. CR160 does not modify executable source code, state schema, installation scripts, runtime services, data lake contents, provider data, broker state, catalog pointers or registry entries.

## Rollback Options

| Option | Use When | Action |
|---|---|---|
| CP8 reject | User does not accept `READY_WITH_RISK` | Mark CP8 as rejected and keep CR160 active or return to CP3 design clarification. |
| CP8 changes requested | User wants wording or follow-up changes | Revise CP8 release docs/checkpoint and relaunch CP8. |
| Reopen CP3 design | HLD/checklist semantics are not acceptable | Route back to CP3; update HLD/checklist/seed classification and rerun CP7. |
| New follow-up CR | User wants checker/schema/runtime/paper authorization | Close or pause CR160 design slice and start a separate scoped CR. |

## Verification After Rollback

Run CP7 result/evidence validation again if any HLD, checklist or CR155 classification evidence changes. No runtime rollback command is applicable.

## Non-Rollback Items

No live data, broker state, catalog pointer, registry state, model state, prediction output or remote release artifact was changed by CR160.
