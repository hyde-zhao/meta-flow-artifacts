---
handoff_id: "CR-033-REQ-CLAR-meta-pm-RETURN"
cr_id: "CR-033"
phase: "requirement-clarification"
canonical_role: "meta-pm"
status: "completed"
completed_at: "2026-07-28T11:40:00+08:00"
context_ref: "process/changes/CR-033.md"
revision: "v1.2 (CP3 scope extension: environment file driver applied)"
---

# CR-033 requirement-clarification 交还摘要（v1.2 - 含 CP2 范围修改 + CP3 评审范围扩展）

## CP3 评审范围扩展记录（v1.2）

用户在 CP3 评审阶段提出环境文件驱动（resolve_env_refs）范围扩展，已确认作为 CR-033 范围扩展（非子 CR）。

| 扩展点 | 内容 | 影响产物 |
|--------|------|---------|
| 用例与环境解耦 | 用例用 ${ENV.*} 占位符引用环境参数，resolve_env_refs 自动解析 | R-F-027 新增, UC-EX-11 新增 |
| DUT 接口自动预配置 | 框架按环境文件自动 fw_update_interface，用例不写预配置 | R-F-028 新增 |
| 设备 URL 自动解析 | TREX_API_URL 从环境文件注入，devices.yaml fallback | R-F-029 新增 |
| ptm-atomic 约束 | TG 操作仍经 ptm-atomic，框架不直接调 TG REST API | R-C-009 新增 |
| 向后兼容 | 无 ${ENV.*} 的字面值 args 原样透传 | R-C-010 新增 |
| 门控不变 | dry-run 默认门 + --execute 授权门不变 | R-C-011 新增 |

## CP2 范围修改记录（v1.1）

用户在 CP2 门禁 pending 时回复"修改"，提出 5 点范围修改，已与用户确认 3 个决策点：

| 修改点 | v1.0 | v1.1 | 影响产物 |
|--------|------|------|---------|
| 执行入口 | --case-files(文件列表) | --tag(精确过滤) / --keyword(模糊匹配) | R-F-004 更新, R-F-026 新增 |
| 用例目录结构 | cases/upload/ 扁平 | cases/三级/四级/五级/<编号>-<名称>.md | R-F-022 新增, UC-EX-10 新增 |
| 用例 md 结构 | 仅 case_steps YAML | frontmatter 16 列 + case_steps YAML | R-F-024 新增 |
| 用例命名 | 原文件名 | <用例编号>-<用例名称>.md | R-F-023 新增 |
| 标签执行 | 无 | --keyword(关键词列模糊) + --tag(tags列精确) | R-F-025, R-F-026 新增 |
| 24 用例整改 | ARP 预热校验 | 目录迁移+重命名+frontmatter+tags+ARP 预热 | R-F-021 扩大, ST-EX-13 扩大 |

## 产物路径清单

| 产物 | 路径 | 状态 |
|------|------|------|
| USE-CASES | `docs/product/USE-CASES-PTM-TE-EXEC.md` | confirmed |
| REQUIREMENTS | `docs/product/REQUIREMENTS-PTM-TE-EXEC.md` | confirmed, ready_for_design=true |
| SCENARIOS | `docs/product/SCENARIOS-PTM-TE-EXEC.yaml` | confirmed |
| TEST-MATRIX | `docs/product/TEST-MATRIX-PTM-TE-EXEC.md` | confirmed |
| STORY-MAP | `docs/product/STORY-MAP-PTM-TE-EXEC.md` | confirmed |
| MVP-SCOPE | `docs/product/MVP-SCOPE-PTM-TE-EXEC.md` | confirmed |
| RELEASE-SLICES | `docs/product/RELEASE-SLICES-PTM-TE-EXEC.md` | confirmed |
| BACKLOG | `docs/product/BACKLOG-PTM-TE-EXEC.md` | confirmed |
| 讨论日志 | `process/discussions/CP2-SCENARIO-DISCUSSION-LOG-CR033.md` | resolved |
| 讨论恢复点 | `process/checks/CP2-DISCUSSION-CHECKPOINT-CR033.json` | resolved |
| CP1 自动检查 | `process/checks/CP1-CR033.result.json` | PASS |
| CP2 自动检查 | `process/checks/CP2-CR033.result.json` | PASS, ready_for_design=true |
| CLARIFICATION-LOG | `process/CLARIFICATION-LOG.md`（CR-033 段落已追加） | appended |

