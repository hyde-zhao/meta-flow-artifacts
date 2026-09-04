---
status: confirmed
version: "1.2"
confirmed_by: "user-via-host-orchestrator-relay"
confirmed_at: "2026-07-28T11:30:00+08:00"
engagement_mode: production
scenario_subject_type: target-artifact
scenario_subject_id: ptm-te-exec
target_artifact_type: skill
governance_mode: conditional
review_policy: standard
delivery_routing:
  mode: project-readme-contract
  output_root: "skills/case-execution/、skills/device-management/、docs/product/、script/ptm_team/install.py"
  source: user-confirmed
total_use_cases: 11
---

# ptm-te 执行引擎 - 用户场景

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 | 文档处理方式 |
|---|---|---|---|---|
| 1.0 | 2026-07-28 | meta-pm | CR-033 初始场景基线：TG 设备建模 + case-execution 执行引擎 + 12 条改进 | 新建文件（CR-033 引入新产品线 ptm-te-exec，与既有 ptm-tse 逆向分析文档隔离） |
| 1.1 | 2026-07-28 | meta-pm | CP2 范围修改：UC-EX-02/03 执行入口更新（--tag/--keyword 替代 --case-files）；新增 UC-EX-10 用例结构化（目录/命名/frontmatter 16 列/tags 列） | 原文档增量更新；保留 v1.0 UC-EX-01..09 语义 |
| 1.2 | 2026-07-28 | meta-pm | CP3 评审范围扩展（环境文件驱动）：新增 UC-EX-11 多环境执行（${ENV.*} 占位符 + --env-file 切换环境） | 原文档增量更新；保留 v1.1 UC-EX-01..10 语义不变 |

## 用户画像（Personas）

| 画像 ID | 角色名称 | 典型背景 | 核心诉求 | 技术水平 |
|---------|---------|---------|---------|---------|
| P-EX-01 | 测试执行工程师 | 负责运行策略路由用例、收集执行结果、排查失败用例 | 用一个命令跑全部或部分用例，得到 PASS/FAIL/KNOWN_FAIL/ERROR 四态结果，不用关心底层 op_mapper 映射 | 中级 |
| P-EX-02 | 测试平台开发者 | 维护 case-execution skill、op_mapper、device-management skill | 新增用例零代码改动；硬编码全参数化；重装后规则和 skill 不丢失 | 高级 |
| P-EX-03 | 测试经理 | 审查执行报告，决定是否推进发布 | 看到结构化报告和结果分级，能区分"DUT 行为差异"和"脚本 bug" | 中级 |
| P-EX-04 | 安全/合规负责人 | 确保执行引擎不会越权写生产设备 | dry-run 是默认门，--execute 需显式授权，授权行为可审计 | 高级 |

## 成功指标（Success Metrics）

| 指标 ID | 指标名称 | 度量方式 | 目标值 |
|---------|---------|---------|--------|
| SM-EX-01 | TG 设备型号覆盖 | devices.yaml tg 块定义数 + device-reference.md 6 组合对照 | 1 个 tg 块 + 6 组合（2 子类型 × 3 硬件平台） |
| SM-EX-02 | 用例执行入口覆盖 | case_runner.py 支持的入口模式数 | 3 种（目录 glob / 文件列表 / 单用例） |
| SM-EX-03 | 硬编码消除 | case_runner.py 中硬编码 IP 地址数 | 0（DUT_URL/TG_URL/DEFAULT_API_URL 全参数化） |
| SM-EX-04 | 统一解析函数 | extract_payload(op_id, envelope) 函数数 | 1 个 |
| SM-EX-05 | 规则固化 | install.py ptm-te-workflow 规则块新增规则数 | ≥4 条（TG 路由 / max_loss / ARP 预热 / session 生命周期） |
| SM-EX-06 | 改进覆盖 | 12 条改进落地数 | 12/12 |
| SM-EX-07 | ARP 预热覆盖 | 24 用例 md ARP 预热校验/补充数 | 24/24 |
| SM-EX-08 | 结果分级 | case_runner.py 输出的结果状态数 | 4 态（PASS / FAIL / KNOWN_FAIL / ERROR） |
| SM-EX-09 | 重装一致性 | install.py 安装后规则与 skill 丢失数 | 0 |
| SM-EX-10 | fw_logout op | op_mapper 新增 fw_logout op 数 | 1 个 |

