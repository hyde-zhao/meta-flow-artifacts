请审查人工门禁 `CP2-CR033-REQUIREMENT-BASELINE`。

checklist 路径: `process/checkpoints/CP2-CR033-REQUIREMENT-BASELINE.md`
自动预检结论: 已生成 Decision Brief；发起前请以 `meta-flow check human-gate --checkpoint process/checkpoints/CP2-CR033-REQUIREMENT-BASELINE.md` 的结果为准。

审批者摘要:
- 本次确认服务的整体目标: 请见 checkpoint `### 审批者摘要`；本消息只承载发起确认所需摘要。
- 推荐动作: 默认推荐 approve，除非你要求调整范围、风险、授权边界或推荐方案。
- approve 后会发生什么: 接受本轮 Decision Brief 中列出的推荐方案，并允许进入 checkpoint 声明的下一阶段。
- approve 不授权什么: 不授权真实运行、凭据读取、安全边界变更、外部接口调用、数据写入、publish、live / 交易类操作。
- 不确认会阻塞什么: 当前 checkpoint 后续推进会保持等待，直到你回复 approve / 修改 / reject。

Context Capsule: 请见 checkpoint 的 `### Context Capsule Summary`，其中包含 capsule、read_profile、默认读取策略和全文档读取边界。
决策收集覆盖: 请见 checkpoint 的 `### Decision Collection Coverage`，本消息只承载发起确认所需摘要。
决策分层:
- 必须用户决策: 6
- 高风险策略确认: 以 checkpoint `### 决策分层` 为准。
- agent 默认处理: 低风险、可逆、局部实现细节默认不进入用户主确认表，完整清单以 checkpoint 为准。
- 仅审计记录: 已按项目规则处理且无需用户拍板的事项只保留审计摘要。
本轮待人工决策项: 6
blocking / high-risk 决策摘要: DQ-01, DQ-02, DQ-03, DQ-04, DQ-05, DQ-06

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-01 | runtime_authorization | case_runner --execute 模式下真实设备写操作的 CP7 验证方式 | CP7 用 static review + dry-run 替代 runtime | 等 VALIDATION-ENV.yaml 授权后执行 runtime | 推荐 static review 不触发写操作安全；备选 runtime 更充分但需设备授权 | 影响 CP7 验证层；风险 runtime 路径未覆盖 |
| DQ-02 | implementation | fw_logout op 在 ptm-atomic 安装版是否暴露 | 安装前 ptm-atomic show fw_logout 验证；未暴露降级清理 session 文件 | 强制 ptm-atomic 升级暴露 fw_logout | 推荐降级保证可用性；备选升级超 CR-033 范围 | 影响 fw_logout op 完整度；风险登出不彻底 |
| DQ-03 | scope | 24 用例 md 的 known_issue 标注是否纳入 CR-033 | 纳入 R-F-021 ARP 整改时同步检查 known_issue 标注 | 单独 CR 处理 known_issue 标注 | 推荐同步整改效率高；备选单独 CR 增协调成本 | 影响 24 用例整改量；风险标注不完整致误判 |
| DQ-04 | scope | 24 用例目录迁移是否全部纳入 CR-033 范围 | 全部纳入 R-F-021 在 M4 一次性完成 | 分批迁移先 P0 后续迭代 | 推荐一次性完成避免半迁移；备选分批降低单次工作量 | 影响 M4 工作量；风险半迁移状态混乱 |
| DQ-05 | implementation | frontmatter 16 列哪些必填哪些可选 | 8 必填（编号/名称/三~五级目录/级别/类型/是否自动化）+ 8 可选 | 全部必填 | 推荐 8+8 平衡完整性和工作量；备选全部必填但信息缺失 | 影响 24 用例 frontmatter 补全量；风险可选列缺失 |
| DQ-06 | implementation | 用例名称连字符与文件名分隔符冲突 | frontmatter 用例编号为唯一标识，文件名按编号前缀正则匹配 | 下划线替代连字符做名称内部分隔 | 推荐编号为唯一标识不改命名习惯；备选改命名增迁移成本 | 影响 case_runner 文件名解析；风险解析失败 |

如果你回复 approve，表示你接受以上 6 项推荐方案，不表示授权以下不授权项。
不授权项: 不授权真实运行、凭据读取、安全边界变更、外部接口调用、数据写入、publish、live / 交易类操作；若 checkpoint 中列有额外不授权项，以 checkpoint 为准。

请只回复以下三个 exact 选项之一：
- approve
- 修改: <具体修改点>
- reject