**文档文件决策说明**：现有 `docs/product/USE-CASES.md` / `REQUIREMENTS.md` 等文件属于 CR-030/CR-031（ptm-tse 逆向分析，scenario_subject_id=ptm-tse）。CR-033 是不同产品线（ptm-te 执行引擎，scenario_subject_id=ptm-te-exec）。为避免混合不同产品线，CR-033 的产品文档使用 `-PTM-TE-EXEC` 后缀新建文件，不修改既有 ptm-tse 文档。

## Scenario Gray Areas 处理结果

| 灰区 ID | 问题 | 用户选择 | 状态 | 影响面 |
|---|---|---|---|---|
| SGA-01 | dry-run 默认门与 runtime_authorization 边界 | A：dry-run 默认门，--execute 授权门 | resolved | 范围/验证方式/CP7/runtime_auth |
| SGA-02 | 跨仓库路由 ptm-team ↔ ptm-te 产物归属 | A：ptm-team canonical + install.py 回填 | resolved | 交付出口/文件所有权/重装一致性 |
| SGA-03 | known_issue DUT 行为差异判定 | A：known_issue 字段显式标记 | resolved | 结果分级/用例整改/自动化 |
| SGA-04 | ARP 预热强制方式 | C：规则 + 引擎双重保障 | resolved | 规则块/引擎逻辑/重装回退 |

- **SGQ 条数**：4 条（SGQ-01..04），全部用户可见场景确认交互完成
- **用户确认方式**：通过 host-orchestrator relay 确认
- **Deferred Ideas**：6 项（DEF-EX-01..06），含 HTML 报告 / --validate 中间模式 / 24 用例迁入 / 语义推断 / ptm-atomic 扩展 / pydantic

## CP2 Decision Brief 输入

### 用户真实意图

从多轮澄清和 CR-033 决策定稿中提炼的用户真实意图：

1. **消除硬编码与重装回退风险**：exec_v4.py 硬编码 17 用例 + DUT/TG 地址，重装后规则丢失；case_runner.py 全参数化 + install.py 规则固化彻底消除
2. **新增用例零代码改动**：测试执行工程师只需写 case_steps md 文件，case_runner 自动解析执行，不需要改 Python 代码
3. **结果可审计可分级**：PASS/FAIL/KNOWN_FAIL/ERROR 四态分级，安全合规负责人可审计 runtime_authorization，测试经理可区分 DUT 行为差异和脚本 bug

### 认知盲区补充

用户未显式提到但会影响维护、失败、权限、安装、升级、协作或长期演进的关键缺口：

1. **fw_logout op 可用性风险**：op_mapper 新增 fw_logout 映射，但 ptm-atomic 安装版是否暴露 auth logout 子命令需安装前验证（类似 fw_delete_object 问题）
2. **24 用例 known_issue 标注完整性**：known_issue 字段是新增的，24 用例 md 需要人工补标注，标注不完整会导致 FAIL 误判为 KNOWN_FAIL 或反之
3. **跨仓库 install.py 回填路径**：install.py 需要同时安装到 ptm-te workspace 的 .claude/skills/，回填路径错误会导致 workspace skill 与 canonical 源不一致
4. **TREX_API_URL 注入边界**：_build_exec_env 注入 TREX_API_URL 后，需确认 ptm-atomic trex-traffic CLI 是否实际消费该环境变量

### Scenario Gray Areas 处理结果

| 灰区 ID | 用户选择 | canonical refs | 讨论日志 | checkpoint |
|---|---|---|---|---|
| SGA-01 | A（dry-run 默认门） | REQUIREMENTS R-F-006,007 / R-NF-001 | `process/discussions/CP2-SCENARIO-DISCUSSION-LOG-CR033.md` SGQ-01 | `process/checks/CP2-DISCUSSION-CHECKPOINT-CR033.json` |
| SGA-02 | A（ptm-team canonical + install.py 回填） | REQUIREMENTS R-C-007,008 / delivery_routing | 同上 SGQ-02 | 同上 |
| SGA-03 | A（known_issue 显式标记） | REQUIREMENTS R-F-010,011 / R-F-021 | 同上 SGQ-03 | 同上 |
| SGA-04 | C（规则 + 引擎双重保障） | REQUIREMENTS R-F-008,019 / R-NF-003 | 同上 SGQ-04 | 同上 |

### 用户可见场景确认证据

