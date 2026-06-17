---
cr_id: "CR-085"
status: "closed-current-delivery"
impact_level: "critical"
workflow_mode_before: "standard"
workflow_mode_after_change: "standard"
fast_lane_upgrade_reason: "涉及远端同步、远端删除、clean publication、可能的历史清理 / force push，必须保持 standard。"
rollback_to: "remote master now 57d4fc1e9b8808b4371e12af72f9cddcba01aaa0; rollback/revert requires a new explicit gate"
approval_result: "approved"
created_at: "2026-06-17T13:04:40+08:00"
created_by: "host-orchestrator"
approved_by: "user"
approved_at: "2026-06-17T13:56:36+08:00"
execution_result: "closed-current-delivery-ready-with-risk"
executed_at: "2026-06-17T15:19:06+08:00"
closed_at: "2026-06-17T15:19:06+08:00"
source: "user request: 需要处理远程同步，不应该推送的内容也需要从远端删除"
linked_issue: ""
parent_cr: "CR-084"
source_checkpoint: "process/checks/CR085-REMOTE-READONLY-SCAN-2026-06-17.md"
source_decision_id: "USER-20260617-REMOTE-SYNC-CLEANUP"
follow_up_type: "remote-synchronization-cleanup-gate"
risk_class: "critical"
owner: "host-orchestrator"
revisit_condition: "用户要求处理远程同步，并要求不应推送内容也从远端删除。"
acceptance_criteria: "完成只读远端扫描；按用户修正保留 .env.example、禁止上传 .env；生成待排除清单；在 CP2/CP3/CP5 批准前不执行 remote add、fetch、push、remote deletion、history rewrite、force push、tag 或 default branch governance。"
close_condition: "用户批准 clean publication / 清理策略后，只按批准方式执行；普通删除提交、clean publication commit、history purge / force push 必须分开授权；任一 stop condition 命中即停止。"
cr_index_path: "process/changes/CR-INDEX.yaml"
---

# CR-085 Remote Synchronization / Cleanup Gate

## 变更描述

用户指出远程同步必须处理，且“不应该推送的内容也需要从远端删除”。这改变了 CR084 的边界：CR084 只覆盖 remote URL、remote name、目标分支、只读 `ls-remote`、`remote add`、set-upstream 和一次普通非 force push；不覆盖远端内容清理、默认分支治理、历史清理、force push 或删除策略。

CR085 承接远端同步和清理策略。当前只读扫描已经完成，不执行任何远端写动作。

用户随后修正边界：`.env.example` 可以保留，`.env` 不要上传。因此 `.env.example` 不再作为远端删除对象；`.env` 是禁止上传边界，本轮未读取 `.env` 内容。

| 字段 | 值 |
|---|---|
| 目标远端 | `git@github.com:hyde-zhao/quant-lab.git` |
| 远端 HEAD | `refs/heads/master` |
| 远端 master | `3ade165e8b1edad031a911490cf6c1cee942616e` |
| 本地候选 commit | `d4d2881 CR078 process ledger publication gate` |
| 远端 tracked 文件数 | 136 |
| 本地候选 tracked 文件数 | 570 |
| 远端当前禁止路径命中 | 无；`.env.example` 为允许保留模板，扫描未见 `.env` |
| 本地候选禁止路径命中 | `runs/RUN-EXEC-20260613-001.md`；`.env.example` 允许保留，扫描未见 `.env` |

## 冲突预检

| 检查项 | 结论 | 说明 |
|---|---|---|
| CR084 | scope-expanded | CR084 已 approved，但远端清理策略超出其普通 push 范围。 |
| CR046 | related-no-overlap | CR085 不恢复 CR046，不读取凭据，不运行 QMT / MiniQMT。 |
| Remote URL | confirmed-for-readonly-scan | 用户选择 quant-lab GitHub；已执行只读 `ls-remote` 和临时 scan clone。 |
| Remote write | not-authorized | 远端删除、push、force、tag、default branch governance 均未授权。 |
| Direct push d4d2881 | blocked | `d4d2881` 包含 `runs/RUN-EXEC-20260613-001.md`，且会把远端 tracked 文件面从 136 扩到 570，不能直接作为远端同步目标。 |
| Cleanup mode | pending-human-decision | 用户选择先扫描清单；当前远端无必须删除项，clean publication 方式尚未批准。 |
| History purge | not-authorized | 如需从历史中消除文件，必须另设高风险 gate 和 force push 授权。 |