## 明确排除（Out of Scope）

- 不改 ptm-atomic CLI 本体（只改 op_mapper / trex-api / case_runner 消费侧）
- 不引入 pytest / robot / 外部 eval 框架
- 不为 devices.yaml 引入 pydantic
- 不做 HTML 报告（result.json + report.md 足够，HTML 进 BACKLOG）
- 不改 traffic-skill / ngfw-install skill
- 不做 collect_sysinfo TG 快照采集（TG 设备纳入清单不纳入快照）
- 不读取凭据 / secret / 账户（NO_CREDENTIAL_READ）
- 不做真实设备 `--execute` 写操作的自动授权（属 NO_PRODUCTION_WRITE，需单独 runtime_authorization）

## 治理附录（Governance）

| 字段 | 当前值 | 说明 |
|------|--------|------|
| `engagement_mode` | production | ptm-te 是目标产品，skill 源在 ptm-team 仓库 |
| `scenario_subject_type` | target-artifact | 目标交付形态为 case-execution skill + device-management 扩展 |
| `scenario_subject_id` | ptm-te-exec | ptm-te 执行引擎产品线 |
| `target_artifact_type` | skill | case-execution SKILL.md + case_runner.py CLI |
| `governance_mode` | conditional | standard 工作流，CP2/CP3/CP5/CP8 关键人工门禁停留确认 |
| `review_policy` | standard | 多 Story + 架构 + 规则，不可 fast-lane |
| `delivery_routing.mode` | project-readme-contract | skill 源在 ptm-team canonical，install.py 安装回 ptm-te workspace |
| `delivery_routing.output_root` | skills/case-execution/、skills/device-management/、docs/product/、script/ptm_team/install.py | canonical 源路径 |
| `delivery_routing.source` | user-confirmed | SGA-02 用户确认跨仓库路由方案 A |

## Scenario Gray Areas

**Discussion Log**：`process/discussions/CP2-SCENARIO-DISCUSSION-LOG-CR033.md`
**Checkpoint**：`process/checks/CP2-DISCUSSION-CHECKPOINT-CR033.json`

| 灰区 ID | 问题 | 为什么重要 | 影响面 | 用户选择 | 状态 |
|---|---|---|---|---|---|
| SGA-01 | case-execution 的 dry-run 默认门与 runtime_authorization 边界 | 决定 case_runner 默认行为是安全干跑还是实际执行，影响 CP7 验证层和授权决策 | 范围 / 验证方式 / CP7 验证层 / runtime_authorization | A：dry-run 默认门，--execute 授权门 | resolved |
| SGA-02 | 跨仓库路由 ptm-team(canonical) ↔ ptm-te(workspace) 产物归属与回填 | 决定 skill 源、过程文档、用例 md 的归属，影响交付出口和重装一致性 | 交付出口 / 文件所有权 / install.py 规则 / 重装一致性 | A：ptm-team canonical + ptm-te workspace + install.py 回填 | resolved |
| SGA-03 | known_issue DUT 行为差异判定与脚本 bug 区分 | 决定结果分级判定逻辑，影响用例 md 改造量和自动化程度 | 结果分级 / 用例 md 改造量 / 24 用例整改 / 自动化程度 | A：known_issue 字段显式标记 | resolved |
| SGA-04 | ARP 预热与正式流相同 template 的强制方式 | 决定 ARP 预热一致性保障机制，影响规则块和引擎逻辑 | 规则块 / case_runner 引擎逻辑 / 24 用例整改量 / 重装回退风险 | C：规则 + 引擎双重保障 | resolved |

## Deferred Ideas

