---
status: repository-push-authorized
version: "1.2"
cr_ref: "CR-052"
scenario_baseline: "CR-052-VNEXT-UJ-R3.2.1-confirmed"
engagement_mode: meta-self-dev
owner: host-orchestrator
updated_at: "2026-07-19"
authorization_boundary: "local reversible implementation only; migration, remote publication, runtime/production writes and destructive Git remain separately gated"
---

# CR-052 Meta Flow vNext 精简治理 HLD 与实施计划

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.2 | 2026-07-19 | host-orchestrator | 用户在本地验收通过后明确授权将 meta-flow 当前 CR 分支和 meta-flow-artifacts 当前 `main` 的整改成果普通 commit/push 到各自远端；该授权不包含 tag 推送、真实过程仓迁移、软链接切换、production/runtime 发布或破坏性 Git。 |
| 1.1 | 2026-07-19 | host-orchestrator | 完成 Wave 1-4 本地实现与验收；补齐空目录发布仓初始化和 Work 关闭时 Phase 最小投影，新增严格 schema/authorization/provenance 负例，记录 `157` 项 vNext、`220` 项相邻兼容和全仓 `854 passed + 70 subtests` 证据；明确真实迁移、commit/push 和发布仍未授权。 |
| 1.0 | 2026-07-19 | host-orchestrator | 将已确认的 `UC-VNEXT-UJ-001..016` 收敛为每项目双仓、弹性 Project/Phase/Work、G0/G1/G2、Work scope/budget、独立安全发布、复盘与有界自进化的最小架构；合并 HLD、ADR、Feature 设计矩阵和实施 Wave，避免重复文档。 |

## 1. 结果先行

本 CR 不再扩建共享 artifact worktree、双 leg 或 aggregate。新默认路径是：

1. 一个项目复用一个发布库，并拥有一个独立过程库；不同项目不共享 working tree、index 或 branch。
2. 发布库中的 `process` 只是指向本项目过程库的相对软链接。
3. 过程库用小型 `PROJECT.yaml`、可选 `ROADMAP.yaml`、可选 `phases/<phase-id>/PHASE.yaml` 和 `works/<work-id>/WORK.yaml` 管理长期与日常状态。
4. 每项变化先成为普通 Work；命中重大变化条件时升级正式 CR。两者共用同一套风险、scope、预算和恢复内核。
5. G0/G1 使用固定上限，G2 必须显式给出预算并经过人工门；任何档位都不是无限预算。
6. 读取、写入和检查只允许发生在当前 Work 声明的范围内；扩读或扩写先重新分类。
7. 发布库与过程库独立 commit/push，准确披露部分成功；不声称跨仓原子性。
8. 项目复盘只生成事实和改进候选；自进化必须经过独立审议并作为新的有界 Work/CR 实施。

## 2. 问题、目标与量化成功标准

### 2.1 问题

旧方案把多个项目放在同一个过程仓 working tree 中，任一项目切分支会改变其他项目看到的过程文档；同时固定全量产品文档、CP、评审和检查让普通工作承担与风险不相称的读写、检查和 token 成本。继续为旧共享布局增加 worktree、双 leg 和聚合状态，只会把治理痛点转化为更多状态机痛点。

### 2.2 目标

