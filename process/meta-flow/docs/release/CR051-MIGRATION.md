---
status: current-execution-na
version: "0.5.0-proposed"
release_artifact_profile: full
release_decision: READY_WITH_RISK
---

# CR-051 Migration

## 1. 迁移结论

| 项目 | 内容 |
|---|---|
| 当前是否执行迁移 | 否（N/A） |
| 是否自动迁移 | 否；没有 migration CLI |
| 当前兼容路径 | 未迁移项目继续使用现有布局/路由，禁止静默切换 |
| 是否可逆 | 当前 N/A；未来逐项目 cutover 必须先证明可回滚 |

CR-051 只交付只读 migration preflight manifest 和 handoff 能力。它不复制、移动、删除产物，不重挂软链接，不创建或切换真实 worktree/ref，也不执行 main↔integration 同步。

## 2. 目标结构与生命周期

项目优先的目标目录为 `<project_name>/docs` 与 `<project_name>/process`。artifact Git 生命周期为 shared `main` → `projects/<project>/integration` → 项目命名空间短期 CR 分支；每个 CR 只从最新项目 integration 派生并回合到该 integration。shared main 的双向同步由 CR 外人工完成。

源码仓库不采用上述长期 integration：它仍从 default branch 创建 CR，并在完成后回 default branch。

## 3. 兼容性判断

| 对象 | 是否变化 | 兼容性 | 当前需要迁移 | 验证方式 | 回滚入口 |
|---|---|---|---|---|---|
| `STATE.md` schema | 无本轮真实状态迁移 | compatible | 否 | CP result/return evidence | N/A |
| artifact 路由元数据 | 未来按项目变化 | conditional | 当前否 | 只读 manifest + routing health | 恢复批准前 inventory |
| 目录结构 | 未来转 `<project>/docs`、`<project>/process` | conditional | 当前否 | mapping/deny/collision 检查 | 保留旧目录与映射 |
| worktree 分支 | 未来驻留项目 integration | conditional | 当前否 | exact ref/OID + clean-state | 恢复经确认 integration |
| 软链接 | 未来按项目重挂 | conditional | 当前否 | link target inventory/no-follow | 恢复旧 link target |
| Agent frontmatter | 否 | compatible | 否 | N/A | N/A |
| Skill 输出格式 | 否 | compatible | 否 | N/A | N/A |
| 命令参数 | aggregate 已实现；migration CLI 不存在 | compatible | 否 | CLI help + API tests | N/A |
| artifact 数据 | 未来逐项目移动/复制属于真实迁移 | unknown-until-pilot | 当前否 | digest/readback/owned-path | per-project rollback plan |

## 4. 未来逐项目迁移的授权前步骤

| Step | 只读/人工活动 | 前置条件 | 通过条件 | 失败行为 |
|---:|---|---|---|---|
| 1 | 选择单个项目并冻结迁移范围 | 用户指定 project、artifact root 与 owned paths | sibling paths 明确排除 | BLOCKED |
| 2 | 人工完成 shared main 与项目 integration 所需同步策略决策 | 精确 OID、方向与保护规则可见 | 决策有审计记录 | BLOCKED |
| 3 | 运行只读 preflight manifest | 读取范围已授权 | 11 个 section 完整、deny/collision/link 风险可审阅 | MANUAL_REVIEW/BLOCKED |
| 4 | 审阅容量、权限、durable intent 与回滚 inventory | clean-state 与恢复路径明确 | 所有不可豁免检查通过 | BLOCKED |
| 5 | 单独请求真实迁移/链接/worktree/ref 授权 | 前四步通过 | 人工明确批准精确动作 | 未批准则停止 |

本文件不会自动进入第 5 步，也不包含可直接执行的迁移命令。

## 5. 已知迁移风险

| Risk ID | 影响 | 缓解/重访条件 |
|---|---|---|
| R-AW005-DIR-ENUM-TOCTOU | 不可信并发 rename/replace 可能超出静态枚举证明 | 此类输入出现前完成 full-LLD + dirfd/openat 或等价设计 |
| CR051-RISK-REAL-MIGRATION-UNVERIFIED | fixture 不证明真实文件系统、跨设备、权限和链接 cutover | 首个项目独立 pilot |
| CR051-RISK-WINDOWS-NATIVE-NOFOLLOW | Windows no-follow 行为未实机验证 | Windows 项目迁移前原生 pilot |
| CR051-RISK-CROSS-DEVICE-PILOT | 跨设备路径/容量行为未真实验证 | 跨设备 artifact root 使用前 pilot |

## 6. N/A 说明

当前没有用户指定迁移项目，也没有真实文件/链接/repository mutation 授权，因此迁移执行、数据恢复和 cutover 验证均为 N/A。CP8 `approve` 只接受交付候选及风险，不自动启动任何项目迁移。
