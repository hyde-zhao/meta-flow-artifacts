---
change_id: CR-027-ptm-te-v1.1-quickfix
workflow_id: WF-PTM-TEAM-20260520-001
created_at: "2026-07-13T18:00:00+08:00"
created_by: host-orchestrator（主进程）
status: "closed"
implementation_status: delivered
closed_at: "2026-07-13T18:40:00+08:00"
workflow_mode: fast-lane
engagement_mode: production
impact_level: low
complexity: simple
rollback_to: requirement-clarification
approval_source: CR-024 T-01 follow-up + nxp290-171 实战 PC-DATA-LC-M10-01-01 评估
depends_on:
  - CR-025（closed，args 对齐 source_network，F-01 适配其产出）
  - CR-024（closed，ptm-te v1 基线）
plan_ref: 无（fast-lane，改动面小，本 CR 即计划）
cross_references:
  - CR-026（并行，ptm-tde PC 契约强化，文件零冲突；CR-027 不碰 ptm-tde 文件，CR-026 不碰 op_mapper）
  - CR-024（ptm-te v1 基线）
  - CR-025（args 对齐前置，已 closed）
lifecycle_status: "closed"
---

# CR-027 - ptm-te v1.1 快速修复（适配 CR-025 + v1 缺陷/增强）

## 变更请求摘要

基于 nxp290-171 实战执行 PC-DATA-LC-M10-01-01 暴露的 12 个改进点评估，ptm-te 侧需整改 7 项。
其中 **F-01 是 CR-025 后的必需适配**（ptm-tde 已产出 `source_network`，op_mapper 仍用 `src_addr` 作 key，会直接报 OP_NOT_FOUND），
其余 6 项为 v1 缺陷（依赖未声明 / session 路径 / 代理环境）与 v1 增强（授权审计 / snapshot diff / 参数预检）。

本 CR 为 **fast-lane**：低风险轻量实现，改动面集中在 ptm-te 私有文件（agent md + 2 个 skill 脚本 + .gitignore），
无架构变更、无外部接口变更、无 Story 拆解。跳过 CP1-CP5，保留 CP0/CP6/CP7/CP8 追溯证据。

## 7 项整改清单

| # | 项 | 性质 | 优先级 | 触发依据 |
|---|----|------|--------|---------|
| F-01 | op_mapper args key 对齐 CR-025（source_network） | 适配 | P1 必修 | CR-025 closed 后 op_mapper 与 ptm-tde 产出失配 |
| P1-6 | device-connection 依赖声明（paramiko/pyyaml） | v1 缺陷 | P1 | SKILL.md 声明 >=3.9,<3.13 但无依赖锁定 |
| P2-8 | session 路径规范（session-<run-id>.json 隔离） | v1 缺陷 | P2 | agent md 默认路径无 run-id 隔离，多 run 串扰 |
| P2-9 | --execute 授权落盘审计（runtime_authorization） | v1 增强 | P2 | authorized 参数无落盘记录，不可审计 |
| P2-10 | snapshot diff 自动产出（snapshot-diff.json） | v1 增强 | P2 | [8] 提"diff 摘要"但无自动产物 |
| P2-11 | 参数合法性预检（validate_args） | v1 增强 | P2 | op_mapper 只检 required flag，不检参数值合法性 |
| P2-12 | 代理环境 NO_PROXY 适配 | v1 缺陷 | P2 | WSL2 代理环境下 ptm-atomic HTTPS 可能误走代理 |

## 逐项整改方案

### F-01 · op_mapper args key 对齐 CR-025【P1 必修】