| ID | 想法 / 风险 / 扩展场景 | 来源 | 延后原因 | 重启条件 |
|---|---|---|---|---|
| DEF-EX-01 | HTML 报告生成 | CR-033 非目标 | result.json + report.md 足够 | 用户明确要求或 CP8 后 |
| DEF-EX-02 | `--validate` 中间授权模式（只读 op 实际执行，写 op 跳过） | SGA-01 候选 B | dry-run + --execute 二级已足够 | 需要更细粒度验证时 |
| DEF-EX-03 | 24 用例 md 迁入 ptm-team 统一管理 | SGA-02 候选 B | 24 用例是运行时数据，留 workspace 更合适 | 需要版本控制用例 md 时 |
| DEF-EX-04 | expected_result 文本语义自动推断 KNOWN_FAIL | SGA-03 候选 B | 文本推断不可靠，显式标记更可审计 | NLP 能力成熟后 |
| DEF-EX-05 | ptm-atomic CLI 本体扩展（如 fw_delete_object 暴露） | CR-033 非目标 | 只改消费侧，不改 ptm-atomic 本体 | ptm-atomic 升级 CR |
| DEF-EX-06 | pydantic devices.yaml schema 校验 | CR-033 非目标 | 不引入 pydantic，保持轻量 | 设备模型复杂度显著增加时 |

## 使用场景列表

### UC-EX-01：TG 设备建模与清单管理

| 字段 | 内容 |
|------|------|
| **使用角色** | 测试平台开发者（P-EX-02） |
| **触发条件** | 需要在 devices.yaml 中新增 TG 类型设备，或查询 TG 设备型号对照 |
| **输入** | TG 设备信息：设备名、IP、串口地址、子类型（ixia-c/trex）、硬件平台（EP/C236/J1900）、api_server 地址 |
| **处理逻辑** | 1. 从用户输入提取 TG 设备信息 2. 查阅 device-reference.md 确认子类型×硬件平台组合（6 组合）3. 按 devices.yaml 模板写入 tg 块（type:TG + host + serial_url + sub_type + hardware_platform + ssh + api_server）4. 凭据使用 `${TG_SSH_PASSWORD}` 等环境变量占位 5. 验证 YAML 语法 |
| **输出/结果** | devices.yaml 新增 tg 块；device-reference.md 补全 6 组合；.env.example 新增 TG 环境变量 |
| **前置条件** | device-management skill 已安装；devices.yaml 模板可用 |
| **排除情况** | 不采集 TG 系统快照（collect_sysinfo 不覆盖 TG）；不引入 pydantic；TG 接口拓扑留 traffic-skill |

**处理流程：**
1. 用户提供 TG 设备信息（名称、IP、子类型、硬件平台等）
2. 查阅 device-reference.md 确认 6 组合中的对应型号
3. 按 tg 块模板写入 devices.yaml（含 api_server: ixia-c:8450 / trex:8000）
4. 补全 .env.example 的 TG_SSH_PASSWORD / TG_TELNET_PASSWORD
5. 验证 YAML 语法和字段完整性

---

### UC-EX-02：用例批量执行-目录 glob / 按标签或关键字入口

| 字段 | 内容 |
|------|------|
| **使用角色** | 测试执行工程师（P-EX-01） |
| **触发条件** | 需要运行一个目录下的全部用例，或按标签/关键字筛选部分用例 |
| **输入** | `case_runner.py run --cases-dir cases/IPv4策略路由 --devices-yaml devices.yaml`（按目录）；或 `case_runner.py run --tag 策略路由,P0 --devices-yaml devices.yaml`（按标签精确过滤）；或 `case_runner.py run --keyword fw_config_policy_route --devices-yaml devices.yaml`（按关键字模糊匹配） |
| **处理逻辑** | 1. 目录 glob 模式：扫描 --cases-dir 下所有 .md 用例文件 2. --tag 模式：扫描全部用例 frontmatter tags 列，精确匹配指定 tag（支持多 tag AND/OR） 3. --keyword 模式：扫描全部用例 frontmatter 关键词列，子串模糊匹配 4. --tag + --keyword 组合：同时满足两个条件的用例 5. 从 devices.yaml 读取 DUT/TG 地址 6. 预登录建立共享 session 7. 按用例顺序逐个执行 8. 每个 step 解析 warming_up/post_op/retry/known_issue 9. 每个用例结束后逆序清理 mutation ops 10. 输出 result.json + report.md |
| **输出/结果** | runs/<run-id>/result.json（含四态分级 + 匹配用例列表）+ report.md（结构化报告） |
| **前置条件** | devices.yaml 已配置 firewall + tg 块；op_mapper 已安装；用例 md 在 cases/ 目录结构下且含 frontmatter tags/关键词列 |
| **排除情况** | 不引入 pytest/robot 框架；不硬编码用例列表；不依赖 --case-files 文件列表模式（已替换为 --tag/--keyword） |