## 文档处理决策

| 受影响文档 | 处理方式 | 旧基线保留方式 | 修订记录位置 | 批准状态 |
|---|---|---|---|---|
| `process/changes/CR-085-REMOTE-SYNCHRONIZATION-CLEANUP-GATE-2026-06-17.md` | 新增 | N/A | 本文件 frontmatter / 正文 | pending |
| `process/checks/CR085-REMOTE-READONLY-SCAN-2026-06-17.md` | 新增 | N/A | N/A | scan-complete |
| `process/changes/CR-084-REMOTE-TARGET-CONFIGURATION-PUSH-EXECUTION-GATE-2026-06-17.md` | 原文档更新 | 保留 CR084 approved 事实 | 本文件追加 scope-expanded 说明 | pending |
| `process/STATE.md` | 原文档更新 | 保留 CR046 paused、CR084 approved 和远端扫描证据 | N/A | pending |
| `process/changes/CR-INDEX.yaml` | 原文档更新 | 保留 CR084 和 CR085 索引 | N/A | pending |

## 五维度影响分析

| 维度 | 评估问题 | 受影响对象 | 结论 | 处理动作 |
|------|----------|-----------|------|---------|
| 需求层 | 是否新增远端同步 / 清理需求 | remote master、clean publication、delete list | true | CR085 记录远端同步目标；`.env.example` 允许保留，`.env` 禁止上传。 |
| 场景层 | 是否新增删除 / 回滚场景 | clean publication、`.env` 误上传、历史清理、误删回滚 | true | 当前远端无必须删除项，但需要 CP2/CP3/CP5 决策后续 clean publication 与异常删除路径。 |
| 计划层 | 是否改变 CR084 执行路径 | CR084 普通 push | true | 暂停 CR084 写动作，转入 CR085。 |
| 安全层 | 是否涉及高风险远端操作 | push、delete、force、history rewrite | true | 默认只允许只读扫描；写动作逐项审批。 |
| 交付层 | 是否需要新发布证据 | cleanup manifest、scan report、post-cleanup verification | true | 先生成扫描证据，再门禁删除策略。 |

## 推荐策略

推荐先采用两阶段策略：

1. 边界修正：保留 `.env.example` 模板；禁止上传 `.env`。当前远端扫描未发现 `.env`，因此不因 `.env.example` 发起普通删除提交。
2. 后续 clean publication：不要直接 push `d4d2881`。先生成一个 clean publication commit / branch，排除 `.env`、`runs/**`、`reports/**`、`data/**`、`process/**` 和未批准的过程台账 / 迁移文档；`.env.example` 可以保留为模板，再走独立 push gate。

普通远端删除只在后续扫描发现 `.env` 或其他禁止路径已进入远端最新树时才考虑。历史清理 / force push 只在用户明确要求“从整个 Git 历史中清除”并接受高风险后才考虑。

## 待决策问题

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 影响 / 风险 |
|---|---|---|---|---|---|
| DQ-CP2-CR085-01 | scope | 是否暂停 CR084 的直接 push，改由 CR085 先处理远端同步 / 清理？ | 暂停 CR084 写动作，CR085 先收敛 cleanup manifest。 | 继续 CR084 普通 push；取消远端清理。 | 继续普通 push 会把本地候选中的禁止路径带到远端。 |
| DQ-CP3-CR085-01 | architecture | “从远端删除”在当前扫描结果下如何处理？ | 当前不删除 `.env.example`；只确认 `.env` 禁止上传和 clean publication 排除规则。 | 删除 `.env.example`；直接进入历史清理。 | 删除 `.env.example` 与用户修正冲突；历史清理风险高且未授权。 |
| DQ-CP5-CR085-01 | runtime_authorization | CP5 是否授权后续执行远端写动作？ | 不授权远端写入；approve 后只允许准备 clean publication manifest / candidate 供下一次审查。 | 授权普通 push；授权普通删除提交；授权 history purge。 | 推荐方案保持远端不变；备选会在 clean candidate 未生成前写远端，风险高。 |

