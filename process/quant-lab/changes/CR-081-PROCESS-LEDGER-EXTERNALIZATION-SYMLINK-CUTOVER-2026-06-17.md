---
cr_id: "CR-081"
status: "closed-current-delivery"
impact_level: "high"
workflow_mode_before: "standard"
workflow_mode_after_change: "standard"
fast_lane_upgrade_reason: "改变 process ledger ownership、Git 跟踪边界、软链接入口和 bootstrap 方式，命中文件所有权、权限边界、迁移和回退风险，必须使用 standard。"
rollback_to: "CR080 closed-current-delivery / current in-repo process ledger"
approval_result: "approved-cp2-cp3-cp5-executed-cp8-approved"
created_at: "2026-06-17T08:03:08+08:00"
created_by: "host-orchestrator"
approved_by: "user"
approved_at: "2026-06-17T08:36:45+08:00"
closed_by: "user"
closed_at: "2026-06-17T08:59:23+08:00"
source: "user-request"
linked_issue: ""
parent_cr: "CR-080"
source_checkpoint: "current conversation"
source_decision_id: "USER-20260617-PROCESS-LEDGER-EXTERNALIZATION"
risk_class: "high"
owner: "host-orchestrator"
target_project_root: "/home/hyde/workspace/quant-lab"
legacy_project_root: "/home/hyde/workspace/local_backtest"
default_external_ledger_root: "/home/hyde/workspace"
default_external_project_path: "process"
---

# CR-081 Process Ledger Externalization / Symlink Cutover Gate

## 变更描述

用户澄清目标：过程文件新建一个独立项目管理，本项目的过程文件通过软链接指向该过程项目下的对应目录。过程项目可以由自身 Git 管理，也可以位于 NAS；本项目不应继续把过程文件本体作为主要 owner。

本 CR 只启动设计和执行门禁，不在 CP2 / CP3 / CP5 approve 前创建外部过程项目、不复制或移动 `process/`、不执行 `git rm --cached`、不创建或替换软链接、不修改 `.gitignore`、不删除任何过程文件、不访问 NAS 内容、不执行 Git commit / push。

## 五维度影响分析

| 维度 | 评估问题 | 受影响对象 | 结论 | 处理动作 |
|---|---|---|---|---|
| 需求层 | 是否改变过程文件归属和项目边界 | `process/`、Meta Flow 过程证据、项目仓库职责 | true | 将过程文件 owner 从业务项目 Git 转为外部 process ledger project；本项目保留软链接入口。 |
| 场景层 | 是否改变日常恢复、审计和新环境 bootstrap 场景 | 新 clone、NAS 不在线、ledger repo 未初始化、symlink 断链 | true | 增加 bootstrap、断链检测、回滚和只读检查场景。 |
| 计划层 | 是否需要分阶段迁移 | CP2/CP3/CP5 -> preflight -> mirror -> verify -> cutover -> validation -> CP8 | true | 三门批准前只生成门禁；执行阶段先 mirror 验证，再 cutover。 |
| 安全层 | 是否引入权限和越权风险 | 软链接跟随、NAS 路径暴露、误删、`rsync --delete`、`find -L` | true | 默认不跟踪 symlink、不使用 `--delete`、不跟随 symlink 操作外部目录，NAS 内容保持不授权。 |
| 交付层 | 是否改变 Git 交付边界和文档位置 | `.gitignore`、`process/STATE.md`、CR-INDEX、外部 ledger 元数据 | true | 生成 ledger pointer / bootstrap / migration docs，验证本项目脚本仍能通过 symlink 读取 `process/STATE.md`。 |

## 文档处理决策

| 受影响文档 | 处理方式 | 旧基线保留方式 | 修订记录位置 | 批准状态 |
|---|---|---|---|---|
| `process/changes/CR-081-PROCESS-LEDGER-EXTERNALIZATION-SYMLINK-CUTOVER-2026-06-17.md` | 新增 / 原文档更新 | N/A | N/A | closed-current-delivery |
| `process/STATE.md` | 原文档更新 | 保留 CR080 closure 与 CR046 paused 主线 | `history` / `cr_tracking` / `human_gate_decisions` | closed-current-delivery |
| `process/changes/CR-INDEX.yaml` | 原文档更新 | 保留全部历史 CR 与候选 | `active_crs` | closed-current-delivery |
| `.gitignore` | 原文档更新 | 保留旧数据忽略规则并追加 process symlink ignore | CR081 cutover | executed |
| `LEDGER.md` / `ledger.yaml` | 新增 | N/A | N/A | executed |
| `scripts/link-engineering-ledger.sh` | 新增 | N/A | N/A | executed |
| `process/**` | 迁移 owner | 外部 ledger mirror + 本项目 symlink；旧 Git 索引已按门禁执行 `git rm --cached -r process` | 外部 ledger README / metadata | closed-current-delivery |
| `docs/design` / `docs/features` / `docs/quality` / `docs/release` | 本轮不迁移 | 继续保留当前项目内文档 owner | N/A | deferred |