| 维度 | 内容 |
|------|------|
| 现状 | `op_mapper.py` L74-128 `ARGS_TO_FLAGS` key 用 `src_addr/dst_addr/next_hop`；CR-025 后 ptm-tde 产出已是 `source_network/dst_network/next_hop_ip` |
| 整改 | key 对齐 op yaml params：`src_addr`->`source_network`、`dst_addr`->`dst_network`、`next_hop`->`next_hop_ip`（`in_interface`/`type`/`id`/`moveid` 等不变） |
| 主选 | 保留显式映射表，仅改 key 命名（最稳；`type`->`--policy-route-type` 等特例仍需显式） |
| 备选 | 退化为 `_to_flag(arg)=arg.replace('_','-')` 机械转换 + `FLAG_OVERRIDES` 例外表。**何时切换**：P0-1 扩展到几十个 op 后显式表臃肿时 |
| 文件 | `skills/policy-route-execution/scripts/op_mapper.py` L74-128；`agents/ptm-te.md` 三层映射示例 L231-247 同步 |
| 验证 | `op_mapper.py map --op-id fw_config_policy_route --args '{"source_network":"OBJ_SRC_WEB","in_interface":"GE0_12"}'` 输出含 `--source-network`；`validate` PASS |

### P1-6 · device-connection 依赖声明【v1 缺陷】

| 维度 | 内容 |
|------|------|
| 现状 | 无 pyproject.toml；SKILL.md 声明 `>=3.9,<3.13` 但 paramiko/pyyaml 未锁；根 pyproject.toml 是安装器项目不含 skill 依赖 |
| 整改 | SKILL.md 所有脚本调用改为 `uv run --python 3.12 --with "paramiko>=3.0,<4.0" --with "pyyaml>=6.0" python scripts/...`；新增"依赖声明"章节 |
| 主选 | `--with` 临时依赖（skill 内不新增 pyproject.toml，避免安装到目标项目污染其依赖空间） |
| 备选 | device-connection 内放 pyproject.toml + `uv run --project skills/device-connection`。**何时切换**：`--with` 出现版本漂移问题时 |
| 文件 | `skills/device-connection/SKILL.md`（调用示例 + 新增依赖声明章） |
| 验证 | `uv run --python 3.12 --with paramiko --with pyyaml python skills/device-connection/scripts/ssh_exec.py --help` 成功 |

### P2-8 · session 路径规范【v1 缺陷】

| 维度 | 内容 |
|------|------|
| 现状 | agent md L82 默认 `~/.local/state/ptm-atomic/ngfw/session.json`（无 run-id 隔离） |
| 整改 | 改为 `session-<run-id>.json`；op_mapper.py CLI 默认值同步；.gitignore 补 `**/session*.json` 防入库 |
| 主选 | session-<run-id>.json 隔离 |
| 备选 | 固定 session.json。**何时切换**：单 run 串行无并发时（不推荐） |
| 文件 | `agents/ptm-te.md` L82 + L256；`op_mapper.py` L1002 CLI 默认值；`.gitignore` |
| 验证 | agent md [3] 含 `session-<run-id>.json`；`git check-ignore` 对 session 文件命中 |

### P2-9 · --execute 授权落盘审计【v1 增强】

| 维度 | 内容 |
|------|------|
| 现状 | `execute_op` 有 `authorized` 参数但无落盘审计 |
| 整改 | dry_run=False 时 envelope 加 `runtime_authorization`（who/scope/authorized_at/reason）；result.json 汇总 `authorizations[]` |
| 主选 | envelope 内嵌（随 exec-log.jsonl 落盘，审计链完整） |
| 备选 | 独立 `authorization-ledger.jsonl`。**何时切换**：需跨 run 查询授权历史时 |
| 文件 | `op_mapper.py` `_build_envelope`/`execute_op`；`agents/ptm-te.md` envelope 契约 L155-168 补字段 |
| 验证 | dry_run=False 执行后 envelope 含 `runtime_authorization` |

### P2-10 · snapshot diff 自动产出【v1 增强】

| 维度 | 内容 |
|------|------|
| 现状 | [8] 提"快照 diff 摘要"但无自动产物 |
| 整改 | [8] 产出 `runs/<run-id>/snapshot-diff.json`（before/after 4 维度对比）；report.md 引用 |
| 主选 | collect_sysinfo.py 新增 `--diff <before-dir> <after-dir>` 子命令 |
| 备选 | agent 编排时 in-process 对比。**何时切换**：diff 逻辑简单且不跨 skill 复用时 |
| 文件 | `skills/device-connection/scripts/collect_sysinfo.py`（新增 diff 函数）；`agents/ptm-te.md` [8] + 运行目录表补 `snapshot-diff.json` |
| 验证 | 执行后 `runs/<run-id>/snapshot-diff.json` 存在；report.md 含 diff 摘要 |

