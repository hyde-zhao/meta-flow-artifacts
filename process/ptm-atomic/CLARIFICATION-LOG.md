# CLARIFICATION-LOG

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| v0.1 | 2026-05-15 | meta-po | 初始化阶段零事实核对记录。 |
| v0.2 | 2026-05-15 | meta-po | 补齐根目录 `AGENTS.md`，作为仓库级 Agent 规则文件。 |
| v0.3 | 2026-05-15 | meta-po | 根据 `README.md` 修正交付面：不使用 `delivery/`，交付物落在仓库原生目录。 |
| v0.4 | 2026-05-15 | meta-po | 记录本轮目标：将 `.input/capacity/` 与 `.input/ngfw-install/` 转译为 atomic-ops 产物。 |
| v0.5 | 2026-05-15 | meta-pm | 追加阶段零调研发现、场景发现摘要与需求阻塞项。 |
| v0.6 | 2026-05-15 | meta-po | 采用安全默认决策关闭高风险初始化与 schema/CLI 扩展阻塞项。 |
| v0.7 | 2026-05-15 | meta-po | 撤销未确认的安全默认推进，恢复人工确认门控。 |
| v0.8 | 2026-05-15 | meta-pm | 记录 `CR-001` 的场景纠偏结果：按真实业务链路重建安装、初始化、登录、配置与验证基线，并同步重整需求。 |
| v0.9 | 2026-05-18 | meta-po | 补齐 CP0/CP1/CP2 检查点文档，并发起 CP2 人工确认稿。 |
| v1.0 | 2026-05-18 | meta-po | CP2 人工确认通过，`USE-CASES.md` 与 `REQUIREMENTS.md` 已确认为 HLD 输入基线。 |

## 调研发现（2026-05-15）

### 现有可复用资源

- 当前仓库已具备 `schemas/atomic-op.schema.yaml`、示例 atom `atoms/fw/fw_verify_acl_rule.yaml`、`packages/` 过滤视图和 `atomic-ops` 本地只读 CLI，可直接作为迁移目标契约。
- `process/INPUT-INDEX.md` 已完成对 `.input/capacity/` 与 `.input/ngfw-install/` 的高价值索引，可作为需求与场景的证据目录。
- `.input/capacity/` 提供了 API 模板、产品容量参数、接口/对象/ACL/策略路由/静态路由/NAT/带宽/黑白名单/SSL VPN/升级日志检查等能力来源，适合拆成多个配置类或验证类 atom 候选。
- `.input/ngfw-install/` 提供了串口安装 CLI、原始需求和参数约束，适合提炼为安装前校验、安装后健康检查和高风险初始化边界，不适合直接复制为交付实现。
- 元工作流侧可复用 `use-case-discovery` 与 `requirement-extraction` 两个 Skill 的结构约束，用于生成 `USE-CASES.md` 与 `REQUIREMENTS.md`。

### 平台能力约束

- 当前仓库的产品交付面固定为 `atoms/`、`schemas/`、`packages/`、`docs/`、`src/atomic_ops/`、`scripts/`、`pyproject.toml`、`uv.lock`，`delivery/` 不可作为默认落盘面。
- `atomic-ops` 当前 CLI 是离线优先的同步与查询工具，不连接真实设备，不管理凭据，不执行 atom，也不创建独立日志目录。
- `schemas/atomic-op.schema.yaml` v1 只描述单个 atom 的参数、返回、幂等、超时和错误类型，不包含风险等级、人工确认门控、凭据策略或执行模式字段。
- `device_type` 当前只允许 `fw`、`tg`、`mock`、`sw`；新增设备类型不能在需求阶段默认假定成立。
- `packages/*.yaml` 只允许维护 `op_id` 引用，不复制 atom 文件或外部脚本。

### 对需求的初步影响

- `capacity` 派生能力可优先转化为配置类 atom、验证类 atom、日志检查 atom，以及少量 package 与文档支持需求。
- `ngfw-install` 派生能力必须拆分处理：安装前校验和安装后健康检查可作为 atom 候选，高风险初始化步骤需要额外 schema/CLI/docs 治理判断，卸载/安装/重启/删包等破坏性脚本实现禁止直接交付。
- 由于 schema v1 缺少风险门控字段，`ngfw-install` 的可变更型步骤是否进入 atom 范围构成本轮 requirement-clarification 的主要 BLOCKING 未决项。

## 阶段零事实核对

