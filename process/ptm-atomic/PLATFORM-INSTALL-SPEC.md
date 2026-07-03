---
version: "1.0"
status: "confirmed"
created_at: "2026-05-18T14:35:00+0800"
owner: "meta-se"
confirmed: true
confirmed_by: "user-via-CP5-ALL-STORIES-LLD-BATCH"
confirmed_at: "2026-05-18T16:47:38+0800"
applies_to: "atomic-ops production repository"
---

# Platform Install Spec：atomic-ops 原生交付面边界

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-05-18 | meta-se acting_role on default agent | 明确本项目没有独立 `delivery/` 安装包交付面；Story planning 以后续 README 原生目录作为实现目标。 |
| 1.1 | 2026-05-18 | meta-po | CP5 全量 LLD 批量人工确认中，用户接受 U-002 默认建议：接受 README 原生交付面为 STORY-006 强输入，并回填 `confirmed=true`。 |

## 结论

本项目当前不是 Agent/Skill 安装包交付，不生成 `delivery/agents/`、`delivery/skills/`、`delivery/rules/` 或 `delivery/scripts/`。README 定义的原生交付面是唯一正向目标：

```text
atoms/
schemas/
packages/
docs/
src/atomic_ops/
scripts/
pyproject.toml
uv.lock
```

## README 原生交付面

| 目录 / 文件 | 当前用途 | Story planning 决策 |
|---|---|---|
| `atoms/` | 单个 atom YAML 契约 | 后续 Story 可新增防火墙安装、初始化、登录、守卫、配置、验证与批次契约 atom。 |
| `schemas/` | `atomic-op.schema.yaml` 权威结构契约 | 后续 Story 冻结 schema v1.1 字段族；不得在 CP4 阶段修改。 |
| `packages/` | package 过滤视图 | 后续 Story 可新增安装、capacity 配置、批次、验证 package；package 只保存 op_id。 |
| `docs/` | 字段、命名、错误码、工程师手册、用户手册 | 后续 Story 可同步字段、安全边界、使用手册和测试模板。 |
| `src/atomic_ops/` | CLI 源码 | 后续 Story 只允许扩展只读解析、展示或校验；禁止真实设备动作命令。 |
| `scripts/` | 仓库级静态检查 | 后续 Story 可新增 `security_gate_check.py` 或等价检查；禁止连接设备。 |
| `pyproject.toml` / `uv.lock` | Python 包依赖声明和锁定 | 当前计划不要求新增依赖；实现阶段若新增依赖必须通过 `uv` 更新。 |

## N/A：独立安装包边界

| 对象 | 状态 | 原因 |
|---|---|---|
| `delivery/` | N/A | 当前项目不是 meta-flow 自身交付包，且 README 未定义 `delivery/`。 |
| 平台 Agent 安装 | N/A | 本轮不交付 `.codex/agents`、`.claude/agents` 或 OpenClaw agent。 |
| Skill 安装 | N/A | 本轮不交付 `.agents/skills`、`~/.agents/skills` 或 Skill 资产。 |
| 安装器脚本 | N/A | README 的 CLI 安装方式是 `uv tool install .`，不是独立 install.py/install.sh/install.ps1。 |

## CLI 安装事实

README 当前给出的 CLI 安装和开发入口如下，后续 Story 文档必须保持一致：

```bash
uv tool install .
atomic-ops --version
uv run atomic-ops --help
```

约束：

- Python 代码、验证命令、脚本执行统一使用 `uv run` 或 `uv tool`。
- 不使用裸 `pip install` 作为默认入口。
- `atomic-ops sync` 是唯一访问远端 Git 的命令；`list/show/packages/validate` 只读本地缓存。
- 未来新增校验脚本必须是静态检查，不连接防火墙设备，不读取 `.input/` 运行时资产。

## 禁止交付路径

| 路径 / 行为 | 处理 |
|---|---|
| `delivery/` 作为本项目默认交付目录 | 禁止 |
| `.input/capacity` 或 `.input/ngfw-install` 源码复制到产品面 | 禁止 |
| `.input` 的 env、日志、缓存、凭据、运行时资产进入正式产物 | 禁止 |
| CLI 新增 `run/execute/apply/configure` 等真实设备动作命令 | 禁止 |
| docs/atoms/scripts 落入真实 IP、token、cookie、FTP 凭据、原始默认密码 | 禁止；唯一允许显式出现的密码策略值为 `Ngfw@123` |

## Story 影响

| Story | 交付面 | 安装结论 |
|---|---|---|
| STORY-001 | `schemas/`、`docs/`、示例 atom 策略 | 修改原生规范文件，不生成安装包。 |
| STORY-002 | `atoms/`、`packages/`、少量 docs 引用 | 新增原生 atom/package，不安装外部脚本。 |
| STORY-003 | `atoms/`、`packages/` | 新增 capacity 配置与验证契约，不复制 capacity 源码。 |
| STORY-004 | `atoms/`、`packages/`、docs batch 契约 | 新增批次配置契约，不执行设备配置。 |
| STORY-005 | `scripts/`、`src/atomic_ops/` | 增强静态校验和只读 CLI，命令边界不变。 |
| STORY-006 | `README.md`、`docs/`、`CHANGELOG.md` | 更新用户文档，不改变安装机制。 |