**处理流程：**
1. 用户指定 --cases-dir / --tag / --keyword（三选一或组合）
2. 目录 glob：扫描 .md 文件；--tag：按 frontmatter tags 列精确过滤；--keyword：按关键词列模糊匹配
3. 从 devices.yaml 读取 DUT host 和 TG api_server
4. 预登录 DUT 建立共享 session（fw_login_web_management）
5. 逐用例执行：parse_steps -> 逐 step exec_op -> warming_up/post_op 处理 -> retry 轮询 -> known_issue 判定
6. 每用例结束后逆序清理（fw_delete_policy_route / fw_delete_object / tg_stop_traffic_stream）
7. 全部用例结束后 fw_logout 登出共享 session
8. 输出 result.json + report.md（含匹配用例列表和筛选条件）

---

### UC-EX-03：用例单文件执行-单用例入口

| 字段 | 内容 |
|------|------|
| **使用角色** | 测试执行工程师（P-EX-01） |
| **触发条件** | 需要运行单个用例（调试或针对性验证） |
| **输入** | `case_runner.py run --case-file cases/upload/IPv4策略路由-PC-M3-01-02.md --devices-yaml devices.yaml` |
| **处理逻辑** | 1. 读取单个 .md 用例文件 2. 从 devices.yaml 读取 DUT/TG 地址 3. 预登录建立 session 4. 执行该用例的全部 steps 5. 清理 mutation ops 6. 输出 result.json |
| **输出/结果** | runs/<run-id>/result.json（单用例结果） |
| **前置条件** | 同 UC-EX-02 |
| **排除情况** | 不硬编码用例路径 |

---

### UC-EX-04：用例 dry-run 校验-默认安全门

| 字段 | 内容 |
|------|------|
| **使用角色** | 测试执行工程师（P-EX-01）、安全/合规负责人（P-EX-04） |
| **触发条件** | 需要校验用例的命令构建和参数映射正确性，不实际执行写操作 |
| **输入** | `case_runner.py run --cases-dir cases/upload --devices-yaml devices.yaml`（不加 --execute） |
| **处理逻辑** | 1. 默认 dry-run 模式 2. 逐 step 调用 op_mapper build_command（不调 execute） 3. 校验 op_id 映射、args->flag 映射、required flag 完整性 4. 输出每步构建的命令和校验结果 5. 不校验业务结果（PASS/FAIL），只校验 PARAM_INVALID / OP_NOT_FOUND / VALIDATION_FAILED |
| **输出/结果** | runs/<run-id>/result.json（mode=dry-run，每步含 command + 校验状态） |
| **前置条件** | op_mapper 已安装 |
| **排除情况** | 不实际执行任何写操作；不触发设备连接 |

**处理流程：**
1. case_runner 默认 dry_run=True
2. 逐 step 调用 build_command()，不调用 execute_op()
3. 校验：op_id 是否在映射表 / args->flag 映射是否完整 / required flag 是否缺失 / 参数合法性预检
4. 输出每步构建的 ptm-atomic 命令和校验结果（OK / PARAM_INVALID / OP_NOT_FOUND / VALIDATION_FAILED）
5. 不连接设备，不执行写操作，不校验业务结果

---

### UC-EX-05：用例授权执行---execute 授权门

