请审查人工门禁 `CP2-CR034-REQUIREMENT-BASELINE`。

checklist 路径: `process/checkpoints/CP2-CR034-REQUIREMENT-BASELINE.md`
自动预检结论: 已生成 Decision Brief；发起前请以 `meta-flow check human-gate --checkpoint process/checkpoints/CP2-CR034-REQUIREMENT-BASELINE.md` 的结果为准。

审批者摘要:
- 本次确认服务的整体目标: 请见 checkpoint `### 审批者摘要`；本消息只承载发起确认所需摘要。
- 推荐动作: 默认推荐 approve，除非你要求调整范围、风险、授权边界或推荐方案。
- approve 后会发生什么: 接受本轮 Decision Brief 中列出的推荐方案，并允许进入 checkpoint 声明的下一阶段。
- approve 不授权什么: 不授权真实运行、凭据读取、安全边界变更、外部接口调用、数据写入、publish、live / 交易类操作。
- 不确认会阻塞什么: 当前 checkpoint 后续推进会保持等待，直到你回复 approve / 修改 / reject。

Context Capsule: 请见 checkpoint 的 `### Context Capsule Summary`，其中包含 capsule、read_profile、默认读取策略和全文档读取边界。
决策收集覆盖: 请见 checkpoint 的 `### Decision Collection Coverage`，本消息只承载发起确认所需摘要。
决策分层:
- 必须用户决策: 3
- 高风险策略确认: 以 checkpoint `### 决策分层` 为准。
- agent 默认处理: 低风险、可逆、局部实现细节默认不进入用户主确认表，完整清单以 checkpoint 为准。
- 仅审计记录: 已按项目规则处理且无需用户拍板的事项只保留审计摘要。
本轮待人工决策项: 3
blocking / high-risk 决策摘要: DQ-034-01, DQ-034-02, DQ-034-03

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-034-01 | implementation | 缺口 2 环境文件路径：topology.yaml/devices.yaml 归档位置 + .gitignore + --env-file 传参 | 选项 A：workspace 根 `topology.yaml`/`devices.yaml`（.gitignore 忽略实际文件）+ `templates/*.example` 入库 + `--env-file topology.yaml`；模板重命名 `topology-link3.yaml.example`->`topology.yaml.example | B：`env/` 专用目录；C：保留 `input/` 命名仅统一引用 | A 最简且与 `--cases-dir cases/` 同级直观；B 多一层目录；C 命名语义不清且 .gitignore `.input/` 与 `input/` 不匹配需修正 | 影响 SKILL.md L39/45/51 + ptm-te.md 工作目录 + .gitignore + 模板重命名；低风险 |
| DQ-034-02 | implementation | 缺口 2 devices.yaml.example schema 方向（device_groups vs 顶层 firewall/tg） | 选项 A：case-execution 新建专用 `skills/case-execution/templates/devices.yaml.example`（顶层 `firewall`+`tg`，对齐 resolve_addresses L505-520）；device-management 保留 device_groups | B：改 device-management example 为顶层；C：保留 device_groups + 文档说明转换 | A 不破坏 device-management 多设备组设计，case-execution 模板与实现一致；B 破坏 device-management；C example 仍误导 | 影响 新增 case-execution/devices.yaml.example + SKILL.md 引用 + ptm-te.md；低风险 |
| DQ-034-03 | scope | 24 用例 md 是否同步四态/retry/known_issue 标注 | 不扩范围：仅同步规范文档，用例标注属 FU-01 runtime 验证范畴 | 扩范围同步 24 用例标注 | 推荐不扩范围符合 R-C-008 + 范围控制；备选扩范围违背用例不入库 + 无 runtime 验证标注正确性 | 无（确认性记录）；风险：用例标注滞后但规范文档已就位 |

如果你回复 approve，表示你接受以上 3 项推荐方案，不表示授权以下不授权项。
不授权项: 不授权真实运行、凭据读取、安全边界变更、外部接口调用、数据写入、publish、live / 交易类操作；若 checkpoint 中列有额外不授权项，以 checkpoint 为准。

请只回复以下三个 exact 选项之一：
- approve
- 修改: <具体修改点>
- reject