### P2-11 · 参数合法性预检【v1 增强】

| 维度 | 内容 |
|------|------|
| 现状 | op_mapper 只检 required flag，不检参数值合法性 |
| 整改 | 新增 `validate_args(op_id, args)`：对象名格式（`OBJ_*` 正则）、必填 flag 非空、引用对象存在性（config 前 verify 查询）；build_command 前调用，失败报 `eParamIllegal`/`eObjParse` |
| 主选 | op_mapper 内 `validate_args`（与映射表同文件，单点维护） |
| 备选 | 独立 `precheck.py`。**何时切换**：预检规则跨 skill 复用时 |
| 文件 | `op_mapper.py` 新增 `validate_args` + `build_command`/`execute_op` 调用；`agents/ptm-te.md` [4] 补预检说明 |
| 验证 | `op_mapper.py execute --args '{"source_network":"INVALID FORMAT"}'` 预检报 `eParamIllegal` 不执行 |
| 注 | preconditions 消费待 CR-026 契约定稿后补充（CR-026 新增 atomic_op.preconditions） |

### P2-12 · 代理环境 NO_PROXY 适配【v1 缺陷】

| 维度 | 内容 |
|------|------|
| 现状 | 无 NO_PROXY 处理，WSL2 代理环境下 ptm-atomic HTTPS 可能误走代理 |
| 整改 | `execute_op` 调 ptm-atomic 时传 `env={**os.environ, "NO_PROXY": <设备IP>}`；ssh_exec/collect_sysinfo 同理；agent md [2] 记录代理适配 |
| 主选 | subprocess 显式传 env（代码层保证，不依赖用户预置） |
| 备选 | 要求用户预置 `NO_PROXY` 环境变量。**何时切换**：运行环境固定且统一预置时 |
| 文件 | `op_mapper.py` `execute_op`/`_reconnect_and_retry` 的 `subprocess.run` 加 env；`ssh_exec.py`/`collect_sysinfo.py`；`agents/ptm-te.md` [2] |
| 验证 | 设 `HTTPS_PROXY` 后执行，ptm-atomic 调用日志显示直连设备 IP 未走代理 |

## 修改文件清单

| 文件 | 变更项 | 变更类型 |
|------|--------|---------|
| `skills/policy-route-execution/scripts/op_mapper.py` | F-01 args key + P2-9 授权字段 + P2-11 validate_args + P2-12 NO_PROXY env | 脚本扩展 |
| `skills/device-connection/SKILL.md` | P1-6 依赖声明 + 调用示例 | 文档扩展 |
| `skills/device-connection/scripts/collect_sysinfo.py` | P2-10 diff 子命令 + P2-12 NO_PROXY | 脚本扩展 |
| `skills/device-connection/scripts/ssh_exec.py` | P2-12 NO_PROXY env | 脚本扩展 |
| `agents/ptm-te.md` | F-01 示例 + P2-8 session 路径 + P2-9 envelope 字段 + P2-10 目录 + P2-11 预检 + P2-12 代理 | 文档扩展 |
| `.gitignore` | P2-8 补 `**/session*.json` | 配置追加 |

### 文档处理决策

| 受影响对象 | 处理方式 | 旧基线保留方式 |
|-----------|---------|--------------|
| `agents/ptm-te.md` | 原文档增量更新 | 追加 `## 修订记录`，旧约定保留在 git 历史 |
| `op_mapper.py` | 脚本扩展 | git 历史 |
| `device-connection/SKILL.md` | 原文档增量更新 | 追加 `## 修订记录` |
| `collect_sysinfo.py` / `ssh_exec.py` | 脚本扩展 | git 历史 |
| `.gitignore` | 追加 | git 历史 |

## 五维度影响分析

