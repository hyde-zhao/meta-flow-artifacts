---
status: draft
version: "1.0"
complexity: standard
selected_option: "A"
confirmed: false
cr_id: "CR-016-atomic-ops-consumption-gap"
split_from: null
companion_hld:
  - "process/HLD-ptm-tde.md（ptm-tde 主 HLD v7.1）"
  - "process/HLD-CR-017.md（CR-017 因子库发现，Step 1.5 设计模式来源）"
created_at: "2026-06-06T00:00:00+08:00"
created_by: "meta-se（hld-designer Skill）"
---

# 高层设计（HLD）：CR-016 — m-analyzer 原子操作消费缺口修复

> 基于 CR-016 CR intake（已 approved）+ atomic-ops CLI 验证结果 + CR-017 HLD Step 1.5 设计模式。

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-06-06 | meta-se | 初始 HLD |

---

## 1. 问题定义

### 问题陈述

m-analyzer Step 2C（检查原子操作支撑）声明"消费全局 atomic-ops（全部 op_id）"，但实际没有任何 CLI 调用机制。Agent 对操作步骤的 `action_source_ref` 只能做精确匹配，未匹配的全部标记为候选。导致 3 条实际已存在的原子操作被误标：

| 候选（误标） | 实为 | 原因 |
|---|---|---|
| `fw_config_subinterface` | `fw_config_interface` | params.type 区分子接口类型 |
| `fw_config_trunk_interface` | `fw_config_interface` | 同上 |
| `fw_config_lag_interface` | `fw_config_interface` | 同上 |

**根因**：Step 2C 只有"精确匹配 action_source_ref"和"标记候选"两层逻辑，完全缺少中间语义匹配层。而因子库有完整的三层跟踪机制（lock→bindings→resolution-report），原子操作完全没有对应基础设施。

### 目标

| 优先级 | 目标 | 度量 |
|--------|------|------|
| P0 | 新增 atomic-ops CLI 查询步骤，一次 `list --format json` 覆盖全部 79 个 op | Step 1.6 执行成功 |
| P0 | 加权分词语义匹配消除 3 个假阳性候选 | `fw_config_subinterface` 等正确匹配到 `fw_config_interface` |
| P1 | 建立 atomic-op-usage 跟踪目录（lock+bindings+resolution-report） | 3 个文件正常生成 |
| P2 | gate-spec + data-flow-spec 对齐 | 文档与行为一致 |

### 约束

| 类型 | 约束 |
|------|------|
| 技术 | 依赖 atomic-ops CLI（已安装，79 ops, tags + parameters_summary 完整） |
| 降级 | CLI 不可用时优雅降级回现有逻辑 |

---

## 2. 架构灰区

| 灰区 ID | 问题 | 影响面 | 状态 |
|---|---|---|---|
| AGA-01 | Step 1.6 插入位置：在 CR-017 Step 1.5 之后、Step 2 之前 | 模块 | resolved（CR intake 已决定） |
| AGA-02 | 语义匹配算法阈值设计 | 模块 / 验证 | resolved（CR016-DQ-02: 加权分词，≥6.0 强匹配） |
| AGA-03 | atomic-op-usage/ 与 factor-usage/ 的关系 | 数据 | resolved（CR016-DQ-03: 完全平行独立） |

---

## 3. 候选方案

### 方案 A（推荐）：Step 1.6 CLI 查询 + 加权分词语义匹配 + atomic-op-usage 跟踪

**核心**：在 Step 1.5 后插入 Step 1.6，运行 `atomic-ops list --format json`，构建 op_id→meta 索引。Step 2C 用加权分词重叠做语义匹配，阈值 ≥6.0 强匹配。建立 `mfq/atomic-op-usage/` 平行跟踪目录。

### 方案 B（备选）：仅精确匹配增强

**核心**：不新增 CLI 查询步骤，仅在 Step 2C 中增加同义词表做精确匹配扩展。不建立跟踪目录。

| 维度 | 方案 A | 方案 B |
|------|--------|--------|
| 修复完整度 | ⭐⭐⭐⭐⭐ | ⭐⭐ |
| 实现复杂度 | ⭐⭐⭐ | ⭐ |
| 与 CR-017 模式一致性 | ✅ | ❌ |

**推荐方案 A**。

---

## 4. 推荐方案

**复杂度**：standard（5 文件，~235 行，涉及 CLI 集成 + 语义算法）

**核心架构**：

```
CR-017 Step 1.5: 因子库清单加载（已实现）
         │
         ▼
Step 1.6 🆕 原子操作清单加载
  ├─ 1.6.1 运行 atomic-ops list --format json
  ├─ 1.6.2 解析 OperationSummary（op_id/description/tags/params）
  ├─ 1.6.3 构建内存索引 op_id→{description,tags,params,...}
  ├─ 1.6.4 生成/校验 atomic-op-lock.yaml
  └─ 1.6.5 CLI 不可用降级
         │
         ▼
Step 2C 修改: 分级语义匹配（替换现有二层逻辑）
  ├─ L1: action_source_ref 精确匹配 → strong-exact-match
  ├─ L2: 加权分词重叠 ≥6.0 → strong-semantic-match
  ├─ L3: 加权分词 3.0-5.9 → weak-semantic-match（人工审查）
  └─ L4: <3.0 → 候选
```

**权重设计**：op_id 分词 3.0 + description 1.0 + tags 2.0 + params 1.5 = 满分 7.5

**产物**：5 文件（m-analyzer + test-point-integrator + agents/ptm-tde.md + gate-spec + data-flow-spec）

---

## 5-7. 模块职责 / 场景模拟 / 关键流程

| 模块 | 职责 |
|------|------|
| m-analyzer Step 1.6（新增） | CLI 查询 + 索引构建 + lock 管理 |
| m-analyzer Step 2C（修改） | L1-L4 分级语义匹配 |
| test-point-integrator Step 7.5.3（新增） | 原子操作候选交叉验证 |
| agents/ptm-tde.md（修改） | 目录布局 +atomic-op-usage/ |
| gate-spec.md（修改） | GATE-1#3 更新 + GATE-3 新增 M9 |
| data-flow-spec.md（修改） | Entity 8 消费卡片 + 8.8 小节 |

### 关键场景模拟

| SIM | 场景 | 结果 |
|-----|------|:--:|
| SIM-01 | 步骤含 "创建子接口"，无 action_source_ref → 语义匹配 fw_config_interface，score=6.6 | PASS |
| SIM-02 | 步骤含 "配置 BFD"，action_source_ref=fw_config_bfd → 精确匹配 | PASS |
| SIM-03 | CLI 不可用 → 降级回现有逻辑 + WARNING | PASS |

---

## 自审与 CP3 输入

| 自审项 | 结果 |
|------|:--:|
| 灰区已处理 | PASS（3 AGA resolved at CR intake） |
| 场景模拟 | PASS（3 SIM） |
| 拆分信号 | N/A（1 Story，无拆分信号） |

---

<!-- CP3 确认记录 -->
## CP3 确认记录

**确认状态**：⬜ 待审核