## CR085 CP2/CP3/CP5 批准结果

用户于 `2026-06-17T13:56:36+08:00` 回复“好的继续”，按 CR085 CP2 / CP3 / CP5 approve 处理。

本次批准接受以下边界：

- `.env.example` 可以保留为模板。
- `.env` 不得上传、读取、打印或保存。
- 当前远端 master 没有 `.env`、`reports/`、`runs/`、`data/`、`process/` 命中，因此当前远端删除清单为空。
- 直接 push `d4d2881` 仍然禁止，因为本地候选包含 `runs/RUN-EXEC-20260613-001.md` 且会扩大远端 tracked 文件面。
- approve 不授权任何远端写入，只允许准备 clean publication manifest / candidate 供下一次审查。

## Clean Publication Candidate 路径清单

已生成路径级候选清单和扫描证据：

| 产物 | 路径 | 状态 |
|---|---|---|
| clean publication candidate manifest | `docs/release/CLEAN-PUBLICATION-CANDIDATE-MANIFEST-CR085.yaml` | PASS_WITH_RISK |
| candidate path scan evidence | `process/checks/CR085-CLEAN-PUBLICATION-CANDIDATE-PATH-SCAN-2026-06-17.md` | PASS_WITH_RISK |

路径分类结果：

| 指标 | 数值 | 说明 |
|---|---:|---|
| 远端 tracked 文件数 | 136 | 当前 remote master `3ade165e8b1edad031a911490cf6c1cee942616e`。 |
| 本地候选 tracked 文件数 | 570 | 本地候选 `d4d2881`。 |
| must include 路径数 | 291 | 核心源码、测试、配置、ledger 指针和允许保留的 `.env.example`。 |
| conditional include 路径数 | 145 | `docs/**`、`experiments/**`，需内容级 public-safe 审查。 |
| default exclude 路径数 | 135 | `.env`、运行记录、数据、报告、过程台账、checkpoint、缓存和本地工具路径。 |
| unclassified 路径数 | 0 | 所有源路径均已分类。 |

本次没有复制文件、初始化临时仓库、创建 commit / branch / tag、配置 remote、push、远端删除、history rewrite、force push 或读取 `.env`。

下一步若继续，需要新门禁批准 clean snapshot materialization 和内容级扫描；任何远端写入仍必须另行明示授权。

## Clean Snapshot Materialization / Content Scan 结果

用户随后回复“同意”，按下一步授权处理：仅允许本地 clean snapshot materialization 和内容级扫描，不授权远端写入。

已生成：

| 产物 | 路径 | 状态 |
|---|---|---|
| materialization manifest | `docs/release/CLEAN-SNAPSHOT-MATERIALIZATION-MANIFEST-CR085.yaml` | PASS_WITH_RISK |
| materialization scan evidence | `process/checks/CR085-CLEAN-SNAPSHOT-MATERIALIZATION-SCAN-2026-06-17.md` | PASS_WITH_RISK |

扫描结果摘要：

| 候选 | 文件数 | 结论 | 说明 |
|---|---:|---|---|
| source full | 436 | FAIL_TOPOLOGY | `docs/**` 等 conditional 内容触发 80 个 topology blocker。 |
| source v2 | 302 | FAIL_TOPOLOGY | 排除 docs 后仍有 14 个 blocker。 |
| source v3 clean overlay set | 288 | PASS_WITH_WARNINGS | blocker=0；仅作为 overlay 输入，不是完整远端树。 |
| remote master baseline | 136 | PASS_WITH_EXISTING_TOPOLOGY_RISK | 当前远端 README 和 `docs/USER-MANUAL.md` 已有 2 个 topology blocker。 |
| overlay A retain docs | 297 | PASS_WITH_EXISTING_TOPOLOGY_RISK | 保留 README / USER-MANUAL，也保留 2 个既有 blocker。 |
| overlay B strict remove docs | 295 | PASS_WITH_WARNINGS | blocker=0，但若用于远端发布会删除 README 和 `docs/USER-MANUAL.md`。 |