| 字段 | 内容 |
|------|------|
| **使用角色** | 测试执行工程师（P-EX-01） |
| **触发条件** | dry-run 校验通过后，需要实际执行用例验证业务结果 |
| **输入** | `case_runner.py run --cases-dir cases/upload --devices-yaml devices.yaml --execute --authorized` |
| **处理逻辑** | 1. --execute 触发授权门 2. 逐 step 调用 op_mapper execute_op（dry_run=False, authorized=True） 3. 实际连接设备执行 op 4. 解析 envelope 校验业务结果 5. 遇 STATE_INVALID 自动重连 6. 遇 ConnectTimeout 自动重试 7. 输出 PASS/FAIL/KNOWN_FAIL/ERROR 四态 |
| **输出/结果** | runs/<run-id>/result.json（mode=execute，含 runtime_authorization 审计字段） |
| **前置条件** | dry-run 校验通过；devices.yaml 已配置；运行时环境已就绪（VALIDATION-ENV.yaml 或等价 runtime_authorization） |
| **排除情况** | 不自动授权；设计通过不等于运行授权 |

**处理流程：**
1. 用户显式传 --execute --authorized
2. case_runner 设置 dry_run=False, authorized=True
3. 预登录 DUT 建立共享 session
4. 逐 step 调用 execute_op()，实际连接设备执行
5. 解析 envelope：status=success -> PASS；status=error + known_issue 标记 -> KNOWN_FAIL；status=error 无标记 -> FAIL；解析失败/超时 -> ERROR
6. 遇 STATE_INVALID 自动重连（最多 1 次）；遇 ConnectTimeout 自动重试（TG 最多 3 次，DUT 最多 1 次）
7. 每用例结束逆序清理 mutation ops
8. 全部结束 fw_logout 登出
9. 输出 result.json（含 runtime_authorization 审计字段）

---

### UC-EX-06：失败自动诊断与结果四态分级

| 字段 | 内容 |
|------|------|
| **使用角色** | 测试执行工程师（P-EX-01）、测试经理（P-EX-03） |
| **触发条件** | 用例执行后需要查看结果分级和失败诊断 |
| **输入** | runs/<run-id>/result.json |
| **处理逻辑** | 1. 逐 step 结果分级：success+NONE -> PASS；error+known_issue 标记 -> KNOWN_FAIL；error 无标记 -> FAIL；解析失败/超时 -> ERROR 2. 失败 step 自动诊断：提取 error_type / error_code / reason / details 3. 幂等容错：对象已存在/流不存在/被引用阻止（符合预期）视为 PASS 4. 输出结构化 report.md（含失败诊断摘要） |
| **输出/结果** | report.md（含四态统计 + 失败诊断 + 幂等容错记录） |
| **前置条件** | 用例已执行（dry-run 或 --execute） |
| **排除情况** | 不做 HTML 报告；不自动修复失败 |

**处理流程：**
1. 读取 result.json 中每步的 status / error_type / known_issue
2. 分级判定：
   - status=success + error_type=NONE -> PASS
   - status=error + step 有 known_issue 标记 -> KNOWN_FAIL
   - status=error + step 无 known_issue 标记 -> FAIL
   - 解析失败 / 超时 / UNKNOWN_ERROR -> ERROR
3. 幂等容错检查：
   - fw_config_object + "存在同名" -> PASS（对象已存在）
   - tg_stop_traffic_stream + RESOURCE_NOT_FOUND -> PASS（流不存在）
   - fw_delete_object + eBeingReferenced + expected_result 含"被阻止" -> PASS（符合异常用例预期）
4. 失败诊断：提取 error_type / error_code / reason / details / command
5. 输出 report.md（四态统计 + 失败 step 诊断 + 幂等容错记录）

---

### UC-EX-07：ARP 预热自动清理（warming_up + post_op 引擎强制）

