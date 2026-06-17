---
check_id: "CP2-CR080-DATA-REPORTS-RESTORE-BASELINE"
change_id: "CR-080"
checkpoint_type: "automatic"
status: "PASS"
created_at: "2026-06-17T06:42:48+08:00"
owner: "host-orchestrator"
context_capsule: "process/context/CP2-CR080-DATA-REPORTS-RESTORE-CONTEXT.yaml"
---

# CP2 CR080 Data/reports Restore Baseline 自动预检

## Entry Criteria

| 条目 | 结果 | 证据 |
|---|---|---|
| 用户明确启动候选 | PASS | 当前请求：`启动CR079-restore-candidate` |
| CR079 已关闭并保留 restore/copy candidate | PASS | `process/checkpoints/CP8-CR079-DELIVERY-READINESS.md` |
| 正式 CR080 已创建 | PASS | `process/changes/CR-080-DATA-REPORTS-RESTORE-COPY-GATE-2026-06-17.md` |
| 本阶段未执行 restore/copy | PASS | 仅创建 process / checkpoint / context 文件。 |

## Checklist

| # | 检查项 | 结果 | 说明 |
|---|---|---|---|
| 1 | CR 编号无正式文件冲突 | PASS | CR077 / CR078 仍为候选；CR079 已正式关闭；新正式 CR 使用 CR080。 |
| 2 | 候选项未重复绑定正式 CR | PASS | `CR079-restore-candidate.formal_cr_path` 原为空，本轮指向 CR080。 |
| 3 | 冲突预检覆盖 active / candidate | PASS | CR046 paused、CR077/CR078、NAS compare、provider rebuild 均已分类。 |
| 4 | CP2 Decision Brief 已包含 Decision Collection Coverage | PASS | 见 CP2 人工审查稿。 |
| 5 | 不授权项覆盖敏感范围 | PASS | 内容读取、凭据、NAS/provider/lake/catalog、remote Git、runtime、old root retirement、cleanup 均排除。 |

## Exit Criteria

| 条目 | 结果 | 说明 |
|---|---|---|
| 可发起 CP2 人工审查 | PASS | 阻断项 0。 |
| CP2 approve 不等于执行授权 | PASS | 执行仍需 CP2/CP3/CP5 全部 approved 和 preflight PASS。 |

## Deliverables

| 交付物 | 路径 |
|---|---|
| CP2 context | `process/context/CP2-CR080-DATA-REPORTS-RESTORE-CONTEXT.yaml` |
| CP2 checkpoint | `process/checkpoints/CP2-CR080-DATA-REPORTS-RESTORE-BASELINE.md` |

## 结论

- 结论：`PASS`
- 阻断项：0
- 下一步：等待用户审查 CP2/CP3/CP5。