共同扫描结论：

- 禁止路径命中：0。
- 精确密钥命中：0。
- `.env.example`：29 行、19 个赋值，均为空或占位；未发现 realish value key。
- `.env`：未读取、未复制、未打印、未保存、未上传。
- 大文件：0 个超过 5 MiB；最大文件为 `engine/research_dataset.py`，336211 bytes。
- warning：4 个测试 fixture 警告，来自 `/tmp` 或私网 IP 示例；需下一门禁作为风险接受项确认。

当前推荐：不要直接 push。下一门禁必须让用户选择：

1. overlay A：保留远端 README / USER-MANUAL，并接受它们已有的 topology blocker；
2. overlay B：删除 README / USER-MANUAL，获得 blocker=0 的严格候选；
3. sanitize-docs：先修订 / 清理 README 和 USER-MANUAL，再重新扫描。

任何远端写入、远端删除、force、history rewrite、tag 或 default branch governance 仍未授权。

## Sanitized Docs Candidate / Remote Write Gate

用户随后回复“同意”，按 CP5 Remote Publication Readiness approve 处理：

- 选择 `sanitize-docs` 推荐方案。
- 接受 4 个测试 fixture warning 为非阻断风险。
- 继续不授权远端写入、push、remote deletion、force、history rewrite、tag、default branch governance、current dirty worktree commit 或 `.env` 读取。

已生成 sanitized docs v6 候选与扫描证据：

| 产物 | 路径 | 状态 |
|---|---|---|
| sanitized docs manifest | `docs/release/SANITIZED-DOCS-SNAPSHOT-MANIFEST-CR085.yaml` | PASS_WITH_RISK |
| sanitized docs scan evidence | `process/checks/CR085-SANITIZED-DOCS-SNAPSHOT-SCAN-2026-06-17.md` | PASS_WITH_RISK |
| remote write readiness auto check | `process/checks/CP5-CR085-REMOTE-WRITE-READINESS.md` | PASS |
| remote write readiness manual gate | `process/checkpoints/CP5-CR085-REMOTE-WRITE-READINESS.md` | approved |

v6 候选摘要：

| 指标 | 值 |
|---|---:|
| 候选路径 | `/tmp/cr085-remote-overlay-v6-sanitize-docs-20260617T143722` |
| 文件数 | 297 |
| forbidden path hits | 0 |
| precise secret hits | 0 |
| topology blockers | 0 |
| topology warnings | 4 |
| path list sha256 | `fc6436fad0621be7ff2f21c0a87c1c5252cb482fc7a28c1ec73dbe85afdb42fe` |
| content manifest sha256 | `7eb0ba516471fcd38c808176f2f621bd0d8f6cea7e32a864090da508363190e7` |

下一门禁 `CP5-CR085-REMOTE-WRITE-READINESS` 只在用户再次明确 `approve` 后，才授权 exact sequence：

1. 只读 remote head check，确认 `master` 仍为 `3ade165e8b1edad031a911490cf6c1cee942616e`。
2. 使用临时 clone / worktree，不使用当前 dirty worktree。
3. 应用 `/tmp/cr085-remote-overlay-v6-sanitize-docs-20260617T143722`。
4. 创建普通 clean publication commit。
5. 以普通 fast-forward push 更新 `refs/heads/master`。

Stop condition：remote head 不匹配、候选扫描不一致、push 非 fast-forward、发现 `.env` 或其他禁止路径时立即停止。

## Remote Write Execution / Post-push Verification

