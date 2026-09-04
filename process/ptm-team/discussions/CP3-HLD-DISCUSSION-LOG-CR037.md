# CP3 HLD 讨论日志 — CR-037（ptm-te 集成 topo 管理）

> 阶段：solution-design（CP3 前置）
> CR：CR-037
> 日期：2026-08-05
> 前置基线：CP2 已 approved（DQ-037-01/02/04）；用户已确认：三层文件结构、平台别名消费 manaul device-reference.md 完整映射、测试造数隔离不进目标项目、方案 A 安装。

## Architecture Gray Areas（advisor 表格）

> 格式：Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch

| 灰区 | Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|------|--------|------|------|----------------|----------------|------------------------------|
| **AG-1** topo-planning 与 topo-config 模块边界 | A: 独立 topo-planning skill + import topo-config 引擎（方案 A，已定）<br>B: 直接扩展 topo-config | A: 职责清晰、topo-config 保持通用<br>B: 少一层封装 | A: 多一个 skill 安装<br>B: topo-config 耦合 ptm-te | 安装（PTM_TE_SKILLS）、skill 结构 | **A**（用户已选方案 A） | topo-config 若需被多消费方复用则保持独立；仅 ptm-te 用时可考虑 B |
| **AG-2** per-link 端口约束接入点 | A: 回溯匹配阶段剪枝（`_check_connectivity`/`_port_meets_requirement` 按 link_id 取约束）<br>B: 仅结果生成后校验 | A: TE 约束提前剪枝，无解早失败<br>B: 实现简单 | A: 需改回溯逻辑<br>B: 白跑后才发现无 TE 口 | topo_mapper.py | **A**（回溯剪枝） | 全局约束（无 per-link）时退化为现有 port_requirement 路径 |
| **AG-3** devices.yaml × physical_pool.yaml 归并策略 | A: host 关联（`device_groups[].firewall.host` ↔ `pool.nodes[].management.host`），池为准、devices 覆盖型号/平台<br>B: 池为准，devices 仅校验存在性 | A: 池承载接线，devices 承载真实设备，联合最完整<br>B: 池全量时不依赖 devices | A: 需定义关联规则与冲突处理<br>B: devices 型号变化不反映 | pool_merge.py | **A**（host 关联，池接线为准 + devices 覆盖设备元数据） | 池已全量承载时降级 B；冲突（同 host 型号不同）以 devices.yaml 为准并 warning |
| **AG-4** NLP → limit 转换边界 | A: agent 解析 NLP → 结构化 limit 参数/文件，CLI 只收结构化<br>B: CLI 直接收 NLP | A: 转换可测、语法文档可验证<br>B: 少一步 | A: agent 需读语法规范<br>B: CLI 解析 NLP 不可控 | topo-planning skill、CLI | **A**（agent 转换，G5） | limit 语法稳定后可考虑 CLI 收简化自然语言子集 |
| **AG-5** 产物 IP 规划来源 | A: 复用 topo-config IPAM（198.18.0.0/15 自动 or per-broadcast-domain 指定）<br>B: 按用例模板 default_ip | A: 多环境隔离、避免冲突<br>B: 简单 | A: 需接 IPAM<br>B: 复用逻辑 topo 默认网段 | exporter.py | **A**（复用 IPAM） | 单环境且用例固定网段时可 B |
| **AG-6** 失败反馈结构化 | A: 结构化限制校验报告（缺设备/缺 TE 端口/链路约束不满足，JSON+人读）<br>B: 纯对话文本 | A: 可作 CP7 验证证据、可断言<br>B: 简单 | A: 定义报告 schema<br>B: 无法机器断言 | exporter.py、CLI | **A**（O2 已采纳） | 反馈报告写入 runs/ 或输出文件 |
| **AG-7** install 首次生成模板副本实现位置 | A: install.py 在安装 ptm-te 时检查并生成 `topology/physical_pool.yaml` 模板副本<br>B: skill 内置脚本首次调用时生成 | A: 安装即就绪<br>B: 不侵入 install.py | A: 改 install.py 安装流程<br>B: 首次运行时才有文件 | install.py、S8 | **A**（install.py 生成，对齐 devices.yaml.example 安装模式） | 目标项目已有文件时跳过（幂等） |

## 已闭合决策（非灰区，直接采纳）

- 三层文件结构（用户认可）
- 平台别名识别 = manaul device-reference.md 完整映射（DQ-037-02）
- 端口兼容矩阵：GE=copper\|fiber，TE/XTE/QTE=fiber-only；TE 口以实际接线为准不硬造（DQ-037-03）
- 测试造数隔离在 ptm-team 测试目录，不注入目标项目（用户纠正）
- 映射失败直接反馈不兜底（用户指示）
- 产物命名：默认 `topology/topology.yaml` + 可选 `--output` 描述性命名（O1）

## 待用户确认

AG-3（归并策略 A）、AG-4（agent 转换）、AG-7（install 生成位置）为本次开放灰区，推荐方案已列；AG-1/2/5/6 已由既有决策闭合并入 HLD。

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|------|------|--------|---------|
| v0.1 | 2026-08-05 | host-orchestrator | 初稿：AG-1~7 advisor 表格 + 已闭合决策 |