| SGQ ID | 问题摘要 | 候选 | 用户回答 | 复述确认 | 影响面 |
|---|---|---|---|---|---|
| SGQ-01 | dry-run 默认门与授权边界 | A/B/C | A | dry-run 是默认门，--execute 需显式授权才校验业务 PASS/FAIL | 范围/验证/CP7/runtime_auth |
| SGQ-02 | 跨仓库产物归属与回填 | A/B/C | A | skill 源在 ptm-team canonical，过程文档写 ptm-team docs/product/，24 用例 md 留 ptm-te | 交付出口/文件所有权/重装 |
| SGQ-03 | known_issue DUT 行为差异判定 | A/B/C | A | 用例 md 中 known_issue 字段显式标记，case_runner 遇标记失败输出 KNOWN_FAIL | 结果分级/用例整改/自动化 |
| SGQ-04 | ARP 预热强制方式 | A/B/C | C | 规则定义 + 引擎强制双重保障，warming_up:true 时 case_runner 自动补 post_op | 规则块/引擎逻辑/重装回退 |

### Deferred Ideas

| ID | 内容 | 延后原因 | 重启条件 |
|---|---|---|---|
| DEF-EX-01 | HTML 报告生成 | result.json + report.md 足够 | 用户明确要求 |
| DEF-EX-02 | --validate 中间授权模式 | dry-run + --execute 二级已足够 | 需要更细粒度验证 |
| DEF-EX-03 | 24 用例 md 迁入 ptm-team | 留 workspace 更合适 | 需要版本控制 |
| DEF-EX-04 | expected_result 语义推断 | 显式标记更可审计 | NLP 成熟后 |
| DEF-EX-05 | ptm-atomic 本体扩展 | 只改消费侧 | ptm-atomic 升级 CR |
| DEF-EX-06 | pydantic schema 校验 | 保持轻量 | 设备模型复杂度增加 |

### 候选理解与取舍

| 灰区 | 候选 A（已选） | 候选 B/C（备选） | 取舍理由 |
|---|---|---|---|
| SGA-01 | dry-run 默认门 | B: 三级授权 / C: 默认 --execute | A 安全默认且授权边界清晰；B 增加复杂度；C 与 NO_PRODUCTION_WRITE 冲突 |
| SGA-02 | ptm-team canonical + install.py 回填 | B: 24 用例迁入 / C: 过程文档写 workspace | A canonical 源单一且可版本控制；B 运行时数据污染源仓库；C workspace 非 git 仓库 |
| SGA-03 | known_issue 字段显式标记 | B: 文本语义推断 / C: 不区分 | A 显式可审计；B 文本推断不可靠；C 人工后处理负担重 |
| SGA-04 | 规则 + 引擎双重保障 | A: 只靠规则 / B: 只靠引擎 | C 双重保障防重装回退；A 重装后规则可绕过；B 规则不可见 |

### 推荐范围

**Scope（In Scope）**：
- TG 设备建模：devices.yaml tg 块 + 6 组合 + SKILL.md + device-reference.md
- case-execution 引擎：case_runner.py 三入口 + dry-run + --execute + 逆序清理 + fw_logout
- 引擎增强：warming_up/post_op + retry + known_issue 四态 + 失败诊断 + 结构化报告 + extract_payload
- 规则固化：install.py ≥4 条新规则
- 12 条改进全量落地（P0->P3 四期）
- 24 用例 ARP 预热批量整改

**Out of Scope**：改 ptm-atomic 本体 / 引入 pytest/robot / pydantic / HTML 报告 / 改 traffic-skill/ngfw-install / TG 快照采集 / 凭据读取 / 真实设备自动授权

**推荐理由**：范围目标已由用户与 host-orchestrator 在前置对话中确认并定稿（含审核修订）。meta-pm 基于已确认范围产出正式产物，不重新做范围决策。

### 验证场景摘要

| 类型 | 数量 | 覆盖情况 |
|------|------|---------|
| 正向 | 6 | TG 建模 / 目录批量 / 单用例 / dry-run / ARP 预热 / fw_logout |
| 负向 | 4 | dry-run 拒绝写 / 未授权拒绝 / known_issue KNOWN_FAIL / md 格式错误 |
| 边界 | 4 | 空目录 / 缺 tg 块 / STATE_INVALID 重连 / ConnectTimeout 重试 |
| 权限 | 1 | runtime_authorization 审计 |
| 失败恢复 | 2 | 逆序清理 / 幂等容错 |
| 预检 | 3 | op_mapper 一致性 / install.py 重装 / 24 用例 ARP 校验 |
| **合计** | **20** | static review 11 + runtime 9（需 runtime_authorization） |

### MVP / 发布切片摘要

