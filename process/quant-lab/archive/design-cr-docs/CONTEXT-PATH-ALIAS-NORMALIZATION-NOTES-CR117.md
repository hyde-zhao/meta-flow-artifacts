---
title: "Context Path Alias Normalization Notes"
cr_id: "CR-117"
status: "closed-ready"
created_at: "2026-06-22T17:53:44+08:00"
created_by: "host-orchestrator"
---

# Context Path Alias Normalization Notes

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| v0.1 | 2026-06-22 | host-orchestrator | 创建 CR117 路径别名展示约定草案，供 CP2 审查。 |
| v0.2 | 2026-06-22 | host-orchestrator | CP2 approved 后冻结推荐路径展示原则，准备 CP8 readiness 审查。 |
| v1.0 | 2026-06-22 | host-orchestrator | CP8 approved；CR117 closed READY；后续 `FU-CR117-001` 转入 `CR-118` CP2 门禁。 |

## 背景

CR114 / CR116 复盘时发现，同一类设计证据在不同上下文中可能出现两种路径表达：

- `process/docs/design/*`
- `docs/design/*`

当前仓库的 `process` 是指向外置 artifact process root 的 symlink；`process/docs/design/*` 是运行态 / 审计态的稳定入口。`docs/design/*` 在当前项目中可能作为历史说明、语义别名或本地 symlink 入口出现，但不应在人工门禁中替代真实 process 证据路径。

## 推荐展示约定

| 场景 | 推荐路径表达 | 可接受别名 | 说明 |
|---|---|---|---|
| checkpoint / context capsule / CR 证据引用 | `process/docs/design/<file>` | `docs/design/<file>` 仅作语义别名 | 人工审查优先使用 process 入口，便于从 `process/STATE.md` 和 CR ledger 追溯。 |
| 对话中的 clickable file link | `process/docs/design/<file>` | 无需同时打印别名 | 避免让用户判断两个路径是否为不同文件。 |
| 历史文档中已有 `docs/design/*` | 保留历史文本 | 可在新 notes 中解释为 legacy alias | 不批量重写历史文档。 |
| 外置 artifact 真实路径 | 不默认展示绝对路径 | 仅在路由审计或 workspace health 中展示 | 减少设备相关路径扩散。 |
| 源码仓库公开 docs | 按目标项目 README / docs 约定 | N/A | 本 notes 不改变生产项目文档路由。 |

## 当前 CR117 规则

| 规则 ID | 规则 | 理由 | 例外 / 切换条件 |
|---|---|---|---|
| PATH-CR117-01 | 新增 checkpoint、CR、context capsule 默认引用 `process/...` 路径 | 与 process symlink / artifact routing 的运行态入口一致 | 用户明确要求面向公开 docs 的文案时，可用目标项目 docs 路径。 |
| PATH-CR117-02 | `docs/design/*` 可作为历史语义别名保留，但不作为人工门禁唯一证据路径 | 防止同一证据出现两个“真相路径” | 若未来迁移完成并统一 docs 路由，可另起 CR 更新本规则。 |
| PATH-CR117-03 | 不在本 CR 中创建、删除、修改 symlink 或移动文件 | 当前目标是说明约定，不是路径迁移 | 需要真实路径变更时，启动目录 / symlink 迁移 CR。 |
| PATH-CR117-04 | 不把 artifact repo 绝对路径写入普通人工门禁消息 | 减少环境耦合，避免用户误以为需要手工访问外置仓库 | workspace health、路由审计和故障排查可展示绝对路径。 |

## 发现与风险

| 发现 ID | 发现 | 严重度 | 状态 | 处理 |
|---|---|---|---|---|
| F-CR117-01 | `process/docs/design` 和 `docs/design` 混用会让人工审查误以为存在两份不同设计证据 | MEDIUM | open-for-cp2 | CP2 确认展示约定后，后续新文档按推荐路径写。 |
| F-CR117-02 | 批量改写历史路径会制造无意义 diff，并可能破坏历史审计语境 | MEDIUM | accepted-risk | 本 CR 不批量改历史，只对新产物和后续提示采用约定。 |
| F-CR117-03 | 若未来 checker 要强制路径规范，需源码 / tests / checker implementation 授权 | LOW | follow-up-candidate | 不在 CR117 实施。 |

## 非目标

- 不修改目录结构。
- 不修改 symlink。
- 不迁移文件。
- 不修改源码、tests 或 checker implementation。
- 不启动 runtime、NAS、凭据、交易写或 provider/lake/catalog publish。
- 不批量重写历史 `STATE.md`、checkpoint 或旧 CR 正文。

## CP2 待确认

| 决策 ID | 决策类型 | 推荐方案 | 备选方案 | 风险 / 回退 |
|---|---|---|---|---|
| DQ-CP2-CR117-01 | scope | 只做 no-code/no-runtime notes | A. 暂停；B. 改目录迁移 CR；C. 改 checker implementation CR | 推荐方案风险低；B/C 需独立授权。 |
| DQ-CP2-CR117-02 | implementation | 新增证据引用优先 `process/...`，`docs/design/*` 作为历史/语义别名 | A. 只写 `process/docs/design/*`；B. 只写 `docs/design/*`；C. 保持混用 | 推荐方案兼顾可点击入口和历史语义。 |
| DQ-CP2-CR117-03 | runtime_authorization | 不授权目录、symlink、源码、tests、checker、runtime、NAS、凭据、交易写或 publish | A. 另起目录迁移 CR；B. 另起 checker CR；C. 另起 runtime gate | 越界时停止 CR117 默认路线。 |

## CP2 审批后结论

| 字段 | 内容 |
|---|---|
| 审批状态 | approved |
| 审批时间 | 2026-06-22T18:02:18+08:00 |
| 采用规则 | 新增 checkpoint、CR、context capsule 和人工门禁提示默认引用 `process/...` 路径；`docs/design/*` 作为历史 / 语义别名保留，不作为唯一证据路径。 |
| 保留边界 | 不改目录、不改 symlink、不移动或重命名文件、不改源码、不改 tests、不改 checker implementation、不启动 runtime。 |

## CP8 关闭结论

| 字段 | 内容 |
|---|---|
| 审批状态 | approved |
| 审批时间 | 2026-06-22T18:21:08+08:00 |
| 关闭状态 | CR117 closed-current-delivery / READY |
| 接受风险 | 历史 `docs/design/*` 路径表达不批量重写，新门禁 / CR / capsule 引用优先使用 `process/...` 路径。 |
| 后续事项 | `FU-CR117-001 Path Alias Checker Enforcement Candidate` 已按用户要求启动为 `CR-118` CP2 门禁；当前不直接改源码、tests 或 checker implementation。 |
| 不授权边界 | 不改目录、不改 symlink、不移动或重命名文件、不改源码、不改 tests、不改 checker implementation、不启动 runtime、NAS、凭据、交易写或 provider/lake/catalog publish。 |
