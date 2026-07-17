---
status: draft
version: "2.2"
source_scenarios: "docs/product/SCENARIOS.yaml"
---

# ptm-tse 逆向分析 — 用户旅程 / 需求 / 验证覆盖矩阵

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-15 | host-orchestrator | 初始工程场景覆盖矩阵。 |
| 1.1 | 2026-07-15 | host-orchestrator | 以用户可观察结果重写场景说明；保留 SCN / REQ / Story ID。 |
| 1.3 | 2026-07-16 | host-orchestrator | CP7 独立复核整改 v2：SCN-RA-01~13 全部 13 场景回写 static review completed；覆盖统计与缺口状态全面更新。 |
| 1.2 | 2026-07-15 | host-orchestrator | 追加 ITR 摄取、保存失败保护、逐单/批量总结和改进候选的用户结果覆盖。 |
| 2.1 | 2026-07-17 | host-orchestrator | CR-031：新增项目级安装、运行根隔离及安装失败阻断的两条工程场景；保留 CR-030 已完成验证，不把新增场景误记为完成。 |
| 2.2 | 2026-07-17 | host-orchestrator | CR-031 scope reframe：项目级安装已验证；SCN-RA-14/15 改为运行数据权限、分类、support 来源和生命周期治理。 |

| 用户旅程场景 | 用户可观察结果 | Requirement ID | Story ID | 验证类型 | 自动化状态 | 手工验收状态 | 验证文件 / 命令 | 未覆盖原因 |
|---|---|---|---|---|---|---|---|---|
| SCN-RA-01（测试经理启动预防） | P1 得到启动、责任人和时限；P2 可明确选择；不会把证据不足伪装成结论 | 001,002,003 | ST-RA-01 | static review | completed | passed | `skills/reverse-analysis/SKILL.md` §1-§2 资格检查与可信输入（P1/P2/P3/P4 资格判定 + 5 条证据线 + 三线阈值）；CP6 检查文件 | 无 runtime reviewer 交互环境（以 Skill 静态契约和状态机审查替代） |
| SCN-RA-02（架构师建立事实） | 证据不足时得到具体缺口与 Owner，不能确认根因 | 002,010 | ST-NRA-01 | static review | completed | passed | `skills/reverse-analysis/SKILL.md` §7 证据不足保护（11 项禁止 P-01~P-11 + 二次校验 + gap_source 4 分类）；CP6 检查文件 | 无 runtime 证据不足触发环境（以 Skill 静态禁止规则和阈值契约审查替代） |
| SCN-RA-03（团队确认解释） | 团队能审计根因、反证、引入点和控制失效；不同意时保持草案 | 003–006 | ST-RA-02 | static review | completed | passed | `skills/reverse-analysis/SKILL.md` §3-§5 六维分析引擎（根因四层状态机 + 反证引入点 + 控制失效 + 指标降级）；CP6 检查文件 | 无 runtime 分析执行环境（以 Skill 静态状态机和指标契约审查替代） |
| SCN-RA-04（设计负责人接收改进） | 已批准输入带来源、范围、验收和限制；未批准输入不进入改进工作 | 007,008 | ST-RA-03 | static review | completed | passed | `skills/improvement-tracker/SKILL.md` §3 CA/PA 治理（草案生成 + 批准状态机 + Approved Input + 消费者映射）；CP6 检查文件 | 无 runtime CA/PA 批准交互环境（以 Skill 静态状态机契约审查替代） |
| SCN-RA-05（质量负责人关闭） | 基于有效性、观察窗和复发证据做关闭/保持开放决定 | 009,013 | ST-RA-04 | static review | completed | passed | `skills/improvement-tracker/SKILL.md` §4 闭环跟踪（行动项状态机 + 有效性检查 + 观察窗 30d + 四条件关闭硬断言）；CP6 检查文件 | 无真实 30 天观察数据（以 Skill 静态关闭条件契约审查替代） |
| SCN-RA-06（合规负责人守边界） | 收到拒绝和合规替代路径，且无外部读取/生产操作 | 011 | ST-NRA-02 | static review / security | completed | passed | `skills/reverse-analysis/SKILL.md` §8 权限边界拒绝与越权保护（23 项禁止规则 + 9 类拒绝矩阵 + 三层防护）；CP6 检查文件 | 无 runtime 越权测试环境（以 Skill 静态禁止清单和拒绝契约审查替代） |
| SCN-RA-07（测试经理识别不适用） | 内部问题获得不适用和重启条件，不套用现网模型 | 012 | ST-RA-01, ST-NRA-01 | static review | completed | passed | `skills/reverse-analysis/SKILL.md` §2.4 P3/P4 rejected + 内部 deferred + 重启条件；§7.1 二次校验；CP6 检查文件 | 无 runtime 内部问题触发环境（以 Skill 静态资格判定契约审查替代） |
| SCN-RA-08（架构师摄取并保存） | 获得可追溯问题单输入、数量和数据质量提示；不发生凭据读取/写入 | 011,014,015 | ST-RA-05.1-INGEST | static review | completed | passed | `skills/itr-ticket-ingestion/SKILL.md` §1-§5（allowlist 校验 + HTTP GET + 快照保存 + 批次清单 + DB 写入）；CP6 检查文件 | 无 runtime ITR 实际调用（内网环境不可用），以 Skill 文本契约和静态结构审查替代 |
| SCN-RA-09（摄取失败保护） | 看到可恢复失败说明，历史快照和结论不被覆盖 | 016,017 | ST-NRA-03 | static review | completed | passed | `skills/itr-ticket-ingestion/SKILL.md` §8 摄取失败保护（6 类失败分类 + 事务边界 + 幂等保护 + 降级策略）；CP6 检查文件 | 无 runtime 故障注入环境，以 Skill 静态错误处理契约审查替代 |
| SCN-RA-10（总结与候选措施） | 获得可审计逐单总结、批量趋势和待批准 CA/PA 候选 | 017,018 | ST-RA-05.3-ANALYZE、ST-RA-03 | static review | completed | passed | `skills/reverse-analysis/SKILL.md` §6 S1 分析管线（逐单/批量分析 + analysis_run 管理 + 报告草案 + reviewer 发布路径）；CP6 检查文件 | CA/PA 批准与分发由 ST-RA-03 improvement-tracker 负责，不在本次验证范围 |
| SCN-RA-11（质量阻断） | 数据质量不达标时没有可信分析报告 | 015,016 | ST-RA-05.2-CLEAN、ST-RA-06.1-DETECT | static review | completed | passed | `skills/itr-ticket-ingestion/SKILL.md` §7 QCOMB-01~05 组合风险规则 + overall_status 状态机；CP6 检查文件 | 无真实 ITR 数据源验证质量门控触发（以静态规则契约审查替代） |
| SCN-RA-12（结论降级） | 无分母不称密度；无控制证据只输出候选流出层 | 017,021 | ST-RA-05.2-CLEAN、ST-RA-06.2-REFRESH | static review / boundary | completed | pass_with_risk | `skills/reverse-analysis/SKILL.md` §5.2 无可信分母降级策略；`skills/itr-ticket-ingestion/SKILL.md` §6.3 清洗规则；CP6 检查文件 | 无可信分母运行测试数据（N/A），降级逻辑为静态契约验证；风险：实际分母数据格式未经验证 |
| SCN-RA-13（基线/敏感保护） | 无措施基线为 needs-baseline；未分类字段不进入 LLM/报告 | 011,022 | ST-RA-06.2-REFRESH | static review / security | completed | passed | `skills/reverse-analysis/SKILL.md` §8 权限边界拒绝与越权保护（P-12~P-19 禁止清单 + denial_record 审计格式）；CP6 检查文件 | 无 runtime 越权测试环境（以静态禁止清单和拒绝契约审查替代） |
| SCN-RA-14（既有运行数据治理） | SQLite/WAL/SHM、快照和元数据有分类、Owner、来源、权限和保留状态；不执行真实修复 | 023,024,025 | ST-RA-07（待拆分） | governance fixture / read-only audit | planned | planned | 待 CR-031 CP3/CP5：数据清单、分类字典、权限核验、support provenance fixture | 现有安装已验证；待设计运行数据治理契约，CP2 未批准前不得修改实际数据 |
| SCN-RA-15（数据治理不合格阻断） | 权限、分类、support 来源或清理授权不合格时，发布/自动修复/清理均被拒绝且真实数据不变 | 023,024,025,026 | ST-NRA-04（待拆分） | governance negative fixture | planned | planned | 待 CR-031 CP3/CP5：权限失败、未分类、来源缺失、无 Owner 授权 fixture | 失败分类、保留授权和人工修复路径尚未设计；CP2 未批准前不得实施 |