| 切片 | 范围 | Story 数 | 验证入口 |
|------|------|---------|---------|
| RS-EX-01（P0） | TG 建模 + 规则固化 | ST-EX-01,02,03 | SCN-EX-01,18,19 |
| RS-EX-02（P0） | case_runner 核心 + dry-run | ST-EX-04,05,06,07 | SCN-EX-02,03,04,07,08,16 |
| RS-EX-03（P1） | 引擎增强 + 四态分级 | ST-EX-08,09,10,11,12 | SCN-EX-05,09,10,17 |
| RS-EX-04（P2-P3） | 用例整改 + 消费侧 | ST-EX-13,14 | SCN-EX-20 |

### 待人工决策项

| DQ ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣 / 影响 / 风险 / 回退 |
|---|---|---|---|---|---|
| DQ-01 | runtime_authorization | case_runner --execute 模式下的真实设备写操作授权 | CP7 验证时使用 static review + dry-run 替代 runtime | 等待 VALIDATION-ENV.yaml 授权后执行 runtime 验证 | 推荐：static review 不触发写操作，安全；备选：runtime 验证更充分但需设备和授权。回退：runtime 失败则降级为 static review |
| DQ-02 | implementation | fw_logout op 在 ptm-atomic 安装版是否暴露 | 安装前 `ptm-atomic show fw_logout` 验证；未暴露时降级为清理 session 文件 | 强制要求 ptm-atomic 升级暴露 fw_logout | 推荐：降级方案保证可用性；备选：升级 ptm-atomic 但超出 CR-033 范围。回退：降级为 session 文件清理 |
| DQ-03 | scope | 24 用例 md 的 known_issue 标注是否纳入 CR-033 范围 | 纳入 R-F-021（24 用例 ARP 预热整改时同步检查 known_issue 标注） | 单独 CR 处理 known_issue 标注 | 推荐：同步整改效率高；备选：单独 CR 但增加协调成本。回退：只做 ARP 预热整改，known_issue 标注进 BACKLOG |
| DQ-04 | scope | 24 用例目录迁移是否全部纳入 CR-033 范围（还是分批迁移） | 全部纳入 R-F-021，在 M4 里程碑一次性完成 | 分批迁移（如先迁移 P0 用例，后续迭代迁移剩余） | 推荐：一次性完成避免半迁移状态；备选：分批降低单次工作量。回退：若工作量超预期，P3 用例迁移进 BACKLOG |
| DQ-05 | implementation | frontmatter 16 列哪些必填哪些可选 | 必填：用例编号/用例名称/三级目录/四级目录/五级目录/用例级别/测试类型/是否自动化；可选：组网描述/组网约束/预置条件/测试步骤/预期结果/首次创建版本/最后变更版本/关键词/tags | 全部必填 | 推荐：8 必填 + 8 可选平衡完整性和工作量；备选：全部必填但部分列可能信息缺失。回退：缺失列填 N/A 或 TBD |
| DQ-06 | implementation | 用例名称中的连字符与文件名分隔符冲突（如"创建策略路由-有效参数-策略ID不存在-创建成功"中多个连字符） | 以 frontmatter 用例编号列为唯一标识；文件名解析时按编号前缀(如 PC-M1-01-01)正则匹配，剩余部分为名称 | 用下划线替代连字符做名称内部分隔（如 创建策略路由_有效参数_策略ID不存在_创建成功） | 推荐：frontmatter 编号为唯一标识，不改变名称中的连字符；备选：改变命名习惯但增加迁移成本。回退：解析失败时报错让用户手动指定 |

### 成功指标

| 指标 ID | 指标 | 目标值 | 验收口径 |
|---|---|---|---|
| SM-EX-01 | TG 设备型号覆盖 | 6 组合 | device-reference.md 6 组合全覆盖 |
| SM-EX-02 | 用例执行入口 | 3 种 | case_runner.py 支持 --cases-dir/--case-files/--case-file |
| SM-EX-03 | 硬编码消除 | 0 | case_runner.py 无硬编码 IP |
| SM-EX-04 | 统一解析函数 | 1 个 | extract_payload(op_id, envelope) |
| SM-EX-05 | 规则固化 | ≥4 条 | install.py 规则块含 TG路由/max_loss/ARP预热/session |
| SM-EX-06 | 改进覆盖 | 12/12 | 12 条改进全量落地 |
| SM-EX-07 | ARP 预热覆盖 | 24/24 | 24 用例 warming_up step 合规 |
| SM-EX-08 | 结果分级 | 4 态 | PASS/FAIL/KNOWN_FAIL/ERROR |
| SM-EX-09 | 重装一致性 | 0 丢失 | 重装后规则+skill 存在 |
| SM-EX-10 | fw_logout op | 1 个 | op_mapper 含 fw_logout 映射 |

### 场景充分性判断