### 1. 需求影响
| 维度 | 影响 |
|------|------|
| 现有需求 | 不变。ptm-te 编排流程 [1]-[8] 不变，仅增强各步骤健壮性 |
| 新增需求 | F-01 适配 CR-025 + 6 项缺陷/增强 |
| 需求冲突 | 无。无存量 run，不破坏基线 |

### 2. 设计影响
| 维度 | 影响 |
|------|------|
| 架构决策 | 无。CR-024 ADR-01（op_mapper 承载映射）不变；F-01 仅改 key 命名对齐 CR-025 |
| HLD 影响 | 无。ptm-te 自身架构不变 |
| ADR 影响 | 无新增 ADR；CR-024 ADR-01 适配 CR-025 后的 args 命名 |

### 3. Story/实现影响
| 维度 | 影响 |
|------|------|
| 受影响文件 | 6 文件（1 agent md + 3 skill 脚本/SKILL + 1 .gitignore + 本 CR） |
| 复杂程度 | 简单。脚本增强 + 文档同步，无逻辑架构变更 |
| 依赖 | 无 Story 拆解 |

### 4. 安全/权限影响
| 维度 | 影响 |
|------|------|
| 凭据 | 无。session 路径隔离增强隔离性，不引入新凭据 |
| 运行时授权 | P2-9 增强授权审计（落盘），不改变授权门控（仍 ADR-04 dry-run 默认门） |
| 风险 | 极低。NO_PROXY 防误走代理，授权审计增强可追溯性 |

### 5. 交付影响
| 维度 | 影响 |
|------|------|
| 安装器 | 无。ptm-te 已安装，脚本增强不影响安装 |
| 文档 | `agents/ptm-te.md` + `device-connection/SKILL.md` 增量更新 |
| 向后兼容 | 完全兼容。args key 对齐 CR-025 产出（无旧 PC）；session 路径 run-id 隔离是增强 |

## 文件所有权冲突分析

| 现有 CR | 状态 | 冲突 | 处理 |
|---------|------|------|------|
| CR-026 | open（in_progress） | **无**。CR-026 改 `agents/ptm-tde.md` + 4 ptm-tde skill + `gate-spec.md` + `run_checkpoint.py`；CR-027 改 `agents/ptm-te.md` + `op_mapper.py` + `device-connection/`。文件完全正交 | 并行推进 |
| CR-025 | closed | 无（CR-025 改 ptm-tde args 命名，CR-027 适配 op_mapper，不同文件） | F-01 适配 CR-025 产出 |
| CR-024 | closed | 无（CR-027 是 CR-024 v1.1 增强，同 agent 文件，CR-024 已关闭） | 增强不破坏 v1 基线 |

**与 CR-026 并行判定**：文件零重叠，依赖不阻塞（F-01 依赖 CR-025 closed 不依赖 CR-026；P2-11 preconditions 消费部分待 CR-026 定稿，基础预检可先做）。满足单写规则，可并行。

## 验证方法

- `validation_mode=static-only`（agent md / SKILL.md 契约）+ `dry-run-only`（脚本逻辑，不触发 --execute）
- F-01：`op_mapper.py validate` PASS + `map` 子命令用 `source_network` key 输出正确 flag
- P1-6：`uv run --python 3.12 --with paramiko --with pyyaml python ssh_exec.py --help` 成功
- P2-8：agent md [3] 含 `session-<run-id>.json`；`.gitignore` 含 `**/session*.json`
- P2-9：dry_run=False envelope 含 `runtime_authorization`
- P2-10：collect_sysinfo.py `--diff` 子命令产出 snapshot-diff.json
- P2-11：非法参数预检报 `eParamIllegal` 不执行
- P2-12：subprocess 传 NO_PROXY env
- 安装器 dry-run 验证 ptm-te 安装不受影响

## 风险与回退