| ID | 目标 | 可验收阈值 |
|---|---|---|
| SC-01 | 项目物理隔离 | 在两个并列项目 fixture 中切换 A 的分支后，B 的 HEAD、index、working tree 和过程文件 hash 变化数均为 `0` |
| SC-02 | 唯一路由 | 每个发布库解析出的可写过程根恰好为 `1`；缺失、歧义、跨项目或断链均 fail closed |
| SC-03 | 轻量长期治理 | `PROJECT.yaml` 必需字段不超过 `8` 个；Roadmap 和 Phase 均可选；最小项目只需 Project + Work |
| SC-04 | 可解释分级 | 已定义的重大触发项命中 G2/CR 的覆盖率为 `100%`；未知高风险不得降为 G0/G1 |
| SC-05 | 有界执行 | G0 上限 `8 reads / 8 writes / 3 check groups / 32k token`；G1 上限 `20 / 24 / 8 / 96k`；G2 无显式预算时不可执行 |
| SC-06 | scope 强制 | 越界读、越界写和未声明检查在单元测试与 CLI check 中阻断率为 `100%` |
| SC-07 | 最小查询 | 默认项目状态查询读取直接引用对象不超过 `5` 个、跨项目读取数为 `0` |
| SC-08 | 安全独立发布 | 两仓各自输出五态；任一侧失败时不得报告整体 PASS，自动 force/merge/rebase/tag/回滚次数为 `0` |
| SC-09 | 证据化复盘 | 报告固定覆盖价值、规范/证据、质量/恢复、流动效率、token/context、流程适配 `6` 个维度，并区分 measured/proxy/unavailable |
| SC-10 | 有界自进化 | 复盘报告直接触发源码修改、commit、push 或下一轮自进化的次数为 `0` |

### 2.3 非目标

- 本 CR 本地实现阶段不迁移真实项目、不重挂当前 `process`、不创建远端仓、不 commit/push。
- 不重写旧共享过程仓历史，不强制转换旧 CP/CR/Story。
- 不保留双 leg、aggregate、project integration branch 作为新默认路径。
- 不实现分布式多写者自动 merge，也不把目录隔离误称为并发写安全。
- 不取消所有人工门；公共契约/状态机、真实迁移、远端/生产写入仍需人工确认。
- 不把 token 降低作为唯一优化指标；返工、缺陷、证据覆盖和恢复能力必须同时保持或改善。

## 3. 架构决策

| ADR | 主选 | 备选 | 选择理由 | 切换条件 |
|---|---|---|---|---|
| ADR-052-01 仓库边界 | 每项目一个发布库 + 一个独立过程库 | 共享过程仓 + per-project worktree | 前者直接消除跨项目分支联动，状态最少，Git 心智模型最简单 | 只有出现被证明必须跨项目原子提交的真实业务要求时重新评估 |
| ADR-052-02 过程入口 | 发布库 `process` 使用相对软链接指向 sibling 过程库 | 配置文件绝对路径、复制目录 | 相对链接可移植、用户可见、无需第三套挂载运行时 | 不支持 symlink 的平台使用显式 project binding 配置，不能静默复制 |
| ADR-052-03 长期层级 | `Project -> [Roadmap] -> [Phase] -> Work` 弹性层级 | 强制四层齐全 | 大项目保留路标，小项目不生成空治理对象 | 若监管要求固定层级，则通过项目策略显式启用，不改变全局默认 |
| ADR-052-04 工作容器 | Work 默认；重大触发项升级 CR；共用内核 | 所有变化均 CR | 日常工作走短路，高风险仍保留正式决策边界 | 若某项目强审计，项目策略可将所有 Work 升级 CR/G2 |
| ADR-052-05 成本控制 | risk profile + scope + budget 三者同时强制 | 只设 token 上限 | 仅控 token 会诱发漏读/漏验；三者共同约束才可解释 | telemetry 不可用时保留 read/write/check 硬限制并标记 unavailable |
| ADR-052-06 Git 发布 | 两仓独立计划、独立结果、expected OID/FF-only | 双 leg/aggregate、伪原子回滚 | 减少隐藏状态并真实披露部分成功 | 只有外部事务协调器具备可验证原子语义时才引入新机制 |
| ADR-052-07 兼容策略 | 旧模块只读兼容/显式 legacy 命令；新 CLI 走 vNext | 在旧 artifact lifecycle 内继续打补丁 | 避免新旧状态互相污染，允许渐进替换 | 新路径完成两个试点后再决定删除时间，不在本 CR 删除历史代码 |
| ADR-052-08 自进化 | 复盘、建议批准、实现验证分成三个状态 | 报告自动改代码 | 防止授权偷渡和递归自修改 | 不切换；这是安全不变量 |

## 4. 领域模型与文件布局

### 4.1 过程库最小布局

