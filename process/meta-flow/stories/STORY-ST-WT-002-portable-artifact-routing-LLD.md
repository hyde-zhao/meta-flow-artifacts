---
story_id: "ST-WT-002"
title: "建立 portable artifact routing 与唯一内部 docs canonical"
story_slug: "portable-artifact-routing"
lld_version: "1.0"
tier: "L"
status: "approved"
confirmed: true
created_by: "host-orchestrator-inline/meta-dev"
created_at: "2026-07-14"
confirmed_by: "user"
confirmed_at: "2026-07-15T12:05:17Z"
feature_design_refs: ["process/docs/features/cr047-routing/DESIGN.md", "process/docs/features/cr047-routing/TEST-PLAN.md", "process/docs/features/cr047-routing/TASKS.md"]
lld_policy: {required_level: "full-lld", trigger_reasons: ["cross-repo-routing", "portability", "shared-contract"]}
open_items: 0
---

# LLD: ST-WT-002 — Portable Artifact Routing

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-14 | host-orchestrator-inline/meta-dev | 冻结相对路由、canonical docs 与 clean clone 契约。 |

## 0. 工程依据与上游设计依据

工程依据为 ADR-WT-002、Feature routing design、`.meta-flow-process.yaml` 当前 contract、`workspace/routing.py` 和 DIRECTORY-CONTRACT。`process` 是项目入口，真实存储在 artifact repo；内部 docs 只以 `process/docs/**` 为 canonical。

## 1. 目标

使外置 process 路由在 workspace 移动/重新 clone 后通过相对锚点恢复，并保证内部 docs canonical copy 数=1、根目录兼容视图即使存在也只能是 ignored/generated symlink。

## 2. 需求（Functional / Non-Functional）

- 解析 `artifact_root`/`project_process_root` 必须基于声明 anchor，不固化 `/home/hyde/...`。
- `workspace link/check` 对健康 symlink、断链、project_name 冲突、regular path 冲突给出确定性结果。
- `.meta-flow-process.yaml` 的再次 link 不应只因时间戳改变而产生内容漂移。
- `process/docs/**` 是内部产品/设计/质量文档唯一 tracked canonical；重复可写 copy 为 blocker。
- 移动 workspace fixture 通过率=100%，不访问真实 artifact repo 以外目标。

## 3. 模块拆分与职责

| 模块 | 职责 |
|---|---|
| `meta_flow/workspace/routing.py` | 锚点解析、link/check、元数据幂等与路径边界 |
| `meta_flow/cli.py` | 复用 workspace 命令并渲染健康信息 |
| `delivery/rules/DIRECTORY-CONTRACT.{md,yaml}` | 声明 canonical/compatibility/forbidden 布局 |
| `tests/test_cr047_artifact_routing.py` | relocation、idempotency、conflict、canonical-count fixtures |

## 4. 代码结构与文件影响范围

修改 routing、directory contracts，必要时最小 CLI 接线；创建 CR-047 routing 测试。不得处理 prelink backup，不得触碰 `process/quant-lab/**`，不得在根 `docs/product|design|quality` 生成可写 copy。

## 5. 数据模型与持久化设计

`ProcessRoute` 含 project_root、link_path、artifact_root、project_process_root、project_name、routing_mode、anchors；所有持久化值应为相对或 anchor+relative。`RouteFinding` 覆盖 `REGULAR_PATH_CONFLICT`、`BROKEN_LINK`、`PROJECT_NAME_MISMATCH`、`ANCHOR_ESCAPE`、`DUPLICATE_CANONICAL_DOC`。`created_at` 不参与幂等内容比较或只在首次创建写入。

## 6. API / Interface 设计

`resolve_process_route(project_root, metadata) -> ProcessRoute`；`link_workspace(..., replace=False)` 只创建/验证 symlink与元数据；`check_workspace(...) -> health+findings`。调用方是 CLI、Doctor、installer dry-run；失败时不静默迁移 regular path。

## 7. 核心处理流程

1. 读取项目根与显式 artifact root；规范化 anchor。
2. 验证 `<artifact-root>/process/<project-name>` 及 metadata project_name。
3. 若 `process` 是健康 symlink则幂等检查；regular path/断链则 fail-closed。
4. 扫描内部 docs canonical/compatibility 形态并计算 writable tracked copy。
5. 输出健康明细；不修改其他项目目录。

## 8. 技术细节与设计细节

路径比较统一用 `Path.resolve(strict=False)` 加 `relative_to` 边界检查；metadata writer 使用稳定字段顺序，健康 relink 保留原 `created_at`。clean clone fixture 在临时父目录创建 sibling source/artifact repos 后整体移动，验证无绝对路径依赖。

## 9. 安全与性能设计

拒绝 artifact root 逃逸、project_name 路径分隔符、symlink 指向非声明 project。测试只使用临时目录。扫描仅覆盖声明的内部 docs 层，复杂度 `O(files)`；不递归 archive/backup/quant-lab。

## 10. 测试设计

| 场景 | 预期 |
|---|---|
| 健康 sibling route | health=ok |
| workspace 整体移动后 check | 100% 通过，无旧绝对路径 |
| 再次 link | metadata hash 不变 |
| `process` regular path | 明确冲突、无自动迁移 |
| broken link/project mismatch/anchor escape | 专用 blocker |
| canonical+可写根 copy | duplicate blocker |
| canonical+ignored symlink view | copy count=1、合法 |
| 既有 workspace tests | 无回归 |

## 11. 实施步骤

1. `TASK-WT-002-01`：归一化 anchors、relative metadata 与幂等 writer。
2. `TASK-WT-002-02`：实现 route/canonical docs findings 和 fail-closed 分支。
3. `TASK-WT-002-03`：补 relocation、repeat-link、regular-path、duplicate-copy fixtures并同步 DIRECTORY-CONTRACT。
4. 运行定向 pytest、workspace link/check 临时目录 dry-run；不操作用户 backup。

## 12. 风险、难点与预研建议

风险是 resolve 后跨边界、幂等 link 重写时间戳、把 generated view 当 canonical。通过路径边界、metadata golden、tracked/ignored/symlink 分类解决。若需迁移 regular path，必须独立用户授权，不由本 Story 隐式执行。

## 13. 回滚与发布策略

新解析逻辑出错时回滚到既有 link/check，并保留 relocation fixtures；不自动还原或删除用户路径。directory contract 变更与 checker 同步发布，避免文档先行造成假约束。

## 14. DoD（Definition of Done）

- [ ] 0–14 章节完整、`open_items=0`。
- [ ] relocation/relink/conflict/escape/canonical-count fixtures 100% 通过。
- [ ] 健康 repeat-link metadata hash 不变。
- [ ] tracked writable internal docs canonical copy=1。
- [ ] backup/quant-lab 访问与修改次数=0。
- [ ] CP5 未批准前 `confirmed=false`，不开始实现。

## 人工确认区

- 结论：`approved`
- 审查人：user
- 审查时间：2026-07-15T12:05:17Z
- 修改意见：
- 风险接受项：inline fallback；CP7/CP8 风险上限。
