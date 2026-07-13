---
story_id: STORY-024-02
story_name: device-management + device-connection skill
workflow_id: WF-PTM-TEAM-20260520-001
change_id: CR-024-ptm-te-agent
tier: M
wave: W2
depends_on: [STORY-024-01]
blocks: [STORY-024-04]
status: lld-ready
lld_policy: full-lld
feature_design_refs: [ptm-te-agent]
file_ownership:
  - skills/device-management/SKILL.md
  - skills/device-management/reference/device-reference.md
  - skills/device-management/templates/devices.yaml.example
  - skills/device-connection/SKILL.md
  - skills/device-connection/scripts/ssh_exec.py
  - skills/device-connection/scripts/collect_sysinfo.py
parallel_safe: true
hld_ref: process/HLD-CR-024.md
lld_ref: process/stories/STORY-024-02-device-management-connection-LLD.md
---

# STORY-024-02：device-management + device-connection skill

## 1. 目标

新建两个 skill：device-management（元数据：devices.yaml + 型号映射查表）+ device-connection（SSH/Telnet 双轨连接 + 回退 + 系统快照采集）。迁移自 manaul 项目，连接逻辑从 collect_sysinfo.py 提取适配。

## 2. 范围

- **新建文件**：6 个（device-management 3 + device-connection 3）
- **迁移来源**：manaul device-management（元数据可直接迁移）+ collect_sysinfo.py/ssh_exec.py（连接逻辑迁移适配）
- **不改动**：agents/ptm-te.md（S1）、policy-route-execution（S3）、install.py（S4）
- **详细设计证据**：`process/stories/STORY-024-02-device-management-connection-LLD.md`（full-lld 14 章）

## 3. 验收标准

- [ ] AC01：device-management SKILL.md 含设备清单管理 + 型号映射查表（纯元数据，无连接逻辑）
- [ ] AC02：device-management/templates/devices.yaml.example 凭据用 `${ENV_VAR}` 占位（非明文）
- [ ] AC03：device-management/reference/device-reference.md 含型号映射表
- [ ] AC04：device-connection SKILL.md 含 SSH/Telnet 双轨 + 回退 + 快照采集职责
- [ ] AC05：device-connection/scripts/ssh_exec.py（paramiko SSH 执行封装，函数级可调用）
- [ ] AC06：device-connection/scripts/collect_sysinfo.py（系统快照采集，before/after 两点）
- [ ] AC07：脚本声明 `>=3.9,<3.13`（telnetlib 兼容）
- [ ] AC08：连接前置校验（devices.yaml 存在 + ${ENV_VAR} 占位 + 环境变量已设 + IP 可达）
- [ ] AC09：凭据不入库明文（devices.yaml.example + .env.example，.gitignore 忽略实际文件）

## 4. 依赖与门控

- **依赖**：STORY-024-01（agent 骨架定义设备准备流程 [2]）
- **阻塞**：STORY-024-04（安装器需注册 device-management + device-connection skill）
- **并行**：与 STORY-024-03 文件所有权无冲突，W2 可并行
- **dev_gate**：S2 LLD 确认后进入实现