| 字段 | 内容 |
|------|------|
| **使用角色** | 测试执行工程师（P-EX-01） |
| **触发条件** | 用例 step 包含 `warming_up: true` 字段（ARP 预热探测流） |
| **输入** | case_steps 中 step 含 `warming_up: true` + 可选 `post_op` |
| **处理逻辑** | 1. case_runner 识别 warming_up: true 2. 执行主 op（如 tg_start_traffic_stream） 3. 等待 post_delay 4. 引擎强制执行 post_op（tg_stop_traffic_stream）：即使 md 未写 post_op，case_runner 自动补充 5. post_op 使用与主 op 相同的 ports/txport/rxport/name 参数 6. 继续后续 step |
| **输出/结果** | result.json 中该 step 含 warming_up=true + post_op 执行记录 |
| **前置条件** | case_runner 已实现 warming_up/post_op 解析；install.py 规则块已定义 ARP 预热要求 |
| **排除情况** | 不依赖用例 md 手动写 post_op（引擎强制补充） |

**处理流程：**
1. case_runner 解析 step，发现 warming_up: true
2. 执行主 op（如 tg_start_traffic_stream 发送探测流）
3. 等待 post_delay（默认 2s）
4. 引擎强制执行 post_op：
   - 若 md 显式写了 post_op -> 按 md 执行
   - 若 md 未写 post_op -> case_runner 自动补充 tg_stop_traffic_stream（使用主 op 的 ports/txport/rxport/name）
5. 记录 post_op 执行结果到 step 结果
6. 继续后续 step（如正式流 start_traffic_stream）

---

### UC-EX-08：规则固化与重装一致性

| 字段 | 内容 |
|------|------|
| **使用角色** | 测试平台开发者（P-EX-02） |
| **触发条件** | 安装或重装 ptm-te 工作流到 workspace |
| **输入** | `install.py install ptm-te --component full` |
| **处理逻辑** | 1. install.py 安装 skills（case-execution / device-management / policy-route-execution / trex-traffic）到 .claude/skills/ 2. 安装 CLAUDE.md ptm-te-workflow 规则块（≥4 条新规则）3. 规则块内容：TG 路由（DUT/TG 地址从 devices.yaml 读取）/ max_loss（verify_loss 必须传 max_loss）/ ARP 预热（warming_up:true 必须 post_op 清理）/ session 生命周期（login 共享 + logout 清理）4. 安装后验证规则块和 skill 存在 |
| **输出/结果** | .claude/skills/case-execution/ + CLAUDE.md ptm-te-workflow 规则块 + 安装验证通过 |
| **前置条件** | ptm-team 仓库有 skills/case-execution/ 和 script/ptm_team/install.py |
| **排除情况** | 不安装 ptm-atomic 本体；不改 traffic-skill / ngfw-install skill |

**处理流程：**
1. install.py 读取 ptm-team skills/ 目录
2. 复制 case-execution / device-management / policy-route-execution / trex-traffic 到目标 .claude/skills/
3. 生成 CLAUDE.md ptm-te-workflow 规则块，含 ≥4 条新规则：
   - TG 路由：DUT/TG 地址必须从 devices.yaml 读取，禁止硬编码
   - max_loss：tg_verify_traffic_loss 必须传 max_loss 参数
   - ARP 预热：warming_up:true 的 step 必须有 post_op 清理（引擎强制补充）
   - session 生命周期：共享 session + fw_logout 登出
4. 安装后验证：检查 skill 目录存在 + CLAUDE.md 含规则块 + op_mapper validate 通过

---

### UC-EX-09：fw_logout 会话清理

| 字段 | 内容 |
|------|------|
| **使用角色** | 测试执行工程师（P-EX-01） |
| **触发条件** | 用例执行结束（正常或异常），需要清理 DUT session |
| **输入** | case_runner.py 执行结束触发 cleanup |
| **处理逻辑** | 1. case_runner 在全部用例执行结束后触发 cleanup 2. cleanup 调用 op_mapper execute_op(fw_logout, ...) 3. fw_logout 是 op_mapper 新增 op（OP_ID_TO_SUBCOMMAND: fw_logout -> auth, logout）4. 登出共享 session，清理 session 文件 5. 记录登出结果 |
| **输出/结果** | session 文件清理 + result.json 含 logout 状态 |
| **前置条件** | op_mapper 已新增 fw_logout op 映射；共享 session 已建立 |
| **排除情况** | 不在每用例后登出（共享 session 复用）；不读取凭据 |

