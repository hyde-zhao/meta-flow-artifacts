---
story_id: "ST-NRA-02"
title: "权限边界拒绝（外部访问/生产操作）"
status: "ready-for-verification"
priority: "P0"
wave: 2
source_cr: "CR-030"
source_story: "ST-NRA-02"
feature: "FEAT-RA-ANALYSIS"
feature_design_refs:
  - "docs/features/feat-ra-analysis/DESIGN.md"
lld_policy:
  required_level: "technical-note"
  trigger_reasons: []
  rationale: "deny-by-default 拒绝逻辑，正向 Story LLD 覆盖"
depends_on:
  - "ST-RA-01"
output_files:
  - "skills/reverse-analysis/SKILL.md"
created_by: "meta-se"
created_at: "2026-07-16"
---

# ST-NRA-02: 权限边界拒绝

## dev_context

### 技术说明
实现 deny-by-default 拒绝逻辑。当请求涉及凭据读取、非 ITR 外部系统访问或生产写入时，拒绝并输出可审计的拒绝记录 + 替代路径 + 独立 CR 建议。

## validation_context
- 关键验证：凭据请求 → 拒绝；生产写入请求 → 拒绝；外部读取请求 → 拒绝

## acceptance_criteria
1. 凭据/认证头请求被拒绝
2. 外部系统访问请求被拒绝
3. 生产操作请求被拒绝
4. 拒绝记录含替代路径 + 独立 CR 建议

## 技术说明

> **lld_policy**: technical-note
> **设计证据类型**: Story 卡片内嵌技术说明
> **CP5 评审意见 H3**：deny-by-default 是 HLD 七项可信治理的核心安全契约，本技术说明需显式引用正向 LLD 的精确覆盖章节、测试 ID 和接口定义，证明"正向 Story 已覆盖"的断言有据可查。

### 正向 LLD 覆盖追溯

| 拒绝类别 | 正向 LLD 覆盖位置 | 覆盖章节 | 关联测试 ID | 接口定义 |
|---------|------------------|---------|------------|---------|
| 凭据/认证拒绝 | `ST-RA-01-LLD.md` | §2.2 输入校验（凭据字段检测），§8 安全与权限（deny-by-default 入口守卫） | T-RA01-03（非法输入拒绝） | `validate_input()` 拒绝包含 credential/token/api_key/password 的输入 |
| 非 ITR 外部系统访问 | `ST-RA-05.1-INGEST-LLD.md` | §2.1 allowlist 白名单（URL pattern + 参数白名单 + 拒绝认证头），§8 安全（禁止凭据推断） | T-ING-01（allowlist 拒绝非 ITR URL） | `validate_request(url, params)` → ValueError for non-allowlist |
| 生产写入/操作 | `ST-RA-03-LLD.md` | §5 状态机（CA/PA 只产出 draft，不自动分发），§8 安全（文件化只读交接） | T-IMP-07（禁止自动分发） | Approved Improvement Input 不可变 + 下游只读 |
| 自动确认 | `ST-RA-02-LLD.md` | §5 根因四层状态机（raw_statement → AI candidate → evidence-backed → reviewer-confirmed），§8（三线阈值硬阻断） | T-RA02-05（无证据不确认） | `confirm_root_cause()` 仅 reviewer 角色可调用 |

### 设计依据

本 Story 实现 HLD REV-03 的 **deny-by-default** 安全约束（§12：安全性 0 个未授权外部/生产/凭据路径）和 Feature DESIGN §5（安全与治理约束）中的拒绝逻辑。所有禁止操作的拒绝点集中在 `reverse-analysis` Skill 的入口和 ptm-tse Agent 的编排层。

### 文件影响

| 文件 | 操作 | 写入段落 | 补充说明 |
|------|------|---------|---------|
| `skills/reverse-analysis/SKILL.md` | 修改 | 在 ST-RA-01 的资格检查段和 SKILL.md 全局 `禁止事项` 段中追加拒绝规则 | 依赖 ST-RA-01 先建立资格检查段（串行约束已由 CP4 确认） |

### 拒绝矩阵

| 请求类别 | 拒绝条件 | 拒绝输出 | 替代路径 |
|---------|---------|---------|---------|
| 凭据/认证 | 请求包含 credential、token、api_key、password 字段或认证头 | `denial_record: type=credential, reason="deny-by-default"` | 使用脱敏摘要；独立发起 runtime/security CR |
| 非 ITR 外部系统访问 | 尝试 HTTP GET/POST/PUT 到非 allowlist URL | `denial_record: type=external_access, target=<url>` | ITR 固定 GET 已由 F-020 的 itr-ticket-ingestion 处理，不在 reverse-analysis 范围 |
| 生产写入/操作 | 请求修改 ticket 表、执行修复脚本、调用写 API | `denial_record: type=production_write` | 通过 improvement-tracker 产出已批准改进输入，由下游 Agent 消费 |
| 自动确认 | AI 尝试标记 reviewer_confirmed | 见 ST-NRA-01 的根因四层硬阻断 | 必须人工 reviewer 操作 |

### 拒绝记录格式

```yaml
denial_record:
  id: string
  type: enum           # credential | external_access | production_write | auto_confirm
  requested_action: string
  reason: string       # "deny-by-default: <policy reference>"
  alternative_path: string
  cr_suggestion: string  # 建议创建的 CR 类型（如 runtime/security CR）
  timestamp: datetime
```

### 实现策略

- **Skill 层**：在 `禁止事项` 节显式列出上述四类禁止操作，并声明「遇到此类请求时立即停止，输出拒绝记录」
- **Agent 层**（`agents/ptm-tse.md`）：在编排流程中说明「ptm-tse 不持有凭据、不发起外部连接、不执行生产操作；如有需求，引导用户发起独立 CR」
- 拒绝记录写入 `process/` 或 `data/` 的可审计目录

### 异常和回退

- 拒绝是不可绕过的硬阻断，没有例外流程
- 若未来需要特定外部读取授权，必须通过独立 CR 新增 allowlist entry，并在 `PLATFORM-CONTRACTS.yaml` 或授权配置中记录

### 测试入口

| 场景 | fixture | 预期 |
|------|---------|------|
| 凭据请求 | forbidden_request.json（含 api_key 字段） | 拒绝，输出 credential 拒绝记录 |
| 外部系统访问 | 同 fixture（含 URL） | 拒绝，输出 external_access 拒绝记录 |
| 生产写入请求 | 同 fixture（含 UPDATE ticket 指令） | 拒绝，输出 production_write 拒绝记录 |
| 正常分析请求 | eligible_p1_ticket.json | 不触发任何拒绝 |

### 风险与重访条件

| 风险 | 缓解 | 重访条件 |
|------|------|---------|
| 过度拒绝（合法扩展功能被误拦） | 只拒绝显式列出的四类操作 | discovery 阶段发现新的合法操作类型时扩展 |
| Skill 文本描述 vs 实际执行不一致 | CP7 使用 fixture 验证拒绝行为 | 平台 Agent 框架更新时重新验证 |

### 偏离记录

无偏离。完全按 HLD REV-03 deny-by-default 原则和 Feature DESIGN §5 安全约束实现。
