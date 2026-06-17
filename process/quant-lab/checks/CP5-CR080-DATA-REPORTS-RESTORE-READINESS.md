---
check_id: "CP5-CR080-DATA-REPORTS-RESTORE-READINESS"
change_id: "CR-080"
checkpoint_type: "automatic"
status: "PASS"
created_at: "2026-06-17T06:42:48+08:00"
owner: "host-orchestrator"
context_capsule: "process/context/CP5-CR080-DATA-REPORTS-RESTORE-READINESS-CONTEXT.yaml"
---

# CP5 CR080 Data/reports Restore Readiness 自动预检

## Entry Criteria

| 条目 | 结果 | 证据 |
|---|---|---|
| CP5 context 存在 | PASS | `process/context/CP5-CR080-DATA-REPORTS-RESTORE-READINESS-CONTEXT.yaml` |
| 替代 LLD 门禁说明存在 | PASS | `process/changes/CR-080-DATA-REPORTS-RESTORE-COPY-GATE-2026-06-17.md` |
| 本阶段未执行 restore/copy | PASS | 仅创建门禁文件；禁止项计数为 0。 |

## Checklist

| # | 检查项 | 结果 | 说明 |
|---|---|---|---|
| 1 | 执行授权条件完整 | PASS | 必须 CP2/CP3/CP5 全部 approved，且 post-approval preflight PASS。 |
| 2 | preflight 必检项明确 | PASS | CR tracking、target 冲突、legacy 来源、denylist、命令白名单均需 PASS。 |
| 3 | 复制命令边界明确 | PASS | 本地 `rsync` 不使用 `--delete` / `--checksum`，不跨远端，不复制 credential-like path。 |
| 4 | 验证证据范围明确 | PASS | 存在性、数量、规模、mtime、exclude 命中、禁止操作计数；不读内容。 |
| 5 | 失败路径明确 | PASS | 任何冲突或越界转 BLOCKED，不自动 cleanup。 |
| 6 | 后续候选不自动启动 | PASS | NAS compare、provider rebuild、CR077、CR078 保持候选。 |

## Exit Criteria

| 条目 | 结果 | 说明 |
|---|---|---|
| 可发起 CP5 人工审查 | PASS | 阻断项 0。 |
| CP5 approve 后仍需 preflight | PASS | approve 不跳过 post-approval preflight。 |

## Deliverables

| 交付物 | 路径 |
|---|---|
| CP5 context | `process/context/CP5-CR080-DATA-REPORTS-RESTORE-READINESS-CONTEXT.yaml` |
| CP5 checkpoint | `process/checkpoints/CP5-CR080-DATA-REPORTS-RESTORE-READINESS.md` |

## 结论

- 结论：`PASS`
- 阻断项：0
- 下一步：等待用户审查 CP2/CP3/CP5。