**处理流程：**
1. case_runner 全部用例执行结束（或异常中断）
2. 触发 cleanup 阶段
3. 调用 execute_op("fw_logout", {}, dut_url, shared_session, dry_run=False, authorized=True)
4. op_mapper 映射 fw_logout -> (auth, logout)，构建 ptm-atomic run auth logout 命令
5. 执行登出，清理 session 文件
6. 记录登出状态到 result.json

---

### UC-EX-10：用例结构化管理（目录/命名/frontmatter/tags）

| 字段 | 内容 |
|------|------|
| **使用角色** | 测试平台开发者（P-EX-02）、测试执行工程师（P-EX-01） |
| **触发条件** | 需要新增用例或迁移现有用例到结构化目录 |
| **输入** | 用例信息：功能模块/子模块/用例组/用例编号/用例名称/级别/组网/预置条件/关键词/tags + case_steps YAML |
| **处理逻辑** | 1. 按 cases/三级(功能模块)/四级(子模块)/五级(用例组)/<用例编号>-<用例名称>.md 结构放置 2. 文件命名：<用例编号>-<用例名称>.md（编号保留组网前缀如 PC-M1-01-01，名称来自 frontmatter 用例名称列） 3. frontmatter 16 列：三级目录/四级目录/五级目录/用例名称/用例编号/用例级别/组网描述/组网约束/预置条件/测试步骤/预期结果/首次创建版本/最后变更版本/关键词/测试类型/是否自动化 4. tags 列：结构化列表如 [策略路由, IPv4, PC, 正向, P0] 5. 关键词列：逗号分隔如 fw_config_policy_route, fw_verify_policy_route, S01 6. 保留 case_steps YAML 块 7. 24 用例从 cases/upload/ 迁移到新结构 |
| **输出/结果** | cases/三级/四级/五级/<编号>-<名称>.md 结构化用例文件，含 frontmatter 16 列 + tags/关键词列 + case_steps YAML |
| **前置条件** | 用例目录结构约定已定义；case_runner 支持解析 frontmatter |
| **排除情况** | 不引入 pydantic 校验 frontmatter；不改变 case_steps YAML 格式 |

**处理流程：**
1. 确定用例的三级/四级/五级目录归属（如 IPv4策略路由/配置管理/策略配置）
2. 按命名约定创建文件：<用例编号>-<用例名称>.md
3. 填写 frontmatter 16 列（必填/可选见 DQ-05）
4. 填写 tags 列（结构化列表）和关键词列（逗号分隔）
5. 保留或补充 case_steps YAML 块
6. 24 用例从 cases/upload/ 批量迁移到新结构（含重命名 + frontmatter 补全 + tags 标注 + ARP 预热校验）

---

### UC-EX-11：多环境执行（环境文件驱动 ${ENV.*}）

| 字段 | 内容 |
|------|------|
| **使用角色** | 测试执行工程师（P-EX-01）、测试平台开发者（P-EX-02） |
| **触发条件** | 同一用例需要在多个环境（如 link3 三链路 / link4 四链路）下执行，不想改用例 |
| **输入** | `case_runner.py run --case-file cases/.../PC-COMB-M4-01-01-xxx.md --env-file topology-link3.yaml --devices-yaml devices.yaml --execute --authorized`；用例 case_steps 的 args 中环境相关参数使用 ${ENV.*} 占位符（如 `${ENV.dut.port1}` / `${ENV.tg.port1}` / `${ENV.tg.url}` / `${ENV.dut.next_hop}`） |
| **处理逻辑** | 1. case_runner 加载 --env-file（环境文件 YAML，含 nodes.dut1.interfaces / nodes.tg1.trex_api_url / nodes.dut1.url 等） 2. login 后按 nodes.dut1.interfaces 自动 fw_update_interface 预配置 DUT 接口 IP 3. 逐 step 执行前，resolve_env_refs 扫描 args 中的 ${ENV.*} 占位符，从环境文件解析为物理值 4. 测试意图参数（max_loss / object_name / template name）保持字面值 5. 未含 ${ENV.*} 的字面值 args 原样透传（向后兼容） 6. TREX_API_URL 从环境文件 nodes.tg1.trex_api_url 注入 _build_exec_env，devices.yaml tg.api_server 作 fallback 7. 用例结束后逆序清理：先清理 mutation ops，再清理预配置的 DUT 接口 8. 换环境只需换 --env-file（link3 -> link4），用例不改一行 |
| **输出/结果** | runs/<run-id>/result.json（含 env_file 路径 + 解析后的物理值参数）+ report.md |
| **前置条件** | 环境文件 YAML 格式正确；用例 args 已用 ${ENV.*} 占位符替换环境相关参数；devices.yaml 已配置（fallback） |
| **排除情况** | 不直接调 TG REST API（TG 操作仍经 ptm-atomic run tg trex <action>）；不改 dry-run 默认门和 --execute 授权门；不强制改造现有无 ${ENV.*} 的用例 |