用户随后回复“同意”，按 CP5 Remote Write Readiness approve 处理；已执行 exact sequence。

| 产物 / 指标 | 值 |
|---|---|
| execution evidence | `process/checks/CR085-REMOTE-WRITE-EXECUTION-2026-06-17.md` |
| CP8 auto check | `process/checks/CP8-CR085-DELIVERY-READINESS.md` |
| CP8 manual checkpoint | `process/checkpoints/CP8-CR085-DELIVERY-READINESS.md` |
| remote | `git@github.com:hyde-zhao/quant-lab.git` |
| branch | `refs/heads/master` |
| previous remote master | `3ade165e8b1edad031a911490cf6c1cee942616e` |
| new remote master | `57d4fc1e9b8808b4371e12af72f9cddcba01aaa0` |
| temporary clone | `/tmp/cr085-remote-write-20260617T150133` |
| push mode | ordinary fast-forward |

已验证：

- push 后 `git ls-remote` 显示 remote master 为 `57d4fc1e9b8808b4371e12af72f9cddcba01aaa0`。
- commit parent 为 `3ade165e8b1edad031a911490cf6c1cee942616e`。
- v6 candidate path/content hash 与 manifest 一致。
- `.env` 未读取、未复制、未上传；`.env.example` 保留为模板。
- 未执行 force push、history rewrite、tag、default branch governance、remote deletion、current dirty worktree commit、rollback/revert、data/reports 内容读取、NAS/provider/lake/catalog/runtime、CR046 recovery、old root retirement 或 cleanup。

## CP8 关闭结果

用户于 `2026-06-17T15:19:06+08:00` 回复“同意”，接受 `DQ-CP8-CR085-01`、`DQ-CP8-CR085-02`、`DQ-CP8-CR085-03` 推荐方案，关闭 CR085 当前交付为 `closed-current-delivery / READY_WITH_RISK`。

关闭范围：

- 远端 master 已通过已批准的普通 fast-forward clean publication push 更新到 `57d4fc1e9b8808b4371e12af72f9cddcba01aaa0`。
- `.env.example` 保留为模板。
- `.env` 未读取、未复制、未上传。
- 接受 4 个测试 fixture warning 与未做 Git history purge 作为残余风险。

继续不授权：

- force push、history rewrite、tag、default branch governance、remote deletion。
- `.env` 读取、打印、保存、复制或上传。
- current dirty worktree commit、rollback/revert。
- data/reports 内容读取、NAS/provider/lake/catalog/runtime、CR046 recovery、old root retirement 或 cleanup。

后续若需要清理 Git 历史、回滚 / revert、治理默认分支或执行额外远端写动作，必须另起独立 CR / 人工门禁。

## 处理结论

- 审批结论：approved
- 当前结论：CR085 CP2/CP3/CP5、Remote Write gate 和 CP8 均已批准并执行 / 关闭；远端 master 已更新到 `57d4fc1e9b8808b4371e12af72f9cddcba01aaa0`；`.env.example` 保留，`.env` 禁止上传且未读取。
- 阻断项：无远程同步阻断项；当前交付关闭为 `READY_WITH_RISK`。
- 下一步：CR085 已关闭；如需 Git history purge、force、tag、default branch governance、remote deletion、额外 push、rollback/revert、`.env` 处置、data/reports/NAS/runtime、CR046 recovery、old root retirement 或 cleanup，必须另起独立 CR / 人工门禁。

## 不授权项

- 不授权额外 `git remote add`、`git remote set-url`、`git remote remove`。
- 不授权 `git fetch` 到当前仓库。
- 不授权额外 `git push`、`git push -u`、force push、tag、default branch governance、history rewrite、remote deletion、rollback/revert。
- 不授权读取、打印或保存 token、password、private key、cookie、session。
- 不授权上传、读取、打印或保存 `.env`。
- 不授权 data/reports 内容读取、NAS 内容操作、provider/lake/catalog、QMT/MiniQMT runtime、CR046 recovery、old root retirement 或 cleanup。
