---
document_type: story-fix-input
cr_id: CR-051
story_id: ST-AW-001
status: rework-required
---

# CR-051 / ST-AW-001 回修输入

| Finding | 必须修改 | 必须新增验证 | 完成条件 |
|---|---|---|---|
| CP7-AW-001-F01 | adapter 在采用 target 前验证 `config.project_id == decision.project_id`，不一致时 `route_mismatch` | project-a config + project-b decision | health.ok=false、actual target 不作为授权、错误可行动 |
| CP7-AW-001-F02 | legacy owned path 逐项类型校验并转为稳定 `RoutingValidationError` | 非字符串元素至少 1 个 | 无裸 `TypeError`，字段定位含 index，错误码属于既有集合 |

回修不得修改 layout、branch topology、真实 artifact 或其他 Story 文件。若发现必须改变公共契约，停止并提交 design delta / clarification。