```text
<project>-process/
├── PROJECT.yaml
├── ROADMAP.yaml                  # 可选
├── phases/
│   └── PH-001/PHASE.yaml         # 可选
├── works/
│   └── W-001/
│       ├── WORK.yaml             # 机器真相与短摘要
│       ├── REQUEST.md            # 用户确认后的几段话，可选引用外部需求
│       ├── HANDOFF.yaml          # 只有暂停/交接时生成；机器可校验的最小恢复快照
│       └── RETROSPECTIVE.md      # 只有该 Work 被选作复盘范围时生成
├── retrospectives/
│   └── RETRO-*.yaml
└── evolution/
    └── EVO-*.yaml
```

发布库只需要相对 `process` 软链接以及可选的轻量 binding 文件；业务代码、发布文档和构建产物继续遵循项目原有布局。

### 4.2 对象契约

| 对象 | 关键字段 | 不允许承载 |
|---|---|---|
| Project | schema_version、project_id、name、objective、status、current_phase_ref、active_work_refs、updated_at | 完整历史、完整 Work、凭据、绝对设备路径 |
| Roadmap | project_id、outcome、status、phase_refs、updated_at | Work 执行日志、diff、测试正文 |
| Phase | phase_id、project_id、objective、status、work_refs、result_refs、updated_at | 其他 Phase 的副本、完整 Work 正文 |
| Work | work_id、container_kind、objective、status、risk、scope、budget、usage、phase_ref、result_refs、recovery、updated_at | transcript、全项目文件列表、凭据、隐式授权 |
| Retrospective | scope、evidence_refs、六维结论、measurement_quality、improvement_candidates、approval | 自动执行授权 |
| Evolution Package | source_retro_ref、objective、scope、risk、budget、acceptance、canary、rollback、approval | commit/push/production 授权、递归触发器 |

所有 ref 必须相对过程库根解析，禁止绝对路径和 `..`；所有 ID 必须在单项目内唯一。

## 5. 分级、范围与预算

### 5.1 Work/CR 判定

以下任一项为 `true` 或 `unknown` 时，不得保持普通低风险 Work：公共 API/共享契约、架构边界、安全、权限、不可逆迁移、生产写、正式发布/部署、强审计、风险接受、跨阶段重构。明确为 `true` 时容器为 CR；为 `unknown` 时推荐 CR/G2 并阻断执行，直到人工补齐事实。

文件数量本身不决定 CR。没有重大触发项时，局部日常交付保留 Work。

### 5.2 G profile

| 档位 | 判定 | 默认评审 | 默认验证 | 人工门 |
|---|---|---|---|---|
| G0 | 单个可逆文档/配置/局部机械改动，无接口和高风险 | 无独立设计评审 | 目标测试/静态检查 + diff/status | 无；越界即升级 |
| G1 | 多文件/多步骤/内部接口或复杂失败路径，但没有 G2 触发项 | 最多一次 Work 范围轻量评审 | 目标测试 + 必要构建 + Work 定向检查 | 仅新决策/越界时 |
| G2 | 正式 CR 或命中重大触发项 | HLD/ADR + 人工设计门 | 批准的全量/独立验证 | 设计、风险/授权、发布按需门控 |

### 5.3 预算语义

- `reads`、`writes` 是文件/对象访问事件，不按 token 调用次数猜测；同一对象重复读取也累计，以暴露重复扩读。
- `check_groups` 是解决一个具体风险的一组命令，禁止拆命令规避上限。
- token 必须记录 `measured | proxy | unavailable`；unavailable 不等于 0，也不解除其他上限。
- 达到任一硬上限前必须返回 `NEEDS_RECLASSIFICATION`；不得先执行后补预算。
- G2 必须显式填写四个上限，缺任一项即 `BLOCKED`。

## 6. 模块与集成契约

