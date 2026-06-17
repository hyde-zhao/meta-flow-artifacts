---
status: ready
version: "1.0"
release_artifact_profile: full
release_decision: READY_WITH_RISK
---

# Migration CR080

## 1. 迁移结论

| 项目 | 内容 |
|---|---|
| 是否需要迁移 | yes |
| 是否自动迁移 | yes, within approved local copy only |
| 是否保留兼容路径 | yes, legacy root retained |
| 是否可逆 | partially; process ledger reversible, target asset deletion not authorized |

## 2. 兼容性判断表

| 对象 | 是否变化 | 兼容性 | 需要迁移 | 验证方式 | 回滚方式 |
|---|---|---|---|---|---|
| `STATE.md` schema | no | compatible | no | CR tracking consistency | process ledger patch |
| 模板字段 | no | N/A | no | N/A | N/A |
| 配置格式 | no | compatible | no | N/A | N/A |
| 安装路径 | no | compatible | no | N/A | N/A |
| Agent frontmatter | no | N/A | no | N/A | N/A |
| Skill 输出格式 | no | N/A | no | N/A | N/A |
| 命令参数 | no | compatible | no | N/A | N/A |
| 数据存储结构 | yes | compatible-with-risk | yes | metadata-only source/target count and bytes | destructive cleanup CR required for target deletion |
| target root `reports/` | yes | compatible-with-risk | yes | `rsync -an --stats` 0 transferred | retain or separate cleanup CR |
| target root `data/` | yes | compatible-with-risk | yes | `rsync -an --stats` 0 transferred | retain or separate cleanup CR |

## 3. 迁移步骤

| Step | 操作 | 前置条件 | 验证 | 回退 |
|---|---|---|---|---|
| 1 | CP2/CP3/CP5 approve | 用户回复“同意” | checkpoint status approved | 回退到 candidate |
| 2 | post-approval preflight | target missing / source exists / space sufficient | preflight PASS | 不执行 copy |
| 3 | local `rsync -a` copy | preflight PASS | execution PASS | 不自动 cleanup |
| 4 | metadata-only verification | copy PASS | CP7 PASS_WITH_RISK | 另起 content validation CR |
| 5 | CP8 readiness | CP6/CP7 complete | CP8 pending user approval | 用户 reject / 修改 |

## 4. N/A 说明

| 项目 | 原因 | 后续触发条件 |
|---|---|---|
| 内容 hash migration proof | CR080 不授权 content read/checksum | 用户要求内容级证明 |
| NAS/provider migration | 本轮只用 local retained assets | local assets 不足或用户要求归档/重建 |
| old root retirement | 破坏性动作，CR080 不授权 | 用户启动 CR077 或等价 CR |