8 维后台覆盖扫描后，场景足以进入 HLD：
- D1-D8 全部已覆盖
- 20 个验证场景覆盖正向/负向/边界/权限/失败恢复/预检
- 21 条功能需求均有 Given/When/Then 验收条件
- BLOCKING 未决项 = 0

**判断**：场景充分，可进入 HLD。

### 用户自由表达确认记录

本轮所有灰区选择由用户通过 host-orchestrator relay 确认，无拒绝结构化选择或纠正上下文的情况。用户在选择 SGA-04 时选择了推荐项 C（规则 + 引擎双重保障）而非 A（只靠规则），表明用户对重装回退风险有明确认知。

### 风险与影响

| 风险 ID | 风险 | 影响 | 缓解 |
|---|---|---|---|
| RISK-CR033-CROSS-REPO | 跨仓库回填后 workspace skill 与 canonical 不一致 | 重装后行为不一致 | install.py 安装后验证 + op_mapper validate |
| RISK-CR033-DEVICE-WRITE | --execute 模式触发真实设备写操作 | 设备状态变更 | dry-run 默认门 + --execute 需显式授权 + runtime_authorization 审计 |
| RISK-CR033-MIGRATION-REGRESSION | exec_v4.py 迁移后 case_runner 行为不一致 | 用例执行结果偏差 | dry-run 校验 + 对比测试 + exec_v4.py 废弃标记 |
| RA-001 | 跨仓库回填不一致 | 重装后规则/skill 丢失 | install.py 验证 |
| RA-002 | devices.yaml 格式变更 | case_runner 取址失败 | 启动时校验 |
| RA-003 | known_issue 标注不完整 | FAIL/KNOWN_FAIL 误判 | ARP 整改时同步检查 |
| RA-004 | fw_logout 未暴露 | 登出失败 | 安装前验证 + 降级清理 session |
| RA-005 | TREX_API_URL 注入无效 | tg op 走旧地址 | 集成测试验证 |
| RA-008 | 重装后 ARP 预热规则被绕过 | warming_up 无 post_op 清理 | SGA-04=C 双重保障 |
| RA-009 | exec_v4.py 仍被误用 | 硬编码回退 | 废弃标记 + README 指向 case_runner |

### 待用户决策

无阻止进入 HLD 的关键问题。所有 BLOCKING 未决项已清零。3 个待人工决策项（DQ-01..03）可在 CP2 人工确认时一并决策，不阻塞 HLD 启动。

## CP1/CP2 证据

| 检查点 | 路径 | 结论 |
|---|---|---|
| CP1 用户场景完备门 | `process/checks/CP1-CR033.result.json` | PASS |
| CP2 需求/场景/范围基线门 | `process/checks/CP2-CR033.result.json` | PASS, ready_for_design=true |

## Evidence Refs

- `process/handoffs/CR-033-REQ-CLAR-meta-pm.md`（handoff）
- `process/changes/CR-033.md`（CR 完整范围）
- `process/REQUEST.md`（三需求原始请求）
- `process/CLARIFICATION-LOG.md`（CR-033 调研段落）
- `process/discussions/CP2-SCENARIO-DISCUSSION-LOG-CR033.md`（4 条 SGQ）
- `process/checks/CP2-DISCUSSION-CHECKPOINT-CR033.json`（恢复点）
- `process/checks/CP1-CR033.result.json`（CP1 自动检查）
- `process/checks/CP2-CR033.result.json`（CP2 自动检查）
- `docs/product/USE-CASES-PTM-TE-EXEC.md`（11 场景，v1.2）
- `docs/product/REQUIREMENTS-PTM-TE-EXEC.md`（29 功能 + 11 约束 + 5 非功能，v1.2）
- `docs/product/SCENARIOS-PTM-TE-EXEC.yaml`（29 验证场景，v1.2）
- `docs/product/TEST-MATRIX-PTM-TE-EXEC.md`（覆盖矩阵，v1.2）
- `docs/product/STORY-MAP-PTM-TE-EXEC.md`（17 Story 候选，v1.2）
- `docs/product/MVP-SCOPE-PTM-TE-EXEC.md`（19 In Scope，v1.2）
- `docs/product/RELEASE-SLICES-PTM-TE-EXEC.md`（4 切片）
- `docs/product/BACKLOG-PTM-TE-EXEC.md`（8 Backlog 项）

## N/A / WAIVED 项

| 项 | 原因 | 影响范围 | 后续触发条件 |
|---|---|---|---|
| 无 N/A 项 | 全部产物已产出 | N/A | N/A |
| 无 WAIVED 项 | 全部检查项已检查 | N/A | N/A |