## 冲突预检

| 对象 | 影响面 | 结论 | 说明 |
|---|---|---|---|
| `CR-046` | QMT / MiniQMT strategy framework paused | related-no-runtime-overlap | CR081 不恢复 CR046、不启动 runtime、不读凭据；但会影响 process ledger 读取路径，必须保留 CR046 paused 记录。 |
| `CR-080` | data/reports restore closed | dependency | CR080 已关闭，CR081 不触碰 `data/` / `reports/` 内容或恢复资产。 |
| `CR-073` / `CR-074` | target root authority | dependency | target authoritative root 为 `/home/hyde/workspace/quant-lab`；CR081 设计需以 target root 为主。 |
| `CR-076` / `CR-079` | data/reports no-touch 和 metadata-only inventory | no-overlap | CR081 不授权数据内容读取、NAS compare 或 provider rebuild。 |
| `CR-077` / `CR-078` candidates | old root retirement / remote governance | related-no-overlap | CR081 不删除旧根、不做远端治理。 |

冲突预检结论：允许创建正式 CR081 并发起 CP2/CP3/CP5 人工门禁；三门批准前不得执行任何目录迁移、Git 索引变更、软链接替换或外部 ledger 写入。

## 推荐方案

推荐采用外部过程项目为唯一过程文件 owner，本项目只保留本地 ignored symlink 入口。用户在 CP2/CP3/CP5 人工确认时将默认路径修订为 `/home/hyde/workspace/process`，即与 `/home/hyde/workspace/quant-lab` 同级：

```text
/home/hyde/workspace/
  quant-lab/
    process -> ../process

  process/
    README.md
    STATE.md
    changes/
    checks/
    checkpoints/
    context/
    ...
```

外部 process 目录本轮先作为独立 filesystem project 创建；后续如需 Git 或 NAS 管理，可由该外部项目独立初始化。本项目 Git 不跟踪 `process` symlink 本体，只跟踪 `LEDGER.md`、`ledger.yaml`、bootstrap 脚本和 `.gitignore` 规则。

## 分阶段执行设计

| 阶段 | 动作 | 批准前是否允许 | 说明 |
|---|---|---:|---|
| Gate | 创建 CR081、context、CP2/CP3/CP5 checks/checkpoints、launch message | done | 已完成。 |
| Preflight | 检查 current `process/` 是否为真实目录、外部 ledger root 是否不存在或为空、Git tracked process count、断链风险 | approved | 三门已批准，执行前必须记录结果。 |
| Mirror | 将 `process/` 复制到 `/home/hyde/workspace/process` | approved | 默认 `rsync -a`，不得 `--delete`。 |
| Verify | `rsync -an --stats`、CR tracking consistency、human gate check | approved | 不读取数据内容，不访问 NAS 内容。 |
| Cutover | 本项目停止跟踪 `process/**`，本地 `process` 替换为 symlink | approved | 已明确批准 `git rm --cached -r process` 与本地目录备份/替换。 |
| Bootstrap | 新增 `LEDGER.md`、`ledger.yaml`、`scripts/link-engineering-ledger.sh`、`.gitignore` | approved | 支持新环境重建 symlink。 |
| CP8 | 发布就绪、回滚说明、风险接受 | approved | 用户于 2026-06-17T08:59:23+08:00 确认，关闭为 READY_WITH_RISK。 |

## 执行结果