## 覆盖统计

| 维度 | 总数 | covered | gap | waived | 说明 |
|---|---:|---:|---:|---:|---|
| HIGH 场景 | 11 | 9 | 2 | 0 | CR-030 的 9 个 HIGH 场景已通过；CR-031 的 SCN-RA-14/15 待设计与验证 |
| 正向 / 负向 / 边界 | 15 | 13 | 2 | 0 | SCN-RA-01~13 已完成；SCN-RA-14/15 planned |
| 权限 / 安全场景 | 1 | 1 | 0 | 0 | 权限边界拒绝契约已通过静态审查（reverse-analysis §8） |
| 外部失败场景 | 0 | 0 | 0 | 1 | 外部系统连接不在授权范围 |

## 缺口处理

| Gap ID | 来源 | 缺口 | 阻断等级 | 推荐处理 | 责任方 |
|---|---|---|---|---|---|
| GAP-RA-01 | SCN-RA-04 | 下游契约格式未定 | RESOLVED — improvement-tracker §3 已定义 Approved Input 契约 | meta-se |
| GAP-RA-02 | SCN-RA-05 | 真实 30 天观察不可在 CI 执行 | RESOLVED — improvement-tracker §4 已定义四条件关闭契约，static review 已验证 | meta-qa |
| GAP-RA-03 | SCN-RA-06 | 禁止能力测试尚无实现 | RESOLVED — reverse-analysis §8 已定义 23 项禁止 + 9 类拒绝矩阵，static review 已验证 | meta-dev / meta-qa |
| GAP-RA-04 | SCN-RA-08 | ITR 响应 schema、数据分类、保留与存储位置未定 | **RESOLVED** — CR-030 CP5 已固定：allowlist-config.yaml + field-mapping.yaml + data/snapshots/ + SQLite schema v1.0 | host-orchestrator / meta-se |
| GAP-RA-05 | SCN-RA-10 | 总结与措施的数据/人工 review 契约未定 | **RESOLVED** — reverse-analysis §6 S1 分析管线定义了 analysis_run 管理、报告草案输出与 reviewer 发布路径契约 | meta-se |
| GAP-RA-06 | SCN-RA-11..13 | 质量、指标、证据、基线和敏感字段的 LLD schema 未定 | **RESOLVED** — itr-ticket-ingestion §7 QCOMB 规则 + reverse-analysis §5.2 降级策略 + §7 证据保护 + §8 权限边界均已版本化定义 | meta-se / meta-dev |
| GAP-RA-07 | SCN-RA-14/15 | 数据分类/Owner、文件权限、support 来源和保留/清理授权尚未形成可执行契约 | **BLOCKING（CR-031）** — CP2 确认产品基线后进入 CP3/CP5；未通过前不得修改真实运行数据、权限或生命周期 | host-orchestrator / meta-se / meta-dev |

> CR-030 的 SCN-RA-01..13 保持 13/13 completed。CR-031 的 SCN-RA-14/15 当前为运行数据治理场景，仍是 2 planned；不得以已完成的安装验证或 CR-030 结论替代数据治理验证。
