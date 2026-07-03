# INPUT-INDEX

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| v0.1 | 2026-05-15 | meta-po | 建立 `.input/capacity` 与 `.input/ngfw-install` 参考输入索引。 |

## 输入边界

| 输入路径 | 类型 | 用途 | 处理策略 |
|---|---|---|---|
| `.input/capacity/` | 外部 Python 自动化项目 | 提炼防火墙容量配置、对象、策略、路由、NAT、带宽、黑白名单、SSL VPN、动态路由、升级日志检查等 atomic-op 候选能力 | 只作为需求、场景和设计参考；不得直接复制源码、环境文件或凭据 |
| `.input/ngfw-install/` | 外部 Skill/CLI 参考项目 | 提炼 NGFW/TGFW 卸载安装、重启等待、健康检查、初始化、Web 改密等安装类 atomic-op 候选能力 | 只作为高风险运维流程参考；不得直接复制破坏性脚本为产品交付物 |

## `.input/capacity/` 摘要

| 维度 | 已识别内容 |
|---|---|
| 核心目标 | 老版本 API 模板积累、不同设备形态容量适配、公共检查、向升级自动化扩展 |
| 数据面 | `data/api/*.json` API 模板；`data/product/*.yaml` 产品容量参数 |
| 代码面 | `src/api_service/` 封装接口、对象、ACL、策略路由、静态路由、动态路由、NAT、带宽、黑白名单、SSL VPN、升级等 |
| 场景面 | `场景测试/` 与 `特性测试/` 包含单机、混合部署、接口、路由、科信接口、升级等脚本 |
| 可借鉴点 | API 模板 + 产品容量覆盖 + 批量生成 payload + 场景串联 + 升级日志关键词检查 |
| 主要风险 | `env.yaml` 含设备地址和凭据；Windows 绝对路径；依赖声明不一致；真实设备直连；缺少 dry-run；多处脚本运行时问题 |

## `.input/ngfw-install/` 摘要

| 维度 | 已识别内容 |
|---|---|
| 核心目标 | 通过串口服务器对 NGFW/TGFW 设备执行卸载、安装、重启后验证和初始化 |
| 文件组成 | `SKILL.md`、`references/ngfwinstall_tool原始需求.md`、`scripts/ngfwinstall_tool.py` |
| CLI 入参 | 必填：`serial-url`、`package-url`、`device-type`、`device-ip`；可选：`password`、`md5`、`ftp-user`、`ftp-password` |
| 主流程 | telnet 连接、环境检查、下载或使用本地安装包、卸载、解压、安装、等待重启、健康检查、初始化、Web 改密和登录验证 |
| 可借鉴点 | 长耗时运维流程拆步、重启等待、健康检查门控、初始化步骤、安装日志路径规则 |
| 主要风险 | 破坏性操作；telnet 明文连接；默认凭据；FTP 凭据；`verify=False`；会删除和重写设备文件；写入本地 `logs/` |

## 禁止直接转入交付物的内容

- `.input/**/.venv/`、`.input/**/.idea/`、`.input/**/.vscode/`、`__pycache__/`、egg-info、日志样本和解释器生成缓存。
- `env.yaml` 中的真实设备地址、认证 token、角色 token、SSH/Web 默认密码、FTP 密码和其他凭据。
- 会直接连接真实设备、卸载系统、删除文件、重启设备或修改认证配置的脚本实现。
- 与当前仓库 README 交付面冲突的 Skill 安装结构、`delivery/` 包结构或外部项目运行目录假设。

## atomic-op 候选能力域

| 候选域 | 来源 | 初步映射方向 | 待 meta-pm 细化 |
|---|---|---|---|
| 配置容量生成 | capacity | 描述批量生成接口、对象、策略、路由、NAT、带宽等操作的参数契约 | 哪些应作为独立 atom，哪些应作为 package 或文档模板 |
| 配置验证 | capacity | 将 ACL、路由、接口、对象存在性或状态检查转为验证型 atom | 当前 schema 是否足够表达验证返回数据 |
| 升级日志检查 | capacity | 将日志关键词扫描和排除规则转为离线验证型 atom | 是否需要新增日志输入参数与错误分类 |
| NGFW 安装准备 | ngfw-install | 将串口连接、环境检查、安装包校验描述为高风险前置 atom | 是否允许定义破坏性操作 atom，是否需要 schema 标记风险等级 |
| NGFW 安装后验证 | ngfw-install | 将 VPP/Agent/curl 健康检查转为验证型 atom | 是否作为安装流程 postcondition，还是独立 atom |
| 安装后初始化 | ngfw-install | 将关闭验证码、启用 SSH、管理路由、license 恢复作为候选初始化操作 | 是否应拆分，是否默认禁止直接执行 |

## 交给 meta-pm 的最小任务

- 基于本索引产出 `process/USE-CASES.md`，明确目标用户、场景主体、成功指标和边界。
- 基于场景产出 `process/REQUIREMENTS.md`，把候选能力转为结构化需求。
- 明确哪些能力只生成 atom YAML，哪些需要 schema 扩展，哪些需要 CLI 或文档支持。
- 将高风险、凭据、安全、dry-run、日志和真实设备操作限制写成可验证约束。
