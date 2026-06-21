请审查：`process/checkpoints/CP5-CR-034-LLD-BATCH.md`

自动预检结论：PASS

Context Capsule：`process/context/CP5-CR-034-LLD-CONTEXT.yaml`，状态 `ready`，read_profile=`compact`。默认读取策略是先读 capsule；仅在缺失、冲突、字段不足、人工审计或深度评审时读取完整文档。

决策收集覆盖：已扫描 6 个来源，发现候选问题 11 个，纳入待决策 5 个；N/A / 缺失来源 2 个，原因见 checkpoint 的 Decision Collection Coverage。

本轮待人工决策项：5

如果你回复 approve，表示你接受以下 5 项推荐方案，并允许 CR-034 进入实现阶段；不表示授权 quant-lab 写入、CR-033、凭据、runtime、SaaS、production write、真实交易或真实发布。

待人工决策清单：

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| CP5-CR034-DQ-01 | implementation | adoption readiness CLI 入口命名采用哪种形式 | 使用 `meta-flow doctor adoption --project-root .` | 备选 A：新增 `meta-flow adoption check`；备选 B：只写文档命令链 | 推荐方案与现有 doctor 体系一致；备选 A 更独立但增加顶层命令；备选 B 实现少但无法形成统一报告 | 影响 CLI、README、USER-MANUAL、测试 | 若 doctor 参数解析冲突，切换到 `meta-flow adoption check` |
| CP5-CR034-DQ-02 | implementation | workspace/state/ledger bootstrap 采用单命令还是分步命令 | 新增 `meta-flow workspace bootstrap`，同时保留底层 `workspace link` 和 `state init/migrate-v2` | 备选 A：只新增 `state init`；备选 B：继续手工初始化 | 推荐方案减少漏步；备选 A 仍需串命令；备选 B 改动少但保留半初始化风险 | 影响 workspace routing、state v2、ledger scaffold、文档和测试 | 若实现复杂度超出本 CR，先落地 `state init` + 文档命令链 |
| CP5-CR034-DQ-03 | security | package identity / delivery routing scan 是否自动写入目标项目 STATE | 默认只读扫描，不自动写入 production 目标项目 STATE | 备选 A：scan 后自动写入 STATE.current；备选 B：只输出 stdout | 推荐方案保守；备选 A 自动化高但越权风险大；备选 B 安全但恢复弱 | 影响 production 交付路径、安全边界和人工决策 | 用户在目标项目明确授权后，可增加 `--write` |
| CP5-CR034-DQ-04 | implementation | bootstrap CR index 主写 YAML 还是 v2 JSON | 当前 legacy 项目继续同步 `CR-INDEX.yaml`，新 state v2 bootstrap 支持 v2 JSON；adoption doctor 同时识别两者 | 备选 A：只支持 YAML；备选 B：强制新项目只支持 JSON | 推荐方案兼容当前和未来；备选 A 简单但延迟 v2；备选 B 干净但破坏 legacy | 影响 CR tracking、state v2、summary、ledger | 若双写复杂，先 YAML 为主并记录 v2 JSON 后续项 |
| CP5-CR034-DQ-05 | follow_up_tracking | 当前未提交 quality governance / doctor 相关变更是否纳入 CR-034 | 全部纳入 CR-034 S05 作为 adoption readiness 质量治理前置能力 | 备选 A：拆成独立 CR；备选 B：回滚这些变更 | 推荐方案减少未归属 diff；备选 A 更干净但延迟 adoption；备选 B 可能丢失已有工作 | 影响当前源码 diff、测试范围和发布说明 | 若审查发现无关内容，拆分无关子集为后续 CR |

不授权项：
- 不进入或写入 `quant-lab`。
- 不创建 `quant-lab` 的 CR、checkpoint、handoff、result、context 或 process 产物。
- 不启动 `CR-033`。
- 不访问凭据、账户、NAS、MiniQMT/QMT、xtquant、交易网关或外部 SaaS。
- 不执行 trace upload、publish、live、production write、真实交易或真实发布。

该文件包含本检查点的 Entry Criteria、Checklist、Exit Criteria、Deliverables、自动预检摘要、Decision Brief、待人工决策清单和人工审查结果区。

回复 `approve` 表示接受上表全部推荐方案；如需调整，请用 `修改: <具体修改点>` 指明决策 ID 和修改内容。

请直接回复以下任一整行：

approve
修改: <具体修改点>
reject