| 模块 | 调用方/时机 | 输入 | 输出 | 失败/降级 |
|---|---|---|---|---|
| `meta_flow.project.vnext` | `project init/check/status`；项目接入和查询前 | project/process root、Project/Roadmap/Phase payload | 已校验对象、唯一 refs、诊断 | 路由或 ID 冲突 fail closed；不自动重建 |
| `meta_flow.work.model` | 所有 Work 命令和后续 agent | WORK payload | typed Work snapshot/findings | unknown key、非法 ref、超大小直接 ERROR |
| `meta_flow.work.classifier` | REQUEST 确认后、scope 变化后 | 可枚举风险事实、变更形态 | Work/CR、G 档、理由、阻断项 | 重大事实 unknown 时 G2 + blocked |
| `meta_flow.work.scope` | 读/写/检查前 | allowed patterns、requested path/check | allow/deny finding | deny 时不执行；无隐式扩权 |
| `meta_flow.work.budget` | 每次访问事件和恢复前 | profile limits、current usage、delta | OK/NEAR_LIMIT/EXCEEDED | 超限返回重分类/拆 Work 建议 |
| `meta_flow.work.store` | `work init/check/status/pause/close` | process root、Work payload/transition | 原子写入结果、短摘要 | expected status/OID 不符时不写 |
| `meta_flow.workspace.project_pair` | 新建/接入双仓 | 两仓路径、身份、期望状态 | 默认 dry-run plan、route health | 非空/已有链接/跨 repo/dirty 冲突阻断；apply 单独显式启用 |
| `meta_flow.workspace.publication` | 用户明确 commit/push 时 | per-repo plan、allowlist、expected OID | per-repo terminal status | 保留部分成功；不自动 merge/回滚 |
| `meta_flow.retrospective` | Phase/Project/切片结束 | 索引、evidence/usage/token 摘要 | 六维报告和候选 | 证据缺口标注，不扩读冒充事实 |
| `meta_flow.evolution` | 人工审议和试点 | 已确认报告、单项候选 | 未执行的 package / validation result | 未批准、过期、基线漂移即阻断 |

调用方必须传显式 `project_root/process_root`，不得通过扫描 sibling 目录猜项目；vNext 模块不调用旧双 leg/aggregate 生命周期。

## 7. CLI 主路径

```text
meta-flow project init ...                 # 默认 dry-run；--apply 仅本地初始化
meta-flow project check ...
meta-flow project status ...
meta-flow work classify ...
meta-flow work init ...                    # 默认 dry-run；--apply 写一个 Work packet
meta-flow work check --work W-001 ...
meta-flow work status --work W-001 ...
meta-flow work pause|resume|close ...
meta-flow retrospective build|check ...
meta-flow evolution package|check ...
```

远端 push 不放进 `work close` 或项目初始化；继续使用独立 publication 命令，并在该命令上保留 typed authorization。旧 `workspace bootstrap/push` 明确标记 legacy，不再由新项目向导推荐。

## 8. 前置校验与失败路径

| 阶段 | 前置校验 | 失败行为 | 可恢复入口 |
|---|---|---|---|
| 项目初始化 | 路径解析、Git root、项目 ID、两仓不相同、目标链接、非空目录 | dry-run 报 conflict，不修改文件 | 修改路径/复用策略后重建 plan |
| Work 创建 | Project 唯一、REQUEST 已确认、风险事实完整、scope/path 合法、预算完整 | blocked，不生成 active Work | 补事实、缩 scope、升级档位 |
| 执行 | status=active、requested operation 在 scope 内、预算未超、OID 未漂移 | 停止当前操作并记 finding | 重分类、拆 Work、恢复摘要 |
| 评审/验证 | 对象存在、changed paths 未越界、检查映射具体风险 | 不执行范围外检查 | 修改 plan 后重新确认 |
| commit/push | 验证 PASS、allowlist、index/dirty、expected remote OID、FF-only | 该仓 terminal failed/blocked；不触碰另一仓既有成功 | 重新观察后只重试失败仓 |
| 关闭 | 结果和剩余风险明确、推送状态已知、长期投影目标唯一 | 不写 completed | blocked/cancelled/后续 Work |
| 复盘/进化 | 窗口冻结、证据质量声明、人工批准语义分离 | 报告保留缺口；package 不执行 | 缩小范围、补证据、重新审议 |

## 9. Feature / Story / Wave 设计矩阵