**处理流程：**
1. case_runner 加载 --env-file（如 topology-link3.yaml），解析 YAML
2. 预登录 DUT 建立共享 session（fw_login_web_management）
3. 按 nodes.dut1.interfaces 自动执行 fw_update_interface 配置 3 个接口 IP（login 后、用例主体前）
4. 逐 step 执行前，resolve_env_refs 扫描 args 中的 ${ENV.*}：
   - `${ENV.dut.port1}` -> 环境文件 dut.port1 物理值（如 "GE0_3"）
   - `${ENV.tg.port1}` -> 环境文件 tg.port1 物理值（如 "2_3"）
   - `${ENV.tg.url}` -> 环境文件 tg.url 物理值（如 "http://10.113.55.170:8000"）
   - `${ENV.dut.next_hop}` -> 环境文件 dut.next_hop 物理值（如 "192.168.102.1"）
5. 测试意图参数（max_loss: 0 / object_name: "OBJ-SRC-192" / template: "pr-pc-comb-m4-01-09"）保持字面值
6. 未含 ${ENV.*} 的字面值 args 原样透传
7. TREX_API_URL 从环境文件 nodes.tg1.trex_api_url 注入 _build_exec_env（tg_* op 执行时）
8. 用例结束后逆序清理：mutation ops -> 预配置的 DUT 接口
9. 换环境：`--env-file topology-link4.yaml`，同一用例不改一行

<!-- coverage-checklist: begin -->
## 附录：覆盖自检表

| 维度 ID | 维度名称 | 状态 | 涉及场景 | 备注 |
|---------|---------|------|---------|------|
| D1 | 用户维度 | 已覆盖 | UC-EX-01..11 | 4 画像（执行工程师/平台开发者/测试经理/安全合规） |
| D2 | 任务维度 | 已覆盖 | UC-EX-01..11 | 建模/执行/dry-run/授权/诊断/ARP清理/规则固化/登出/用例结构化/多环境执行 |
| D3 | 动机维度 | 已覆盖 | UC-EX-01..11 | 零代码新增用例/硬编码消除/重装一致性/安全默认/四态分级/用例结构化管理/换环境不改用例 |
| D4 | 时间维度 | 已覆盖 | UC-EX-02,05,07,10,11 | 批量执行/单用例执行/ARP预热时序/post_delay/retry轮询/目录迁移/多环境切换 |
| D5 | 环境维度 | 已覆盖 | UC-EX-02,04,05,08,10,11 | ptm-te workspace/ptm-team canonical/跨仓库/install.py/三级目录结构/多环境文件驱动 |
| D6 | 方式维度 | 已覆盖 | UC-EX-02,03,04,05,10,11 | 目录glob/单文件/--tag/--keyword/dry-run/--execute/用例结构化/${ENV.*}环境文件驱动 |
| D7 | 异常维度 | 已覆盖 | UC-EX-05,06,07,09,10,11 | STATE_INVALID重连/ConnectTimeout重试/幂等容错/known_issue/登出失败/命名冲突/占位符解析失败 |
| D8 | 集成维度 | 已覆盖 | UC-EX-01,02,08,10,11 | devices.yaml/op_mapper/install.py/trex-traffic/用例目录结构/环境文件 集成 |
<!-- coverage-checklist: end -->
