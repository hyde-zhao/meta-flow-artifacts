---
status: ready
version: "1.0"
release_artifact_profile: full
release_decision: READY_WITH_RISK
---

# Rollback CR080

## 1. 回滚摘要

| 项目 | 内容 |
|---|---|
| 回滚目标版本 | pre-CR080 process ledger / target root without restored reports/data |
| 回滚范围 | 默认只回滚过程账本；target asset 删除 / 移动不在当前授权范围 |
| 是否涉及数据恢复 | yes, but destructive asset rollback is not authorized |
| 是否存在不可回滚项 | yes |
| 决策人 | human |

## 2. 回滚触发条件

| Trigger ID | 条件 | 监控 / 证据 | 决策人 |
|---|---|---|---|
| RB-CR080-T01 | CP8 不接受 metadata-only 风险 | CP8 reject / 修改 | human |
| RB-CR080-T02 | 后续发现 target assets 与预期不符 | 用户反馈 / follow-up validation | human |
| RB-CR080-T03 | 需要删除或移动 target restored assets | 独立 destructive cleanup request | human |

## 3. 回滚步骤

| Step | 操作 | 前置条件 | 验证 | 风险 |
|---|---|---|---|---|
| 1 | 将 CR080 过程状态标记为 `blocked` / `changes_requested` | 用户明确要求 | CR-INDEX / STATE 一致性检查 | 只影响过程账本 |
| 2 | 保留 target `reports/` / `data/`，不自动删除 | 默认 | target 目录仍存在 | 占用磁盘空间 |
| 3 | 若必须删除 / 移动 target assets，另起 destructive cleanup CR | 用户独立授权 | 新 CR CP2/CP3/CP5 | 破坏性操作风险 |

## 4. 回滚验证

| 验证项 | 方法 | 结果 |
|---|---|---|
| 过程账本回滚 | CR tracking consistency | conditional |
| target asset 删除 / 移动 | N/A | 当前不授权 |
| 状态 / 配置恢复 | STATE / CR-INDEX review | conditional |

## 5. 不可回滚项

| 对象 | 是否存在 | 原因 | 处理 |
|---|---|---|---|
| target `reports/` / `data/` 已写入 | yes | 删除/移动属破坏性动作，CR080 未授权 | 保留；如需处理，另起 CR。 |
| 内容级验证缺失 | yes | 内容读取 / checksum 未授权 | 另起 content validation CR。 |