| 风险 | 等级 | 缓解 | 回退路径 |
|------|------|------|---------|
| F-01 改 key 后 op_mapper 测试用例失配 | 低 | 同步改测试 args key | 改回旧 key（但与 CR-025 产出失配，不推荐） |
| P2-11 validate_args 误报阻断正常 op | 低 | 预检规则保守（只检明确非法格式），引用存在性失败降级 warning 不阻断 | 移除 validate_args 调用 |
| P2-12 NO_PROXY 误覆盖用户代理配置 | 低 | 仅设 NO_PROXY 不动 HTTP_PROXY/HTTPS_PROXY | 移除 env 传参 |
| 与 CR-026 并行时 preconditions 契约未定稿 | 低 | P2-11 基础预检先做，preconditions 消费留 follow-up | 补充 CR-027 v1.2 |

## Checkpoint Index

| CP | 类型 | 说明 | 状态 |
|----|------|------|------|
| CP0 | auto | 原始请求受理（本 CR 创建） | pass |
| CP1 | n/a | 场景完备（fast-lane，无场景需求变更） | n/a |
| CP2 | n/a | fast-lane 跳过 | n/a |
| CP3 | n/a | fast-lane 跳过 HLD/蓝图 | n/a |
| CP4 | n/a | fast-lane 跳过 Story 拆解 | n/a |
| CP5 | n/a | fast-lane 跳过 LLD | n/a |
| CP6 | auto | 编码完成（6 文件修改，7 项） | pass |
| CP7 | auto | 验证完成（static-only + dry-run-only） | pass |
| CP8 | auto | 交付就绪 | pass |

> fast-lane 模式：低风险轻量实现，跳过 CP1-CP5，保留 CP0/CP6/CP7/CP8 追溯证据与终验摘要。

## 终验摘要

CR-027 ptm-te v1.1 快速修复已交付。7 项整改全部通过 static + dry-run 验证：

| 项 | 验证结果 | 证据 |
|----|---------|------|
| F-01 | ✅ | op_mapper validate PASS；source_network key -> --source-network；旧 src_addr 失效 |
| P1-6 | ✅ | device-connection/SKILL.md 5 处 paramiko>=3.0,<4.0 版本约束 + --with 调用 |
| P2-8 | ✅ | agent md session-<run-id>.json；.gitignore **/session*.json 命中 |
| P2-9 | ✅ | _build_envelope 含 runtime_authorization 参数；execute_op dry_run=False 填充 |
| P2-10 | ✅ | collect_sysinfo --diff 产出 snapshot-diff.json；cpu.changed=true / memory.changed=false |
| P2-11 | ✅ | validate_args 拦截非法对象名 + 占位符；PARAM_INVALID error_type |
| P2-12 | ✅ | _build_exec_env NO_PROXY 含设备 IP；3 处 subprocess 传 env |

**事实纠正（实施中）**：P2-12 原方案含 ssh_exec.py/collect_sysinfo.py NO_PROXY，实施时确认 SSH/Telnet 是 paramiko TCP 直连，不读 HTTP_PROXY 环境变量，故取消（仅 op_mapper 调 ptm-atomic HTTPS 需 NO_PROXY）。记录于 CP6/CP7 result。

**并行交付**：与 CR-026（ptm-tde PC 契约强化）文件零冲突并行完成。CR-026 改 ptm-tde 私有文件，CR-027 改 ptm-te 私有文件，git 改动可分离提交。

**不授权项**：本 CR 仅文档/脚本静态增强，不含真实设备运行授权；dry_run 默认门（ADR-04）不变，--execute 仍需单次授权。

## 后续 CR 候选

| 编号 | 描述 | 优先级 | 状态 |
|------|------|--------|------|
| CR-028 | ptm-te v2 op_mapper 扩展（op 覆盖矩阵 + operation-log/object/interface 族 + op_id 容错） | P1 | candidate（依赖 CR-026 op_id 规范定稿） |
| F-02 | P2-11 preconditions 消费（待 CR-026 契约定稿） | P2 | candidate |

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|------|------|--------|---------|
| v1.0 | 2026-07-13 | host-orchestrator（主进程） | CR 创建，覆盖 F-01 + P1-6 + P2-8/P2-9/P2-10/P2-11/P2-12 七项整改，与 CR-026 并行 |