| Feature | 覆盖场景 | 实现对象 | 设计深度 | Wave |
|---|---|---|---|---|
| F-01 轻量治理内核 | UJ-003/011/013 | Project/Roadmap/Phase schema、store、check/status | full | 1 |
| F-02 Work 内核 | UJ-004..009/011..013 | model、classifier、scope、budget、lifecycle、CLI | full | 1 |
| F-03 双仓初始化/接入 | UJ-001/002 | project pair plan、route、relative link、snapshot manifest | full | 2 |
| F-04 独立发布/恢复 | UJ-010/012 | per-repo plan/result、OID/FF preflight、resume | full | 3 |
| F-05 最小查询 | UJ-007/013 | compact status/handoff、bounded resolver | technical-note | 3 |
| F-06 复盘 | UJ-014 | telemetry adapter、六维 report、checker | full | 4 |
| F-07 有界进化 | UJ-015/016 | package、approval separation、canary/result/rollback | full | 4 |
| F-08 契约与文档切换 | 全部 | CLI help、README、USER-MANUAL、Agent/Skill slim contract、legacy 标记 | full | 4 |

实施顺序：

1. **Wave 1**：先交付纯本地、纯文件契约内核；旧模块不删除。
2. **Wave 2**：交付项目双仓 dry-run 和临时 fixture apply；不得触碰真实项目。
3. **Wave 3**：交付本地 bare remote fixture 下的独立 publication/部分成功恢复；真实 remote 仍禁用。
4. **Wave 4**：交付复盘、自进化 package 和文档契约；再做 meta-flow dogfood 与第二个临时试点。

每个 Wave 都先定向测试，再跑相邻回归；只有 Wave 4 结束才跑全仓回归与 guardrail。

## 10. 验证矩阵

| 验证层 | 必测内容 | 通过条件 |
|---|---|---|
| 单元 | schema、ref/path、分类规则、预算边界、状态转移、报告质量标签 | 正/负/边界测试全 PASS |
| 契约 | CLI JSON/文本稳定字段、退出码、dry-run mutation=0 | 每个命令至少 1 正向 + 1 fail-closed 测试 |
| 集成 | 两并列项目、两独立 Git 仓、相对链接、branch switch 隔离 | SC-01/02 全部满足 |
| Git fixture | local bare remote、stale OID、FF-only、partial success/resume | 无 force/merge/rebase/tag；状态不误报 |
| 兼容 | 现有 project/token/workspace 定向回归 | 基线 `35 passed, 11 subtests` 不退化 |
| 治理 | delivery guardrail、CLI help、文档路径和 legacy 提示 | checker 全 PASS，无旧默认示例残留 |
| dogfood | meta-flow + 临时第二项目各两个 Work 周期 | scope/token/恢复/关闭/查询数据完整；真实迁移前只做临时路径 |

## 11. 风险与回退

| 风险 | 应对 | 回退 |
|---|---|---|
| 新旧 CLI/对象混用 | 新命令使用独立 vNext schema；legacy 明示；不自动转换 | 关闭 vNext 命令入口，旧模块保持不变 |
| YAML 解析能力不足 | 首版只使用当前 parser 可稳定处理的标量、列表、对象；checker 拒绝未知结构 | 回退 JSON 机器真相 + YAML 人类视图，不引入重量依赖 |
| G0/G1 误判 | 重大触发项 allowlist + unknown fail closed + 用户可升级 | 项目策略整体提高一档 |
| token telemetry 不准 | 记录 measured/proxy/unavailable，读写检查硬限独立生效 | 只报告 proxy，不据此自动降档 |
| 过程仓 main 并发冲突 | 单写 publisher + expected OID；首版不自动 merge | 停止、重新观察、人工串行处理 |
| 简化导致证据不足 | 高风险仍 G2；每个 Work 保留最小 request/result/ref | 升档并补充当前 Work 范围证据 |
| 自进化递归或越权 | 报告、package、执行三阶段；批准语义分离 | package 标记 rejected/expired，源码不变 |

## 12. Gotchas