| 检查项 | 结果 |
|---|---|
| 仓库根目录 | `/home/hyde/projects/ptm-atomic` |
| 项目定位 | 防火墙测试原子操作规范仓库与本地消费 CLI。 |
| Python 管理 | 使用 `uv`；`pyproject.toml` 是依赖声明源，`uv.lock` 是锁定结果。 |
| 当前源码 | `src/atomic_ops/`，包含 CLI 入口、git 同步、本地 repository 读取、metadata、错误处理和子命令。 |
| 当前数据面 | `schemas/`、`atoms/`、`packages/`、`docs/`。 |
| 当前参考输入 | `.input/capacity/` 与 `.input/ngfw-install/`，包含其他项目的 Python 脚本、Skill/CLI 参考、环境配置、文档、IDE 配置和 `.venv` 文件。 |
| 参考输入边界 | `.input/` 只作为需求和实现参考，不直接纳入当前源码、运行态文档或交付物。 |
| 初始化前运行态 | 未发现 `process/`、`checkpoints/`、`delivery/`。 |
| 初始化前未提交变更 | `.gitignore` 已修改，来源不属于本次初始化。 |
| 仓库级 Agent 规则 | 已创建根目录 `AGENTS.md`，内容来自初始化输入中的 SCOPE-Pack 规则与 Agent Memory Policy。 |
| README 交付模型 | 本仓库同时交付原子操作规范库和 Python CLI；交付目录为 `atoms/`、`schemas/`、`packages/`、`docs/`、`src/atomic_ops/`、`scripts/`、`pyproject.toml`、`uv.lock`。 |
| `delivery/` 处理 | `delivery/` 不是本项目默认交付目录；先前按通用模板创建的空目录已移除。 |
| `.input/capacity/` 处理 | 作为防火墙配置容量、场景测试和升级日志检查参考；敏感凭据、Windows 路径、虚拟环境和直连设备脚本不得直接进入交付物。 |
| `.input/ngfw-install/` 处理 | 作为 NGFW/TGFW 卸载安装、健康检查和初始化流程参考；破坏性脚本、默认凭据、FTP 凭据和日志写入行为不得直接进入交付物。 |

## 参考资料处理策略

- 可以阅读 `.input/capacity/` 中的测试脚本和文档来理解外部项目如何描述网络、防火墙、路由、升级或场景测试。
- 可以阅读 `.input/ngfw-install/` 中的 Skill、原始需求和 CLI 帮助来理解卸载安装流程，但该脚本默认视为高风险黑盒参考。
- 不复制 `.input/capacity/.venv/`、`.input/capacity/.idea/`、`.input/capacity/.vscode/` 作为当前项目产物。
- 从 `.input/` 提炼需求时，应明确标注来源和转化结果，避免把参考项目的实现细节误认为当前项目约束。

## 待澄清问题

| ID | 状态 | 问题 | 影响 |
|---|---|---|---|
| Q-001 | RESOLVED 2026-05-15 | 本轮首个目标是将 `.input/capacity/` 与 `.input/ngfw-install/` 转译为符合本仓库规范的 atomic-ops 产物。 | 进入 `USE-CASES.md` 与 `REQUIREMENTS.md` 产出。 |
| Q-002 | RESOLVED 2026-05-15 | 两个输入项目都作为参考：`capacity` 参考配置容量/场景/日志检查，`ngfw-install` 参考安装/健康检查/初始化流程。 | 阶段零读取范围已写入 `process/INPUT-INDEX.md`。 |
| Q-003 | PARTIAL 2026-05-15 | 原子操作优先复用现有 `fw` 与 `tg` 设备类型；是否新增类型由 `meta-pm` 与后续 HLD 根据场景判断。 | 影响 schema、命名规范、示例 atom 和 package 设计。 |
| Q-004 | DEFERRED_TO_HLD 2026-05-18 | 破坏性安装类能力是否只作为描述型 atom，还是需要扩展 schema/CLI 表达风险等级、执行模式或人工确认？ | CP2 已允许作为 HLD 设计输入继续评估，不得在需求阶段默认实现执行能力。 |
| Q-005 | PARTIAL 2026-05-15 | 升级日志检查是否只建模为离线日志分析 atom，而不包含 SFTP 拉取日志实现。 | 影响 `capacity` 升级能力的 atom 边界与文档要求。 |
| Q-006 | RESOLVED 2026-05-18 | 安装后初始化步骤中的关闭验证码、启用 SSH、恢复 license、配置管理路由、修改默认密码，哪些允许进入原子操作建模，哪些只能保留为文档化人工步骤。 | 当前基线仅包含关闭验证码、配置管理路由、统一改密；启用 SSH 与恢复 license 不进入当前主基线，后续扩展需新 CR。 |

## 场景发现摘要（2026-05-15）

- 当前模式字段判定为 `engagement_mode=production`、`scenario_subject_type=target-artifact`、`scenario_subject_id=atomic-ops-capacity-ngfw-migration`。
- 目标交付形态判定为 `tool`，治理方式判定为 `review-gated`，审查强度为 `strict`，原因是需求同时触及 schema/CLI/docs 和高风险运维能力边界。
- 已形成四个基线场景：`capacity` 配置类 atom 建模、`capacity` 验证与升级日志检查建模、`ngfw-install` 前后置校验建模、以及 schema/CLI/docs 支持缺口识别。
- `capacity` 与 `ngfw-install` 已明确分流：前者以可复用配置和验证为主，后者仅提炼前置检查、后置健康检查和受控初始化边界，破坏性安装实现不进入正式交付面。
- `USE-CASES.md` 最初创建为 `draft`；已于 2026-05-18 通过 CP2 人工确认，当前版本为 confirmed。