| 项目 | 结果 | 证据 |
|---|---|---|
| 外部 process 项目 | `/home/hyde/workspace/process` 已创建并成为过程文件 owner。 | `process/README.md` |
| mirror | `process/` 已 mirror 到外部目录，dry-run 仅剩目录时间戳差异。 | `process/checks/CR081-PROCESS-LEDGER-CUTOVER-EXECUTION-2026-06-17.md` |
| Git index externalization | 已执行 `git rm --cached -r process`，产生 1572 个 staged process removal；未 commit / push。 | `git diff --cached --name-status -- process` |
| 目录备份 | 原项目内真实 `process/` 已移动到 `/home/hyde/workspace/process.backup-cr081-20260617T083645`。 | `process/checks/CR081-PROCESS-LEDGER-CUTOVER-EXECUTION-2026-06-17.md` |
| symlink | `/home/hyde/workspace/quant-lab/process -> ../process` 已生效，`process/STATE.md` 和 `process/README.md` 可读。 | `readlink process`; `scripts/link-engineering-ledger.sh` |
| CP6 / CP7 | CP6 PASS；CP7 PASS_WITH_RISK。 | `process/checks/CP6-CR081-PROCESS-LEDGER-CUTOVER-CODING-DONE.md`; `process/checks/CP7-CR081-PROCESS-LEDGER-CUTOVER-VERIFICATION-DONE.md` |
| CP8 | READY_WITH_RISK，用户已确认。 | `process/checkpoints/CP8-CR081-DELIVERY-READINESS.md` |

## 不授权项

| ID | 不授权内容 |
|---|---|
| NA-CR081-001 | CP2/CP3/CP5 approved 前，不创建外部过程项目、不复制、移动或删除 `process/`。 |
| NA-CR081-002 | 不执行 `git rm --cached`、`git add`、`git commit`、`git push`、remote set-url、branch/default branch governance、history rewrite、force push 或 tag。 |
| NA-CR081-003 | 不创建、替换或删除 `process` 软链接。 |
| NA-CR081-004 | 不迁移 `docs/design`、`docs/features`、`docs/quality`、`docs/release`，本 CR 第一阶段仅处理 `process/`。 |
| NA-CR081-005 | 不访问 NAS 内容、不创建 NAS mount、不写 NAS、不删除 NAS 文件、不做 NAS promote。 |
| NA-CR081-006 | 不读取 `.env`、token、密码、cookie、session、private key、账户或交易事实。 |
| NA-CR081-007 | 不触碰 `data/`、`reports/` 内容，不读取样本、不解析、不 hash。 |
| NA-CR081-008 | 不恢复 CR046、不启动 QMT/MiniQMT runtime、不执行交易相关操作。 |
| NA-CR081-009 | 不删除、重命名或移动旧根 `/home/hyde/workspace/local_backtest` 或 target root `/home/hyde/workspace/quant-lab`。 |
| NA-CR081-010 | 不执行 `rsync --delete`、`find -L`、destructive cleanup 或跟随 symlink 的批量操作。 |

## 推荐门禁

| Gate | 文件 | 当前状态 | 说明 |
|---|---|---|---|
| CP2 | `process/checkpoints/CP2-CR081-PROCESS-LEDGER-EXTERNALIZATION-BASELINE.md` | approved | 已确认范围和外部 owner 模式。 |
| CP3 | `process/checkpoints/CP3-CR081-PROCESS-LEDGER-ARCHITECTURE-REVIEW.md` | approved | 已确认 symlink / bootstrap / external project 架构。 |
| CP5 | `process/checkpoints/CP5-CR081-PROCESS-LEDGER-CUTOVER-READINESS.md` | approved | 已确认执行顺序和实际授权边界。 |

## fast-lane 判定

| 条件 | 是否命中 | 说明 |
|---|---|---|
| 仅低风险文档更新 | false | 会改变 `process/` owner 和 Git 跟踪边界。 |
| 修改权限 / 安全边界 | true | 涉及软链接、NAS 可选位置、Git 索引和外部 ledger。 |
| 修改安装 / bootstrap 路径 | true | 新环境需要重建 symlink。 |
| 修改外部接口或文件 owner | true | `process/**` owner 从项目仓转到外部 ledger project。 |
| 是否保持 fast-lane | false | standard。 |

## LLD 设计批次门禁

- 是否需要 Story LLD 设计批次：false
- 替代门禁：`CP5-CR081-PROCESS-LEDGER-CUTOVER-READINESS`
- 理由：CR081 是 repository/process-ledger ownership migration gate，不修改应用业务代码、API、数据 schema 或策略逻辑；执行设计由 CP3/CP5 Decision Brief 承载。

## 处理结论

- 当前结论：`closed-current-delivery / READY_WITH_RISK`
- 当前门禁：CP2/CP3/CP5 与 CP8 已人工确认；外部 process 项目路径为 `/home/hyde/workspace/process`；cutover、CP6、CP7 和 CP8 已完成。
- 下一步：建议启动 CR078 remote Git governance / publication gate，处理 staged process/** removals、ledger pointer files 和 CR081 docs 的提交 / 推送策略；启动前仍不授权 Git commit/push、remote write、cleanup、data/reports、凭据、NAS 内容、runtime 或 CR046 recovery。