1. “两个目录”不等于两个仓库；必须验证 Git common dir、index、HEAD 和 worktree 身份均独立。
2. 相对软链接的基准是链接所在目录，不是命令执行目录；生成后必须解析并反向校验目标。
3. 过程库 main-only 不解决多人并发写；仍要 expected OID 和单写发布者。
4. `allowed_reads` 是允许上限，不是默认必读集合；查询应从最小 refs 开始。
5. 预算剩余不能授权越界，scope 允许也不能绕过风险门；risk、scope、budget 三个条件缺一不可。
6. 常规 push 可继承 G0/G1，但“已预授权仓/ref”必须有可验证策略；新 remote、protected branch 和 production publication 仍是 G2。
7. 复盘中的流程符合度和流程适配性是两个结论：完全遵循错误流程仍可能得到低适配评分。
8. `unavailable` 不能记作零 token 或零耗时，避免错误地把不可观测阶段判成高效。
9. 旧 artifact worktree/dual-leg 代码暂不删除，因为历史 CR 仍需审计；但任何新文档和 CLI 示例不得把它作为默认推荐。

## 13. 当前人工门状态

用户已确认 16 个场景，并授权按本计划开始本地实现。以下默认选择属于场景的直接工程化，不再单独阻塞：新模块边界、vNext 文件名、CLI 子命令、临时 fixture 和定向测试。

以下动作仍必须再次请求人工确认：

1. 改变本文件已冻结的公共风险/预算语义或主状态机；
2. 对当前 meta-flow 或任何真实项目执行过程仓迁移/软链接切换；
3. commit、push、tag、远端仓创建、受保护分支或外部 publication；
4. runtime/production/权限/凭据相关写入；
5. force/reset/rebase/orphan、历史重写、自动冲突解决或删除旧证据。

## 14. 本地实施与验证结果

### 14.1 已实现

- `project`：复用现有发布仓或在不存在/空目录初始化本地发布仓、独立 sibling 过程仓 dry-run/apply、相对 `process` 链接、唯一健康路由、snapshot-only 接入、弹性 Roadmap/Phase 和最多 5 个直接对象查询。
- `work`：Work/CR 与 G0/G1/G2 分类、deny-default scope、固定/显式预算、最小状态机、usage、评审/验证计划、暂停/阻塞 handoff、OID/scope 恢复检查，以及创建/关闭时对 Project/Phase 的最小索引投影。
- `repository`：单仓 allowlist commit、exact observed OID + fast-forward-only push、typed authorization、部分成功/失败 receipt，禁止自动回滚、force、merge、rebase 和 tag。
- `retrospective/evolution`：六维复盘、measured/proxy/unavailable、事实确认、建议逐项决策、有界进化包、独立实现启动授权、验收/回归/恢复/canary 结果；每一层均不能授权下一层。
- CLI、README、USER-MANUAL 和安装规则将 vNext 标为新默认，旧 shared artifact/workspace/dual-leg/aggregate 明示为 legacy。

### 14.2 验证证据

| 检查 | 结果 |
|---|---|
| vNext 新增测试 `tests/test_vnext_*.py` | `157 passed` |
| workspace/artifact/CR 相邻兼容回归 | `220 passed` |
| 全仓测试 | `854 passed, 70 subtests passed` |
| `ruff check meta_flow tests` | `PASS` |
| `scripts/check_delivery_guardrails.py` | `PASS`；仅报告被忽略的本地 `__pycache__` 提示 |
| 顶层及 `project/work/repository/retrospective/evolution --help` | `PASS` |
| `git diff --check` | `PASS` |

### 14.3 本轮未执行

本地验收完成时，meta-flow 的真实 `process` 仍指向 legacy artifact 路径，且尚未 commit/push。随后用户已单独授权把当前 meta-flow CR 分支和 meta-flow-artifacts `main` 的现有整改成果普通 commit/push 到各自远端；执行结果以 Git 远端 OID 和会话最终报告为准。该授权仍不包含真实过程仓迁移、软链接切换、tag 推送、production/runtime 写入、force/reset/rebase/orphan、自动 merge 或历史重写。基线 tag `pre-vnext-simplification-20260719` 继续指向实施前源码提交。