## 需求提取摘要（2026-05-15）

- 已创建 `process/REQUIREMENTS.md draft`，按 capacity 配置、capacity 验证、ngfw 前后置校验、schema/CLI/docs 支持和禁止直接交付边界拆分功能需求、约束需求和非功能需求。
- 需求中已显式记录交付分流矩阵，区分 atom YAML 候选、需要 schema/CLI/docs 支持的能力，以及禁止直接交付的破坏性脚本实现。
- `ready_for_design` 已更正回 `false`。此前由 Agent 直接采用安全默认关闭 `Q-004` 与 `Q-006` 属于流程错误；需求确认必须由人工检查点完成。

## CR-001 场景纠偏摘要（2026-05-15）

### 本轮修订结论

- 已按用户纠正把当前主基线切换为六段业务链路：防火墙安装、安装后初始化、使用目标密码登录并记录登录状态、配置前登录态校验与失效重登、基于 `capacity` 接口执行配置、安装/配置后的健康检查与诊断收口。
- 原 `UC-01..UC-04` 和原 `R-*` 需求未删除，已在 `process/USE-CASES.md` 与 `process/REQUIREMENTS.md` 中降级为历史基线，并建立 `CR-001` 映射关系。
- 初始化场景当前仅保留用户明确指定的动作：串口登录、关闭验证码、配置管理路由、统一把 Web 管理密码修改为 `Ngfw@123`。
- 登录状态失效后的重登已纳入当前主基线；凭据/状态记录边界已转化为显式约束，禁止记录原始密码、完整 token 和其他敏感认证载荷。
- 安装后健康检查、配置后验证/诊断已纳入主基线；自动回滚因参考依据不足，暂不纳入当前主基线。

### 本轮参考事实

- `.input/ngfw-install/scripts/ngfwinstall_tool.py` 的结构性事实已用于确认安装、重启等待、VPP/Agent/HTTPS 健康检查、初始化顺序和改密后验证登录链路。
- `.input/capacity/src/shared/device.py` 与 `src/api_service/` 已用于确认当前配置接口来源覆盖接口、对象、ACL/策略、策略路由、静态路由、NAT、带宽、黑白名单、SSL VPN、动态路由等接口族。
- 依据用户要求，本轮没有从 `.input` 复制真实设备 IP、token、默认密码、`env.yaml` 或其他敏感运行时数据。

### 仍需人工确认的问题

| ID | 级别 | 问题 | 当前处理 |
|---|---|---|---|
| CRQ-001 | RESOLVED 2026-05-18 | 初始化是否还要追加启用 SSH、恢复 license 等参考动作 | 当前保持最小初始化集合；启用 SSH、恢复 license 不进入当前基线，后续扩展需新 CR |
| CRQ-002 | DEFERRED_TO_HLD 2026-05-18 | 登录状态记录的后续持久化边界应如何设计，才能复用又不落敏感信息 | HLD 必须给出不落敏感载荷的状态模型 |
| CRQ-003 | DEFERRED_TO_HLD 2026-05-18 | 配置后验证是否需要按 `capacity` 接口族继续细分 | HLD 比较统一验证与接口族细粒度验证方案 |
| CRQ-004 | RESOLVED 2026-05-18 | 自动回滚是否需要进入后续场景和需求基线 | 当前不纳入主基线；后续如需要通过新 CR 引入 |

## CP0/CP1/CP2 检查点补齐记录（2026-05-18）

- 已创建 `process/checks/CP0-REQUEST-INTAKE.md`，结论为自动检查通过。
- 已创建 `process/checks/CP1-USE-CASE-COMPLETENESS.md`，结论为自动检查通过但保留开放项。
- 已创建 `process/checks/CP2-REQUIREMENTS-BASELINE.md`，结论为自动预检通过；2026-05-18 已完成 CP2 人工确认。
- 已创建 `checkpoints/CP2-REQUIREMENTS-BASELINE.md`，用于用户审阅 `process/USE-CASES.md` 与 `process/REQUIREMENTS.md` 是否可进入 HLD。
- CP2 确认前，`process/REQUIREMENTS.md` 必须保持 `ready_for_design: false`，不得启动 `meta-se`；2026-05-18 用户确认通过后，已设置 `ready_for_design=true`。

## CP2 确认记录（2026-05-18）

- 用户回复：`通过`
- 确认结果：接受 `UC-05..UC-10` 与 `R-F-012..R-F-021` 作为 HLD 输入基线。
- 状态回写：`process/USE-CASES.md` 与 `process/REQUIREMENTS.md` 已标记为 confirmed；`process/REQUIREMENTS.md` 已设置 `ready_for_design=true`。
- 变更单状态：`process/changes/CR-001.md` 已关闭。
- 下一阶段：进入 `solution-design`，可唤醒 `meta-se` 输出 HLD 与 ADR。
